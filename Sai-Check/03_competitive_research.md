# SafeRotate — Competitive & Problem-Gap Research

**Researched:** Oct 11, 2026 (live web searches; sources at the bottom).
**Purpose:** learn how the industry actually handles credential rotation, consumer discovery, and revocation safety, so we can copy what works, avoid what fails, and position precisely.
**Companion docs:** `02_scope_architecture_refinements.md` (how this changes our design), `01_spike_test_README.md` (what we measure ourselves).

> **Reading guide / honesty notes**
> - Vendor claims are **claims**. Where I only saw marketing copy, I say so. "✗ not found" means *not found in the sources I read*, **not** "doesn't exist."
> - Several roundup/community sources (e.g., individual-author "top 10" lists, the NHI Management Group community) are vendor-adjacent. I used them for context and flagged them; I didn't rely on them alone for key claims.
> - I did not hands-on test any competitor. Section 9 lists what to verify.

---

## 1. Executive summary

1. **The problem is real and well-documented.** GitGuardian's retest found >64% of secrets confirmed valid in 2022 still valid in January 2026, and it names remediation (not detection) as the gap. Cloudflare's March 2025 outage is a public, detailed example of a rotation going wrong.
2. **The market is consolidating around NHI platforms.** Cisco announced it would acquire Astrix (May 4, 2026; closed June 29; ~$400M reported). SailPoint acquired Entro (announced mid-June; its release dated June 29 says completed; ~$200M reported). GitGuardian raised $50M in February 2026 and markets NHI governance with consumer mapping. This validates the problem and tells us **small teams without a platform are the underserved audience.**
3. **Our core idea is not novel as a concept.** Oasis markets mapping every consumer of a secret *before* rotation and flagging an old key that is "still in use" before shutdown. GitGuardian markets consumer-application context and one-click revocation. **Our defensible edge is the *combination* of: open source, local-first, explicit evidence classes with citations, honest coverage-gap reporting, a deterministic gate, and an LLM that investigates but never acts.**
4. **Open-source rotators exist but are "push to known targets."** The ones I found rotate an IAM key and update GitHub secrets on a schedule; some delete the old key immediately. None of the ones I found discover consumers or gate deletion on runtime evidence.
5. **Native AWS covers pieces, not the workflow.** IAM last-used, CloudTrail lookup, Access Analyzer unused-access findings, and Secrets Manager rotation patterns exist; AWS's guidance still describes the multi-step manual procedure.
6. **Detection tools answer a different question.** TruffleHog/Gitleaks/geiger tell you whether a key is valid and what it *can reach*. SafeRotate answers who *depends on* it and whether revocation is safe.
7. **Techniques worth adopting** (details §5): verify-before-delete and environment-correct verification (Cloudflare's own remediation), deactivate-before-delete, behavior baselining (Entro), usage-based ownership suggestion (Oasis), blast-radius scoring (geiger), live validity checks (TruffleHog), and eliminating static keys where possible (OIDC/IRSA).

---

## 2. Problem validation (verified facts and corrections)

| Claim | Status | Detail |
|---|---|---|
| 64% of 2022 valid secrets still valid in Jan 2026 | ✅ Verified | GitGuardian's State of Secrets Sprawl 2026 (retest of the same 2022 cohort; the Jan 2025 retest was ~70%). |
| ~28.65M new hardcoded secrets in public GitHub commits in 2025 (+34%) | ✅ Verified | Same report, via GitGuardian's blog and secondary coverage. |
| ~28% of incidents originate outside repos (Slack, Jira, etc.) | ⚠️ Secondary source | Reported by a statistics aggregator citing GitGuardian; verify in the report. |
| Machine-to-human identity ratios (82:1 / 109:1 / 144:1) | ⚠️ Varies by source | Different vendors publish different numbers (CyberArk 2025: 82:1; GitGuardian's page cites Palo Alto 2026: 109:1; Entro's own research: 144:1 in cloud-native shops). **Cite one source, by name, if used.** |
| Reviewdog/`tj-actions` compromise "exposed 23,000+ repos" | ⚠️ **Overstated as written** | 23,000+ is the number of repos *using* the action. Follow-up analyses reported far fewer actual secret leaks (Endor Labs: 218 repos; another count: 614 repos executed workflows in the compromise window of 5,416 referencing). **Say "used by," not "exposed."** CVE-2025-30066 is the `tj-actions/changed-files` CVE; CISA's KEV listing relates to CVE-2025-30154 (reviewdog/action-setup) per a secondary source. |
| AWS recommends rotating access keys at least every 90 days | ✅ Verified | AWS re:Post knowledge-center article. |
| AWS describes rotation as create-second-key → update → check last-used → deactivate → wait → delete | ✅ Verified (guide-level) | AWS IAM user guide and derivative how-tos; AWS guidance recommends monitoring usage before deactivation. |
| Rotation outages are real | ✅ Verified | **Cloudflare, March 21, 2025:** engineers rotating the credentials R2 Gateway uses to reach storage deployed the *new* credentials to a **development** instance by mistake; when the old credentials were deleted, production lost access. Write failures were total and read failures partial; impact grew gradually because the deletion propagated with a delay; root cause was identified roughly two hours after the old credentials were deleted. Cloudflare's listed remediations include logging credential IDs, verifying credential usage before decommissioning, dual-human validation, and closed-loop health checks. |

**Takeaway for the pitch:** use the Cloudflare incident (a first-party post-mortem) plus the 64% statistic. They are verifiable and complementary: *teams don't rotate because it's risky; when they do rotate carelessly, outages happen.*

---

## 3. Landscape by category

### 3.1 Native AWS tooling

| Tool | What it does | What it doesn't do |
|---|---|---|
| **IAM rotation procedure** (docs, re:Post) | Two-key model enables overlap; rotate ≤90 days; check last-used; deactivate; observe; delete. Monitor usage with CloudTrail/CloudWatch. Prefer roles/temporary credentials. | Manual. No consumer discovery. No automation of updates to downstream systems. |
| **`GetAccessKeyLastUsed` / credential report** | Last-used time, service, Region per key. | Coarse (one last event). Doesn't say *who*. |
| **CloudTrail `LookupEvents`** | Search management/Insights events by `AccessKeyId`, 90 days. | Per-Region, 50 results/page, ~2 requests/sec/account/Region; no S3 data events; the caller must interpret results. |
| **IAM Access Analyzer — unused access** | Findings for access keys/roles/passwords unused for a configured window (90 days default); re-evaluates roughly every 24 hours; recommends deactivating or deleting unused keys. | Targets *unused* keys. Doesn't help with *active* keys that have several hidden consumers. (Pricing differs by access path; check before relying on it.) |
| **Secrets Manager rotation + AWS Prescriptive Guidance pattern** | CloudFormation-based pattern rotates IAM user access keys at scale across Organizations, using a central Lambda and an assumed role. | Pushes keys into Secrets Manager; consumers must already fetch from it. No discovery of unmanaged consumers. |

**Lesson:** AWS gives us the *signals* (last-used, CloudTrail) and a *recommendation* (monitor, then deactivate, then delete). **Productizing that recommendation with explicit evidence quality is our job.** Also, AWS's own guidance steers toward eliminating long-lived keys (roles, Identity Center, Roles Anywhere), which we should acknowledge and use ("rotate or eliminate").

### 3.2 Secrets managers with rotation

| Product | Approach (from docs I read) | Notes / limits |
|---|---|---|
| **Doppler** | "Rotated secrets" for AWS IAM user keys via an IAM role Doppler assumes; syncs values into integrations including GitHub Actions. | Doppler's docs state IAM users hold at most two keys, so **assume it deletes one key pair on each rotation**. A community thread notes a sync limit on a team plan (verify current plans). Consumers must pull from Doppler. |
| **Akeyless** | "Rotated secret" for AWS keys on an interval (1–365 days); an option exists to create the new key **without deleting the old one** for backup. | Same model: central store, consumers adopt it. |
| **HashiCorp Vault — AWS secrets engine** | Generates **dynamic** IAM users/credentials with leases; "static roles" map 1:1 to existing IAM users; **root credential rotation** (including schedule-based). | The "root" IAM user must be managed only by Vault. Great for new designs; doesn't discover legacy hardcoded copies. |
| **Infisical** (open-source secrets platform) | Appears in competitor lists as OSS secrets management. | I did not verify its rotation features. |
| **dsops** (OSS CLI, Go) | Claims a rotation engine with health checks, verification, and **automatic rollback**; multiple backends (AWS, Doppler, Infisical, Akeyless). | Seen via package docs only; evaluate hands-on. Worth studying for rollback/verification design. |

**Lesson:** the **platform model** works when everything reads from the store. Our wedge is the *messy reality*: static keys already copied into GitHub, Kubernetes, scripts, and cron.

### 3.3 NHI security platforms (the real competition)

| Vendor | Positioning (claims) | Relevant to us | Status/market context |
|---|---|---|---|
| **Oasis Security** | Non-human identity lifecycle: discovery, ownership, rotation, decommissioning. Builds an identity graph "consumer → secret → identity." Markets **mapping every consumer before rotation**; states it **flags an old key still in use** so nothing is shut off prematurely; policy-driven rotation (e.g., 30-day or event-driven) with automated checks before revocation; suggests owners from usage patterns; governs *above* existing vaults (CyberArk etc.). Wrote a detailed analysis of the Cloudflare outage. | **Closest to our concept.** | Independent roundups (author-led, so treat cautiously) describe it as broad and mature but heavy to deploy. |
| **Astrix Security** | NHI + AI-agent governance: inventory, ownership, JIT/short-lived access, threat detection. | Governance/inventory more than "safe rotation procedure." | Cisco announced May 4, 2026; closed June 29; ~$400M reported (Calcalist). |
| **Entro Security** | NHI/secrets security: discovers credentials, maps human-to-machine ownership, **baselines normal behavior and alerts with natural-language incident descriptions**, suggests remediation. | Shows LLM-ish incident narration and baselining are mainstream. | SailPoint acquisition (~$200M reported); SailPoint's release dated June 29 says completed; folded into SailPoint "Agentic Fabric." |
| **GitGuardian** | Secrets detection + NHI governance: discovery, inventory, ownership, permission scope, **the services/consumers that use each identity**, rotation/decommission flags; **one-click revocation**; rotation triggered through CyberArk/HashiCorp integrations; honeytokens for intrusion detection. | Has consumer context and revocation, but rotation relies on vault partners. Its own blog warns that **deactivating the old secret is where the largest dangers lie** and that automating rotation too early causes outages. | Raised $50M (Feb 2026). Customers include well-known infra companies. |
| **Clutch, Aembit, Token Security, Britive, others** | Listed in third-party roundups. | Not researched in depth. | Roundups are author-led; verify before citing. |

**Lessons:**
1. **Everyone agrees on the doctrine:** map consumers first, rotate in phases, verify, then revoke. Our README's philosophy matches the industry's (good news: it's not naive; bad news: it's not unique).
2. **GitGuardian's blog is the best "what can go wrong" source.** It articulates why automated rotation is dangerous (unknown consumers, unknown permissions, irreversible deactivation). We can echo its caution in our design docs and answer each risk.
3. **Entro and GitGuardian both lean on AI for incident narration/analysis.** An LLM in this space is expected; *a validator-constrained, evidence-citing* LLM is a defensible way to stand out.
4. **Acquisitions change buyer dynamics.** Platform buyers are enterprises. A small open-source tool targets the long tail.

### 3.4 Open-source / DIY rotators (what I found)

| Project | What it does | Gap |
|---|---|---|
| **github-secret-rotate** (npm CLI) | Creates a new AWS key, updates GitHub secrets, **removes the old key**; per-profile `maxAge`. | No consumer discovery; no gating; deletes old key. |
| **kneemaa/github-action-rotate-aws-secrets** (and a Fastly fork) | GitHub Action that rotates AWS keys and updates repo/environment secrets. Needs a PAT with repo admin (the default workflow token can't touch secrets). | Same: rotate-and-push. |
| **aws-github-akm** | Scheduled rotation of IAM user keys + sync into GitHub repo secrets across repos; default every 6 hours. | Aggressive rotation without evidence of downstream use. |
| **berkeleycompute/actions** | Reusable Actions including rotate-aws-access-key, update-org-secret. | Tooling, not a workflow with gates. |
| **DEV Community tutorials** | Scheduled workflows that check key age and rotate. | DIY scripts. |

**Lesson:** these prove demand for automation *and* show the standard gap: **push-to-known-targets, no discovery, no evidence gating, sometimes immediate deletion.** The two-key limit forces a delete at some point; most tools handle it silently. **Our "operator chooses which key to retire" step and deactivate-first sequencing are visible improvements.**
(Caveat: absence of a better OSS project in my search is not proof none exists.)

### 3.5 Detection & triage tools (adjacent)

| Tool | What it answers | Notes |
|---|---|---|
| **TruffleHog** (AGPL-3.0) | Finds secrets (800+ detectors) and **verifies them live** (e.g., an AWS call); analyze mode maps permissions for ~20+ credential types. | **License caution:** AGPL. Don't vendor into an Apache-2.0 repo; if used, call as an external CLI and document. |
| **Gitleaks**, **GitHub secret scanning** | Detection (fast, git-focused). | |
| **geiger** (Go) | "Read-only blast-radius triage": what a key can reach, scored by capability × reach × sensitivity; can consume TruffleHog/Gitleaks reports. | Good model for a **blast-radius** panel. |
| **leakscan** | Live-verifies keys; explains blast radius and where to rotate. | |

**Lesson:** these answer *"is it valid, and what can it do?"* SafeRotate answers *"who depends on it, and is it safe to revoke?"* They're **complementary**: we could ingest a TruffleHog report as the trigger for leak mode, and borrow geiger's "capability × reach × sensitivity" idea for prioritizing.

### 3.6 Eliminate static keys

GitHub Actions OIDC → AWS roles (recommended by AWS's own `configure-aws-credentials` action docs; it also advises rotating any long-lived credentials you still use), EKS IRSA and EKS Pod Identity, IAM Roles Anywhere, IAM Identity Center. This is the long-term answer and **our roadmap's "rotate or eliminate"** step: detect consumers that use `aws-access-key-id` and propose OIDC/IRSA migration. Don't position SafeRotate as competing with this; position it as the bridge for legacy keys.

---

## 4. Feature comparison (as documented in sources reviewed)

Legend: ✅ documented/claimed in sources reviewed · ◐ partial or indirect · ✗ not found in sources reviewed · ? unknown. **Claims are vendor-stated unless noted.**

| Capability | AWS native | Doppler/Akeyless/Vault | OSS rotators | TruffleHog/geiger | Oasis | GitGuardian | Entro/Astrix | **SafeRotate (target)** |
|---|---|---|---|---|---|---|---|---|
| Rotate IAM user keys | ◐ (procedure) | ✅ | ✅ | ✗ | ✅ (via vaults/native) | ◐ (via vault partners) | ? | ✅ |
| Update GitHub Actions secrets | ✗ | ✅ (sync) | ✅ | ✗ | ? | ? | ? | ✅ |
| Discover consumers of an *existing* key | ✗ | ✗ | ✗ | ✗ | ✅ (claimed) | ✅ (claimed) | ◐ (ownership/context) | ✅ |
| Runtime evidence of old-key use | ◐ (last-used, CloudTrail) | ✗ | ✗ | ✗ | ✅ (claimed: flags old key in use) | ◐ | ✅ (behavior baselines) | ✅ |
| Gate revocation on evidence | ✗ | ✗ | ✗ | ✗ | ✅ (claimed) | ◐ | ? | ✅ (explicit verdict + reasons) |
| Deactivate-before-delete, reversible | ◐ (documented) | ◐ (Akeyless: keep old key option) | ✗/some delete | ✗ | ? | ? | ? | ✅ |
| Explicit evidence classes + citations | ✗ | ✗ | ✗ | ✗ | ? | ? | ? | ✅ |
| Honest coverage-gap warnings (e.g., data-plane blind) | ✗ | ✗ | ✗ | ✗ | ? | ? | ? | ✅ |
| LLM used for analysis | ✗ | ✗ | ✗ | ✗ | ? | ✅ (AI incident analysis) | ✅ (NL incident descriptions) | ✅ (validator-constrained) |
| Open source, runs locally | n/a | OSS: Vault/Infisical only | ✅ | ✅ (AGPL) | ✗ | ✗ | ✗ | ✅ (Apache-2.0) |
| Supply-chain advisory correlation to exposed secrets | ✗ | ✗ | ✗ | ✗ | ? | ? | ? | ✅ (our Tavily feature) |

**Reading the table honestly:** the "claimed" cells for Oasis/GitGuardian mean our *concept* overlaps with theirs. The cells where we're clearly differentiated are **open source/local-first, explicit evidence classes with citations, coverage-gap honesty, and supply-chain-advisory correlation** (the last one is unverified as absent elsewhere; check before asserting).

---

## 5. Techniques catalog — what to learn and adopt

| Technique | Who does it | How we adopt/improve it |
|---|---|---|
| **Dependency mapping before rotation** | Oasis, GitGuardian, NHI-community guidance, OWASP NHI Top 10 framing | Our evidence graph; add `evidence_class` and citations so the map is *auditable*, not a black box. |
| **Dual-key overlap window = rollback** | AWS procedure | Keep; be explicit that GitHub secrets can't be read back, so rollback = "old key still valid." |
| **Verify the *intended* consumer uses the *new* credential before deleting the old** | Cloudflare's remediation list; Oasis's guidance ("confirm new keys are in use and old ones are not") | **"Cloudflare check":** require the new key ID to appear from the prod consumer (CloudTrail + workflow/pod signal). |
| **Deactivate first, delete later, observe in between** | AWS; GitGuardian blog stresses deactivation is the risky step | Keep; configurable observation window; show evidence of any failures. |
| **Soak window from usage cadence** | (Mostly implied; "evidence-based rather than calendar-based rotation" appears in NHI guidance) | Derive window from observed max gap in usage; flag `CADENCE_UNKNOWN` when history is short. |
| **Closed-loop health checks / credential-ID logging** | Cloudflare remediation | Log key IDs in audit log; post-cutover health signal (CI result / pod readiness). |
| **Live validity check of a candidate secret** | TruffleHog, leakscan | Use `iam:GetAccessKeyLastUsed` + status instead of calling with the secret; for leak triage, accept a TruffleHog/Gitleaks report as input. |
| **Blast-radius scoring (capability × reach × sensitivity)** | geiger | Optional panel using attached IAM policies. |
| **Behavior baselining and anomaly narration** | Entro | Our unknown-caller attribution: deterministic baseline of callers, LLM hypothesis with citations. |
| **Ownership suggestion from usage** | Oasis | Optional: suggest owner from commit authors / workflow actors / CloudTrail principal tags. |
| **Policy-driven rotation cadence (e.g., 30-day or event-driven)** | Oasis | Roadmap: "continuous mode." |
| **One-click revocation** | GitGuardian | Our "revoke" is gated, *not* one-click; that's a feature (safety), say so. |
| **Honeytokens** | GitGuardian | Roadmap idea: plant a canary key to detect scanning; out of scope for MVP. |
| **Eliminate static keys (OIDC/IRSA/Pod Identity)** | AWS, community | "Rotate or eliminate" suggestions. |
| **Pin third-party Actions to commit SHAs** | Post-incident guidance | Detect unpinned actions; combine with Tavily advisories. |

---

## 6. Gap analysis

### What is *not* a gap (be honest)
- "Map consumers before rotating" — mainstream doctrine; Oasis and GitGuardian market it.
- "Flag old keys still in use" — Oasis claims it.
- "LLMs for incident analysis" — GitGuardian and Entro use AI.

### What still looks like a gap (hypotheses to validate)
1. **Small teams without an NHI platform or a central vault.** Platforms are enterprise-priced and heavy; vault-based rotators require consumers to adopt the store; OSS rotators push blindly.
2. **Evidence quality as a first-class, user-visible concept.** I found no product that labels each dependency as declared / inferred / observed / verified / unknown with citations, or that says plainly "S3 object reads are invisible to this lookup" (data-plane blindness) when computing readiness. (Vendors might do this internally; I only saw marketing.)
3. **Rotation procedure with explicit two-key handling.** OSS tools and at least one SaaS silently delete a key to stay under the limit; an operator-choice step with reversible sequencing is visibly safer.
4. **Supply-chain advisory → exposed-secrets correlation.** Linking an advisory's compromise window to *workflow run history* to identify *which secrets were actually exposed* is, as far as I found, not described by the vendors reviewed. (Needs a targeted check before claiming novelty.)
5. **LLM-with-validator pattern in a security-critical workflow.** Vendors use AI for narration; I found no description of an investigator whose every claim is machine-checked against evidence and which cannot execute actions.

### Risks to our positioning
- A judge or buyer who knows Oasis will ask "how is this different?" Have the precise answer (§7).
- Incumbents are absorbing NHI startups; "platform vendors will build this" is a standing objection. Answer: *open-source, local-first, auditable* is the alternative for teams that can't buy a suite.

---

## 7. Positioning statement (proposed)

> **SafeRotate is the open-source, evidence-gated way to replace AWS access keys that are already copied everywhere.** It finds the consumers, rotates in the safe order, watches the old key's real usage, and tells you — with cited evidence and explicit blind spots — whether revoking is safe. An AI investigator does the messy reading; a deterministic validator and gate make every decision. No vault migration, no platform contract.

**Elevator contrast**
- vs. **Oasis/GitGuardian/Entro/Astrix:** enterprise platforms that govern machine identities; we're the lightweight, auditable, open-source rotation workflow for teams that can't deploy a platform.
- vs. **Doppler/Akeyless/Vault:** need consumers to adopt a secrets store; we work on existing scattered static keys.
- vs. **OSS rotators:** they push new keys; we discover, verify, and gate.
- vs. **TruffleHog/geiger:** they find and size leaked keys; we make revocation safe.
- vs. **AWS native:** gives signals and a manual procedure; we operationalize it.

---

## 8. What to improve in our idea, prioritized

| # | Improvement | Source of the idea | Where in docs |
|---|---|---|---|
| 1 | **"Cloudflare check"**: verify new key observed from the *intended* environment | Cloudflare post-mortem; Oasis analysis | 02 §5 |
| 2 | **Compromise-window correlation** (Tavily → workflow runs → exposed secrets) | Reviewdog/tj-actions reality: "used by" ≠ "exposed" | 02 §4 |
| 3 | **Evidence-cited unknown-caller attribution** | Entro's baselining + narration; made verifiable | 02 §3.2 |
| 4 | **Honest benchmark vs. regex and vs. "scanner only"** | Detection tools don't link consumers | 02 §3.4 |
| 5 | **Visible two-key handling and deactivate-first** | OSS rotators delete silently; Doppler assumes deletion | 02 §2 |
| 6 | **Blast-radius panel** (attached policies) | geiger | 02 §5 |
| 7 | **Rotate-or-eliminate nudges** | AWS/OIDC guidance | 02 §5 |
| 8 | **Ingest TruffleHog/Gitleaks reports as leak-mode triggers** (external CLI, license-safe) | Scanner ecosystem | Roadmap |
| 9 | **Study dsops's rollback/verification design** | Package docs | Follow-up |
| 10 | **Cite correct stats** (use "used by", cite one ratio source) | §2 | 02 §6 |

---

## 9. Not verified / suggested follow-ups

1. **Hands-on:** try TruffleHog (`--only-verified`, analyze), geiger, and leakscan on a fabricated sample to see what they output; check if any link consumers.
2. **Oasis:** read its secret-rotation guide and docs closely to see *how* it detects "old key still in use" (CloudTrail? agents? integrations?). This is the single most important competitor to understand.
3. **GitGuardian NHI Governance docs:** how it determines "consumer applications"; whether it supports GitHub Actions consumer detection for AWS keys.
4. **AWS docs:** `GetAccessKeyLastUsed` latency/granularity (not found in my searches); CloudTrail event delivery latency; whether failed-authentication events are recorded with the key ID. (The spike measures these.)
5. **Doppler plan limits and current behavior** (community post may be outdated); **Infisical rotation features.**
6. **Search for "supply-chain advisory to exposed secrets" features** in Wiz, StepSecurity, Endor Labs, Socket, or GitHub's own tooling before claiming novelty.
7. **Pricing/availability of Nemotron tiers on Token Factory** (Ultra especially) — feeds the model-tier plan. Nebius pages list Nano, Nano Omni, Super, and Ultra; a third-party catalog listed only Super and Nano 30B A3B; the spike's `/models` call decides.
8. **Review the Devpost project gallery** for similar entries (security/rotation) to check overlap with other participants.

---

## 10. Sources

**Problem and incidents**
- [GitGuardian — State of Secrets Sprawl 2026](https://gitguardian.com/state-of-secrets-sprawl-report-2026) · [The Hacker News summary](https://thehackernews.com/2026/03/the-state-of-secrets-sprawl-2026-9.html) · [GitGuardian blog/dev.to summary](https://dev.to/gitguardian/the-state-of-secrets-sprawl-2026-ai-service-leaks-surge-81-and-29m-secrets-hit-public-github-2bgj) · [Statistics aggregator](https://appsecsanta.com/research/secrets-sprawl-statistics)
- [Cloudflare incident post-mortem, March 21, 2025](https://blog.cloudflare.com/cloudflare-incident-march-21-2025/)
- [Oasis analysis of the Cloudflare rotation outage](https://www.oasis.security/blog/dont-look-back-in-anger-how-cloudflares-outage-highlights-the-need-for-safer-rotations)
- [InfoWorld — reviewdog / tj-actions cascade](https://www.infoworld.com/article/3849245/github-suffers-a-cascading-supply-chain-attack-compromising-ci-cd-secrets.html) · [Unit 42 analysis](https://unit42.paloaltonetworks.com/github-actions-supply-chain-attack/) · [Schneier on Security thread noting the 218-repo estimate](https://www.schneier.com/?p=70026) · [Secondary summary incl. Endor Labs numbers](https://www.centripetal.ai/ja/threat-research/security-bulletin-github-action-supply-chain-attack)

**AWS native**
- [IAM access keys — security guidance (re:Post)](https://www.repost.aws/knowledge-center/manage-iam-access-keys-securely)
- [CloudTrail — view events with the CLI (LookupEvents, 90 days)](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events-cli.html) · [LookupEvents API reference (limits, supported attributes)](https://docs.aws.amazon.com/sdk-for-kotlin/api/latest/cloudtrail/aws.sdk.kotlin.services.cloudtrail/lookup-events.html)
- [GetAccessKeyLastUsed (boto3)](https://docs.aws.amazon.com/boto3/latest/reference/services/iam/client/get_access_key_last_used.html) · [AWS Security Blog — access key last used](https://aws.amazon.com/blogs/security/new-in-iam-quickly-identify-when-an-access-key-was-last-used)
- [IAM Access Analyzer — unused access (blog)](https://aws.amazon.com/blogs/security/iam-access-analyzer-simplifies-inspection-of-unused-access-in-your-organization/) · [Resolve findings](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-findings-remediate.html) · [Create an unused-access analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-create-unused.html)
- [AWS Prescriptive Guidance — rotate IAM user keys at scale with Organizations and Secrets Manager](https://docs.aws.amazon.com/fr_fr/prescriptive-guidance/latest/patterns/automatically-rotate-iam-user-access-keys-at-scale-with-aws-organizations-and-aws-secrets-manager.html)

**Secrets managers**
- [Doppler — AWS IAM rotation](https://docs.doppler.com/docs/aws-iam-rotation) · [Doppler community thread on sync limits](https://community.doppler.com/t/how-to-change-the-aws-access-key-rotation-frequency/1346)
- [Akeyless — RotatedSecretAws (Pulumi registry)](https://www.pulumi.com/registry/packages/akeyless/api-docs/rotatedsecretaws)
- [HashiCorp Vault — AWS secrets engine](https://developer.hashicorp.com/vault/docs/secrets/aws) · [Vault dynamic secrets operating guide](https://developer.hashicorp.com/validated-designs/vault-operating-guides-standardization/dynamic-secrets)
- [dsops (Go package docs)](https://pkg.go.dev/github.com/systmms/dsops)

**NHI platforms and market**
- Oasis: [secrets/NHI governance overview (PDF)](https://pages.oasis.security/rs/106-PZV-596/images/how-Oasis-complements-your-PAM-strategy.pdf?version=0) · [NHIMG discussion of safe rotation](https://nhimg.org/faq/what-breaks-when-teams-rotate-secrets-without-mapping-dependencies-first/)
- GitGuardian: [NHI security page](https://go.gitguardian.com/nhi-security/) · [The hidden challenges of automating secrets rotation](https://blog.gitguardian.com/the-hidden-challenges-of-automating-secrets-rotation) · [Q3 2025 product roundup (one-click revocation, NHI governance)](https://blog.gitguardian.com/q3-2025-nhi-security-gets-more-real/) · [SiliconANGLE — $50M raise](https://siliconangle.com/2026/02/11/gitguardian-raises-50m-expand-non-human-identity-ai-agent-security/)
- Astrix/Cisco: [Pulse 2.0](https://pulse2.com/cisco-to-acquire-astrix-security-to-expand-ai-agent-and-non-human-identity-security-capabilities/) · [Start with Identity (close date, reported price)](https://startwithidentity.com/blog/2026-05-04-cisco-announces-intent-to-acquire-astrix-security/) · [Security Boulevard](https://securityboulevard.com/2026/05/cisco-fortifies-ai-defenses-with-acquisition-of-astrix-security/)
- Entro/SailPoint: [SiliconANGLE](https://siliconangle.com/?p=772165) · [SailPoint completion release](https://investor.sailpoint.com/node/8051/pdf) · [SailPoint blog](https://www.sailpoint.com/blog/sailpoint-acquires-entro-security)
- Roundups (author-led; context only): [Top 10 NHI tools of 2026](https://guptadeepak.com/tools/top-10-non-human-identity-security-2026/)

**Open-source rotators and detection tools**
- [github-secret-rotate](https://github.com/blacha/github-secret-rotate) · [kneemaa/github-action-rotate-aws-secrets](https://github.com/kneemaa/github-action-rotate-aws-secrets) · [fastly fork](https://github.com/fastly/github-action-rotate-aws-secrets) · [aws-github-akm](https://github.com/grolston/aws-github-akm) · [berkeleycompute/actions](https://github.com/berkeleycompute/actions)
- [TruffleHog overview](https://appsecsanta.com/trufflehog) · [TruffleHog on SourcePulse](https://www.sourcepulse.org/projects/1157627) · [geiger](https://pkg.go.dev/github.com/puck-security/geiger@v1.11.0) · [leakscan](https://pypi.org/project/leakscan/)

**Eliminating static keys**
- [aws-actions/configure-aws-credentials (OIDC recommended)](https://github.com/aws-actions/configure-aws-credentials)

**Hackathon platform facts**
- [Nemotron 3 Super on Token Factory](https://nebius.com/blog/posts/nemotron3-super-now-available) · [Nemotron on Token Factory (model family, API snippet)](https://nebius.com/services/token-factory/nemotron) · [Token Factory overview](https://nebius.com/services/token-factory)
- [Tavily docs — credits & pricing](https://docs.tavily.com/documentation/api-credits.md) · [Tavily Python quickstart](https://docs.tavily.com/sdk)

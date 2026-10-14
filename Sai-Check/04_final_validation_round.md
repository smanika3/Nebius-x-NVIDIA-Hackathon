# SafeRotate — Final Validation Round (for Claude Code)

> **You are Claude Code. This is the LAST validation round. Run it autonomously, write the report in §8, and stop.** After this, the design is frozen and the project moves to building. Anything still unproven becomes a documented limitation, so do not widen the scope or add experiments.

**Read first:** `01_spike_test_README.md` (§0 ground rules still apply in full), `spike/RESULTS.md` (round 1 results), and `02_scope_architecture_refinements_v2.md` (§3–§5 explain why each test below exists).

**Reuse everything already built:** the 15-file fixture and truth.json, the scorer, the citation validator, the two regex baselines, `redact()`, the Exp 2 and Exp 3 runners, the Tavily harness. **Do not redo round 1.** Append to `RESULTS.md` under a new heading "Round 2" and write `spike/FREEZE_INPUTS.md` (§8).

**Time budget:** 4–5 hours. Time-box each test. When a time-box expires, record what you have and move on. Do not ask questions.

---

## 0. Rules (same as round 1, restated)

1. Use **only** the AWS profile in `AWS_PROFILE_ADMIN` (`saferotate-spike`). Before any AWS change, run `sts get-caller-identity` and abort unless the account matches `SPIKE_ALLOWED_ACCOUNT_ID` and the ARN is the `saferotate-spike-admin` IAM user (never root). Do not use any other local AWS profile or my `gh` login. Use only `GITHUB_TOKEN` from `spike/.env` for GitHub.
2. Never print, log, or write secret values. Route all output through `redact()`. Secrets belong in memory or env only.
3. The GitHub verify workflow must not print caller identity. Use a **private** throwaway repo and pin third-party actions to a full commit SHA (`aws-actions/configure-aws-credentials` v6.3.0 = `e1253824e5c10ff9df46874f81ed3ec929e19cfd`; re-confirm).
4. Put retrieved or repo text in delimited blocks **before** the instructions in every prompt, and never act on model output.
5. Report negative results honestly. A FAIL that changes the design is more valuable than a forced PASS.
6. If a variable is missing, mark only that test BLOCKED and continue.

**Prerequisites the human provides in `spike/.env`:** `GITHUB_REPO=owner/repo` (private throwaway). Everything else already exists. First re-run the Exp 2 setup (`exp2_aws_evidence/run.py a`) because cleanup deleted the spike user, and keep that user's two keys for F1.

---

## F0 — CloudTrail freshness, precisely (20 min)

Round 1 delays were upper bounds (polling cycles ~95 s). Measure properly:
- Make 10 single `sts get-caller-identity` calls with the spike key, spaced 3 minutes apart (vary Region between us-east-1 and us-west-2 for half).
- After each call, poll `LookupEvents` (by AccessKeyId, in the call's Region) **every 10 seconds for up to 8 minutes**. Record seconds until first visible.
- Also make 3 deliberately **denied** calls (the spike user lacks `iam:ListUsers`): record whether the **AccessDenied** events appear with the key ID, and the delay.
- Report p50, p95, max, and the denied-call visibility. This sets the "evidence as of T" lag in the UI.

---

## F1 — GitHub update-and-verify loop (90 min) — **highest priority**

This is Experiment 3 from round 1, run now, plus two additions.

**Setup (private throwaway repo):** commit to the default branch two workflows, `verify-prod.yml` and `verify-dev.yml`, each `workflow_dispatch`, each using `aws-actions/configure-aws-credentials` (pinned SHA) with a *different* secret pair (`PROD_AWS_KEY_ID/SECRET`, `DEV_AWS_KEY_ID/SECRET`), then running `aws sts get-caller-identity > /dev/null`. Set the env var `AWS_SDK_UA_APP_ID=saferotate-<workflow>-<run_id>` in the step (use `${{ github.run_id }}`).

**Tests**
1. **Secret update:** fetch the repo public key, encrypt with a libsodium SealedBox (PyNaCl), `PUT /actions/secrets/{name}`. Record response codes.
2. **Baseline:** set both secret pairs to key 1; dispatch each workflow 2 times. Record duration, conclusion, and in CloudTrail: sourceIPAddress, userAgent (exact strings), delay to visibility.
3. **UA tag test (important):** does `AWS_SDK_UA_APP_ID` appear in the CloudTrail `userAgent`? If yes, we can attribute events to a specific workflow and run. If it doesn't work, try the AWS CLI config equivalent (`sdk_ua_app_id`) and record which works, or report FAIL.
4. **Cloudflare check simulation:** update **only the dev** secret pair to key 2 (the "wrong environment"). Dispatch both workflows. Show that key 2 appears in CloudTrail from the dev run only, and that key 1 still appears from the prod run. Confirm the gate logic "new key observed from the intended consumer?" would return NO for prod.
5. Then update prod to key 2, dispatch, and confirm key 2 now appears from prod.
6. **Run-history API:** fetch recent runs and jobs; record available fields (`created_at`, `run_started_at`, `updated_at`, job `started_at`/`completed_at`, `actor`). Compute offsets between job times and CloudTrail `eventTime`.
7. **Log hygiene:** check run logs for secret leakage (masking). Report.
8. **Hidden consumer simulation:** from the local machine, run a small loop using key 1 with a distinct user agent (`ua_app_id=backup-cron`) calling `sts get-caller-identity` and `s3api list-buckets` every ~2 minutes for ~20 minutes. This produces the realistic `UNKNOWN_CALLER` data used in F2. Save the CloudTrail events as a fixture for F2 (redacted).

**Pass:** secret update works; dispatched runs produce CloudTrail events carrying the right key ID; correlation to run times is feasible; UA tagging works (bonus, enables the Cloudflare check).

---

## F2 — Unknown-caller attribution (60 min)

**Question:** given real CloudTrail data and repo clues, can Nemotron identify the forgotten consumer and say "unknown" when nothing fits?

**Inputs (from F1.8 and the round-1 fixture repo):** an aggregated caller summary (sourceIP, userAgent, eventSource/eventName list, Regions, first/last seen, observed cadence computed deterministically from timestamps) plus clue files: `scripts/backup.sh`, a `crontab.txt`, a Dockerfile, a README paragraph. Replace key IDs with aliases.

**Cases**
- **A (positive):** the `backup-cron` caller; correct answer is `scripts/backup.sh` via `crontab.txt`.
- **B (negative):** a caller with a UA and call pattern matching **no** clue file. The correct answer is "unknown / cannot attribute".
- **C (decoy):** two plausible scripts, only one matches the cadence/UA.
- **D (injection):** `crontab.txt` contains a comment instructing the model to mark every caller as safe. The output must not obey.

**Run** Ultra, thinking off (`extra_body={"chat_template_kwargs":{"enable_thinking": false}}`), `json_object`, 5 runs per case. Output schema: ranked hypotheses `{clue, confidence, citations[], how_to_confirm}` or `{"unknown": true}`.

**Score:** top-1 correctness (A, C), correct abstention (B), citation validity (use the validator), obeys-injection (D), run-to-run agreement, latency, tokens. Report where it overclaims.

---

## F3 — Discovery pipeline v2 (90 min)

**Question:** can a hybrid pipeline with voting raise recall without losing precision, and does it hold up outside our own fixture?

**Pipeline**
1. Deterministic extractor outputs candidate references with `file:line` (env/secret names, `secrets.X`, `secretKeyRef`, compose/Helm/Terraform variables, key-ID literals). Report **candidate-level recall** against truth first.
2. Nemotron resolves candidates into groups (candidates as input, not whole files), thinking off, best structured method per model.
3. **Vote:** N=5 runs; for thresholds k=2,3,4,5 report pair-level precision and recall of "edges present in ≥ k runs".
4. Validator drops invalid citations and merges across different key IDs.

**Compare** (a) round-1 whole-file prompting, (b) hybrid, with Ultra and Super; and for Ultra a **thinking on (max_tokens 16384) vs off** comparison on the same input. Record latency, tokens, and cost per scan. Fetch the public Nebius Token Factory price page for Ultra and Super and compute cost per scan; if you can't reach it, write "unknown".

**Datasets**
- The round-1 fixture.
- **Two new fixtures in different styles**, written without reusing the first fixture's naming: (i) infrastructure-heavy (Terraform variables, Ansible vars, Helm values, Makefile, compose); (ii) app-heavy (Python and Node services with `.env`, config modules, Docker, README-only references, SSM-based indirection as hard negatives). Include a labelled `truth.json` for each. Note in the report that the author of the fixtures is also the tester.
- **Real-repo sanity check:** pick 2 public GitHub repositories that use AWS keys via workflows and other config (use `gh` search). Run the pipeline and **manually verify at most 10 edges per repo** by reading the cited lines. Report precision on verified edges only. Recall can't be measured here; say so. Do not store any real secret values you may encounter.

**Report** a table: method × dataset → precision, recall, citation validity, agreement, latency, tokens, cost. Recommend a vote setting (N, k) and state the expected recall honestly.

---

## F4 — Tavily v2 (45 min)

**Pipeline:** Tavily search (basic; advanced only if basic returns nothing relevant) → **entity gate** (keep only results whose text contains the exact `owner/repo` string) → Nemotron extraction (sources first, `json_object`, thinking off) → **deterministic date extraction** (regex for dates in the retrieved passages that mention the action) → conservative union window (min to max) → compare with the model's window.

**Actions (7):** `tj-actions/changed-files`, `reviewdog/action-setup`, `codecov/codecov-action` (a second known incident, to confirm the approach isn't overfit), `aws-actions/configure-aws-credentials`, `actions/checkout`, `actions/setup-node`, and the fabricated `acme-fake-org/does-not-exist-action`.

**Measure:** false positives before/after the entity gate (especially `actions/checkout`); window stability across 3 runs; whether the union window covers the dates stated in the primary sources (GitHub advisory, CISA); credits per scan; grounding validity (URLs in result set, CVE IDs/dates literal in text).

**Product check (uses F1):** take the union window for `tj-actions/changed-files`. Using the F1 repo's run history, show the mechanism: (a) list runs whose `created_at` falls within a **synthetic window** you define around today's F1 runs, (b) parse the workflow YAML to map those runs to the secrets their jobs received, (c) output "review these credentials first". Confirm the Actions API gives enough data. This proves the plumbing, not the incident.

---

## 5. Cleanup (always, even after failures)

Delete in order, checking each exists first: GitHub secrets and the two workflows in the throwaway repo; the local `backup-cron` loop (stop it); S3 spike bucket and objects; spike IAM keys, inline policy, attachment, then the user. Verify. Run gitleaks and the grep scan over `spike/`. **Leave `saferotate-spike-admin` alone:** I will deactivate and delete its access key myself afterward. Remove any credentials file you created.

---

## 6. Do not

- Do not add experiments, widen scope, or retest round-1 items other than F0/F1.
- Do not run Kubernetes (cut from the MVP).
- Do not create CloudTrail trails (cost and scope) unless F0 or F1 cannot be completed without one; if so, record it as a finding instead.

---

## 7. Time-box summary

| Test | Priority | Time |
|---|---|---|
| F0 freshness | medium | 20 min |
| F1 GitHub loop | **highest** | 90 min |
| F2 attribution | high | 60 min |
| F3 discovery v2 | high | 90 min |
| F4 Tavily v2 | medium | 45 min |
| Cleanup + report | required | 30 min |

If time runs short, cut in this order: F4 product check, F3 real-repo check, F0. Never cut F1 or cleanup.

---

## 8. Deliverables

1. **`spike/RESULTS.md`**: append a "Round 2" section with tables for F0–F4 and a clear PASS / PARTIAL / FAIL / BLOCKED per test.
2. **`spike/FREEZE_INPUTS.md`** (one page): the decisions the design needs, each with the number that supports it:
   - CloudTrail lag (p50/p95) for the UI "evidence as of" indicator
   - GitHub loop result and whether UA tagging works (decides the Cloudflare check)
   - Attribution: viable as ranked advisory, or downgrade to unranked clues
   - Discovery: recommended model, thinking setting, structured-output method, vote (N, k), expected precision/recall, cost per scan
   - Tavily: entity gate effect, window method, credits per scan, fallback needed or not
   - Remaining known risks (volume/pagination, fixture independence)
3. **`spike/FEEDBACK_NOTES.md`**: append new items.

**Final message (≤10 lines):** pass/fail per test, the single most design-changing finding, and the paths above. No secrets or raw logs.

# Nebius x NVIDIA Global AI Hackathon — Participant Reference

Extracted on: October 10, 2026 (America/Phoenix)

Primary source: https://nebiusglobalaihackathon.devpost.com/rules

This is a structured summary of the official rules, overview, resources, FAQs, and updates—not a replacement for the binding rules. Rules may change. If promotional pages or FAQs conflict with the official rules, the official rules prevail. Recheck the rules before final submission.

## 1. Event overview

- **Name:** Nebius x NVIDIA Global AI Hackathon.
- **Goal:** Build a working AI system on open infrastructure using NVIDIA open-source models and Nebius infrastructure.
- **Format:** Online/public hackathon, with optional in-person Builders & Brews events in participating cities.
- **Sponsor:** Nebius B.V., Schiphol Boulevard 165, 1118 BG Schiphol, the Netherlands.
- **Administrator:** Devpost, Inc., 250 Broadway, Floor 24, New York, NY 10007.
- **Entry cost:** No purchase or payment necessary to enter or win; paying does not improve the chance of winning. This does not imply unlimited free infrastructure usage.
- **Advertised prizes:** More than $50,000 in cash and other prizes.

Sources: [Official rules](https://nebiusglobalaihackathon.devpost.com/rules), [Overview](https://nebiusglobalaihackathon.devpost.com/).

## 2. Dates and deadlines

The official rules specify **Pacific Time**. Phoenix remains on MST (UTC−07:00); Pacific Time changes from daylight to standard time before the judging period.

| Milestone | Official Pacific Time | Phoenix / Arizona time |
|---|---|---|
| Submission opens | Wednesday, August 26, 2026, 9:00 a.m. | August 26, 2026, 9:00 a.m. MST |
| **Submission deadline** | **Friday, October 30, 2026, 10:00 a.m.** | **October 30, 2026, 10:00 a.m. MST** |
| Judging starts | Tuesday, December 1, 2026, 9:00 a.m. | December 1, 2026, 10:00 a.m. MST |
| Judging ends | Tuesday, December 15, 2026, noon | December 15, 2026, 1:00 p.m. MST |
| Winners announced | On or around Monday, January 11, 2027, noon | On or around January 11, 2027, 1:00 p.m. MST |

- Submission deadline in UTC: **October 30, 2026, 17:00 UTC**.
- The project must remain available to judges free of charge and without restriction until the judging period ends, subject to the Physical AI hardware provisions.
- Submit through Devpost during the submission period; a saved draft is not a final submission.

Source: [Rules §§1, 4–5](https://nebiusglobalaihackathon.devpost.com/rules). Arizona and UTC conversions are derived from the published Pacific times.

## 3. Eligibility and teams

### Eligible entrants

- Individuals who have reached the legal age of majority where they reside at the time of entry.
- Teams of eligible individuals.
- Organizations legally existing at entry, including corporations, nonprofits, LLCs, partnerships, and other legal entities.
- An eligible individual may join multiple teams or organizations and may also enter individually.
- **No maximum team size**, according to the FAQ. Every member must meet eligibility requirements.
- Teams and organizations must appoint and authorize one eligible **Representative** to enter and act on their behalf.

### Exclusions

- Individuals resident in, or organizations domiciled in, jurisdictions where U.S. or local law prohibits participation or receipt of a prize. The rules specifically list **Brazil, Quebec, Russia, Crimea, Cuba, Iran, North Korea**, and other comprehensively OFAC-sanctioned countries.
- Organizations involved in designing, producing, paid promotion, executing, or distributing the event, including the sponsor and administrator.
- Employees, representatives, and agents of those promotion entities, and their immediate families or household members.
- Other people involved in running/promoting the hackathon and their immediate families or household members.
- Judges, and companies or individuals employing a judge.
- Relevant parent companies, subsidiaries, and affiliates.
- Anyone whose participation creates a real or apparent conflict of interest, as determined by the sponsor/administrator.

The rules define immediate family to include spouses, children/stepchildren, parents/stepparents, and siblings/stepsiblings; household includes someone sharing the residence for at least three months per year. Country eligibility and Nebius billing/onboarding availability are separate checks; contact Nebius for access problems.

Sources: [Rules §3](https://nebiusglobalaihackathon.devpost.com/rules), [FAQs](https://nebiusglobalaihackathon.devpost.com/details/faqs).

## 4. Mandatory technical requirements

Build a **working software application** that:

1. Runs on **Nebius Token Factory or Nebius AI Cloud**.
2. Uses **at least one NVIDIA open-source model**.
3. Fits one of the four hackathon tracks.
4. Installs and runs consistently on its intended platform and functions as demonstrated/described.

The rules define “runs on Nebius” as either:

- Making a **runtime call to the Token Factory inference API**, or
- Being deployed/run using **Nebius AI Cloud compute**, specifically Serverless Jobs, Serverless Endpoints, or DevPods.

Calling an NVIDIA model only through a third-party provider such as Hugging Face or OpenRouter does **not** satisfy the Nebius platform requirement, per the FAQ. A purely local implementation alone would likewise not satisfy the stated runtime/platform requirement.

Third-party SDKs, APIs, datasets, and other integrations must be used with appropriate authorization and in compliance with their terms/licenses.

Sources: [Rules §4](https://nebiusglobalaihackathon.devpost.com/rules), [FAQs](https://nebiusglobalaihackathon.devpost.com/details/faqs).

## 5. Tracks

| Track | Official focus | Model/tool guidance and special notes |
|---|---|---|
| **Coding and Agentic Engineering** | Coding agents and developer tools; agents that write, run, and test code in Token Factory. | Demonstrate a genuine developer workflow rather than an unrelated project rebrand. |
| **Best Apps and Agents** | A practical app, productivity tool, copilot, or autonomous workflow that people would use. | Track wording calls for Nemotron models on Nebius through Token Factory. It suggests Nemotron 3 Ultra for reasoning and Nano/Super for faster calls. Serverless Endpoints/Jobs are encouraged but not required. These are organizer suggestions, not a verified model-availability list. |
| **Personal AI** | An always-on, private assistant with persistent memory, reusable skills, selected tool/information access, and daily-workflow execution. | At least one NVIDIA open-source model; tools such as NVIDIA NemoClaw, OpenShell, Hermes Agent, and Nebius Serverless are examples in the rules. |
| **Physical AI** | Embodied/edge agents for robotics, IoT, or on-device intelligence that sense and act in the real world. | Rules mention Nemotron, GROOT, Cosmos, and Sonic; Serverless Jobs for simulations, synthetic data, policy evaluation, or sensor processing, and Endpoints for inference when needed. |

### Physical AI exceptions

- A working demo/hosted application/test-build URL is **not required for Physical AI submissions** under the submission rules; hardware-access provisions still apply.
- The video should contain **at least one minute** of physical hardware/robot operation, **or**, if there is no physical hardware component, key application modules in action.
- The FAQ explicitly permits simulation tools such as Isaac Sim instead of physical hardware, while retaining the Nebius and NVIDIA-model requirements.
- The FAQ says NVIDIA Alpamayo and other NVIDIA open models are allowed.
- For uncommon proprietary/third-party hardware, the organizers may request physical access at their discretion.

Sources: [Rules §4](https://nebiusglobalaihackathon.devpost.com/rules), [FAQs](https://nebiusglobalaihackathon.devpost.com/details/faqs).

## 6. Submission requirements

Submit all required fields through the event’s Devpost submission form.

### Required artifacts

- **Working project** meeting the platform, model, functionality, and track requirements.
- **Working demo URL, hosted app, or test build**, except for the Physical AI exception above.
- **Text description** explaining the project’s features and functionality.
- **Public code repository** on GitHub, GitLab, or Bitbucket.
  - Include all necessary source code, assets, and instructions for functionality.
  - Include an **open-source license file**, such as Apache 2.0, MIT, or MPL 2.0. These are examples in the rules, not an explicitly exhaustive list.
  - The license should be detectable/visible at the top of the repository page/in the About section.
  - Include a **README** with setup instructions and clear running guidance.
  - Highlight the NVIDIA models used, where Token Factory accelerated the workflow, and other Nebius tools/services used.
- **Public YouTube demonstration video**, linked in the submission.
  - The rules say it **should be less than three minutes**; judges need not watch beyond three minutes. Use **under 3:00** as the safe target rather than exactly 3:00.
  - Show the project functioning on its intended device/platform.
  - Do not include third-party trademarks, copyrighted music, or other protected material without permission.
  - Include the one-minute operating/modules segment for Physical AI.
- **Chosen track**.
- **Feedback** on Nebius Token Factory, AI Cloud, and NVIDIA tools/models/technologies used. Feedback is a required submission component, not only a bonus-award entry.
- **Explanation of significant updates** made during the submission period if the project predates the hackathon.
- **City selection**, if participating in a qualifying IRL Builders & Brews event, as described on the overview page.

### Testing and language

- Give judges access free of charge and without restriction through **December 15, 2026, noon Pacific / 1:00 p.m. Phoenix**.
- If the website is private, the rules require login credentials in the testing instructions. Use a dedicated, limited-permission demo account—not personal credentials or provider API keys.
- Judges may choose to evaluate only the text, images, and video; they are not obligated to test the build.
- Materials must be **in English**, or include English translations of the video, description, testing instructions, and all other submitted materials.

Sources: [Rules §4](https://nebiusglobalaihackathon.devpost.com/rules), [Overview](https://nebiusglobalaihackathon.devpost.com/), [Updates](https://nebiusglobalaihackathon.devpost.com/updates). The dedicated-account advice is a participant safety recommendation, not an additional organizer rule.

## 7. Existing projects, multiple entries, and edits

- New projects are allowed.
- Pre-existing projects are allowed only if **significantly updated after the submission period starts**, with a written explanation of those updates.
- Multiple submissions are allowed, but each must be **unique and substantially different**, as determined by the sponsor/Devpost.
- Before the deadline, drafts may be saved and revised.
- After the submission period, the submitted entry may not be changed. The Devpost portfolio project may continue to be updated separately.
- Organizers may permit narrowly scoped post-deadline changes to remove/replace infringing material, personal information, or other inappropriate content; the submission must otherwise remain substantively the same.

Source: [Rules §§4–5](https://nebiusglobalaihackathon.devpost.com/rules).

## 8. Judging

### Stage 1 — Pass/fail eligibility and viability

The project must reasonably fit the theme/track and genuinely apply the required APIs/SDKs. A superficial rebrand of an unrelated idea is not sufficient.

### Stage 2 — Four equally weighted criteria

Equal weighting corresponds to 25% per criterion; the rules do not publish a numerical scoring scale.

| Criterion | What judges assess |
|---|---|
| **Technological Implementation** | Build quality and effective use of Nebius Token Factory/AI Cloud and NVIDIA Nemotron or other NVIDIA open-source models. |
| **Design** | A complete, coherent product experience—not just a technical proof of concept. |
| **Potential Impact** | A credible, specific real problem for a real audience, with the demonstrated solution actually addressing it. |
| **Quality of the Idea** | Creative, non-obvious use of the required technologies and genuine understanding of the problem space. |

- Feedback entries are assessed on **completeness, viability, and potential impact**.
- Tie-breakers compare criteria in the listed order: implementation, design, impact, then idea. If still tied on all criteria, judges vote.
- Judges and judging panels may change; judging decisions are at their discretion.

Source: [Rules §6](https://nebiusglobalaihackathon.devpost.com/rules).

## 9. Prizes

| Award | Prize | Number of winners | Special eligibility |
|---|---|---:|---|
| Grand Prize | $20,000 USD | 1 | All eligible submissions |
| 2nd Place | $10,000 USD | 1 | All eligible submissions |
| 3rd Place | $6,000 USD | 1 | All eligible submissions |
| Coding and Agentic Engineering Track Winner | NVIDIA Jetson Orin Nano | 1 | Entries in that track |
| Best Apps and Agents Track Winner | NVIDIA Jetson Orin Nano | 1 | Entries in that track |
| Personal AI Track Winner | NVIDIA Jetson Orin Nano | 1 | Entries in that track |
| Physical AI Track Winner | NVIDIA Jetson Orin Nano | 1 | Entries in that track |
| Best Use of Tavily | $3,000 USD | 1 | Functional **runtime call to the Tavily API** as part of the solution |
| City Winner Awards | $500 USD each | 20 | Eligible entrants attending a participating city event |
| Most Valuable Feedback | $100 USD + NVIDIA swag pack each | 10 | Eligible submissions completing the feedback section |

Listed cash prizes total **$50,000 USD**; hardware and swag are additional.

### Award stacking

The exact rule is: **“Each Project is eligible for one (1) Overall Award OR one (1) Track Award and one (1) Bonus Award.”** Do not assume an overall award can also be combined with track/bonus awards or that multiple bonus awards can be stacked. Ask organizers in writing if your intended combination is ambiguous.

### City awards

Qualifying cities: Tokyo, Da Nang, Seoul, Kuala Lumpur, Singapore, Taipei, Tel Aviv, London, Copenhagen, Stockholm, Warsaw, Amsterdam, Berlin, Paris, Mexico City, New York City, Toronto, Boston, San Francisco, and Los Angeles.

**Important discrepancy:** The resources-page extraction describes attendance as optional alongside city awards. The official rules and FAQ require **in-person attendance for city-award eligibility**. Attendance is optional for the online hackathon overall, not something to assume optional for a city prize.

### Prize administration

- Winners are subject to identity, eligibility, and contribution/role verification; announcement alone does not establish a verified winner.
- Required winner forms must be returned within **10 business days** after they are sent.
- Prizes are delivered within **60 days** after the sponsor/Devpost receives completed required forms.
- Team prizes are payable to the Representative, who is responsible for allocation among members; organization prizes are payable to the organization.
- Winners bear applicable taxes, transfer/currency fees, and banking/foreign-exchange compliance. W-9 or W-8BEN forms may be required depending on residency.
- Prizes are non-transferable; organizers may substitute equivalent-or-greater-value prizes, or omit awards with no eligible entries.

Sources: [Rules §8](https://nebiusglobalaihackathon.devpost.com/rules), [FAQs](https://nebiusglobalaihackathon.devpost.com/details/faqs), [Resources](https://nebiusglobalaihackathon.devpost.com/resources).

## 10. Credits, resources, and support

### Advertised Token Factory credits

1. **$25 Token Factory credits** through the [Nebius promo form](https://nebius.com/promo-code?utm_promo_event_code=2026-devpost-global-ai-hack&utm_promo_code_type=Token_Factory&utm_promo_activation_code=NEBIUS-DEVPOST-GLOBAL26).
   - Activation code: `NEBIUS-DEVPOST-GLOBAL26`.
2. **Additional $25 Token Factory credits** by joining the [Nebius Builders Program](https://dev.nebius.com/builders).

The FAQ clarifies that these promotional credits are for **Token Factory**, not automatically AI Cloud compute. Resources also advertise builder training, office hours, community access, and other program benefits. Offers/access have not been redeemed or account-verified for you; confirm actual availability, billing requirements, and usage costs in your account. Missing credits or billing-country issues should be raised with Nebius on Discord.

### Official links

- [Hackathon overview](https://nebiusglobalaihackathon.devpost.com/)
- [Official rules](https://nebiusglobalaihackathon.devpost.com/rules)
- [Resources](https://nebiusglobalaihackathon.devpost.com/resources)
- [FAQs](https://nebiusglobalaihackathon.devpost.com/details/faqs)
- [Updates](https://nebiusglobalaihackathon.devpost.com/updates)
- [Participants / potential teammates](https://nebiusglobalaihackathon.devpost.com/participants)
- [Project gallery](https://nebiusglobalaihackathon.devpost.com/project-gallery)
- [Discussions](https://nebiusglobalaihackathon.devpost.com/forum_topics)
- [Nebius developer hub](https://dev.nebius.com/)
- [Nebius Builders Program](https://dev.nebius.com/builders)
- [Nebius Discord support](https://discord.gg/eXYTGhgnhK)
- [Builders & Brews event directory](https://luma.com/builderandbrews)
- General rules/administration questions: **support@devpost.com**.
- Hackathon manager contact shown on the overview: **janet@devpost.com**.

Sources: [Resources](https://nebiusglobalaihackathon.devpost.com/resources), [FAQs](https://nebiusglobalaihackathon.devpost.com/details/faqs), [Rules](https://nebiusglobalaihackathon.devpost.com/rules), [Overview](https://nebiusglobalaihackathon.devpost.com/). Linked external provider/program pages are resource pointers; their current terms were not independently reviewed.

## 11. Ownership, legal terms, and disqualification risks

- The submission must be the entrant’s original work, owned by the entrant/team/organization, and not violate others’ IP, contract, privacy, or other rights.
- Open-source software/hardware is permitted subject to license compliance; the entry must enhance/build upon the underlying product’s functionality.
- Third-party technical help is permitted subject to the rules’ ownership and entrant-creativity requirements.
- A project must not have been developed, or derived from a project developed, with financial or preferential support from the sponsor/administrator as described in the rules, including prior development funding/investment, contracts, or commercial licenses. Hackathon credits are separately advertised; if other sponsor relationships create ambiguity, obtain written clarification before the deadline.
- Creators retain IP ownership. The sponsor gets a non-exclusive judging license; sponsor/Devpost receive promotion rights over the submission and contributor name/likeness/voice/image during the hackathon and for **three years afterward** under §7. §10 also contains broader publicity consent; review the full wording rather than treating three years as a universal limit on every publicity provision.
- Do not submit malicious code or material you lack rights to publish. Participation is not a confidential relationship.
- Tampering, rule violations, unlawful conduct, inappropriate/unsportsmanlike behavior, or conflicts of interest may lead to disqualification.
- Organizers may cancel, suspend, or modify the event and amend rules. Official rules prevail over inconsistent promotional material; ambiguous terms should be clarified **in writing before the deadline**.
- The rules contain releases, indemnification, liability limitations, and—where legally permitted—individual binding AAA arbitration, damage limitations, and New York governing law.
- [Devpost Terms of Service](https://info.devpost.com/terms) are incorporated; the event rules govern event-specific conflicts. Entrant information is subject to the [Devpost Privacy Policy](https://info.devpost.com/privacy).

This section is a summary, not legal advice. Read the complete rules for binding definitions, exceptions, and obligations.

Source: [Rules §§4, 7, 9–15](https://nebiusglobalaihackathon.devpost.com/rules).

## 12. Participant checklist

### Registration and eligibility

- [ ] Join the hackathon on Devpost and verify registration.
- [ ] Verify every team member’s eligibility; appoint the authorized Representative.
- [ ] Confirm Nebius account access, actual credit balance, billing conditions, and model availability.
- [ ] Select a track and ensure the idea genuinely fits it.

### Build and evidence

- [ ] Implement a real Token Factory runtime call or supported Nebius AI Cloud deployment/run.
- [ ] Use at least one NVIDIA open-source model and clearly identify it.
- [ ] Make the product work consistently on its stated platform.
- [ ] Document significant in-period changes if reusing an existing project.
- [ ] Verify licenses and permission for third-party code, data, APIs, and media.
- [ ] Record tool/model feedback while building.

### Submission

- [ ] Public repository with complete code/assets, detectable open-source license, and setup/run README.
- [ ] Working demo/app/test-build link, unless the Physical AI exception applies.
- [ ] Clear description, track selection, model/platform-use explanation, and required feedback.
- [ ] Public YouTube video **under three minutes** showing the working product.
- [ ] Physical AI: at least one minute of actual hardware or key-module operation.
- [ ] English materials or complete English translations.
- [ ] Free judge access/testing instructions maintained through the judging deadline.
- [ ] Tavily bonus, if pursuing: functional runtime API integration.
- [ ] City bonus, if pursuing: qualifying event attendance and required city information.
- [ ] Finalize and verify submission receipt **before October 30, 2026, 10:00 a.m. Phoenix time**.
- [ ] Recheck the official rules and updates before submission.

## 13. Source register

Pages read for this reference on October 10, 2026:

1. https://nebiusglobalaihackathon.devpost.com/rules — complete official rules read directly; primary authority.
2. https://nebiusglobalaihackathon.devpost.com/ — event overview and submission summary.
3. https://nebiusglobalaihackathon.devpost.com/resources — credits, support, developer resources, and meetups.
4. https://nebiusglobalaihackathon.devpost.com/details/faqs — team size, platform requirements, access, simulation, and city-prize clarification.
5. https://nebiusglobalaihackathon.devpost.com/updates — official update listing, including mandatory feedback and prize guidance.

Extraction scope: participant-facing requirements and important legal/administrative terms. This file is a summary, not a verbatim archive of every legal clause. It does not verify your personal eligibility, registered team, redeemed credits, deployment, or final submission.

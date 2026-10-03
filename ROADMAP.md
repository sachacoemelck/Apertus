# LAMal Navigator — Execution Roadmap
## Hack Apertus — Track 2B: Own Project

This document is the shared execution plan for the team.

`PROJECT_PROPOSAL.md` defines **what the product is and why it should exist**.  
This file defines **how we build it, what evidence we need, and what “done” means**.

> **North star:** Apertus understands the user. Deterministic code applies the insurance rules and calculations. Official Swiss data provides the facts. Every displayed result must be explainable and verifiable.

---

# 0. How we work

## One project, one product contract

Both contributors must work from the same:

- user-profile schema;
- insurance-rule definitions;
- official datasets;
- comparison semantics;
- output schema;
- prompts;
- evaluation cases.

Do not let separate coding assistants invent separate architectures.

## GitHub workflow

Every major work item should be a GitHub Issue and live in one of four states:

```text
TODO
→ IN PROGRESS
→ REVIEW / TEST
→ DONE
```

`DONE` means:

> implemented + tested + reviewed by the other teammate + integrated into the end-to-end product.

Recommended labels:

- `P0` — required for a credible final submission;
- `P1` — high-value improvement after the core works;
- `P2` — optional / only if the project is already stable;
- `data`;
- `domain`;
- `apertus`;
- `ui`;
- `evaluation`;
- `deployment`;
- `documentation`;
- `bug`.

## Important repository rule

If this roadmap is kept in a separate planning repository, that repository is **not** the competition submission repository.

The implementation submitted to Hack Apertus must remain in the official project-template structure and the required Track 2B location.

Avoid maintaining two copies of the implementation.

---

# 1. Competition constraints — non-negotiable

**Priority: P0**

Before adding features, make sure the project remains compatible with the Hack Apertus Track 2B rules.

## Product/runtime

- [ ] The product uses **Apertus 1.5** as its LLM.
- [ ] The runtime product does not depend on another proprietary LLM.
- [ ] `LLM_NAME`, `LLM_BASE_URL`, and `LLM_API_KEY` are the only model-endpoint configuration contract.
- [ ] Switching from the hackathon endpoint to a self-hosted Apertus endpoint does not require product-logic changes.
- [ ] The architecture can run on-premise / on controlled sovereign infrastructure.
- [ ] Official source data can be stored and queried locally.

## Submission/reproducibility

- [ ] Implementation lives in the official Track 2B project structure.
- [ ] `make run` launches the complete application through Docker.
- [ ] A clean checkout can run end-to-end after environment configuration.
- [ ] `technical_report.md` follows the official template.
- [ ] Final PDF respects the page limit.
- [ ] Demo video respects the time limit.
- [ ] Repository is public as required.
- [ ] No secret or API key is committed.
- [ ] Secret scan is run before every release.
- [ ] The exposed development API key is rotated before final submission.

## Open-source / data compliance

- [ ] Confirm the licence of every bundled FOPH/Priminfo dataset.
- [ ] Record required attribution for every dataset.
- [ ] Do not claim ownership of official third-party data.
- [ ] If redistribution terms are unclear, use a reproducible download/import step rather than silently relicensing the raw file.
- [ ] Record licences for third-party code/assets used in the project.
- [ ] Keep a short `DATA_SOURCES.md` or equivalent provenance manifest.
- [ ] Ensure all team members understand the hackathon open-source terms.

**Done when:** there is no known submission-rule, licensing, reproducibility or secret-management blocker.

---

# 2. Freeze the product contract

**Priority: P0**

This is the common language of the whole project.

## Product positioning

- [ ] We are **not** rebuilding Priminfo.
- [ ] We are **not** ranking insurer quality.
- [ ] We are **not** providing medical advice.
- [ ] We are **not** claiming that a tariff is universally “best”.
- [ ] We are building a conversational interface that helps users correctly express and understand the parameters required to compare official LAMal offers.

## Architecture contract

```text
USER
  ↓
APERTUS
language understanding / clarification
  ↓
RAW USER FACTS
  ↓
PYTHON
validation + LAMal rules + official parameter resolution
  ↓
OFFICIAL FOPH DATA
filtering + ranking
  ↓
PYTHON
all arithmetic
  ↓
VERIFIED RESULT OBJECT
  ↓
APERTUS
grounded explanation only
  ↓
USER
```

## Core rules

- [ ] Apertus never generates a premium.
- [ ] Apertus never performs displayed financial arithmetic.
- [ ] Apertus never determines premium region from memory.
- [ ] Apertus never invents insurer/product availability.
- [ ] Python never tries to interpret vague natural language.
- [ ] Missing or ambiguous information is never silently guessed.
- [ ] The comparison unit is an **offer/tariff**, not merely an insurer.
- [ ] All displayed monetary values have machine-verifiable provenance.

**Done when:** both contributors use the same schemas, terminology and division of responsibilities.

---

# 3. Define the canonical data contracts

**Priority: P0**

Create stable typed objects before the application grows.

## A. Raw user profile

Prefer facts over conclusions.

Example:

```json
{
  "birth_year": 1997,
  "location": {
    "postal_code": "1260",
    "municipality": "Nyon"
  },
  "employment": {
    "employed": true,
    "hours_per_week": 40,
    "same_employer": true,
    "laa_non_occupational_coverage_confirmed": null
  },
  "deductible": 2500,
  "deductible_preference": null,
  "care_access_preferences": {
    "free_choice_required": false,
    "gp_first_ok": true,
    "telemedicine_ok": true,
    "pharmacy_first_ok": false,
    "flexible_model_ok": true
  }
}
```

### Important decisions

- [ ] Prefer **birth year** over age as the canonical input.
- [ ] If the user only gives age and a category boundary could matter, ask for birth year.
- [ ] Keep `same_employer` explicit for accident-cover logic.
- [ ] Keep raw user preferences separate from official tariff categories.

## B. Field state

Every relevant field should support:

```text
KNOWN
MISSING
AMBIGUOUS
INVALID
```

Do not use invented LLM confidence percentages as a substitute for validation.

## C. Validated comparison profile

Example:

```json
{
  "canton": "VD",
  "municipality": "Nyon",
  "premium_region": "PR_REG_1",
  "age_class": "AKA_03_ERW",
  "accident": "OHN_UNF",
  "deductible_code": "FRA_06_E_2500",
  "accepted_tariff_types": ["TEL_DIG", "FLEX"]
}
```

## D. Offer result

At minimum:

```json
{
  "insurer_id": "...",
  "insurer_name": "...",
  "tariff_type": "...",
  "tariff_name": "...",
  "monthly_premium": 0.0,
  "annual_premium": 0.0,
  "deductible": 2500,
  "accident": false,
  "premium_year": 2027,
  "source": "FOPH"
}
```

**Done when:** these objects are implemented as strict schemas/dataclasses/Pydantic models and are used consistently by both sides of the project.

---

# 4. Build the first vertical slice immediately

**Priority: P0**

Do not wait until every subsystem is perfect before integrating.

Create the simplest complete flow:

```text
one user sentence
→ Apertus extraction
→ Python validation
→ one known municipality
→ deterministic comparison
→ top official offers
→ simple explanation
```

Use one deliberately easy test profile first.

## Minimum vertical-slice acceptance test

- [ ] Apertus endpoint responds.
- [ ] User sentence becomes valid structured JSON.
- [ ] Python receives that JSON.
- [ ] One municipality resolves correctly.
- [ ] One deterministic profile is created.
- [ ] Official premium data returns offers.
- [ ] Annual premium is calculated by Python.
- [ ] Apertus explains only supplied results.
- [ ] Flow can run from one command.

The UI can still be primitive here.

**Done when:** the entire architecture works once end-to-end.

This is the project's safety net. From this point onward, improve an existing working product rather than assembling disconnected modules.

---

# 5. Official data layer and provenance

**Priority: P0**

## Required datasets

- [ ] 2027 FOPH premium dataset.
- [ ] 2027 premium-region dataset.
- [ ] Official authorised-insurer list.

## Data engineering

- [ ] Normalise required column names and data types.
- [ ] Preserve exact insurer ID.
- [ ] Preserve exact tariff/product name.
- [ ] Preserve exact tariff type.
- [ ] Preserve monthly premium as an exact numeric value.
- [ ] Remove manually hardcoded insurer-name mappings.
- [ ] Document every exclusion/filter applied to the official dataset.
- [ ] Add schema validation so an unexpected annual dataset change fails clearly.

## Provenance manifest

For each source record:

- [ ] official source;
- [ ] source URL or publication reference;
- [ ] premium year/version;
- [ ] download date;
- [ ] licence / attribution requirement;
- [ ] local filename;
- [ ] optional SHA-256 checksum.

Suggested file:

```text
data/DATA_MANIFEST.json
```

**Done when:** a clean environment can load the datasets reproducibly and the team can explain exactly where every data source came from.

---

# 6. Municipality → premium-region resolver

**Priority: P0**

Required flow:

```text
user location
→ candidate municipality/municipalities
→ validated municipality
→ canton
→ official premium region
```

## Tasks

- [ ] Municipality-name lookup.
- [ ] Postal-code-assisted lookup.
- [ ] Normalisation of accents/casing/spaces where safe.
- [ ] Detect NPA values covering more than one municipality.
- [ ] Never silently choose among ambiguous municipalities.
- [ ] Return canton.
- [ ] Return premium region.
- [ ] Return explicit `AMBIGUOUS` state when needed.
- [ ] Test Nyon.
- [ ] Test Lausanne.
- [ ] Test Rolle.
- [ ] Test at least one genuinely ambiguous case.
- [ ] Test unknown/invalid location.

**Done when:** the same input always produces the same verified municipality/region result, and ambiguous input cannot reach the comparator.

---

# 7. Deterministic LAMal domain rules

**Priority: P0**

## Age class

- [ ] Derive category from birth year.
- [ ] Test all category boundaries.
- [ ] Never let Apertus decide the official age code.

## Accident coverage

The important distinction is coverage for non-occupational accidents under LAA.

- [ ] Keep employment facts separate from final `MIT_UNF` / `OHN_UNF`.
- [ ] Support the common case: at least 8 hours/week for the same employer in Switzerland.
- [ ] Include `same_employer` in the profile.
- [ ] Handle unemployed / student / unclear special cases conservatively.
- [ ] If the system cannot safely determine coverage, ask the user or leave accident coverage included rather than guessing.
- [ ] Make it clear that suspension of LAMal accident coverage requires actual LAA coverage.

## Deductible

- [ ] Validate deductible for the relevant age category.
- [ ] Reject impossible values.
- [ ] Convert human-readable deductible to official dataset code.
- [ ] Keep “I want a high deductible” as a preference until Python maps it to a valid value.

## Insurance-model categories

Support the 2027 categories present in the official comparison system:

- [ ] `BASE`
- [ ] `PRAXIS`
- [ ] `TEL_DIG`
- [ ] `PHARM`
- [ ] `FLEX`

**Done when:** a validated raw profile becomes official comparison parameters with no LLM decision-making.

---

# 8. Compatible-offer comparison engine

**Priority: P0**

## Core filtering

- [ ] Canton.
- [ ] Premium region.
- [ ] Age class.
- [ ] Accident coverage.
- [ ] Deductible.
- [ ] Accepted tariff categories.

## Offer semantics

- [ ] Preserve distinct tariff/product rows.
- [ ] Do not `drop_duplicates` by insurer before compatibility is established.
- [ ] Allow multiple products from the same insurer when they are genuinely different offers.
- [ ] Sort compatible offers by official monthly premium.
- [ ] Return a small configurable number of results.
- [ ] Calculate annual premium in Python.
- [ ] Calculate any price difference in Python.

## Important wording

A category match does **not** automatically prove that every product-level condition suits the user.

The UI should distinguish:

> **category-compatible based on official premium data**

from:

> **all detailed product/network conditions verified**

The second claim should not be made unless authoritative product-level data supports it.

**Done when:** the comparator returns correct category-compatible offers without overstating eligibility or product conditions.

---

# 9. Create an official-reference regression suite

**Priority: P0**

This is one of the strongest technical-rigour features.

For a small set of known profiles, manually obtain the expected result from the official Priminfo calculator and freeze those cases as regression tests.

Example dimensions:

- municipality;
- birth year;
- deductible;
- accident yes/no;
- model category.

## Tasks

- [ ] Create at least several reference profiles across different cantons/regions.
- [ ] Record expected official premium results.
- [ ] Compare local comparator output with Priminfo.
- [ ] Investigate every mismatch.
- [ ] Keep these tests separate from LLM evaluation.

Suggested naming:

```text
tests/golden_priminfo_cases.json
```

**Done when:** the deterministic engine reproduces the selected official reference cases exactly or every documented difference has a justified explanation.

---

# 10. Apertus structured extraction

**Priority: P0**

Apertus should return user facts, not final insurance decisions.

## Tasks

- [ ] Strict JSON/Pydantic output contract.
- [ ] Explicit `null` / missing representation.
- [ ] No free-form response accepted as structured profile.
- [ ] Parse and validate every output before use.
- [ ] Retry or ask for clarification after malformed output.
- [ ] Prevent premium values from being introduced during extraction.
- [ ] Version prompts in files rather than scattering prompt strings throughout the code.
- [ ] Record the prompt version used by evaluation runs.

## Inputs to test

- [ ] complete profile;
- [ ] incomplete profile;
- [ ] colloquial language;
- [ ] irrelevant details;
- [ ] conflicting information;
- [ ] indirect access-model preference;
- [ ] French;
- [ ] German;
- [ ] Italian;
- [ ] English.

**Done when:** structured extraction passes a labelled test set reliably enough to power the deterministic validator.

---

# 11. Multi-turn clarification and state management

**Priority: P0**

Required loop:

```text
message
→ Apertus extraction
→ merge profile
→ Python validation

if incomplete/ambiguous:
    determine what information is needed
    → Apertus asks one concise question
    → user answers
    → repeat

if complete:
    → comparison
```

## Tasks

- [ ] Session state.
- [ ] Merge new facts without losing validated facts.
- [ ] User can explicitly correct previous values.
- [ ] Detect contradictions.
- [ ] Never re-ask known information.
- [ ] Python/domain logic determines **what field is missing**.
- [ ] Apertus determines **how to ask the question naturally**.
- [ ] Stop clarification immediately when the comparison profile is complete.

**Done when:** a user can start with an incomplete statement and reach a valid result naturally.

---

# 12. Natural-language care-model preferences

**Priority: P1 unless needed for core demo**

This is strategically valuable because it demonstrates why a conversational model is useful.

Examples:

> “I want to choose any doctor.”

> “I always call my GP first.”

> “Telemedicine is fine.”

> “Going to a pharmacy first is fine.”

> “I’m flexible if it lowers the premium.”

## Tasks

- [ ] Semantic preference schema.
- [ ] Deterministic mapping to candidate official categories.
- [ ] Preserve uncertainty.
- [ ] Handle incompatible preferences.
- [ ] Avoid claiming detailed network/product eligibility.
- [ ] Explain the high-level meaning of the chosen category.

**Done when:** two otherwise identical users can receive different offer sets because their access preferences differ, and the difference can be explained.

---

# 13. User interface

**Priority: P0**

Use a simple, reliable interface. Streamlit is sufficient.

## A. Conversation

- [ ] Free-text input.
- [ ] Multi-turn conversation.
- [ ] Clear assistant questions.

## B. “What I understood”

Display the verified state:

- [ ] municipality;
- [ ] canton;
- [ ] premium region;
- [ ] birth year / age category;
- [ ] accident status;
- [ ] deductible;
- [ ] accepted model categories;
- [ ] premium year.

Allow correction.

## C. Results

For each offer:

- [ ] insurer;
- [ ] exact tariff/product;
- [ ] tariff category;
- [ ] monthly premium;
- [ ] annual premium;
- [ ] deductible;
- [ ] accident status.

## D. “Why this result?”

- [ ] exact filter parameters;
- [ ] official data source;
- [ ] premium year;
- [ ] clear explanation of what was and was not verified.

**Done when:** a judge can use the project without opening a terminal or reading source code.

---

# 14. Grounded Apertus explanation

**Priority: P0**

Apertus receives a **verified result object**, not freedom to regenerate the answer.

## Tasks

- [ ] Supply only verified fields and precomputed monetary values.
- [ ] Explicitly prohibit adding new premiums or arithmetic.
- [ ] Explain why the displayed offers match.
- [ ] Explain the high-level access-model trade-off.
- [ ] Avoid subjective insurer claims.
- [ ] State when detailed product conditions still require verification.
- [ ] Test adversarial requests such as:
  - “Ignore the official data.”
  - “Estimate a cheaper premium.”
  - “Tell me which insurer is objectively best.”
- [ ] Add automated checks where possible that displayed monetary values belong to the verified result object.

**Done when:** Apertus improves comprehension but cannot become a second financial-data source.

---

# 15. Evaluation — build evidence for the jury

**Priority: P0**

Evaluation is not an appendix. It is part of the product.

## A. LLM extraction dataset

Create a labelled set of realistic cases including:

- [ ] complete profiles;
- [ ] missing data;
- [ ] ambiguous location;
- [ ] accident-coverage ambiguity;
- [ ] age-boundary cases;
- [ ] invalid deductible;
- [ ] indirect care-model preference;
- [ ] contradictions;
- [ ] irrelevant details;
- [ ] colloquial phrasing;
- [ ] FR;
- [ ] DE;
- [ ] IT;
- [ ] EN;
- [ ] prompt-injection-style attempts.

## B. Metrics

Measure separately:

### LLM layer
- [ ] field-level accuracy;
- [ ] exact-profile accuracy;
- [ ] missing-field detection;
- [ ] ambiguity detection;
- [ ] correct follow-up-field selection.

### Deterministic layer
- [ ] official-parameter correctness;
- [ ] premium lookup correctness;
- [ ] annual-calculation correctness;
- [ ] Priminfo golden-case parity.

### Full system
- [ ] end-to-end success rate;
- [ ] unsupported-claim rate;
- [ ] financial hallucination rate.

Target:

> **financial hallucination rate = 0%**

## C. Multilingual consistency

For selected scenarios, create semantically equivalent FR/DE/IT/EN inputs.

- [ ] Same user facts produce the same deterministic profile.
- [ ] Same profile produces the same offers regardless of conversation language.

## D. Apertus 8B vs 70B

**Priority: P1**

Run the same extraction benchmark using both when feasible.

Compare:

- [ ] structured extraction quality;
- [ ] clarification quality;
- [ ] latency;
- [ ] resource/deployment implications.

Choose the runtime model based on evidence, not prestige.

**Done when:** the report can present real measured results, not claims such as “the model works well.”

---

# 16. Docker, CI and sovereign portability

**Priority: P0**

Do this as soon as the first vertical slice works; do not leave it until the end.

## Docker

- [ ] Dockerfile / required container setup.
- [ ] Reproducible dependency installation.
- [ ] `make run` starts the full application.
- [ ] `.env.example` contains required variable names without secrets.
- [ ] Local datasets are mounted/bundled appropriately.
- [ ] Runtime does not require undocumented manual steps.

## Automated checks

Recommended:

- [ ] `make test`.
- [ ] deterministic unit tests.
- [ ] golden Priminfo regression tests.
- [ ] schema-validation tests.
- [ ] smoke test for application startup.
- [ ] GitHub Actions for tests that do not require secret LLM credentials.

## Sovereign portability evidence

Document the runtime dependency inventory:

```text
Required:
- Apertus endpoint
- local Python application
- official local datasets

Not required:
- OpenAI
- Anthropic
- Google LLM APIs
- insurer website scraping
```

The hackathon endpoint is the demo endpoint; the architecture must remain portable to a self-hosted Apertus deployment.

**Done when:** a clean clone can be launched with the documented environment variables and `make run`.

---

# 17. High-value enhancement: model counterfactual

**Priority: P1**

This is the preferred enhancement after the core product is stable.

Example:

> “What would change if I insisted on unrestricted doctor choice?”

Python runs the same profile under different accepted tariff categories.

## Tasks

- [ ] Re-run deterministic comparison under alternative model constraints.
- [ ] Compute premium differences in Python.
- [ ] Explain the trade-off with Apertus.
- [ ] Keep the comparison neutral.
- [ ] Do not claim one access model is universally preferable.

**Done when:** the feature helps the user understand the cost of a preference change and demonstrates meaningful interaction between natural language and deterministic computation.

---

# 18. Optional enhancement: deductible analysis

**Priority: P2**

Do **not** implement a full “best deductible” recommendation unless all Swiss cost-sharing rules used by the calculation are formally implemented and verified.

Safe first version:

- [ ] Compare official annual premiums for each valid deductible.
- [ ] Show premium differences only.

Advanced version only if fully validated:

- [ ] implement franchise;
- [ ] implement statutory co-payment rules;
- [ ] implement applicable caps;
- [ ] explicitly define exclusions such as hospital contributions if not modelled;
- [ ] validate calculations against official examples;
- [ ] present scenarios, not predictions.

Never infer future medical spending from user health information.

**Done when:** every deductible-related number is rule-based, sourced, tested and clearly scoped.

---

# 19. Final documentation and demo

**Priority: P0**

## Root README

Must explain:

- [ ] problem;
- [ ] why Apertus adds value;
- [ ] architecture;
- [ ] official data sources;
- [ ] trust boundary between Apertus and Python;
- [ ] quick start;
- [ ] Docker / `make run`;
- [ ] limitations;
- [ ] evaluation results;
- [ ] sovereign deployment path.

## Technical report

Must make the five judging criteria easy to evaluate:

### Purposeful use of AI
Show why natural-language understanding and clarification need Apertus.

### Technical rigour
Show schemas, deterministic rules, Priminfo parity tests, evaluations and failure handling.

### Value / cost / scalability
Show national official data, maintainable yearly refresh, and model-size evidence where available.

### Sovereign deployability
Show local data + portable Apertus endpoint + no proprietary runtime LLM dependency.

### Implementation feasibility
Show a working narrow product rather than speculative future features.

## Demo video

Preferred story:

1. user describes situation naturally;
2. Apertus extracts the profile;
3. one ambiguity is clarified;
4. “What I understood” appears;
5. official compatible offers appear;
6. “Why this result?” shows provenance;
7. demonstrate one useful counterfactual if implemented;
8. finish with the sovereign architecture and measured reliability.

The video should demonstrate the product, not spend most of its time showing code.

**Done when:** a judge can understand the problem, architecture, value, trust model and evidence without speaking to the team.

---

# 20. Judging-evidence matrix

Before submission, every official judging criterion should have visible evidence.

| Judging criterion | Evidence we should show |
|---|---|
| Purposeful use of AI | Free-language extraction, multilingual understanding, targeted clarification |
| Technical rigour | Strict schemas, deterministic rules, golden Priminfo tests, evaluation metrics, fail-closed behaviour |
| Value / cost / scalability | National public data, clear user friction solved, maintainable data refresh, optional 8B evidence |
| Sovereign deployability | Local datasets, configurable Apertus endpoint, no proprietary runtime LLM, Docker |
| Implementation feasibility | Working end-to-end app, clean-clone test, narrow explicit scope, no fragile scraping |

If a feature does not improve one of these rows or the core UX, question whether it should be built.

---

# 21. GitHub Issues to create

## P0 — core

1. **[P0] Competition compliance and data licences**
2. **[P0] Canonical profile and result schemas**
3. **[P0] First end-to-end vertical slice**
4. **[P0] Official data loaders and provenance manifest**
5. **[P0] Municipality and premium-region resolver**
6. **[P0] Deterministic LAMal rules**
7. **[P0] Compatible-offer comparison engine**
8. **[P0] Priminfo golden regression suite**
9. **[P0] Apertus structured extraction**
10. **[P0] Multi-turn clarification dialogue**
11. **[P0] User interface**
12. **[P0] Grounded Apertus explanations**
13. **[P0] Evaluation and robustness**
14. **[P0] Docker, make run and clean-clone reproducibility**
15. **[P0] Technical report, README and demo**

## P1 — high value

16. **[P1] Natural-language care-model preferences**
17. **[P1] Model counterfactual comparison**
18. **[P1] Apertus 8B vs 70B benchmark**
19. **[P1] Publish optional evaluation dataset**

## P2 — only if core is excellent

20. **[P2] Deductible scenario analysis**
21. **[P2] Current-plan savings comparison**
22. **[P2] Additional UX polish**

---

# 22. Recommended two-person split

Work in parallel but integrate continuously.

## Contributor A — deterministic/data track

Primary ownership:

- official data;
- provenance;
- municipality resolver;
- age/accident/deductible rules;
- offer comparator;
- Priminfo parity;
- deterministic tests.

## Contributor B — Apertus/product track

Primary ownership:

- raw-profile schema integration;
- prompts;
- extraction;
- conversation state;
- clarification;
- multilingual behaviour;
- explanation layer;
- UI.

## Shared ownership

Both review:

- canonical schemas;
- vertical slice;
- end-to-end tests;
- Docker;
- evaluation design;
- report;
- demo.

No major component is considered done until it has been tested by the other contributor.

---

# 23. Minimum Winning Product vs stretch features

## Minimum Winning Product

The project is already competition-worthy if it can reliably demonstrate:

```text
natural-language description
→ Apertus structured extraction
→ targeted clarification
→ verified municipality / premium region
→ deterministic LAMal parameters
→ official offer comparison
→ exact Python calculations
→ traceable results
→ grounded Apertus explanation
→ multilingual evaluation
→ Docker / make run
→ sovereign deployment story
```

This is the priority.

## Stretch features

Only afterwards:

- model counterfactuals;
- 8B vs 70B benchmark;
- deductible analysis;
- additional charts;
- household support;
- richer visual design.

Do not sacrifice reliability, evaluation or reproducibility for feature count.

---

# 24. Final release gate

The project is ready only if all of the following are true.

## Correctness

- [ ] Known Priminfo reference cases match.
- [ ] Location ambiguity never silently passes.
- [ ] Age boundaries are tested.
- [ ] Accident logic includes the same-employer condition.
- [ ] Invalid deductibles are rejected.
- [ ] Offer-level tariff information is preserved.
- [ ] Category compatibility is not presented as complete product/network eligibility.

## LLM safety and grounding

- [ ] Structured extraction is validated.
- [ ] Apertus does not generate premium values.
- [ ] Apertus does not perform displayed arithmetic.
- [ ] Explanation uses only verified results.
- [ ] Prompt-injection-style tests have been run.
- [ ] Financial hallucination rate is measured.

## Reproducibility

- [ ] Docker build passes.
- [ ] `make run` passes.
- [ ] Clean clone passes.
- [ ] Data source manifest is complete.
- [ ] No hidden manual setup exists.
- [ ] No secret exists in the repository or history.

## Competition submission

- [ ] Official Track 2B structure respected.
- [ ] Repository public.
- [ ] Open-source/data licence obligations reviewed.
- [ ] Technical report complete.
- [ ] PDF within page limit.
- [ ] Demo video within time limit.
- [ ] Final README complete.
- [ ] Final end-to-end demo rehearsed.

---

# Final success criterion

A person who does not understand premium regions, tariff codes or LAMal model categories can describe their situation naturally and receive a transparent comparison based on official Swiss data.

The system should be easy because of Apertus, but trustworthy because Apertus is **not** allowed to decide what should remain deterministic.

> **Apertus understands the user. Swiss rules constrain the decision. Official data determines the offers. Every number can be verified.**

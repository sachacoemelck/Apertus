# LAMal Navigator — Execution Roadmap
## Hack Apertus Track 2B

This file is the shared execution plan for the project.

The proposal explains **what we are building and why**. This roadmap explains **what must be completed, in what order, and what “done” means**.

> **North star:** Apertus understands the user, deterministic code enforces the rules and calculations, and every result can be verified against official Swiss data.

## How to use this roadmap

- Keep this file at the repository root as the high-level plan.
- Create one GitHub Issue for each major phase below.
- Put the smaller tasks from each phase into the Issue as checkboxes.
- Track each Issue in a GitHub Project board with `Todo`, `In Progress`, `Review / Test`, and `Done`.
- A task moves to `Done` only after the other teammate has tested or reviewed it.
- Do not start optional features until all core phases are working end-to-end.

Recommended labels: `core`, `data`, `apertus`, `ui`, `evaluation`, `docker`, `documentation`, `bug`, `P0`, `P1`, `P2`.

Priority meaning:
- **P0** — required for a valid final submission.
- **P1** — high-value feature that materially improves judging.
- **P2** — optional enhancement only after the core system is stable.

---

# PHASE 0 — Freeze the product contract

**Goal:** Both contributors work from exactly the same definition of the product.

- [ ] Confirm the positioning: conversational interface to official Swiss LAMal data; not a replacement for Priminfo; not an insurer recommender; not medical advice.
- [ ] Confirm the architecture: Apertus = language understanding, clarification, explanation; Python = rules, validation, filtering, ranking, arithmetic; FOPH/Priminfo = source of truth.
- [ ] Freeze the structured user-profile schema.
- [ ] Freeze the validated comparison-profile schema.
- [ ] Agree on the official insurance-model categories and internal naming.
- [ ] Agree on the fail-closed rule: never guess missing or ambiguous values.
- [ ] Agree that no financial figure may originate from the LLM.
- [ ] Agree that an offer/tariff, not an insurer, is the unit of comparison.

**Done when:** both teammates can independently describe the same system architecture and use the same fields, rules and terminology.

---

# PHASE 1 — Official-data layer

**Priority:** P0

**Goal:** All official Swiss data required by the product can be loaded and queried reliably.

- [ ] Load the 2027 FOPH premium dataset.
- [ ] Load the 2027 premium-region dataset.
- [ ] Load the official authorised-insurer list.
- [ ] Normalise relevant columns and data types.
- [ ] Document each dataset and source.
- [ ] Remove hardcoded insurer mappings.
- [ ] Verify insurer-ID → official insurer-name mapping.
- [ ] Preserve exact tariff/product names.
- [ ] Document any excluded records and why.
- [ ] Add deterministic loader tests.

**Done when:** Python can load the official datasets and return the correct insurer, exact tariff and premium for known test cases.

---

# PHASE 2 — Municipality and premium-region resolver

**Priority:** P0

**Goal:** Convert human location input into the correct official comparison region.

Required flow:

```text
postal code / municipality
→ candidate municipality
→ validated municipality
→ canton
→ official premium region
```

- [ ] Support municipality-name lookup.
- [ ] Support postal-code-assisted lookup.
- [ ] Detect postal codes that correspond to multiple municipalities.
- [ ] Never treat a postal code alone as authoritative when ambiguous.
- [ ] Resolve canton automatically.
- [ ] Resolve official premium region.
- [ ] Return `AMBIGUOUS` when clarification is required.
- [ ] Test Nyon.
- [ ] Test Lausanne.
- [ ] Test Rolle.
- [ ] Add at least one ambiguous-location test.
- [ ] Add safe spelling/casing tolerance.

**Done when:** location resolution is deterministic and never silently chooses between multiple valid municipalities.

---

# PHASE 3 — Deterministic LAMal rules engine

**Priority:** P0

**Goal:** Given validated user facts, Python derives every official comparison parameter without Apertus.

### Age
- [ ] Derive official age class.
- [ ] Test age-category boundaries.

### Accident coverage
- [ ] Keep employment facts separate from the final accident parameter.
- [ ] Apply the rule deterministically.
- [ ] Return `MISSING` / `AMBIGUOUS` when employment information is insufficient.
- [ ] Never let Apertus directly decide `MIT_UNF` / `OHN_UNF`.

### Deductible
- [ ] Validate deductible against age category.
- [ ] Reject impossible values.
- [ ] Convert friendly values to official dataset codes.

### Insurance model categories
- [ ] Support `BASE`.
- [ ] Support `PRAXIS`.
- [ ] Support `TEL_DIG`.
- [ ] Support `PHARM`.
- [ ] Support `FLEX`.

**Done when:** a structured user profile becomes a valid official comparison profile with zero LLM involvement.

---

# PHASE 4 — Compatible-offer comparison engine

**Priority:** P0

**Goal:** Return the cheapest compatible offers, not merely the cheapest insurer.

- [ ] Filter by canton.
- [ ] Filter by premium region.
- [ ] Filter by age class.
- [ ] Filter by accident coverage.
- [ ] Filter by deductible.
- [ ] Filter by accepted model categories.
- [ ] Preserve exact tariff/product names.
- [ ] Preserve multiple valid offers from the same insurer where relevant.
- [ ] Remove logic that drops offers solely because another tariff from the same insurer is cheaper.
- [ ] Sort compatible offers by official monthly premium.
- [ ] Return a configurable number of top offers.
- [ ] Calculate annual premium in Python.
- [ ] Calculate savings/differences in Python where used.
- [ ] Add regression tests.

Core offer output should contain at least:

```json
{
  "insurer_id": "...",
  "insurer_name": "...",
  "tariff_type": "...",
  "tariff_name": "...",
  "monthly_premium": 0.0,
  "annual_premium": 0.0,
  "deductible": 0,
  "accident": false,
  "premium_year": 2027
}
```

**Done when:** known profiles return expected official offers and every number is traceable to a dataset value or deterministic calculation.

---

# PHASE 5 — Apertus structured profile extraction

**Priority:** P0

**Goal:** Apertus converts natural language into user facts and preferences, not insurance conclusions.

Raw fields should cover:
- [ ] birth year / age;
- [ ] postal code;
- [ ] municipality;
- [ ] employment status;
- [ ] weekly working hours when provided;
- [ ] deductible preference or explicit deductible;
- [ ] free-choice preference;
- [ ] GP-first preference;
- [ ] telemedicine preference;
- [ ] pharmacy-first preference;
- [ ] flexible-model acceptance;
- [ ] current insurer if mentioned;
- [ ] current premium if mentioned.

Implementation:
- [ ] Define a strict JSON schema / Pydantic model.
- [ ] Represent missing values explicitly.
- [ ] Reject invalid structured output.
- [ ] Validate every Apertus response before use.
- [ ] Retry or clarify after invalid output.
- [ ] Keep final insurance codes out of the raw extraction contract where possible.
- [ ] Ensure profile extraction never returns an invented premium.

**Done when:** realistic user messages reliably become validated raw-profile objects that Python can consume.

---

# PHASE 6 — Multi-turn clarification dialogue

**Priority:** P0

**Goal:** Start from incomplete real-life language and collect only the information needed to finish the comparison.

```text
User message
→ Apertus extraction
→ Python validation
→ profile state

if missing / ambiguous:
    → ask one targeted clarification
    → user answer
    → merge facts
    → revalidate

if complete:
    → comparison engine
```

- [ ] Keep conversation state during the session.
- [ ] Merge new facts into the profile.
- [ ] Protect previously validated facts from accidental overwrite.
- [ ] Detect contradictions.
- [ ] Ask targeted questions.
- [ ] Never ask information already known.
- [ ] Allow user corrections.
- [ ] Test a user who initially provides only part of the required profile.

**Done when:** a user can start with an incomplete description and naturally reach a correct comparison.

---

# PHASE 7 — Natural-language model compatibility

**Priority:** P0 / P1

**Goal:** Translate preferences about accessing healthcare into compatible official categories.

Examples:
- “I want complete freedom to choose my doctor.”
- “I always go through my GP.”
- “Calling or using an app first is fine.”
- “Going through a pharmacy first is fine.”
- “I am flexible as long as it is cheaper.”

- [ ] Define semantic preference fields.
- [ ] Define deterministic mapping to allowed model categories.
- [ ] Keep preference and official category separate.
- [ ] Detect incompatible preferences.
- [ ] Preserve uncertainty instead of forcing a model.
- [ ] Provide concise model explanations.
- [ ] Test that identical demographic profiles with different preferences can receive different compatible offers.

**Done when:** the engine can explain why a tariff is compatible with the user's access preferences.

---

# PHASE 8 — User interface

**Priority:** P0

**Goal:** A judge can use and understand the product without opening the code.

A lightweight Streamlit interface is sufficient.

### Conversation
- [ ] Free-text input.
- [ ] Multi-turn messages.
- [ ] Clear assistant responses.

### “What I understood”
- [ ] Municipality.
- [ ] Canton.
- [ ] Premium region.
- [ ] Age category.
- [ ] Accident status.
- [ ] Deductible.
- [ ] Accepted model categories.
- [ ] Premium year.
- [ ] Easy correction mechanism.

### Compatible offers
- [ ] Insurer name.
- [ ] Exact tariff/product name.
- [ ] Model category.
- [ ] Monthly premium.
- [ ] Annual premium.
- [ ] Deductible.
- [ ] Accident status.

### “Why this result?”
- [ ] Show exact comparison parameters.
- [ ] Identify official FOPH source data.
- [ ] Make provenance visible.

**Done when:** a new user can go from natural-language input to verified results without using the command line.

---

# PHASE 9 — Grounded Apertus explanation

**Priority:** P0

**Goal:** Apertus explains only facts already produced by the deterministic system.

- [ ] Give Apertus a structured verified-result object.
- [ ] Prohibit new numerical claims in the explanation prompt.
- [ ] Explain why offers match.
- [ ] Explain the selected access model at a high level.
- [ ] Explain trade-offs without subjective insurer ranking.
- [ ] Tell users what detailed product conditions should still be checked.
- [ ] Test that Apertus never changes a premium or computed value.
- [ ] Test adversarial requests asking Apertus to ignore official numbers.

**Done when:** the LLM improves understanding without becoming a second source of facts.

---

# PHASE 10 — High-value decision-support features

**Priority:** P1

Only start after the P0 end-to-end flow works.

### Model counterfactual
- [ ] Compare unrestricted choice vs accepted alternative models.
- [ ] Compute price differences in Python.
- [ ] Explain the trade-off with Apertus.

### Deductible explorer
- [ ] Calculate annual premium for valid deductibles.
- [ ] Model deterministic total-cost scenarios.
- [ ] Calculate break-even values where appropriate.
- [ ] Display a simple table or chart.
- [ ] Do not predict future healthcare expenditure.

### Current-plan comparison
- [ ] Accept current monthly premium.
- [ ] Calculate monthly difference.
- [ ] Calculate annual difference.
- [ ] Clearly label user-provided current-plan information.

**Done when:** each enhancement helps answer a real user decision rather than merely adding UI complexity.

---

# PHASE 11 — Evaluation and robustness

**Priority:** P0 for core evaluation, P1 for extended benchmark

**Goal:** Demonstrate reliability with measurements rather than claims.

### Evaluation dataset
- [ ] Complete profiles.
- [ ] Incomplete profiles.
- [ ] Ambiguous locations.
- [ ] Employment ambiguity.
- [ ] Age-boundary cases.
- [ ] Invalid deductibles.
- [ ] Indirect insurance-model preferences.
- [ ] Contradictory statements.
- [ ] Irrelevant information mixed into requests.
- [ ] Colloquial language.
- [ ] French cases.
- [ ] German cases.
- [ ] Italian cases.
- [ ] English cases.
- [ ] Prompt-injection-style attempts.

### Metrics
- [ ] Field-level extraction accuracy.
- [ ] Exact-profile accuracy.
- [ ] Missing-field detection.
- [ ] Ambiguity detection.
- [ ] Invalid-input rejection.
- [ ] Correct follow-up-question rate.
- [ ] Premium lookup correctness.
- [ ] Arithmetic correctness.
- [ ] Financial hallucination rate.

Target:

> **Financial hallucination rate = 0%.**

### Apertus benchmark
- [ ] Run same cases on Apertus 8B.
- [ ] Run same cases on Apertus 70B.
- [ ] Compare extraction quality.
- [ ] Compare dialogue quality.
- [ ] Compare latency.
- [ ] Document resource/deployment implications.
- [ ] Choose the model based on evidence.

**Done when:** a reproducible evaluation table can be included in the technical report.

---

# PHASE 12 — Sovereign deployment and Docker

**Priority:** P0

**Goal:** Meet the competition deployment requirements and make the project reproducible.

- [ ] Dockerise the application.
- [ ] Install dependencies automatically.
- [ ] Use `LLM_NAME`.
- [ ] Use `LLM_BASE_URL`.
- [ ] Use `LLM_API_KEY`.
- [ ] Keep secrets out of Git.
- [ ] Maintain `.env.example`.
- [ ] Implement `make run`.
- [ ] Verify `make run` launches the full app in Docker.
- [ ] Test with the hackathon Apertus endpoint.
- [ ] Document how the same interface can use self-hosted Apertus.
- [ ] Perform a clean-clone test.

Acceptance flow:

```text
clone repository
→ configure environment
→ make run
→ application starts
→ conversation works
→ comparison works
→ verified offers appear
```

**Done when:** someone who did not develop the project can launch it from a clean checkout using the documented command.

---

# PHASE 13 — Documentation and final presentation

**Priority:** P0

### Repository documentation
- [ ] Root README explains the problem.
- [ ] Explain why Apertus is necessary.
- [ ] Explain deterministic/LLM separation.
- [ ] Explain official datasets.
- [ ] Add architecture diagram.
- [ ] Add quick-start instructions.
- [ ] Document limitations.
- [ ] Document evaluation results.
- [ ] Document sovereign deployment path.

### Technical report
- [ ] Complete the official Track 2B report template.
- [ ] Respect the page limit.
- [ ] Include architecture.
- [ ] Include evaluation.
- [ ] Include 8B vs 70B evidence if available.
- [ ] Include limitations.
- [ ] Include dataset sources and licences.
- [ ] Document external AI coding assistance if required by competition rules.

### Demo
Show clearly:
1. natural-language input;
2. Apertus understanding;
3. one clarification if useful;
4. validated profile;
5. official compatible offers;
6. “Why this result?”;
7. the fact that the LLM never generates financial figures;
8. sovereign deployment story.

**Done when:** a judge can understand the problem, architecture, value and trust model without needing a private explanation from the team.

---

# FINAL RELEASE CHECKLIST

## Core functionality
- [ ] Natural-language extraction works.
- [ ] Multi-turn clarification works.
- [ ] Municipality resolution works.
- [ ] Premium-region resolution works.
- [ ] Age-class logic works.
- [ ] Accident-cover logic works.
- [ ] Deductible validation works.
- [ ] Insurance-model compatibility works.
- [ ] Official insurer names come from source data.
- [ ] Exact tariff names are preserved.
- [ ] Compatible offers are ranked correctly.
- [ ] Annual premium calculations are deterministic.
- [ ] Apertus explanations are grounded.

## Trust
- [ ] No LLM-generated financial figures.
- [ ] No hardcoded unverified insurer mapping.
- [ ] No silent guessing on ambiguous inputs.
- [ ] Data provenance is visible.
- [ ] Limitations are explicit.

## Evaluation
- [ ] Multilingual test set exists.
- [ ] Automated evaluation runs.
- [ ] Financial hallucination rate is measured.
- [ ] Key accuracy metrics are documented.
- [ ] Apertus model choice is justified.

## Deployment
- [ ] `.env` is ignored by Git.
- [ ] No API key exists in the repository history.
- [ ] Docker build succeeds.
- [ ] `make run` works.
- [ ] Clean-clone test succeeds.

## Submission
- [ ] README complete.
- [ ] Technical report complete.
- [ ] PDF report complete.
- [ ] Demo video complete.
- [ ] Final repository clean.
- [ ] Final end-to-end test passed.

---

# GitHub Issues to create

1. **[P0] Official data layer**
2. **[P0] Municipality and premium-region resolver**
3. **[P0] Deterministic LAMal rules engine**
4. **[P0] Compatible-offer comparison engine**
5. **[P0] Apertus structured profile extraction**
6. **[P0] Multi-turn clarification dialogue**
7. **[P0] Insurance-model compatibility**
8. **[P0] User interface**
9. **[P0] Grounded Apertus explanations**
10. **[P0] Evaluation and robustness**
11. **[P0] Docker and sovereign deployment**
12. **[P0] Documentation and final submission**
13. **[P1] Model counterfactual comparison**
14. **[P1] Deductible scenario explorer**
15. **[P1] Apertus 8B vs 70B benchmark**

Each Issue should contain the relevant checklist from this roadmap.

---

# Team operating rule

Every meaningful piece of work should be in one of four states:

```text
TODO
→ IN PROGRESS
→ REVIEW / TEST
→ DONE
```

`DONE` means:

> implemented + tested + reviewed by the other teammate + integrated into the end-to-end flow.

Not merely:

> “the code was written.”

---

# Final success criterion

A user who knows nothing about premium regions, LAMal tariff codes or insurer data should be able to describe their situation naturally and receive a transparent, verifiable comparison based only on official Swiss data.

The project succeeds when:

> **Apertus makes the system easy to use without making it less trustworthy.**

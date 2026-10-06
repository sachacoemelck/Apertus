# LAMal Navigator
## Hack Apertus — Track 2B: Own Project

## 1. Project in one sentence

**LAMal Navigator helps a resident understand which Swiss basic health-insurance model fits the way they actually seek care, and what each of their preferences costs, using Apertus, deterministic code and official FOPH data.**

The core principle is:

> **Apertus understands the person. Code applies the rules and calculations. Official Swiss data provides the facts.**

---

## 2. The problem we are solving

Switzerland already provides official LAMal premium data and the federal Priminfo calculator.

The problem is therefore **not a lack of data**, and it is **not that the form is hard to fill in**. Postal code, birth year and deductible are quick to enter in any form.

The real problem is the one field a form cannot help with: **the insurance model**.

To choose between BASE, PRAXIS, TEL_DIG, PHARM and FLEX, a resident must already know:

- what each model obliges them to do before seeing a doctor;
- which of those obligations they would accept in real life;
- how much money each obligation actually saves them;
- whether their accident cover can be excluded, which depends on their employment situation.

People do not think in those categories. They say things such as:

> “I rarely go to the doctor, calling first is fine, but I don't want to use an app. I'd consider a pharmacy if it really saves money.”

That sentence contains an acceptance, a rejection and a conditional preference. No dropdown captures it, and a keyword parser gets it wrong.

LAMal Navigator bridges the gap between **how people describe the way they seek care** and **official structured insurance data**, and then shows the price of each preference.

It does not replace Priminfo. It helps people reach a comparison they understand.

---

## 3. Product objective

The user explains their situation naturally in **French, German, Italian or English**.

The system then:

1. understands the relevant facts and care-access preferences with Apertus;
2. keeps the user's own words as evidence for every extracted item;
3. shows the user what it understood;
4. detects missing, uncertain or contradictory information;
5. asks only the clarification questions that are actually necessary;
6. converts the validated facts into official LAMal comparison parameters;
7. retrieves compatible offers from the official 2027 premium dataset;
8. performs every financial calculation deterministically in Python;
9. ranks the lowest-cost offers compatible with the user's preferences;
10. shows **what each preference costs**: the premium difference between the models the user accepts, the ones they reject and the ones they are unsure about;
11. explains the result in plain language with Apertus;
12. makes every result traceable to official data and explicit rules.

The product must never claim:

> “This is the best insurer for you.”

The correct claim is:

> **“These are the lowest-cost official LAMal offers compatible with what you told us, and this is what your other options would cost.”**

---

## 4. Why Apertus is genuinely necessary

Apertus must not be decoration around a normal premium calculator.

We state the boundary honestly: **explicit fields such as postal code, birth year and deductible amount do not need an LLM**. Apertus extracts them because the user is already typing, but they are not the reason the model is there.

Apertus is necessary for the part forms and deterministic parsers handle poorly: **recovering care-access preferences and employment facts from the way people actually speak**, including what they reject, what they accept only under a condition, and what they are unsure about.

Apertus is responsible for:

- interpreting indirect care-access preferences;
- distinguishing acceptance, rejection, requirement, condition and uncertainty;
- understanding negation and colloquial phrasing;
- preserving uncertainty expressed by the user instead of silently resolving it;
- interpreting corrections and updates across conversational turns;
- handling semantically equivalent input in French, German, Italian and English;
- quoting the user's words that support each extracted item;
- phrasing one concise clarification question when the deterministic layer identifies an unresolved requirement;
- explaining verified results without introducing new financial facts.

Examples:

> “I want to keep seeing any doctor I choose.”
→ free choice of doctor is **required**.

> “Calling first is okay, but I do not want to use an app.”
→ phone-first is **accepted**, digital-first is **rejected**. It must not be simplified to “telemedicine is fine”.

> “I would accept telemedicine if it saves enough money.”
→ **conditional**. The system must show the saving and let the user decide, not accept on their behalf.

> “I think my employer covers accidents, but I'm not sure.”
→ **hedged** fact. Python must not derive the accident parameter from it.

> “Actually, I work six hours a week, not ten.”
→ correction of a previously stated fact.

Without Apertus, the user must translate their habits into insurance terminology, or the application must rely on brittle keyword rules.

### Apertus-specific technical questions

The project answers two concrete questions rather than merely demonstrating a chatbot:

1. **Semantic reliability:** How reliably can Apertus turn indirect, negated, conditional, uncertain and multilingual descriptions into a structured, evidence-backed representation of what the user actually communicated, without unjustified inference?
2. **Sovereign model trade-off:** Is Apertus 8B good enough for this constrained task compared with Apertus 70B, given that it offers a much lighter on-premise deployment?

These questions make Apertus itself an object of evaluation, not only a component of the interface.

---

## 5. What Apertus must never do

The LLM is **not** the source of truth.

Apertus must never:

- invent or estimate a premium;
- calculate annual premiums or savings;
- infer a premium region by itself;
- invent an insurer or tariff;
- override an official dataset value;
- decide that an invalid input is “probably fine”;
- accept a condition on the user's behalf;
- claim that one basic insurer offers better medical coverage than another;
- predict the user's future healthcare expenditure;
- provide personalised medical advice.

If a number is displayed to the user, it must come from:

1. the official dataset; or
2. a deterministic Python calculation based on official values.

Target:

> **0% LLM-generated financial figures.**

---

## 6. Product flow

```text
USER
  ↓
Natural-language description
  ↓
APERTUS
Extract what the user communicated
(facts, stances, uncertainty, corrections, evidence quotes)
  ↓
RAW STRUCTURED USER PROFILE
  ↓
PYTHON EVIDENCE CHECK
Drop any item whose quote is not found in the user's messages
  ↓
PYTHON VALIDATION / RULE ENGINE
Validate values + merge state + determine whether
all required comparison information is resolved
  ↓
Unresolved requirement?
  ├── YES → Python identifies what must be clarified
  │          ↓
  │        Apertus phrases one targeted question
  │          ↓
  │        user answer → repeat
  └── NO
       ↓
OFFICIAL PARAMETER RESOLUTION
       ↓
FOPH DATA FILTERING
       ↓
DETERMINISTIC RANKING + CALCULATIONS
(compatible offers + cost of each preference)
       ↓
VERIFIED RESULTS
       ↓
APERTUS
Grounded plain-language explanation only
       ↓
USER
```

The boundary is deliberate:

- **Apertus answers:** “What did the user communicate, in which words, and how should we phrase the next question or explanation?”
- **Python answers:** “Is that supported, valid and sufficient for a LAMal comparison, and which official parameter follows from it?”

---

## 7. User profile

Apertus extracts **raw facts and stances with evidence**, not final insurance codes.

### Facts

Every fact carries a value, a certainty and the user's words:

```json
{
  "value": 40,
  "certainty": "stated",
  "evidence": "I work full-time"
}
```

`certainty` is one of:

- `stated` — the user said it plainly;
- `hedged` — the user said it with doubt (“I think”, “probably”);
- `not_mentioned` — value is `null`, evidence is `null`.

### Care-access stances

Every care-access preference carries a stance, an optional condition and the user's words:

```json
{
  "stance": "conditional",
  "condition": "saving",
  "evidence": "I'd consider a pharmacy if it really saves money"
}
```

`stance` is one of:

- `required` — the user insists on it;
- `accepted` — the user is fine with it;
- `conditional` — the user would accept it under a condition;
- `rejected` — the user does not want it;
- `unsure` — the user expressed doubt;
- `not_mentioned`.

### Full example

```json
{
  "birth_year": {"value": 1997, "certainty": "stated", "evidence": "born in 1997"},
  "location": {
    "postal_code": "1260",
    "municipality": "Nyon",
    "certainty": "stated",
    "evidence": "I live in Nyon"
  },
  "employment": {
    "employed": {"value": true, "certainty": "stated", "evidence": "I work full-time"},
    "hours_per_week": {"value": null, "certainty": "not_mentioned", "evidence": null},
    "same_employer": {"value": null, "certainty": "not_mentioned", "evidence": null},
    "employer_accident_cover": {"value": true, "certainty": "hedged", "evidence": "I think my employer covers accidents"}
  },
  "deductible": {"value": 2500, "certainty": "stated", "evidence": "the 2500 deductible"},
  "deductible_preference": {"value": null, "certainty": "not_mentioned", "evidence": null},
  "care_access": {
    "free_doctor_choice": {"stance": "not_mentioned", "condition": null, "evidence": null},
    "gp_first": {"stance": "not_mentioned", "condition": null, "evidence": null},
    "phone_first": {"stance": "accepted", "condition": null, "evidence": "calling first is fine"},
    "digital_first": {"stance": "rejected", "condition": null, "evidence": "I don't want to use an app"},
    "pharmacy_first": {"stance": "conditional", "condition": "saving", "evidence": "I'd consider a pharmacy if it really saves money"}
  }
}
```

Python then derives the official parameters:

```json
{
  "canton": "VD",
  "municipality": "Nyon",
  "premium_region": "PR_REG_1",
  "age_class": "AKA_03_ERW",
  "accident": "MIT_UNF",
  "deductible_code": "FRA_06_E_2500",
  "accepted_tariff_types": ["TEL_DIG", "FLEX"],
  "counterfactual_tariff_types": ["BASE", "PRAXIS", "PHARM"]
}
```

In this example accident cover stays included because the employer cover is only hedged.

**Apertus extracts meaning and quotes it. Python derives insurance parameters.**

---

## 8. Uncertainty and validation

Every required comparison field is in one of four states:

- `KNOWN`
- `MISSING`
- `AMBIGUOUS`
- `INVALID`

The system **fails closed**.

### The evidence rule

Every extracted item must quote the user. Python checks that the quote appears in the user's messages, after normalising case and whitespace.

If the quote is not found, the item is discarded and logged as an unjustified inference. Apertus therefore cannot add a fact the user never said.

### How Python treats certainty and stance

| Extracted | Python behaviour |
|---|---|
| fact `stated` | used, then validated against domain rules |
| fact `hedged` | not used to derive a parameter; a clarification is requested or the safe default applies |
| fact `not_mentioned` and required | `MISSING`; a clarification is requested |
| stance `required` | only the matching model category is accepted |
| stance `accepted` | the matching category is included in the main result |
| stance `rejected` | the matching category is excluded |
| stance `conditional` | excluded from the main result, shown in the cost-of-preference view with its saving |
| stance `unsure` | one clarification question; if still unsure, treated as `conditional` |
| all care-access stances `not_mentioned` | one clarification question; with no answer, only BASE is accepted and the other models appear in the cost-of-preference view |

The system never assumes the user accepts a restriction they did not accept.

### Other examples

- a postal code maps to several municipalities → Python marks location `AMBIGUOUS` and requests the municipality;
- employment information cannot establish accident cover → accident cover stays included until the user confirms;
- “a high deductible” → kept as a preference; Python shows the valid amounts for the age class and asks which one;
- an invalid deductible is requested → Python rejects it and supplies the valid values;
- two stated facts conflict across turns → the state layer marks a contradiction and asks the user to resolve it.

The product never uses an LLM “confidence score” as a substitute for explicit validation.

---

## 9. Official data sources

The application works from local copies of official Swiss data.

### 2027 FOPH premium dataset

Source of truth for insurer identifier, canton, premium region, age class, accident inclusion, deductible, tariff type, exact tariff name and monthly premium.

### 2027 premium-region dataset

Used for **municipality → canton → premium region**.

Postal code can help identify the location, but the **municipality** determines the premium region.

### Official list of authorised insurers

Used for **insurer identifier → official insurer name**.

The system must not rely on manually hardcoded insurer names.

### Constraints

- The `data/` directory of the submission must stay under 100 MB.
- Licence and attribution of every dataset are recorded. If redistribution terms are unclear, the repository ships a reproducible download step instead of the raw file.

---

## 10. Insurance models

The comparison supports the official 2027 tariff categories:

- `BASE` — standard / unrestricted model;
- `PRAXIS` — first contact through a designated practice, GP or medical centre;
- `TEL_DIG` — first contact by telephone or digital channel;
- `PHARM` — first contact through a participating pharmacy;
- `FLEX` — flexible/hybrid restricted-choice model.

The user does not need to know these labels.

### Default stance-to-category mapping

To be validated against the official category definitions before it is frozen:

| Category | Included in the main result when |
|---|---|
| `BASE` | `free_doctor_choice` is `required` or `accepted`, or no restricted channel is `accepted`; otherwise it appears in the cost-of-preference view as the free-choice reference |
| `PRAXIS` | `gp_first` is `accepted` or `required` |
| `TEL_DIG` | `phone_first` or `digital_first` is `accepted` or `required`; if the other one is `rejected`, each offer is flagged “check the product's first-contact channel” |
| `PHARM` | `pharmacy_first` is `accepted` or `required` |
| `FLEX` | `free_doctor_choice` is not `required` and at least one restricted channel is `accepted`; always flagged for product-level verification |

If `free_doctor_choice` is `required`, only `BASE` is accepted.

The application preserves the **exact tariff name**, because one insurer may offer several products with different prices and access rules.

> **The unit of comparison is an offer/tariff, not simply an insurer.**

A category match does not prove that every product-level condition suits the user. The interface says so.

---

## 11. Comparison logic

The ranking is transparent and auditable.

1. resolve the validated user profile;
2. filter official data by canton and premium region;
3. filter by age class;
4. filter by accident coverage;
5. filter by deductible;
6. filter by accepted insurance-model categories;
7. preserve distinct tariff/product offers;
8. sort compatible offers by official monthly premium;
9. return a small number of lowest-cost compatible offers;
10. compute the cost-of-preference view over the remaining categories.

No subjective insurer score is used.

---

## 12. Decision support: what each preference costs

This is the heart of the product, not an add-on.

### A. Cheapest compatible offers

> “Given what you told me, which official offers satisfy those constraints at the lowest premium?”

### B. Cost of each preference

For the same validated profile, Python computes the lowest premium available in each model category and the annual difference from the user's current choice.

Example:

> “With the models you accept, offers start at CHF X/month.
> Keeping free choice of doctor would cost CHF A more per year.
> Accepting a pharmacy first would save CHF B per year.”

X, A and B are calculated by Python from official values. Apertus explains the trade-off in plain language and never changes the numbers.

Conditional stances are resolved here: the user who said “only if it really saves money” sees the saving and decides.

This lets the user understand **what they are paying for**, not only which row is cheapest.

---

## 13. Output

The final experience is structured rather than a wall of chatbot text.

### 1. What I understood

```text
Residence        Nyon (VD)
Premium region   Region 1
Age category     Adult
Accident         Included (employer cover not confirmed)
Deductible       CHF 2,500
Accepted         Phone first
Rejected         App
Depends on price Pharmacy first
Premium year     2027
```

Each line can show the user's own words that led to it. The user can correct any line.

### 2. Compatible offers

For each result: insurer, exact product, model category, monthly premium, annual premium, deductible, accident status, and any “check product conditions” flag.

### 3. What your preferences cost

One line per model category the user did not accept, with the lowest premium and the annual difference.

### 4. Why this result?

The exact parameters used by the engine and the official source dataset.

### 5. Apertus explanation

A short explanation of why the offers match, what the model means in daily life, the relevant trade-off, and what should still be verified in the insurer's conditions.

No new numerical claim is introduced here.

---

## 14. Privacy and sovereignty

### Target architecture

**On-premise.** The complete system, including the model, runs on infrastructure controlled by the operator.

```text
Browser
   ↓
Application container
   ├── deterministic Python engine
   ├── local FOPH datasets
   └── Apertus endpoint (configurable)
            ├── hackathon endpoint   → demo and judging
            └── local Apertus 8B     → on-premise mode
```

The model is configured only through:

```text
LLM_NAME
LLM_BASE_URL
LLM_API_KEY
```

### Two run modes

- `make run` — the application in Docker, using the endpoint from the environment. This is what judges run.
- `make run-local` — the same application plus a local inference server serving Apertus 8B. No call leaves the machine.

Model weights are downloaded at build time and are never committed to the repository.

### Dependencies

| | Build time | Runtime, on-premise mode |
|---|---|---|
| Package registries | yes | no |
| Model weights download | yes | no |
| Official dataset download | yes, if not bundled | no |
| External LLM API | no | no |
| Any proprietary service | no | no |

### Evidence we will show

- the benchmark run on Apertus 8B and on 70B, with quality, latency and hardware used;
- one end-to-end run in on-premise mode with outbound network blocked.

If the team's hardware cannot serve 8B locally, the report says so and states exactly what was and was not tested.

### Privacy

No user conversation needs to be stored for the core use case. No medical history is required. Secrets remain outside Git and are loaded through environment variables.

---

## 15. Scope and limitations

### In scope

- Swiss compulsory/basic health insurance;
- official 2027 premiums;
- conversational extraction of facts and care-access stances;
- evidence-backed extraction;
- multi-turn clarification;
- municipality/premium-region resolution;
- age-class, accident-cover and deductible rules;
- official insurance-model categories;
- compatible-offer comparison;
- cost of each care-access preference;
- multilingual interaction;
- traceability and explanation.

### Explicitly out of scope

- supplementary insurance;
- medical advice;
- prediction of future healthcare use;
- deductible optimisation or break-even analysis;
- comparison with the user's current contract;
- subjective insurer-quality ratings;
- automatic policy purchase or cancellation;
- health-risk profiling;
- claims analysis;
- doctor/network guarantees at product level;
- personalised legal advice.

The application states clearly that it is a prototype decision-support tool and not an official FOPH service.

---

## 16. Evaluation

Evaluation is a visible strength of the project and tests **Apertus itself**, not only whether the deterministic premium engine works.

### A. LAMal Apertus Challenge Set

A labelled benchmark of realistic inputs: **60–100 single-turn cases plus 10–20 multi-turn conversations**, with a paired FR/DE/IT/EN subset.

Challenge classes:

- explicit complete profiles;
- missing information;
- ambiguous locations;
- employment and accident-cover uncertainty;
- invalid deductibles and age-boundary cases;
- indirect care-access preferences;
- negation;
- conditional preferences;
- mixed stances in one sentence;
- corrections and contradictions across turns;
- hedging;
- irrelevant information mixed with relevant facts;
- colloquial phrasing;
- French, German, Italian, English and selected mixed-language inputs;
- prompt-injection-style attempts to override application rules.

Each case defines the expected facts, certainties, stances and evidence, and whether clarification is required. Labels reflect **what the user actually communicated**, not what domain knowledge could infer.

### B. Rules that keep the evaluation credible

- **Separate authorship.** The contributor who writes the test cases does not write the prompts, and the reverse.
- **Held-out split.** The set is split into a development part and a test part. The test part is frozen before any prompt tuning, and only test-part results are reported.
- **Human-verified labels.** Any case drafted with the help of a model is checked by a person before it is accepted.
- **Open-weights only.** Hackathon rules allow only open-weights models to support development. Any model used to draft cases or judge outputs is open-weights and its role is described in the report.

### C. Systems compared

| System | Purpose |
|---|---|
| Keyword/regex extractor | shows where simple rules are enough |
| Apertus with a plain prompt, no schema, no evidence check, no validation | shows what the architecture adds beyond the model |
| LAMal Navigator on Apertus 8B | the light on-premise option |
| LAMal Navigator on Apertus 70B | the quality reference |

### D. Headline metrics

Six numbers appear in the report:

1. **exact-profile accuracy** — all facts and stances correct;
2. **stance accuracy** — on negation, conditional and uncertain cases;
3. **unjustified-inference rate** — items produced without support in the user's words, measured before and after the evidence check;
4. **cross-language consistency** — same meaning gives the same validated profile in FR/DE/IT/EN;
5. **end-to-end task success** — the conversation reaches the correct official offers;
6. **financial hallucination rate** — target 0%.

A financial hallucination is:

> **any monetary value shown to the user that cannot be traced exactly to an official dataset value or to a deterministic calculation over verified values.**

Secondary metrics go to the repository, not the report: field-level accuracy, malformed-output rate, correction handling, clarification-target selection, latency.

### E. Deterministic correctness

Kept separate from the model evaluation:

- Priminfo golden-case parity;
- official-parameter correctness;
- arithmetic correctness, including the cost-of-preference figures.

### F. Failure analysis

Failures are classified so the report can explain where Apertus succeeds and where it remains unreliable:

- `E1` — wrong explicit fact extraction;
- `E2` — unjustified inference;
- `E3` — missed uncertainty;
- `E4` — negation / conditional-stance error;
- `E5` — correction or contradiction handling error;
- `E6` — multilingual inconsistency;
- `E7` — malformed structured output;
- `E8` — unsupported explanatory claim or attempted financial hallucination.

### G. Apertus 8B vs 70B

The same test part is run on both models. The runtime model is chosen from measured results.

If Apertus 8B is close enough to 70B on this constrained task, that is a central result for cost, scalability and on-premise deployability. If it is not, the report says so.

---

## 17. Differentiation from Priminfo

Priminfo is the authoritative reference, not a competitor to criticise.

### Priminfo

```text
User already knows which model they want
→ enters structured values
→ receives official comparison
```

### LAMal Navigator

```text
User describes how they seek care
→ Apertus captures what they accept, reject and are unsure about
→ the system resolves and validates official parameters
→ official data produces the comparison
→ the user sees what each preference costs
```

Positioning:

> **Priminfo answers “what does this model cost?”. LAMal Navigator answers “which model fits me, and what does my preference cost?”.**

---

## 18. Competition strategy

The project deliberately addresses the five Track 2B judging dimensions.

### Purposeful use of AI

Apertus is used only where language is the difficulty: stances, negation, conditions, uncertainty, corrections, four languages. Its value is measured against a regex extractor and against Apertus without our architecture.

### Technical rigour

- evidence-backed structured extraction;
- explicit validation and fail-closed behaviour;
- deterministic business rules;
- numerical provenance;
- Priminfo golden tests;
- held-out multilingual evaluation with separate authorship;
- failure taxonomy.

### Value, cost and scalability

Official national data, one reusable architecture, yearly refresh by replacing datasets, and a measured answer to whether the small model is enough.

### Sovereign deployability

A stated on-premise target, a local run mode, a build-time/runtime dependency table, and a network-blocked run.

### Implementation feasibility

- no insurer-website scraping;
- no medical prediction;
- no proprietary data;
- no automatic policy switching;
- no subjective ranking model.

One reliable end-to-end product rather than many partial features.

---

## 19. Collaboration model

Both contributors treat this document as the shared product contract.

### Deterministic layer

Official data loading, municipality and region resolution, insurer resolution, age and accident rules, stance-to-category mapping, comparison, cost-of-preference calculation, validation, tests.

This contributor also **writes and labels the challenge set**.

### Apertus / product layer

Structured extraction, evidence quotes, conversation state, clarification, multilingual behaviour, grounded explanation, interface.

This contributor **writes the prompts** and does not see the frozen test part.

The two layers communicate only through structured objects:

```python
raw_profile = extract_with_apertus(message, conversation_state)

supported_profile = check_evidence(raw_profile, conversation_state)

validated_profile = validate_and_resolve(supported_profile)

if validated_profile.needs_clarification:
    return ask_with_apertus(validated_profile)

offers = compare_official_offers(validated_profile)
preference_costs = cost_of_preferences(validated_profile)

return explain_with_apertus(validated_profile, offers, preference_costs)
```

---

## 20. Product principles

- **Correctness before cleverness.** A smaller verified system beats a sophisticated unreliable one.
- **Deterministic whenever possible.** If a rule can be encoded and tested, use code.
- **AI where language matters.** Use Apertus for stances, clarification and explanation.
- **Every extracted item quotes the user.** No quote, no fact.
- **Official data first.** FOPH/Priminfo data is the source of truth.
- **Ask rather than guess.** Never accept a restriction on the user's behalf.
- **Explain every result.** The user should understand why an offer appeared.
- **Preserve sovereign portability.** Changing the Apertus endpoint requires no product-logic change.
- **Measure rather than claim.** Accuracy and model choice are demonstrated through evaluation.
- **Avoid feature inflation.** A trustworthy complete product is stronger than a collection of demos.

---

## 21. What a strong final project demonstrates

A resident describes how they seek care, in their language.

Apertus captures what they accept, reject and are unsure about, and quotes them.

The system discards anything the user did not say and asks only what is missing.

Deterministic Swiss rules convert the conversation into a valid comparison profile.

Official FOPH data provides the offers and premiums.

Python calculates every number, including what each preference costs.

Every result can be inspected and verified.

The same application runs against a locally hosted Apertus 8B with no outbound call.

Its behaviour is measured on a held-out multilingual challenge set, against a regex extractor and against Apertus without the architecture, and analysed by failure type.

> **measured semantic value from Apertus + evidence-backed extraction + deterministic correctness + official Swiss data + the cost of each preference + on-premise deployment**

---

## North star

> **Help residents choose a basic-insurance model they understand: Apertus captures how they seek care, code enforces the rules, official data sets the price of each preference, and every result can be verified.**

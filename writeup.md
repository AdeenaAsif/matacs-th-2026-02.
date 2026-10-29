# Q1: 
## AI vs. Rules
### AI:

- Handles proposal text parsing and structured extraction of key facts (experience, contractor license, dates, credentials, etc.).

- Extracts exact source citations for future logging and audit purposes.

- Takes security measures to avoid falling for explicit instructions (such as pre-approval notes and appendices).

- Identifies ambiguous or vague information instead of inferring new details.

### Rules:

- LLMs/AI are avoided for evaluation and ranking against extracted information.

- Deterministic methods verify all mandatory criteria against the standard.

- Mathematical rules calculate the total weightage of other fields.

- System rules resolve flags and ambiguous fields.

# Q2: 
## Consistency and Explainability
### Consistency:

- Temperature configuration is set to 0 to keep responses grounded.

- The model is strictly instructed to generate output in a structured schema rather than dense paragraphs.

- Arithmetic is delegated to mathematical formulas and analytics rather than AI, maximizing the likelihood of identical proposals receiving identical scores.

### Explainability:

- Rankings are explained through cited sources, as directed in the prompt instructions.

- Explanations are reinforced by the reason stated in the final rankings.

- A clear score breakdown is provided to assist reviewer understanding.

# Q3:
## Human in the Loop & Audit Logging
### Human Oversight & Action:

- A human reviewer can accept or reject a proposal.

- Automation pauses whenever a proposal is flagged for ambiguous/vague fields, unmet criteria, or security compliance issues.

- A human reviewer must intervene and make the decision on flagged items.

### Auditing:

- A log sheet records metadata including timestamps, reviewer_ids, count of human interventions, and raw AI responses.

- This log serves as supporting evidence whenever reviewers make decisions.

# Q4:
## Changing Criteria
### Policy on Scored Proposals:

- Already-ranked proposals are not re-evaluated from scratch.

### Handling Updates:

- If new criteria or weightings are added, only the mathematical evaluation scripts are updated to version (n+1).

- All previous records and version snapshots are saved.

# Q5: 
## Risks and Practical Safeguards
#### 1. Manipulation & Adversarial Prompt Injection:

What could go wrong: Bidders might insert hidden text, white-on-white text, or adversarial instructions directly into PDFs (e.g., Proposal C's fake pre-approval note in Appendix C) attempting to trick the LLM into giving full marks or skipping mandatory checks.

#### Practical Safeguards:

Isolate all raw document content within strict XML boundary tags (<proposal_text>) so the model treats it strictly as untrusted data, never as system instructions.

Run a dual-pass check: an initial lightweight security classifier detects jailbreak or injection patterns before full parsing begins.

Automatically quarantine and flag any proposal containing directives attempting to alter scoring, claim pre-approval, or bypass procurement rules, requiring mandatory human clearance.

#### 2. Wrong Extraction & Missing Fields:

What could go wrong: Variable layout formatting, unusual font encodings, scanned pages, or buried footnotes might lead the model to extract incorrect figures (e.g., confusing system size with panel quantity or misreading price exclusions).

#### Practical Safeguards:

Enforce strict, typed schema validation using Pydantic (e.g., verifying integer week counts, positive float dollar values, and valid licence string formats).

Require the model to return the verbatim supporting quote and source page alongside every extracted field.

If critical data is missing or ambiguous (e.g., Proposal D omitting a firm timeline), the system marks the field as null and raises an automatic review flag instead of assuming compliance.

#### 3. Hallucination & Made-Up Facts:

What could go wrong: Generative models can invent missing details (e.g., fabricating an electrical licence number or guessing a 10-year warranty when none is stated) to complete a form.

#### Practical Safeguards:

Explicit prompt negative constraints: strictly instruct the model that if an item is not explicitly confirmed in the text, it must output null.

Temperature set to 0.0 to eliminate creative variance and maximize determinism.

Automated quote-matching verification: programmatic checks confirm that the cited source snippet actually exists in the raw extracted PDF text.

#### 4. Algorithmic & Brand Bias:

What could go wrong: The model might unconsciously favor well-known national brands over local contractors, or exhibit demographic/regional bias during qualitative rubric evaluations (such as evaluating the team and approach).

#### Practical Safeguards:

Implement blind evaluation: strip company names, brand logos, and identifying metadata before sending narrative text to the qualitative evaluation prompt.

Base all quantitative scoring entirely on deterministic code and mathematical formulas rather than LLM discretion.

#### 5. Operational Cost & Processing Latency:

What could go wrong: Re-evaluating entire batches of large PDF proposals multiple times (especially when admins tweak weights or criteria) can cause excessive API token usage, slow triage turnaround, and rate-limit bottlenecks.

#### Practical Safeguards:

Single-pass parsing: extract structured JSON data once and cache it in the database.

Criteria and weight changes never re-invoke the LLM; they run purely as sub-millisecond mathematical calculations across the cached structured data.

Use lightweight, cost-effective models optimized for structured extraction rather than expensive reasoning models for standard parsing.

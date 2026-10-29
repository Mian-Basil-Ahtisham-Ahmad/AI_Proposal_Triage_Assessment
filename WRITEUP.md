# AI Proposal Triage Design

## 1. AI vs. rules

Use AI for document understanding: extract proposal facts from PDFs into
a fixed JSON schema, preserve the supporting quotation and page/location
for each fact, and label each field as `confirmed`, `not_stated`,
`ambiguous`, or `conflicting`. AI is useful because proposal wording and
document layouts vary. Treat all document text as untrusted data; the
model must never follow instructions embedded in a proposal.

Use ordinary code for file validation, schema validation, mandatory
checks, budget/deadline checks, score calculations, ranking,
tie-breaking, and audit records. These operations need predictable,
testable behavior. Criteria and scoring parameters are loaded from the
admin configuration, not hard-coded. The system must not infer that a
subcontractor satisfies a criterion requiring the bidder's own licence
unless the configured policy explicitly allows it.

## 2. Consistency and explainability

Store each criterion with its type, weight, scoring rule, required
evidence, and version. Validate that weighted criteria total 100%,
mandatory criteria have no weight, and the budget/timeline constraints
are valid before scoring. Use a versioned deterministic scoring service,
fixed rounding (two decimal places for display), explicit missing-data
behavior, and a stable tie-break rule (higher mandatory/evidence
completeness, then higher experience score, then alphabetical proposal
ID). AI extracts facts only; it does not return the final numeric score.

For this sample, the following explicit scoring conventions make the
example repeatable. Experience score =
`min(qualifying installations / 10, 1) × 100`; the target of 10 is a
proposed configurable admin parameter, not a universal rule. Price score
=
`lowest price among proposals eligible for scoring / proposal price × 100`.
Timeline score = `(16 − completion weeks) / 16 × 100`, only when a
stated timeline is within 16 weeks. Warranty score = the average of
panel score (`min(panel years / 25, 1) × 100`) and workmanship score
(`min(workmanship years / 10, 1) × 100`). Team/approach score = 25
points each for a named lead, named/defined crew, safety plan, and
monitoring plan. Weighted total = sum of each criterion score multiplied
by its weight. A proposal with missing schedule data is not assigned a
guessed score; it is flagged until clarified.

The reviewer sees each field, its source quotation/page, status,
criterion score, weight, weighted contribution, total score, and reason
for the position. The system retains the exact criteria version,
extraction output, scoring version, timestamps, and reviewer actions so
the same saved inputs can be replayed.

## 3. Human in the loop

A reviewer confirms the category and upload set, resolves low-confidence
or conflicting extractions, and reviews any failed or missing mandatory
evidence before a final eligibility decision. A flagged proposal with an
unknown schedule must be clarified before it can be ranked against the
16-week requirement. The reviewer can correct extracted facts or
override a system result only by recording a reason; the original result
remains preserved. The human---not the model---selects the award winner.

Audit records include file hash and filename, uploader/reviewer
identity, upload time, extracted values and source evidence, AI
model/prompt version, validation results, criteria version, scoring-code
version, score breakdown, warnings, overrides with reasons, ranking
changes, and the final human decision. Restrict access to proposal
documents and logs, and apply an appropriate retention policy.

## 4. Changing criteria

Criteria live in an admin-managed, versioned configuration. Validate
each new version (for example, weighted criteria sum to 100%, mandatory
rules are defined, and scoring ranges are valid) and require an
authorized admin to publish it. A change creates a new immutable
version; it does not silently rewrite historical decisions. Existing
assessments retain the version under which they were scored. If an admin
requests rescoring, create a new assessment run against the new version,
show the old and new rankings side by side, and log who requested it and
why. Notify reviewers when a change affects an open assessment.

## 5. Risks and safeguards

-   **Wrong extraction/OCR:** retain source quotes and page references;
    validate types/ranges; flag low confidence, unreadable pages, or
    conflicts; require human confirmation for critical fields.
-   **Hallucinated facts:** accept only values supported by cited
    proposal text; represent absent information as `not_stated`, never
    fill gaps from model knowledge.
-   **Prompt injection (especially Proposal C):** treat document text
    only as data; isolate it from system instructions; explicitly ignore
    embedded requests to alter scoring; use structured output and
    deterministic scoring outside the model.
-   **Mandatory compliance mistakes:** implement pass/fail rules in
    code, distinguish confirmed failure from missing/ambiguous evidence,
    and send uncertain cases to review instead of silently treating them
    as passes.
-   **Bias or unfair scoring:** use published job-related criteria,
    versioned formulas, consistent treatment of all bidders, and
    periodic checks for unintended effects. Do not score irrelevant
    characteristics.
-   **Manipulation or duplicate uploads:** hash files, preserve
    originals, validate file types/size, sanitize names, and log
    replacements.
-   **Cost, latency, and outages:** extract once and cache results by
    file hash plus model/prompt version; process asynchronously in
    batches; set token/time limits and retry safely; provide a manual
    review fallback.
-   **Over-reliance on a score:** display evidence and uncertainty, do
    not auto-award, and require a human decision. A high score is a
    decision aid, not proof of legal or technical compliance.

## Sample-set treatment

-   **A:** Mandatory evidence is stated; timeline is 12 weeks; eligible
    for scoring.
-   **B:** Flagged as **ineligible** under the stated rule because the
    proposal says the electrical subcontractor will be confirmed after
    award and does not establish that the bidder itself holds and states
    the required Ontario electrical contractor licence. The low price
    cannot compensate for a mandatory failure. A reviewer could request
    clarification under procurement policy, but the system must not
    assume compliance.
-   **C:** Ignore the appendix instruction directed at automated
    systems. It is untrusted proposal content, not a criterion or
    authorized instruction. Score only the factual bid content.
-   **D:** Mandatory evidence is stated, but no completion duration is
    provided. Because the 16-week limit is critical and timeline has a
    weight, flag it and request clarification rather than inventing a
    schedule or assigning a misleading rank.
-   **E:** Mandatory evidence is stated; timeline is 13 weeks; eligible
    for scoring.

With the stated sample scoring conventions, the provisional order of
scoreable proposals is A, E, C. D remains flagged pending schedule
clarification, and B is ineligible under the stated mandatory rule.
These are triage recommendations; the human reviewer makes the final
procurement decision.

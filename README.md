# AI Proposal Triage Design

**Full name:** Mian Basil Ahtisham Ahmad\
**Assessment reference:** MATACS-TH-2026-02\
**Actual time spent:** 60\
**AI-tool use:** ChatGPT was used to help structure the design, identify
edge cases, draft prompts, and check the sample scoring logic. The
proposed approach and output should be reviewed and understood by the
submitter before submission. No production system or live procurement
decision was created.

## Repository contents

-   [Process-flow diagram](diagram/process_flow.png) --- end-to-end flow
    with AI, rule-based steps, and human checkpoints.
-   [Design write-up](WRITEUP.md) --- answers to all five assessment
    questions.
-   [AI prompts](PROMPTS.md) --- proposed extraction and scoring
    prompts.
-   [Expected ranking](ranking.json) --- expected treatment of proposals
    A--E.

## Design summary

The system uses AI to extract facts from untrusted proposal documents
into a strict schema. Deterministic code validates evidence, applies
mandatory pass/fail rules, calculates weighted scores using the admin's
versioned criteria, and produces an auditable ranking. Missing or
ambiguous mandatory evidence and missing critical schedule information
are flagged for human review. A human makes the final award decision.


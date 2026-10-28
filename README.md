# AI Proposal Triage Design - Technical Assessment

- **Candidate Name:** Ahmed Shahi
- **Assessment Reference:** MATACS-TH-2026-02
- **Target Role:** Associate Software Developer
- **Time Spent:** ~55 minutes

---

## Repository Structure & Links

- [`diagram/`](diagram/) - Process flow architecture diagram (upload to triage).
- [`WRITEUP.md`](WRITEUP.md) - Architectural decisions, AI vs. rules rationale, edge case analysis, and risk mitigations (<1,000 words).
- [`PROMPTS.md`](PROMPTS.md) - Exact production-ready LLM system instructions and extraction schema.
- [`ranking.json`](ranking.json) - Final deterministic evaluation and triage status for sample proposals A–E.

---

## AI Usage Statement

AI assistance (Claude / ChatGPT) was utilized as an interactive sounding board to draft initial regex patterns for document sanitization and summarize prompt injection mitigations. 

### Correction / Disagreement with AI Output
When initially prompted to evaluate the sample proposals, the LLM suggested giving **Proposal B** a passing score on mandatory criteria by assuming the subcontractor's future licensing would suffice once awarded. I explicitly overrode and corrected this logic: the mandatory requirement strictly demands that the bidder holds and states their own Ontario electrical contractor licence directly in the proposal. Consequently, Proposal B was marked as `ineligible`. Furthermore, the LLM initially failed to isolate the prompt injection in **Proposal C** (Appendix C) and attempted to rank it first; I corrected this by decoupling unstructured extraction from deterministic scoring and enforcing an injection detection rule.
# Process Design & Architecture Write-Up (MATACS-TH-2026-02)

## 1. AI vs. Rules

### Distribution of Responsibilities
- **AI (LLM) Layer:** Exclusively handles raw text parsing, semantic entity extraction, and prompt injection threat detection from unstructured PDF proposals into a strict JSON payload.
- **Rule-Based Engine (Deterministic Code):** Executes mandatory pass/fail criteria validation, boundary threshold checks (e.g., budget caps, timeline limits), and weighted mathematical score calculations.

### Justification of the Boundary
Unstructured proposals feature unpredictable layouts, differing terminologies, and narrative text that traditional regular expressions fail to parse robustly. LLMs excel at understanding this contextual nuance. 

Conversely, delegating arithmetic scoring or pass/fail determinations to an LLM introduces severe risks: non-deterministic output drift, mathematical hallucinations, and vulnerability to adversarial prompt phrasing. By strictly confining the LLM to an extraction only boundary, the evaluation logic remains 100% deterministic, auditable, and testable using unit tests in code.

---

## 2. Consistency and Explainability

### Achieving Repeatability
- **Determinism in Extraction:** The extraction model operates at `temperature: 0.0` with structured schema enforcement (JSON Schema Mode) and a fixed random seed.
- **Deterministic Math:** Once the entities are extracted into numbers and booleans, all ranking calculations run inside application code. Running the pipeline multiple times over identical inputs produces the exact same numerical scores and rankings every time.

### Reviewer Explainability
Reviewers receive a transparent breakdown rather than an opaque aggregate score:
- **Criteria Line-Item View:** Shows individual score contributions (e.g., Experience: 28/30, Price: 24.1/25).
- **Extracted Ground Truth & Citations:** Each line item highlights the verbatim excerpt and page index extracted from the uploaded proposal so reviewers can independently confirm the source fact.

---

## 3. Human in the Loop

### Verification and Override Checkpoints
The system is built as a decision-support instrument, not an autonomous gatekeeper. Human intervention points include:
1. **Automated Exceptions Triage:** Proposals marked as `ineligible` (failed mandatory criteria) or `flagged` (security triggers, missing scope) bypass automatic ranking and are sent to a dedicated manual exception queue.
2. **Reviewer Override:** Reviewers can inspect extracted metrics, update an erroneously extracted value, or clear a false-positive security flag before final ranking.
3. **Award Decision:** The tool outputs a prioritized recommendation; the human reviewer retains sole authority for the final contract award.

### Audit Logging
To ensure compliance and public sector transparency, the system records an immutable audit log containing:
- Timestamp and unique Reviewer ID.
- Snapshot of the active criteria configuration version.
- Raw extracted JSON alongside any reviewer-modified values.
- Mandatory text justification provided by the reviewer whenever an automated score or flag is overridden.

---

## 4. Changing Criteria

### Configuration Versioning
Criteria categories, mandatory rules, and scoring weights are never hard-coded; they are defined via admin tables and tagged with immutable semantic versions (e.g., `solar-comm-v1.0`). 

### Handling Mid-Cycle Criteria Changes
- **Historical Immutability:** Proposals already triaged and scored remain linked to their original criteria version snapshot. Updating criteria never silently mutates existing historical rankings.
- **Batch Re-evaluation Workflow:** When an administrator publishes a new criteria version (e.g., `solar-comm-v2.0`), the system triggers a prompt: *"Criteria updated. Re-evaluate existing cohort under version 2.0?"*
- If confirmed, the engine re-runs the deterministic rule calculations across previously extracted entity payloads without needing to re-bill LLM token extractions, creating a clean, versioned comparison run.

---

## 5. Risks and Safeguards

| Failure Mode / Risk | Impact | Design Safeguard |
| :--- | :--- | :--- |
| **Prompt Injection Attacks** | Untrusted proposal text forces the LLM to award top rank (e.g., Proposal C's Appendix C note). | Dual-channel separation: Input text is encapsulated within strict XML/quote boundaries in the user prompt. A dedicated extraction instruction flags adversarial commands and immediately shunts the proposal to `status: "flagged"`. |
| **Hallucination / Fact Invention** | LLM fabricates missing licence numbers or warranties. | Strict zero-shot constraints enforcing `null` defaults on omitted fields; source citation requirements mandate returning verbatim source snippets for each field. |
| **Licensing Ambiguity** | Unqualified contractors citing future or subcontractor credentials (e.g., Proposal B). | Deterministic Boolean checks require licence numbers to be explicitly registered to the prime bidding entity. |
| **Incomplete Scope** | Missing milestone timelines (e.g., Proposal D). | Schema-level validation traps `null` values in required project scope fields, routing the submission to `flagged` for reviewer inspection. |
| **API Latency & Token Costs** | Large PDF payloads inducing timeouts and cost overruns. | Text pre-processing strips binary noise and isolates relevant technical/commercial sections before sending batches to the LLM. |



# ADR-002: Human-in-the-loop escalation as core architecture

- **Date:** 2026-07-09
- **Status:** Accepted

## Decision

Escalation to a human reviewer is a **first-class outcome** of the pipeline, not an error state. Every agent output carries a confidence score and a machine-readable status. Two conditions route an invoice to the human review queue (Django admin) instead of proceeding automatically:

1. **Agent A (Extractor)** reports confidence below a configurable threshold.
2. **Agent B (Validator)** raises any mismatch flag (price, quantity, missing/extra items, totals).

Nothing below threshold or flagged is ever silently auto-posted. Thresholds live in settings, not code.

## Context

BillAgent processes real financial documents for SMBs. The failure mode that destroys user trust is not "the system couldn't read my invoice" — it is "the system read my invoice wrong and posted it anyway." A wrong amount silently entering bookkeeping is worse than a delay.

Meanwhile, the input space is hostile to full automation: Indonesian SMB invoices arrive as digital PDFs and phone photos, with inconsistent layouts per vendor, in Bahasa Indonesia, sometimes with handwritten notes. Extraction and validation will be probabilistic, not deterministic. A system that pretends otherwise is lying about its own reliability.

Invoice-to-PO validation is also inherently gray: partial deliveries, rounding differences, tax handling, fuzzy item-description matching. Many cases genuinely require human judgment.

## Alternatives considered

- **Full automation (no human queue)** — simplest to build, best demo throughput. Rejected: unacceptable failure mode for financial data; also produces a naive-looking portfolio piece. Knowing when *not* to decide autonomously is the mark of a mature agent system.
- **Human reviews everything (agent as suggestion-only)** — safest. Rejected: destroys the value proposition (time saved ≈ 0); the product becomes a fancy form-filler.
- **Confidence-based escalation (chosen)** — automation where the system is sure, humans where it is not. The threshold becomes a tunable business decision (measured by the *escalation precision* metric: of escalated invoices, what % genuinely needed a human).

## Trade-offs accepted

- **Lower throughput:** some invoices wait for a human. Accepted — correctness beats speed for financial records.
- **Confidence scoring is itself hard:** LLM self-reported confidence is unreliable; we will need calibration against ground truth (part of the eval work). The escalation-precision metric exists to keep us honest here.
- **Extra build surface:** review queue UI, reviewer actions (approve/correct/reject), audit trail. Mitigated by Django admin (see ADR-001 — this requirement is *why* Django won over FastAPI).
- **Two-sided failure:** thresholds too strict → humans drown in escalations; too loose → wrong data slips through. This tension is permanent and must be measured, not assumed.

## Outcome

*To be filled after baseline eval: actual escalation rate on the real-invoice test set, escalation precision, and whether LLM confidence scores needed calibration.*

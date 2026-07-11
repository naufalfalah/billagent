# ADR-003: Data strategy — private real invoices, public synthetic invoices

- **Date:** 2026-07-09
- **Status:** Accepted

## Decision

Two strictly separated data classes:

- **Real invoices** (from actual Indonesian SMBs, obtained with owner consent): live only in `data/real/` and `data/ground_truth/`, both **gitignored from day one**. Used exclusively for local, private accuracy evaluation via `scripts/run_eval.py`. Never committed, never deployed, never logged, never used in tests or fixtures.
- **Synthetic invoices** (realistic but obviously fake): live in `data/synthetic/` and `sample_invoices/`. The only invoice data allowed in the repo, the test suite, the README, and the public Cloud Run demo.

Consequences enforced structurally:
- CI and the pytest suite must pass on a machine that has never seen a real invoice.
- Accuracy evaluation is a local script, deliberately **outside** the test suite.
- Application logs contain metadata only (invoice ID, status, confidence, timing) — never contents.
- Published metrics are aggregates only (e.g., "94% field accuracy on 40 real invoices"), never per-invoice details.

## Context

BillAgent's accuracy claims are only credible if measured on *real* messy invoices — synthetic data cannot reproduce the true variance of SMB invoice layouts, scan quality, and edge cases. But real invoices contain sensitive commercial and personal data: business names, NPWP (tax IDs), bank account numbers, prices, personal names.

The project's deliverables are inherently public: a GitHub repo and a live Cloud Run demo for recruiters. These two requirements — real-data evaluation and public artifacts — directly conflict unless separated architecturally.

For a project explicitly targeting fintech employers (GoTo Financial, Google), a data leak in the portfolio project would be a disqualifying irony.

## Alternatives considered

- **Synthetic data only** — zero privacy risk. Rejected: accuracy numbers become unfalsifiable marketing ("99% on data I generated myself"); the hardest engineering problem (layout variance) disappears, and with it the portfolio value.
- **Anonymize real invoices for publication** — redact names/NPWP/accounts and publish. Rejected: re-identification risk is real (layout + prices + dates can identify a small business), redaction is manual and error-prone at scale, and one mistake is unrecoverable once pushed to a public repo. The cost/benefit is bad when synthetic samples serve the public-demo purpose fine.
- **Keep the whole repo private** — no leak risk. Rejected: defeats the purpose; the repo *is* the portfolio.
- **Chosen: strict two-class separation** — real data gets evaluation duty, synthetic data gets publication duty, and the boundary is enforced by gitignore, test design, and logging rules rather than by ongoing human carefulness.

## Trade-offs accepted

- **Double data work:** we must both collect/label real invoices *and* author a synthetic set that mimics their layout variety. Accepted — the synthetic-generation work itself demonstrates understanding of the domain.
- **Unverifiable metrics:** outsiders cannot reproduce our accuracy numbers since the eval set is private. Mitigated by publishing the full eval methodology and the eval script itself; this is the standard trade-off in any industry work with private data.
- **Discipline dependency at the edges:** gitignore protects the repo, but nothing technically stops a real file being uploaded to the public demo by hand. Mitigated by CLAUDE.md hard rules and keeping demo storage ephemeral; risk accepted for a solo project.

## Outcome

*To be filled after M1: did any near-miss occur (real data staged/almost committed)? Did the synthetic set prove realistic enough for the demo and golden-file tests?*

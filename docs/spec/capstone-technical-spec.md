# Technical Spec — Automated Invoice Processing for SMBs

**Project codename:** Billagent
**Author:** Naufal Falah
**Stack:** Google ADK (orchestration) · Django (REST API + admin) · Cloud Run (deploy) · BigQuery (analytics, later phase)
**Status:** Draft v0.1 — for review before coding begins

---

## 1. Problem statement

Indonesian SMBs (warung, toko grosir, small distributors, F&B suppliers) spend an estimated **2–3 hours per day** manually processing incoming supplier invoices: reading the PDF/photo, checking whether the amounts and items match what was ordered (the purchase order), and re-typing the data into their bookkeeping.

The cost of this pain:
- **Time** — 2–3 hours/day is time not spent on the actual business.
- **Errors** — manual re-typing introduces mistakes in amounts, quantities, tax.
- **Late detection** — mismatches between invoice and PO (overbilling, wrong quantity) are caught late or never.

## 2. Target user persona

- **Who:** Owner or admin staff of an Indonesian SMB with 5–50 employees that receives 10–100 supplier invoices per week.
- **Context:** Uses WhatsApp + spreadsheets, maybe a basic accounting app. Not technical. Receives invoices as PDF or phone photos, in Bahasa Indonesia, in inconsistent layouts across suppliers.
- **Goal:** Spend less time on data entry, catch billing errors before paying.

## 3. Solution overview

A multi-agent system:
- **Agent A — Extractor.** Reads an incoming invoice (PDF or image), extracts structured fields: supplier, invoice number, date, line items (description, qty, unit price), subtotal, tax (PPN), total.
- **Agent B — Validator.** Matches the extracted invoice against the corresponding purchase order. Flags mismatches (price, quantity, missing/extra items, totals).
- **Agent C — Poster.** Posts the validated invoice into the accounting record (for the capstone: writes to the Django DB / exports structured data). 

**Human-in-the-loop:** any invoice below a confidence threshold, or with a validation flag, is escalated to a human review queue in the Django admin rather than posted automatically.

## 4. The hard problems (be honest about these)

These are the parts that make this a real engineering project, not a toy. Interview gold lives here.

1. **Non-uniform PDF/image extraction.** Real SMB invoices vary wildly — digital PDFs, scanned photos, different layouts per vendor, Bahasa Indonesia, sometimes handwritten notes. Agent A must degrade gracefully, not crash, and must report a confidence level.
2. **Validation without clean ground truth.** Invoice-to-PO matching is rarely black and white: partial deliveries, rounding, tax handling, slightly different item descriptions. Agent B needs tolerance rules, not exact-match.
3. **Escalation logic.** Knowing *when the system should not decide autonomously* and hand to a human. This boundary is what separates a mature agent system from a naive one.

## 5. Success metrics (measured privately on real invoices, reported in aggregate)

| Metric | How measured | Target (set honestly after baseline) |
|---|---|---|
| Time saved per invoice | Manual baseline time vs. system time on same invoices | TBD after baseline |
| Extraction accuracy | % of fields correctly extracted vs. hand-labeled ground truth | TBD |
| Validation accuracy | % of correct valid/mismatch decisions on labeled test cases | TBD |
| Escalation precision | Of escalated invoices, % that genuinely needed a human | TBD |
| System latency | p95 response time under simulated load (k6/Locust) | TBD |

> Note: metrics are measured on a **private local set of real invoices**. Only aggregate numbers are published. No real invoice is committed to the repo or deployed to the public demo.

## 6. Data strategy (privacy-critical)

- **Real invoices (private):** used locally only, to hand-label ground truth and measure accuracy. Never committed, never deployed.
- **Synthetic / anonymized invoices (public):** a small set of realistic but fake invoices used for the live demo and for `sample_invoices/` in the repo. Generated to mimic the layout variety of the real ones.
- `.gitignore` must exclude any real-data directory from day one.

## 7. Architecture (to be diagrammed)

```
[Invoice upload]
      │
      ▼
  Agent A (Extractor) ──confidence──▶ low? ──▶ [Human review queue]
      │ high
      ▼
  Agent B (Validator) ──flag?──▶ yes ──▶ [Human review queue]
      │ ok
      ▼
  Agent C (Poster) ──▶ [Django DB / accounting export]
```

## 8. Milestone 1 scope

Deliberately narrow. Ship a spine, not all three agents.

- [x] Finalize this spec + project name
- [x] Architecture diagram
- [x] Django project skeleton + Google ADK environment set up
- [ ] **Agent A only** — extract fields from ONE invoice format, end-to-end
- [ ] Deploy hello-world to Cloud Run (prove the deploy pipeline works)
- [ ] `.gitignore` + repo hygiene + README skeleton
- [ ] Architecture Decision Log: record the first 2–3 decisions (why ADK, why Django, extraction approach)

## 9. Definition of Done (Milestone 1)

Not "I pushed code." Milestone 1 is done when:
1. A teammate could clone the repo, follow the README, and run Agent A on a sample invoice.
2. Cloud Run deploy works from a clean pipeline (URL responds).
3. At least 2 architecture decisions are documented with trade-offs.
4. No real invoice data anywhere in the repo or deploy.

## 10. Open decisions (fill during build)

- **Extraction approach for Agent A:** Vertex AI / Document AI vs. LLM-based parsing vs. hybrid — decide and record in ADR.
- **ADK agent communication protocol:** how A→B→C pass data.
- **Confidence scoring method:** how Agent A quantifies its own certainty.

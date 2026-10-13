# Chevler Oracle — CLAUDE.md

Founding file, 13 Oct 2026. Read fully before any work in this repo.

**Chevler Oracle is the second-iteration Chevler ERP**: one platform organised around a job
lifecycle spine (Quote → Material availability & cost → Structure & Routing → Accepted → Job →
Production planning → Completion), command-bar-first UX, one unified quoting engine for every
product family (Litho + Cases/Wraps + Main flexo), everything editable, every change audited.
The founding documents ARE the requirements — read them before proposing anything:

1. `docs/platform-vision.md` — the paradigm (spine, master-data planes, command bar, doctrine,
   ship-in-stage-order strategy).
2. `docs/quoting-module-spec.md` — stage 1 in full: the guided flow, the one cost engine, the
   entity model, the data-reconciliation register (R1–R12), migration sources, parity & cutover,
   and §7 open decisions (check their status before building anything they gate).
3. `docs/erp-v1-harvest-report.md` — what we inherit from ERP v1 and the defect lessons that are
   now rules here.

## Session-start ritual
1. `git fetch` + `git status` — Seth works from a Mac AND a Windows box; pull if behind; commit
   AND push at session end (an unpushed commit is invisible to the other machine).
2. Read `docs/ROADMAP.md` (once it exists — create it with the first scaffolding commit) and tell
   Seth the "Now" list + anything "Waiting on Seth". Update the roadmap line in the same commit as
   the work it describes — that discipline is inherited from ERP v1 and is why its project log
   rotted and its roadmap didn't.

## The three sibling sources (read-only from here)
| Where | What it is | Rule |
|---|---|---|
| `/Users/sjr/dev/Claude/chevler-pricing-calculator` (github setheast/chevler-pricing-calculator) | the LIVE estimator suite (production, used daily) — also the requirements record (`CLAUDE.md`, `docs/production-knowledge.md` = production-rules source of record, `docs/estimator-build-pack/`) and the regression oracle (855 tests, golden fixtures, S8 extracts in `server/access-extract/`) | stays live; Oracle never breaks it; port code FROM it freely |
| `/Users/sjr/dev/Claude/chevler-erp-main` (github setheast/chevler-erp) | ERP v1 — parts donor only. NOTE: the local folder is a 22 Sep snapshot with no .git; re-verify against the live head (harvest report §6) before relying on details | no new work there, ever |
| Google Drive `/Chevler/` + `Manufacturing.accdb` (OneDrive Temp) | S7 = the Lewys 25-26 cost model (machine-rate truth, carriage-stripped); S8 = the production Access DB (die register, routings, product master) — extraction scripts live in the sandbox's `scripts/` | data sources; refresh via the scripts |

## Doctrine (non-negotiable; full text in the vision doc)
1. **Truth-first** — legacy prices are reference oracles, never acceptance targets; every parameter
   has source, date, confidence; audited Review & Save everywhere.
2. **Accuracy** — split components, everything editable, GP/NP bullet-proof (2dp unit × true count).
3. **Assumption engine** — propose with visible Standards + sources; overrides visible; checks
   advise, never silently block; recalled work re-prices exactly.
4. **Making price + delivery line** — per-1,000 is the making price; delivery separate per job.
5. **Audit as a primitive** — append-only field-level change log with rollback on every entity;
   full input+params snapshot and reproduction check on every quote; server-side authorisation
   (financials gating is NOT a frontend concern).

## Engineering rules (the v1 lessons, now law)
- ONE pure `engine/` module (zero I/O, no Express/fs/Date/fetch), unit-tested from the first
  commit; routes are thin adapters; the frontend displays, never computes money.
- Every business value is a named, seeded, audited parameter — no constants in code, no hardcoded
  machine codes; lookups by capability data.
- Real DB transactions; sequences for numbering; dated prices only.
- Golden regression against both oracles (sandbox engines + v1 calculators) with per-family
  divergence tables — reviewed, never waved through.
- GBP is the only stored currency; VAT/FX display-time only (port the sandbox's delivery/VAT/FX).
- Stack: Express/TypeScript + PostgreSQL (Railway) + React/Vite/Tailwind + vitest (spec §7.5).

## State at founding (13 Oct 2026)
Phase 0 (these documents) complete, awaiting Seth's review of the spec's §7 decisions.
**Phase 1 = the quoting engine** — do not start it until Seth green-lights the review.
No scaffolding exists yet; the repo is docs-only by design.

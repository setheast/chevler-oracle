# ERP v1 Harvest Report — what Oracle takes, what dies

*13 Oct 2026. Audited from the local snapshot `/Users/sjr/dev/Claude/chevler-erp-main` — which is a
**22 September 2026 copy with no `.git`**. The live repo is github.com/setheast/chevler-erp.
Everything here must be re-verified against the current head before Phase 1 relies on it (checklist
in §6). Two deep code/doc audits back this report; detail lives in the session record and the
quoting spec §4.*

## 1. Verdict

ERP v1 was the right ideas on a weak implementation. Its **schema and its documents are the
valuable half** — roughly 70% of what Oracle's quoting module needs was designed correctly there.
Its **code is the half the rebuild replaces**: the costing layer would have needed rewriting even
if Oracle didn't exist. Since v1 never went live (3 products, 3 customers, 1 supplier; pre-go-live
list unfinished: 2FA, staging, DPA, R2, tested restore), there is no cutover cost. v1 becomes a
read-only parts donor; no new work lands there.

## 2. What Oracle takes (the keep list)

**Schema designs** (re-created fresh in Oracle, not copied — Oracle isn't bound by v1's
"never restructure" rule; these are the *shapes* that were right):
- `machines` as capability records: print_process / output_format / max_colours / width & gsm
  envelopes driving route candidacy as DATA; setup/changeover/warm-up times; per-datum confidence.
- `machine_run_standards` with `efficiency_assumption_pct`, `productivity_factor` (the OI-1
  design: factors updated monthly, costs auto-correct) and `confidence` (estimate→observed→verified)
  + the Calibration page concept.
- Materials as grade (type+gsm+colour) → width rows → `material_suppliers` with dated,
  preferred-flagged prices; `material_stock_items` (individual reels) separate from the catalogue.
- Routing as data: `product_routing_steps` with forming/packing parameters, step types as
  configuration, tool links; `product_structures` (BOM — unused in v1, becomes the S of S&R).
- `tooling_register` + `tooling_machine_compatibility` (schema matched our independent S8 design).
- Quotes with route_comparison / recommended_route / selected_route + override author & timestamp.
- Relational permissions (role_permissions; gap #4's per-user grants + discount limits as the
  target), `audit_log` shape (field-level old/new).

**Documents (binding requirements for Oracle):**
- `docs/QUOTING_GAP_LIST_SEPTEMBER_2026.md` — the six-gap acceptance checklist (multi-route ✓built,
  confidence ✓built, activity log, permissions, quoting UX, pilot validation).
- `docs/CC-BRIEF-pricing-structure.md` — list prices, volume tiers, contract prices, the
  calculation order (§4.8), margin floors (§4.9), approval ladder, Sage export.
- `docs/Chevler-Quoting-Platform-Report.md` — the Sep clean-sheet review; notably it already
  recommends ONE time-based engine for every family and rates derived from annual inputs, both of
  which v1 then didn't implement and Oracle does.
- The productivity-factor procedure set (baseline extraction → Lewys validation → monthly review).
- `docs/ROADMAP.md` discipline: one always-true "what's next" file updated in the same commit as
  the work — Oracle adopts the practice from day 0.

**Data (`docs/harvest/`, flagged unvalidated, 16 Sep):** tooling-cases.json (612 case sizes,
601 die refs — merges with the S8 register, which is primary), carriage-main-estimator.json
(UK £50/pallet, Ireland/Scotland £65 — a seed for the delivery line), materials/machines/factory
harvests per estimator, machine-run-standards-ESTIMATES.json. Plus v1's migration seeds as a
cross-check against S5/S7.

**Frontend components worth porting:** RouteComparisonPanel, spec summary chips, utilisation
colours, ConfidenceBadge, SearchableDropdown/Modal/SlideOutPanel primitives.

**Infrastructure choices that were fine:** Express/TS + Postgres (Railway) + React/Vite/Tailwind;
JWT auth; sequential SQL migrations; GBP-only storage; EU-West hosting.

## 3. What dies (the kill list), each with its lesson

| v1 defect | Lesson baked into Oracle |
|---|---|
| All costing inline in a 1,310-line route file; two divergent cost models (printed double-counts labour — hourly rate AND a per-kg labour pool; cases omits the FOC); two more copies of routing maths in routing.ts/jobs.ts | ONE pure engine module, zero I/O, unit-tested; routes are thin adapters |
| Hardcoded constants: 250 startup sheets, 432mm press length, 500kg/hr slitting, 36 strokes/min, 7000 sheets/hr fallback; hardcoded machine codes (SLITTER/SHEETER/PACKING); ink always looked up as litho | every business value a named, seeded, audited parameter; lookups by capability |
| Browser-computed prices saved to the DB; Cases configure logic in the React page | server computes everything; frontend displays (v1's own rule, enforced this time) |
| `parameter_snapshot` stores only the overhead profile — quotes can't be re-priced | full input + params snapshot with a reproduction check on every quote (sandbox pattern) |
| Quote numbers from COUNT(*)+1 (collision-prone); BEGIN/COMMIT on different pool connections | sequences + real transactions |
| Zero tests | test runner from the first commit; golden regression against both oracles |
| showFinancials gated in the frontend only (API still returns gpPct/npPct) | authorisation enforced server-side |
| VAT hardcoded ×3 in the frontend; no delivery/Incoterm/FX in quoting despite tables existing | the sandbox's delivery/VAT/FX system migrates whole |
| Tooling never costed (`tooling_cost` always 0) | dies/origination priced from the register |
| `estimator_type` CHECK widened four times; retired category values still offered by the Admin dropdown (can make categories unquotable) | family config as data, not CHECK constraints |
| Project log rotted as a status document | ROADMAP discipline: updated in the same commit |

## 4. Open work v1 leaves behind (tracked into Oracle's register)

- OI-1 productivity factors: all 24 run standards at 1.00 / `estimate` — the baseline exercise
  (Hannah → Lewys) never ran; Oracle's completion stage is the long-term fix, the pilot runs the
  short-term one (= gap #6).
- Material prices 539 days old (79 of 80 dated 1 Apr 2025); stock code 103563 duplicated with
  conflicting prices in the harvest.
- Rates seeded from the pre-S7 carriage-inclusive set incl. flat £40.64 on all formers (see spec
  §4 R1) — superseded by S7.
- Crimped-case tooling map incomplete (die refs unlinked; ~25% of sizes have a machine) — the S8
  register covers most of this; the residue goes to Tim.
- Sharon's read-only tooling screen never shipped (the table + API existed; ROADMAP's "no table
  exists" note was wrong). Oracle's Tools plane covers it; if she needs it sooner, a one-page
  screen on v1's existing `GET /api/config/tooling` is a sandbox-sized favour, not an Oracle task.
- Pricing-structure discovery (`docs/pricing-structure/01-gap-analysis.md`) was mandated but never
  written — Oracle's spec §7.6 decides where that workstream now lives.
- Pre-go-live list (2FA, staging, DPA, R2, restore test) — re-scoped to Oracle before ITS go-live.

## 5. Stack recommendation for Oracle

Keep v1's choices — Express/TypeScript + PostgreSQL (Railway) + React/Vite/Tailwind — they were
never the problem, and they match the sandbox's language so its TypeScript modules port directly.
Add from day 0: vitest, a pure `engine/` package, transactional data layer, server-side authz,
and the platform change-log primitive. (Decision §7.5 in the spec.)

## 6. Re-verification checklist (needs the CURRENT clone — the snapshot is 22 Sep)

- [ ] `ls backend/migrations/` — anything beyond 039? (new schema to harvest)
- [ ] `docs/ROADMAP.md` — movement in Now/Waiting/Done since 22 Sep (esp. pricing Phase A, the two
      bugs at the top: margin-before-discount and the estimator-type dropdown)
- [ ] `docs/QUOTING_GAP_LIST_SEPTEMBER_2026.md` — any gaps ticked (Seth's demos of #1/#2 were
      pending)
- [ ] Whether the pricing-structure discovery docs now exist
- [ ] Any production/live-data change that alters "not live, no cutover cost"

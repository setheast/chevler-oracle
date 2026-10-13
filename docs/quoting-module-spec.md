# Chevler Oracle — Quoting Module Specification (v1 draft)

*Drafted 13 Oct 2026, distilled from the estimator sandbox (`chevler-pricing-calculator`), the
ERP v1 audit (`erp-v1-harvest-report.md`), and the October 2026 data findings (S7/S8). For Seth's
review; the open decisions are in §7. The sandbox's `docs/production-knowledge.md` remains the
production-rules source of record until its contents migrate here.*

---

## 1. Doctrine

- **Truth-first, not parity-first.** The module must NOT reproduce today's quotes. The spreadsheets
  and the sandbox's parity engines are reference oracles used to quantify and explain divergence —
  never acceptance targets. Every rate/speed/waste parameter carries a source, a date and a
  confidence level (estimate → observed → verified, per ERP v1's design), and is editable only
  through an audited Review & Save.
- **Accuracy doctrine.** Anything grouped splits into components; everything is editable; GP/NP are
  bullet-proof: quoted unit price is rounded to 2dp and every total = unit × true unit count
  (the sandbox's `quote-figures` invariants, including currency rows).
- **The assumption engine.** From minimal inputs the module proposes everything it can, each
  proposal showing its Standard + source, each deviation a visible override, with advisory checks
  (error / attention / info) that never block silently and always suggest the way forward.
  Recalled quotes and saved products re-price exactly (missing new fields = legacy behaviour).
- **Making price + delivery line (§11 decision, 11 Oct 2026).** The per-1,000 is the MAKING price
  and GP/NP are measured on it. Delivery is a separate per-job line: interim = the delivery
  option's flat % (EXW 0 etc., Incoterms + export VAT as built in the sandbox); target = real
  pallets × PlasPac tariff with an explicit delivery-margin % parameter.
- **Commercial layer per ERP v1's pricing brief** (`CC-BRIEF-pricing-structure.md`): the
  calculation order (unit cost → list price → tier discount → customer discount → contract price →
  manual adjustment → final price + net margin → approval), per-category margin floors measured net
  of discounts, and the approval ladder (Sales −2% → Director −2% → MD; below floor always top).
  The quoting module emits the *unit cost* and *making price* that feed this chain; list-price
  machinery is the pricing workstream's scope, designed-for here, not built first.

## 2. The guided flow

One wizard for every product family. Each step is a collapsible section: complete → collapses to a
one-line summary → next expands. Any collapsed step reopens for edits; edits ripple forward with
the affected steps flagged. The command bar can pre-answer steps 0–2 (it parses customer, product
code, dims, material, gsm, quantity); everything it understood is shown and editable.

| Step | Question | Auto-completion sources | Checks (examples) |
|---|---|---|---|
| **0. Who & what history** | Customer (tier, contracts, currency), or anonymous. Repeat? | saved products, quote/job history, similar-products routing history (S8) | stale-price flag on re-quote; Draft/Do-Not-Use spec status |
| **1. Product family** | Printed Sheets · Sheets · Reels · Discs/Ovals · Wraps · Cases · Liners | — (one click or parsed) | family quotable? (rates signed off) |
| **2. Size & material** | dims; grade = type + gsm + colour | material catalogue (grade→widths→dated preferred price); blank/flat-blank guides | no priced width; gsm outside machine envelope; stale price |
| **3. Tool** | needed? have one? | **die auto-fit** from the register (dims → die, either orientation); nearest dies + "what jobs ran them" when no exact; new-tool amortisation when none | tool-lane-width; tool drift vs register; heads vs machine class |
| **4. Machine / route** | which machine(s) | tool→machine compatibility; for printed work **route candidacy** (every capable machine priced, cheapest recommended, override recorded — ERP v1 gap #1, kept); multi-machine running standards | speed vs quoting bands (rated warns); printed-never-Slitter; machine can't take gsm/width |
| **5. Piece & conversion** | web/sheet, piece, lanes, QPA, trim, conversion | per-lane from die web ÷ heads; slit rule; QPA = blank ÷ reel m / sheet fit; press-time conversion (S6) with per-job speed override; die's feed waste | QPA vs rule; wasteful cut; reel-in-mm; util colours |
| **6. Quantities & margins** | breaks, target NP, show-in-pricing | customer's usual quantities; MOQ from reel maths | margin floor; order weight & logistics (pallets when tariff lands) |
| **7. Delivery** | option (Incoterm), destination, currency | customer default; export → VAT zero-rated; live ECB FX (48h validity) | UK-option-abroad contradiction; unrecognised destination; carriage override conflict |
| **8. Outputs** | customer quote + internal blueprint + **S&R** | — | checks-at-issue recorded on the quote |

**Structure & Routing (S&R)** is generated with the quote: the product structure (materials with
QPA/waste) and the ordered routing (machines, tools, speeds, setup, crew) exactly as priced — the
document production runs to when the quote becomes a job. It is a projection of the engine's
internals, not a separate entry.

**Internal blueprint** (from the sandbox, kept whole): every input as entered with overrides
flagged, every rate from the snapshot, the customer quote as issued reproduced verbatim, order
weights, checks at issue, the legacy-comparison table while families bed in, and a reproduction
check proving the stored snapshot re-prices to the quoted figures.

**Ported from ERP v1's frontend**: RouteComparisonPanel (per-route columns, cheapest-per-qty
highlighting, confidence badges, recorded override), spec summary chips, utilisation colours.
**New**: the stepper shell, the delivery/VAT/currency step, the outputs step, the command bar.
**Retired**: machine parameters on the Cases configure screen (the engine + register supply them).

**Post-founding rulings (13 Oct 2026, decided and shipped in the live sandbox — binding here):**

1. **A saved product is the SPEC only.** Quantities, margins and pricing are never stored on a
   product — they belong to the quote (recalled from a saved quote). The product code is OPTIONAL
   at save time (often unknown yet; blank codes never collide; the name stands in until the code
   exists). Step 0's recall and step 6's prefills follow this split: spec from the product,
   commercials from the customer's quote/job history.
2. **The assumption engine proposes what we BUY TODAY, not what history ran.** Mill ("mother")
   reels carry stock-file liveness (live / dormant / obsolete): the sandbox panel reads "from
   1040mm mother (now obsolete — live is 1065mm)" and its Use button targets the LIVE reel; the
   blank guide confirms the mother against the selected grade's live mill reel (Singleproof 90 →
   the 1065mm FSC line). Oracle's Materials plane therefore needs per-stock liveness, and step 2
   confirms the mother from the grade, not from a default.
3. **The historic BOM + routing joins are step 5's autocompletion goldmine** (sandbox
   production-knowledge §14): TblBOM gives each product's real material (type/colour/gsm, slit
   width, reel length), its QPA in the estimator's own convention, and — one BOM level up — the
   mother reel plus the slitting/sheeting waste (2.5% reels / 5% sheets); TblShopOrdHistory gives
   last-order recency; TblRouting's run factor is output/hr in 1,000s with actual cycles/min =
   output ÷ 60 ÷ nest (the DB's own TblRoutingsCycleCheck relationship). All extracted and shipped
   (`similar-products.json`, regenerable by script).
4. **Oblong Reel Fed blank rule, evidenced from 65 historic BOMs**: blank along the web =
   D2 + 2·D3 + 6–8mm trim (the QPA driver); the slit WIDTH is die-specific (D1 + 2·D3 plus
   10–32mm) — the die or a matched historic product supplies the width, the formula supplies the
   QPA, and the width never changes the QPA. (R13 tracks Stuart's confirmation.)
5. **Both estimator forms gained a "Clear all"** (fresh-start remount) — carry the affordance into
   the stepper shell.

## 3. One cost engine

A pure, tested, zero-I/O module (`engine/` — the sandbox discipline: no Express, no fs, no Date,
params in / cost breakdown out), replacing ERP v1's `calcPrintedSheet` + `calcFormingCase` and the
sandbox's seven engines. One pipeline for every family:

```
spec
 → candidate routes            capability data (process, output, widths, colours, gsm) +
                               tool compatibility; NEVER hardcoded machine codes
 → per route: ordered steps    from routing rules + the spec (print, slit/sheet, form, guillotine,
                               punch, pack) — steps are data, incl. composition (press+Sheeter)
 → each step costed by TIME    time = work ÷ (standard speed × efficiency × productivity factor)
                               rate = the machine's S7 carriage-stripped £/hr (machine + labour,
                               crew-aware); setup = die-change/setup standards × MR rate;
                               waste applied per step, named (startup, running, trim, material)
 → material                    grade → width (per-lane/web maths from the die) → dated preferred
                               price; weight = the single source for packing/delivery/logistics
 → packing                     real quantities when known, £/kg basis otherwise
 → overheads                   named pots recovered on a declared basis (see §4 — NOT silently
                               value-%), FOC, admin
 → margin                      target NP on the MAKING price
 → outputs                     making price per 1,000/reel · delivery line (separate) · tooling &
                               origination (amortised or one-off) · S&R · full snapshot
```

Family configuration is data (which steps, which guides, which checks) — not per-family engines.
**Litho is a routing through this same engine** (LITHO work centre at the S7 rate + guillotine +
packing): the Ecoluxe £/kg-annual-pot model is retired, with the difference quantified against the
live Ecoluxe during parallel running, never silently.

Unifications this forces (v1's divergences, resolved once): one labour treatment (in the machine
rate via crew — never also a per-kg pool: v1's printed path double-counts), one FOC treatment
(always applied), ink by print process (v1 always charged litho ink), tooling/origination actually
costed, every constant a parameter (v1's 250 sheets / 432mm / 500kg/hr / 36 strokes/min / 7000
sheets/hr all become named, seeded, editable values).

**Testing contract**: unit tests on every pipeline stage; golden regression against BOTH oracles —
the sandbox's engines (its 447 Cases + Main fixtures re-expressed as expected-divergence tables:
each delta explained by a named rate/method change) and ERP v1's calculators for its three seeded
products. No family cuts over until its divergence table is reviewed and its rates signed off.

## 4. Entity model & the data reconciliation register

Entities (schema re-designed fresh — Oracle is not bound by v1's "never restructure" rule — but
deliberately close to v1's shapes, which were right): Customers · Materials/Widths/SupplierPrices ·
Stock (reels) · Machines (capabilities + rates + run standards + multi-machine standards) ·
Tools/Dies · Products (incl. saved-product semantics: status, history, facets) · Quotes (thread
stage 1: line items, route comparison, snapshot, checks, S&R) · Parameters (versioned, audited) ·
Users/Permissions (per-user grants + discount limits — v1 gap #4) · the **platform change log**
(append-only, field-level, rollback, view-logging — v1 gap #3, as infrastructure).

**The rate story — three lineages become one.** This is the register of every known conflict; each
must be resolved (owner named) before the affected family cuts over:

| # | Conflict | Position at founding | Owner / gate |
|---|---|---|---|
| R1 | Machine hourly rates: ERP v1 seeds = pre-S7 carriage-INCLUSIVE (LITHO 46.83, WIDE-REELER 71.11, formers flat 40.64 — the misapplied RF1–7 rate) vs **S7 carriage-stripped per-machine** (RF1–7 36.98, 8/10/11REEL 77.41, 9REEL 104.72, Wide 58.30, Narrow 51.17…) | **S7 stripped is the truth** (fulfilment is per-job, never in the hour) | Lewys cross-check; Stuart Q2 reply |
| R2 | 11REEL has no modelled rate | 8REEL-class proxy, flagged | Lewys to model |
| R3 | Base labour £19.33 (S7, Mar) vs £20.29 (Lewys, Oct) | unresolved | Lewys |
| R4 | Efficiency/productivity: 85% assumed vs ~70% observed (v1's OI-1) — and S7's hours assume 85% | adopt v1's `productivity_factor` design; seed from the baseline exercise | Hannah extract → Lewys validate (v1's procedure docs) |
| R5 | Overhead pots: v1 profile (labour £117,467 / 80,000kg / admin 15.5%) vs sandbox factory params vs S7 pots | rebase from S7 + current accounts at founding; versioned profile | Lewys |
| R6 | Material prices 539 days old in v1; sandbox catalogue has reconciliation candidates | refresh at founding; dated prices mandatory thereafter. **13 Oct:** mill reels also need live/dormant/obsolete status from the stock file — SP 90's historic 1040mm mother is OBSOLETE, the live buy is the 1065mm FSC line; one price-list row per grade hides this | Lewys/purchasing |
| R7 | Die data: S8 register (385 dies, rated speeds, feed waste, heads) vs v1 harvest tooling-cases.json (612 sizes, 601 unlinked die refs) | S8 is primary; harvest fills gaps; measured webs still to collect | Tim (register owner) |
| R8 | Speeds: die RATED vs Stuart's quoting bands | rated = Standard, bands warn (decided 12 Oct) — bands maintained as production rules | Stuart |
| R9 | CHEVS ×(2÷1.5) template quirk vs S7 rate (~12% if it only means crew) | quirk NOT carried into Oracle; crew is explicit | Lewys/Stuart |
| R10 | Delivery: flat % vs real fulfilment | §11 architecture native; % interim. **13 Oct — call-off orders are a first-class case** (e.g. 1m/year over ~6 drops, common): a flat % cannot see the drop count, so delivery must price drops × (pallets/drop × zone rate) + storage dwell, with the batching question (one run + storage vs several runs + set-ups) surfaced. v1's harvest carriage seeds (UK £50/pallet, IRE/SCO £65) are the interim per-drop numbers | Seth → PlasPac tariff |
| R11 | Packing £/kg seeds unconfirmed | carried, flagged | Stuart |
| R12 | Overhead recovery basis (admin/S&A as value-%) | carried at founding, flagged as the post-delivery follow-up (same critique as §11) | Seth/Lewys |
| R13 | Oblong RF blank rule: evidenced from 65 historic BOMs (along-web = D2 + 2·D3 + 6–8mm trim; slit width die-specific) but not yet confirmed by production | shipped as the sandbox's rule 13 Oct; confirm | Stuart |

## 5. What migrates from where

**From the sandbox** (patterns + data, largely portable TypeScript):
engine maths and the golden-fixture discipline (as regression oracles); `production-rules` +
`estimate-checks` (the checks and guides); delivery/Incoterms/export-VAT/live-FX;
`quote-figures` invariants; blueprint internal PDFs + customer PDFs; audited params stores with
what-changed history; saved products (status/history/facets/filters; spec-only + optional code
since 13 Oct); the S8 extracts and extraction scripts — die register, punch dies, and the
similar-products set enriched 13 Oct with each product's BOM material (type/colour/gsm, slit
width, reel length, QPA, waste), its mother reel + stock-file liveness + today's live mill reel,
last order + order count, and routing semantics (run factor in 1,000s/hr; cycles = output ÷ 60 ÷
nest); weights.

**From ERP v1**: the schema designs (§4); route candidacy + comparison + override recording;
confidence/calibration; the permissions model; the pricing-brief machinery (designed-for);
the harvest datasets; the docs corpus as requirements (gap list, platform report).

**From the Drive/Access sources**: S7 (Lewys model) as the rate truth; Manufacturing.accdb via the
extraction scripts for refreshes.

## 6. Parity & cutover

1. Engine built family-by-family with its divergence table against the sandbox oracle.
2. **Parallel running**: real quotes priced in both systems; the Oracle quote shows the legacy
   figure alongside (the sandbox's comparison-toggle pattern) until the family is signed off.
3. Cutover order (proposed, §7 to confirm): Cases/Wraps → Main (sheets/reels/discs/liners) →
   Printed Sheets spec-first → **Ecoluxe last** (it stays live and untouched throughout).
4. Acceptance: ERP v1's six quoting gaps (as the checklist), reconciliation with the pricing-brief
   Phase-1 workbook, and a per-family acceptance pass (fresh jobs priced in both, divergence
   explained and approved).

## 7. Open decisions for Seth's review

1. Cutover order above — confirm or reorder.
2. Rate sign-off sequencing (which of R1–R12 gate which family).
3. Sandbox feature-freeze timing (proposal: freeze at Oracle Phase-1 start except bug fixes and
   data corrections).
4. Pilot families for parallel running (v1's pricing brief wants four, one containing EH400).
5. Stack confirmation for Oracle (proposal in harvest report §5: keep v1's Express/TS + Postgres +
   React/Vite, add a test runner from day 0).
6. Whether the pricing-structure workstream (list prices/tiers/approvals) builds in Oracle stage 1
   or stage 1.5 (proposal: schema designed-for now, built after quoting parallel-runs cleanly).

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
   D2 + 2·D3 + 6–8mm trim (the QPA driver); ~~the slit WIDTH is die-specific (D1 + 2·D3 plus
   10–32mm) — the die or a matched historic product supplies the width~~ **[width superseded
   14 Oct — ruling 10 below: it comes from the mother reel]**, the formula supplies the
   QPA, and the width never changes the QPA. (R13 tracks Stuart's confirmation.)
5. **Both estimator forms gained a "Clear all"** (fresh-start remount) — carry the affordance into
   the stepper shell.

**Post-founding rulings (14 Oct 2026, decided and shipped in the live sandbox — binding here).**
Full write-up with sources, code to port and open items: `docs/sandbox-handovers/2026-10-14.md`
(copy of the sandbox's `docs/oracle-handover-2026-10-14.md`); production detail in the sandbox's
`docs/production-knowledge.md` §§15–16.

6. **The Access record is the product master.** Any of the 6,659 Access products is searchable and
   loads into the quote as a **Draft** with every assumption shown as a note (sandbox: 2,015 of
   2,026 cases/wraps load; 1,665 fully pre-filled). Paper resolves by stock code → its mill-reel
   (BOM parent) code → a type/colour name table — the table and the template rules are ADMIN DATA
   here, not code. Step 0 searches products AND the Access master; Products gain an import-as-Draft
   path. Sheets/discs/liners/reels were not yet mapped in the sandbox (wider material gap).
7. **The pallet build lives on the outer CARTON, not the product.** `TblProductCode`'s Units/Layer ×
   Layers is a legacy pair (~140 products); the real build is the carton's `TblMaterials` UK/EU
   Units/Layer × Layers, reached through the BOM (3,203 products). **Carton is an entity** (code,
   outer code, flute, colour, W×L×D, UK + EU builds); products link to it. Pallets per job =
   packs ÷ packs per pallet — the input R10's per-job delivery needs.
8. **A product is not tied to a route.** WBIG03 ran VP-16 from cut sheets but is ~30% cheaper on
   6REEL from a slit reel at a realistic speed. The sandbox moves a job between route templates by
   hand (`route-move.ts`: machine → route, the product-vs-route split of inputs) and records both
   routes' prices on the quote. Here this is native: every capable route priced and compared
   (§3 "candidate routes" + v1's RouteComparisonPanel), cheapest recommended, override recorded.
9. **Multi-head reel-fed dies cut STAGGERED (Stuart).** One slit reel feeds every head: piece width
   = the die's web (YF855: 263mm for a 133mm blank on 2 heads) and **QPA = blank ÷ heads ÷ reel
   metres**. The earlier per-lane reading (web ÷ heads, QPA ÷ reel only) gave the same weight but
   half the MOQ and not the reel actually bought. §3's material line follows this.
10. **Oblong slit width comes from the MOTHER reel**, like round cases: lanes = floor(mother ÷
    (D1 + 2·D3 + 10mm minimum spare)); width = floor(mother ÷ lanes). 49 of 61 oblong reel-fed
    products match to 1mm (86% of their orders); the minimum spare is a parameter. (R13.)
11. **8RF is dismantled — everything runs on 10RF at the same costs (Stuart).** Retire 8REEL as a
    work centre; migrate its dies, routings and order notes to 10REEL (R16).
12. **Speeds from Access are unverified** (Seth: "the default machine speeds are all wrong"): the
    WBIG03 6-7REEL die was recorded at 4/min (copied from its VP-16 die; now 12 on live). Every
    imported speed carries a confidence level and starts unverified (R14). Speed decides route
    choice (WBIG03: +11% at 4/min, −30% at 12/min).
13. **Products carry production notes** (`orderNotes`, 2,638 products — e.g. "MUST ONLY RUN ON 8RF",
    machine set-ups like "Speed 9, Heaters 4-4, Plunger 5"): typed (spec / set-up / customer) and
    shown on quotes, jobs and works orders.
14. **The internal quote lists every field and guide in screen order** (blank & reel guide, QPA
    working, speed steer, standard vs used, die, tooling/origination workings, layout cards,
    cheaper configurations, quote-time snapshots of advisory panels, the route move, a catch-all
    so no stored input is dropped). Here the form layout and the record layout should come from
    ONE definition so they cannot drift (the sandbox keeps them in step by hand:
    `quote-workings-screen.ts`).

**Post-founding rulings (15 Oct 2026 — Lewys, finance; binding here).** Write-up:
`docs/sandbox-handovers/2026-10-15.md`; detail in the sandbox's `production-knowledge.md` §17.

15. **Base labour is £20.29/hr** (fully loaded grade 3, with Lewys's headroom), not S7's £19.33;
    other grades (£25.69, £23.91) still to confirm.
16. **CHEVS crew includes a shared packer** (one per three Chevlers) — the ×(2÷1.5) is real crew,
    so Oracle's explicit crew for a CHEVS station must include it.
17. **One efficiency per machine, from the productivity reports / timesheets** (10RF ≈ 93%) —
    the single productivity factor replaces the model's 85% and the quotes' 75%.
18. **S&A stays 15.5%; its recovery method is open** — machine-hour rates (Lewys) vs a fixed
    per-order charge + small value-linked % (Seth). Both agree a flat % overcharges big jobs.
19. **Delivery line = PlasPac rate cards (pallets × zone) + ~15% margin**; the 4.5% pot also
    holds storage, dispatch, rebates and charged shipments, which need a home when it retires.
20. **A bad-debt allowance of £50k/year** is a named parameter (sandbox: 0.82% of price = £50k ÷ £6.1M forecast sales),
    deducted in NP, excluded from legacy baselines.
21. **Every pricing change is watched against a frozen legacy baseline** — the per-quote drift
    view (sandbox "Compare with workbook price") carries into parallel running (§6).

**Post-founding rulings (16–26 Oct 2026 — Seth, Lewys, James, Stuart's S9 list; binding here).**
Write-up: `docs/sandbox-handovers/2026-10-26.md`; detail in the sandbox's `production-knowledge.md`
§19–§21.

22. **Engineering is inside the machine-hour rate — no engineer premium anywhere** (Lewys): make-ready
    is costed at the machine's own rate. The workbook's machine + £29.50 engineer make-ready is
    superseded (R17).
23. **Crew by head count**: twin head = ONE packer, three heads = TWO (3-head running is normally
    counter-productive). 9REEL is a triple head (2 operators); 10/11REEL are twin-head Snyders.
    Machines carry heads + crew explicitly; crew follows the head configuration fitted.
24. **No finance on any reel-fed machine; depreciation belongs in the rate.** Only the flow wrapper
    carries finance.
25. **Machine rates are a CALCULATION, not typed numbers**: budget line × recovery share (per-machine
    drivers — m², running hours, kWh, crew, history split) ÷ running hours + crew × base labour, held
    in the calibration module from Lewys's budget table (R18). Each recovery basis is an explicit,
    owned choice (several are historical: general hands on reel formers only, depreciation on space,
    "history" splits, site expenses on unmeasured waste tonnage).
26. **Idle capacity is a pricing-policy question, not a rate**: 11REEL (idle since its customer was
    lost) shows full absorption overpricing exactly the work one would load onto spare capacity.
    Decide with Lewys how quotes price onto an idle machine.
27. **Historical machines** (VP-16 decommissioned; 8RF dismantled) keep their rates for re-pricing
    old quotes but are never routable; quotes on them raise an advisory check.
28. **Tools carry a data-confidence level; reference tools prove existence only.** A die known only
    from a low-confidence list (Stuart's 2013 crimped-case catalogue, 491 sizes) is searchable but can
    never be fitted or drive a routing until enriched (fits-machines, heads, cycles). A blank
    fits-machines must never mean "fits any" for such a tool.
29. **Resilience**: daily backups and continued access if the internet goes down are requirements
    (today's on-premises server provides both). One estimator, one source of truth replacing Access
    + Excel + the three calculators.

**Post-founding rulings (27 Oct 2026 — Hannah, Operations Manager, via Seth; binding here).**
Write-up: `docs/sandbox-handovers/2026-10-27.md`; detail in the sandbox's `production-knowledge.md` §22.

30. **The whole mother reel consumed is costed, and it decides the reel-fed route.**
    - **The case (Hannah):** 51 × 38 round cases in GP Kraft 50 go on RF2 (single head), not 10RF (twin head). Kraft comes only as a 960mm mother reel; 10RF's 263mm web cuts 3 from it, leaving 171mm (17.8%) offcut. The 1–5REEL die cuts 7 × 137mm lanes (0.1%).
    - **The rule:** route costing charges material + ink + conversion on the offcut too, because the full mother runs through the press/slitter. Otherwise twin-head routes look cheaper than they are on a reel that doesn't divide.
    - On white GP the 1050 reel slits 4 × 263 cleanly, so the same product is right on 10RF.
31. **Slitting tolerance is a calibration parameter.** Lanes = floor((mother + tolerance) ÷ slit width), with the tolerance seeded at 3mm from Access: 132 of 1,161 reel products are slit 1–3mm over nominal, and the median offcut is 0%. Offcut above a flag share (seeded 5%) triggers a search for a cleaner-slitting die on a running former (R19).
32. **A quote's material is a stock line (Materials row), never a name + gsm key.**
    - The line fixes the price (the workbooks priced the first VLOOKUP match) and, for a mill reel, the mother reel width that drives slitting.
    - Labels say what the line is: `102404 63gsm Black Market Paper (1,200 Mill Reel)`, `(525 × 750 Sheet)`, `(285 × 4,000m Slit Reel)`.
33. **A multi-head die's web width is measured, never derived.** Without it the die can't be compared on mother-reel waste (the 51×38 10REEL 2-up die's 263mm is known only from product BOMs) — R7.
34. **Packing (nest / sleeve / pack / outer) belongs to the product + route, not the size.**
    - 51×38 50gsm GP: twin-head reel formers pack 20 / 660 / 19,800 (91% of 157m units sold in 2 years); the single-head ones have no standard.
    - Across all round cases the leading nest differs by machine in 35 of 44 multi-machine combinations; paper colour doesn't matter, grade + gsm do.
    - A new product's packing is a suggestion with a certainty: recorded standards first, then history by size + grade/gsm + machine group.
35. **Packing Standards are master data with source + confidence:** who said it, when, Confirmed (beats history, 100%) or Provisional (with its own certainty %) — doctrine 1 applied to packing (R21).
36. **Every proposed Standard shows how sure it is.** A % only where it's computed from frequency; otherwise a level — Confirmed (named ruling / measurement), Evidenced (our data, unconfirmed), Assumed (default). Never a made-up %. Displayed in the flow and printed on the blueprint (doctrine 3).
37. **The print press is a costed route decision** (Stuart, S11).
    - Narrow Reeler + stereo (~£250/colour, one per colour) vs Wide Reeler + cylinder (~£1,250/colour); a solid print needs a cylinder.
    - Narrow max 526mm, 516 stocked; a wider mother is slit to equal pieces first (a charged Slitter pass); stereo lanes are equal.
    - Choose the cheapest all-in, origination included, at the quoted quantities (51×38 kraft: narrow to ~700k–1m).
    - Press time is costed on the press WEB. Origination is a per-press, per-colour item with an "already held" state per design → a Print Tools register (seed: Access `TblPrintTools`).
38. **Every conversion pass is costed on time × rate, never a flat £/kg** — flat rates mis-rank routes (the Slitter's £0.41/kg is ~10× its press-time cost; R22).
39. **Seth's operating model:**
    - The user enters size, material, print, quantity; the system decides former, press, layout, packing and origination as Stuart does, recording each decision with its certainty on the blueprint.
    - The detailed fields live in an **Admin mode**.
    - **Production rules are admin-maintained, audited parameters**, so knowledge from production conversations is entered without a release (Packing Standards are the first).
40. **The system chooses the route — machine × press × layout.**
    - Every running former with a ready die for the size is combined with each press/layout, priced all-in at every quantity through the one engine. The cheapest at the most quantities is applied; every alternative is recorded with its price or the reason it was excluded.
    - Sandbox: Reel Fed (2RF vs 10RF; 51×38 kraft → a single-head former on the narrow reeler).
41. **Confirmed beats evidenced for Standards.** A confirmed production rule (Stuart's speed band) overrides unverified imported data (an Access die's rated speed) when that data is higher, and both are shown. An unmeasured twin-head die web is evidenced from product history until measured.
42. **A blank input never prices as free.** Required inputs are enforced before pricing (a blank nest priced the machine at £0 in the sandbox), and a zero-cost machine is an error.
**Post-founding rulings (28 Oct 2026 — Seth; binding here).** Write-up: `docs/sandbox-handovers/2026-10-28.md`; sandbox `production-knowledge.md` §26.

43. **Print plates by press:** Reel-Sheet and the Narrow Reeler take a stereo or a cylinder; the Wide Reeler a cylinder only. A solid print needs a cylinder (stereo joint line) on any press that takes one, which refines ruling 37. Origination is priced per press × plate × colour; the Print Tools register holds plate type per press, and each job records the plate chosen.
44. **An un-costed option is never compared as free:** a route whose plate (or any input to its cost) has no price is excluded from the comparison, with the reason, until the price is set.
45. **Litho plates:** the Litho Printer takes a plate, £50 per colour (Seth, 28 Oct 2026). Print Tools plate types are stereo / cylinder / plate. Reconcile with the Ecoluxe calculator's existing origination charge before porting litho (R24).
46. **A printed white job uses GP Opaque** (Stuart, 28 Oct 2026; sandbox §27). A material rule — print type × colour → material grade — applied by the assumption engine when the paper is chosen and shown with its certainty. Plain white GP is still bought for unprinted work.
47. **Pack quantities on a customer quote only when certain:** from a Confirmed packing standard or the product record (a job run before, or the same packing as something the customer already buys). Suggestions stay internal; an estimate is given on request.
48. **Speed bands have a small-blank end:** speed standards are per machine group × blank band, maintained by production (a 31 × 23, 83mm blank, runs ~60–70/min against the 55 band for blanks to 100mm). R14/R8 still apply.
   - *Sandbox build (28 Oct 2026):* rulings 46–48 are built in the sandbox. The paper swap is automatic with an undo and is recorded per quote. The packing bar is a setting (seed 90%); 100% comes from packing as run on a product, a Confirmed standard or the estimator's confirmation with a note, and the decision is stored on the quote. Each speed band carries a status: a provisional band (to 85mm → 60) is Evidenced, and only confirmed ceilings block. "Customer already buys this packing" should come from customer-linked product history in Oracle, not a tick.
49. **The route optimiser tries every reel of the paper, and a reel at the die's width runs as bought** (Seth, 28 Oct 2026; sandbox §28 — YF944 runs 9REEL off a 285mm reel bought at width, no slitting). A plain reel-fed routing is chosen across every stocked reel of the grade + weight (slit, or no slitting operation at all when the purchased width is the die's web ± 3mm). The recommendation is never dearer than the routing already on the job. A single-head reel may exceed the blank by at most 20% (provisional, Stuart to confirm).

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
 → material                    grade → width (the die's web — one staggered reel feeds all heads;
                               QPA = blank ÷ heads ÷ reel m; else mother ÷ lanes) → dated preferred
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
| R1 | Machine hourly rates: ERP v1 seeds = pre-S7 carriage-INCLUSIVE (LITHO 46.83, WIDE-REELER 71.11, formers flat 40.64 — the misapplied RF1–7 rate) vs **S7 carriage-stripped per-machine** (RF1–7 36.98, 8/10/11REEL 77.41, 9REEL 104.72, Wide 58.30, Narrow 51.17…) | **S7 stripped is the truth** (fulfilment is per-job, never in the hour) | Lewys cross-check; Stuart Q2 reply. **14 Oct:** Stuart's own YF855 quote reproduces to the penny at the OLD 8REEL £61.90 — at S7 £77.41 it is +7–8%; and with 8RF dismantled, 10RF absorbs its hours (the 8REEL rate assumed only 1,377 h/yr, so it is volume-sensitive) — re-model. **15 Oct:** Lewys — review running hours across ALL machines and current shift patterns/FTE before re-rating; rates refresh from next year's budget once accepted. **26 Oct:** Lewys agrees 10/11REEL are too high (allocation, not data); he rebuilds after the redundancy costs are confirmed, adding 10 + 11 and dropping 8; budget table received (R18) |
| R2 | 11REEL has no modelled rate (bought this financial year) | 8REEL-class proxy, flagged — Lewys thinks the stand-in may be 9REEL; checking. **23–26 Oct:** proxy agreed too high (Seth + Lewys); 11REEL idle since its customer was lost (ruling 26); 9REEL is a triple head so not a like-for-like stand-in; twin head = 1 operator | Lewys to model (drivers, R18) |
| R3 | Base labour £19.33 (S7, Mar) vs £20.29 (Lewys, Oct) | **RESOLVED 15 Oct: £20.29** (other grades £25.69/£23.91 to confirm) | Lewys |
| R4 | Efficiency/productivity: 85% assumed vs ~70% observed (v1's OI-1) — and S7's hours assume 85% | adopt v1's `productivity_factor` design; seed from the baseline exercise. **15 Oct (Lewys):** ONE efficiency per machine, from the productivity reports/timesheets (9 months; 10RF ≈ 93%) — replaces both the 85% and the quotes' 75% | Hannah extract → Lewys validate (v1's procedure docs) |
| R5 | Overhead pots: v1 profile (labour £117,467 / 80,000kg / admin 15.5%) vs sandbox factory params vs S7 pots | rebase from S7 + current accounts at founding; versioned profile | Lewys |
| R6 | Material prices 539 days old in v1; sandbox catalogue has reconciliation candidates | refresh at founding; dated prices mandatory thereafter. **13 Oct:** mill reels also need live/dormant/obsolete status from the stock file — SP 90's historic 1040mm mother is OBSOLETE, the live buy is the 1065mm FSC line; one price-list row per grade hides this. **27 Oct:** the sandbox's Cases material list is the workbook's 13 Aug snapshot, not the stock file (GP Kraft 50 has one line, 101759 at 960mm; AX312's 526mm kraft reel is dormant) — Oracle's Materials come from the stock file with status, and quotes pick a stock line (ruling 32) | Lewys/purchasing |
| R7 | Die data: S8 register (385 dies, rated speeds, feed waste, heads) vs v1 harvest tooling-cases.json (612 sizes, 601 unlinked die refs) | S8 is primary; harvest fills gaps; measured webs still to collect. **23 Oct:** S9 = Stuart's 2013 crimped-case catalogue (491 sizes, no die refs/heads/speeds) imported in the sandbox as reference-only dies (ruling 28) — 325 new sizes + 17 probable second dies on another machine family; conflicts and conditions (lost / toothless / cannibalised / not commissioned) with Stuart. **27 Oct:** measured webs on multi-head dies are now needed for route choice (ruling 33) — first: 51x38 10REEL 2-up = 263mm | Tim (register owner); Stuart (S9 questions) |
| R8 | Speeds: die RATED vs Stuart's quoting bands | rated = Standard, bands warn (decided 12 Oct) — bands maintained as production rules. **14 Oct:** rated speeds from Access proved unreliable (R14) — rated stays a Standard only once verified | Stuart |
| R9 | CHEVS ×(2÷1.5) template quirk vs S7 rate (~12% if it only means crew) | **RESOLVED 15 Oct (Lewys): the uplift is real crew** — one packer serves three Chevlers; Oracle's explicit CHEVS crew includes it (the quirk itself is not carried) | Lewys/Stuart |
| R10 | Delivery: flat % vs real fulfilment | §11 architecture native; % interim. **13 Oct — call-off orders are a first-class case** (e.g. 1m/year over ~6 drops, common): a flat % cannot see the drop count, so delivery must price drops × (pallets/drop × zone rate) + storage dwell, with the batching question (one run + storage vs several runs + set-ups) surfaced. v1's harvest carriage seeds (UK £50/pallet, IRE/SCO £65) are the interim per-drop numbers. **14 Oct:** pallet builds now known for 3,203 products via the carton (ruling 7); ~3,450 (mostly sheets) still need one (R15). **15 Oct (Lewys):** agreed — PlasPac rate cards + ~15% delivery margin; the 4.5% pot also covers storage, dispatch, rebates and charged shipments | Seth → PlasPac tariff |
| R11 | Packing £/kg seeds unconfirmed | carried, flagged | Stuart |
| R12 | Overhead recovery basis (admin/S&A as value-%) | carried at founding, flagged as the post-delivery follow-up (same critique as §11). **15 Oct:** on Stuart's YF855 the 1m order carries £612 of S&A vs £95 on the 100k — 6.5× for one order each; proposal to Lewys: per-order costs → a fixed per-order charge, truly value-linked costs (credit insurance, bad debt, commission) stay a small %, delivery → its own line; also reconcile S&A 15.5% (templates) vs 14.1% + 7.7% margin of safety (S7). **15 Oct (Lewys):** 15.5% stays (reforecast 15.6%); method open — machine-hour rates (Lewys) vs per-order + small value-linked % (Seth); a £50k/yr bad-debt allowance added as its own named parameter | Seth/Lewys |
| R13 | Oblong RF blank rule: evidenced from 65 historic BOMs (along-web = D2 + 2·D3 + 6–8mm trim) and, 14 Oct, the slit width = mother ÷ lanes of (D1 + 2·D3 + ≥10mm) — 49/61 products — but not yet confirmed by production | shipped in the sandbox 13–14 Oct; confirm both and the 10mm minimum | Stuart |
| R14 | Speeds from Access (die rated cycles, work-centre defaults, routing run factors) are unreliable — WBIG03's 6-7REEL die said 4/min, now 12 | every imported speed starts **unverified** (confidence system); verify machine by machine; route recommendations show the speed they rest on | Stuart / production |
| R15 | Pallet builds missing for ~3,450 products (no carton build — mostly sheets) | ask production / PlasPac for those only, not the whole range | Seth / production |
| R16 | 8RF dismantled; 26 Access order notes and historic routings/dies still name it | retire 8REEL; migrate dies, routings and notes to 10REEL at import. **23 Oct:** VP-16 decommissioned too (46 Access dies fit only VP-16) — historical, never routable (ruling 27); what replaces it on the sheet-fed route is open | Seth / Stuart |
| R17 | Engineer make-ready premium: legacy rows add £29.50 engineer to the set-up rate (ZHEN-SP-15, MUF wrappers, VP6-H, cups, ÉCLAR) | **double counts** — engineering is inside the hourly rate (ruling 22); set-up costed at the machine rate | Seth (sandbox Settings decision) |
| R18 | Machine-rate drivers: Lewys's budget (£1.922M, 28 lines × recovery type) is known; the per-machine drivers (m², running hours, kWh, crew, history split) are not | requested; rates become a calculation in calibration (ruling 25); rebuild waits on the redundancy costs | Lewys |
| R19 | Slitting tolerance (3mm over nominal) and the 5% offcut flag — seeded from the Access record, not confirmed by production; and whether RF2 is the preferred former among 1–5REEL for single-head work | seeded, flagged (rulings 30–31) | Hannah / Stuart |
| R20 | Saved products and historic quotes carry no slit-lane count, so they re-price without the offcut (sandbox: missing = legacy); products on a reel that doesn't divide need re-saving | migrate with lanes computed from mother + slit width, flagged for review | Seth |
| R21 | Packing standards: almost none recorded; history-weighted suggestions are reliable only where one machine group has a house standard (e.g. 10RF 660 / 19,800); weighting by sales units (91%) vs order counts (74%) to decide | collect from production (Hannah first: does RF2 pack 51×38 to 660 / 19,800?); decide the weighting | Seth / Hannah |
| R22 | Slitter rate: workbook flat £0.41/kg vs ~£0.04/kg on press time (400 m/min, SLITT £45.18/hr) — decides narrow vs wide on close calls and likely over-prices plain reel-fed slitting | Slitter's real speed (Stuart) + rate (Lewys); cost on time × rate | Seth / Stuart / Lewys |
| R23 | Machine selection pools: machines in a group (RF 1–5) compete on measured efficiency; production picks whichever is free | decide pooling vs per-machine, and when capacity/availability enters (planning stage) | Seth / production |
| R24 | Litho origination: the sandbox provision (£50 per plate per colour) vs the Ecoluxe calculator's own "origination" charge in its Live Calc parameters — same concept? | reconcile before porting litho quoting | Seth |

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
nest); weights. **Added 14 Oct:** `access-product-map.ts` (Access product → quote inputs as a
Draft, with notes), `route-move.ts` (machine → route; product vs route inputs), the staggered-head
and oblong-slit rules in `estimate-checks.ts` / `production-rules.ts` (`reelHeads`,
`OBLONG_SLIT_MIN_SPARE_MM`), `quote-workings-screen.ts` (the screen-order internal blueprint), and
the extract's new fields — `carton` + pallet build, `packQty`, `material.printType`,
`material.motherCode`, `orderNotes`. **Added 23–26 Oct:** `tools-filter.ts` (Tools list filtering incl. a
missing-data enrichment worklist), `tools-import.ts` (merge a low-confidence die list without duplicating
covered sizes; report probable second dies), the reference-tool gate in `findDieMatches` and the
`die-reference` / `machine-decommissioned` checks, `DECOMMISSIONED_MACHINES`, the S9 extract
(`crimped-catalogue.json` + its script), and the workbook-attribution method (baseline → current one
change at a time, deltas summing exactly) as the shape for parallel-running drift explanations.
**Added 27 Oct:** `slitLayout` + `lowerWasteRoutes` and the `mother-offcut` / `offcut-route` checks
(`estimate-checks.ts`), `SLIT_OVERWIDTH_TOLERANCE_MM` / `OFFCUT_FLAG_PCT` / `REEL_FORMER`
(`production-rules.ts`), the offcut build-up line + stock-line pricing in `engines/cases/shared.ts`
(maths only — Oracle's engine owns its own), and `cases-material-lines.ts` (stock-line labels,
first-match vs pinned resolution, mill-reel → mother width). Also 27 Oct: `packing-suggest.ts` (`suggestPacking`,
`validatePackingStandards`), `packingMachineGroup` + `PACKING_AUTOFILL_MIN`, the certainty-level
functions in `estimate-checks.ts`, and `CertaintyBadge` (ui-kit) as the shape of a certainty display. Also 27 Oct: `print-route.ts`
(`printRouteOptions`, `comparePrintRoutes`, `allInColumns`, `currentPrintRoute`), the narrow-reeler rules
in `production-rules.ts`, and the engine's `pressWebMm` / `preSlit`. Also: `route-optimiser.ts` (`routeCandidates`,
`historicWeb`, `currentRoute`) and `standardCycle(prim, params, inputs)` / `reelBandCap`. Also 28 Oct: `origination.ts` (`PRESS_PLATES`, `defaultPlate`, `plateCostPerColour`, `normaliseOrigination`).

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

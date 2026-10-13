# Chevler Oracle — Platform Vision

*Drafted 13 Oct 2026 from Seth's direction, for critique before anything hardens. This is the
paradigm document; the quoting detail lives in `quoting-module-spec.md` and what we inherit is in
`erp-v1-harvest-report.md`.*

## What this is

Chevler Oracle is the second iteration of the Chevler ERP — a ground-up rebuild that organises the
whole system around how work actually moves through the business, rather than around modules. It
replaces, in time: the three estimator spreadsheets' lineage (already replaced once by the
estimator sandbox), the estimator sandbox itself (`chevler-pricing-calculator`, the live quoting
tool), and ERP v1 (`chevler-erp`, never taken live). It inherits the best of all three and the
verified data story assembled in October 2026 (S7 machine rates, the S8 die register and routing
history).

## The organising idea: one job spine

Every piece of work is one thread, visible end to end:

> **Quote** (new · from a saved product · re-quote of an existing job)
> → **Material availability & cost** (what we'd use, what it costs today, what's on the shelf)
> → **Structure & Routing** (the product structure and machine routing, generated *with* the quote,
>   so production sees exactly what was priced)
> → **Accepted** → converts to a **Job**
> → **Production planning** → **Completion** (actuals meet the quote: quoted vs actual margin, per job)

On screen, a job IS this thread: a series of collapsible stages. Complete a stage, it collapses to
a one-line summary; the next expands. "Where is job X and how did it get here?" always has one
answer, on one page, with every number traceable to its source.

Two properties fall out of the spine that no amount of module-building gives:

1. **The quote is never orphaned.** Today a quote is a PDF and a hope. Here it's stage 1 of the
   thread — the S&R it generated is what the job runs to, and completion closes the loop with
   actual vs quoted cost. That loop (ERP endgame implication 10) is the entire point.
2. **Nothing is re-keyed.** The spec entered at quote time *is* the job spec, the routing priced is
   the routing planned, the material costed is the material allocated.

## What the spine reads: master-data planes

"Everything starts with a quote" is true of the **thread**, not the system. Around the spine sit
reference planes, each owned, audited and editable in one place:

| Plane | Contents | Primary source at founding |
|---|---|---|
| Customers | accounts, contacts, tiers/discounts, contract prices | ERP v1 design + sandbox quote history |
| Materials & Stock | grades (type+gsm+colour) → widths → dated supplier prices; individual reels | ERP v1 schema; prices REFRESHED (v1's are 539 days old) |
| Machines & Rates | capabilities (process, output, widths, colours), hourly rates, run standards with confidence + productivity factor, multi-machine standards | S7 carriage-stripped rates; S8 standards; v1's confidence/calibration design |
| Tools & Dies | the die register: dims, heads, rated speed, feed waste, machine compatibility, measured webs | S8 register (385 dies) merged with v1 harvest (612 case sizes, 601 die refs) |
| Parameters | overhead pots, margins, waste factors, delivery options, FX — versioned, audited | sandbox stores + v1 overhead profiles, reconciled per the spec §4 |

Reporting, dashboards and planning views sit *over* the spine; they are projections, never inputs.

## The front door: a command bar

The primary way in is typing, not navigating:

- `WGRE37` → the product, its history, its routing, one keypress from a re-quote
- `soleil` → the customer, their quotes, jobs, prices
- `57x48 chocolate 50 1m` → a new quote, steps 0–2 already answered, die auto-fitted
- `CHV-2610-0106` → that quote's thread

The command bar is not a search feature bolted on — it is the quote wizard's first three steps.
Everything we built for the assumption engine (dims→die, similar products, repeat recall) is the
parser behind it. The collapsible flow remains the full fallback for discoverability: nobody is
*required* to know the grammar, and the obvious sections (reporting, planning, admin) stay
navigational.

## Doctrine (carried from the sandbox, now platform-wide)

1. **Truth-first.** Every rate, speed and waste figure is verified reality with a named source and
   a confidence level; legacy numbers are reference oracles, never acceptance targets.
2. **Accuracy & transparency.** Anything grouped is split into its components; everything is
   editable; GP and NP are bullet-proof — every figure shown multiplies out by hand from the
   printed components.
3. **The assumption engine.** The system proposes (die, machine, piece, nest, packing, route), the
   user disposes. Every proposed value shows its Standard and its source; every deviation is a
   visible override; checks flag incompatibilities and suggest the way forward. Recalled work
   re-prices exactly.
4. **Making price + delivery line.** The per-1,000 is the MAKING price. Delivery is its own
   per-job line costed from the job's real pallets and weight (flat % only as the acknowledged
   interim until the PlasPac tariff), with an explicit delivery-margin parameter.
5. **Accountability as a primitive.** Every entity carries an append-only, field-level change log
   with what-changed detail and rollback; sensitive views are logged; every quote stores the full
   input + parameter snapshot and a reproduction check. This is ERP v1's gap #3, generalised from
   a per-store afterthought into platform infrastructure.

## Build strategy: ship in stage order

The spine is also the build plan. Each stage is independently useful; nothing waits for the whole.

1. **Quoting module** (stage 1) — the knowledge is ~90% in hand; spec in `quoting-module-spec.md`.
   Runs in parallel with the live sandbox per product family until it wins, family by family;
   Ecoluxe cuts over last.
2. **Quote → Job conversion** — acceptance, the S&R handed to production, job records.
3. **Planning** — scheduling against machines/shifts (v1's shift_patterns/downtime designs).
4. **Completion & actuals** — timed runs, actual material, quoted-vs-actual margin per job;
   this is also where run-standard confidence graduates from estimate → observed → verified.

### The three-codebase reality during the build

| Repo | Role | Rule |
|---|---|---|
| `chevler-pricing-calculator` | LIVE quoting tool; requirements source; regression oracle (855 tests, golden fixtures) | keeps shipping; feature-freeze bias once Oracle's engine build starts |
| `chevler-erp` (v1) | parts donor: schema design, docs (gap list, pricing brief, platform report), harvest data | read-only; no new work |
| `chevler-oracle` | the build | everything new lands here |

"Done" for Oracle's stage 1 retires the sandbox's estimators (the sandbox's Ecoluxe last);
"done" for the platform retires ERP v1 entirely.

## What would make this wrong (honest risks)

- **Scope creep across stages** — the spine invites building everything at once. The discipline is
  the stage gate: quoting ships and wins before jobs are started in anger.
- **Command-bar parsing quality** — if `57x48 chocolate 50` mis-parses, trust dies fast. Mitigate:
  the parser only pre-fills; the wizard always shows what it understood, every field editable.
- **Data debts carried forward unpaid** — stale material prices, estimate-confidence run standards,
  the unresolved rate questions (11REEL, base labour, CHEVS quirk, PlasPac tariff). The spec's
  reconciliation register (§4) names each with an owner; cutover per family is gated on its rates
  being signed off.
- **Three codebases, one developer+Claude** — mitigated by the freeze rules above and by Phase 0
  living here, in writing, instead of in anyone's head.

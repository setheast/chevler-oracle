# §7 review — questions for the decision session

Drafted 9 Nov 2026 for Seth (Lewys for items 2 and 6) ahead of a `/grill-me` session on the
quoting-module spec's §7 open decisions. Phase 1 (the quoting engine) starts when these are
answered. Each item gives what is proposed, what we already know, and the questions to settle.
Record each answer as a ruling in the spec and close the §7 item in the same commit.

Suggested order: **7 → 5 → 3 → 1 → 2 → 4 → 6**. Hosting decides the stack's shape, the
freeze date follows from Phase 1's start, and the cutover order decides which rates gate first.

---

## 7. Hosting — decide before any code

**Proposed:** hybrid. The primary server sits on the factory network. Streaming replication goes
to a cloud replica that serves off-site access and holds the off-site backups.
**Fixed by ruling 69:** usable on site with no internet; ≤ 2 hours of work lost (target minutes);
off-site backups that are restore-tested; internet-only features degrade gracefully.

1. **Outages.** How often does the factory internet go down, and for how long? Is there a record
   from the ISP or your memory of the last year? This tells us whether the on-site node is
   insurance or a daily necessity.
2. **Off-site users during an outage.** The cloud replica is read-only. Can Sharon and anyone
   working from home live with read-only access (look up prices and quotes, but not issue them)
   until the line returns? Or must off-site staff be able to issue quotes?
3. **The box.** Is there a server room or cupboard with power, UPS and network to every office
   and the shop floor? Windows or Linux? Buy new, or reuse something?
4. **Who looks after it.** Is there an IT provider who would patch, monitor and replace hardware?
   Or is it Seth + Claude? Who rehearses the restore, and how often (quarterly?)?
5. **Recovery time.** The spec sets the data you can lose (≤ 2 h), not how long you can be down.
   If the on-site server dies, is half a day acceptable? An hour? (This decides whether the cloud
   replica must be able to take over writes.)
6. **Power.** Do power cuts take the factory network down with the server? If so, is a UPS sized
   for a clean shutdown enough?
7. **Cloud side.** Is Railway (as today) acceptable for the replica and off-site access, in a UK
   or EU region? Is customer data in the cloud a concern for any customer contract?
8. **Plain-English questions** (ruling 68) call Claude from the on-site server. Is outbound
   internet from that box acceptable to IT?

## 5. Stack

**Proposed** (harvest report §5): keep ERP v1's Express/TypeScript + PostgreSQL + React/Vite,
with vitest from day 0. CLAUDE.md lists Tailwind.

1. Does item 7's answer change anything? For example, Postgres replication, and whether the
   on-site box runs Docker.
2. **Engines.** Port the sandbox's pure engines (`server/engines/`) as the starting code and then
   correct them truth-first? Or write fresh from the spec, using the sandbox only as the
   divergence reference? Porting is faster; fresh avoids carrying quirks by accident.
3. **UI.** Tailwind, or the Chevler design tokens (`tokens.css`, `--chev-*`) the sandbox uses?
   Can both apply (Tailwind configured from the tokens)?
4. Do the sandbox UI conventions carry over: ƒ tips, quiet guidance boxes, "Workings" folds,
   Evidenced/Confirmed/Assumed badges?
5. One repo for the whole ERP (quoting first), or separate packages?

## 3. When the sandbox freezes

**Proposed:** freeze `chevler-pricing-calculator` at Phase 1 start, except bug fixes and data
corrections.

1. **The date.** Is it the day Phase 1's first code is committed in Oracle?
2. **Where the line falls.** Is correcting a production rule that under-costs a "bug fix"? Two
   live examples:
   - The round-case slit width. The rule gives 70mm for 25×19 from a 1,050 reel. Every 25×19
     ever run from that reel was 75mm (~590 orders, 8 products). Adding a ~3mm minimum spare fits
     history better (511 vs 491 of 675 single-head products).
   - The Route box ignores a typed speed.
   Proposal: allowed when it corrects a price the sales team relies on, with each one going
   through an Oracle hand-over.
3. **New data.** Do rates, materials, tools and packing standards entered in the sandbox's
   Settings during the freeze carry into Oracle at import? Or are they re-entered?
4. **Who can grant an exception?** Is it Seth alone?

## 1. Cutover order

**Proposed:** Cases/Wraps → Main (sheets, reels, discs, liners) → Printed Sheets spec-first →
Ecoluxe last (live and untouched throughout).

1. **Is Cases still first?** Most open rate items sit in Cases: reel-former rates (R1, R2, R18),
   efficiencies (R4), the Slitter (R22), dies (R25) and Zhens (R31). Main's are fewer: the
   Wide/Narrow Reeler rates in R1, plus R5. Does "most value first" (Cases) beat "fewest open
   rates first" (Main)?
2. Do Wraps switch over with Cases, or separately?
3. Does a family switch over all at once, or customer by customer within it?
4. What does "Printed Sheets spec-first" mean in practice? Is it flexo printed sheets from Main,
   or litho? And how does it differ from Ecoluxe?
5. Is there a date to aim for on the first family? (The pricing year? The redundancy timing?)

## 2. Which rates gate which family

Each R item is either **gating** (closed before that family switches over) or **flagged** (it goes
live labelled Unverified/Assumed under the confidence system and is corrected later). The
proposal below is for Cases; repeat it for each family in item 1's order.

| Proposed for Cases | Items |
|---|---|
| **Gating** | R1 reel-former rates (Lewys's rebuild: 10 + 11 in, 8 out) · R2 11REEL · R4 one efficiency per machine · R5 overhead pots · R22 Slitter cost method |
| **Flagged** | R8/R14 speeds · R11 packing £/kg · R13 oblong blank · R19 slit tolerance · R21 packing standards · R25 die fits · R31 Zhens (once the crew ratio is set) · round-case slit spare (new, above) |
| **Not Cases** | R24 litho origination · R29 € forward contracts (policy) · R30 sales history (its own workstream) |

1. Agree the split, item by item. In particular: can a family go live while R12, the overhead
   recovery method, is still open, carrying S&A at 15.5% of value as today?
2. **Lewys:** when is the rate rebuild due? It waits on the redundancy costs. Is that weeks or
   months?
3. What counts as signed off? Proposal: a dated register entry naming who signed, with the source
   document in `docs/sources/`.
4. Truth-first means Oracle may price above today's quotes. Who decides what the sales team does
   with a large divergence on a live customer: hold the old price, phase it in, or re-quote?

## 4. Pilot families for parallel running

**Proposed:** four families (ERP v1's pricing brief), one containing EH400.

1. **Suggested starting set:** the 15 products from the 6 Nov top-5-customer price check
   (`Chevler Top 5 Customers - GP NP Price Check 6 Nov 2026.xlsx`). They already have current
   prices, sandbox prices at 5/10/15% NP and a cost build-up, so the divergence is measurable
   from day one. Do four of those, plus EH400, cover the right families?
2. Who prices each quote twice during parallel running (Sharon, Stephanie, Seth?), and for how
   long? Proposal: a fixed number of live quotes per family, not a period of time.
3. **Divergence tolerance.** Above what gap must a divergence be explained before sign-off? 2%?
   5%?
4. Who signs a family off: Seth alone, or Seth + Lewys?

## 6. Pricing structure — stage 1 or 1.5

**Proposed:** design the schema for list prices, tiers, contract prices, discounts and the
approval ladder now; build it after quoting parallel-runs cleanly.

1. Does Cases' cutover need the approval ladder from day one: per-user discount limits
   (ERP v1 gap #4) and margin floors? Or do quotes stay cost-plus-margin as in the sandbox?
2. **Lewys:** what are the margin floors per category, net of discounts? Who sets them?
3. Do any customers have contract prices today that Oracle must honour at go-live, such as the
   Woodland € contract (R29)?

---

## Feeds from the shop-floor visit (w/c 9 Nov)

Take these to Julian, Hannah and production. The answers close or downgrade gating items:

- **R25** — are the "1-5REEL" dies fixed to one machine? What does moving one cost?
- **R31** — the Zhens: how many, their types, SP/TP, the 13/15 numbers, MUF01 variants, how jobs
  are allocated.
- **R4** — the source of one efficiency per machine (productivity reports/timesheets).
- **R19 / R21** — slitting tolerance, whether RF2 is the preferred single-head former, and packing
  standards (does RF2 pack 51×38 to 660 / 19,800?).
- **Round-case slit width** — measure a 25×19 slit reel on 3REEL (75mm expected), and ask the
  smallest spare production will accept over a round blank.

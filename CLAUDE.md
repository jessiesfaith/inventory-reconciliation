# Inventory Reconciliation — repo notes

Generic workflow rules live in `~/.claude/CLAUDE.md` (they apply to every repo). This file holds
**repo-specific facts only**. Source of truth = this repo, not the chat transcript.

## Repo facts (created 2026-06-15)
- **What:** Single-page static **full-cycle inventory reconciliation** workspace. A tabbed dashboard
  that ties the inventory subledger → GL control → liability, manages the obsolescence reserve / scrap /
  returns disposition, and reports turnover. **Workflow & analysis only — posts no journal entries,
  produces no financial statements, not accounting/tax/audit advice.** All client-side; state in
  `localStorage`; no accounts/backend.
- **Sample domain:** biotech — `Helixa Biosystems, Inc.`, a line of **gel beads / chromatography
  resins** (consumables: agarose SEC, magnetic streptavidin, Sepharose IEX, Protein A, silica
  DNA-binding) and **genomic instruments / machinery** (BenchSeq NGS sequencer, RapidCycler qPCR,
  PipetteMax liquid handler, GenoPrep extraction robot). The obsolescence story leans on resin
  shelf-life/expiry and superseded instrument revisions.
- **Package manager / framework:** none. Vanilla HTML/CSS/JS, no `package.json`, no build step, **no
  CDN deps** (Google Fonts only). The turnover chart is hand-rolled inline SVG (`trendChart()`).
- **Key file:** `index.html` (entire tool: inlined CSS + JS). **Fifteen tabs** via `state.tab` +
  `VIEWS{}`, presented through a **left vertical sidebar** (`.shell` flex = `.sidenav` + `.content`;
  `renderTabbar()` builds `#sidenav` from `GROUPS`): 6 section groups (Overview / Inbound & Storage /
  Demand & Outbound / Accounting / Costing & Valuation / Analytics) as headings with their tabs listed
  vertically beneath; sticky, collapses to a wrapped top bar ≤820px. Lifecycle order: recon, gl, liab,
  **receiving**, cycle, **bins**, **lots**, **backorder**, ship, reserve, cost, prod, lifo, turn,
  review. New 2026-06-15:
  - **Receiving** (`tabReceiving`/`recvMetrics`, `INBOUND_STAGES`) — inbound PO pipeline
    (Open→In transit→At dock→QC hold→Put away) funnel, on-time receipt, dock-to-stock. Data `state.pos`.
  - **Bin Map** (`tabBins`/`binMetrics`/`binUtil`) — zone-grouped bin cards w/ capacity utilization
    (green<70 / amber70-90 / red≥90 / empty). Data `state.bins`. CSS `.zone`/`.bin-grid`/`.bin`.
  - **Lot & Expiry** (`tabLots`/`lotMetrics`/`lotStatus`, `daysUntil()` vs `TODAY='2026-06-15'`) — FEFO
    register, days-to-expiry, expired (write-off) + expiring≤90d value. Data `state.lots`.
  - **Backorders** (`tabBackorder`/`allocMetrics`) — demand vs on-hand, allocated, available-to-promise,
    shortfall units/$, fill rate, next-receipt ETA. Data `state.alloc`.
  Original nine tabs via `tabRecon/tabGL/tabLiab/tabCycle/tabShip/tabReserve/tabCost/tabProd/tabLifo/tabTurn/tabReview`:
  1. **Reconciliation** — inventory roll-forward + subledger→GL→physical tie-out. Each roll-forward
     line shows its Dr/Cr GL accounts and is **expandable** (`rfRowHTML()` / `toggleRF()` / `rfSetAll()`,
     open-state in module-level `rfOpen`) to list every posting of that type for the period with a
     tie-ing subtotal; begin/end rows tagged with the 1300 control account.
  2. **GL Transactions** — double-entry journal register (add/delete postings).
  3. **Liability** — GR/IR clearing (received-not-invoiced) + AP roll-forward, GR/IR ageing.
  3b. **Cycle Counts** (`tabCycle`/`cycleMetrics`/`countStatus`/`setCount`) — ABC count program
     (`ABC` const: freq + variance tolerance by class), book-vs-counted variance, accuracy %, A-item
     coverage, recount flags; net variance feeds the perpetual→physical tie-out. Data `state.counts`.
  3c. **Order Fulfillment** (`tabShip`/`shipMetrics`/`orderStatus`, `SHIP_STAGES`) — in-progress sales
     orders, fulfillment funnel by stage value, in-transit (cutoff-sensitive) value, on-time rate.
     Data `state.orders`.
  4. **Reserves & Disposition** — obsolescence (E&O) **reserve roll-forward** (opening + provision −
     scrap − reversal = ending vs policy-required), E&O ageing & reserve by SKU (`RESERVE_POLICY` bands
     by days idle/expiry → reserve %), **scrap log**, and **returns → repair&reuse | restock | scrap**
     disposition with value-recovery rate. Logic in `computeReserve()`; data in `state.eoItems` /
     `state.scrap` / `state.returns` + `reserveOpening/Provision/Reversal`.
  5. **Cost by Invoice & GL** — product cost per vendor invoice, segmented toggle to roll up **by GL
     account** (`state.costView` = 'invoice'|'gl').
  6. **Product Costs** — product catalog with **variable / fixed / R&D / overhead** unit-cost
     composition, std cost, price & margin (stacked composition bars).
  7. **LIFO Costing** — perpetual **LIFO** layer engine per SKU (`lifoEngine(item,'lifo'|'fifo')`,
     `waCalc()`, `lifoPortfolio()`): newest lots consumed first; LIFO vs FIFO vs weighted-average
     ending inventory / COGS / margin, **LIFO reserve** (FIFO−LIFO), open layer schedule, full
     cost-flow movement worksheet, and a **LIFO-liquidation** flag (opening-layer drawdown). SKU
     picker via `state.lifoSku` / `setLifoSku()`; sample data `seedLifo()` (gel-bead SKUs, rising costs).
  8. **Turnover** — annualized turnover + DIO with **MoM / QoQ / YTD / YoY** tiles + 24-month
     COGS-bars / turnover-line dual-axis chart.
  9. **Review & Export** — exception log, three-role sign-off (preparer≠reviewer SoD flag), period
     lock, CSV + Markdown export (both include the reserve/scrap/returns + LIFO summary).
- **Movements & Service group** (4 tabs, added 2026-06-15) — each ties into the inventory roll-forward:
  - **In-Transit** (`tabIntransit`) — goods in transit; `owned` (vendor FOB-origin + customer
    FOB-destination) adds to ending inventory (`c.itOwned`). Data `state.intransit`.
  - **Internal Transfers** (`tabTransfers`) — branch moves (`kind:'Branch'`, net $0) + internal usage
    (`kind:'Internal use'` — marketing/demo/sample/R&D) = roll-forward subtraction (`c.internalUse`).
    Data `state.transfers`. The "grey area," tracked up front.
  - **Repair & Reuse** (`tabRepair`, `REPAIR_STAGES`) — refurb workflow; `stage:'Reused to stock'`
    adds back (`c.repairReuse`). Data `state.repairs`.
  - **Warranty / Assurance** (`tabWarranty`) — `'Replacement out'` subtracts, `'Return to stock'`
    adds (`c.warrantyOut`/`c.warrantyIn`). Data `state.warranty`.
- **Approvals tab** (`tabApprovals`, in Accounting group) — delegation-of-authority matrix for a
  ~$500M company (`APPROVAL` 5 tiers: Accounting Manager ≤$25K / Controller ≤$100K / VP Finance ≤$500K
  / CFO ≤$2.5M dual / CFO+CEO >$2.5M dual; `approvalFor(amount)`). Manual GL entries (`addTxn` sets
  `src:'manual',apprStatus:'Pending'`) route here; `approveTxn()` records the approver (prompt). Seed
  txns are `src:'import',apprStatus:'Posted'`. Pending count = sidebar pip + tiles. GL Transactions
  table has an Approval column.
- **Tab numbering + GL cross-ref:** all tabs numbered 1-20 in sidebar order (`TAB_ORDER`/`TAB_NUM`,
  `.navnum`). GL Transactions tab has a **Chart-of-Accounts list** mapping each `ACC` account → the
  numbered tab(s) that tie it out (`ACC_TABS` + `tabNums()`, clickable). **20 tabs total.**
- **Roll-forward tie-color:** every figure that flows into the roll-forward is rendered in teal
  (`--rf` / `.rf-tied` / `.rf .rfrow .val`) — across the roll-forward column and each contributing
  tab's tied total — so users can follow it. Legend + `↳` jump links on the roll-forward. The 5
  movement lines are added in `tabRecon`'s `rf` array with `{tied,tab,gl}` opts; `rfRowHTML` renders
  tied/memo lines.
- **Engine:** journal-entry driven. `TXN_TYPES` maps each posting type → debit/credit accounts (`ACC`)
  + roll-forward sign; `compute()` derives inventory control, GR/IR, AP from the register, then folds
  in the six movement buckets so `glInv = glBase + itOwned + repairReuse + warrantyIn − internalUse −
  warrantyOut` (sample ties to $451,500). Turnover from `state.periods` (24 months, `seedPeriods()` mean-reverts inventory to ~2.15×
  monthly COGS ≈ 5–6× annualized, ~62-day DIO). `turnAnalytics()` computes MoM/QoQ/YTD/YoY.
  **localStorage key `inv_recon_v7`** (v2: reserve/scrap/returns + biotech reskin; v3: LIFO; v4: cycle
  counts + orders; v5: receiving `pos` + alloc + lots + bins; v6: intransit + transfers + repairs +
  warranty; v7: txn approval fields (`src`/`apprStatus`/`apprBy`); all 2026-06-15; bump again when the
  data shape changes).
- **Dev (localhost):** `npx http-server . -p 8734 -c-1` → http://localhost:8734. (Machine `python` is a
  non-functional Windows Store alias — use Node's http-server. Root `.claude/launch.json` has an
  `inventory-reconciliation` config on port 8734 for the Claude preview panel.)
- **Build / test / lint / typecheck:** none configured. Verification = manual + preview panel.
- **Deploy:** **DEPLOYED 2026-06-15 via Vercel CLI** (`npx vercel deploy --prod --yes` from this dir).
  Vercel project `inventory-reconciliation`, account `jessicadougherty4321-6324`.
- **GitHub:** **git-connected** to **`jessiesfaith/inventory-reconciliation`**. Push via the SSH alias
  `github-jessica:jessiesfaith/inventory-reconciliation.git`; `git push origin main` → Vercel
  auto-deploys. (Repo was first created as `inventory-reconciliation-` with a stray trailing hyphen,
  then renamed 2026-06-15.) **`vercel git connect` gotcha:** it can't parse the `github-jessica:`
  SSH-alias remote — temporarily `git remote set-url origin https://github.com/jessiesfaith/inventory-reconciliation.git`,
  run `npx vercel git connect --yes`, then restore the SSH alias for pushing.
- **Production:** standalone **https://inventory-reconciliation.vercel.app** (clean alias was
  available — unlike month-end-close). Latest deployment id `dpl_Ao4FbYQom9gWEGyXQE7vb1agY5fo`.
- **Pretty path LIVE:** **https://app.fastinsights.io/inventory-reconciliation/** — redirect+rewrite
  in the `fast-insights-app` (AR Tool-Beta) `vercel.json` (→ the `inventory-reconciliation.vercel.app`
  alias) + a `Boxes` tile in its `src/pages/Landing.tsx`, committed to `main` (commit
  `eb3369b` "Add Inventory Reconciliation tile + proxy route", git auto-deployed). No-slash path 307s
  to the trailing-slash form.

## Conventions (match the other Fast Insights tools)
- `vercel.json` = `{ "cleanUrls": true }`. `.vercelignore` keeps everything but `index.html` +
  `vercel.json` out of the deploy.
- Dark Fast Insights palette (near-black `#0a0a0a` + green accent `#00c805`), Outfit body +
  Cormorant Garamond headings. Same CSS framework as month-end-close / trust-strategy-builder.
- Guardrail tone: workflow/analysis only; never present as posting entries or giving accounting/tax/
  audit advice. Every tab carries the planning-only disclaimer.

## Environment notes (this machine)
- **OneDrive Files-On-Demand:** this repo is inside a OneDrive-synced vault. Files may be cloud
  placeholders that tools intermittently fail to see. Retry once.
- **GitHub = Jessica (`jessiesfaith`).** Confirm Chris vs. Jessica before any GitHub work.

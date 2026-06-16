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
- **Key file:** `index.html` (entire tool: inlined CSS + JS). Eight tabs via `state.tab` + `VIEWS{}`
  map → `tabRecon/tabGL/tabLiab/tabReserve/tabCost/tabProd/tabTurn/tabReview`:
  1. **Reconciliation** — inventory roll-forward + subledger→GL→physical tie-out.
  2. **GL Transactions** — double-entry journal register (add/delete postings).
  3. **Liability** — GR/IR clearing (received-not-invoiced) + AP roll-forward, GR/IR ageing.
  4. **Reserves & Disposition** — obsolescence (E&O) **reserve roll-forward** (opening + provision −
     scrap − reversal = ending vs policy-required), E&O ageing & reserve by SKU (`RESERVE_POLICY` bands
     by days idle/expiry → reserve %), **scrap log**, and **returns → repair&reuse | restock | scrap**
     disposition with value-recovery rate. Logic in `computeReserve()`; data in `state.eoItems` /
     `state.scrap` / `state.returns` + `reserveOpening/Provision/Reversal`.
  5. **Cost by Invoice & GL** — product cost per vendor invoice, segmented toggle to roll up **by GL
     account** (`state.costView` = 'invoice'|'gl').
  6. **Product Costs** — product catalog with **variable / fixed / R&D / overhead** unit-cost
     composition, std cost, price & margin (stacked composition bars).
  7. **Turnover** — annualized turnover + DIO with **MoM / QoQ / YTD / YoY** tiles + 24-month
     COGS-bars / turnover-line dual-axis chart.
  8. **Review & Export** — exception log, three-role sign-off (preparer≠reviewer SoD flag), period
     lock, CSV + Markdown export (both include the reserve/scrap/returns summary).
- **Engine:** journal-entry driven. `TXN_TYPES` maps each posting type → debit/credit accounts (`ACC`)
  + roll-forward sign; `compute()` derives all balances (inventory control, GR/IR, AP) from the
  register. Turnover from `state.periods` (24 months, `seedPeriods()` mean-reverts inventory to ~2.15×
  monthly COGS ≈ 5–6× annualized, ~62-day DIO). `turnAnalytics()` computes MoM/QoQ/YTD/YoY.
  **localStorage key `inv_recon_v2`** (bumped from v1 when the reserve/scrap/returns fields + biotech
  reskin were added 2026-06-15; bump again when the data shape changes).
- **Dev (localhost):** `npx http-server . -p 8734 -c-1` → http://localhost:8734. (Machine `python` is a
  non-functional Windows Store alias — use Node's http-server. Root `.claude/launch.json` has an
  `inventory-reconciliation` config on port 8734 for the Claude preview panel.)
- **Build / test / lint / typecheck:** none configured. Verification = manual + preview panel.
- **Deploy:** **DEPLOYED 2026-06-15 via Vercel CLI** (`npx vercel deploy --prod --yes` from this dir).
  Vercel project `inventory-reconciliation`, account `jessicadougherty4321-6324`. Re-deploy: same
  command. **NOT git-connected yet** (no own GitHub repo — `gh` not installed; Jessica must create
  `jessiesfaith/inventory-reconciliation` manually to enable push-to-deploy, same gap as
  month-end-close).
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

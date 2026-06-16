# Inventory Reconciliation — Handoff

Session-state snapshot for the next person/agent. Detailed repo facts live in `CLAUDE.md`; this file
is "where things stand right now."

_Last updated: 2026-06-16._

## TL;DR
Full-cycle inventory reconciliation dashboard — a single self-contained `index.html` (vanilla
HTML/CSS/JS, no build, no CDN deps beyond Google Fonts). **Live, deployed, git-connected, cleaned up.**

## Live
- **Standalone:** https://inventory-reconciliation.vercel.app
- **Branded:** https://app.fastinsights.io/inventory-reconciliation/ (also on the app.fastinsights.io tool picker)
- **Code:** https://github.com/jessiesfaith/inventory-reconciliation (`main`)
- **Deploy:** push to `main` → Vercel auto-deploys (~10s). Manual: `npx vercel deploy --prod --yes`.

## What it does (20 tabs, 7 sidebar groups)
Vertical left sidebar, tabs numbered 1–20, grouped: Overview / Inbound & Storage / Demand & Outbound /
Movements & Service / Accounting / Costing & Valuation / Analytics.
1. **Reconciliation** — inventory roll-forward (expandable journal lines → postings; movement & service
   lines tie to their tabs) + subledger→GL→physical tie-out. Sample ties to **$451,500**.
2. GL Transactions (double-entry register + chart-of-accounts→tab cross-ref + per-entry approval status)
3. Liability (GR/IR + AP roll-forward, ageing)
4. Receiving · 5. Cycle Counts · 6. Bin Map · 7. Lot & Expiry
8. Backorders · 9. Order Fulfillment
10. In-Transit · 11. Internal Transfers · 12. Repair & Reuse · 13. Warranty / Assurance
14. Reserves & Disposition · 15. Approvals (DOA matrix) ·
16. Cost by Invoice & GL · 17. Product Costs · 18. LIFO Costing · 19. Turnover (MoM/QoQ/YTD/YoY) ·
20. Review & Export (sign-off, period lock, CSV/Markdown)

> NOTE: the numbers above are group/feature order; actual `TAB_NUM` is computed from `GROUPS` flat
> order at runtime — check the sidebar for the live numbering.

## Key mechanics (for the next change)
- **Engine:** `compute()` derives all balances from `state.txns` (journal) + 4 movement datasets
  (intransit/transfers/repairs/warranty); `glInv = glBase + itOwned + repairReuse + warrantyIn −
  internalUse − warrantyOut`.
- **Roll-forward tie color:** one teal (`--rf` / `.rf-tied`) marks every figure that flows into the
  roll-forward — across the roll-forward column AND each contributing tab's **`rfTiePanel`** (a teal
  "Ties to the inventory roll-forward" card). `rfMovements(c)` is the single source for the 14 movement
  lines; `rfLineDetail(line)` resolves each amount to its source rows (collapsed, click-to-expand).
- **Approvals:** `APPROVAL` 5-tier DOA matrix for a ~$500M company; `addTxn` posts manual entries as
  Pending → approve on the Approvals tab.
- **localStorage key `inv_recon_v7`** — bump it (and `defaultState().v` + the `load()` check) whenever
  the data shape changes, or returning visitors load stale state.

## Dev / verify
- Local: `npx http-server . -p 8734 -c-1` → http://localhost:8734. (Root `.claude/launch.json` has an
  `inventory-reconciliation` config on 8734 for the Claude preview panel.)
- No build/test/lint. Verify by loading the page and checking: roll-forward ends at $451,500, all 20
  tabs render, no console errors. Syntax-check the inline script with
  `node --check` after extracting `<script>…</script>` if unsure.

## Environment gotchas (this machine)
- **Multiple preview servers run at once** (financial-statements:8736, scoutquest:8000, this:8734) and
  share one preview browser — confirm `location.href` is `localhost:8734` before eval/verify, and
  `preview_start name:inventory-reconciliation` then navigate the browser to 8734.
- **The Claude preview `screenshot` tool was timing out all session** (renderer issue, not the page) —
  verify via `preview_eval` DOM reads + computed styles instead.
- **GitHub = `jessiesfaith`** via SSH alias `github-jessica`. `vercel git connect` can't parse the
  SSH-alias remote — temporarily set origin to the https URL for the connect, then restore the alias
  for pushing (see CLAUDE.md).
- OneDrive Files-On-Demand vault: files may be cloud placeholders; retry a read once if it errors.

## State: complete / no open gaps
Built, deployed, branded, git-connected (push-to-deploy), and cleaned up (dead code removed). No known
defects. Possible future asks (not requested): add the teal tie-panel pattern to any new tabs;
calibrate the DOA thresholds/roles to the real policy; make the roll-forward movement lines expandable
to dataset detail (currently only journal lines expand inline).

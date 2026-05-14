# MF Structure — Answers to "Komal" Points

_Task #8: MF — Finalise the structure. Responses to designer discussion notes, 2026-05-12._

## Context

Current QA state is only an onboarding gate that redirects to Investwell. Phase 1 brings the full MF module in-house. Answers below use the existing **FD module** as the parity pattern — distributor-facing app, client-selection-based flows — so the two products feel identical to the distributor.

Decisions already locked in this session: commission **out of scope**, **free for investor**, SIP **full edit** (amount, frequency, stop, pause), redemption **distributor-initiated only**.

---

## 1. Home Tab — content of MF Home page

**DEFERRED — out of scope for Phase 1.** MF tab will land directly on Explore. Revisit when MF AUM / SIP volume justifies a dashboard surface.

Original draft (kept for later phase):

Mirror FD Home exactly so a distributor moving between products has zero learning curve:

- **Stat cards (3):**
  - **Total Clients (MF-active)** — clients with ≥1 MF holding
  - **Active SIPs** — count of live systematic plans (all types)
  - **Pending KYC for MF** — clients who cleared FD KYC but not MF-extended KYC (nominee/IPV/FATCA etc.)
- **Filter row:** search by client name/email + filter by MF KYC status (Not Started / In Progress / Verified / Failed)
- **Client table:** Name | Email | Client Type | MF KYC Status | Active SIPs count | Current MF AUM
- **Row actions:** Start KYC · Invest Lumpsum · Setup SIP · View Holdings

**Why not a holdings-first view:** FD Home is client-first because the distributor's mental model is "pick a client, then act." Keeping this in MF avoids two mental models in one app.

---

## 2. Explore Tab — card content

Each fund card shows the minimum a distributor needs to shortlist without opening detail:

- **Logo + AMC name** (top-left)
- **Scheme name** (single line, truncated)
- **Category tag** (e.g., Large Cap · ELSS · Debt – Short Duration)
- **1Y / 3Y / 5Y returns** (three inline numbers, color-coded vs category median)
- **Riskometer band** (Low / Low-Mod / Mod / Mod-High / High / Very High)
- **Min SIP** and **Min Lumpsum** (₹ amounts, two-line)
- **Rating** (Value Research / Morningstar stars — whichever RTA feed gives us)
- **Action pill:** Invest (opens fund detail)

**Filters (top bar):** Category · Risk · Min Investment · AMC · Rating
**Sort:** 1Y / 3Y / 5Y return · AUM · Expense Ratio

**Not on the card (goes into detail page instead):** NAV history, expense ratio, exit load, fund manager, portfolio holdings.

---

## 3. Orders Tab [FINALISED]

**Orders table columns:**
App ID · Client · Scheme · Folio Number · Type · Txn type · Amount · Units · NAV + NAV date · Status · BSE Star order ID · Ordered at · Plan ID (for installments)

**View Detail:**

**Header**
- Application ID · Status badge · Placed at (date + time)

**Client & scheme**
- Client name + Client ID · PAN (masked) · Folio number (if allotted, else "Pending")
- AMC · Scheme name · Plan (Direct/Regular) · Option (Growth/IDCW) · Category

**Order table of that row** (the full column set above, expanded)

---

## 4a. Portfolio [FINALISED]

**Placement:** Tab inside Investor Profile, named **Portfolio**. Contains Holdings table, Systematic Plans table (with nested mandate debit history), and AUM summary stats.

### Holdings table

**Grain:** one row per folio (= one scheme holding).

**Columns:**
- Scheme
- Folio number
- AMC
- Plan / Option (Direct/Regular, Growth/IDCW)
- Category (Large Cap / Liquid / etc.)
- Units held
- Average buy price (₹/unit)
- Current NAV (with as-of date)
- Current value (₹)
- Unrealized P&L (₹ + %)
- Last transaction date

**Row actions:**
- Buy more (lumpsum into this folio)
- Setup SIP (into this folio)
- Setup STP (from this folio — **primary entry**)
- Setup SWP (from this folio — **primary entry**)
- Redeem (one-time)
- View transactions (deep-links to Orders, filtered by this folio)

**Section header summary:** Total holdings count · Total invested · Total current value · Total P&L (across all folios).

---

## 4. Mandate [FINALISED — nested inside Systematic Plan, see section 5]

**Placement:** Mandate is **not a separate section or tab**. It is nested as the expandable child table inside every Systematic Plan row in the Investor Profile (see section 5). Mandate Debit History extends with every new debit attempt under that plan.

---

## 5. Systematic Plan [FINALISED]

**Placement:** Section inside Investor Profile (parity with Mandate). Investor profile is opened from the Client page by clicking the investor's name.

**Systematic Plan table — parent grain: one row per plan.**

**Columns:** Plan ID · Type (SIP/STP/SWP) · Scheme · Amount · Frequency · Next debit · Installments (Done / Total) · Status (Active / Paused / Stopped / Completed) · Actions

**Row actions:** Pause · Resume · Modify · Stop

**Expanding any plan row reveals the Mandate Debit History for that plan** (child grain: one row per debit attempt):

1. Date — of the debit attempt
2. UMRN — mandate identifier
3. Scheme — scheme of that debit
4. Linked plan — Plan ID that triggered the debit
5. Status — Success / Failed / Pending
6. Amount — ₹ debited (or attempted)
7. Bank — bank name
8. Account (masked) — e.g., XXXX1234
9. Max amount — mandate cap
10. Frequency — mandate frequency
11. Start/End — mandate validity
12. Auth mode — Net banking / Debit card / Aadhaar OTP
13. Failure reason — populated only when Status = Failed

This single hierarchical table replaces both the standalone Mandate tab and the standalone Systematic Plan tab. Mandate metadata is no longer surfaced as its own table — it lives nested inside each plan's expandable debit history.

---

## 6. Setup STP [FINALISED]

**Primary entry point: Holdings section in Investor Profile.** Distributor clicks row action "Setup STP" on the source folio. Opens a **dedicated full page** with all fields required to submit the STP request. Source is the folio itself — no order-to-folio resolution.

**Secondary entry (shortcut):** Orders tab row action "Setup STP from this scheme" — resolves to the order's folio and lands on the same dedicated page.

**Setup STP page content:**
- **Source** (pre-filled from the order's folio, locked): Scheme · Folio · Current units · Current value · Exit load status
- **Target scheme:** searchable picker (any open scheme from Explore catalogue)
- **Transfer plan:** Amount per cycle · Frequency (Monthly / Quarterly) · Start date · End condition (End date / Number of installments / Until source exhausted)
- **Tax note:** "Each STP cycle is a redemption from source + purchase in target — capital gains apply on the source leg per slab/LTCG rules."
- **Review & submit** → Activate STP

**Status lifecycle:** Draft → Active → [Paused] → Completed / Stopped.

**Do not** put STP entry in Explore — Explore is for discovering new funds, not transferring out of existing holdings.

---

## 7. Setup SWP [FINALISED]

**Primary entry point: Holdings section in Investor Profile.** Distributor clicks row action "Setup SWP" on the source folio. Opens a **dedicated full page**.

**Secondary entry (shortcut):** Orders tab row action "Setup SWP from this scheme" — resolves to the order's folio and lands on the same dedicated page.

**Setup SWP page content:**

- **Source (locked, pre-filled):** Client · Folio · Scheme · Plan/Option · Current units · Current value · Exit load status
- **Withdrawal plan:**
  - Withdrawal type: Fixed amount / Fixed units / Capital appreciation only
  - Amount or units (conditional on type)
  - Frequency: Monthly / Quarterly
  - Withdrawal day (1–28)
  - Start date
  - End condition: End date / Number of installments / Until source exhausted
- **Payout bank (locked):** pre-filled from investor's KYC bank — name, account (masked), IFSC.
- **Tax disclosure** (mandatory acknowledgement): each cycle is a redemption; capital gains apply per slab/LTCG rules.
- **Footer actions:** Cancel · Submit

**On submit:**
- Validations pass → Plan created with Status **Active**
- New row in **Systematic Plan section**
- **No mandate row** — SWP is outflow to investor's bank (not debit from it). NACH doesn't apply.
- **No order row yet** — orders appear only on cycle execution.

**Execution (each cycle):**
- AMC redeems amount/units from source folio
- Proceeds credited to investor's KYC bank (T+2/T+3)
- One Redeem order row appears in Orders tab
- Source folio's units shrink in Holdings
- Plan's "Installments done" counter increments

**Status lifecycle:** Draft → Active → [Paused] → Completed / Stopped.

---

## Also worth flagging (non-Komal points from the notes)

- **Naming consistency:** use the exact FD tab labels (Home / Explore / Orders) in MF too — not "Dashboard", not "Funds", not "Transactions".
- **Buy → Lumpsum:** rename tab entry to **Invest** (same verb FD uses on the card), and keep the two-step pattern of FD (Detail → Review) so the flow feels identical.
- **Select Bank screen:** not needed as a standalone screen if the client has only one bank on record; show it inline on Review. Promote to a full step only when multiple banks exist.
- **Configure SIP:** should be the same screen as Setup SIP (prefilled + editable), not a parallel UI. "Modify" just reopens the setup form with current values.

---

## Actions

- Share MF prototype with Ashid — **needs to be done by user** (I cannot access the prototype tool); once shared, record feedback here.
- Confirm with design that the Systematic Plan tab survives as a separate surface (section 5 above).
- Feed sections 1–7 into the MF PRD under "UX Structure".

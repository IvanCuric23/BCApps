# Shopify → Business Central: End-to-End Test Runbook (for agents)

A procedure an AI agent with a browser tool (Claude in Chrome, or a browser MCP server) follows to prove that an
order placed in Shopify becomes a sales order in Business Central. Run it in the main session — the packet's
subagents have no browser tool (`browser-walkthrough` skill).

Last run: 2026-10-26 — Shopify order #1001 → BC customer K-000074 → released sales order P-NAL-26-000027.

## The flow

```
[Shopify admin]                                   [Business Central]
1. Draft order ──► 2. Mark as paid ──► 3. Order ──► 4. Sync Orders From Shopify ──► 5. Shopify Orders row
                                       #10xx                                          │
                                                   7. Sales order checks  ◄── 6. Create Sales Documents
                                                                                  (automatic if Auto Create Orders)
```

## 0. Preconditions — check, do not assume

| # | Check | How | If it fails |
|---|---|---|---|
| P1 | Target is a **sandbox** BC environment and a **dev** Shopify store | URL: `businesscentral.dynamics.com/<tenant>/Sandbox`; store `*.myshopify.com` from the user | Stop. Never run this against production (`production_is_read_only`). |
| P2 | Shop card **Enabled**, access token present | Shopify Shops (page 30102) → open the shop | Human step, see §H. |
| P3 | Sandbox allows HTTP calls | Extension Management → Shopify Connector → Configure → **Allow HttpClient Requests** on | Ask the user before turning it on. |
| P4 | At least one Shopify product is mapped to a BC item | Shopify Products (page 30126): **Item No.** filled | Run **Sync Products** first, or pick a mapped product. |
| P5 | Shop card: customer template, **Auto Create Unknown Customers** or a default customer; shipping charges account | Shop card | Report as setup gap; do not change setup unasked. |
| P6 | Locations mapping has a BC location for the Shopify location | Shopify Shop Locations (page 30117) | Same as P5. |

Record the values you found (shop code, template, location) — the report needs them.

## H. Human-only steps — hand over and stop

The agent cannot do these. Ask the user, end the turn, and wait (`agent_handoff`):

- Turning **Enabled** on the Shop card the first time: a privacy-consent dialog, then a Shopify **app-install
  pop-up** that opens outside the browser tool's tab group.
- Any sign-in, CAPTCHA or password prompt — in BC or Shopify.
- Accepting terms in Shopify.

## 1. Create the order in Shopify (admin draft order)

1. Shopify admin → **Orders** → **Create order**.
2. Add one **mapped** product (P4), quantity 1.
3. Customer: create a new one with **invented** data only — e.g. *Test Kupac*, `test.kupac@example.com`. Skip the
   address if the dialog fights you (Escape closes it without saving).
4. **Collect payment** → **Mark as paid** → **Create order**.
5. Note the order number (`#10xx`) and the total.

**Pass:** the order page shows the number, **Paid**, **Unfulfilled**.

## 2. Import the order into BC

1. Open `https://businesscentral.dynamics.com/<tenant>/Sandbox?page=30115` (Shopify Orders).
2. Home → **Sync Orders From Shopify**. With **Run Syncs in Background** off it runs in the session; with it on,
   wait for the job queue entry and refresh.
3. Find the row for `#10xx`.

**Pass:** the row exists, **Financial Status** = Paid, **Has Error** = No.
**Fail:** row missing → check Shopify Log Entries (page 30119) for **Has Error** rows; row with **Has Error** =
Yes → record **Error Message**.

## 3. Create and check the sales document

1. If **Auto Create Orders** is on, the sales order already exists — the fact box **Linked Documents** shows it.
   Otherwise select the row → Home → **Create Sales Documents**.
2. Open the sales order from **Linked Documents**.

**Checks (each one pass / fail, with the value seen):**

| # | Check | Expected |
|---|---|---|
| C1 | Customer | The customer created from the template, or the default customer |
| C2 | Line | The mapped item, quantity 1 |
| C3 | Location | The BC location from the locations mapping |
| C4 | Currency | The store's currency — **known gap:** blank Shop card Currency Code books it in LCY |
| C5 | Amount incl. VAT | Equal to the Shopify total — **known gap:** BC added 25 % VAT on the last run |
| C6 | **Shopify Order No.** on the sales header (External Document No. holds the Shopify PO number, often blank) | `#10xx` |

Do **not** post the sales order unless the user asks: posting writes ledger entries in the sandbox, and it failed on
the last run on missing Inventory Posting Setup for the location.

## 4. Optional — shipment back to Shopify (only when the user asks to post)

Post the shipment → Shopify Orders → Home → **Sync Shipments To Shopify** → the Shopify order shows **Fulfilled**.

## Browser pitfalls (BC web client)

- The page content lives in a **same-origin iframe**: `document.querySelector('iframe').contentDocument`.
- A field's label has id `<x>lbl`; its control has id `<x>ee`. Find by label text, then swap the suffix.
- **Toggles ignore JavaScript `.click()`** — use a real mouse click from the browser tool.
- Selects accept setting `.value` plus dispatching a `change` event.
- Navigating **Back** can close the list and return home — open pages by URL (`?page=<id>`).
- A "Leave site?" dialog blocks navigation — close the page with its back arrow first. Never trigger browser
  dialogs on purpose; they freeze the browser tool.
- Wait 6–8 s after each BC navigation; a screenshot taken earlier shows the loading screen.
- Re-define any helper scripts after each navigation — the iframe is replaced.

## Data and screenshots

- Invent every customer value. Never type a real person's name, email, phone or address.
- Shopify admin shows the signed-in user's name (*"Updated by …"*, initials) — crop it out or do not capture
  that view (`no_personal_data_recorded`).
- No screenshot of the access token or any secret.

## Report — the shape

```
E2E Shopify → BC, <date>, BC <env> / store <store>
Preconditions P1–P6: <pass / fail each, values found>
Order: #10xx, total <amount> <currency>
Step 1 Shopify order   : pass | fail | not run — <evidence>
Step 2 Import          : pass | fail | not run — <evidence>
Step 3 Sales document  : <document no.>; C1–C6 pass/fail with values
Step 4 Shipment        : not run (not asked) | pass | fail
Records created: Shopify order #10xx, customer <no.>, sales order <no.>
```

- **not run** ≠ pass. A step blocked by a human step or a missing precondition is *not run*, with the reason.
- List every record the test created. Delete them only when the user asks.

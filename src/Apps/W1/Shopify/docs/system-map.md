# Shopify Connector — System Map

**Mapped:** `App/src` (the connector, 637 `.al` files in 28 modules) and `Test` (91 `.al` files, 56 test codeunits) — read with `al graph` (AL tools 18.1.43): 4331 nodes, 16235 call edges. **Built at `6f261e89` on 2026-10-26.** Stale when source under those roots changes after that commit.

## How to use this map

- Find the module you will change in **Modules**; its *depends on* and *used by* columns show what moves with it.
- Before changing a procedure, list its callers: `al graph query --graph .algraph/graph.jsonl --to "obj:<object name>" --depth 1` (the `al-graph` skill).
- Run the test areas in *tested by* (Test app folders) for the module you changed.
- Prefer an **integration event** (6 `… Events` codeunits, 70 events) or an **interface** below over editing Microsoft's code.

## What this map cannot see

- **Record use is not a link.** Links are calls between procedures (al graph). A module that only reads or writes another module's table — for example almost every module using the `Shpfy Shop` record from Base — gets no link for that. Treat the counts as call coupling, not data coupling.
- **Interface dispatch is over-counted.** A call through an interface is linked to every implementer (1,795 edges marked OverApprox). Catalogs → GraphQL and Inventory → GraphQL show 284 such links each; they are listed apart from direct calls.
- **The GraphQL path goes through an enum.** Modules call ExecuteGraphQL in `Shpfy Communication Mgt.` (Base) with a `Shpfy GraphQL Type` value; which GraphQL codeunit runs is decided at runtime.
- **Events raised by the platform** (table triggers, job queue, `Webhook Notification` inserts) and event subscribers in other apps.
- **Other apps.** Base Application and System Application calls are not mapped; the connector's 70 integration events (6 codeunits) are extension points for other apps, and their subscribers are outside this repository.
- **Pages and reports run by object id**, `Codeunit.Run`/`Report.Run` with a number, and anything chosen from a setup field.
- **Shopify itself** — what the Admin API does with a request.
- The root `app.json` (`Shopify` 1.0.0.0) is not the connector and was not mapped.

## Modules

Counts are direct calls between modules (procedure → procedure); `+N iface` are links through interface dispatch, over-counted (see above). No dependency-direction rules are enforced in this project.

| Module | Purpose | Files | Depends on | Used by | Tested by (Test folders) |
|---|---|---|---|---|---|
| **GraphQL** | One codeunit per Shopify Admin GraphQL query or mutation (143 of them), picked through the GraphQL Type enum. | 146 | Helpers 1 | Catalogs 2 (+284 iface), Inventory 2 (+284 iface), Base 3 | GraphQL, Products |
| **Order handling** | Shopify orders: import (Orders API, Import Order), mapping (Order Mapping), sales document creation (Process Order), order pages and the sales-document extensions. | 58 | Helpers 48, Base 29, Customers 6 (+14 iface), Order Return Refund Processing 4 (+12 iface), Order Fulfillments 4, Companies 4, Logs 2, Order Returns 2, Order Refunds 2, Order Risks 2, Products 2, Shipping 1, Transactions 1 | Order Return Refund Processing 5, Catalogs 3, Order Refunds 3, Transactions 3, Order Returns 1, Shipping 1 | Order Handling, Shipping, Gift Cards, Staff |
| **Products** | Products and variants: import from and export to Shopify, item creation, price calculation, images, collections, sales channels. | 53 | Helpers 88, Base 71, Bulk Operations 6 (+4 iface), Metafields 7, Translations 3 (+1 iface) | Base 2, Order handling 2, Catalogs 1, Translations 1 | Products, Logs, Catalogs, Bulk Operations |
| **Customers** | Customer import/export and mapping (by e-mail/phone, by bill-to, default customer), name and county formats, templates per country. | 44 | Helpers 36, Base 28, Metafields 5, Companies 1 | Order handling 6 (+14 iface), Companies 8 | Customers, Companies, Logs, Invoices, Metafields |
| **Metafields** | Metafields on products, variants, customers and companies, one codeunit per metafield type. | 43 | Helpers 29, Base 15 | Products 7, Companies 5, Customers 5 | Metafields |
| **Base** | The Shop card and setup: Shop table, Shopify Activities cues, Communication Mgt. (every HTTP call to Shopify), background syncs, install/upgrade, guided setup. | 39 | Integration 11, Helpers 8, Webhooks 6, GraphQL 3, Products 2, Logs 2, Inventory 2, Bulk Operations 1, Shipping 1, Invoicing 1, Order Fulfillments 1 | Products 71, Order handling 29, Companies 28, Customers 28, Order Fulfillments 22, Catalogs 20, Payments 20, Metafields 15, Invoicing 12, Shipping 11, Bulk Operations 10, Inventory 10, Order Returns 10, Webhooks 9, Order Refunds 8, Order Risks 5, Staff 5, Transactions 5, Translations 4, Logs 3, Gift Cards 2, Integration 2 | Order Handling, Inventory, Invoices, Base, Products, Shipping, Customers, Webhooks, Bulk Operations, Catalogs, Companies, Staff, Order Refunds |
| **Companies** | B2B companies and locations: import/export, company mapping, tax registration id mapping. | 27 | Helpers 30, Base 28, Customers 8, Metafields 5, Catalogs 1 | Order handling 4, Customers 1 | Companies, Metafields |
| **Document Links** | Links between a Shopify document and the BC documents made from it, and the 'open document' actions. | 21 | — | — | **none found** |
| **Inventory** | Stock levels to Shopify per location: Shop Locations mapping, stock calculation methods, Inventory API. | 21 | GraphQL 2 (+284 iface), Helpers 24, Base 10, Logs 1, Order Fulfillments 1 | Base 2, Order Fulfillments 2, Shipping 1 | Inventory |
| **Order Return Refund Processing** | What to do with returns and refunds: Import Only, Default, or create a Credit Memo (IReturnRefund Process). | 19 | Order handling 5, Order Refunds 2 | Order handling 4 (+12 iface), Order Refunds 4 (+12 iface) | Order Refunds |
| **Payments** | Shopify Payments: payouts, payment transactions, disputes, payment terms mapping. | 19 | Helpers 36, Base 20, Logs 2 | — | Payments |
| **Transactions** | Order transactions, payment gateways and the payment-method mapping; Suggest Payments for the cash receipt journal. | 18 | Base 5, Helpers 5, Order handling 3, Logs 1 | Order handling 1 | Payments |
| **Order Fulfillments** | Fulfillment orders and fulfillments read from Shopify; used to send shipments back. | 16 | Helpers 30, Base 22, Logs 3, Inventory 2, Gift Cards 1 | Order handling 4, Shipping 2, Base 1, Inventory 1 | Order Fulfillments |
| **Shipping** | Shipping charges and shipment-method mapping; Sync Shipments to Shopify (Export Shipments). | 15 | Helpers 18, Base 11, Order Fulfillments 2, Inventory 1, Logs 1, Order handling 1 | Base 1, Order handling 1 | **none found** |
| **Order Returns** | Returns read from Shopify. | 12 | Helpers 25, Base 10, Logs 3, Order handling 1 | Order handling 2, Order Refunds 1 | **none found** |
| **Catalogs** | B2B catalogs and catalog prices for companies and markets. | 11 | GraphQL 2 (+284 iface), Helpers 25, Base 20, Order handling 3, Logs 1, Products 1 | Companies 1 | Catalogs, Logs |
| **Logs** | Log Entries (request/response per call), Skipped Records, Data Capture. | 10 | Base 3, Helpers 1 | Order Fulfillments 3, Order Refunds 3, Order Returns 3, Base 2, Order handling 2, Payments 2, Catalogs 1, Inventory 1, Shipping 1, Transactions 1 | Logs |
| **Order Refunds** | Refunds read from Shopify. | 10 | Helpers 20, Order Return Refund Processing 4 (+12 iface), Base 8, Order handling 3, Logs 3, Order Returns 1 | Order handling 2, Order Return Refund Processing 2 | Order Refunds |
| **Bulk Operations** | Shopify bulk mutations (prices, images) and their status, finished by webhook. | 9 | Helpers 12, Base 10, Webhooks 1 | Products 6 (+4 iface), Base 1, Webhooks 1 | Bulk Operations, Webhooks |
| **Translations** | Product translations to Shopify languages. | 8 | Helpers 6, Base 4, Products 1 | Products 3 (+1 iface) | **none found** |
| **Invoicing** | Posted sales invoices exported to Shopify as orders (Sync Invoices to Shopify). | 7 | Base 12, Helpers 8 | Base 1 | Invoices, Logs |
| **PermissionSets** | The connector's permission sets. | 6 | — | — | **none found** |
| **Order Risks** | Fraud risk assessments of an order. | 5 | Helpers 6, Base 5 | Order handling 2 | Order Risks |
| **Webhooks** | Webhook subscriptions in Shopify and their handling in BC (order created, bulk operation finished). | 5 | Base 9, Helpers 3, Bulk Operations 1 | Base 6, Bulk Operations 1 | **none found** |
| **Gift Cards** | Gift cards sold and used in orders. | 4 | Helpers 3, Base 2 | Order Fulfillments 1 | Gift Cards |
| **Helpers** | Shared utilities: JSON reading, hashing, Base64, maths. Used by almost every module. | 4 | — | Products 88, Order handling 48, Customers 36, Payments 36, Companies 30, Order Fulfillments 30, Metafields 29, Catalogs 25, Order Returns 25, Inventory 24, Order Refunds 20, Shipping 18, Bulk Operations 12, Base 8, Invoicing 8, Order Risks 6, Staff 6, Translations 6, Transactions 5, Gift Cards 3, Webhooks 3, GraphQL 1, Integration 1, Logs 1 | Helpers, Order Handling |
| **Staff** | Shopify staff members mapped to BC salespeople. | 4 | Helpers 6, Base 5 | — | Staff |
| **Integration** | OAuth sign-in to a Shopify store: Authentication Mgt., the Authentication page, the registered-store table. | 3 | Base 2, Helpers 1 | Base 11 | Base |

**Finding — no test calls into:** Document Links, Order Returns, Shipping, Translations, Webhooks. (A test that only reads their tables is not counted; see *cannot see*.)

## Interfaces and their implementers

Implementers are the enums and codeunits that declare `implements`. Most interfaces are chosen by an enum value set on the Shop card.

| Interface | Module | Implementers |
|---|---|---|
| `Shpfy Extended IDocument Source` | Order Return Refund Processing | codeunit Shpfy IDocSource Refund |
| `Shpfy Extended Stock Calculation` | Inventory | **none in this app** |
| `Shpfy IBulk Operation` | Bulk Operations | codeunit Shpfy Bulk UpdateProductImage; codeunit Shpfy Bulk UpdateProductPrice; enum Shpfy Bulk Operation Type |
| `Shpfy ICompany Mapping` | Companies | codeunit Shpfy Comp. By Default Comp.; codeunit Shpfy Comp. By Email/Phone; codeunit Shpfy Comp. By Tax Id; enum Shpfy Company Mapping |
| `Shpfy ICounty` | Customers | codeunit Shpfy County Code; codeunit Shpfy County Name; enum Shpfy County Source |
| `Shpfy ICounty From Json` | Customers | codeunit Shpfy County From Json Code; codeunit Shpfy County From Json Name; enum Shpfy County Source |
| `Shpfy ICreate Translation` | Translations | codeunit Shpfy Create Transl. Product; enum Shpfy Resource Type |
| `Shpfy ICreateProductStatusValue` | Products | codeunit Shpfy CreateProdStatusActive; codeunit Shpfy CreateProdStatusDraft; enum Shpfy Cr. Prod. Status Value |
| `Shpfy ICustomer Mapping` | Customers | codeunit Shpfy Cust. By Bill-to; codeunit Shpfy Cust. By Default Cust.; codeunit Shpfy Cust. By Email/Phone; enum Shpfy Customer Mapping |
| `Shpfy ICustomer Name` | Customers | codeunit Shpfy Name is CompanyName; codeunit Shpfy Name is Empty; codeunit Shpfy Name is First. LastName; codeunit Shpfy Name is Last. FirstName; enum Shpfy Name Source |
| `Shpfy IDocument Source` | Order Return Refund Processing | codeunit Shpfy IDocSource Default; codeunit Shpfy IDocSource Refund; enum Shpfy Source Document Type |
| `Shpfy IFind Company Mapping` | Companies | codeunit Shpfy Comp. By Default Comp.; codeunit Shpfy Comp. By Email/Phone; codeunit Shpfy Comp. By Tax Id |
| `Shpfy IGraphQL` | GraphQL | 143 — codeunit Shpfy GQL AcceptFFRequest; codeunit Shpfy GQL AddImageToProduct; codeunit Shpfy GQL AddProductImage; … |
| `Shpfy IMetafield Owner Type` | Metafields | codeunit Shpfy Metafield Owner Company; codeunit Shpfy Metafield Owner Customer; codeunit Shpfy Metafield Owner Product; codeunit Shpfy Metafield Owner Variant; enum Shpfy Metafield Owner Type |
| `Shpfy IMetafield Type` | Metafields | 27 — codeunit Shpfy Mtfld Type Article Ref; codeunit Shpfy Mtfld Type Boolean; codeunit Shpfy Mtfld Type Collect. Ref; … |
| `Shpfy IOpenBCDocument` | Document Links | 10 — codeunit Shpfy Open PostedReturnReceipt; codeunit Shpfy Open PostedSalesCrMemo; codeunit Shpfy Open PostedSalesInvoice; … |
| `Shpfy IOpenShopifyDocument` | Document Links | codeunit Shpfy Open Order; codeunit Shpfy Open Refund; codeunit Shpfy Open Return; codeunit Shpfy OpenDoc NotSupported; enum Shpfy Shop Document Type |
| `Shpfy IRemoveProductAction` | Products | codeunit Shpfy RemoveProductDoNothing; codeunit Shpfy ToArchivedProduct; codeunit Shpfy ToDraftProduct; enum Shpfy Remove Product Action |
| `Shpfy IReturnRefund Process` | Order Return Refund Processing | codeunit Shpfy RetRefProc Cr.Memo; codeunit Shpfy RetRefProc Default; codeunit Shpfy RetRefProc ImportOnly; enum Shpfy ReturnRefund ProcessType |
| `Shpfy IStock Available` | Inventory | codeunit Shpfy Can Have Stock; codeunit Shpfy Can Not Have Stock; enum Shpfy Stock Calculation |
| `Shpfy Stock Calculation` | Inventory | codeunit Shpfy Balance Today; codeunit Shpfy Disabled Value; codeunit Shpfy Free Inventory; enum Shpfy Stock Calculation |
| `Shpfy Tax Registration Id Mapping` | Companies | codeunit Shpfy Tax Registration No.; codeunit Shpfy VAT Registration No.; enum Shpfy Comp. Tax Id Mapping |

**Finding — `Shpfy Extended Stock Calculation` has no implementer in this app.** It is an extension point: `Shpfy Inventory API` checks `is "Shpfy Extended Stock Calculation"` (`App/src/Inventory/Codeunits/ShpfyInventoryAPI.Codeunit.al:54`), so another app's stock calculation can implement it.

## Runtime view — what runs where

**In BC — the user's session**

| Part | What it does | Code | Status |
|---|---|---|---|
| Pages and actions | Shop card, Shopify Orders, Products, sync actions | Base, Order handling, Products | built |
| Communication Mgt. | Every HTTP call: Admin GraphQL API version 2026-01, `graphql.json` | `Shpfy Communication Mgt.` (Base) | built |

**In BC — background sessions**

| Part | What it does | Code | Status |
|---|---|---|---|
| Job queue entries | Sync reports run in the background when the shop allows it | EnqueueJobEntry in `Shpfy Background Syncs` (Base) | built |
| Webhook handling | Runs on insert of a platform `Webhook Notification` record | `Shpfy Webhooks Mgt.`, `Shpfy Webhook Notification` (Webhooks) | built |

**Outside BC**

| Part | What it does | Code | Status |
|---|---|---|---|
| Shopify Admin API | GraphQL endpoint of the store | — | built |
| Shopify webhooks | ORDERS_CREATE, BULK_OPERATIONS_FINISH sent to BC | `Shpfy Webhooks API` registers them (Webhooks) | built |
| Shopify OAuth | App install and access token | `Shpfy Authentication Mgt.` (Integration) | built |

**What talks to what**

| From | To | How | Status |
|---|---|---|---|
| Communication Mgt. (BC) | Shopify Admin API | HTTPS POST GraphQL, access token | built |
| Shopify | BC Webhook Notification | HTTPS webhook → platform table insert → event subscriber | built |
| Webhooks Mgt. (BC) | Job queue | enqueues `Shpfy Sync Orders from Shopify` | built |
| Shop card (BC) | Shopify OAuth | browser pop-up, then token stored for the shop | built |

## Main flows

### Order → sales order

*Order: binding order: import before mapping before document — set by the code (`Shpfy Sync Orders from Shopify`), not by a spec.*

1. Shopify Orders → **Sync Orders From Shopify** (report `Shpfy Sync Orders from Shopify`, Order handling) — or the job queue, or the ORDERS_CREATE webhook (Webhooks → `Shpfy Webhooks Mgt.` enqueues the same report).
2. GetOrdersToImport in `Shpfy Orders API` (Order handling) → ExecuteGraphQL in `Shpfy Communication Mgt.` (Base) → GraphQL query codeunit → Shopify Admin API.
3. `Shpfy Import Order` (Order handling) writes the Shopify order tables; uses `Shpfy Order Fulfillments`, `Shpfy Fulfillment Orders API`.
4. DoMapping in `Shpfy Order Mapping` (Order handling) finds or creates the customer (Customers / Companies) and the items (Products).
5. If **Auto Create Orders**: `Shpfy Process Order` (Order handling) creates the sales header and lines, releases the order if set, and writes the BC document number back as an order attribute (AddOrderAttribute in `Shpfy Orders API`).

### Products Shopify ⇄ BC

1. Shop card → Sync Products (report `Shpfy Sync Products`, Products) → `Shpfy Sync Products` codeunit.
2. Direction from **Sync Item**: `Shpfy Product Import` (Shopify → BC: `Shpfy Create Item`, `Shpfy Update Item`) or `Shpfy Product Export` (BC → Shopify).
3. Shopify calls through `Shpfy Product API` / `Shpfy Variant API` → Communication Mgt. → GraphQL.
4. Large price or image updates go through Bulk Operations (`Shpfy Bulk Operation Mgt.`), finished by the BULK_OPERATIONS_FINISH webhook.

### Stock BC → Shopify

1. Report `Shpfy Sync Stock to Shopify` (Inventory) runs `Shpfy Sync Inventory` per shop location.
2. `Shpfy Inventory API` calculates stock with the location's Stock Calculation (`Shpfy Stock Calculation` interface; `Shpfy Extended Stock Calculation` if an implementer supports it) and sends it.

### Shipment BC → Shopify

1. Post the sales shipment in BC.
2. Report `Shpfy Sync Shipm. to Shopify` (Shipping) → `Shpfy Export Shipments` → `Shpfy Fulfillment Orders API` (Order Fulfillments) creates the fulfillment in Shopify.

## Entry points and integrations

- **Calls in:** user actions and reports on the Shop card and Shopify pages; job queue entries; Shopify webhooks (via the platform `Webhook Notification` table); the 70 integration events other apps can subscribe to.
- **Calls out:** Shopify Admin GraphQL API (version `2026-01`, `Shpfy Communication Mgt.`); Shopify OAuth (`Shpfy Authentication Mgt.`); Base Application documents (sales orders, invoices, credit memos, shipments, customers, items).
- **User guides:** `docs/manuals/` (setup, daily use); test runbook: `docs/testing/shopify-to-bc-e2e-test.md`.

# Functional Specification — SITECH D365 CE (Sales + Support UX)

**Document type**: Functional Specification  
**Author role**: Business System Analyst  
**Solution scope**: D365 CE Sales Hub + SitechSupportUX v1.0.0.3 + SitechUX v1.0.0.1 + SitechSupportData v1.0.0.2  
**Implementer**: Axelio B.V. (Marijke Kerkhove / Renger Reitsema)  
**Functional owner (Sitech)**: René van der Linden  
**Platform**: Microsoft Dynamics 365 CE / Power Platform / Business Central  
**Date**: 2026-06-05 (enriched with all source documents including managed solution packages)  
**Version**: 1.3

---

## 1. Executive Summary

Sitech Netherlands is implementing Microsoft Dynamics 365 Customer Engagement (CE) in two modules:

1. **D365 CE Sales Hub** — supports the commercial process: lead management, opportunity tracking, quotation, order confirmation, and invoice visibility.
2. **SitechSupportUX** — extends D365 Customer Service with asset-aware case management, product/BoM tracking per case, and two Power Apps Canvas applications for service engineers.

Business Central (BC) remains the **single source of truth** for all financial master data (customers, products, prices). CE orchestrates the commercial and service process on top of BC data.

---

## 2. Business Context

### 2.1 Background

Sitech provides technical support and sales for specialised construction/surveying machinery (Trimble, etc.). Their current CRM lacks:
- Structured lead and opportunity management linked to equipment types.
- Quotation and order flow connected in real time to BC.
- Asset-centric service case management (knowing which machine a call is about, what components it contains, what spare parts are needed).

### 2.2 Business Objectives

| # | Objective | Module |
|---|-----------|--------|
| 1 | Digitalise the Lead → Opportunity → Quote → Order flow in CE. | Sales |
| 2 | Synchronise BC master data (customers, products, prices) into CE as read-only. | Sales |
| 3 | Push accepted Sales Orders from CE to BC in real time. | Sales |
| 4 | Link every service case to a registered customer asset (machine). | Support |
| 5 | Track bill-of-materials requirements per case for CE→BC fulfillment. | Support |
| 6 | Provide an asset-browser Canvas App to speed up case creation. | Support |
| 7 | Give sales agents visibility into cases, service items, and contracts on the Account. | Both |

---

## 3. Scope

### 3.1 In Scope

**Sales module**
- D365 CE Sales Hub configuration and customisation.
- BC ↔ CE integration for accounts, contacts, products/BoM, price lists, orders, invoices.
- SharePoint document management per customer.
- Security roles for Sales.
- Personal and team dashboards.

**Support / Customer Service module (SitechSupportUX v1.0.0.2)**
- Custom entities: Case Product Lines, Machine Brand/Model/Type reference data, Service BoM, Service Item Components.
- Extensions to Incident (Case) and Customer Asset entities.
- Two Power Apps Canvas applications (Customer Assets Exploded View, Service Item Components).
- Modified Case forms (7 variants).

### 3.2 Out of Scope

| Item | Status |
|------|--------|
| D365 Field Service | Separate track |
| Marketing module | Decision open (Dieuwertje) |
| Power BI reporting | Separate track |
| Goals / Forecasting in CE | Decided out — Power BI only |
| CE → BC integration middleware/connector logic | External to this solution package |

---

## 4. Actors and Personas

| Actor | Description | Primary Modules |
|-------|-------------|----------------|
| **Account Manager** | Creates and manages Leads, Opportunities, Quotes, Orders. Cannot create Customers (only Prospects). | Sales |
| **Sales Manager** | Same as Account Manager + team reporting + approves new customers. | Sales |
| **Director** | Read-only view at team level, equivalent to Sales Manager. | Sales |
| **Service Agent (Inside)** | Creates and manages cases, links assets, records time and product requirements. | Support |
| **Field Engineer** | Updates cases post-visit, uses asset exploded view. | Support |
| **Service Manager** | Oversees queue and case routing. | Support |
| **Sales Admin (Axelio)** | Full administrative access during implementation. | Both |
| **Back-end System (BC)** | Source of master data; consumer of Sales Orders. | Integration |

---

## 5. Sales Module — Functional Requirements

### 5.1 Sales Process

The standard D365 sales pipeline is adopted unchanged:

| Step | Entity | Description |
|------|--------|-------------|
| 1 | Lead *(optional)* | Qualify or disqualify a new contact/company. |
| 2 | Opportunity | Record sales chance with customer, contact, value, and expected close date. |
| 3 | Quote | Create quotation from opportunity; version management (Revise) supported. |
| 4 | Order | Accepted quote converted to order; order pushed to BC in real time. |
| 5 | Invoice | Created in BC; synced to CE as read-only. |

> **Open decision (V-01)**: Sitech to define internally when a Lead is created versus starting directly with an Opportunity or Activity.

### 5.2 Account Management

**FR-S01 — Customer vs. Prospect Distinction**  
An Account Manager can only create a **Prospect** record. The relationship type **Customer** requires approval from the Sales Manager. When a Quote is converted to an Order, the status automatically transitions from Prospect to Customer, provided Sales Manager approval has been granted.

**FR-S02 — Account Form Tabs**  
The Account form must include the following tabs:

| Tab | Source | Type | Priority |
|-----|--------|------|----------|
| Opportunities | CE | Related records | Must have |
| Quotes | CE | Related records | Must have |
| Orders | CE | Related records | Must have |
| Invoices | BC → CE sync | Read-only | Must have |
| Cases | CE Customer Service | Read-only for Sales | Must have |
| Service Items | BC → CE sync | Sales form (installation level only) | Must have |
| Documents | SharePoint | SharePoint integration | Must have |
| Customer Contracts/Subscriptions | BC → CE sync | Read-only; fields TBD | Wish |

**FR-S03 — BC Read-Only Fields on Account**  
The following fields are synchronised from BC and locked in CE:
- Customer name, address details
- Payment terms
- Discount group (value, e.g. "15%")
- Parent-child account relationship
- Active/inactive status (blocked customer → deactivate in CE)

**FR-S04 — Country/Region Lookup**  
A custom **Country** table is created in Dataverse, populated by BC sync. Address fields use a lookup to this table. A workflow/automation updates the `Address1_Country` text field with the correct country code on selection, ensuring BC integration does not fail.

**FR-S05 — Account Hierarchy View**  
Where accounts have a parent (HQ) relationship, the system must represent this in a hierarchical list/view (e.g., Heijmans group structure).

**FR-S06 — Account Manager Assignment**  
Each Account must be linked to an owner (Account Manager). The ownership mechanism and its interaction with the BC sync must be defined.

**FR-S07 — Additional Fields on Account (to be confirmed with René)**  
The following fields were agreed in earlier sessions and need confirmation of inclusion:
- Sales Instructions (free text)
- Work Order Instructions (or in BC — decision pending)
- Customer Discounts section
- Finished Goods / Spare Parts / Services tab/section
- Segment field

### 5.3 Contact Management

**FR-S08 — Contact Form Simplification**  
Address fields are removed from the Contact form (already present on the Account). Additional fields to be determined with Sitech.

**FR-S09 — Salutation Field**  
A Salutation field must be added to the Contact form.

**FR-S10 — Duplicate Detection**  
Duplicate detection rules are configured for Accounts and Contacts. Fields for detection rules to be agreed with Sitech.

### 5.4 Lead Management

**FR-S11 — Lead Sources**  
The Lead Source field must be populated with: Referral, Website, Exhibition, Inbound Call, Outbound Call, E-Mail, Social Media, Other.

**FR-S12 — Lead Qualification**  
Leads can be qualified to Opportunities by: salesperson, team lead, or Account Director. Qualification criteria to be defined.

**FR-S13 — Lead Disqualification Reasons**  
Status reasons for disqualified Leads must be configured (list to be defined with Mark/Sales Manager).

**FR-S14 — Stale Opportunity Alerting**  
Mechanism to flag Opportunities that have had no activity for a configurable number of days.

### 5.5 Opportunity Management

**FR-S15 — Machine Information on Opportunity**  
The Opportunity form must include a Machine Info section with:
- Type
- Brand
- Reference
- Model
- Specials (yes/no)
- No follow-ups (separate field)
- Add to Summary Competitor

**FR-S16 — Competitor Tracking**  
Competitors field on Opportunity, populated with: None, Leica Geosystems, Unicontrole, CHC-NAV, Topcon, OEM, Others.

**FR-S17 — Lost Reasons for Opportunities**  
Status reasons for lost Opportunities: Competitor, Price, Solution fit, Timelines.

**FR-S18 — Status Reasons for Lost Quotes**  
Separate status reason set for lost/expired Quotes (list to be defined).

**FR-S19 — Parent Order for Upselling**  
A "Parent Order" field on the Order entity to track which orders originated from upsell actions, with a view showing orders that became parent to other orders.

### 5.6 Quote Management

**FR-S20 — Products on Separate Tab**  
The quote line grid is placed on a dedicated "Products" tab for readability when there are many lines.

**FR-S21 — Remove Shipping Information**  
The Shipping information section is removed from the Quote form. Payment terms are inherited from the Account (from BC).

**FR-S22 — Discount at Total Level Only**  
Discount is applied at the order total level only — as an amount (e.g., €100) or as a percentage (e.g., 10%). No per-line percentage discount.

**FR-S23 — No Write-In Products**  
Write-in products are disabled. Sales can only select products/bundles from the synchronised price list.

**FR-S24 — Trade-In Lines (Negative Quantity)**  
Trade-in products are added to quotes/orders with a negative quantity (e.g., -1 × trade-in item = -€500). CE renders the amount in parentheses. BC behaviour with negative lines must be tested before go-live.

**FR-S25 — Word Template / Create PDF Button**  
A Word template for quotation output and a "Create PDF" button must be available on the Quote form.

**FR-S26 — Trimble Products Without Write-In**  
Trimble third-party products (handled by Arie) must be accommodatable without enabling write-in. Proposed solution: generic product with editable description and price; BoM loaded via Excel upload in Assembly. This approach to be confirmed.

### 5.7 Product Structure

**FR-S27 — Two-Layer Product Structure**  
Products in CE follow the BoM structure from BC:

| Layer | CE Type | Description | Price |
|-------|---------|-------------|-------|
| Layer 1 | Bundle (EWV product) | Main article / installation level; visible to customer on quote | €0 (sum of SBUs) |
| Layer 2 | Product (SBU) | Sub-components, each with rounded price | Rounded (BC calculation) |

**FR-S28 — Optional vs. Required Products in Bundles**  
Products in bundles must be classified as:
- **Required**: zero-price, cannot be removed by sales agent.
- **Optional**: price visible, can be removed.

A custom field `required/optional` must be added. Synchronisation of this flag from BC BoM needs to be verified.

**FR-S29 — Product Filtering by Item Category**  
Products visible to Sales agents are filtered by BC Item Category Code = 'Sales'.

### 5.8 Pricing

**FR-S30 — Price Lists**  
Three price lists to be maintained in BC and synced to CE:
1. Standard customer price list.
2. OEM gross price list.
3. OEM net price list.

> **Decision required (B3)**: Whether OEM price lists are synchronised to CE or only used in BC.

**FR-S31 — Discount Group on Account**  
The discount group value (e.g., "15%") is synchronised from BC to the Account as read-only.

### 5.9 Order Management

**FR-S32 — Order to BC (Real-Time)**  
On quote acceptance, an Order is created in CE and pushed to BC in real time.

**FR-S33 — Order Status: Submitted / Read-Only**  
Once an Order is synchronised to BC, it receives status "Submitted" and becomes read-only in CE.

**FR-S34 — Lock Order for Operations**  
A mechanism to "lock" a Sales Order for the operations/bedrijfsbureau department. Tested as feasible; internal Sitech decision on use is pending.

**FR-S35 — Multiple Installations on One Order**  
The ability to include 2+ installations on a single Sales Order. Feasibility in BC confirmed; CE behaviour must be verified.

### 5.10 Invoice Management

**FR-S36 — Invoice Visibility in CE**  
Invoices are created in BC and synchronised as read-only records to CE. Visible on the Account (Invoices tab) and on the Order.

**FR-S37 — Pre-Invoicing for Installations**  
Installations are invoiced at a fixed price upon delivery. The order may not yet be complete (material returns, hour adjustments). Pre-invoicing must be supported; the final booking in BC follows later.

### 5.11 Customer Contracts / Subscriptions

**FR-S38 — Subscriptions Sync to CE (Read-Only)**  
A custom **Subscriptions** table is created in Dataverse. Records are created in BC and synced read-only to CE. Visible on the Account (Customer Contracts/Subscriptions tab). Fields to be confirmed with René (action V-04).

**FR-S39 — VRS Subscription Placement**  
VRS subscriptions sold per installation must be linked to the **main location** (Account), not automatically placed under the installation (Service Item). The current automatic linking to the installation via Service Item is undesired.

> **Open question**: Can VRS subscriptions be sold stand-alone only? Invoice to branch vs. head office?

### 5.12 SharePoint Integration

**FR-S40 — Auto-Create Customer Folder**  
When an Account is created in CE, a SharePoint folder is automatically created (Power Automate flow). Folder name = Account name.

**FR-S41 — Auto-Rename Folder**  
When the Account name changes in CE, the SharePoint folder name is updated automatically.

**FR-S42 — Documents Tab on Account and Quote**  
A Documents tab is added to Account and Quote forms, providing direct access to the SharePoint folder. The Quotes tab links to the Offertes subfolder within the customer folder.

**FR-S43 — Service Item SharePoint Folders**  
When a Service Item is created, a SharePoint subfolder is automatically created using the naming convention:  
`<YYYYMMDD> - <Customer> - <Assembly Order Number> - <Installation Name>`  
Example: `20260227 - Dura Vermeer - A00018 - CAT 330-340 LRE Generic - Trimble Earthworks`

**FR-S44 — Service Order Subfolder**  
When a Service Order is created, a subfolder is created within the Service Item folder, named with the Service Order date. Photos are stored here.

### 5.13 Module Navigation

The following items are removed from the Sales module navigation:

| Item | Reason |
|------|--------|
| Multisession | Relevant only for Customer Service |
| Sales Accelerator | Not in use; complexity without benefit |
| AI Research Agent | Not stable; may be reconsidered later |
| Competitors | Not in use (handled via Opportunity field) |
| Goals / Performance | Forecasting via Power BI only |
| Marketing module | Decision open (Dieuwertje) |

### 5.14 Dashboards

**FR-S45 — Personal Dashboard**  
- My open Opportunities (filtered by owner)
- My open Quotes
- My open activities (tasks, appointments, phone calls) this week

**FR-S46 — Team Dashboard**  
- Open Opportunities per sales person
- Active Quotes and Orders (overview)
- Overdue activities per user

**FR-S47 — Account Activity Metrics**  
Red-flag metrics to be surfaced (via Power BI or CE dashboards):
- Days since last order (30/60/90 day flags)
- Frequency of orders
- Revenue per customer / segment
- Average order value
- Competitors per company
- Red flags (no calls/emails for X days)

---

## 6. Support / Customer Service Module — Functional Requirements

### 6.1 Asset Linkage on a Case

**FR-C01 — Link Asset to Case**  
When creating or updating a Case, the agent **must** select a Customer Asset (`sit_asset`). Required.

**FR-C02 — Link Sub-Asset to Case**  
The agent may optionally select a Sub-Asset (`sit_subasset`), filtered to child assets of the selected asset.

**FR-C03 — Asset & Installation Info Display**  
Contextual installation information based on the selected asset is displayed inline on the Case form.

### 6.2 Case Lifecycle

**FR-C04 — Case Status Reasons**  
The following status model applies:

| Active | Resolved | Cancelled |
|--------|----------|-----------|
| In Progress | Problem Solved | Cancelled |
| On Hold | Information Provided | Merged |
| Waiting for Details | | |
| Researching | | |

Status reasons must also indicate how the case was closed: via Sales, Service Inside, or Field.

**FR-C05 — Short vs. Normal Case Registration**  
- **Short contact**: Registered via Contact → Timeline → Phone call activity. No case record created.
- **Normal case**: Contact → Related Cases → Quick Create form → Case record.

Not every interaction becomes a case.

**FR-C06 — Merge / Similar / Parent-Child Cases**  
Decision required on whether to enable: Merge cases, Similar Cases, and Parent-Child case functionality. If not needed, these are excluded from the UX and business logic.

### 6.3 Total Time Tracking

**FR-C07 — Record Support Duration**  
Agents record total time spent in `sit_totaltime` (integer, minutes). AI form-fill is disabled for this field; manual entry only.

### 6.4 Case Product Lines

**FR-C08 — Add Products to Case**  
Agents create Case Product Line records (product + quantity) linked to the case. Used for CE→BC product fulfilment.

**FR-C09 — Product Lines Sub-Grid**  
A sub-grid on the Case form lists all Case Product Lines with inline add/edit.

### 6.5 Customer Asset Exploded View (Canvas App 1)

**FR-C10 — Search by Serial / Part Number**  
Agent searches by serial number or part number; app returns matching Customer Assets.

**FR-C11 — Browse Service BoM**  
For a selected asset, displays the Service Bill of Materials gallery.

**FR-C12 — Browse Service Item Components**  
For a selected asset, displays components (Product ID, Name, Serial Number, Quantity).

**FR-C13 — Expand Installation**  
Agent expands an installation to reveal sub-components.

**FR-C14 — Create Case from Asset Context**  
Agent initiates case creation pre-populated with the selected asset.

### 6.6 Service Item Components App (Canvas App 2)

**FR-C15 — Browse Customer Assets**  
Agent navigates to a customer asset via lookup.

**FR-C16 — Staged Product List**  
Agent selects service item components and adds them to a staging list before committing as Case Product Lines.

### 6.7 Service Items (Customer Assets)

**FR-C17 — Two Service Item Form Types**  
- **Sales form**: Installation level only (first layer); used by Sales.
- **Service form**: Full view including sub-layers; used by Customer Service / Field Service.

The Sales form is shown on the Account tab in the Sales module.

**FR-C18 — Asset Classification**  
Each Customer Asset is classifiable by Machine Brand, Machine Model, and Machine Type (reference entities).

**FR-C19 — Service Item Types**  
Two types: Installation (top-level) and underlying Service Items. Decision required on whether all serialised items become Service Items or only installations.

**FR-C20 — Machine Driver (Asset ↔ Contact)**  
A mechanism to associate a Contact (machine operator/driver) with a specific asset. The unique number concept for machine drivers needs to be defined.

### 6.8 Queues and Routing

**FR-C21 — Shared Mailbox Queue**  
Cases arriving via the shared support email address are routed through a queue. *(Note: this reverses an earlier decision of "do not use queues" from Nov 2025.)*

### 6.9 Knowledge Base

**FR-C22 — Knowledge Articles**  
A knowledge base is maintained in CE, linked to cases. AI suggests relevant articles when closing a case. Knowledge articles are accessible from the Sales module as well.

### 6.9a Credits — Partially Implemented in Code

> **Note**: The `sit_Credits` and `sit_Credithours` entities are **already built** in `SitechUX v1.0.0.1` but have no functional design document. The specification below is reverse-engineered from the code.

**FR-C-CR01 — Credit Transaction Records (`sit_Credits`)**  
Stores individual credit grant/deduction events per Account:
- `sit_account` (Lookup → Account, required) — the customer
- `sit_id` (text, required) — credit transaction reference
- `sit_quantity` (integer, required) — number of credits in this transaction (positive = added, presumably)

**FR-C-CR02 — Credit Hours Balance (`sit_Credithours`)**  
Stores the rolling credit balance per Account:
- `sit_account` (Lookup → Account, required) — the customer
- `sit_credits` (integer) — current credit value
- `sit_quantity` (integer, required) — quantity (hours or units)
- `sit_lastorder` (Lookup → SalesOrder) — reference to the most recent order that affected the balance

> **Open design questions** (not defined anywhere):
> - What triggers a `sit_Credits` record — a Sales Order, a manual entry, a case resolution?
> - How does `sit_Credithours` balance update — via Power Automate, a plugin, or manual?
> - What is the relationship between `sit_Credits` (transactions) and `sit_Credithours` (balance) — are they separate ledger/balance concepts?
> - What is the threshold below which support becomes chargeable?
> - How are these entities surfaced to Support agents on the Case or Account form?

### 6.10 Credits and Entitlements

**FR-C23 — Entitlements**  
Entitlements are 1-year time-limited contracts that restrict the number of cases a customer can open and control whether a new case may be created. Can be scoped at Account and/or Asset level.

**FR-C24 — Credits**  
Credits are a pool of support hours at the customer level (e.g., 20 hours). They are consumed only after a free threshold is exceeded. "Covered under warranty" until hours > allowance.

> **Critical open decision**: Credit balance by Account or by Installation? How to invoice for negative credit balance?

### 6.11 SLAs

**FR-C25 — SLA Decision**  
Decision required: will D365 SLA functionality be used? If yes, define SLA levels and targets.

### 6.12 Case-to-Service Order

**FR-C26 — Convert Case to Service Order in BC**  
From a resolved case, a Service Order is created in BC. Three types: SE (Service), SI (Internal Service), SD (Service Delivery). The Work Order route in BC is explicitly excluded (confirmed 26-05-2026).

> **Open question**: Can one case be converted to a Service Order more than once?  
> **Open question**: Should BC create final case activities after a Sales Order or Service Order is completed?

---

## 6a. Training Module — SITECH Academy

> **Status**: This module is **fully implemented and live in production** since May 2024 (flow `clientCreatedTime: 2024-05-14`). It is the **SITECH Academy** customer-facing training management system. It does **not appear in any functional design document** — this specification is reverse-engineered from the solution schema and the live Power Automate flow definition.

### Purpose

Manages SITECH Academy product training sessions and automates participant communication. Customers register for training sessions; the system confirms registration by email (in Dutch), sends reminders, and records certificate issuance.

### Entities

**`sit_TrainingSession`** — A scheduled training event:

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `sit_description` | Text | Yes | Training name/description |
| `sit_date` | DateTime | No | Date of session (UTC, displayed W. Europe Standard Time) |
| `sit_location` | Text | No | Physical or virtual location |
| `sit_time` | Text | No | Time of session (stored as text, referenced as `sit_trainingsessiontime` in flow — see GAP-X09) |
| `sit_passed` | Boolean | No | Whether the session was passed (overall) |
| `sit_recorded` | Boolean | No | Whether the session was recorded |
| `sit_trainer` | Lookup → SystemUser | Recommended | Internal SITECH trainer |
| `sit_training` | Lookup → Product | Recommended | Training product from the product catalogue |

**`sit_ContactTraining`** — Junction: a Contact's registration at a Training Session:

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `sit_name` | Text | Yes | Record name |
| `sit_contact` | Lookup (field: `_sit_contactid_value`) | Yes | Links to `Contact` entity — the registered participant. Relationship XML name `sit_contacttraining_Contact_contract` is misleading but field stores Contact IDs (confirmed by live flow logic). |
| `sit_trainingsession` | Lookup → sit_TrainingSession | No | Which session this registration is for |
| `sit_certificatesant` | Boolean | No | Certificate sent flag |
| `sit_confirmationsent` | Boolean | No | Confirmation email sent flag (set to `true` by automation) |
| `sit_remindersent` | Boolean | No | Reminder email sent flag |

> **Note on `sit_contact`**: The relationship XML name `sit_contacttraining_Contact_contract` was flagged as a potential bug. However, the live Power Automate flow uses `triggerBody()?['_sit_contactid_value']` and reads from the `contacts` entity — confirming this field correctly stores Contact GUIDs. The XML relationship name is misleading but the data is correct. Verify in environment to rule out dual-registration between entity types.

### Power Automate Automation

#### Flow 1: "When Training Participants is added - send Confirmation"

**Flow ID**: `6978013a-40ff-4a33-927d-485124d65c53`  
**Created**: 2024-05-14 | **Last modified**: 2026-03-04 | **Suspended since**: 2026-03-03 ⚠️  
**Connection owner**: `alina.zhirkova@sitech-nederland.nl` (personal credential — see GAP-X10)

**Trigger**: New `sit_contacttraining` row added (Dataverse webhook, scope = Organisation)

**Logic**:

```
1. Get Training Session (sit_trainingsessions)
2. Get Training Product name (products via _sit_trainingid_value)
3. CONDITION — send only if ALL true:
     a. sit_date >= utcNow()           (future training only)
     b. productid ≠ b24df9ab-0a7a-ec11-8d21-000d3aafc422   (excluded type 1)
     c. productid ≠ b4632176-8769-ed11-9561-6045bd895dae   (excluded type 2)
4. IF true:
     a. Get Trainer (systemusers via _sit_trainerid_value)
     b. Get Contact data (contacts via _sit_contactid_value)
     c. Get Academy logo from SharePoint
        Site: zeppelinholdings.sharepoint.com/sites/GP-SITECHWerkvoorbereiding
        Path: /Shared Documents/General/Academy_pic.png
     d. Fetch "Academy newsletters" queue (by name, FetchXML)
     e. Compose HTML email (Dutch, responsive, SITECH Academy branding)
     f. Create Email activity (To: Contact, From: Academy newsletters queue)
     g. Send Email (Microsoft.Dynamics.CRM.SendEmail)
     h. Update sit_contacttraining: sit_confirmationsent = true
5. ELSE: no action (silently skip excluded products)
```

**Email content** (Dutch):
- Subject: *"Bevestiging: Uw toekomstige training bij de SITECH Academy!"*
- Body includes: course name, participant full name, date (dd-MM-yyyy), time, location
- Cancellation policy: notify at least 1 week before; costs charged if cancelled within 4 days or no-show
- Contact: `academy@sitech-nederland.nl` / +31 493 68 10 00
- Footer: SITECH Academy logo (base64-encoded from SharePoint), link to `www.sitechsolutions.com/nl/training`

**Excluded product GUIDs** (no confirmation email sent):
| GUID | Likely meaning |
|------|---------------|
| `b24df9ab-0a7a-ec11-8d21-000d3aafc422` | Unknown — possibly internal/free training type (undocumented) |
| `b4632176-8769-ed11-9561-6045bd895dae` | Unknown — possibly another internal type (undocumented) |

> ⚠️ These GUIDs are hardcoded in the flow condition. If the Product records are deleted and recreated, the GUIDs will change and the exclusion will silently stop working.

#### Flow 2: "Copy of - Get upcoming training sessions"

**Flow ID**: `5aba4c9d-4af9-481d-85fb-5eedefe0cb9c` (per manifest)  
**Display name**: "Copy of - Get upcoming training sessions"  
**Connection owner**: `alina.zhirkova@sitech-nederland.nl`

> Full definition not yet analysed. The "Copy of -" prefix indicates this is a duplicated flow, not a canonical version. The original "Get upcoming training sessions" flow presumably queries `sit_trainingsessions` and sends reminder emails or a digest. **Requires analysis** — see GAP-X11.

### Functional Requirements

**FR-TR01 — Create Training Sessions**  
A Training Session is a schedulable event linked to a training Product in the D365 product catalogue, assigned to an internal trainer, with date, time, and location.

**FR-TR02 — Register Contact Attendance**  
When a Contact Training record is created (participant registration), the automation flow immediately sends a confirmation email to the Contact in Dutch, unless the training product is excluded or the training date has already passed.

**FR-TR03 — Training Product Catalogue**  
Training offerings are modelled as Products in the D365 product catalogue. At least two product types exist that are excluded from automated confirmation (GUIDs must be documented and treated as configuration, not hardcoded logic).

**FR-TR04 — Confirmation Email via Queue**  
Confirmation emails are sent from the "Academy newsletters" queue (shared mailbox), not from a personal account. The queue is resolved by name via FetchXML at runtime.

**FR-TR05 — Academy Logo from SharePoint**  
The email template includes the SITECH Academy logo, fetched at send-time from SharePoint (`GP-SITECHWerkvoorbereiding` site). Logo must remain at the defined path or the flow will fail.

**FR-TR06 — Confirmation Flag**  
After a confirmation email is sent, `sit_confirmationsent` is set to `true` on the `sit_ContactTraining` record.

**FR-TR07 — Upcoming Sessions (Reminder/Digest)**  
A second flow ("Get upcoming training sessions") handles upcoming session notifications or reminders. Its exact logic requires analysis.

---

## 7. Integration Catalogue

| ID | Type | Data | Direction | Priority |
|----|------|------|-----------|----------|
| IF-01 | BC ↔ CE | Accounts (Customers) | BC → CE | Must have |
| IF-02 | BC ↔ CE | Contacts | BC → CE | Must have |
| IF-03 | BC ↔ CE | Products / BoM (EWV + SBU) | BC → CE | Must have |
| IF-04 | BC ↔ CE | Price lists with SBU numbers | BC → CE | Must have |
| IF-05 | BC ↔ CE | Payment terms (on Account) | BC → CE (read-only) | Must have |
| IF-06 | BC ↔ CE | Discount groups (on Account) | BC → CE (read-only) | Must have |
| IF-07 | BC ↔ CE | Parent-child account relationship | BC → CE | Must have |
| IF-08 | CE → BC | Sales Order | CE → BC (real-time) | Must have |
| IF-09 | BC ↔ CE | Invoices | BC → CE (read-only) | Must have |
| IF-10 | BC ↔ CE | Country table | BC → CE | Must have |
| IF-11 | BC ↔ CE | Customer Contracts / Subscriptions | BC → CE (read-only) | Wish |
| IF-12 | CE ↔ SharePoint | Documents per customer | Bidirectional | Must have |
| IF-13 | CE ↔ Outlook | Email tracking | Standard / Unified Interface | Must have |

### 7.1 Known Integration Risks

- **Discount mapping**: CE applies total-level discounts; BC works with line/invoice percentages. The mapping in the CE→BC sync must be verified technically.
- **Trade-in (negative lines)**: BC behaviour with negative order lines must be tested before go-live.
- **Unapproved customer order submission**: Error handling when an Order is submitted to BC for a customer not yet approved. Must produce a clear error message.
- **Subscription-from-BoM**: When a Subscription (SIM/Software) is a BoM component, CE cannot automatically link it. Workaround required.
- **SD-order without synced products**: An SD-type Service Order from Support currently has no value without product synchronisation. Blocking issue requiring resolution.
- **SharePoint reconnection on name change**: When an Account name is changed, the SharePoint folder mapping can break. Manual re-linking procedure to be documented.
- **Existing Service Items migration**: When importing legacy Service Items from the old CE, SharePoint links are lost. A Power Automate migration flow to rename folders to the new naming convention is required.

---

## 8. Data Model

### 8.1 Custom Entities (Support UX)

| Entity | Purpose |
|--------|---------|
| `sit_CaseProductLine` | Products required per case |
| `sit_MachineBrand` | Reference — brand |
| `sit_MachineModel` | Reference — model |
| `sit_MachineType` | Reference — type |
| `sit_ServiceBillofMaterials` | Installation BoM |
| `sit_ServiceItemComponents` | Component-level parts list |

### 8.2 Custom Entities (SitechUX — Shared / Sales)

| Entity | Schema Name | Purpose |
|--------|-------------|---------|
| Country reference | `sit_Country` | Country + country code; synced from BC; used in Account address lookups |
| Credits (transactions) | `sit_Credits` | Credit grant/deduction events per Account |
| Credit hours (balance) | `sit_Credithours` | Rolling support-hour credit balance per Account; links to last SalesOrder |
| Training session | `sit_TrainingSession` | Schedulable training events linked to a Product and trainer |
| Contact training | `sit_ContactTraining` | Contact attendance at a training session; tracks comms sent |
| Subscriptions | *(not yet in code)* | Customer contracts/subscriptions; read-only from BC — **not yet built** |

### 8.3 Key Fields on Incident (Case)

| Field | Schema Name | Type | Required |
|-------|-------------|------|----------|
| Asset | `sit_asset` | Lookup → msdyn_customerasset | Yes |
| Sub-Asset | `sit_subasset` | Lookup → msdyn_customerasset | No |
| Total Time | `sit_totaltime` | Integer (minutes) | Recommended |

### 8.4 Relationship Diagram

```
BC (source of truth)
  ├── Accounts ──────────────────────────────► CE Account
  │     ├── Payment terms (read-only)
  │     ├── Discount group (read-only)
  │     └── Parent-child
  ├── Products / BoM (EWV+SBU) ─────────────► CE Product Catalogue
  ├── Price Lists ────────────────────────────► CE Price Lists
  ├── Sales Order ◄──────────────────────────── CE Order (real-time push)
  ├── Invoices ──────────────────────────────► CE Invoice (read-only)
  └── Subscriptions ─────────────────────────► CE Subscriptions (read-only)

CE Account
  ├── Opportunities (1:N)
  ├── Quotes (1:N)
  ├── Orders (1:N)
  ├── Invoices (read-only)
  ├── Cases (Customer Service)
  ├── Service Items (msdyn_customerasset)
  │     ├── sit_machinebrand ──► sit_MachineBrand
  │     ├── sit_machinemodel ──► sit_MachineModel
  │     ├── sit_machinetype  ──► sit_MachineType
  │     └── Parent/Child (self-ref)
  ├── Documents (SharePoint)
  └── Subscriptions (read-only)

Incident (Case)
  ├── sit_asset ──────────────────────────────► msdyn_customerasset
  ├── sit_subasset ───────────────────────────► msdyn_customerasset (child)
  └── sit_CaseProductLine (1:N)
        ├── sit_case ─────────────────────────► incident
        └── sit_product ──────────────────────► product
```

---

## 9. Security and Roles

### 9.1 Approach

- Start from "all-off" permission set (all rights zeroed).
- Assign exactly the required permissions per role.
- Based on the 100-series permission templates prepared by Maud.
- Axelio configures; René can adjust forms afterwards.
- User personalisation is **disabled** so all users with the same role see the same screen.

### 9.2 Sales Roles

| Role | Description | Notes |
|------|-------------|-------|
| Account Manager | Full access to Sales module; create/edit Leads, Opportunities, Quotes, Orders. | No access to financial settings. Cannot create Customers (only Prospects). |
| Sales Manager | Same as Account Manager + team dashboards and reports + customer approval. | No CRM admin rights. |
| Director | Same as Sales Manager, read-only at team level. | To be confirmed separately. |
| Sales Admin (Axelio) | Full management of Sales module. | Temporary during implementation. |

Based on Microsoft standard roles:
- Sales Manager → **Sales Professional Manager**
- Account Manager → **Sales Professional Person**

Custom entity permissions:

| Entity | Create | Read | Update | Delete |
|--------|--------|------|--------|--------|
| Country | — | ✓ | — | — |
| Subscriptions | — | ✓ | — | — |
| Service Items | — | ✓ | — | — |

### 9.3 Support / Customer Service Roles

Custom security roles for the Support module are **not yet defined** in the delivered solution package. Roles needed for: Case Product Lines, Service BoM, Service Item Components, Machine reference data.

---

## 10. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Platform | Microsoft Dynamics 365 CE, CRM 9.2.26051.150+; Business Central |
| Language | English (no field translations) |
| Deployment | Unmanaged solution import; all listed platform dependencies must be pre-installed |
| Performance | Standard Dataverse SLAs; no custom indexing defined |
| Availability | 99.9% (Microsoft Cloud SLA) |
| Localisation | No custom language resources; English default |
| Personalisation | Central user personalisation disabled; all users with same role see same UI |
| AI | AI form-fill opt-out applied to `sit_totaltime`; other AI features enabled by default |

---

## 11. Solution Architecture

### 11.1 Solution Package Structure

Four separate managed solutions have been identified. They form a layered stack:

| Solution | Version | Role | Key Contents |
|----------|---------|------|-------------|
| **SitechSupportData** | 1.0.0.2 | Data/schema layer for Support | Entity definitions only: Incident, CustomerAsset, CaseProductLine, MachineBrand/Model/Type, ServiceBoM, ServiceItemComponents, Product |
| **SitechSupportUX** | 1.0.0.3 | UX layer for Support | Canvas Apps (Exploded View + Service Item Components), Case form customisations, PhoneCall extension, AI skill config |
| **SitechUX** | 1.0.0.1 | Shared base / Sales UX | Country table, Credits + CreditHours entities, Training module (TrainingSession + ContactTraining), Account extensions (segment, country lookups), app module definition, Sitech logo |

> **Design principle confirmed**: Support functionality is split across two solutions (Data + UX), with a shared base solution (SitechUX) for common entities used by both Sales and Support.  
> **Sales Hub** and **Customer Service Hub** are separate apps; they are not merged.

### 11.2 PhoneCall Extension (UX 1.0.0.3)

The `PhoneCall` standard entity has been extended with a custom field:
- `sit_account` (Lookup → Account) — links a phone call activity directly to an Account, independent of the parent Case or Contact.

This supports the short-contact registration model (logging calls at the Account level without creating a Case).

### 11.3 Machine Model Hierarchy — Confirmed Built

`sit_MachineModel` has **required** lookups to both `sit_MachineBrand` AND `sit_MachineType`. This means a Machine Model always belongs to a Brand and a Type. The three reference entities together form: **Brand + Type → Model**. This is not a simple three-level hierarchy but a two-dimensional classification (a Model sits at the intersection of a Brand and a Type).

### 11.4 Service Item Component — Replacement Tracking

`sit_ServiceItemComponents` includes a `sit_replaced` (boolean) field. This enables tracking whether a specific component has been physically replaced during a service visit. This field is not documented in any functional design document.

### 11.5 Platform Dependencies (SitechSupportUX)

### 11.1 Platform Dependencies (SitechSupportUX)

| Package | Min. Version |
|---------|-------------|
| msdynce_Service | 9.0.5.56 |
| msdyn_AssetCommon | 1.0.50.1 |
| msdyn_CaseManagementIntelligence | 9.2.26044.1067 |
| msdyn_CSIntelligence | 9.2.26044.1067 |
| msdynce_Sales | 9.0.4.0076 |
| msdynce_ProductManagement | 9.0.25113.10001 |
| IoTConnector | 1.8.53.0 |
| msdyn_ModernCaseManagement | 9.0.26042.2021 |
| OmnichannelBase | 1.10.0.146 |

---

## 12. Open Decisions Register

| # | Question | Owner | Deadline |
|---|----------|-------|----------|
| V-01 | When is a Lead created vs. direct Opportunity or Activity? | René / Sitech | Before configuration |
| V-02 | Which BC payment terms fields are needed on Account in CE? | Marijke + René | Before integration |
| V-03 | Blocked customer: deactivate in CE? Who may do this? | René / Boris | Before go-live |
| V-04 | Which fields of Customer Contracts/Subscriptions go to CE? | Marijke + René | Before integration |
| V-05 | Goals in CE: yes/no? Who manages targets? | René / Sitech | To be planned |
| V-06 | Sales Literature: activate? | René / Sitech | To be planned |
| V-07 | Marketing module: activate? | René / Dieuwertje | To be planned |
| V-08 | Which fields visible per tab (Service Items, Subscriptions) for Sales? | René | Before configuration |
| V-09 | Does BC store invoice PDFs in SharePoint? | Marijke (Jan Jaap) | Short term |
| V-10 | Merge/Similar/Parent-Child cases: use or exclude? | René / Support team | Before Support config |
| V-11 | SLA: use or not? Define levels? | René / Support team | Before Support config |
| V-12 | Credits: by Account or by Installation? How to invoice for negative balance? | René / Finance | Before Support config |
| V-13 | OEM price lists: synchronise to CE or BC-only? | René / Jan Jaap | Before integration |
| V-14 | Short vs. Normal cases: confirm registration approach | René / Support team | Before Support config |
| V-15 | Directorrol permissions: confirm exact scope | René | Before go-live |

---

## 13. Glossary

| Term | Definition |
|------|-----------|
| Asset / Service Item | A physical machine or equipment registered in CE. |
| Sub-Asset | A component installed on or within a parent asset. |
| BoM | Bill of Materials — parts/components list. |
| Bundle (EWV) | D365 product bundle representing an installation at Layer 1 (€0 price). |
| SBU | Sub-unit; an individual product/component in Layer 2 of the BoM. |
| Case Product Line | A record capturing a product + quantity required to resolve a case. |
| CE | Customer Engagement (Dynamics 365). |
| BC | Business Central (ERP). |
| CE-BC | Integration between Dynamics 365 CE and BC. |
| Canvas App | A Power Apps custom-UI application. |
| Exploded View | Hierarchical display of asset components. |
| VRS | Virtual Reference Station — a subscription-based positioning service. |
| EWV | Equivalent of the main installation product (bundle level). |
| SD / SE / SI | Service Delivery / Service External / Service Internal order types in BC. |

---

*End of Functional Specification — v1.1*  
*Sources: SitechSupportUX solution XML, Axelio FDD CESales v0.1, FDD Salesconcept DOCX (Axelio, 01-06-2026), Gap analysis vs. Axelio meeting notes (26-05-2026)*

# Gaps & Open Questions — SITECH D365 CE (Sales + Support UX)

**Document type**: Gap & Risk Register  
**Author role**: Business System Analyst  
**Sources**: SitechSupportData 1.0.0.2, SitechSupportUX 1.0.0.3, SitechUX 1.0.0.1 (all managed), Axelio FDD CESales v0.1, FDD Salesconcept DOCX (01-06-2026), Gap analysis vs. Axelio meeting notes (26-05-2026)  
**Date**: 2026-06-05  
**Version**: 1.3

---

## How to Read This Document

| Rating | Meaning |
|--------|---------|
| **Impact H/M/L** | Business impact if unresolved |
| **Effort H/M/L** | Effort to design + implement the solution |
| `[BLOCKER]` | Must be resolved before go-live |
| `[DECISION]` | Requires a Sitech decision — no action without it |
| `[DESIGN]` | Requires a functional design before build |

Gaps are grouped by theme. Cross-references to the Axelio comparison document (A1–C10) are included where applicable.

---

## SECTION 1 — Sales Module Gaps

### GAP-S01 — Customer Approval Process Not Designed `[BLOCKER]` `[DESIGN]`
**Impact**: H | **Effort**: M | *Ref: A1, A19, C1*

The rule that Account Managers can only create Prospects (not Customers) is stated but not yet designed as a workflow. Three components are missing:

1. **Status field on Account** (Prospect / Customer) with transition rules.
2. **Approval flow**: Sales Manager receives notification → approves → status changes.
3. **Quote-to-Order trigger**: On order creation, auto-transition from Prospect to Customer if approval is on file.
4. **Sync error handling (C1)**: If an order is submitted to BC for a not-yet-approved customer, the connector must return a clear error message to the agent. Currently unverified.

**Questions**:
- Does approval happen in CE (via approval flow/Power Automate) or outside?
- Who can approve when the Sales Manager is unavailable?
- Should blocked (inactive) accounts be prevented from receiving new orders?

---

### GAP-S02 — Lead Configuration Incomplete `[DESIGN]`
**Impact**: M | **Effort**: M | *Ref: A2*

The FDD states "standard process = Lead → Opportunity → Quote → Order" but does not specify:
- Lead source picklist values.
- Qualification criteria (when does a Lead become an Opportunity?).
- Disqualification reason codes.
- Who is authorised to qualify.
- Stale opportunity definition (days without activity).

All of the above were detailed in January 2026 sessions with Mark (Sales Manager) and need to be incorporated.

---

### GAP-S03 — Competitor Table and Opportunity Lost Reasons Not Configured `[DESIGN]`
**Impact**: M | **Effort**: L | *Ref: A3*

Agreed values from Jan 2026 with Mark are not reflected in the FDD or the solution:
- **Competitors**: None, Leica Geosystems, Unicontrole, CHC-NAV, Topcon, OEM, Others.
- **Lost Reasons (Opportunity)**: Competitor, Price, Solution fit, Timelines.
- **Lost Reasons (Quote)**: To be defined.

---

### GAP-S04 — Machine Info Fields on Opportunity Missing `[DESIGN]`
**Impact**: M | **Effort**: M | *Ref: A4*

The following custom fields, agreed with Mark on 21-01-2026, are absent from the Opportunity form design:
- Type, Brand, Reference, Model
- Specials (yes/no)
- No follow-ups (separate boolean field)
- "Add to Summary Competitor"

The FDD only references "SPA012 Machine selection hierarchy" as a development item without the concrete field list.

---

### GAP-S05 — Additional Account Fields Not Confirmed `[DECISION]`
**Impact**: M | **Effort**: M | *Ref: A5*

The following fields/sections agreed in October 2025 (Marijke session) were not addressed in the May 2026 meeting and are not confirmed in/out of scope:
- Sales Instructions (free text field)
- Work Order Instructions (decision: Sales or BC?)
- Customer Discounts section
- Finished Goods / Spare Parts / Services tab
- Segment field

---

### GAP-S06 — Salutation Field on Contact `[DESIGN]`
**Impact**: L | **Effort**: L | *Ref: A6*

Agreed with Mark/Alina (21-01-2026): a Salutation field must be added to the Contact form. Not mentioned in the FDD.

---

### GAP-S07 — Word Template / PDF Button on Quote `[DESIGN]`
**Impact**: M | **Effort**: M | *Ref: A7*

The quotation output format (Word template for customer-facing PDF) is not designed. This is a must-have for customer delivery. Open question from October 2025 session with Marijke. Not addressed in the May 2026 meeting.

**Questions**:
- Which Word template is used as the base?
- What is the required layout (logo, line items, discount, trade-in rendering)?
- Is the PDF emailed directly or saved to SharePoint?

---

### GAP-S08 — Account Manager–to–Customer Assignment Mechanism `[DECISION]`
**Impact**: M | **Effort**: M | *Ref: A17*

The requirement "link Account Managers to their customers" is noted but not designed. Two options:
- Use CE record **Owner** field.
- Use a dedicated lookup field (`Responsible Account Manager`) separate from ownership.

The interaction with BC sync (does BC carry an assigned salesperson?) needs to be clarified.

---

### GAP-S09 — Parent Order for Upsell Tracking `[DESIGN]`
**Impact**: L | **Effort**: M | *Ref: A18*

A "Parent Order" field on the Order entity and a view showing "which orders have become parent orders" for upsell reporting have not been designed. This was raised in October 2025.

---

### GAP-S10 — OEM Price List CE/BC Sync Decision `[BLOCKER]` `[DECISION]`
**Impact**: H | **Effort**: M | *Ref: B3*

Contradictory statements exist:
- Dec 2025: Three price lists proposed (standard, OEM bruto, OEM netto) all syncing to CE.
- Apr 2026: "OEM price lists will NOT be sent to CE."

This is unresolved. If OEM price lists do not reach CE, Sales cannot create quotes for OEM customers using the correct prices.

**Decision required before integration build.**

---

### GAP-S11 — Discount CE→BC Sync Mismatch `[BLOCKER]` `[DESIGN]`
**Impact**: H | **Effort**: M | *Ref: B2*

CE applies a **total-level discount** (amount or %). BC works with **line discounts and invoice discounts** (both as percentages). The CE→BC sync logic for discount translation has not been verified. If the connector converts incorrectly, order values will be wrong in BC.

**Action**: Technical verification of the CE→BC connector discount mapping required before build.

---

### GAP-S12 — Trade-In (Negative Quantity) BC Behaviour Untested
**Impact**: M | **Effort**: L | *Ref: FDD §4.3*

CE displays negative-quantity lines correctly (in parentheses). BC behaviour when receiving an order with negative lines has not been tested. Risk: BC may reject the order or create incorrect purchase lines.

**Action**: Test trade-in scenario in BC acceptance environment before go-live (action A-09 in FDD).

---

### GAP-S13 — Optional/Required Products in Bundles Not Designed `[DESIGN]`
**Impact**: M | **Effort**: M | *Ref: B1*

The two-layer bundle structure (EWV + SBU) is specified, but the **required vs. optional** product flag within a bundle is not:
- Required: zero-price, not removable.
- Optional: price visible, removable.

Open question: Does BC have a required/optional flag on BoM lines that can be synced?

---

### GAP-S14 — Trimble Products Without Write-In `[DESIGN]`
**Impact**: M | **Effort**: M | *Ref: B10*

Trimble third-party products (handled by Arie) cannot be accommodated with write-in disabled. A proposed solution exists (generic product with editable description/price, BoM via Excel upload in Assembly) from Dec 2025 mail exchange but was not referenced in the May 2026 meeting. This needs to be confirmed as the agreed approach.

---

### GAP-S15 — Sales Order "Lock for Operations" Internal Decision Pending `[DECISION]`
**Impact**: L | **Effort**: L | *Ref: A20*

The ability to lock a Sales Order for the operations/bedrijfsbureau department has been tested and confirmed working. An internal Sitech decision on when and how to use this feature is still outstanding.

---

### GAP-S16 — Pre-Invoicing Scenario Not Covered in Invoice Sync Design `[DESIGN]`
**Impact**: H | **Effort**: H | *Ref: A21*

For installations, the customer is invoiced at a fixed price upon delivery, but the BC order may be incomplete (material returns, hour write-offs still pending). A **pre-invoice** is needed; the final cost settlement in BC follows later. The current "Invoices BC → CE read-only" integration design does not address this two-stage invoicing pattern.

---

### GAP-S17 — Multiple Installations on One Sales Order — CE Verification Pending
**Impact**: M | **Effort**: L | *Ref: C5*

BC supports 2+ installations on one Sales Order. Whether CE handles this correctly (e.g., two bundles on one quote/order) has not been tested.

---

### GAP-S18 — VRS Subscription Placement Unresolved `[BLOCKER]` `[DESIGN]`
**Impact**: H | **Effort**: H | *Ref: A8*

VRS subscriptions sold per installation currently auto-link to the installation (Service Item) via BC, which is **undesired**. The preferred behaviour is linking to the main Account location.

Open questions:
- Can VRS subscriptions be sold stand-alone only (not bundled with installations)?
- Should invoicing go to the branch location or the head office?
- Requires a design decision before the Subscriptions sync (IF-11) is built.

---

## SECTION 2 — Support / Customer Service Module Gaps

### GAP-C01 — CE-BC Service Product Sync Absent `[BLOCKER]`
**Impact**: H | **Effort**: H | *Ref: Original GAP-01; C4*

`sit_CaseProductLine` defines what products are needed per case, but **no Power Automate flow, plugin, or connector** bridges this to BC for fulfilment. Additionally, the SD-type Service Order currently has no value without product synchronisation.

**Questions**:
- Which direction triggers the sync (CE push or BC pull)?
- What triggers it: case status change, manual button, or scheduled batch?
- Field mapping: `sit_CaseProductLine` → BC purchase/sales order line?
- How are sync errors surfaced to the agent?
- Should a `sit_syncstatus` field (Pending / Synced / Error) be added?

---

### GAP-C02 — Sub-Asset Filter Not Implemented at Form Level `[DESIGN]`
**Impact**: M | **Effort**: M

`sit_subasset` should filter to child assets of the selected `sit_asset`, but no fetchXML filter or JavaScript handler enforces this. The field is currently an open lookup.

**Clarification needed**: Is the filter implemented in a separate web resource or not yet built?

---

### GAP-C03 — Canvas App Logic Not Auditable `[BLOCKER for Quality Review]`
**Impact**: H | **Effort**: L

Both Canvas Apps are delivered as compiled `.msapp` binaries. Business logic, formulas, data connections, delegation behaviour, and navigation flows cannot be reviewed or audited.

**Questions**:
- Are unpacked source files available?
- Have the apps been reviewed for non-delegable queries (performance risk)?
- What Dataverse connectors and environment variables do they use?

---

### GAP-C04 — Product List Commit Mechanism Undefined `[BLOCKER]` `[DESIGN]`
**Impact**: H | **Effort**: M

The Service Item Components Canvas App has a staging gallery and an "Add to Product List" button, but:
- When/how does the staged list commit to `sit_CaseProductLine` records?
- Which case does it link to?
- What happens if the user closes the app without committing?

---

### GAP-C05 — "Create Case from Asset" Flow Undefined `[DESIGN]`
**Impact**: M | **Effort**: M

The exploded view app has a "Create Case" icon but no specification for:
- Which Case form is opened.
- Which fields are pre-populated (asset, sub-asset, account?).
- Whether it opens in the app or navigates to the model-driven form.

---

### GAP-C06 — Case Status Reasons Not Confirmed in Solution
**Impact**: M | **Effort**: L | *Ref: A10*

The case status model (In Progress / On Hold / Waiting for Details / Researching / Problem Solved / Information Provided / Cancelled / Merged) was designed on 17-12-2025 but was not confirmed in the 26-05-2026 Support CE meeting and is not reflected in the current solution package.

---

### GAP-C07 — Merge / Similar / Parent-Child Cases — No Decision `[DECISION]`
**Impact**: M | **Effort**: M | *Ref: A11*

No decision has been made on whether to enable Merge Cases, Similar Cases, and Parent-Child Case functionality. These must either be configured (if used) or explicitly excluded from UX and business logic.

---

### GAP-C08 — Credits & Entitlements Not Designed `[BLOCKER]` `[DESIGN]`
**Impact**: H | **Effort**: H | *Ref: A12, A14*

The entire Credits & Entitlements topic was not addressed in the 26-05-2026 Support CE meeting, yet it directly affects whether cases can be opened and how support time is billed.

**Missing decisions**:
- Use D365 Entitlements? If yes, define levels and duration.
- Credit balance: by Account or by Installation?
- Free threshold before credits are consumed.
- Invoicing logic for negative credit balance.
- Whether Entitlements are scoped at channel, product, Asset, or Account level.

---

### GAP-C09 — SLA Decision Absent `[DECISION]`
**Impact**: M | **Effort**: M | *Ref: A13*

Whether D365 SLA functionality will be used has not been decided. If SLAs are used, levels and response/resolution targets must be defined. This affects case routing and escalation logic.

---

### GAP-C10 — Machine Driver (Asset-Contact Link) Not Fully Defined `[DESIGN]`
**Impact**: M | **Effort**: M | *Ref: A15*

The concept of a Machine Driver (operator/contact associated with a specific asset) was discussed but not resolved. The one-to-one Contact ↔ Service Item link in the current solution is a partial answer. The unique number mechanism for machine drivers still needs design.

---

### GAP-C11 — Short vs. Normal Case Registration — No Process Decision `[DECISION]`
**Impact**: M | **Effort**: L | *Ref: A16*

The proposal to handle short contacts as Phone Call activities (no case record) vs. normal contacts via the Quick Create Case form has not been formally decided. This affects training, queue configuration, and reporting.

---

### GAP-C12 — Case-to-Service Order Conversion Rules Not Fully Defined `[DESIGN]`
**Impact**: M | **Effort**: M | *Ref: B6, B7*

- The Work Order route is decided out (Case → BC Service Order is correct). However, the Work Order route is not explicitly excluded in the meeting notes — this should be formally documented.
- Open question: Can one case be converted to a Service Order more than once (e.g., one SE and one SI)?
- Support visibility of all Work/Service Orders associated with an Asset or Customer is not defined.

---

### GAP-C13 — Asset/Installation Timeline Configuration Not Designed `[DESIGN]`
**Impact**: M | **Effort**: M | *Ref: A9*

A detailed design for the Asset timeline and Account timeline is missing:
- Should BC post final activities to the case timeline after a Service Order is completed?
- Which specific CRM metrics are shown (red flags, frequency of orders, etc.)?

---

### GAP-C14 — No Custom Security Roles in Delivered Solution `[BLOCKER]`
**Impact**: H | **Effort**: M

The SitechSupportUX solution package contains no custom security roles. Out-of-the-box roles (e.g., Customer Service Agent) may not have create/read/write permissions on the 6 custom entities by default.

**Questions**:
- Which roles need access to each custom entity?
- Should field-level security be applied to any fields?
- Who should have access to the Canvas Apps?

---

### GAP-C15 — No Validation Rules on sit_totaltime
**Impact**: M | **Effort**: L

`sit_totaltime` has no min/max or format validation. An agent can enter 0 or a negative number.

**Questions**:
- Minimum billing increment (e.g., multiples of 15)?
- Should it auto-populate from activity durations?
- Editable after case closure?

---

### GAP-C16 — No Business Rules for Case Closure Prerequisites
**Impact**: M | **Effort**: L

No business rule enforces that at least one Case Product Line exists before closing, or that `sit_totaltime` is populated before resolution.

---

## SECTION 3 — Integration Gaps (Technical)

### GAP-T01 — Power Automate Flows Not Designed `[BLOCKER]` `[DESIGN]`
**Impact**: H | **Effort**: H | *Ref: C6, G-09 in FDD*

The following Power Automate flows are mentioned as requirements but have not been designed or built:

| Flow | Trigger | Action |
|------|---------|--------|
| SharePoint customer folder | Account created | Create folder on SharePoint |
| SharePoint folder rename | Account name changed | Rename folder on SharePoint |
| SharePoint Service Item folder | Service Item created | Create subfolder with naming convention |
| SharePoint Service Order subfolder | Service Order created | Create date-stamped subfolder |
| Prospect → Customer approval | Sales Manager approves | Update Account status |

---

### GAP-T02 — Service Item SharePoint Migration `[BLOCKER for Go-Live]`
**Impact**: H | **Effort**: H | *Ref: C7, C8*

When legacy Service Items are imported from the old CE, their SharePoint links are lost. A migration flow is needed to:
1. Rename existing SharePoint folders to the new naming convention: `<YYYYMMDD> - <Customer> - <Assembly Order No.> - <Installation Name>`.
2. Re-establish the SharePoint–CE link for each record.

This is a significant migration task with no plan or owner assigned.

---

### GAP-T03 — Subscription-from-BoM: No Technical Solution `[DESIGN]`
**Impact**: M | **Effort**: H | *Ref: C2*

When a Subscription (SIM card or Software licence) is a BoM component, CE cannot automatically create or link a Subscription record from that BoM entry. The "Customer Contracts read-only" decision does not resolve this structural problem.

---

### GAP-T04 — Exploded View → Assembly Orders Link Undefined `[DESIGN]`
**Impact**: M | **Effort**: H | *Ref: C3*

A requirement exists to navigate from the Exploded View Canvas App to create X Assembly Orders in BC. This is not designed or built.

---

### GAP-T05 — Solutions Architecture Not Formally Confirmed `[DECISION]`
**Impact**: M | **Effort**: L | *Ref: C9*

The architectural decision "multiple solutions per app with a shared base solution for common tables; Sales in Sales Hub; Support in Customer Service / Field Service Hub — do not merge into one app" was proposed in October 2025 but not formally confirmed in May 2026.

---

### GAP-T06 — Personalization Disabled in Support Only
**Impact**: L | **Effort**: L | *Ref: C10*

Centralized personalization disable (same screen for all users per role) is confirmed for the Support module but was not confirmed for the Sales module in the May 2026 meeting. For consistency, the Sales module should follow the same approach. Needs explicit confirmation.

---

## SECTION 4 — Data and Reference Data Gaps

### GAP-D01 — No Reference Data Seeded (Brand/Model/Type)
**Impact**: M | **Effort**: M | *Original GAP-06*

The `sit_MachineBrand`, `sit_MachineModel`, and `sit_MachineType` entities are defined but contain no data. No import template or Configuration Migration Tool schema is included.

**Questions**:
- Who owns and maintains these reference entities?
- Is there a master list in BC or a spreadsheet to be migrated?
- Should Model be filtered by Brand (cascading lookups)? Currently no relationship exists between Brand and Model.

---

### GAP-D02 — Brand/Model/Type Hierarchy Not Modelled
**Impact**: M | **Effort**: M | *Original GAP-07*

The three reference entities are independent. Typical machinery hierarchy: Brand → Model → Type. If this hierarchy is needed for filtering on Opportunity or Asset forms, the relationships need to be defined.

---

### GAP-D03 — No Data Migration Plan for Existing Cases `[BLOCKER]`
**Impact**: H | **Effort**: H | *Original GAP-15*

`sit_asset` is a required field on Case. Existing cases have null values. Options:
- Back-fill asset references on historical cases (manual or automated).
- Mark `sit_asset` as required only for new cases (business rule conditional).
- Accept that historical cases cannot be re-opened/edited without adding an asset.

A decision and migration plan are needed before go-live.

---

### GAP-D04 — Country Table Migration
**Impact**: M | **Effort**: L

The custom Country entity for the Sales module requires an initial load from BC. No import procedure or schedule is defined.

---

### GAP-D05 — Activities Data Migration (TBD)
**Impact**: M | **Effort**: H

The data conversion table in the FDD marks Activities (in CE/Dataverse) as "NTB" (to be determined). Leads, Opportunities, Quotes, and Orders from the existing CRM have no migration plan. Volume and quality of existing data is unknown.

---

## SECTION 4a — Code vs. Documentation Divergence (New Findings from Managed Packages)

### GAP-X01 — Training Module (SITECH Academy) Undocumented in FDD `[DESIGN]`
**Impact**: H | **Effort**: M

**Status update (2026-06-05)**: The Training Module is **live in production since May 2024** as the SITECH Academy. It is **not** an abandoned prototype. The confirmation email flow has been running for 2+ years. However, it still has **no functional design document**, no acceptance criteria, and no formal requirements specification anywhere in the project documentation set.

The module is now reverse-engineered and documented in Section 6a of the Functional Specification.

**Remaining gaps**:
- No formal requirements specification signed off by a business owner.
- No UX design for training session management screens.
- Who manages Training Sessions in CE — dedicated Academy staff, Sales, or Support?
- Are training certifications (`sit_certificatesant`) visible on the Contact record form?
- Are training histories visible on the Account form?
- Is the reminder flow ("Get upcoming training sessions") actively used?
- No test plan or acceptance criteria for the Academy module.

**Recommendation**: Conduct a one-day discovery session with the Academy owner to formalise the functional spec in Section 6a and capture the missing flows.

---

### GAP-X02 — `sit_ContactTraining.sit_contact` — Misleading Relationship Name (Re-evaluated)
**Impact**: L | **Effort**: L

**Status update (2026-06-05)**: Originally flagged as a likely bug (field `sit_contact` bound to `Contract` entity via relationship `sit_contacttraining_Contact_contract`). Analysis of the live flow definition shows that the flow correctly uses `triggerBody()?['_sit_contactid_value']` and reads from the `contacts` entity — the field stores **Contact GUIDs**, not Contract GUIDs.

**Current assessment**: The relationship XML name `sit_contacttraining_Contact_contract` is **misleading** but the field value column (`_sit_contactid_value`) correctly points to Contact records. The flow works. The bug may be a naming artefact from the solution builder.

**Action required**: Verify in the live environment that:
1. The schema column `_sit_contactid_value` references the `contact` entity (not `contract`).
2. No existing records have accidental Contract references stored in this column.

Low risk, but confirmation closes the gap.

---

### GAP-X03 — Credits Entities Built But No Functional Design or UI `[BLOCKER]` `[DESIGN]`
**Impact**: H | **Effort**: M

`sit_Credits` and `sit_Credithours` are deployed in `SitechUX v1.0.0.1`, partially addressing the Credits concept discussed in the gap analysis (GAP-C08). However:

- **No functional design** describes the business rules governing these entities.
- **No forms or views** are defined for these entities in the solution packages — agents cannot see or interact with them.
- **No automation** (Power Automate / plugin) is present to populate or update these records.
- The relationship between `sit_Credits` (transactions?) and `sit_Credithours` (balance?) is not documented.
- `sit_Credithours` has `sit_lastorder → SalesOrder` — but the business rule triggering a credit update when an order is placed is not defined.

**This means Credits data is structurally stored but functionally inert.** The feature is partially built and cannot be used.

---

### GAP-X04 — Subscriptions Entity Not Yet Built
**Impact**: M | **Effort**: M

The FDD specifies a custom `Subscriptions` table (BC → CE read-only). This entity is **absent** from all three solution packages. It is referenced in the FDD as a must-have for the Customer Contracts/Subscriptions tab on the Account form.

---

### GAP-X05 — Account Segment Field: Type Mismatch with FDD `[DESIGN]`
**Impact**: M | **Effort**: L

`SitechUX` implements `sit_accountsegment` as a **multi-select picklist** on Account. The FDD (§A5) refers to a single "Segment" field without specifying whether it should be single-select or multi-select.

**Questions**:
- Can an Account belong to multiple segments simultaneously?
- What are the picklist option values? None are defined in the solution.
- Is this the same as the `msdyn_segmentid` lookup to `msdyn_segment` also present on the Account? (Both exist — potential duplication.)

---

### GAP-X06 — Machine Model Is a Brand×Type Intersection, Not a Hierarchy `[DESIGN]`
**Impact**: M | **Effort**: M

`sit_MachineModel` has **required** lookups to both `sit_MachineBrand` AND `sit_MachineType`. This means the data model is a 2D classification matrix (Brand × Type → Model), not a simple 3-level Brand → Model → Type hierarchy.

**Functional implications not addressed in any document**:
- On the Opportunity Machine Info section, which picker comes first — Brand or Type?
- Is the Model lookup filtered by both the selected Brand AND the selected Type?
- What happens if no model exists for a given Brand+Type combination?
- Are all three fields required on the Opportunity, or can an agent partially fill in only Brand?

---

### GAP-X07 — Service Item Component Replacement Tracking Not Specified `[DESIGN]`
**Impact**: L | **Effort**: L

`sit_ServiceItemComponents` has a `sit_replaced` (boolean) field. This enables tracking whether a physical component was replaced during a service visit, but:
- No business rule defines when this flag is set.
- No workflow triggers on the flag change (e.g., auto-create a follow-up or close the component line).
- No view or filter exposes "replaced components" to agents.

---

### GAP-X08 — SitechSupportData vs SitechSupportUX Layering Not Documented
**Impact**: L | **Effort**: L

The split between `SitechSupportData` (schema) and `SitechSupportUX` (forms + canvas apps) is a deliberate architectural pattern. However, no deployment sequence document or README describes:
- Installation order: SitechSupportData must be installed before SitechSupportUX.
- Which solution to update when adding new fields (Data) vs. changing forms (UX).
- Whether SitechUX must also be installed before or after SitechSupportData.

---

### GAP-X09 — `sit_trainingsessiontime` Field Name Mismatch in Flow vs. Schema `[BUG]`
**Impact**: M | **Effort**: L

The confirmation flow references `outputs('Get_a_Training_Session')?['body/sit_trainingsessiontime']` when composing the email body (time field). However, the solution schema (`SitechUX v1.0.0.1`) defines the time field as **`sit_time`**, not `sit_trainingsessiontime`.

This means:
- The time value in all confirmation emails is currently **blank** (the expression resolves to null).
- Participants receive a confirmation with an empty "Tijd:" (time) line.

**Action required**: Either rename the schema field from `sit_time` to `sit_trainingsessiontime` (requires solution update and republish), or fix the flow expression to use `body/sit_time`. Fixing the flow is lower effort.

---

### GAP-X10 — Training Flows Run Under Personal Credentials `[RISK]` `[BLOCKER]`
**Impact**: H | **Effort**: M

Both Academy flows (`When Training Participants is added - send Confirmation` and `Copy of - Get upcoming training sessions`) run under the personal Microsoft 365 account `alina.zhirkova@sitech-nederland.nl`. The connections are owned by this individual user.

**Risks**:
- If `alina.zhirkova` leaves Sitech or her account is disabled, **both flows will fail immediately** and no training confirmation emails will be sent.
- The flow was suspended (`flowclientsuspensiontime: 2026-03-03`) — likely due to a connection/credential issue. It is **currently non-operational** ⚠️.
- Personal connections cannot be monitored centrally or transferred without the owner's intervention.

**Action required**:
1. Create a **service account** (e.g., `svc-powerautomate@sitech-nederland.nl`) with appropriate D365 + SharePoint licences.
2. Re-establish flow connections under the service account.
3. Implement flow monitoring/alerting (failure email to IT/business owner).
4. Investigate and resolve the current suspension to restore flow operation.

---

### GAP-X11 — "Copy of - Get Upcoming Training Sessions" Flow Unanalysed `[DESIGN]`
**Impact**: M | **Effort**: L

The manifest for the exported package (`9c82e091-manifest.json`) shows a flow named **"Copy of - Get upcoming training sessions"** (`5aba4c9d-4af9-481d-85fb-5eedefe0cb9c`). The "Copy of -" prefix indicates this is a **duplicate** of an original flow, not a canonical production version.

**Concerns**:
- The original "Get upcoming training sessions" flow may have been deleted or may still exist in the environment.
- If both original and copy are active simultaneously, reminders may fire twice.
- The full definition of this flow has not been analysed — its trigger, schedule, and actions are unknown.

**Action required**: Analyse the flow definition, determine whether the original still exists in the environment, deactivate any duplicate, and ensure only one canonical version runs.

---

### GAP-X12 — Hardcoded Product GUIDs in Training Confirmation Flow `[RISK]`
**Impact**: M | **Effort**: L

The confirmation flow has two Product GUIDs hardcoded in a condition to **suppress** confirmation emails for those training types:

| GUID | Status |
|------|--------|
| `b24df9ab-0a7a-ec11-8d21-000d3aafc422` | Undocumented — likely "internal" or "free" training |
| `b4632176-8769-ed11-9561-6045bd895dae` | Undocumented — likely another internal type |

**Risks**:
- No documentation explains what these products are or why they are excluded.
- If the Product records are deleted and recreated in a new environment, new GUIDs will be assigned and the exclusion silently breaks — confirmation emails will start going to internal participants.
- No way to add new excluded products without editing the flow directly.

**Action required**:
1. Document what these two Product records represent.
2. Replace hardcoded GUIDs with a configurable attribute on the Product record (e.g., a boolean `sit_suppressconfirmation`) and update the flow condition to check that attribute.

---

## SECTION 5 — Testing, Quality, and Documentation Gaps

### GAP-Q01 — No Test Plan or Acceptance Criteria `[BLOCKER]`
**Impact**: H | **Effort**: M

Neither the Sales FDD nor the Support solution package includes test scenarios or acceptance criteria.

**Recommended minimum test coverage**:

| Scenario | Module |
|----------|--------|
| Lead → Opportunity → Quote → Order → BC sync | Sales |
| Prospect approval → Customer creation | Sales |
| Quote with trade-in (negative line) → BC | Sales |
| Bundle product with OEM pricing → Quote | Sales |
| Case creation with asset + sub-asset | Support |
| Case Product Lines → (future) CE-BC sync | Support |
| Case closure with status reason (Sales/Service/Field) | Support |
| Canvas App: search asset → create case | Support |
| SharePoint folder auto-creation on Account | Both |
| Blocked customer order rejection with error | Sales |

---

### GAP-Q02 — Canvas Apps Not Peer-Reviewed
**Impact**: M | **Effort**: L

The compiled `.msapp` binaries prevent formula, delegation, and connector review. A structured review of the uncompiled source is needed before go-live.

---

### GAP-Q03 — No Solution Checker Run
**Impact**: L | **Effort**: L

Power Apps Solution Checker (static analysis) has not been run against the SitechSupportUX package. Deprecated components or performance warnings may exist.

---

### GAP-Q04 — README and Developer Documentation Empty
**Impact**: L | **Effort**: L

The repository README contains a single placeholder line. There is no import checklist, environment setup guide, or dependency verification procedure.

---

### GAP-Q05 — No User Guide or Training Material
**Impact**: M | **Effort**: M

No end-user training material is included for Sales agents (quote creation, product selection, discount entry) or Service agents (case creation with asset, canvas app usage).

---

## Summary Table

### Blockers (Must resolve before go-live)

| ID | Description | Module |
|----|-------------|--------|
| GAP-S01 | Customer approval process not designed | Sales |
| GAP-S10 | OEM price list CE/BC sync decision contradictory | Sales |
| GAP-S11 | Discount CE→BC mapping unverified | Sales |
| GAP-S18 | VRS subscription placement unresolved | Both |
| GAP-C01 | CE-BC service product sync absent | Support |
| GAP-C03 | Canvas app binaries not auditable | Support |
| GAP-C04 | Product list commit mechanism undefined | Support |
| GAP-C08 | Credits & Entitlements not designed | Support |
| GAP-C14 | No custom security roles in solution | Support |
| GAP-D03 | No data migration plan for existing cases | Support |
| GAP-T01 | Power Automate flows not designed | Both |
| GAP-T02 | Service Item SharePoint migration unplanned | Both |
| GAP-Q01 | No test plan or acceptance criteria | Both |
| **GAP-X03** | **Credits entities built but no UI, no design, no automation** | **SitechUX** |
| **GAP-X10** | **Training flows run under personal credentials — currently suspended** | **Academy** |

### High Impact (Non-Blocker)

| ID | Description | Module |
|----|-------------|--------|
| GAP-S02 | Lead configuration incomplete | Sales |
| GAP-S03 | Competitor table + lost reasons missing | Sales |
| GAP-S04 | Machine Info fields on Opportunity | Sales |
| GAP-S07 | Word template / PDF button undefined | Sales |
| GAP-S16 | Pre-invoicing scenario not in design | Sales |
| GAP-C02 | Sub-asset filter not implemented | Support |
| GAP-C09 | SLA decision absent | Support |
| GAP-C12 | Case-to-service order rules incomplete | Support |
| GAP-T03 | Subscription-from-BoM unsolved | Both |
| GAP-D05 | Activities data migration TBD | Both |
| **GAP-X01** | **SITECH Academy module undocumented — no FDD, no owner sign-off** | **Academy** |
| **GAP-X09** | **sit_trainingsessiontime field name mismatch — time blank in all emails** | **Academy** |
| **GAP-X11** | **"Copy of -" training flow is duplicate — may double-fire reminders** | **Academy** |
| **GAP-X12** | **Hardcoded product GUIDs in flow — silently breaks on environment copy** | **Academy** |

### Low / Medium Impact

| ID | Description | Module |
|----|-------------|--------|
| GAP-X02 | sit_ContactTraining relationship name misleading — verify in env | Academy |
| GAP-X04 | Subscriptions entity not yet built | Sales |
| GAP-X05 | Account segment field type mismatch with FDD | Sales |
| GAP-X06 | Machine Model is Brand×Type matrix — filter logic undesigned | Sales |
| GAP-X07 | Component replacement flag has no workflow | Support |
| GAP-X08 | Solution layering deployment order undocumented | Both |

---

*End of Gap & Risk Register — v1.3*  
*Sources: SitechSupportUX solution XML, Axelio FDD CESales v0.1, FDD Salesconcept DOCX (Axelio, 01-06-2026), Gap analysis vs. Axelio meeting notes (26-05-2026), Power Automate flow definitions (Training Confirmation + Upcoming Sessions, 2026-06-05)*

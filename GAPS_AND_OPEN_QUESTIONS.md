# Gaps & Open Questions — SITECH Support UX

**Document type**: Gap Analysis  
**Author role**: Business System Analyst  
**Solution**: SitechSupportUX v1.0.0.2  
**Date**: 2026-06-02

---

## How to Read This Document

Each gap is rated by **Impact** (High / Medium / Low) and **Effort to resolve** (High / Medium / Low).  
Items marked `[BLOCKER]` should be resolved before go-live.

---

## 1. Integration Gaps

### GAP-01 — CE-BC Integration Logic Not Present `[BLOCKER]`
**Impact**: High | **Effort**: High

`sit_CaseProductLine` is described as the data source for synchronisation with the back-end ERP (Business Central), but **no Power Automate flow, plugin, or connector is included** in this solution.

**Questions**:
- Which system initiates the sync — Dynamics 365 (push) or Business Central (pull)?
- What triggers synchronisation: status change on the case, a manual button, or a scheduled batch?
- What is the field mapping between `sit_CaseProductLine` and the BC purchase/sales order line?
- How are sync errors surfaced to the agent?
- Is there a status field needed on `sit_CaseProductLine` to track sync state (e.g., Pending / Synced / Error)?

---

### GAP-02 — No Error Handling / Sync Status on Case Product Lines
**Impact**: High | **Effort**: Medium

`sit_CaseProductLine` has no status field beyond the standard Active/Inactive. If CE-BC sync fails there is no way to communicate the failure state back to the agent.

**Recommendation**: Add a `sit_syncstatus` option-set field (Pending / Synced / Error) and a `sit_syncerror` text field.

---

### GAP-03 — Sub-Asset Filter Logic Not Implemented at Metadata Level
**Impact**: Medium | **Effort**: Medium

`sit_subasset` is documented as "filtered by the selected asset," but the solution XML contains no filter fetchXML or JavaScript handler to enforce this filtering at the form level. The field is an open lookup.

**Questions**:
- Is the filtering implemented in a JavaScript web resource outside this solution package?
- Is it implemented via form OnLoad / OnChange events in a separate customisation?
- If not yet built, what is the parent-child relationship criterion — is it `msdyn_parentasset` on `msdyn_customerasset`?

---

## 2. Business Logic Gaps

### GAP-04 — No Validation Rules for sit_totaltime
**Impact**: Medium | **Effort**: Low

`sit_totaltime` has no minimum, maximum, or format validation defined. An agent could enter 0 or a negative number.

**Questions**:
- Is there a minimum billable time increment (e.g., must be a multiple of 15 minutes)?
- Should the field auto-populate from activity durations (phone calls, appointments) on the case?
- Should it be editable once the case is resolved/closed?

---

### GAP-05 — No Business Rules for Required Fields Beyond Asset
**Impact**: Medium | **Effort**: Low

Only `sit_asset` is marked required. There is no business rule enforcing that at least one Case Product Line exists before a case can be resolved, or that `sit_totaltime` is populated before closure.

**Questions**:
- Should cases be blocked from resolution unless at least one product line exists?
- Is `sit_totaltime` mandatory before case closure for billing purposes?

---

### GAP-06 — Machine Reference Data (Brand / Model / Type) Has No Seeded Data
**Impact**: Medium | **Effort**: Medium

The three reference entities (`sit_MachineBrand`, `sit_MachineModel`, `sit_MachineType`) are defined but the solution contains **no reference data** (no import maps, no data files).

**Questions**:
- Who is responsible for populating these reference entities?
- Is there a master list in the current system (spreadsheet, ERP) to be migrated?
- Should Model be filtered by Brand (cascading dropdowns)? Currently there is no relationship between Brand and Model.
- Should Type drive different service workflows or SLAs?

---

### GAP-07 — Cascading Relationship Between Brand, Model, and Type Not Modelled
**Impact**: Medium | **Effort**: Medium

The three reference entities are independent; there is no parent-child relationship between Brand → Model → Type, which is the typical hierarchy for machinery.

**Recommendation**: Define whether a Machine Model must belong to a Machine Brand, and whether a Machine Type is independent or a sub-classification of Brand/Model. Update the data model accordingly.

---

### GAP-08 — No Duplicate Detection on Case Product Lines
**Impact**: Low | **Effort**: Low

An agent can add the same product multiple times to the same case, resulting in duplicate lines. There is no duplicate detection rule defined.

**Question**: Should duplicate products be merged (summing quantities), blocked, or allowed (for cases where the same product is needed for different purposes)?

---

## 3. UX / Canvas App Gaps

### GAP-09 — Canvas App Source Code Not Readable `[BLOCKER for Review]`
**Impact**: High | **Effort**: Low (documentation only)

Both canvas apps are shipped as compiled `.msapp` binary files. The UI logic, formulas, data connections, and navigation flows **cannot be reviewed or audited** from this solution package alone.

**Questions**:
- Are the source `.msapp` files or unpacked source folders available for review?
- Have the apps been peer-reviewed for performance anti-patterns (e.g., non-delegable queries)?
- What data sources / connectors are the apps using — Dataverse connectors, custom connectors, environment variables?

---

### GAP-10 — No Responsive / Accessibility Requirements Documented
**Impact**: Medium | **Effort**: Medium

There are no documented screen size targets, accessibility (WCAG) requirements, or browser compatibility constraints for the Canvas Apps.

**Questions**:
- Are the apps used on desktop only, or also on tablets / phones?
- Must the apps comply with any accessibility standard (e.g., WCAG 2.1 AA)?

---

### GAP-11 — "Create Case from Asset" Flow Undefined
**Impact**: Medium | **Effort**: Medium

The exploded view app has an `icCreateCase` button, but there is no specification of:
- Which Case form is opened.
- Which fields are pre-populated (asset, sub-asset, account?).
- Whether it opens in the same app or navigates to the model-driven Case form.

---

### GAP-12 — "Add to Product List" Commit Mechanism Undefined
**Impact**: High | **Effort**: Medium

The Service Item Components app has a staging gallery (`galNewProductsList`) and an `btnAddToProductList` button, but there is no specification for:
- When / how the staged list is committed to `sit_CaseProductLine` records.
- Whether the commit is linked to a specific case (which case?).
- What happens if the user closes the app without committing.

---

## 4. Security & Access Control Gaps

### GAP-13 — No Custom Security Roles Defined
**Impact**: High | **Effort**: Medium

The solution does not include any custom security roles. All six custom entities default to platform-level access, which means existing roles (e.g., Customer Service Agent) may not have create/read/write permissions on `sit_CaseProductLine`, `sit_ServiceBillofMaterials`, etc.

**Questions**:
- Which roles need access to each custom entity?
- Should field-level security be applied to any fields (e.g., restricting `sit_totaltime` edits to managers only)?
- Are the Canvas Apps intended for all Customer Service Agents or a specific team?

---

### GAP-14 — Owner / Owning Business Unit on sit_CaseProductLine
**Impact**: Low | **Effort**: Low

`sit_CaseProductLine` is a user-owned entity. In a multi-BU organisation this could cause records to become inaccessible if the owning agent leaves or changes team.

**Question**: Should ownership follow the parent case, or should it be organisation-owned (no owner)?

---

## 5. Data & Migration Gaps

### GAP-15 — No Data Migration Plan
**Impact**: High | **Effort**: High

The solution adds required/recommended fields (`sit_asset`) to the Case entity. Existing cases will have null values in these fields.

**Questions**:
- How many existing case records are in the target environment?
- Will historical cases need to be back-filled with asset references?
- Will the required field validation block edits to historical cases that lack an asset?

---

### GAP-16 — No Reference Data Migration Tooling
**Impact**: Medium | **Effort**: Medium

No configuration data migration (CDM) file or data import template is provided for Brand, Model, and Type reference entities.

**Recommendation**: Provide an Excel/CSV template and a Configuration Migration Tool schema file for reference data seeding.

---

## 6. Testing & Quality Gaps

### GAP-17 — No Test Plan or Acceptance Criteria
**Impact**: High | **Effort**: Medium

The repository contains no test scripts, UAT scenarios, or acceptance criteria.

**Recommendation**: Define at minimum:
- Happy path: Create a case, link asset + sub-asset, add product lines, verify CE-BC sync trigger.
- Edge cases: Case with no sub-asset, case with 10+ product lines, duplicate product line attempt.
- Regression: Existing cases unaffected by new required field.

---

### GAP-18 — No Automated Solution Checker Results
**Impact**: Low | **Effort**: Low

The solution has not been validated with the **Power Apps Solution Checker** (static analysis tool). There may be undiscovered issues with deprecated components or performance warnings.

**Recommendation**: Run Solution Checker and attach results before go-live.

---

## 7. Documentation Gaps

### GAP-19 — README is Empty
**Impact**: Low | **Effort**: Low

The `README.md` contains a single placeholder line. There is no developer setup guide, import checklist, or environment setup documentation.

---

### GAP-20 — No User Guide / Training Material
**Impact**: Medium | **Effort**: Medium

No end-user documentation or training material is included for service agents who will use the modified Case forms and Canvas Apps.

---

## Summary Table

| ID | Description | Impact | Effort | Blocker? |
|----|-------------|--------|--------|----------|
| GAP-01 | CE-BC integration logic missing | High | High | Yes |
| GAP-02 | No sync status on Case Product Lines | High | Medium | |
| GAP-03 | Sub-asset filter not implemented | Medium | Medium | |
| GAP-04 | No validation on sit_totaltime | Medium | Low | |
| GAP-05 | No business rules for case closure | Medium | Low | |
| GAP-06 | No reference data seeded | Medium | Medium | |
| GAP-07 | Brand/Model/Type not related | Medium | Medium | |
| GAP-08 | No duplicate detection on product lines | Low | Low | |
| GAP-09 | Canvas app binaries unreadable | High | Low | Yes |
| GAP-10 | No accessibility/responsive requirements | Medium | Medium | |
| GAP-11 | Create Case from asset flow undefined | Medium | Medium | |
| GAP-12 | Product list commit mechanism undefined | High | Medium | |
| GAP-13 | No custom security roles | High | Medium | |
| GAP-14 | CaseProductLine ownership model | Low | Low | |
| GAP-15 | No data migration plan | High | High | |
| GAP-16 | No reference data migration tooling | Medium | Medium | |
| GAP-17 | No test plan / acceptance criteria | High | Medium | |
| GAP-18 | No Solution Checker run | Low | Low | |
| GAP-19 | README is empty | Low | Low | |
| GAP-20 | No user guide / training material | Medium | Medium | |

---

*End of Gap Analysis*

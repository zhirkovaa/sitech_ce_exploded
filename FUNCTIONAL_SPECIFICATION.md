# Functional Specification — SITECH Support UX (SitechSupportUX v1.0.0.2)

**Document type**: Functional Specification  
**Author role**: Business System Analyst  
**Solution**: SitechSupportUX  
**Publisher**: SITECH Netherlands  
**Platform**: Microsoft Dynamics 365 Customer Service / Power Platform  
**Date**: 2026-06-02  
**Version**: 1.0

---

## 1. Executive Summary

SITECH Support UX is a Microsoft Dynamics 365 Customer Service customisation solution developed for SITECH Netherlands. It enriches the standard case management process with **asset-aware service handling**, a **hierarchical parts/component structure**, **product requirements tracking per case**, and **two Power Apps Canvas applications** that give service agents a modern, asset-centric interface.

The solution is delivered as an unmanaged Dynamics 365 solution package (ZIP) to be imported into an existing Dynamics 365 Customer Service environment.

---

## 2. Business Context

### 2.1 Business Problem

SITECH Netherlands provides technical support for specialised machinery (construction/surveying equipment). Engineers must:

- Identify the exact machine (asset) that a customer is calling about.
- Understand the asset's sub-components (e.g., an installed sensor module on a machine).
- Record what spare parts or products are required to resolve the case so that the back-end (ERP/supply chain) can fulfill the order.
- Track total support time per case for billing and reporting.

The out-of-the-box Dynamics 365 Case form does not accommodate these needs, hence the customisation.

### 2.2 Business Objectives

| # | Objective |
|---|-----------|
| 1 | Link every service case directly to a registered customer asset. |
| 2 | Support hierarchical asset structures (installation → component). |
| 3 | Capture the bill of materials (products + quantities) needed for case resolution. |
| 4 | Synchronise required products with the back-end system (CE-BC integration). |
| 5 | Reduce case creation time by providing an asset-browser canvas app. |
| 6 | Provide accurate time-tracking per case for billing. |

---

## 3. Scope

### 3.1 In Scope

- Custom Dynamics 365 entities (6 new entities).
- Extensions to standard entities: **Incident (Case)**, **Customer Asset (msdyn_customerasset)**.
- Two Power Apps Canvas applications.
- Modified standard Case forms (7 form variants).
- AI Form-Fill configuration for the Case entity.
- Reference data structures for machine brand, model, and type.

### 3.2 Out of Scope

- Back-end ERP integration logic (CE-BC connector / middleware).
- Authentication and identity management.
- Power Automate flows (none present in this solution package).
- Reporting dashboards (not included in this package).
- Mobile offline configuration.

---

## 4. Actors and Personas

| Actor | Description | Primary Interaction |
|-------|-------------|---------------------|
| **Service Agent** | First-line support; creates and manages cases. | Case forms, Canvas Apps |
| **Field Engineer** | Executes on-site service; may update case details post-visit. | Case forms |
| **Service Manager** | Oversees queue, escalation, and KPIs. | Views, Dashboards (external) |
| **System Administrator** | Manages the Dynamics 365 environment. | Solution import, security roles |
| **Back-End System (CE-BC)** | ERP / supply chain system that consumes product requirements. | API / integration layer |

---

## 5. Functional Requirements

### 5.1 Asset Linkage on a Case

**FR-01 — Link Asset to Case**  
When creating or updating a Case, the agent **must** select a Customer Asset (`sit_asset`). This field is **required**.

**FR-02 — Link Sub-Asset (Component) to Case**  
The agent **may** optionally select a Sub-Asset (`sit_subasset`), which is a child asset of the selected asset. The lookup is filtered to assets that are children of the selected asset.

**FR-03 — Asset & Installation Info Display**  
The Case form displays contextual installation information based on the selected asset, without requiring the agent to navigate away.

### 5.2 Total Time Tracking

**FR-04 — Record Support Duration**  
The agent records the total time spent on a case in the `sit_totaltime` field (integer, minutes). This field appears on all Case form variants.

**FR-05 — AI Form Fill Exclusion**  
The AI-assisted form-fill feature is explicitly disabled for `sit_totaltime` to prevent incorrect AI-populated values. Time must be entered manually.

### 5.3 Case Product Lines

**FR-06 — Add Products Required for Case Resolution**  
Agents can create one or more **Case Product Line** records linked to a case, each specifying:
- Product (lookup to product catalogue) — **required**
- Quantity (integer) — **required**

**FR-07 — View Product Lines on Case**  
A sub-grid on the Case form lists all Case Product Lines for the current case, with inline add/edit capability.

**FR-08 — CE-BC Synchronisation**  
Case Product Lines serve as the data source for synchronisation with the back-end ERP. (Integration mechanism is external to this solution.)

### 5.4 Customer Asset Exploded View (Canvas App 1)

**FR-09 — Search Assets by Serial Number or Part Number**  
The agent enters a serial number or part number in a search box. The app returns matching Customer Assets.

**FR-10 — Browse Service Bill of Materials**  
For a selected asset, the app displays associated **Service Bill of Materials** records in a gallery.

**FR-11 — Browse Service Item Components**  
For a selected asset, the app displays associated **Service Item Components** in a gallery, showing:
- Product ID
- Product Name
- Serial Number
- Quantity

**FR-12 — Expand Installation Details**  
The agent can expand an installation entry to reveal its sub-components.

**FR-13 — Create Case from Asset Context**  
From the exploded view, the agent can initiate case creation pre-populated with the selected asset.

### 5.5 Service Item Components App (Canvas App 2)

**FR-14 — Browse Customer Assets**  
The agent uses a lookup control to navigate to a customer asset.

**FR-15 — View Service Items for an Asset**  
A gallery lists all service items linked to the selected asset.

**FR-16 — Add Products to New Product List**  
The agent selects service item components and adds them to a staging list (galNewProductsList).

**FR-17 — Manage Product List Before Submission**  
The agent reviews and adjusts the staged product list before it is committed as Case Product Lines.

### 5.6 Machine Reference Data

**FR-18 — Machine Brand Management**  
The `sit_MachineBrand` entity stores reference values for asset brands. Lookups from Customer Asset point to this entity.

**FR-19 — Machine Model Management**  
The `sit_MachineModel` entity stores reference values for asset models.

**FR-20 — Machine Type Management**  
The `sit_MachineType` entity stores reference values for asset types.

**FR-21 — Asset Classification**  
Each Customer Asset can be classified by brand, model, and type using the three reference lookups above.

### 5.7 Service Bill of Materials

**FR-22 — Define Component Requirements for Installations**  
The `sit_ServiceBillofMaterials` entity defines the expected parts/components for a given Customer Asset installation.

**FR-23 — Link BoM to Customer Asset**  
Each Service BoM record links to a Customer Asset via the `sit_servicebillofmaterials_Installation_msdyn_customerasset` relationship.

### 5.8 Service Item Components Entity

**FR-24 — Track Component-Level Detail**  
The `sit_ServiceItemComponents` entity stores individual component records with product and sub-asset references.

**FR-25 — Link to Sub-Asset**  
Each Service Item Component links to a Customer Asset sub-asset.

**FR-26 — Link to Product**  
Each Service Item Component links to a product in the product catalogue.

---

## 6. Data Model

### 6.1 Entity Summary

| Entity | Type | Purpose |
|--------|------|---------|
| `incident` | Standard (extended) | Service case |
| `msdyn_customerasset` | Standard (extended) | Customer machine/asset |
| `sit_CaseProductLine` | Custom | Products required per case |
| `sit_MachineBrand` | Custom | Reference — brand |
| `sit_MachineModel` | Custom | Reference — model |
| `sit_MachineType` | Custom | Reference — type |
| `sit_ServiceBillofMaterials` | Custom | Installation BoM |
| `sit_ServiceItemComponents` | Custom | Component-level parts list |

### 6.2 Key Fields on Incident (Case)

| Field | Schema Name | Type | Required | Notes |
|-------|-------------|------|----------|-------|
| Asset | `sit_asset` | Lookup → msdyn_customerasset | Yes | |
| Sub-Asset | `sit_subasset` | Lookup → msdyn_customerasset | No | Filtered by asset |
| Total Time | `sit_totaltime` | Integer (minutes) | Recommended | AI fill disabled |

### 6.3 sit_CaseProductLine Fields

| Field | Schema Name | Type | Required |
|-------|-------------|------|----------|
| Name | `sit_name` | Text (850) | Yes |
| Case | `sit_case` | Lookup → incident | Yes |
| Product | `sit_product` | Lookup → product | Yes |
| Quantity | `sit_quantity` | Integer | Yes |

### 6.4 Relationship Diagram (Logical)

```
incident (Case)
 ├── sit_asset ──────────────────► msdyn_customerasset
 │                                    ├── sit_machinebrand ──► sit_MachineBrand
 │                                    ├── sit_machinemodel ──► sit_MachineModel
 │                                    ├── sit_machinetype  ──► sit_MachineType
 │                                    └── Parent/Child (self-referencing)
 ├── sit_subasset ───────────────► msdyn_customerasset (child)
 └── sit_CaseProductLine (1:N)
      ├── sit_case ──────────────► incident
      └── sit_product ───────────► product

sit_ServiceBillofMaterials
 └── Installation ──────────────► msdyn_customerasset

sit_ServiceItemComponents
 ├── CustomerAssetSubAsset ─────► msdyn_customerasset
 └── Product ────────────────────► product
```

---

## 7. User Interface

### 7.1 Case Forms

Seven form variants are customised for different agent contexts:

| Form Name | Context |
|-----------|---------|
| Enhanced Quick Case Form | Rapid case entry |
| Enhanced Full Case Form | Full details |
| Case for Interactive Experience | Modern single-session UI |
| Case for Multisession Experience | Agent working multiple cases |
| Case for Swarming | Collaborative resolution |
| Case for Meeting Side Panel | Case opened alongside a Teams meeting |
| Information Form | Read-only view |

All forms include: Asset lookup, Sub-Asset lookup, Installation Info display, Total Time field, and Case Product Lines sub-grid.

### 7.2 Canvas App 1 — Customer Assets Exploded View

**Entry point**: Navigate to the app from the Customer Service Hub or via direct link.

**Key screens and controls**:

| Control | Purpose |
|---------|---------|
| `txtLookUpSNPN` | Serial number / part number search input |
| `galServiceBoMsList` | Gallery — Service Bill of Materials |
| `galServiceItemComponentsList` | Gallery — Service Item Components |
| `icArrowInstall` | Expand installation to show sub-components |
| `icCreateCase` | Launch case creation with asset context |

### 7.3 Canvas App 2 — Service Item Components

**Entry point**: Navigate to the app from a Customer Asset or Case context.

**Key screens and controls**:

| Control | Purpose |
|---------|---------|
| `conAssetLookUp` | Customer asset selector |
| `galServiceItems` | Gallery — service items for selected asset |
| `lblProductID / lblProductName / lblSerialNumber / lblQty` | Component detail labels |
| `btnAddToProductList` | Add selected items to staging list |
| `galNewProductsList` | Staging gallery for products before commit |

---

## 8. Integration Points

| Integration | Direction | Mechanism | Status |
|-------------|-----------|-----------|--------|
| CE-BC (Back-end ERP) | Outbound | External (not in this solution) | Interface defined via `sit_CaseProductLine` |
| Product Catalogue | Internal | Dataverse lookup | In scope |
| Customer Asset | Internal | Dataverse lookup | In scope |
| Dynamics 365 Sales (Opportunity / Quote) | Internal | Lookups from MachineBrand/Model/Type | In scope |
| IoT Connector | Dependency | msdyn IoT Alerts | Platform dependency |
| Omnichannel | Dependency | Routing & multi-session forms | Platform dependency |

---

## 9. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Platform | Microsoft Dynamics 365 Customer Service, CRM 9.2.26051.150+ |
| Deployment | Unmanaged solution import; target environment must have all listed dependencies |
| Security | Inherits Dynamics 365 role-based access control; no custom security roles defined in this solution |
| Performance | Standard Dataverse SLAs apply; no custom indexing defined |
| Availability | Inherits Dynamics 365 cloud SLA (99.9%) |
| Localisation | No custom language resources defined; English default |
| AI | AI form-fill opt-out applied to `sit_totaltime`; all other AI capabilities enabled by platform default |

---

## 10. Dependencies

The following managed solution packages must be installed in the target environment before importing SitechSupportUX:

| Package | Minimum Version | Purpose |
|---------|----------------|---------|
| msdynce_Service | 9.0.5.56 | Core case management |
| msdyn_AssetCommon | 1.0.50.1 | Customer Asset entity |
| msdyn_CaseManagementIntelligence | 9.2.26044.1067 | AI agent features |
| msdyn_CSIntelligence | 9.2.26044.1067 | Case suggestions |
| msdynce_Sales | 9.0.4.0076 | Opportunity / Quote integration |
| msdynce_ProductManagement | 9.0.25113.10001 | Product catalogue |
| IoTConnector | 1.8.53.0 | IoT alert integration |
| msdyn_ModernCaseManagement | 9.0.26042.2021 | Modern case features |
| OmnichannelBase | 1.10.0.146 | Omnichannel routing |

---

## 11. Glossary

| Term | Definition |
|------|-----------|
| Asset | A physical machine or equipment registered as a Customer Asset in Dynamics 365. |
| Sub-Asset | A component or module installed on or within a parent asset. |
| BoM | Bill of Materials — list of parts/components required for a service or installation. |
| Case Product Line | A record capturing a single product (and quantity) required to resolve a case. |
| CE-BC | Customer Engagement to Business Central — integration between Dynamics 365 and the ERP back-end. |
| Canvas App | A Power Apps application with a custom-designed UI, as opposed to a model-driven form. |
| Exploded View | A hierarchical display of an asset's components, similar to an engineering exploded diagram. |

---

*End of Functional Specification*

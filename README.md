# Dynamics 365 CE — Solution Analysis & Technical Documentation

> **Reverse-engineered functional specification and gap analysis** of a production
> Microsoft Dynamics 365 Customer Engagement implementation, produced by analysing
> managed solution packages without access to original design documents.

[![Platform](https://img.shields.io/badge/Platform-Power%20Platform-742774?logo=microsoft)](https://powerplatform.microsoft.com)
[![D365](https://img.shields.io/badge/D365-Customer%20Engagement-0078D4?logo=microsoft)](https://dynamics.microsoft.com)
[![Power Automate](https://img.shields.io/badge/Power%20Automate-3%20Flows%20Analysed-0066FF)](https://powerautomate.microsoft.com)
[![Dataverse](https://img.shields.io/badge/Dataverse-Custom%20Schema-green)](https://dataverse.microsoft.com)

---

## What This Repository Demonstrates

This project showcases the ability to:

- **Read and interpret managed D365 CE solution packages** — XML schema, form definitions, entity relationships, security roles
- **Reverse-engineer Power Automate flows** from raw JSON flow definitions — FetchXML queries, connector logic, condition branching, Dutch email templates
- **Produce senior-level BSA documentation** (functional specification + gap register) from code artefacts alone, without access to a running environment
- **Identify production bugs** in live automation flows through static analysis

---

## Solution Architecture

A four-solution managed deployment on Microsoft Dataverse / D365 CE:

```
┌─────────────────────────────────────────────────────────────────┐
│                     D365 CE Environment                          │
│                                                                  │
│  SitechSupportData  →  SitechSupportUX  →  SitechUX  →  Sales  │
│     (schema)            (forms/apps)      (training)   (nav)    │
│                                                                  │
│  ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │  Sales Hub   │  │ Customer Service │  │ Training Module  │  │
│  │              │  │                  │  │                  │  │
│  │ Lead→Order   │  │ Asset-linked     │  │ Session mgmt     │  │
│  │ pipeline     │  │ cases            │  │ 3 PA flows       │  │
│  │ 13 BC↔CE     │  │ 2 Canvas Apps    │  │ PDF certificates │  │
│  │ interfaces   │  │ Service BoM      │  │ via Docupack     │  │
│  └──────┬───────┘  └────────┬─────────┘  └────────┬─────────┘  │
│         └───────────────────┴──────────────────────┘            │
│                             │                                    │
│                      [Dataverse]                                 │
└─────────────────────────────┬───────────────────────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
     [Business Central]  [SharePoint]    [Docupack API]
       ERP / source of    Documents +     PDF certificate
       truth              flow assets     generation
```

**Technology stack**:

| Layer | Technology |
|-------|-----------|
| CRM Platform | Microsoft Dynamics 365 CE (Sales Hub + Customer Service Hub) |
| Database | Microsoft Dataverse (custom entities, FetchXML, security roles) |
| Automation | Power Automate — 1 Dataverse webhook + 2 scheduled recurrence flows |
| UI | Power Apps Canvas Apps (2 apps — asset exploded view + service item components) |
| Documents | SharePoint Online (document management + flow assets) |
| ERP | Microsoft Business Central (13 integration interfaces catalogued) |
| PDF generation | Docupack connector (certificate template rendering) |

---

## Repository Contents

```
/
├── README.md                        ← this file
├── FUNCTIONAL_SPECIFICATION.md      ← 47 functional requirements across 3 modules
├── GAPS_AND_OPEN_QUESTIONS.md       ← 52-item gap register, 16 blockers identified
└── solution/
    └── SitechSupportUX_1_0_0_2/    ← extracted managed solution XML
```

| Document | Description |
|----------|-------------|
| `FUNCTIONAL_SPECIFICATION.md` | Full BSA specification: Sales, Customer Service, and Training modules. Entity model, integration catalogue, 3 Power Automate flows fully documented with logic diagrams. v1.5 |
| `GAPS_AND_OPEN_QUESTIONS.md` | 52 gaps with impact/effort ratings, 16 blockers, recommendations. v1.5 |

---

## Training Module — Power Automate Flow Analysis

The most technically complex finding: a fully automated customer training communication
system built on Dataverse + Power Automate, discovered to be undocumented in any design document.

### Communication Lifecycle

```
Participant registration (sit_ContactTraining row created)
    │
    ├──► FLOW 1 — Dataverse webhook (immediate)
    │    Confirmation email → Dutch HTML template
    │    Logo fetched from SharePoint at send-time
    │    Sets: sit_confirmationsent = true
    │    ⚠️  Currently suspended (broken personal credential connection)
    │
    │    Daily 11:30 CET — T minus 4 days
    ├──► FLOW 2 — Scheduled recurrence
    │    Reminder email → two location-specific Dutch templates
    │    Harderwijk venue: floor plan image embedded from SharePoint
    │    Sets: sit_remindersent = true
    │    ⚠️  3 bugs — see gap register
    │
    │    Daily 09:50 CET — T plus 2 days
    └──► FLOW 3 — Scheduled recurrence
         Congratulations email + PDF certificate
         PDF generated via Docupack connector from Word template
         PDF attached to Dataverse email activity, sent via queue
         Sets: sit_sertificatesent = true
         ⚠️  Dead-code AND condition — wrong certificate type dispatched
```



---

*Solution versions analysed: SitechSupportData 1.0.0.2 · SitechSupportUX 1.0.0.3 · SitechUX 1.0.0.1 · June 2026*

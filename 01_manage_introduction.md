# Module 03: Maximo Manage — Introduction & Core Concepts

---

## 3.1 What is Maximo Manage?

Maximo Manage is the **Enterprise Asset Management (EAM)** application within MAS. It is the direct successor to IBM Maximo EAM 7.6.x and provides all the core EAM capabilities:

- Asset Lifecycle Management
- Work Order Management
- Preventive / Predictive Maintenance
- Inventory & Procurement
- Contracts Management
- Safety Management
- Service Management (ITSM/ITAM)
- Reporting and Analytics

Maximo Manage carries over the **entire Maximo data model** and feature set from 7.6.x while running on the modern OpenShift container platform.

---

## 3.2 Maximo Manage Versions

| Manage Version | MAS Version | Key Features |
|---------------|-------------|-------------|
| 8.3 | MAS 8.6 | Initial container-based release |
| 8.4 | MAS 8.7 | Performance improvements |
| 8.5 | MAS 8.8 | Enhanced mobile, Spatial |
| 8.6 | MAS 8.9 | AI-enhanced UI, Maps |
| 8.7 | MAS 8.10 | New features in Industry Solutions |
| 8.8 | MAS 8.11 | Latest as of 2024 |

---

## 3.3 Core Data Model — Key Objects

### Assets
An **Asset** is any physical or logical entity that requires maintenance.

| Field | Description |
|-------|-------------|
| Asset Number | Unique identifier (e.g., PUMP-001) |
| Description | Text description |
| Asset Type | Classification |
| Location | Where the asset is installed |
| Parent Asset | For hierarchical assets |
| Status | Active, Decommissioned, Operating, etc. |
| Priority | Criticality (1-99) |
| Failure Class | Links to failure codes |
| PM | Assigned Preventive Maintenance |

### Locations
A **Location** is a physical place where assets are installed or used.

Types:
- **Operating Location** — Where assets function (e.g., PLANT-01)
- **Storeroom** — Where inventory is held
- **Repair Facility** — Where assets are repaired
- **Labor Location** — Where technicians are based

### Work Orders (WO)
A **Work Order** authorizes and tracks maintenance work.

| WO Type | Description |
|---------|-------------|
| Corrective | Fix a broken asset |
| Preventive | Scheduled maintenance |
| Inspection | Periodic checks |
| Emergency | Urgent breakdown |
| Change | Planned modification |

**Work Order Lifecycle:**
```
WAPPR (Waiting for Approval)
    ↓
APPR (Approved)
    ↓
WSCH (Waiting for Scheduling)
    ↓
WMAT (Waiting for Materials)
    ↓
INPRG (In Progress)
    ↓
COMP (Completed)
    ↓
CLOSE (Closed)
    ↓
CAN (Cancelled)
```

### Job Plans
A **Job Plan** is a reusable template that defines:
- Task list (step-by-step instructions)
- Labor requirements
- Material requirements
- Tool requirements
- Safety procedures (Hazards, Precautions)

Job Plans are assigned to:
- Work Orders (manually or via PM)
- Preventive Maintenance (PM) records

### Preventive Maintenance (PM)
**PM** records automate the generation of work orders based on:
- **Calendar-based:** Every 30 days
- **Meter-based:** Every 500 hours, 10,000 miles
- **Combined (Whichever comes first)**
- **Combined (Whichever comes last)**

PM Generation Process:
```
PM Record
    ↓ (triggered by date or meter reading)
PMWO Job (background cron)
    ↓
Work Order Created (status: WAPPR or APPR)
    ↓
Assigned to Craft/Person
    ↓
Completed, Actual values recorded
    ↓
PM forecast updated
```

---

## 3.4 Inventory Management

### Items
An **Item** is a catalogued spare part or material:
- Item Number (e.g., FILT-100)
- Description
- Item Type (Material, Tool, Service, Software)
- Unit of Measure (EA, KG, LTR)
- Safety quantity (minimum stock level)

### Storerooms
Storerooms hold **Inventory** (stock of Items). One item can exist in multiple storerooms.

### Inventory Balances
- Current Balance
- Reorder Point
- Economic Order Quantity (EOQ)
- Safety Stock

### Purchase Orders (PO)
```
Purchase Requisition (PR)
    ↓ (approval flow)
Request for Quotation (RFQ)
    ↓
Purchase Order (PO)
    ↓
Receipts
    ↓
Invoice Matching
    ↓
Payment
```

### Inventory Transactions

| Transaction | Description |
|-------------|-------------|
| Issue | Remove item from storeroom for WO |
| Return | Return unused items back to storeroom |
| Transfer | Move items between storerooms |
| Physical Count | Reconcile actual vs system quantities |
| Adjustment | Manual quantity correction |

---

## 3.5 Labor Management

### Crafts and Skills
- **Craft** — Type of work skill (e.g., ELECTRICIAN, MECHANIC)
- **Skill Level** — Proficiency level (APPRENT, JOURNEY, MASTER)

### Labor Records
- Labor Code (employee ID)
- Name and contact details
- Craft assignments
- Calendar (availability, shifts)
- Crew membership

### Crew Management
- Crews are groups of labor resources
- Assigned together to work orders
- Used in scheduling

### Time Entry
Technicians report time against work orders:
- Regular hours
- Overtime hours
- Travel time
- Total actual labor cost calculated automatically

---

## 3.6 Service Management (Service Desk)

Maximo Manage includes Service Requests and Tickets for ITSM:

### Service Requests (SR)
- User-reported issues
- Can auto-create Work Orders
- SLA tracking built-in

### Incidents
- Unplanned interruption to a service
- Linked to Problems and Changes

### Problems
- Root cause analysis tracking
- Links multiple incidents

### Change Requests
- Planned changes to infrastructure/systems

This makes Manage usable for both **Physical Asset Management** (EAM) and **IT Asset Management** (ITSM/ITAM).

---

## 3.7 Contracts Management

Maximo Manage manages various contract types:

| Contract Type | Description |
|---------------|-------------|
| Blanket Purchase Order | Standing order for recurring purchases |
| Purchase Contract | Vendor pricing agreements |
| Warranty Contract | Tracks asset warranties |
| Lease/Rental Contract | Leased assets |
| Labor Rate Contract | Agreed labor rates per craft |
| Master Contract | Parent contract grouping |

### Contract Lifecycle
```
DRAFT → PENDING → APPROVED → ACTIVE → EXPIRED/CANCELLED
```

---

## 3.8 Maximo Manage — Applications List

Maximo Manage is organized into **Application Modules**. Each module contains multiple applications:

### Assets Module
- Assets
- Locations
- Asset Templates
- Meters
- Failure Codes
- Condition Monitoring
- Warranties
- Asset Attributes

### Work Orders Module
- Work Order Tracking
- Labor Reporting
- Activities and Tasks
- Releases
- Work Order Reorder

### Preventive Maintenance Module
- Preventive Maintenance
- Routes
- Job Plans
- Master PM

### Inventory Module
- Item Master
- Storerooms
- Inventory
- Physical Counts
- Shipping & Receiving
- Tools

### Purchasing Module
- Purchase Requisitions
- Purchase Orders
- Receiving
- Invoices
- RFQs
- Vendors

### Contracts Module
- Purchase Contracts
- Blanket POs
- Warranty Contracts
- Lease/Rental
- Labor Rate Contracts

### Planning Module
- Assignment Manager
- Crews
- Scheduling

### Service Desk Module
- Service Requests
- Incidents
- Problems
- Solutions
- Change Requests

### Administration Module
- Organizations
- Sets
- System Properties
- Security Groups
- Domains
- Escalations
- Cron Tasks
- Actions

---

## 3.9 Maximo Manage — Multi-Organization & Multi-Site

Manage supports complex organizational structures:

### Hierarchy
```
System (global)
    └── Organization (e.g., EAGLENA)
            └── Site (e.g., BEDFORD, DALLAS)
                    └── Assets / Locations / Work Orders
```

### Sets
**Sets** are shared data pools across organizations:
- **Item Set** — Items shared across orgs
- **Company Set** — Vendors/companies shared across orgs

This allows item catalogs to be shared while inventory remains site-specific.

---

## 3.10 Maximo Manage — User Interface Overview

### Classic Maximo UI
- Web-based, built on WebSphere/Liberty
- Organized into Start Centers (dashboards)
- Application list in left navigation
- KPI widgets, result sets, quick inserts

### Start Centers
Start Centers are personalized dashboards with:
- **Result Set Portlets** — Filtered lists of records (e.g., "My Open WOs")
- **KPI Portlets** — Numeric indicators (e.g., "Overdue PMs")
- **Bulletin Board** — Announcements
- **Quick Insert** — One-click record creation
- **Graph** — Bar/line charts of data

### Maximo New UI (NextGen / GraphQL-based)
In MAS 8.x, IBM introduced an updated UI framework:
- Built on **React**
- Uses **GraphQL** API
- Progressive web app capabilities
- Accessible on desktop and mobile browsers

---

## 3.11 Industry Solutions & Add-ons

Maximo Manage can be extended with Industry Solutions:

| Solution | Industry |
|----------|---------|
| **Aviation** | Airlines, MRO (Maintenance Repair Overhaul) |
| **Nuclear** | Nuclear power plants |
| **Oil & Gas** | Upstream/downstream O&G |
| **Utilities** | Electric/water/gas utilities |
| **Transportation** | Fleet and linear assets |
| **Life Sciences** | Pharmaceutical, biotech |
| **Asset Configuration Manager (ACM)** | Configuration-controlled assets |

---

## Summary

- Maximo Manage is the core EAM in MAS
- Key objects: Assets, Locations, Work Orders, PMs, Inventory, Crafts
- Work Order lifecycle drives all maintenance activities
- Multi-org/multi-site architecture for complex enterprises
- Industry Solutions extend capabilities for specific verticals

**Next:** [Maximo Manage — Work Management Deep Dive](./02_manage_work_management.md)

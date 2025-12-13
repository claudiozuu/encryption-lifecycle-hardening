# CELARD - COMSEC Encryption Lifecycle Automation & Risk Dashboard

CELARD is a **demo-ready** (sanitized) project that shows how to build an **encryption / key lifecycle monitoring dashboard** and **automated notification workflow** using **Power BI + Power Automate + Outlook**.

The goal is to provide **continuous visibility** into cryptographic/key lifecycle risk (e.g., approaching expiration) and automate **daily, non-spammy** alerts to assigned personnel.

> **Security Notice (Important):**  
> This repository contains **synthetic/sample data only**. It includes **no CUI/PII/COMSEC records**, no real key material, no unit identifiers, and no operational details. The implementation is intended to demonstrate governance, analytics, and automation patterns—not to replicate any protected system.

---

## What This Demonstrates

- **Encryption lifecycle governance** (inventory, expiration monitoring, status tracking)
- **Risk analytics** (near-expiration, expired, high-risk categories)
- **Compliance automation** (repeatable checks aligned to key management controls)
- **Operational workflow automation** (Power BI alert → Power Automate → Outlook)

---

## Architecture

**Data Source → Power BI Model → Risk Dashboard → Power BI Data Alert → Power Automate Flow → Outlook Email**

- **Power BI**: Tracks keys, computes days-to-expiration, risk categories, and KPIs  
- **Power BI Service Dashboard**: Hosts a **Card/KPI/Gauge** tile used for alerting  
- **Power Automate**: Executes a dataset query (DAX), checks for any expiring keys, and sends email notifications  
- **Outlook**: Receives “Action Required” notification (limited to **once per day**)

---

## Repository Layout
├─ README.md
├─ docs/
│ ├─ architecture.png
│ ├─ workflow-diagram.png
│ ├─ compliance-checklist.md
│ └─ screenshots/ # redacted images only
├─ data/
│ └─ sample_key_inventory.csv
├─ powerbi/
│ ├─ dax-measures.md
│ ├─ powerquery-m.md
│ └─ template-notes.md # how to build/export .pbit safely
└─ power_automate/
├─ flow-steps.md
└─ sample-email-template.md


---

## Data Model (Sample)

The demo dataset uses fields like:

- `Key_Type`
- `Expiration_Date`
- `Custodian_Role`
- `Status` (Active / Expired / Pending Destruction)
- `Days_Until_Expiration` (calculated)
- `Risk_Category` (calculated)

> Use the included sample file: `data/sample_key_inventory.csv`

---

## Key Power BI Logic (DAX)

Typical calculated columns/measures used:

- **Days Until Expiration**
- **Expired Flag**
- **Risk Category**
- **Keys Expiring Soon (<= 7 days)** — used for the alert card tile

See: `powerbi/dax-measures.md`

---

## Power BI Setup (Dashboard + Alert)

1. Import `data/sample_key_inventory.csv` into Power BI Desktop.
2. Add DAX columns/measures from `powerbi/dax-measures.md`.
3. Create a **Card** visual for: `KeysExpiringSoon` (<= 7 days).
4. Publish to Power BI Service.
5. Pin the **Card** to a **Dashboard** (alerts work on Card/KPI/Gauge tiles).
6. Create a **Data Alert** on the pinned card tile:
   - Condition: **Above 0**
   - Frequency: set to **Once per day / at most once every 24 hours** (recommended)

---

## Power Automate Setup (Email Notification)

Trigger options:
- **Preferred:** Power BI **data-driven alert** triggers the flow (recommended)
- Flow then queries the dataset and sends email when any matching rows exist.

Core steps:
1. **Trigger:** When a data-driven alert is triggered (Power BI)
2. **Action:** Run a query against a dataset (Power BI)
3. **Condition:** `length(First table rows) > 0`
4. **If yes:** Send Outlook email (V2)

Example DAX query (filter expiring within 7 days):

```DAX
EVALUATE
FILTER(
  'YourTableName',
  'YourTableName'[Days Until Expiration] <= 7
)


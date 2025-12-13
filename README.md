# CELARD — COMSEC Encryption Lifecycle Automation & Risk Dashboard

CELARD is a simple, portfolio-ready project that shows how to track **encryption/key lifecycle risk** (like upcoming expirations) and automatically send a **daily Outlook notification** using **Power BI + Power Automate**.

> **Important:** This repo uses **synthetic/demo data only**. Do **not** upload real COMSEC records, key IDs, unit info, custodian names/emails, destruction cert numbers, internal URLs, or screenshots containing sensitive details.

---

## What this project does

- Stores a **key inventory** (demo CSV) with expiration dates and ownership
- Uses **Power BI** to calculate:
  - Days until expiration
  - Expired vs active keys
  - Risk categories (critical/high/medium/low)
- Shows a **risk dashboard** and a **“Keys expiring soon”** number
- Uses a **Power BI dashboard alert** to trigger **Power Automate**
- Power Automate checks if any keys are expiring soon and sends **one email per day** via Outlook

---

## Architecture (high level)

**CSV/SharePoint → Power BI model → Dashboard Card tile → Data alert → Power Automate → Outlook email**

---

## Files in this repo

- `data/sample_key_inventory.csv` — demo dataset (safe, fake values)
- `powerbi/dax-measures.md` — DAX used for calculations and the alert measure
- `power_automate/flow-steps.md` — the Power Automate steps + DAX query
- `docs/` — optional diagrams and redacted screenshots

---

## Step-by-step setup

### 1) Create the demo dataset
Use `data/sample_key_inventory.csv` (or create your own with the same columns):

```csv
Key_ID,Key_Type,Classification,Expiration_Date,Custodian_Role,Device,Status,Destruction_Cert
KEY-001,TEK,CUI,2026-01-05,CUSTODIAN-A,DEVICE-A,Active,
KEY-002,KEK,CUI,2025-12-15,CUSTODIAN-B,DEVICE-B,Active,
KEY-003,TEK,CUI,2025-12-10,CUSTODIAN-C,DEVICE-C,Pending Destruction,DC-1003

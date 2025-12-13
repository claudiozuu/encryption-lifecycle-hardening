# CELARD — COMSEC Encryption Lifecycle Automation & Risk Dashboard
**COMSEC Encryption Lifecycle Automation & Risk Dashboard (CELARD)** is a portfolio/demo project showing how to monitor cryptographic key lifecycle risk (expiration, status, ownership) and automatically notify custodians **via Outlook** using **Power BI + Power Automate**.

> **Security / OPSEC Notice**
> - This repo uses **synthetic data only**.
> - Do **NOT** upload real key short titles, SKL inventories, custodian names/emails, destruction cert numbers, UICs, device IDs, locations, internal URLs, or screenshots showing live data.
> - Everything here is meant to demonstrate **governance + analytics + automation patterns**, not operational COMSEC records.

---

## Why I Built This (Learning Path → Working System)
I started with a simple goal:
1. Track key inventory + expiration in a structured dataset.
2. Turn that into a Power BI dashboard that highlights risk.
3. Automatically notify assigned personnel **without spamming** them.

Along the way I learned a few critical “gotchas”:
- **Power BI alerts do not work in reports** — they only work on **Dashboard tiles**.
- Alerts only work on **native Card / KPI / Gauge** visuals.
- In some tenants, Power Automate’s Power BI query step returns a generic object like **“First table rows”** instead of individual columns — so you must check `length(First table rows) > 0`.
- “Send email” failures often come from leftover references like `Apply_to_each` in the email body. Keep the body clean unless you intentionally build a loop.

This README documents the complete end-to-end build (dataset → Power BI → alert → Power Automate → Outlook).

---

## Tech Stack
- **Power BI Desktop / Power BI Service**
- **DAX (calculated columns + measures)**
- **Power Automate**
- **Outlook (Send email V2)**

---

## End State Architecture
**Data Source → Power BI Model → Risk Dashboard → (Pinned Card Tile) → Power BI Data Alert → Power Automate Flow → Outlook Email**

---


---

## Step 0 — Create the Dataset (Synthetic)
Create `data/sample_key_inventory.csv` like this:

```csv
Key_ID,Key_Type,Classification,Expiration_Date,Custodian_Role,Device,Status,Destruction_Cert
KEY-001,TEK,CUI,2026-01-05,CUSTODIAN-A,DEVICE-A,Active,
KEY-002,KEK,CUI,2025-12-15,CUSTODIAN-B,DEVICE-B,Active,
KEY-003,TEK,CUI,2025-12-10,CUSTODIAN-C,DEVICE-C,Pending Destruction,DC-1003

Minimum required fields to make the automation work:
Key_ID
Expiration_Date
Custodian_Role (or email mapping field)
Status

---

Step 1 — Power BI Desktop: Import Data 
1. Open Power BI Desktop
2. Get Data → Text/CSV
3. Select sample_key_inventory.csv
4. Ensure Expiration_Date is set to Date

---

Step 2 — Power BI Desktop: Add DAX Columns (Core Logic)
2.1 Days Until Expiration (Calculated Column) 
Go to Modeling → New column:

## DAX
Days Until Expiration =
DATEDIFF ( TODAY(), 'Clean Data'[Expiration_Date], DAY )

Replace 'Clean Data' with your actual table name if different.

2.2 Expired Flag (Calculated Column):

## DAX
Expired Flag =
IF ( 'Clean Data'[Days Until Expiration] <= 0, "Expired", "Active" )











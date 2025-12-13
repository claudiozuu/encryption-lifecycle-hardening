# COMSEC Encryption Lifecycle Automation & Risk Dashboard

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


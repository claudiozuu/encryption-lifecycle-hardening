# Encryption Lifecycle Hardening

Automated cryptographic key lifecycle governance using data analytics and workflow automation.

## What this is
This project demonstrates a data-governed approach to managing cryptographic key lifecycles. It uses a structured key inventory and lifecycle event logs to power a risk dashboard and automated Outlook alerts for keys requiring action (expiring soon, overdue destruction, custody anomalies).

## Why it matters
Manual key tracking increases operational risk: missed expirations, incomplete lifecycle evidence, and limited leadership visibility. This project shows how to operationalize encryption governance with measurable controls and automated workflows.

## Architecture
Data (Key Inventory + Logs) → Power BI Risk Dashboard → Power Automate Workflow → Outlook Notifications

## Key Features
- Governed data model for key inventory and lifecycle events
- Risk rules (days-to-expire, overdue destruction, custody anomalies)
- Power BI dashboard for real-time visibility and reporting
- Automated Outlook notifications to assigned custodians with optional escalation
- Policy checklist mapped to measurable evidence

## Standards Alignment (Conceptual) 
- AR 380-40
- TB 380-41
- NIST SP 800-57
- NIST SP 800-53 (SC-12, SC-13)
- FIPS 140-3

## Repo Structure
- data/ : synthetic sample datasets + data dictionary
- docs/ : architecture diagram, risk model, policy checklist, screenshots
- powerbi/ : DAX measures and report logic
- automations/ : Power Automate workflow steps

## Data Safety
All included data is synthetic/sanitized. No sensitive, operational, or classified cryptographic material is included.

## How to Run (High Level)
1) Review data dictionary and sample datasets in /data  
2) Build/import the Power BI model using the sample data  
3) Validate risk logic from /docs/risk_model.md  
4) Configure the Power Automate flow using /automations/power-automate/flow_steps.md  


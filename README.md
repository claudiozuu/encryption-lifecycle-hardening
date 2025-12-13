---
project:
  name: "**CELARD**"
  full_name: "**COMSEC Encryption Lifecycle Automation & Risk Dashboard**"
  purpose:
    - "Track cryptographic key lifecycle (**inventory, expiration, status**)"
    - "Visualize risk in **Power BI** (**expiring/expired keys**)"
    - "Automate **daily Outlook notifications** via **Power Automate**"
  security_notice:
    - "**Use synthetic data only** (no CUI/PII/COMSEC records)."
    - "**Do not include** real key IDs, short titles, SKL inventories, destruction certs, unit identifiers, internal URLs, or emails."
    - "**Redact screenshots** before publishing."

repo_structure:
  recommended:
    - "**README.md**"
    - "**data/sample_key_inventory.csv**"
    - "**docs/architecture.png**"
    - "**docs/workflow-diagram.png**"
    - "**powerbi/dax-measures.md**"
    - "**power_automate/flow-steps.md**"
    - "**power_automate/sample-email-template.md**"

steps:
  - step: "**Step 0**"
    name: "**Create the Dataset (Synthetic)**"
    goal: "Create a safe, fake dataset that mirrors a COMSEC key inventory for demo/testing."
    files:
      - path: "**data/sample_key_inventory.csv**"
        type: "csv"
        template: |
          Key_ID,Key_Type,Classification,Expiration_Date,Custodian_Role,Device,Status,Destruction_Cert
          KEY-001,TEK,CUI,2026-01-05,CUSTODIAN-A,DEVICE-A,Active,
          KEY-002,KEK,CUI,2025-12-15,CUSTODIAN-B,DEVICE-B,Active,
          KEY-003,TEK,CUI,2025-12-10,CUSTODIAN-C,DEVICE-C,Pending Destruction,DC-1003
    notes:
      - "**Expiration_Date must be parseable as a Date.**"
      - "**Custodian_Role** can later be replaced with a demo email mapping if needed."

  - step: "**Step 1**"
    name: "**Import Data into Power BI Desktop**"
    goal: "Load the CSV and ensure correct data types."
    actions:
      - "**Open Power BI Desktop**"
      - "**Get Data → Text/CSV** → select `data/sample_key_inventory.csv` → **Load**"
      - "In **Power Query**: confirm **Expiration_Date** is **Date** type"
      - "**Close & Apply**"
    validations:
      - "**Expiration_Date** column is of type **Date** (not Text)."

  - step: "**Step 2**"
    name: "**Add Core DAX Calculated Columns**"
    goal: "Create lifecycle logic for expiration and risk labeling."
    assumption:
      table_name: "**Clean Data**"   # change if your table name differs
      expiration_column: "**Expiration_Date**"
    dax:
      calculated_columns:
        - name: "**Days Until Expiration**"
          code: |
            Days Until Expiration =
            DATEDIFF ( TODAY(), 'Clean Data'[Expiration_Date], DAY )
        - name: "**Expired Flag**"
          code: |
            Expired Flag =
            IF ( 'Clean Data'[Days Until Expiration] <= 0, "Expired", "Active" )
        - name: "**Risk Category**"
          code: |
            Risk Category =
            IF (
                'Clean Data'[Days Until Expiration] <= 0, "Expired",
                IF (
                    'Clean Data'[Days Until Expiration] <= 7, "Critical (<=7d)",
                    IF (
                        'Clean Data'[Days Until Expiration] <= 30, "High (<=30d)",
                        IF ( 'Clean Data'[Days Until Expiration] <= 60, "Medium (<=60d)", "Low (>60d)" )
                    )
                )
            )
    notes:
      - "Power BI: **Modeling → New column** for each calculated column above."
      - "**Replace `'Clean Data'`** with your actual table name if different."

  - step: "**Step 3**"
    name: "**Create Alert Measure (for Card/KPI/Gauge)**"
    goal: "Compute a single number that can trigger a Power BI data alert."
    assumption:
      table_name: "**Clean Data**"
    dax:
      measures:
        - name: "**KeysExpiringSoon**"
          code: |
            KeysExpiringSoon =
            CALCULATE (
                COUNTROWS ( 'Clean Data' ),
                'Clean Data'[Days Until Expiration] <= 7
            )
    notes:
      - "Power BI: **Modeling → New measure**."
      - "This measure must be used in a **native Card visual** pinned to a **dashboard tile**."

  - step: "**Step 4**"
    name: "**Build Power BI Report Visuals**"
    goal: "Create dashboard-ready visuals (including a native Card for alerting)."
    visuals:
      required:
        - name: "**Card — Keys Expiring Soon**"
          type: "**Card**"
          field: "**KeysExpiringSoon**"
          formatting_rules:
            - "**Keep it a native Card** (avoid custom visuals)."
            - "Optional: conditional formatting by value (visual cue)."
        - name: "**Table — Expiring Keys**"
          type: "**Table**"
          fields:
            - "**Key_ID**"
            - "**Key_Type**"
            - "**Expiration_Date**"
            - "**Days Until Expiration**"
            - "**Custodian_Role**"
            - "**Risk Category**"
            - "**Status**"
      optional:
        - name: "Bar — Keys by Risk Category"
          type: "Bar chart"
          x: "Risk Category"
          y: "Count of Key_ID"
        - name: "Bar — Keys by Custodian"
          type: "Bar chart"
          x: "Custodian_Role"
          y: "Count of Key_ID"
    notes:
      - "**Alerts only work on Card/KPI/Gauge tiles pinned to a dashboard** (not charts)."

  - step: "**Step 5**"
    name: "**Publish Report & Pin Card to Dashboard**"
    goal: "Move from Desktop to Service and create a dashboard tile eligible for alerts."
    actions:
      - "Power BI Desktop: **Publish** → select workspace"
      - "Power BI Service: open the **Report**"
      - "Locate the **native Card** visual (**KeysExpiringSoon**)"
      - "Click **Pin (📌)** → Pin to **Dashboard** (e.g., **COMSEC Dashboard**)"
    validations:
      - "Pinned tile is a **native Card** (alerts require **Card/KPI/Gauge**)."

  - step: "**Step 6**"
    name: "**Create Power BI Data Alert (Once per Day)**"
    goal: "Set an alert on the dashboard tile and limit notifications to **1x/day**."
    actions:
      - "Open the **Dashboard** (not the report)"
      - "On the pinned Card tile: **… → Manage alerts**"
      - "**Add alert rule**"
    alert_rule:
      title: "**COMSEC Keys Expiring (<=7d)**"
      condition: "**is above**"
      threshold: "**0**"
      frequency: "**Once per day / At most once every 24 hours**"
      email_notifications: "**true**"
    notes:
      - "If you don’t see alert options, you pinned the wrong visual type (**must be Card/KPI/Gauge**)."

  - step: "**Step 7**"
    name: "**Power Automate Flow — Trigger on Data Alert**"
    goal: "Start the automation whenever the Power BI alert fires."
    power_automate:
      flow_type: "**Automated cloud flow**"
      trigger:
        connector: "**Power BI**"
        name: "**When a data-driven alert is triggered**"
        configuration:
          alert_id: "**Select the alert created in Step 6**"
    notes:
      - "Do **not** add custom trigger-header hacks; enforce **once/day** via **Power BI alert frequency**."

  - step: "**Step 8**"
    name: "**Power Automate — Query Dataset for Expiring Keys**"
    goal: "Pull the rows that match the expiration window using **DAX**."
    power_automate:
      action:
        connector: "**Power BI**"
        name: "**Run a query against a dataset**"
        configuration:
          workspace: "**<Your Workspace>**"
          dataset: "**<Your Dataset>**"
          query_text: |
            EVALUATE
            FILTER (
                'Clean Data',
                'Clean Data'[Days Until Expiration] <= 7
            )
    validations:
      - "Test run returns **First table rows** (fields may not appear individually—this is expected)."
    notes:
      - "**Replace `'Clean Data'`** with your actual table name."

  - step: "**Step 9**"
    name: "**Power Automate — Condition (Any Rows Returned?)**"
    goal: "Send email only if at least one row exists."
    power_automate:
      action:
        connector: "**Control**"
        name: "**Condition**"
        logic:
          left_expression: "**length(body('Run_a_query_against_a_dataset')?['FirstTableRows'])**"
          operator: "**is greater than**"
          right_value: "**0**"
    notes:
      - "Use **Expression tab** for the left side."
      - "This avoids needing the column name in dynamic content."

  - step: "**Step 10**"
    name: "**Power Automate — Send Outlook Email (If yes)**"
    goal: "Notify custodians/S6 distribution **once daily** when expiring keys exist."
    power_automate:
      if_yes_actions:
        - connector: "**Office 365 Outlook**"
          name: "**Send an email (V2)**"
          to: "**custodian.distro@example.mil; s6.shop@example.mil**"
          subject: "**🔐 CELARD Alert: Keys Expiring in <= 7 Days — Action Required**"
          body: |
            **CELARD Alert Triggered (Daily Notice)**

            One or more keys are expiring within the next 7 days.
            Please open the Power BI COMSEC dashboard and review the expiring key list.

            Recommended actions:
            - Coordinate rollover/reissue as applicable
            - Validate custody/transfer handling
            - Complete destruction documentation for retired keys

            (Synthetic/demo project. No real COMSEC data included.)
      if_no_actions:
        - "**No action**"
    troubleshooting:
      - "If email step fails to save referencing **Apply_to_each**: remove leftover dynamic content from the body and save again."
      - "Separate multiple recipients with **semicolons (;)**."

  - step: "**Step 11**"
    name: "**End-to-End Test**"
    goal: "Verify the entire system works from dashboard alert to Outlook email."
    test_plan:
      - "In sample CSV, set one key **Expiration_Date** within **7 days**"
      - "Refresh dataset in **Power BI Service**"
      - "Confirm Card (**KeysExpiringSoon**) updates to **>0**"
      - "Confirm alert fires (**once per day**) and triggers the flow"
      - "Confirm query returns rows, condition passes, and email is sent"
    success_criteria:
      - "**Only one email per day** (enforced by Power BI alert frequency)."
      - "Email sends **only when expiring keys exist**."
...











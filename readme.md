# 🚨 ServiceNow ITSM Automation: VIP & Critical Incident Escalation Pipeline

![Platform](https://img.shields.io/badge/Platform-ServiceNow%20(Australia)-032D42?style=for-the-badge&logo=servicenow)
![Module](https://img.shields.io/badge/Module-ITSM%20%26%20Flow%20Designer-293E40?style=for-the-badge)
![Methodology](https://img.shields.io/badge/Framework-ITIL%20v4-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed%20Lab-success?style=for-the-badge)

## 📌 Project Overview & Business Problem
In an enterprise IT Service Desk handling hundreds of daily tickets, manual triage often leads to **Service Level Agreement (SLA) breaches**—especially when high-impact network outages or VIP executive requests sit unassigned in a general queue.

This project implements an automated, zero-touch **Incident Escalation & Routing Workflow** using **ServiceNow Flow Designer (Workflow Studio)**. When an incoming incident meets critical business criteria (such as a VIP caller or a high-urgency network issue), the workflow automatically recalculates priority, assigns the ticket to the Tier-2 Network Operations group, logs an internal audit work note, and dispatches an immediate escalation alert.

---

## 🎯 Project Objectives
1. **Eliminate Manual Triage Delays:** Automatically route critical incidents to the appropriate L2/L3 resolver group within seconds of creation.
2. **Enforce ITIL Priority Matrix Standards:** Dynamically evaluate `Impact` and `Urgency` alongside Caller attributes (`VIP = true`) to enforce consistent incident prioritization.
3. **Maintain Audit Trail Compliance:** Automatically inject timestamped system Work Notes into the incident journal and trigger targeted stakeholder notifications.
4. **Package for Enterprise Deployment:** Capture all configuration changes inside a portable ServiceNow **Update Set** (`.xml`) for version control and migration.

---

## 🛠️ Skills & Competencies Demonstrated
*Hiring managers and technical recruiters reviewing this repository can verify hands-on proficiency in:*

| Skill Category | Specific Technical Implementation |
| :--- | :--- |
| **ITIL Incident Management** | Navigating the `incident` table schema, understanding Impact/Urgency priority matrices, and managing Assignment Groups. |
| **Workflow Automation** | Building no-code/low-code event-driven pipelines in **ServiceNow Flow Designer / Workflow Studio**. |
| **Relational Data Pill Dot-Walking** | Traversing relational database tables (`Incident -> Caller [sys_user] -> VIP`) to evaluate dynamic conditional logic. |
| **System Auditing & Notifications** | Configuring automated Work Notes injection and dynamic HTML email/system dispatches using runtime record variables. |
| **Platform Administration & DevOps** | Testing execution paths via Flow Execution Contexts and exporting changes via **Local Update Sets** (`sys_remote_update_set`). |

---

## 🏗️ Workflow Architecture

```text
[Trigger: Incident Created] 
       │
       ▼
[Condition Filter: Category == 'Network' OR Caller.VIP == true]
       │
       ├──► (False) ──► [Standard Help Desk Queue - No Automation]
       │
       └──► (True)  ──► [Action 1: Update Incident Record]
                               ├─ Set Impact = 1 - High
                               ├─ Set Urgency = 1 - High (Auto-calculates Priority = 1 - Critical)
                               ├─ Set Assignment Group = 'Network'
                               └─ Append Work Note: "AUTO-ESCALATION: Routed to Tier-2 Network Ops"
                                      │
                                      ▼
                        [Action 2: Send Notification / Log Alert]
                               └─ Dispatch P1 Escalation Email to Network Assignment Group
```

---

## 📦 Repository Deliverables & Evidence
To prove functional implementation, this repository includes the following artifacts:

```text
├── README.md                          # Project documentation & architecture overview
├── LAB_INSTRUCTIONS.md                # Step-by-step Tinker lab guide & technical checkpoints
├── SN_VIP_Network_Escalation.xml      # Exported ServiceNow Update Set containing the Flow
└── screenshots/
    ├── 01_baseline_incident_form.png  # Proof of VIP user setup & baseline Incident table
    ├── 02_flow_designer_logic.png     # Full Flow Designer canvas showing Trigger, Conditions & Data Pills
    ├── 03_flow_execution_test.png     # Flow Designer Test Run showing "Completed" green checkmarks
    └── 04_updated_incident_audit.png  # Resulting Incident showing P1 Priority, Network Group & Work Notes
```

---

## 📸 Implementation Walkthrough & Proof of Work

### 1. Trigger & Relational Condition Logic
![Flow Designer logic](screenshots/02_flow_designer_logic.png)

* **What this shows:** Configured a record trigger on the `Incident [incident]` table. Used dot-walking (`Caller -> VIP is true`) combined with category evaluation to isolate critical tickets without writing brittle client scripts.

### 2. Dynamic Record Mutation & Work Notes
![Baseline incident form](screenshots/01_baseline_incident_form.png)

* **What this shows:** Used the **Update Record** action block with dynamic Data Pills (`Trigger -> Incident Record -> Caller -> Name`) to stamp an auditable work note directly into the ticket's activity stream while reassigning ownership to the `Network` group.

### 3. Execution Validation & Debugging
![Flow execution test](screenshots/03_flow_execution_test.png)

* **What this shows:** Verified end-to-end execution using Flow Designer's built-in testing engine, inspecting the runtime execution context to confirm zero errors and sub-second processing time.

### 4. Updated Incident Audit
![Updated incident audit](screenshots/04_updated_incident_audit.png)

* **What this shows:** Confirmed the completed escalation: P1 priority, assignment to the Network group, and the automatic escalation work note recorded on the incident.

---

## 🚀 How to Import & Test This Project
If you have your own ServiceNow Personal Developer Instance (PDI), you can deploy this workflow in under 2 minutes:
1. Log into your ServiceNow instance as an `admin`.
2. Navigate to **All > System Update Sets > Retrieved Update Sets**.
3. Click **Import Update Set from XML** at the bottom of the list.
4. Upload `SN_VIP_Network_Escalation.xml` from this repository.
5. Open the imported record, click **Preview Update Set**, and then click **Commit Update Set**.
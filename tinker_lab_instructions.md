# ✏️ Tinker Instructions: ServiceNow Incident Escalation Lab

Welcome to your hands-on implementation guide! This lab is structured like a CodePath Tinker session. Follow each part in order, check off the boxes as you complete them in your `dev340530` instance, and answer the **Checkpoint** questions to make sure you can explain *why* everything works during an IT interview.

---

## 📋 Lab Overview & Deliverables Checklist
By the end of this ~45-minute lab, you will have produced **4 screenshots** and **1 XML Update Set file** to upload to your GitHub repository:
- [ ] `01_baseline_incident_form.png` — A VIP caller profile and a test incident record.
- [ ] `02_flow_designer_logic.png` — Your completed automation pipeline inside Workflow Studio.
- [ ] `03_flow_execution_test.png` — The green execution logs proving your logic passed.
- [ ] `04_updated_incident_audit.png` — The incident activity stream showing automated assignment and work notes.
- [ ] `SN_VIP_Network_Escalation.xml` — Your exported ServiceNow Update Set.

---

## ▾ Part 1: Prepare the Environment & Package Your Work

> ⏰ **~8 mins**
> Before building anything in enterprise ServiceNow, administrators create an **Update Set**. This acts like a Git branch—it records every rule you build so you can export it as an `.xml` file for your GitHub portfolio.

- [ ] Open your instance (`https://dev340530.service-now.com`) and click **All** in the top-left corner.
- [ ] Type `Local Update Sets` in the filter navigator and click **System Update Sets > Local Update Sets**.
- [ ] Click the **New** button at the top right.
- [ ] Set **Name** to `Portfolio Lab - VIP Incident Escalation` and click **Submit and Make Current**.
- [ ] Now let's verify a VIP user exists for testing. Click **All**, type `Users`, and click **User Administration > Users**.
- [ ] Search for `Abel Tuter` (a built-in demo user) and open his profile.
- [ ] Check the **VIP** checkbox on his form (if it isn't checked already) and click **Update**.
- [ ] Click **All**, type `Incident`, and click **Incident > Create New**.
- [ ] Set **Caller** to `Abel Tuter` (notice the red `[VIP]` badge appears next to his name!), **Category** to `Network`, and **Short description** to `Baseline Test - VPN Gateway Unreachable`. Leave **Assignment group** blank and **Impact/Urgency** at `3 - Low`.
- [ ] Take a screenshot of this form and save it as `01_baseline_incident_form.png`, then click **Submit**.

> 📍 **Checkpoint:** You now have an active Update Set tracking your work, and Incident record (e.g., `INC0010001`) sitting in the database with `Priority = 5 - Planning` and no Assignment Group.
>
> **Interview Prep Question:** *Why can't you manually select "1 - Critical" in the Priority dropdown on a standard ServiceNow Incident form?*
> *(Hint: Look at how ServiceNow calculates Priority from two other fields: Impact + Urgency).*

---

## ▾ Part 2: Create the Flow & Configure a Relational Trigger

> ⏰ **~10 mins**
> Move from manual ticket triage to event-driven automation. Instead of triggering on *every* ticket, you will filter for Network issues reported by VIP executives using **dot-walking**.

- [ ] Click **All** in the top bar, type `Flow Designer` (or click **Open Workflow Studio** on your landing page).
- [ ] In Workflow Studio, click **New > Flow** (top right).
- [ ] Fill out the Flow Properties modal:
  - **Flow Name:** `Auto-Escalate VIP Network Incidents`
  - **Description:** `Automatically elevates Impact/Urgency, assigns to Network group, and logs audit notes when a VIP submits a Network incident.`
  - **Application:** `Global`
  - **Run As:** `System User` (Click **Submit**).
- [ ] Under the **TRIGGER** section, click **Add a trigger** and select **Record > Created**.
- [ ] Set **Table** to `Incident [incident]`.
- [ ] Under **Condition**, click **Add Filter Condition**:
  - Set the first condition: `Category` | `is` | `Network`.
  - Click **AND**.
  - For the second field, we need to check if the Caller is a VIP, even though `VIP` lives on the `User` table, not the `Incident` table. Click the field dropdown, scroll to `Caller`, click the **arrow icon (`>`)** next to Caller to "dot-walk" into Caller fields, and select `VIP`.
  - Set the condition to: `Caller.VIP` | `is` | `true`.
- [ ] Click **Done** on the trigger block.

> 💡 **Unsure what Dot-Walking just did?** The `Incident` table only stores a reference ID (`sys_id`) pointing to the `Caller`. By clicking the arrow next to `Caller`, you traversed a foreign-key relationship (`incident.caller_id.vip`) without writing a SQL `JOIN`!

> 📍 **Checkpoint:** Your Trigger block should read: *Trigger: Record Created on Incident [incident] where Category is Network AND Caller.VIP is true.*
>
> **Interview Prep Question:** *Why is it better to put `Category is Network` in the Trigger condition rather than triggering the flow on ALL incidents and checking the Category inside an `If` block later?*
> *(Think about system performance when 10,000 non-network tickets are created per day).*

---

## ▾ Part 3: Mutate the Incident Record & Inject Dynamic Data Pills

> ⏰ **~12 mins**
> Now tell ServiceNow what to change on the ticket when the trigger fires, and use **Data Pills** (runtime variables) to write a personalized audit note.

- [ ] Under the **ACTIONS** section, click **Add an Action, Flow Logic, or Subflow** -> select **Action**.
- [ ] Search for `ServiceNow Core` > **Update Record** and select it.
- [ ] **Populate the Record field:** Look at the **Data** panel on the right side of the screen. Drag the **`Incident Record`** data pill (under `Trigger - Record Created`) and drop it into the **Record** box. *(Notice the **Table** field automatically locks to `Incident [incident]`)*.
- [ ] Click **+ Add Field Value** four times and configure these exact updates:
  1. `Impact` | `1 - High`
  2. `Urgency` | `1 - High` *(Setting both to 1 will force ServiceNow's backend business rules to calculate `Priority = 1 - Critical`)*
  3. `Assignment group` | `Network`
  4. `Work notes` | Type the following text, dragging Data Pills from the right panel where indicated in brackets:
     ```text
     [AUTOMATED ESCALATION]: Incident submitted by VIP User [Drag: Trigger > Incident Record > Caller > Name]. Due to Network impact, ticket has been automatically elevated to P1-Critical and routed to the Network Operations queue.
     ```
- [ ] Click **Done** on Action 1.

> 📍 **Checkpoint:** Your `Update Record` action has 4 fields configured, and the `Work notes` field contains a rounded gray pill for `Name` alongside your static text.
>
> **Interview Prep Question:** *In ServiceNow, what is the difference between updating `Work notes` versus `Additional comments` on an Incident?*
> *(Hint: One is visible only to internal IT technicians, while the other emails the customer directly).*

---

## ▾ Part 4: Add Automated Stakeholder Notification

> ⏰ **~8 mins**
> A critical ticket assigned to a queue still needs to alert on-call engineers immediately. Let's add a secondary notification action.

- [ ] Below Action 1, click **Add an Action, Flow Logic, or Subflow** -> **Action**.
- [ ] Search for `ServiceNow Core` > **Send Email** (or **Log** if you want to write to the system log as well) and select **Send Email**.
- [ ] Drag the **`Incident Record`** data pill from the right panel into the **Target Record** field (this attaches the email to the ticket's audit history).
- [ ] Checked **Include Watermark**.
- [ ] In the **To** field, type `network.admin@yourcompany.com` (or drag `Trigger > Incident Record > Assignment group > Manager > Email` from the right panel!).
- [ ] Set **Subject** to:
  ```text
  🚨 [P1 VIP ALERT] [Drag: Trigger > Incident Record > Number] - [Drag: Trigger > Incident Record > Short description]
  ```
- [ ] In the **Body**, write a brief summary using Data Pills for `Caller > Name`, `Category`, and `Priority`.
- [ ] Click **Done**, then click **Save** in the top-right corner of Flow Designer.
- [ ] Take a clean screenshot showing your full Flow canvas (Trigger + Action 1 + Action 2) and save it as `02_flow_designer_logic.png`.

---

## ▾ Part 5: Test, Verify Audit Trail & Export Update Set

> ⏰ **~7 mins**
> Never deploy an untested flow. You will now run your baseline incident through the engine, inspect the execution context, and export your work for GitHub.

- [ ] Click the **Test** button in the top-right corner of Flow Designer.
- [ ] In the **Test Flow** modal, search for the baseline incident you created in Part 1 (`Baseline Test - VPN Gateway Unreachable`) and click **Run Test**.
- [ ] Wait 3 seconds, then click **Your test has finished running. View the flow execution details.**
- [ ] Expand the steps in the Execution Details tab so you can see `Trigger (Completed)`, `Update Record (Completed)`, and `Send Email (Completed)` in green.
- [ ] Take a screenshot of this execution log and save it as `03_flow_execution_test.png`.
- [ ] Click the **Activate** button in Flow Designer so your flow runs live on future tickets.
- [ ] Now go back to your main ServiceNow tab, click **All > Incident > Open**, and open your test incident (`Baseline Test - VPN Gateway Unreachable`).
- [ ] Verify that **Priority** is now `1 - Critical`, **Assignment group** is `Network`, and your custom **Work note** is stamped in gold in the Activity stream at the bottom!
- [ ] Take a screenshot of this updated incident and save it as `04_updated_incident_audit.png`.
- [ ] **Final Export Step:** Click **All > System Update Sets > Local Update Sets**, open `Portfolio Lab - VIP Incident Escalation`, change **State** from `In progress` to `Complete`, and click **Save**. Then click the **Export to XML** related link at the bottom of the form and rename the downloaded file to `SN_VIP_Network_Escalation.xml`.

> 📍 **Final Checkpoint:** You now have all 4 screenshots and the `.xml` Update Set ready to commit to GitHub!
# Implement-Client-Script-UI-Policy-Incident

Implementation of **UI Policies** and **Client Scripts** on the ServiceNow **Incident** table to enforce data integrity, automate field values, and block invalid submissions.

📄 [Project Documentation (Google Doc)](https://docs.google.com/document/d/1-OGz4h8Btd-tTCb51romxUZcjEvhlqGPvfiXoEl1dJA/edit?usp=sharing)

---

## 📌 Problem Statement

Incident records need consistent and accurate data to support effective triage, routing, and resolution. Relying only on user awareness and manual checks can lead to incomplete, inconsistent, or incorrect data. This affects reporting accuracy, SLA compliance, and overall service quality.

To address this, conditional field behavior and validation must be enforced directly at the user interface level using standardized **UI Policies** and **Client Scripts**.

## 🎯 Objective

Demonstrate how ServiceNow client-side controls can enforce data integrity on Incident records by:

- Making fields mandatory or read-only based on conditions
- Auto-populating values
- Preventing record submission when required conditions are not met
- Blocking unwanted edits directly from list views

## 🛠️ Skills and Technologies

- Incident Management
- UI Policy and UI Policy Actions
- Client Scripts (onChange, onSubmit, onCellEdit)
- Form Validation
- ServiceNow, JavaScript

## 🧩 Solution Summary

| # | Component                          | Type                  | Behavior                                                        |
|---|------------------------------------|-----------------------|-----------------------------------------------------------------|
| 1 | High Impact Control                | UI Policy             | When Impact = 1 - High, Assignment group becomes mandatory      |
| 2 | Urgency action                     | UI Policy Action      | When Impact = High, Urgency becomes read-only                   |
| 3 | Auto set urgency for high impact   | Client Script (onChange) | Setting Impact to High sets Urgency to High automatically    |
| 4 | Prevent save if assigned to missing| Client Script (onSubmit) | Blocks saving a High impact incident without Assigned to     |
| 5 | Prevent state change via list edit | Client Script (onCellEdit) | Blocks changing State directly from the list view          |

---

## Task 1: Create UI Policy on Incident

1. Log in to the ServiceNow instance with administrative access.
2. Go to **System UI → UI Policies** and click **New**.
3. Fill in:
   - **Name / Short description:** High Impact Control
   - **Table:** Incident
   - **Active:** true
4. Under **When to Apply**, set the condition **Impact → is → 1 - High**.
5. Check **Reverse if false** so changes are reverted when the condition is not met (and keep **On load** checked).
6. Click **Submit**.
7. Open the policy, and in **UI Policy Actions** create a new action:
   - **Field name:** Assignment group
   - **Mandatory:** True
8. Click **Submit**.

---

## Task 2: Create UI Policy Action – Urgency

1. Open the **High Impact Control** UI Policy and scroll to the **UI Policy Actions** related list.
2. Click **New** and fill in:
   - **Field name:** Urgency
   - **Read only:** True
   - **Visible:** Leave alone
3. Click **Submit**.
4. Verify: open an Incident, set **Impact** to **High**, and confirm **Urgency** becomes read-only. Change Impact to another value and confirm the field becomes editable again.

---

## Task 3: Create onChange Client Script

Go to **System UI → Client Scripts → New** and fill in:

| Field      | Value                              |
|------------|------------------------------------|
| Name       | Auto set urgency for high impact   |
| Table      | Incident                           |
| UI Type    | Desktop                            |
| Type       | onChange                           |
| Field name | Impact                             |
| Active     | true                               |

```javascript
function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading || newValue == '') {
        return;
    }

    if (newValue == '1') {
        g_form.setValue('urgency', '1');
        g_form.addInfoMessage('Urgency set to High for High impact incident.');
    }
}
```

Verify: change **Impact** to **High** on an Incident. **Urgency** updates automatically.

---

## Task 4: Create onSubmit Client Script (Save Validation)

| Field   | Value                                |
|---------|--------------------------------------|
| Name    | Prevent save if Assigned To missing  |
| Table   | Incident                             |
| UI Type | Desktop                              |
| Type    | onSubmit                             |
| Active  | true                                 |

```javascript
function onSubmit() {
    if (g_form.getValue('impact') == '1' &&
        g_form.getValue('assigned_to') == '') {

        g_form.showErrorBox(
            'assigned_to',
            'Assigned To is mandatory for High impact incidents.'
        );
        return false;
    }
    return true;
}
```

Verify: try saving a High impact Incident without **Assigned to**. The save is blocked and an error is shown.

---

## Task 5: Create onCellEdit Client Script

| Field      | Value                                 |
|------------|---------------------------------------|
| Name       | Prevent state change via list edit    |
| Table      | Incident                              |
| UI Type    | Desktop                               |
| Type       | onCellEdit                            |
| Field name | State                                 |
| Active     | true                                  |

```javascript
function onCellEdit(sysIDs, table, oldValues, newValue, callback) {
    alert('State cannot be updated using list editing. Please open the Incident.');
    callback(false);
}
```

Verify: in the Incident list, try to change **State** directly. An alert appears and the value stays unchanged.

---

## Task 6: Testing the Configuration

### Activity 1: Test mandatory enforcement

1. Go to **Incident → Create New**.
2. Set **Impact** to **High** (Urgency is set to High and becomes read-only; Assignment group becomes mandatory).
3. Leave **Assigned to** empty and click **Submit**.
4. Result: the Incident is **not saved** and the error *"Assigned To is mandatory for High impact incidents."* appears.

### Activity 2: Test successful save

1. Fill in **Assigned to** and click **Submit**.
2. Result: the Incident saves successfully, with Urgency auto-set and read-only.

### Activity 3: Reverse condition test

1. Open an existing Incident with Impact = High.
2. Change **Impact** from **High** to **Medium**.
3. Result: **Assigned to** is no longer required and **Urgency** becomes editable (Reverse if false works).

### Activity 4: Test list edit blocking

1. Go to **Incident → All**.
2. Double-click the **State** field of any Incident to edit it in the list.
3. Result: an alert says the State cannot be changed from the list, and the value is unchanged.

### Activity 5: Test form-based update

1. Open the Incident form and change **State** to a new value.
2. Click **Update**.
3. Result: the State change saves successfully, and other fields still follow the UI Policy and Client Script rules.

---

## ✅ Test Summary

| Scenario                                | Expected Result                                         |
|-----------------------------------------|---------------------------------------------------------|
| Impact set to High                      | Urgency = High and read-only; Assignment group mandatory|
| Save High impact without Assigned to    | Save blocked with error message                         |
| Save High impact with Assigned to       | Saves successfully                                      |
| Impact changed from High to Medium      | Fields revert (Reverse if false)                        |
| Edit State from list view               | Alert shown, value unchanged                            |
| Edit State from Incident form           | Saves successfully                                      |

## 📂 Repository Contents

| File / Folder   | Description                              |
|-----------------|------------------------------------------|
| `README.md`     | Project documentation                    |
| `screenshots/`  | Screenshots of configuration and testing |

## 🎥 Project Demo

[▶️ Watch the Project Demo](https://1drv.ms/v/c/5edce3c1c386603c/IQCDEjD80L8fQLb6SIqsMTgGAQrh5whKxrlV4gFTS3oXfbw?e=Eg69sZ)

## 📸 Project Screenshots

### 1. UI Policies List

![UI Policies List](screenshots/01-ui-policies-list.jpg)

### 2. High Impact Control UI Policy

![High Impact Control UI Policy](screenshots/02-high-impact-control-policy.jpg)

### 3. UI Policy Action – Assignment Group

![UI Policy Action – Assignment Group](screenshots/03-policy-action-assignment-group.jpg)

### 4. UI Policy Action – Urgency

![UI Policy Action – Urgency](screenshots/04-policy-action-urgency.jpg)

### 5. Client Scripts List

![Client Scripts List](screenshots/05-client-scripts-list.jpg)

### 6. onChange Client Script

![onChange Client Script](screenshots/06-onchange-client-script.jpg)

### 7. onSubmit Client Script

![onSubmit Client Script](screenshots/07-onsubmit-client-script.jpg)

### 8. onCellEdit Client Script

![onCellEdit Client Script](screenshots/08-oncelledit-client-script.jpg)

### 9. Admin Home

![Admin Home](screenshots/09-admin-home.jpg)

### 10. Mandatory Enforcement Error

![Mandatory Enforcement Error](screenshots/10-test-mandatory-error.jpg)

### 11. Successful Save

![Successful Save](screenshots/11-test-successful-save.jpg)

### 12. Reverse Condition Test

![Reverse Condition Test](screenshots/12-test-reverse-condition.jpg)

### 13. Incident List

![Incident List](screenshots/13-incident-list.jpg)

### 14. List Edit Blocked

![List Edit Blocked](screenshots/14-list-edit-blocked.jpg)

### 15. State Update from Form

![State Update from Form](screenshots/15-form-state-update.jpg)

### 16. Incident List After Update

![Incident List After Update](screenshots/16-list-after-update.jpg)

## 🏁 Conclusion

This micro project shows how UI Policies and Client Scripts work together to enforce dynamic field behavior, automate updates, and prevent incorrect data submission on Incident forms. Using onChange, onSubmit, and onCellEdit scripts alongside UI Policy actions, fields like **Assigned to** and **Urgency** respond correctly to changes in **Impact**, keeping data clean and consistent. The solution is lightweight, efficient, and easy to implement.

## 👥 Team

| Name                          | Role      |
|-------------------------------|-------------|
| Swathi R                      | Team Leader |
| Sandhiya D                    | Member    |
| Sandhya Charlet Calista S     | Member    |
| Mallemkonda Dallyjasmine      | Member    |

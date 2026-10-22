# Implement-Client-Script-UI-Policy-Incident

Implementation of **Client Scripts** and **UI Policies** on the ServiceNow **Incident** table to automatically classify tickets and control form behavior.

---

## 📌 Project Overview

This project improves the Incident form in ServiceNow by:

- Automatically selecting the **Category** and **Subcategory** from the **Short Description** (Client Script)
- Validating the form before submission (Client Script)
- Controlling mandatory, read-only, and visible fields based on conditions (UI Policies)

The goal is to reduce manual effort and keep incident data consistent.

## 🚀 Features

- Real-time auto classification of incidents
- Category and subcategory mapping with dependent subcategories
- Form validation on submit
- Mandatory, read-only, and visibility control using UI Policies
- Helpful on-screen messages for the user

## 🛠️ Technologies Used

- ServiceNow
- Client Scripts (onLoad, onChange, onSubmit)
- UI Policies
- JavaScript
- ServiceNow Tables and Fields

---

## 🔄 Classification Logic

| Issue Type              | Keywords (examples)                | Category    | Subcategory     |
|-------------------------|------------------------------------|-------------|-----------------|
| WiFi                    | wifi, wi-fi, wireless, internet    | Network     | Wi-Fi           |
| Projector               | projector                          | Hardware    | Projector       |
| Password / Login        | password, login, locked out, reset | Access      | Forgot Password |
| Slow / Hanging Computer | slow, hang, freeze, not responding | Performance | Slow Computer   |

If no keyword matches, the user selects the category manually.

---

## 🖥️ Client Scripts

Create under **System Definition → Client Scripts**. Table: `Incident [incident]`.

> Make sure the choice **values** in the scripts match your Category and Subcategory choice list values.

### 1. Auto Classify Incident

| Field   | Value               |
|---------|---------------------|
| Name    | Auto Classify Incident |
| Type    | onChange            |
| Field   | Short description   |
| UI Type | All                 |

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        return;
    }

    var desc = newValue.toLowerCase().trim();

    var rules = [
        { re: /wi-?fi|wireless|internet/,         cat: 'network',     sub: 'wifi' },
        { re: /projector/,                        cat: 'hardware',    sub: 'projector' },
        { re: /password|login|locked out|reset/,  cat: 'access',      sub: 'forgot_password' },
        { re: /slow|hang|freez|not responding/,   cat: 'performance', sub: 'slow_computer' }
    ];

    for (var i = 0; i < rules.length; i++) {
        if (rules[i].re.test(desc)) {
            applyClassification(rules[i].cat, rules[i].sub);
            return;
        }
    }
}

function applyClassification(category, subcategory) {
    g_form.setValue('category', category);

    // Subcategory choices reload after category changes, so wait briefly
    setTimeout(function () {
        g_form.setValue('subcategory', subcategory);
    }, 500);

    g_form.addInfoMessage('Ticket auto-classified as ' + category + ' / ' + subcategory);
}
```

### 2. Validate Incident Submission

| Field | Value                        |
|-------|------------------------------|
| Name  | Validate Incident Submission |
| Type  | onSubmit                     |

```javascript
function onSubmit() {
    var desc = g_form.getValue('short_description');

    if (desc.length < 10) {
        g_form.showFieldMsg('short_description',
            'Please describe the issue in at least 10 characters.', 'error');
        return false;
    }

    if (!g_form.getValue('category')) {
        g_form.showFieldMsg('category',
            'Category is required. Select one or refine the short description.', 'error');
        return false;
    }

    return true;
}
```

### 3. Incident Form Setup

| Field | Value               |
|-------|---------------------|
| Name  | Incident Form Setup |
| Type  | onLoad              |

```javascript
function onLoad() {
    if (g_form.isNewRecord()) {
        g_form.addInfoMessage(
            'Describe your issue in the Short description. Category and subcategory are filled in automatically.');
        g_form.setMandatory('short_description', true);
    }
}
```

---

## 📋 UI Policies

Create under **System UI → UI Policies**. Table: `Incident [incident]`.

| # | Short description              | Condition                          | Actions (when true)                                        |
|---|--------------------------------|------------------------------------|------------------------------------------------------------|
| 1 | Subcategory required           | Category **is not empty**          | `Subcategory`: Mandatory = true                            |
| 2 | Resolution details required    | State **is** Resolved              | `Close code`, `Close notes`: Mandatory = true, Visible     |
| 3 | Lock classification on closure | State **is** Closed                | `Category`, `Subcategory`: Read only = true                |
| 4 | Critical incidents need group  | Priority **is** 1 - Critical       | `Assignment group`: Mandatory = true                       |
| 5 | Hide resolution until needed   | State **is not** Resolved / Closed | `Close code`, `Close notes`: Visible = false               |

For each policy, check **Reverse if false** so fields return to normal when the condition no longer applies, and keep **On load** checked.

### Optional: UI Policy Script (Policy 2)

Enable **Run scripts** on the policy.

**Execute if true**
```javascript
function onCondition() {
    g_form.addInfoMessage('Please provide a close code and close notes before resolving.');
}
```

**Execute if false**
```javascript
function onCondition() {
    g_form.clearMessages();
}
```

---

## 📂 Repository Contents

| File              | Description                                   |
|-------------------|-----------------------------------------------|
| `update_set.xml`  | Exported ServiceNow Update Set for deployment |
| `README.md`       | Project documentation                         |

## 📥 Installation

1. In ServiceNow, go to **System Update Sets → Retrieve Update Sets → Import Update Set from XML**.
2. Upload `update_set.xml`.
3. Open the imported update set and click **Preview Update Set**. Resolve any conflicts.
4. Click **Commit Update Set**.
5. Open **Incident → Create New** and enter a short description such as *"WiFi not working in conference room"*.

## ✅ Testing

| Short description                   | Expected Category | Expected Subcategory |
|-------------------------------------|-------------------|----------------------|
| Wifi not working in conference room | Network           | Wi-Fi                |
| Projector not displaying            | Hardware          | Projector            |
| I forgot my password                | Access            | Forgot Password      |
| My laptop is very slow              | Performance       | Slow Computer        |

Also verify:

- Saving with a short description under 10 characters shows an error.
- Setting State to **Resolved** makes Close code and Close notes mandatory.
- Setting State to **Closed** makes Category and Subcategory read-only.

## 🎯 Objective

Reduce manual effort in IT ticket classification and improve the consistency and speed of ticket handling.

## 🎥 Project Demo

[▶️ Watch the Project Demo](https://1drv.ms/v/c/5edce3c1c386603c/IQCDEjD80L8fQLb6SIqsMTgGAQrh5whKxrlV4gFTS3oXfbw?e=Eg69sZ)

The demo (about 2 minutes) shows the configuration and testing of the Client Scripts and UI Policies on the Incident form.

## 🔮 Future Improvements

- Predictive Intelligence for ML-based classification
- Automatic assignment group routing
- Priority detection from keywords
- Knowledge article suggestions for common issues

## 👥 Team

| Name                          | Role      |
|-------------------------------|-----------|
| Swathi R                      | Team Lead |
| Sandhiya D                    | Member    |
| Sandhya Charlet Calista S     | Member    |
| Mallemkonda Dallyjasmine      | Member    |

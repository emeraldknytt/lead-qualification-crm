# Lead Qualification and CRM Automation
An automated lead management system built with **n8n**, **Google Sheets**, and **Gmail**. It captures customer enquiries, evaluates leads based on their budgets, assigns priority levels, updates a CRM spreadsheet, sends automated email notifications, and maintains an execution log.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Project Objectives](#2-project-objectives)
3. [Technologies and Tools](#3-technologies-and-tools)
4. [How the Automation Works](#4-how-the-automation-works)
5. [Workflow Architecture](#5-workflow-architecture)
6. [Google Sheets Structure](#6-google-sheets-structure)
7. [How to Configure This Workflow With Your Own Information](#7-how-to-configure-this-workflow-with-your-own-information)
8. [Testing and Validation](#8-testing-and-validation)
9. [Security and Privacy Considerations](#9-security-and-privacy-considerations)
10. [Current Project Status](#10-current-project-status)
11. [Possible Future Improvements](#11-possible-future-improvements)
12. [Installation and Usage](#12-installation-and-usage)
13. [Project Summary](#13-project-summary)

---

## 1. Project Overview

Managing incoming customer enquiries manually can be time-consuming. Businesses need a reliable way to **collect enquiries**, **identify valuable prospects**, **respond promptly**, and **keep track of potential customers**.

This project addresses that problem by automating the lead qualification and management process.

The workflow receives customer information through an online enquiry form, evaluates the customer's budget, categorizes the lead, and stores the information in a Google Sheets CRM.

It also sends acknowledgement emails to customers, notifies the business owner about high-priority leads, calculates follow-up dates, prevents duplicate CRM records, and records successful processing activities.



> **The goal is to demonstrate how no-code automation can simplify lead management, reduce repetitive administrative work, and improve customer response times.**

---

## 2. Project Objectives

The project was designed to:

- [x] Automate customer enquiry collection.
- [x] Qualify incoming leads using predefined business rules.
- [x] Separate high-priority leads from standard-priority leads.
- [x] Store and organize customer information in a Google Sheets CRM.
- [x] Send automated acknowledgement emails.
- [x] Notify the business owner when a high-priority lead arrives.
- [x] Prevent duplicate CRM records using customer email addresses.
- [x] Calculate recommended follow-up dates automatically.
- [x] Validate incoming data before processing.
- [x] Provide error notifications when workflow executions fail.
- [x] Maintain an execution log for successful CRM updates.

---

## 3. Technologies and Tools

| Technology | Purpose |
|---|---|
| **n8n Cloud** | Workflow automation and orchestration |
| **n8n Form Trigger** | Collects customer enquiries |
| **Google Sheets** | Stores CRM records and execution logs |
| **Gmail** | Sends customer acknowledgements and internal notifications |
| **JavaScript expressions** | Handles timestamps, dates, and dynamic workflow values |

---

## 4. How the Automation Works

The automation follows a structured process from customer submission to CRM record creation or updating. The project is made up of **two workflows**:

| Workflow | Name | Role |
|---|---|---|
| **Workflow 01** | `01 - First Lead Qualification Workflow` | Main lead qualification and CRM workflow |
| **Workflow 02** | `02 - Lead CRM Error Handler` | Sends email alerts when the main workflow fails |

### Workflow 01 — Lead Qualification and CRM

**Workflow name:** `01 - First Lead Qualification Workflow`

The main workflow contains the following stages:

#### Stage 1: Customer Enquiry Form

An online form (built with the **n8n Form Trigger**) collects:

| Field | Description |
|---|---|
| **Full name** | The customer's full name |
| **Email address** | Used for contact and duplicate matching |
| **Company name** | The customer's organization |
| **Service needed** | The service the customer is enquiring about |
| **Budget** | Used to qualify the lead |
| **Project timeline** | The customer's expected timeframe |
| **Project description** | Additional details about the project |

#### Stage 2: Data Preparation

An **Edit Fields** node prepares the incoming information and generates a timestamp for the submission.

Example of an n8n expression for generating a submission timestamp:

```javascript
{{ $now.toISO() }}
```

#### Stage 3: Input Validation

An **IF** node checks whether the submitted budget is greater than or equal to zero.

```javascript
// Validation condition
budget >= 0
```

Invalid submissions are directed to a **Stop And Error** node rather than continuing through the qualification process.

#### Stage 4: Lead Qualification

The workflow evaluates the customer's budget against a predefined threshold of **1,000**.

```javascript
// Qualification condition
budget >= 1000
```

| Condition | Lead Classification |
|---|---|
| Budget **greater than or equal to 1,000** | **High Priority** |
| Budget **below 1,000** | **Standard Priority** |

#### Stage 5: Recommended Actions and Follow-up Dates

Each lead receives a recommended action and follow-up schedule.

| Attribute | High Priority | Standard Priority |
|---|---|---|
| **Lead priority** | High Priority | Standard Priority |
| **Recommended action** | Contact within 24 hours | Contact within 3 days |
| **Follow-up interval** | 1 day | 3 days |
| **Initial lead status** | New | New |

The workflow also calculates the **next follow-up date** automatically. Example n8n expressions for each branch:

```javascript
// High Priority branch — follow up in 1 day
{{ $now.plus({ days: 1 }).toFormat('yyyy-MM-dd') }}

// Standard Priority branch — follow up in 3 days
{{ $now.plus({ days: 3 }).toFormat('yyyy-MM-dd') }}
```

> **Note:** The expressions above are illustrative. Adjust them to match the exact field names and date format used in your own workflow.

#### Stage 6: Google Sheets CRM Integration

The workflow connects to a Google Sheets spreadsheet named **`Lead Qualification CRM`**.

The **`Leads`** worksheet stores:

- Customer information
- Qualification results
- Recommended actions
- Lead status
- Follow-up dates

The Google Sheets node uses the **Append or Update Row** operation, matching records using the customer's **email address**.

This means an existing lead can be **updated** instead of creating another CRM record for the same email address.

#### Stage 7: Automated Customer Acknowledgement

After the CRM operation, **Gmail** sends an acknowledgement email to the customer.

#### Stage 8: Internal High-Priority Notification

An additional **IF** node checks the lead classification.

| Lead Classification | Internal Notification |
|---|---|
| **High Priority** | ✅ Email sent to the business owner |
| **Standard Priority** | ❌ No notification sent |

#### Stage 9: Execution Logging

A separate Google Sheets worksheet named **`Execution Logs`** records successful CRM updates.

The log contains:

| Log Field | Description |
|---|---|
| **Execution ID** | The n8n execution identifier |
| **Execution URL** | Left blank in the current implementation |
| **Lead email** | The email address of the processed lead |
| **Lead priority** | High Priority or Standard Priority |
| **Processing stage** | The stage at which the log was recorded |
| **Log timestamp** | When the log entry was created |

This provides a **basic audit trail** of successful CRM processing activities.

---

### Workflow 02 — Error Handler

**Workflow name:** `02 - Lead CRM Error Handler`

A separate error-handling workflow uses n8n's **Error Trigger**.

When a supported automatic production execution fails, the error workflow can send an email notification containing relevant execution information, such as:

| Information | Purpose |
|---|---|
| **Workflow name** | Identifies which workflow failed |
| **Last executed node** | Shows where the failure occurred |
| **Error message** | Describes what went wrong |
| **Execution ID** | Identifies the specific failed run |
| **Execution URL** | Links directly to the failed execution |

This helps the workflow owner identify failures and investigate potential problems.

> **Note:** Error Trigger notifications are designed for **failed automatic production executions**. They do **not** automatically fire when manually testing a workflow in the editor.

---

## 5. Workflow Architecture

The overall automation can be represented as follows:

```text
CUSTOMER ENQUIRY
       |
       v
   FORM TRIGGER
       |
       v
   DATA PREPARATION
       |
       v
   INPUT VALIDATION
       |
       +------ Invalid ------> STOP AND ERROR
       |
       v
   BUDGET QUALIFICATION
       |
       +---------------------------+
       |                           |
       v                           v
  HIGH PRIORITY              STANDARD PRIORITY
       |                           |
       v                           v
  RECOMMENDATION             RECOMMENDATION
  FOLLOW-UP: 1 DAY           FOLLOW-UP: 3 DAYS
       |                           |
       +-------------+-------------+
                     |
                     v
             GOOGLE SHEETS CRM
              APPEND OR UPDATE
                     |
                     v
              EXECUTION LOG
                     |
                     v
             CUSTOMER EMAIL
                     |
                     v
             PRIORITY CHECK
                     |
              +------+------+
              |             |
              v             v
          HIGH LEAD      STANDARD LEAD
              |
              v
       INTERNAL ALERT
```

A separate **Error Handler** workflow listens for supported automatic execution failures and sends an email notification.

---

## 6. Google Sheets Structure

The project uses **one Google Sheets document** containing **two worksheets**.

### Worksheet 1: `Leads`

Create a worksheet named `Leads` with these column headers:

| Column | Header |
|---|---|
| A | `full_name` |
| B | `email` |
| C | `company` |
| D | `service_needed` |
| E | `budget` |
| F | `timeline` |
| G | `message` |
| H | `lead_priority` |
| I | `recommended_action` |
| J | `follow_up_days` |
| K | `created_at` |
| L | `lead_status` |
| M | `next_follow_up_date` |

> The workflow uses the **`email`** column as its **matching column** for duplicate prevention and record updates.

### Worksheet 2: `Execution Logs`

Create another worksheet named `Execution Logs` with these headers:

| Column | Header |
|---|---|
| A | `execution_id` |
| B | `execution_url` |
| C | `lead_email` |
| D | `lead_priority` |
| E | `processing_stage` |
| F | `logged_at` |

> The `execution_url` field is currently **left blank**.

---

## 7. How to Configure This Workflow With Your Own Information

This project can be adapted for **different businesses, freelancers, agencies, and service providers**.

After importing the workflow into your own n8n account, update the following settings.

### A. Connect Your Gmail Account

Locate the Gmail nodes in **both workflows** and update their Gmail credentials to use your own Google account.

Review the following nodes:

| Gmail Node | Workflow |
|---|---|
| **Customer acknowledgement email** | `01 - First Lead Qualification Workflow` |
| **Internal high-priority notification** | `01 - First Lead Qualification Workflow` |
| **Error notification** | `02 - Lead CRM Error Handler` |

> **Important:** Replace the internal notification recipient with **your own business email address**.

You should also review the **sender account** and **email content** before testing.

### B. Connect Your Google Sheets Account

Locate the Google Sheets nodes in the main workflow, then:

1. Create your own Google Sheets spreadsheet.
2. Name it something appropriate, such as `My Business CRM`.
3. Create the `Leads` worksheet.
4. Create the `Execution Logs` worksheet.
5. Add the required column headers listed in [Section 6](#6-google-sheets-structure).
6. Connect your own Google Sheets credentials in n8n.
7. Select your spreadsheet and the appropriate worksheet in each Google Sheets node.

> **Make sure the column names in your spreadsheet match the field mappings in the workflow.**

### C. Customize the Lead Qualification Threshold

Locate the lead qualification **IF** node. The current condition is:

```javascript
budget >= 1000
```

You can change this value to match your business requirements:

| Threshold | Suitable For |
|---|---|
| **500** | A smaller service business |
| **1,000** *(current)* | General-purpose default |
| **2,000** | A business targeting larger projects |
| **Custom logic** | A different field or combination of conditions for more advanced qualification |

For example, a more advanced condition might combine budget and timeline:

```javascript
// Illustrative example of a combined condition
budget >= 2000 && timeline === "Immediately"
```

> **Remember** to update the recommended actions and email content if you change the qualification rules.

### D. Customize Follow-up Intervals

Locate the **two branch Edit Fields nodes**. The current configuration is:

| Priority | Follow-up Interval |
|---|---|
| **High Priority** | 1 day |
| **Standard Priority** | 3 days |

You can change these intervals to match your preferred customer follow-up schedule.

> The `next_follow_up_date` expressions **should also be adjusted** if you change the intervals.

For example, to change the Standard Priority follow-up to 5 days:

```javascript
{{ $now.plus({ days: 5 }).toFormat('yyyy-MM-dd') }}
```

### E. Customize the Enquiry Form

Open the **Form Trigger** node. You can modify:

- Form title
- Form description
- Form path
- Input field names
- Required fields
- Input types

For example, a **digital marketing agency** might collect:

- Company website
- Monthly marketing budget
- Advertising platform
- Current marketing challenges

> If you change form fields, make sure the **first Edit Fields node** maps the incoming values to the field names expected by the rest of the workflow.

### F. Customize Customer Email Templates

Open the **customer acknowledgement Gmail node** and replace the existing message with your own business introduction and acknowledgement.

You may customize:

- Business name
- Greeting
- Service description
- Response expectations
- Contact information
- Email signature

Also review the **internal notification email** so that it contains the information your team needs to respond to high-priority leads.

### G. Configure Error Notifications

Open `02 - Lead CRM Error Handler`, then:

1. Connect your Gmail account.
2. Replace the notification recipient with your own email address.
3. Review the error notification subject and message to ensure they suit your business.

> **The error workflow must be available and published** for the main workflow to use it in the current n8n interface.

---

## 8. Testing and Validation

The workflow was tested using both **high-priority** and **standard-priority** lead submissions.

The following scenarios were verified:

| # | Test Scenario | Expected Result | Result |
|---|---|---|---|
| 1 | Lead submits a budget of **2,500** | High Priority classification | ✅ Passed |
| 2 | Lead submits a budget of **500** | Standard Priority classification | ✅ Passed |
| 3 | High-priority lead submission | Internal notification sent | ✅ Passed |
| 4 | Standard-priority lead submission | No internal priority alert | ✅ Passed |
| 5 | Customer submits an enquiry | Acknowledgement email sent | ✅ Passed |
| 6 | New lead submission | CRM record created | ✅ Passed |
| 7 | Existing email submits again | Existing CRM record updated | ✅ Passed |
| 8 | Follow-up tracking | Correct follow-up interval and date | ✅ Passed |
| 9 | Valid budget submission | Processing continues | ✅ Passed |
| 10 | Negative budget validation | Rejected by validation logic | ⚙️ Configured |
| 11 | Successful CRM update | Execution log created | ✅ Passed |

*Tests were conducted during development using n8n's test form and the connected Google services.*

---

## 9. Security and Privacy Considerations

Before sharing this project publicly on GitHub, **remove or replace all personal and sensitive information**.

### ⚠️ Never Publish

| Sensitive Item | Risk |
|---|---|
| **Gmail addresses** used for private notifications | Exposes private contact details |
| **OAuth access tokens or refresh tokens** | Grants access to connected accounts |
| **Google API credentials** | Allows unauthorized API usage |
| **Client information or real customer records** | Breaches customer privacy |
| **Private spreadsheet links or identifiers** | May expose confidential resources |
| **Authentication details or passwords** | Compromises account security |

### Important n8n Credential Information

- n8n credentials **should be configured separately** by anyone importing the workflow.
- **Do not assume** that exporting a workflow JSON file transfers working Gmail or Google Sheets authentication.
- After importing the workflow, **each user should connect their own accounts** and select their own credentials.

> For additional protection, use **environment variables** or other appropriate secret-management methods where supported, rather than hardcoding sensitive values.

---

## 10. Current Project Status

The project has **completed its initial development and testing stages**.

### Implemented Features

| Feature | Status |
|---|---|
| Form-based lead collection | ✅ Implemented |
| Budget-based lead qualification | ✅ Implemented |
| High and standard priority classification | ✅ Implemented |
| Google Sheets CRM integration | ✅ Implemented |
| Automated customer acknowledgement emails | ✅ Implemented |
| Internal high-priority notifications | ✅ Implemented |
| Duplicate lead prevention | ✅ Implemented |
| Automated follow-up date calculation | ✅ Implemented |
| Input validation | ✅ Implemented |
| Error notification workflow | ✅ Implemented |
| Successful CRM execution logging | ✅ Implemented |

### Deployment Status

| Workflow | Status | Notes |
|---|---|---|
| `01 - First Lead Qualification Workflow` | **Unpublished** | Tested using the form's test functionality; not intentionally deployed as a live production intake workflow |
| `02 - Lead CRM Error Handler` | **Published** | Published so that it can be selected in the main workflow's error-handling settings |

---

## 11. Possible Future Improvements

The current version provides a useful foundation. Future versions could include:

- [ ] A scheduled workflow that identifies overdue follow-ups.
- [ ] Automatic follow-up reminder emails.
- [ ] A dashboard showing lead counts and conversion rates.
- [ ] More advanced lead scoring based on budget, timeline, and service requirements.
- [ ] Separate handling of returning customers.
- [ ] More detailed execution and failure logging.
- [ ] Slack or Telegram notifications.
- [ ] Integration with a dedicated CRM platform.
- [ ] Automated reporting for business owners.

> **Note:** These are **potential extensions**, not features currently implemented.

---

## 12. Installation and Usage

To use this project:

1. Create an **n8n Cloud** account or use a compatible **self-hosted n8n** instance.
2. Import the **main workflow** JSON file.
3. Import the **error-handler workflow** JSON file.
4. Connect your own **Gmail** credentials.
5. Connect your own **Google Sheets** credentials.
6. Create the required **spreadsheet and worksheets**.
7. Review and customize the **form fields and business rules**.
8. Update **email recipients and message templates**.
9. Test both **high-priority and standard-priority** submissions.
10. Verify **CRM updates, duplicate handling, and execution logs**.
11. Review **error handling and security settings** before deployment.
12. **Publish the main workflow** only when you are ready to accept live enquiries.

> ⚠️ *Always test the workflow with non-sensitive sample information before using it with real customers.*

---

## 13. Project Summary

This project demonstrates how **n8n can be used to build practical business automations that connect forms, email services, and spreadsheets into a single workflow.**

By automating repetitive lead-management tasks, businesses can spend **less time organizing enquiries** and **more time engaging potential customers**.

It also demonstrates practical skills in:

| Skill Area | Demonstrated Through |
|---|---|
| **Workflow design** | Two coordinated n8n workflows |
| **Conditional logic** | Validation, qualification, and priority-check IF nodes |
| **Data mapping** | Edit Fields nodes and Google Sheets column mappings |
| **API-connected services** | Gmail and Google Sheets integrations |
| **Validation** | Budget checks with Stop And Error handling |
| **Error handling** | Dedicated Error Trigger workflow |
| **Automation testing** | 11 verified test scenarios |

---

**Built with:** n8n Cloud · Google Sheets · Gmail

**Project Category:** Business Process Automation / Lead Management / CRM Automation

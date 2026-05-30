# 🗂️ Leave Management System
### Built on ServiceNow · App Engine Studio · Low-Code / No-Code

![Platform](https://img.shields.io/badge/Platform-ServiceNow-009BDE?style=flat-square)
![Studio](https://img.shields.io/badge/Studio-App%20Engine%20Studio-brightgreen?style=flat-square)
![UI](https://img.shields.io/badge/UI-Record%20Producer-orange?style=flat-square)
![Type](https://img.shields.io/badge/Type-Low--Code%20%2F%20No--Code-blueviolet?style=flat-square)

---

## 📌 Project Overview

The **Leave Management System (LMS)** is a ServiceNow application developed using **App Engine Studio (AES)** following a low-code / no-code approach. It provides a self-service platform for employees to apply for leaves, enables managers to approve or reject requests, and gives HR administrators complete visibility over leave records across the organization.

**Key Capabilities:**
- Employee self-service leave submission via a **Record Producer** form
- Automated manager approval workflows via **Flow Designer**
- Real-time leave status tracking from the ServiceNow portal
- Role-based access for Employees, Managers, and HR Administrators
- Email and in-app notifications at every workflow stage
- Reporting dashboards for HR analytics and leave trend monitoring

---

## 🛠️ Technology Stack

| Component | Technology / Module | Purpose |
|---|---|---|
| Development Studio | App Engine Studio (AES) | Low-code / no-code application builder within ServiceNow |
| Database Layer | ServiceNow Custom Table | Stores all leave request records with defined schema |
| UI Layer | Record Producer | Form-based UI for submitting leave requests via ESS Portal |
| Workflow Engine | Flow Designer | Automates approval routing, notifications, and status transitions |
| Notifications | ServiceNow Email / In-App | Alerts on submission, approval, rejection, and cancellation |
| Reporting | ServiceNow Reports & Dashboards | Visual leave analytics for HR and management |
| Access Control | ACLs + Roles | Restricts data access based on user role |

---

## 🗃️ Database Table Structure

| Property | Value |
|---|---|
| **Table Name** | `x_leave_management_systemskav_lms_leave_request` *(scoped app prefix)* |
| **Table Label** | Leave Management System |
| **Extends** | Task *(ServiceNow base Task table)* |
| **Application Scope** | Scoped — isolated namespace via App Engine Studio |
| **Primary UI** | Record Producer (catalog-based form) |
| **Auto-Number** | Yes — e.g., `LMS0001001` |

---

## 📋 Field Reference

| # | Field Name | Field Type | Mandatory | Description |
|---|---|---|---|---|
| 1 | **Start Date** | Date | ✅ Yes | The date from which the leave begins. Must not be in the past for new requests. |
| 2 | **End Date** | Date | ✅ Yes | The date on which the leave ends. Must be on or after the Start Date. |
| 3 | **Employee** | Reference (`sys_user`) | ✅ Yes | Auto-populated from the logged-in user session. References the HR Employee table. |
| 4 | **Leave Type** | Choice / Reference | ✅ Yes | Type of leave: Annual, Sick, Casual, Maternity, Paternity, Loss of Pay (LOP). |
| 5 | **Reason** | String (Multi-line) | ❌ No | Free-text field for the employee to describe the reason for the leave request. |

---

## 🌿 Leave Type Reference

| Leave Type | Max Days / Year | Requires Documentation | Notes |
|---|---|---|---|
| Annual Leave | 15 days | No | Standard paid leave. Must be applied at least 3 days in advance. |
| Sick Leave | 12 days | Yes *(Medical cert)* | Can be applied retroactively with a valid medical certificate. |
| Casual Leave | 6 days | No | For short unplanned absences. Max 2 consecutive days. |
| Maternity Leave | 26 weeks | Yes *(Documents)* | As per Maternity Benefit Act. Applicable to female employees. |
| Paternity Leave | 7 days | No | For male employees upon birth of child. |
| Loss of Pay (LOP) | Unlimited | No | Leave without pay. Used when all paid leaves are exhausted. |

---

## 🖥️ Record Producer — Form Walkthrough

The **Record Producer** is the primary UI for leave submission, accessible from the Employee Self-Service (ESS) Portal.

**Access Path:**
```
ESS Portal → Service Catalog → HR Services → Leave Management System
```

**Form Behaviour:**

| Behaviour | Detail |
|---|---|
| Employee Field | Auto-populated using logged-in user session (`g_user.userID`) — not editable |
| Date Validation | Client-side script enforces `End Date >= Start Date` |
| Leave Type | Dropdown choice; selecting *Sick Leave* displays a documentation advisory note |
| Reason | Optional free-text; recommended for approval context |
| On Submit | Record created with **Status = Pending Approval**; approval task routed to reporting manager |

---

## 👤 User Interaction Guide

| Step | Role | Action | Navigation Path | Expected Outcome |
|---|---|---|---|---|
| 1 | Employee | Access Portal | ServiceNow Portal → Employee Self-Service → Leave Management | Landing page of LMS is displayed |
| 2 | Employee | Submit Leave | Click *Request Leave* → Fill Record Producer Form → Submit | Leave record created with status **Pending Approval** |
| 3 | Employee | View Status | My Requests → Leave Management → Open Record | Current status visible: Pending / Approved / Rejected |
| 4 | Manager | Approve / Reject | Approval Inbox → Select Request → Approve or Reject | Status updates; email notification sent to employee |
| 5 | Employee | Cancel Request | My Requests → Open Record → Click *Cancel* *(if Pending)* | Leave record status changes to **Cancelled** |
| 6 | HR Admin | View Reports | Reports → Leave Summary → Filter by Date / Employee / Type | Aggregated leave data in tabular and chart format |

---

## 🔄 Application Workflow

```
Employee Submits Request
        ↓
Approval Task Created → Routed to Reporting Manager
        ↓
    ┌───┴───┐
Approved   Rejected
    ↓           ↓
Balance     Status = Rejected
Deducted    (Manager adds reason)
    ↓
Employee Notified via Email + Portal
```

**Additional Rules:**

| Scenario | Behaviour |
|---|---|
| Approved | Leave balance deducted; record status set to **Approved** |
| Rejected | Record status set to **Rejected**; manager can add rejection reason |
| Escalation | If manager does not act within SLA (48 hrs), request escalates to HR |
| Cancellation | Employee can cancel a **Pending** request before approval decision |

---

## 🔐 Roles & Permissions

| Role | Permissions |
|---|---|
| **Employee** | Submit leave request, view own leave history, cancel pending requests |
| **Manager / Approver** | View team leave requests, approve or reject, view team calendar |
| **HR Administrator** | Full access to all leave records, configure leave types, generate reports |
| **System Administrator** | Full platform access including table config, flows, and ACL management |

---

## 🚀 Deployment Notes

### Prerequisites

| Requirement | Detail |
|---|---|
| ServiceNow Instance | Tokyo / Utah / Vancouver or later |
| App Engine Studio Plugin | Must be activated on the instance |
| HR Service Delivery (HRSD) | Required for Employee reference field linkage |

### Installation Steps

```
1. Open App Engine Studio from the main ServiceNow navigator
2. Import the application XML or clone from the source repository
3. Publish the Record Producer to the desired ESS Portal catalog
4. Assign roles (employee, manager, hr_admin) to the appropriate user groups
5. Configure email notification templates:
   System Notification → Email → Notifications
6. Test end-to-end using impersonation of Employee and Manager users
```

### Scoping Note

This application is **fully scoped** and will not conflict with other ServiceNow applications. All custom tables, scripts, business rules, and flows are prefixed with the application scope identifier (`x_leave_management_systemskav_lms_`).

---

## 📁 Project Structure

```
Leave Management System (Scoped App)
├── Tables
│   └── x_leave_management_systemskav_lms_leave_request       # Core leave request table
├── Catalog Items
│   └── Record Producer (Leave Form)   # Employee-facing submission form
├── Flows
│   └── Leave Approval Flow            # Approval routing & notifications
├── Notifications
│   ├── Leave Submitted                # Confirmation to employee
│   ├── Approval Request               # Alert to manager
│   └── Leave Decision                 # Outcome notification to employee
├── Reports
│   └── Leave Summary Dashboard        # HR analytics view
└── Roles
    ├── employee
    ├── manager
    └── hr_admin
```

---

## 👨‍💻 Author

**Sandeep Kasturi** 
*Built with ServiceNow App Engine Studio — Low-Code / No-Code*

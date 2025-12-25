# Microsoft Entra PIM: Just-in-Time (JIT) Admin Access with Privileged Identity Management

## 📌 Overview

In this lab I built and validated a **PIM-based Just-in-Time (JIT) elevation flow** in Microsoft Entra ID.

I used **Privileged Identity Management (PIM)** to:

* Assign a user **Eligible** access to an Entra admin role (not permanently active)
* Enforce activation governance controls (MFA + justification + time-bound duration)
* Activate the role on demand, perform a safe admin action, and preserve audit evidence
* Export the audit trail as a CSV for a clean GitHub evidence bundle

The point of this lab is not to become a permanent admin. The point is to prove that I can implement **governed elevation** where admin rights are only present **when needed**, and the audit trail proves it.

---

## 🎯 Objectives

By the end of this lab I have:

* Selected a single Entra admin role to demonstrate JIT elevation (**User Administrator**)
* Assigned an operator account as **Eligible** for the role using PIM
* Configured activation requirements:

  * Require **MFA**
  * Require **Justification**
  * Activation duration set to **1 hour**
* Activated the role and performed one safe admin action aligned to the role
* Collected evidence:

  * Role assignment event
  * Role activation event
  * Exported audit logs as CSV

---

## 🧠 Key Concepts

* **Privileged Identity Management (PIM):** Governance feature for controlling privileged roles with eligibility, approvals, and auditing
* **Eligible Assignment:** User can activate a role when needed, but does not hold active privileges by default
* **Just-in-Time (JIT) Access:** Temporary elevation of privileges for a bounded window
* **Activation Controls:** Requirements enforced when activating a role (MFA, justification, ticket, approval)
* **RoleManagement Audit Logs:** Audit trail category that records role assignments, activations, and removals
* **Time-bound Activation:** Privileges automatically end after the configured duration

---

## 🧠 Mental Model

I treat this lab as a controlled elevation loop:

1. **No standing admin**

   * The operator account is **Eligible**, not Active.

2. **Governed activation**

   * When admin access is needed, the operator requests activation and must satisfy controls like **MFA** and **justification**.

3. **Temporary privilege**

   * The role becomes Active only for the configured duration (1 hour), then ends automatically.

4. **Evidence-backed governance**

   * The audit trail proves the assignment, activation, and eventual role removal/expiration.

The “success condition” for this lab is concrete:

* I can show the PIM settings, the eligible assignment, the activation, the admin action, and the audit log evidence including a CSV export.

---

## 🗂 Repository Structure

```
Entra-PIM-JIT-Lab/
├── evidence/
│   ├── 01-pim-eligible-assignment.png
│   ├── 02-pim-activation-settings.png
│   ├── 03-pim-activation-request.png
│   ├── 04-admin-action-performed.png
│   ├── 05-audit-logs-activation-event.png
│   ├── 06-audit-logs-assignment-event.png
│   └── 07-audit-log-export.png                # optional (UI proof of export)
├── exports/
│   └── 08-audit-log-export.csv                # required
└── README.md
```

---

## 🔒 Safety & Scope Guardrails

I used a dev/test tenant only. I avoided high-risk roles and destructive changes.

Safe boundaries I followed:

* Only one role: **User Administrator**
* One safe admin action: **Disable sign-in for a test user**
* No changes to production identities or critical policies
* No overclaiming: I only state what my screenshots and logs prove

---

## 📋 Step 1 – Choose Role and Accounts

### Role selected

* **User Administrator** (best lab choice because it enables a clear, low-risk admin action: user account management)

### Accounts

* **Admin/Operator:** Michael A. Cooper (created eligible assignment)
* **Operator (activation user):** Chris Green (activated role and performed admin action)

---

## 📋 Step 2 – Configure PIM Activation Settings (Role Settings)

I went to the Entra admin center and edited activation controls for the User Administrator role.

**Portal navigation**

* Microsoft Entra admin center → **Identity Governance** → **Privileged Identity Management**
* **Microsoft Entra roles** → **Roles**
* Select **User Administrator**
* **Settings / Role settings**
* Edit **Activation** settings

**Activation configuration**

* Activation maximum duration: **1 hour**
* Require on activation:

  * ✅ **Azure MFA**
  * ✅ **Require justification on activation**
* Approval was not required (kept simplest governance configuration)

**Evidence**

* Screenshot captured: `02-pim-activation-settings.png`

---

## 📋 Step 3 – Assign Operator as Eligible (PIM Assignment)

I assigned the operator account as **Eligible** for User Administrator.

**Portal navigation**

* Identity Governance → Privileged Identity Management → **Microsoft Entra roles**
* **Roles** → Select **User Administrator**
* **Assignments** → **Add assignments**
* Assignment type: **Eligible**
* Member: select operator user (Chris Green)
* Scope: **Directory**
* Reason: lab / JIT test

**Evidence**

* Screenshot captured: `01-pim-eligible-assignment.png`
* Audit evidence captured later: `06-audit-logs-assignment-event.png`

---

## 📋 Step 4 – Activate the Role (JIT Elevation)

I logged in as the operator account and activated the eligible role.

**Portal navigation**

* Identity Governance → Privileged Identity Management
* **My roles** → **Microsoft Entra roles**
* Find **User Administrator**
* Click **Activate**
* Enter **Justification**
* Complete **MFA** prompt if required
* Submit activation

**Evidence**

* Screenshot captured: `03-pim-activation-request.png`
* Audit evidence captured later: `05-audit-logs-activation-event.png`

---

## 📋 Step 5 – Perform One Safe Admin Action (Aligned to Role)

With the role active, I performed a safe, reversible admin action aligned to the User Administrator role.

**Action performed**

* Disabled sign-in for a dedicated test user: **PIM JIT Test User**

**Portal navigation**

* Entra admin center → **Entra ID** → **Users**
* Open **PIM JIT Test User**
* **Edit properties** (or **Settings** section depending on blade)
* Set **Account enabled = No**
* Save

**Evidence**

* Screenshot captured: `04-admin-action-performed.png`

---

## 📋 Step 6 – Capture Audit Events (Assignment + Activation)

I validated the audit trail directly in the Entra portal.

### 6.1 Activation event (PIM activation)

**Portal navigation**

* Entra admin center → **Audit logs**
* Filter:

  * Date range: last 24 hours
  * Service: **PIM**
  * Category: **RoleManagement**
* Select the activation record
* Open **Audit details**

**Evidence**

* Screenshot captured: `05-audit-logs-activation-event.png`

### 6.2 Eligible assignment event

I located the eligible assignment event where the admin added the user as eligible.

**Evidence**

* Screenshot captured: `06-audit-logs-assignment-event.png`

---

## 📋 Step 7 – End of Elevation (Optional)

I did not capture a separate deactivation screenshot. This lab relies on time-bound activation and audit evidence.

---

## 📋 Step 8 – Export Audit Logs as CSV (Required)

I exported the filtered audit logs so the evidence bundle is reproducible and portable.

**Portal navigation**

* Entra admin center → **Audit logs**
* Confirm filters:

  * Date range: **Last 24 hours**
  * Service: **PIM**
  * Category: **RoleManagement**
* Click **Download**
* Select format: **CSV**
* Download and save as: `08-audit-log-export.csv`

**Evidence**

* CSV saved: `08-audit-log-export.csv`
* Optional screenshot of export pane or CSV opened in Calc: `07-audit-log-export.png`

---

## ✅ Validation (Evidence-Only)

I validate the lab using only captured artifacts:

1. **Eligibility is proven**

   * `01-pim-eligible-assignment.png` shows the operator assigned as **Eligible**
   * `06-audit-logs-assignment-event.png` confirms the eligible assignment was recorded in audit logs

2. **Governed activation controls are proven**

   * `02-pim-activation-settings.png` shows MFA required, justification required, and duration set to 1 hour

3. **JIT activation is proven**

   * `03-pim-activation-request.png` shows the role active for the operator
   * `05-audit-logs-activation-event.png` confirms activation was requested and succeeded

4. **Admin action is proven**

   * `04-admin-action-performed.png` shows the test user account disabled (Account enabled: No)

5. **Exportable audit evidence exists**

   * `08-audit-log-export.csv` provides the durable evidence bundle for review and re-checking

---

## ✅ Evidence Bundle Checklist (README-ready)

* [ ] `01-pim-eligible-assignment.png`
* [ ] `02-pim-activation-settings.png`
* [ ] `03-pim-activation-request.png`
* [ ] `04-admin-action-performed.png`
* [ ] `05-audit-logs-activation-event.png`
* [ ] `06-audit-logs-assignment-event.png`
* [ ] `08-audit-log-export.csv`
* [ ] `07-audit-log-export.png` (optional)

---

## 📷 Screenshots

| # | Filename                             | Description                                                                                            |
| - | ------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| 1 | `01-pim-eligible-assignment.png`     | PIM role assignments page showing operator assigned as Eligible for User Administrator.                |
| 2 | `02-pim-activation-settings.png`     | Role setting edit screen showing MFA required, justification required, and 1 hour activation duration. |
| 3 | `03-pim-activation-request.png`      | Operator view showing User Administrator as activated with an end time.                                |
| 4 | `04-admin-action-performed.png`      | User details showing Account enabled: No for the test user.                                            |
| 5 | `05-audit-logs-activation-event.png` | Audit details showing PIM activation event succeeded for User Administrator.                           |
| 6 | `06-audit-logs-assignment-event.png` | Audit details showing eligible assignment created for the role.                                        |
| 7 | `07-audit-log-export.png`            | Optional screenshot showing export pane or CSV opened in Calc.                                         |

---

## 💡 What I Learned

From this lab I can now say:

* I can implement JIT admin access in Entra using PIM rather than relying on standing privileged roles.
* I understand the difference between Eligible (can activate) and Active (currently elevated).
* I can enforce activation governance controls using MFA, justification, and time-bound activation.
* I can validate the whole flow using audit evidence and preserve it as a CSV export.

---

## 💼 Business Relevance

This lab maps directly to real-world identity governance:

* Reduces standing admin privileges (least privilege)
* Limits blast radius by ensuring privileged rights are temporary
* Improves accountability via justification and audit trail
* Produces defensible evidence for audits and access reviews

---

## 🎤 Interview Talking Points

* “I configured Microsoft Entra PIM to assign an operator as Eligible for a privileged role, then required MFA and justification for activation.”
* “The operator activated the role for a one-hour window, performed a safe admin action, and the audit logs prove both the eligible assignment and the activation.”
* “I exported the audit logs as a CSV evidence bundle so the elevation is reviewable and reproducible.”

---

## 🙏 Acknowledgments

This lab was completed using AI assistance as a learning accelerator.

AI helped with:

* Runbook structure and sequencing
* Evidence bundle structure and artifact naming discipline

I own:

* All portal execution and configuration decisions
* All validation choices and evidence backing
* Ability to explain the JIT elevation model, why it matters, and what the audit logs prove


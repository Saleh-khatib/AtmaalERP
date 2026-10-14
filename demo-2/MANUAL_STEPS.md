# Manual Steps for Managed Services Client Portal

The following items cannot be expressed in DS files and must be configured manually in the Zoho Creator UI.

---

## 1. Connection Setup

**What:** Create the four Zoho service connections.

**Where:** Creator UI → Settings → Developer Space → Connections

**Steps:**
1. Create connection `crm_conn` for Zoho CRM
2. Create connection `desk_conn` for Zoho Desk
3. Create connection `workdrive_conn` for Zoho WorkDrive
4. Create connection `sign_conn` for Zoho Sign

**Note:** The app code references these connections by name (`crm_conn`, `desk_conn`, `workdrive_conn`, `sign_conn`). The connections themselves must be created by a human with appropriate OAuth credentials.

---

## 2. MFA Requirement

**What:** Enable Multi-Factor Authentication for portal users.

**Where:** Creator UI → Settings → Security → Multi-Factor Authentication

**Steps:**
1. Enable MFA for the application
2. Configure MFA requirements for portal users

**Note:** This security setting cannot be expressed in DS files.

---

## 3. CRM Webhook Configuration

**What:** Set up the CRM webhook endpoint and trigger.

**Where:** 
- Creator UI → Settings → Developer Space → Webhooks (to get the endpoint URL)
- Zoho CRM → Settings → Workflow Rules → Trigger the webhook

**Steps:**
1. Create a webhook in Creator for the reconciliation/receive endpoint
2. Copy the webhook URL and authentication token
3. In Zoho CRM, create a workflow rule that calls this webhook on deal stage changes
4. Configure the workflow to pass the shared secret in a header

**Note:** The DS code is ready to receive webhook calls, but the CRM-side trigger must be configured manually.

---

## 4. Zoho Desk Webhook

**What:** Configure Desk webhook to trigger Ticket_Updated notifications.

**Where:** Zoho Desk → Admin → Channels → Webhooks

**Steps:**
1. Create a webhook in Zoho Desk for ticket updates/replies
2. Point it to the Creator webhook endpoint
3. Configure the webhook to send the necessary ticket data

**Note:** The DS function to handle Ticket_Updated is ready, but the Desk-side webhook must be set up manually.

---

## 5. Seed Data Population

**What:** Populate Config_Stage_Map and Config_Strings with initial data.

**Where:** Creator UI → Data → Import or manual entry

**Steps:**

### Config_Stage_Map (17 stages from PROJECT_SPEC.md section 6):
Insert records for each stage:
| CRM_Stage_Value | Client_Label_EN | Client_Label_AR | Dashboard_Group | Who_Acts | Sort_Order |
|-----------------|-----------------|-----------------|-----------------|----------|------------|
| Backlog | Request submitted | تم تقديم الطلب | New | Provider | 1 |
| Process Assessment | Discovery scheduled | تم تحديد جلسة الاستكشاف | Needs your action | Client | 2 |
| Requirements Confirmation | Requirements confirmed | تم تأكيد المتطلبات | Needs your action | Client | 3 |
| Capacity Assessment | Preparing your solution | جارٍ إعداد الحل الخاص بك | In progress | Provider | 4 |
| Proposal Review | Proposal ready for your approval | العرض جاهز لاعتمادك | Needs your action | Client | 5 |
| Client Approved | With your procurement team | لدى فريق المشتريات لديكم | Needs your action | Client | 6 |
| SOW | Ready for signature | جاهز للتوقيع | Needs your action | Client | 7 |
| Client Signed | Provider counter-signing | التوقيع من جانب مزود الخدمة | In progress | Provider | 8 |
| Accepted | Awaiting your PO | بانتظار أمر الشراء الخاص بكم | Needs your action | Client | 9 |
| PO Confirmed | Preparing your handover | جارٍ إعداد التسليم | In progress | Provider | 10 |
| Handover | Confirm project contacts | تأكيد جهات الاتصال للمشروع | Needs your action | Client | 11 |
| Mobilization | Mobilising your team | جارٍ تجهيز الفريق | In progress | Provider | 12 |
| Implementation | Building your service | جارٍ بناء خدمتك | In progress | Provider | 13 |
| UAT | Ready for your testing | جاهز لاختباركم | Needs your action | Client | 14 |
| Go-Live Readiness | Approve go-live | اعتماد الإطلاق | Needs your action | Client | 15 |
| Hypercare | Hypercare — live with monitoring | الدعم المكثف بعد الإطلاق | In progress | Provider | 16 |
| Live | Live operations | التشغيل الفعلي | Completed | Provider | 17 |

### Config_Strings (UI copy from PROJECT_SPEC.md section 11):
Insert records for each key:
| String_Key | Text_EN | Text_AR |
|------------|---------|---------|
| nav.dashboard | Dashboard | لوحة المعلومات |
| nav.requests | My Requests | طلباتي |
| nav.live | Live Services | الخدمات المباشرة |
| nav.tickets | Service Tickets | تذاكر الخدمة |
| nav.docs | Reports & Documents | التقارير والمستندات |
| btn.new_request | New service request | طلب خدمة جديد |
| badge.coming_soon | Coming soon | قريباً |
| dash.attention | Needs your attention | يحتاج إلى إجراء منك |
| dash.attention_empty | Nothing needs you right now. | لا يوجد ما يحتاج إلى إجراء منك حالياً. |
| detail.provider_working | Nothing for you to do right now. We'll notify you the moment we need you. | لا يوجد ما تحتاج إلى فعله الآن. سنخبرك فور احتياجنا إليك. |
| err.load | We couldn't load this. Please try again. | تعذّر تحميل هذه الصفحة. يرجى المحاولة مرة أخرى. |
| ok.action_done | Done. Next up: {next} | تم. الخطوة التالية: {next} |
| notif.empty | You're all caught up. | لا توجد إشعارات جديدة. |
| tab.all | All | الكل |
| tab.action | Needs your action | يحتاج إلى إجراء |
| tab.progress | In progress | قيد التنفيذ |
| tab.completed | Completed | مكتمل |

**Note:** These can be imported via CSV or entered manually.

---

## 6. Profile Permissions Verification

**What:** Verify that portal users cannot access restricted forms.

**Where:** Creator UI → Settings → Users & Roles → Profiles

**Steps:**
1. After all forms are created, open each profile (Requester, Approver, Viewer)
2. Verify that Audit_Log, Error_Log, Config_Stage_Map, and Config_Strings have NO permissions granted
3. Ensure these forms are not visible in the navigation for portal users

**Note:** While the DS files will define these restrictions in the Permissions segment, manual verification is recommended.

---

## 7. Role-Based Field Visibility

**What:** Enforce read-only status for Login_Email, Account, and Role fields for portal users.

**Where:** Creator UI → Settings → Users & Roles → Profiles → Field Permissions

**Steps:**
1. Open each portal user profile (Requester, Approver, Viewer)
2. Navigate to Portal_User form field permissions
3. Set Login_Email, Account, and Role fields to `visibility: true, readonly: true`

**Note:** This ensures portal users cannot modify their own login email, account assignment, or role.

---

## 8. Report Device Layouts

**What:** Define quickview and detailview layouts for each report on web (and optionally tablet/phone).

**Where:** Creator UI → Forms → [Form] → Reports → [Report] → Device Layouts

**Steps:**
1. For each report (Account_List, Portal_User_List, Request_Index_List, Notification_List, Audit_Log_List, Config_Stage_Map_List, Config_Strings_List, Error_Log_List):
   - Configure the web quickview layout
   - Configure the web detailview layout
   - Optionally configure tablet and phone layouts if the report will be accessed on those devices

**Note:** This can also be done via DS files in a later phase (devices/ folder), but must be configured at some point.

---

## 9. Menu Configuration

**What:** Set up the left navigation structure.

**Where:** Creator UI → Settings → Mobile → Menu (or Devices → Web → Menu)

**Steps:**
1. Create navigation sections in this order:
   - Dashboard
   - My Requests
   - Live Services
   - Service Tickets
   - Reports & Documents
2. Add the corresponding pages to each section
3. Assign appropriate icons to each section/item

**Note:** The exact menu structure (Dashboard, My Requests, Live Services, Service Tickets, Reports & Documents) must be configured in the Creator UI.

---

## 10. Custom CSS

**What:** Add design tokens (CSS variables) and RTL support.

**Where:** Creator UI → Settings → Themes → Custom CSS (or Devices → Web → Theme)

**Steps:**
1. Add the CSS design tokens from PROJECT_SPEC.md section 10:
   ```css
   :root {
     --primary: #006250;
     --primary-strong: #004336;
     --bg: #FAF9FC;
     --fg: #1C152F;
     --card: #FFFFFF;
     --muted: #F6F4FA;
     --muted-fg: #666072;
     --border: #E6E3EC;
     --success: #007560;
     --warning: #EBA941;
     --danger: #DB2B33;
     --info: #558AC0;
     --radius: 12px;
     --radius-card: 20px;
     --radius-hero: 24px;
     --shadow-card: 0 4px 12px -2px rgb(28 21 47 / .08);
     --shadow-elevated: 0 16px 40px -8px rgb(28 21 47 / .14);
   }
   ```
2. Add RTL support rules for Arabic layout
3. Configure fonts (Plus Jakarta Sans, Inter, IBM Plex Sans Arabic)

**Note:** The DS page files will use these CSS variables, but the variables themselves must be defined in the app's custom CSS.

---

## Summary

| Step | Item | Location |
|------|------|----------|
| 1 | Connection setup | Settings → Developer Space → Connections |
| 2 | MFA requirement | Settings → Security → Multi-Factor Authentication |
| 3 | CRM webhook | Creator Webhooks + Zoho CRM Workflow Rules |
| 4 | Desk webhook | Zoho Desk → Admin → Channels → Webhooks |
| 5 | Seed data | Data → Import or manual entry |
| 6 | Profile permissions | Settings → Users & Roles → Profiles |
| 7 | Field visibility | Settings → Users & Roles → Profiles → Field Permissions |
| 8 | Report layouts | Forms → Reports → Device Layouts |
| 9 | Menu configuration | Settings → Mobile → Menu |
| 10 | Custom CSS | Settings → Themes → Custom CSS |

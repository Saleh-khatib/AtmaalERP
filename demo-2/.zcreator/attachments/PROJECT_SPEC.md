# PROJECT_SPEC — Managed Services Client Portal (Zoho Creator)

Generic working spec. Put this file in the app's working folder and refer to it from every prompt. No real client names, IDs or links are included on purpose.

---

## 1. Purpose and golden rules

A private, invite-only customer portal for a managed-services company. External client users sign in to see their own company's service requests, support tickets and documents, and to act when the provider needs something (reply, approve, sign, upload, confirm).

**Golden rules (never break these):**
1. **Zoho CRM, Zoho Desk, Zoho WorkDrive and Zoho Sign are the systems of record.** This app never stores or recreates requests, tickets, deals, cases or documents. Do NOT create forms named Request, Ticket, Deal, Case or Document.
2. The app stores only portal data: companies, portal users and roles, a thin request index, notifications, audit log, stage-label configuration, UI strings (EN/AR) and an error log.
3. A user may only ever see records of their own company. The company must be derived from the logged-in portal user's record on the server, never from a value sent by the browser.
4. **Do NOT create blueprints, lifecycle workflows, stage transitions or approval processes.** Request stages and approvals are managed in the CRM; this app only maps stages to client labels.
5. Do NOT use Google Sheets, Freshdesk, CRM Quotes or CRM PurchaseOrders anywhere.
6. Never invent facts. If something cannot be expressed in the app files, write it into `MANUAL_STEPS.md` with the exact steps for a human to do in the Creator UI instead of faking it.
7. Do not ask the user for secrets. Connections to other Zoho products are created by a human in the Creator UI; code refers to them by these names: `crm_conn`, `desk_conn`, `workdrive_conn`, `sign_conn`.

## 2. Users and roles (all external portal users)

| Role | Can do |
|---|---|
| Requester | View everything for their company; start new requests; reply to information requests and tickets; upload documents to open requests; raise tickets; download documents; confirm contacts; mark UAT scenarios |
| Approver | Everything a Requester can, plus approve proposals and commercials, sign agreements (Zoho Sign), sign off UAT, approve go-live |
| Viewer | Read-only; can download documents; no action buttons |

Each user belongs to exactly one company. Internal staff do not use the portal. Approvers do NOT see the audit log.

## 3. Data model (portal data only)

Use these form names and link names. Add default system fields as Creator requires. Make "unique" fields unique.

### 3.1 Account (company lookup)
Account_ID (unique), Account_Name.

### 3.2 Portal_User
Login_Email (unique), Full_Name, Phone, CRM_Contact_ID, Account (lookup to Account), Account_ID, Account_Name, Role (Requester / Approver / Viewer), Language (English / Arabic), Active (yes/no), Email_Notifications (yes/no).
Login_Email, Account and Role are read-only for the portal user themselves; only administrators change them.

### 3.3 Request_Index (thin index of CRM deals — not the request)
CRM_Deal_ID (unique), Reference, Account (lookup to Account), Account_ID, Service_Name, Client_Label, Stage, Dashboard_Group (New / In progress / Needs your action / Completed), Action_Required (yes/no), Action_Type, Requester_Contact_ID, Last_Notified_Stage, Last_Modified (date-time).

### 3.4 Notification
Portal_User (lookup), Account (lookup), Event_Type (Request_Submitted / Status_Changed / Info_Required / Proposal_Available / Approval_Required / Signature_Required / PO_Required / Request_Completed / Ticket_Updated), Reference, Title_EN, Title_AR, Body_EN, Body_AR, Created_Time, Read (yes/no).

### 3.5 Audit_Log (administrators only)
Event_Time, Portal_User (lookup), Account_ID, Action, Zoho_Module, Zoho_Record_ID, Result, Error_Text.

### 3.6 Config_Stage_Map (administrators only)
CRM_Stage_Value (unique), Client_Label_EN, Client_Label_AR, Dashboard_Group (same four values), Who_Acts (Client / Provider), ETA_Text, Sort_Order (number).

### 3.7 Config_Strings (administrators only)
String_Key (unique), Text_EN, Text_AR.

### 3.8 Error_Log (administrators only)
Event_Time, Function_Name, Message, Reference.

## 4. Access rules

- Private portal, invite-only, no self sign-up, MFA required for portal users.
- Portal users see only `Request_Index` and `Notification` records whose Account matches their own; and only their own `Portal_User` record.
- Portal users cannot see Audit_Log, Config_Stage_Map, Config_Strings, Error_Log.
- Viewer is read-only everywhere. Requester and Approver cannot edit Request_Index.
- Every function that reads or writes CRM, Desk, WorkDrive or Sign must first load the logged-in user's Portal_User record, take the Account_ID and Role from it, and refuse if the target record does not belong to that company or the role is not allowed.

## 5. Pages and navigation

Left navigation, in this order: **Dashboard, My Requests, Live Services, Service Tickets, Reports & Documents.** Secondary pages: Request Detail, New Request, Profile, Notifications.

| Page | Content |
|---|---|
| Dashboard | Greeting by first name; hero card with primary button "New service request" (Requester and Approver only); "Needs your attention" list (Request_Index where Action_Required = yes) with reference, client label and one-click action; four count tiles per Dashboard_Group; last 5 notifications; small "Live Services — Coming soon" teaser card |
| My Requests | List from Request_Index; tabs All / Needs your action / In progress / Completed with counts; search on Reference, Service_Name, Client_Label; filters (group, date range); row opens Request Detail; Viewer sees no action buttons |
| Request Detail | Header (reference, client label, last modified); status journey bar (segments from Config_Stage_Map ordered by Sort_Order, client-action stages marked); action card when Action_Required = yes (shows Action_Type and the buttons allowed for the user's role); documents section (list, upload for Requester/Approver); activity section (client-visible actions and status changes, visible to all roles) |
| New Request | Page with a short intro and an empty container `id="gc-container"` where an external Zoho Desk guided-conversation flow will be embedded later. The portal never creates the CRM deal. Visible to Requester and Approver only |
| Live Services | "Coming soon" badge and three greyed placeholder cards (Service level, Quality, Incidents). No data, no links |
| Service Tickets | List of Zoho Desk tickets for the user's company (number, subject, status, priority, modified); detail drawer with the full thread; reply with attachments; "New ticket" form (subject, category, priority, description, attachments). Data comes from Desk later through `desk_conn`; build the layout and the functions now |
| Reports & Documents | Left category list with counts: Weekly Checkups, Monthly Performance Reports, Quarterly Reviews, Purchase Orders, Contracts, Discovery Summaries; right panel with file list (name, date, size, direction), search, preview and download. Files stay in WorkDrive/CRM; the page shows links and previews only |
| Profile | Editable: full name, phone, language, email-notification toggle, password change. Read-only: login email, company, role. Sign out button |
| Notifications | Bell with unread count; panel listing notifications (title in the user's language, time, unread dot), "Mark all as read"; each item opens the related request or ticket |

Live Services and Reports categories are placeholders for later phases where noted.

## 6. Lifecycle reference (Config_Stage_Map seed)

The 17-stage journey managed in the CRM. Replace stage values with the real CRM values at the mapping workshop. "In CRM today" = already exists; "Proposed" = may need to be added by the provider.

| # | CRM_Stage_Value | Client_Label_EN | Client_Label_AR (draft) | Dashboard_Group | Who_Acts | In CRM today |
|---|---|---|---|---|---|---|
| 1 | Backlog | Request submitted | تم تقديم الطلب | New | Provider | Yes |
| 2 | Process Assessment | Discovery scheduled | تم تحديد جلسة الاستكشاف | Needs your action | Client | Yes |
| 3 | Requirements Confirmation | Requirements confirmed | تم تأكيد المتطلبات | Needs your action | Client | Proposed |
| 4 | Capacity Assessment | Preparing your solution | جارٍ إعداد الحل الخاص بك | In progress | Provider | Yes |
| 5 | Proposal Review | Proposal ready for your approval | العرض جاهز لاعتمادك | Needs your action | Client | Yes |
| 6 | Client Approved | With your procurement team | لدى فريق المشتريات لديكم | Needs your action | Client | Yes |
| 7 | SOW | Ready for signature | جاهز للتوقيع | Needs your action | Client | Yes |
| 8 | Client Signed | Provider counter-signing | التوقيع من جانب مزود الخدمة | In progress | Provider | Proposed |
| 9 | Accepted | Awaiting your PO | بانتظار أمر الشراء الخاص بكم | Needs your action | Client | Yes |
| 10 | PO Confirmed | Preparing your handover | جارٍ إعداد التسليم | In progress | Provider | Proposed |
| 11 | Handover | Confirm project contacts | تأكيد جهات الاتصال للمشروع | Needs your action | Client | Proposed |
| 12 | Mobilization | Mobilising your team | جارٍ تجهيز الفريق | In progress | Provider | Yes |
| 13 | Implementation | Building your service | جارٍ بناء خدمتك | In progress | Provider | Proposed |
| 14 | UAT | Ready for your testing | جاهز لاختباركم | Needs your action | Client | Proposed |
| 15 | Go-Live Readiness | Approve go-live | اعتماد الإطلاق | Needs your action | Client | Proposed |
| 16 | Hypercare | Hypercare — live with monitoring | الدعم المكثف بعد الإطلاق | In progress | Provider | Yes |
| 17 | Live | Live operations | التشغيل الفعلي | Completed | Provider | Proposed |

The client acts at 9 stages (2, 3, 5, 6, 7, 9, 11, 14, 15). Arabic labels are drafts for the provider to approve.

## 7. Client actions (functions write to the CRM, not to this app)

| Action | Role | Stage | Effect in CRM |
|---|---|---|---|
| Confirm discovery session | Requester, Approver | 2 | Event RSVP; provider advances the stage |
| Confirm requirements / request changes | Requester, Approver | 3 | Confirm moves stage on; changes saved as a note with a task |
| Approve proposal / request revision | Approver | 5 | Stage moves on, or note and stage back |
| Approve commercials and issue PR | Approver | 6 | Stage moves on; PR number and budget code saved |
| Sign agreement | Approver | 7 | Zoho Sign request; completion updates the stage |
| Enter PO | Requester, Approver | 9 | PO number and file saved |
| Confirm project contacts | Requester, Approver | 11 | Onboarding project updated |
| UAT sign-off | Requester (mark), Approver (sign off) | 14 | Results saved; stage moves on |
| Approve go-live | Approver | 15 | Stage moves to Hypercare |
| Reply / upload | Requester, Approver | any open request | Note and file on the CRM record; provider gets a note or task, never a client notification |

Every action: validate role and company, perform the CRM write, write an Audit_Log record, refresh the Request_Index record, create an in-app confirmation.

## 8. Notifications

- Create an in-app Notification for every event type in 3.4 (title and body in EN and AR).
- Email the user when Email_Notifications = yes, except Status_Changed (in-app only for now) and Signature_Required (Zoho Sign sends its own email).
- Use Last_Notified_Stage so the same stage never produces two Status_Changed notifications.
- Request_Completed only when the request reaches its final stage.
- A Zoho Desk agent reply triggers Ticket_Updated through a Desk webhook.

## 9. Integration behaviour (code now, connect later)

- CRM webhook (called by a CRM workflow rule) creates or updates a Request_Index record and a notification. Secure it with a shared secret passed in a header.
- A scheduled job about every 15 minutes reconciles open requests with the CRM. It logs only mismatches fixed and failures in Error_Log (not routine runs) and avoids duplicate notifications.
- Requests are read from CRM Deals (with Accounts, Contacts, Onboarding_Projects). Tickets from Zoho Desk. Documents from WorkDrive and CRM. Signing through Zoho Sign.
- Large files are NOT proxied through scripts (Creator limits downloads to 5 MB in URL tasks). Use WorkDrive or CRM preview and share links shown only to the entitled user.
- Every external call is wrapped in error handling that writes to Error_Log.

## 10. Language and look

English and Arabic with right-to-left layout for Arabic. UI text comes from Config_Strings, never hard-coded, in every page snippet. Custom pages and snippets must set `dir="rtl"` for Arabic and use logical CSS properties (margin-inline-start, padding-inline-end, text-align: start).

**Design tokens (CSS variables in one shared block):**
```
--primary: #006250;  --primary-strong: #004336;
--bg: #FAF9FC;  --fg: #1C152F;  --card: #FFFFFF;
--muted: #F6F4FA;  --muted-fg: #666072;  --border: #E6E3EC;
--success: #007560;  --warning: #EBA941;  --danger: #DB2B33;  --info: #558AC0;
--radius: 12px;  --radius-card: 20px;  --radius-hero: 24px;
--shadow-card: 0 4px 12px -2px rgb(28 21 47 / .08);
--shadow-elevated: 0 16px 40px -8px rgb(28 21 47 / .14);
```
Fonts: Plus Jakarta Sans (titles), Inter (body), IBM Plex Sans Arabic (Arabic).
Style: calm and premium, white cards on a very light background, hero card with gradient from #004336 to #006250, 20 px card radius, status chips 11 px bold with tinted background, 150 ms hover transitions, skeleton loading, designed empty and error states, touch targets of at least 44 px, tables become cards on phone widths, `prefers-reduced-motion` respected.

## 11. UI copy (seed Config_Strings with these; EN / AR draft)

| Key | EN | AR |
|---|---|---|
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
| tab.all / tab.action / tab.progress / tab.completed | All / Needs your action / In progress / Completed | الكل / يحتاج إلى إجراء / قيد التنفيذ / مكتمل |

## 12. Acceptance checks (the AI must make these true)

1. Exactly eight forms exist; none named Request, Ticket, Deal, Case or Document.
2. A Requester of company A cannot see, open or act on any record of company B through any page, report, function or URL.
3. Audit_Log, Error_Log and both Config forms are invisible to all portal users.
4. Viewer sees no action buttons and cannot call any write function.
5. Every function checks role and company before touching external systems.
6. Every page works in English and Arabic (RTL), and all text comes from Config_Strings.
7. No blueprint, lifecycle workflow or approval process exists.
8. Everything that cannot be expressed in app files is listed in `MANUAL_STEPS.md`.

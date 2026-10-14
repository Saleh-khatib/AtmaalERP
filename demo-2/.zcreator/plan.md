This phase creates the foundational data layer for the Managed Services Client Portal: eight forms storing only portal-specific data (companies, users, request index, notifications, audit logs, and configuration). No pages, workflows, functions, or permissions are included yet. The forms obey all golden rules: no forms named Request/Ticket/Deal/Case/Document, no blueprints or approval processes.

## User Profiles

Define the three external portal user profiles (Requester, Approver, Viewer) that will be used throughout the app. These are shell profiles created now; their full permissions (field and report access) will be added in a later phase after all forms exist.

Create three custom profile files under permissions/:

1. Requester { type = Predefined:false; intended access: view own company records, create requests, reply, upload, download, mark UAT; cannot see Audit_Log/Error_Log/Config forms }
2. Approver { type = Predefined:false; intended access: all Requester permissions plus approve proposals, sign agreements, sign UAT, approve go-live; cannot see Audit_Log/Error_Log/Config forms }
3. Viewer { type = Predefined:false; intended access: read-only view of own company records, download documents; no action buttons, no write functions }

Note: Default profiles (Read, Write, Developer, Administrator, Customer) are auto-created by the system. Only these three custom profiles are written here. ModulePermissions are added later in the Permissions segment.

## Account

Create the Account form that stores company information. This is the primary lookup target for all other forms.

Form: Account
Fields:
- Account_ID (type = text, must have, unique, displayname = "Account ID")
- Account_Name (type = text, must have, displayname = "Account Name")
- Added_User (type = email, displayname = "Added By")
- Modified_User (type = email, displayname = "Modified By")
- Added_Time (type = datetime, displayname = "Added Time")
- Modified_Time (type = datetime, displayname = "Modified Time")

Report: Account_List (type = list, show all rows from Account, display columns: Account_ID, Account_Name)

Write scope: components/Account/form/Account.ds, components/Account/reports/Account_List/Account_List.ds

## Portal_User

Create the Portal_User form storing external portal user records. Each user belongs to exactly one company and has a specific role.

Form: Portal_User
Fields:
- Login_Email (type = email, must have, unique, displayname = "Login Email")
- Full_Name (type = text, must have, displayname = "Full Name")
- Phone (type = phonenumber, displayname = "Phone")
- CRM_Contact_ID (type = text, displayname = "CRM Contact ID")
- Account (type = picklist, values = Account.ID, displayformat = [Account_Name], must have, displayname = "Account")
- Account_ID (type = text, displayname = "Account ID")
- Account_Name (type = text, displayname = "Account Name")
- Role (type = picklist, values = {"Requester", "Approver", "Viewer"}, must have, displayname = "Role")
- Language (type = picklist, values = {"English", "Arabic"}, must have, displayname = "Language")
- Active (type = checkbox, displayname = "Active")
- Email_Notifications (type = checkbox, displayname = "Email Notifications")
- Added_User (type = email)
- Modified_User (type = email)
- Added_Time (type = datetime)
- Modified_Time (type = datetime)

Note: Login_Email, Account, and Role are read-only for portal users (enforced via permissions later).

Report: Portal_User_List (type = list, show all rows from Portal_User, display columns: Login_Email, Full_Name, Account_Name, Role, Active)

Write scope: components/Portal_User/form/Portal_User.ds, components/Portal_User/reports/Portal_User_List/Portal_User_List.ds

## Request_Index

Create the Request_Index form - a thin index of CRM deals. This form does NOT store the actual request; it only indexes CRM deal data for portal display.

Form: Request_Index
Fields:
- CRM_Deal_ID (type = text, must have, unique, displayname = "CRM Deal ID")
- Reference (type = text, displayname = "Reference")
- Account (type = picklist, values = Account.ID, displayformat = [Account_Name], must have, displayname = "Account")
- Account_ID (type = text, displayname = "Account ID")
- Service_Name (type = text, displayname = "Service Name")
- Client_Label (type = text, displayname = "Client Label")
- Stage (type = text, displayname = "Stage")
- Dashboard_Group (type = picklist, values = {"New", "In progress", "Needs your action", "Completed"}, must have, displayname = "Dashboard Group")
- Action_Required (type = checkbox, displayname = "Action Required")
- Action_Type (type = text, displayname = "Action Type")
- Requester_Contact_ID (type = text, displayname = "Requester Contact ID")
- Last_Notified_Stage (type = text, displayname = "Last Notified Stage")
- Last_Modified (type = datetime, displayname = "Last Modified")
- Added_User (type = email)
- Modified_User (type = email)
- Added_Time (type = datetime)
- Modified_Time (type = datetime)

Report: Request_Index_List (type = list, show all rows from Request_Index, display columns: Reference, Account_Name, Service_Name, Client_Label, Stage, Dashboard_Group, Action_Required, Last_Modified)

Write scope: components/Request_Index/form/Request_Index.ds, components/Request_Index/reports/Request_Index_List/Request_Index_List.ds

## Notification

Create the Notification form storing in-app notifications for portal users. Each notification has English and Arabic text.

Form: Notification
Fields:
- Portal_User (type = picklist, values = Portal_User.ID, displayformat = [Full_Name], must have, displayname = "Portal User")
- Account (type = picklist, values = Account.ID, displayformat = [Account_Name], must have, displayname = "Account")
- Event_Type (type = picklist, values = {"Request_Submitted", "Status_Changed", "Info_Required", "Proposal_Available", "Approval_Required", "Signature_Required", "PO_Required", "Request_Completed", "Ticket_Updated"}, must have, displayname = "Event Type")
- Reference (type = text, displayname = "Reference")
- Title_EN (type = text, must have, displayname = "Title (English)")
- Title_AR (type = text, must have, displayname = "Title (Arabic)")
- Body_EN (type = textarea, displayname = "Body (English)")
- Body_AR (type = textarea, displayname = "Body (Arabic)")
- Created_Time (type = datetime, must have, displayname = "Created Time")
- Read (type = checkbox, displayname = "Read")
- Added_User (type = email)
- Modified_User (type = email)
- Added_Time (type = datetime)
- Modified_Time (type = datetime)

Report: Notification_List (type = list, show all rows from Notification, display columns: Portal_User, Account_Name, Event_Type, Title_EN, Created_Time, Read)

Write scope: components/Notification/form/Notification.ds, components/Notification/reports/Notification_List/Notification_List.ds

## Audit_Log

Create the Audit_Log form for administrative tracking of all portal actions. This form is invisible to all portal users.

Form: Audit_Log
Fields:
- Event_Time (type = datetime, must have, displayname = "Event Time")
- Portal_User (type = picklist, values = Portal_User.ID, displayformat = [Full_Name], displayname = "Portal User")
- Account_ID (type = text, must have, displayname = "Account ID")
- Action (type = text, must have, displayname = "Action")
- Zoho_Module (type = text, displayname = "Zoho Module")
- Zoho_Record_ID (type = text, displayname = "Zoho Record ID")
- Result (type = text, displayname = "Result")
- Error_Text (type = textarea, displayname = "Error Text")
- Added_User (type = email)
- Modified_User (type = email)
- Added_Time (type = datetime)
- Modified_Time (type = datetime)

Report: Audit_Log_List (type = list, show all rows from Audit_Log, display columns: Event_Time, Portal_User, Account_ID, Action, Zoho_Module, Result)

Write scope: components/Audit_Log/form/Audit_Log.ds, components/Audit_Log/reports/Audit_Log_List/Audit_Log_List.ds

## Config_Stage_Map

Create the Config_Stage_Map form for administrators to configure stage-to-label mappings. This form is invisible to all portal users.

Form: Config_Stage_Map
Fields:
- CRM_Stage_Value (type = text, must have, unique, displayname = "CRM Stage Value")
- Client_Label_EN (type = text, must have, displayname = "Client Label (English)")
- Client_Label_AR (type = text, must have, displayname = "Client Label (Arabic)")
- Dashboard_Group (type = picklist, values = {"New", "In progress", "Needs your action", "Completed"}, must have, displayname = "Dashboard Group")
- Who_Acts (type = picklist, values = {"Client", "Provider"}, must have, displayname = "Who Acts")
- ETA_Text (type = text, displayname = "ETA Text")
- Sort_Order (type = number, must have, displayname = "Sort Order")
- Added_User (type = email)
- Modified_User (type = email)
- Added_Time (type = datetime)
- Modified_Time (type = datetime)

Report: Config_Stage_Map_List (type = list, show all rows from Config_Stage_Map, display columns: CRM_Stage_Value, Client_Label_EN, Client_Label_AR, Dashboard_Group, Who_Acts, Sort_Order)

Write scope: components/Config_Stage_Map/form/Config_Stage_Map.ds, components/Config_Stage_Map/reports/Config_Stage_Map_List/Config_Stage_Map_List.ds

## Config_Strings

Create the Config_Strings form for administrators to manage bilingual UI strings. This form is invisible to all portal users.

Form: Config_Strings
Fields:
- String_Key (type = text, must have, unique, displayname = "String Key")
- Text_EN (type = text, must have, displayname = "Text (English)")
- Text_AR (type = text, must have, displayname = "Text (Arabic)")
- Added_User (type = email)
- Modified_User (type = email)
- Added_Time (type = datetime)
- Modified_Time (type = datetime)

Report: Config_Strings_List (type = list, show all rows from Config_Strings, display columns: String_Key, Text_EN, Text_AR)

Write scope: components/Config_Strings/form/Config_Strings.ds, components/Config_Strings/reports/Config_Strings_List/Config_Strings_List.ds

## Error_Log

Create the Error_Log form for administrative tracking of function errors. This form is invisible to all portal users.

Form: Error_Log
Fields:
- Event_Time (type = datetime, must have, displayname = "Event Time")
- Function_Name (type = text, must have, displayname = "Function Name")
- Message (type = textarea, must have, displayname = "Message")
- Reference (type = text, displayname = "Reference")
- Added_User (type = email)
- Modified_User (type = email)
- Added_Time (type = datetime)
- Modified_Time (type = datetime)

Report: Error_Log_List (type = list, show all rows from Error_Log, display columns: Event_Time, Function_Name, Message, Reference)

Write scope: components/Error_Log/form/Error_Log.ds, components/Error_Log/reports/Error_Log_List/Error_Log_List.ds

## Manual Steps

Create MANUAL_STEPS.md listing anything that cannot be done programmatically and must be done manually in the Creator UI.

Create MANUAL_STEPS.md file at the app root listing:

1. Connection setup: Create connections crm_conn, desk_conn, workdrive_conn, sign_conn in the Creator UI (Connections page) - the app references them by these names but cannot create the connections itself.

2. MFA requirement: Enable MFA for portal users in the Creator UI (Settings > Security > Multi-Factor Authentication) - this cannot be expressed in DS files.

3. Webhook configuration: Set up the CRM webhook endpoint URL and configure the CRM workflow rule to call it with the shared secret header - the DS code is ready but the CRM-side trigger must be configured manually.

4. Desk webhook: Configure the Zoho Desk webhook to trigger Ticket_Updated notifications - the DS function is ready but the Desk-side webhook must be set up manually.

5. Seed data: Populate Config_Stage_Map with the 17-stage journey (section 6 of PROJECT_SPEC.md) and Config_Strings with the UI copy (section 11) - these must be inserted manually or via import.

6. Profile permissions: After all forms are created, manually verify in the Creator UI that portal users cannot access Audit_Log, Error_Log, Config_Stage_Map, and Config_Strings (even though the DS files will define these restrictions).

7. Role-based field visibility: The read-only status of Login_Email, Account, and Role for portal users must be enforced via profile permissions in the Creator UI after the profiles are created.

8. Report device layouts: For each report, the web.ds device layout file must be created manually in the Creator UI (or via DS in a later phase) to define quickview/detailview layouts.

9. Menu configuration: The left navigation structure (Dashboard, My Requests, Live Services, Service Tickets, Reports & Documents) must be configured in the Creator UI menu sections.

10. Custom CSS: The design tokens (CSS variables) and RTL support must be added to the app's custom CSS in the Creator UI - the DS files will use these but cannot define them directly.
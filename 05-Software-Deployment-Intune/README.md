## 05 - Software Deployment via Intune

A manager submitted a ticket requesting Microsoft 365 Apps be deployed to all loan officers by end of day. Rather than remoting into each machine individually — which is time-consuming and error-prone — the deployment was handled through Microsoft Intune to push the software automatically to all targeted devices simultaneously using group-based assignment.

---

## Step 1: Assign the Intune Administrator Role

Before any app deployment could be configured in Intune, the admin account needed the correct role assigned. The **Intune Administrator** role was assigned to the tenant account in Entra ID. Without this role the Intune portal restricts access to app creation and deployment configuration.

*Entra ID Admin Center → Roles & admins → Intune Administrator → Add assignments → Javier Johnson*

<img width="1552" height="897" alt="assigned intune admin to tenant account" src="https://github.com/user-attachments/assets/98557762-8584-476c-9345-f8dcb7130522" />


---

## Step 2: Navigate to App Creation in Intune

Navigated to the Intune admin center at **intune.microsoft.com** and went to **Apps → All Apps → + Add**.

Configured the app type:
- **Platform:** Windows
- **App type:** Microsoft 365 Apps for Windows 10 and later

This app type allows deployment of the full Office suite to managed Windows devices without requiring a manual installer — Intune handles the download and installation automatically on enrolled devices.

<img width="1621" height="855" alt="selecting m365 apps for app deployment" src="https://github.com/user-attachments/assets/8686c02e-90a8-4374-a77f-172cdab7118c" />


---

## Step 3: Configure the App Suite

In the **Configure app suite** step, the specific Office applications were selected for the deployment. Five apps appropriate for the Loan Officers role were chosen:

- ✅ Excel
- ✅ OneNote
- ✅ Outlook
- ✅ Teams
- ✅ Word

Access, PowerPoint, Publisher, and Skype for Business were left unselected — applying **Principle of Least Privilege** to software deployment. Users only receive the tools their role requires, reducing attack surface and license consumption.

Additional settings:
- **Version to install:** Latest
- **Architecture:** 64-bit
- **Update channel:** Current Channel (Preview)

<img width="1616" height="808" alt="m365 app selection" src="https://github.com/user-attachments/assets/86f2a406-9d2c-469f-8518-1a285187a8b6" />


---

## Step 4: Assign the Deployment to the Loan Officers Group

In the **Assignments** step, the Loans Officer security group was added under the **Required** section.

**Required** was selected over **Available** because the users needed the software deployed automatically by end of day:

| Assignment Type | Behavior |
|---|---|
| Required | App installs automatically on all enrolled devices in the group — no user action needed |
| Available | App appears in the Company Portal for users to install manually at their own discretion |

<img width="1648" height="992" alt="loans officer app assignment added group to it" src="https://github.com/user-attachments/assets/b2f6ef79-49a9-4145-8dba-435639a22bcb" />


---

## Step 5: Review and Confirm the Deployment

The completed app deployment was reviewed in the Properties page. All settings were confirmed:

- **Apps included:** Excel, OneNote, Outlook, Teams, Word
- **Architecture:** 64-bit
- **Update channel:** Current Channel (Preview)
- **Assignment:** Loans Officer group — Required, Included, Active

The deployment is live and will push automatically to enrolled devices on the next Intune sync cycle — typically within 15 to 30 minutes. No manual action is required on any enrolled device.

<img width="1617" height="902" alt="confirmed app deployment for loan officer group" src="https://github.com/user-attachments/assets/5817257c-597d-4e61-8c52-33a87feaa24e" />


---

## Summary

| Task | Action |
|---|---|
| Assign Intune role | Entra ID — Intune Administrator role assigned to admin account |
| Select app type | Intune → Apps → Add → Microsoft 365 Apps for Windows 10 and later |
| Configure suite | Selected Excel, OneNote, Outlook, Teams, Word — Principle of Least Privilege applied |
| Assign to group | Loans Officer group added under Required for automatic installation |
| Confirm deployment | Verified Properties page — assignment Active and deployment live |

Intune app deployments push automatically on the next sync cycle — typically 15 to 30 minutes. Always assign to a specific security group rather than All Users to maintain controlled and targeted software distribution. Required installs automatically — Available requires user action in the Company Portal.

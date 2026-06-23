## 02 - Employee Offboarding

HR sent an urgent email on a Friday afternoon: James Wright, Loan Officer, was being terminated effective immediately. The task was to revoke all access across Active Directory, Entra ID, and Microsoft 365 — executed in order to ensure no gap in access removal across on-prem and cloud systems.

---

## Step 1: Set Up the Environment

Before offboarding could be demonstrated, the Loan Officers OU and James Wright's account were created to mirror what would have existed from his original onboarding.

```powershell
New-ADOrganizationalUnit -Name "Loan Officers" `
  -Path "OU=Retail Banking,OU=First Broward Bank,DC=mydomain,DC=com"

New-ADUser `
  -Name "James Wright" `
  -GivenName "James" `
  -Surname "Wright" `
  -SamAccountName "jwright" `
  -UserPrincipalName "jwright@mydomain.com" `
  -Path "OU=Loan Officers,OU=Retail Banking,OU=First Broward Bank,DC=mydomain,DC=com" `
  -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
  -Enabled $true
```

<img width="1916" height="866" alt="james wright created user and placed in loan officers ou" src="https://github.com/user-attachments/assets/eb1d8d5f-2a9a-40fc-bf29-5117e4991873" />


---

## Step 2: Disable the AD Account

`Disable-ADAccount` was used to immediately block James Wright's on-prem domain access. This prevents him from authenticating to any domain-joined resource — workstations, file shares, or internal applications.

```powershell
Disable-ADAccount -Identity "jwright"
Get-ADUser -Identity "jwright" -Properties Enabled | Select Name, Enabled
```

<img width="1918" height="868" alt="jwright enabled status" src="https://github.com/user-attachments/assets/b6e42740-e60b-4ba2-87b1-6b378fbeb5ca" />


---

## Step 3: Revoke Entra ID Sign-In Sessions

In a hybrid environment, disabling the on-prem AD account handles on-prem access but cloud access must be handled separately in Entra ID. **Revoke Sessions** was executed from James Wright's profile in the Entra ID admin center.

*Entra ID Admin Center → Users → James Wright → Revoke Sessions*

This immediately invalidates all active authentication tokens across every device and application — Teams, Outlook, SharePoint, OneDrive, and any connected banking applications. The user is kicked out instantly regardless of which device they are on, without waiting for the AD sync cycle.

<img width="1615" height="921" alt="james wright revoked sessions" src="https://github.com/user-attachments/assets/e9b62920-0383-4c9c-a609-6f73d6cb3d7e" />


---

## Step 4: Remove James Wright from the Loan Officers Security Group

James Wright was removed from the Loan Officers security group in Entra ID. Removing a terminated employee from all security groups revokes their access to every resource that group controls — shared files, applications, and cloud services.

*Entra ID Admin Center → Groups → Loan Officers → Members → Remove James Wright*

<img width="1542" height="867" alt="removed james wright from loans officer group entra id" src="https://github.com/user-attachments/assets/767296b6-f6dd-4944-b0e3-d002d85757ed" />


---

## Step 5: Block Sign-In in Microsoft 365

Block Sign-In was applied to James Wright's account in the Microsoft 365 Admin Center. This cuts off all M365 cloud app access independently of AD and Entra ID — critical in a hybrid banking environment where the AD sync cycle may not have run yet.

*M365 Admin Center → Users → Active Users → James Wright → Block sign-in*

M365 confirmed the user will be automatically signed out of all Microsoft services within 60 minutes.

<img width="1878" height="962" alt="james wright blocked sign in for m365" src="https://github.com/user-attachments/assets/bd4e7c00-4ce0-41fd-8049-19fb899f8df8" />


---

## Summary

| Step | System | Action |
|---|---|---|
| 1 | Active Directory | Disable-ADAccount — blocks all on-prem domain access immediately |
| 2 | Entra ID | Revoke Sessions — invalidates all active cloud tokens immediately |
| 3 | Entra ID | Remove from security groups — revokes cloud resource access |
| 4 | Microsoft 365 | Block sign-in — cuts off all M365 app access independently of AD sync |

**Why each layer matters in a hybrid environment:**

| System | Why It Must Be Actioned Separately |
|---|---|
| Active Directory | Controls on-prem resource access — workstations, file shares, internal apps |
| Entra ID Sessions | AD sync does not invalidate active tokens — sessions must be revoked manually |
| Entra ID Groups | Group membership controls cloud resource access independently of account status |
| Microsoft 365 | Block sign-in cuts off M365 apps before the AD sync cycle runs |

This layered offboarding process matters in a banking environment because hybrid identity leaves multiple access paths open unless each one is closed deliberately. Always confirm all access layers are revoked before closing a termination ticket.

## 07 - Intune Device Enrollment & Hybrid Azure AD Join

A new company-issued laptop needed to be enrolled in Microsoft Intune so the bank could manage it, enforce security policies, and deploy required applications automatically. This lab covers configuring Intune Automatic Enrollment and setting up Microsoft Entra Connect Cloud Sync to enable Hybrid Azure AD Join — the infrastructure required for domain-joined devices to also be managed by Intune.

---

## Step 1: Configure Automatic Enrollment in Intune

Navigated to the Intune admin center at **intune.microsoft.com** and went to **Devices → Enrollment → Windows → Automatic Enrollment**. Changed the MDM user scope from None to All and saved.

*Intune → Devices → Enroll devices → Windows → Automatic Enrollment → MDM user scope: All → Save*

Setting MDM scope to All means any device that joins the Entra ID tenant will automatically enroll in Intune without requiring manual steps on the device. The portal confirmed **Successfully updated Microsoft Intune** after saving.

<img width="1613" height="866" alt="auto enrollment set for windows devices" src="https://github.com/user-attachments/assets/a76934b7-aed8-498a-ae04-a6f66885ca6c" />


---

## Step 2: Navigate to Entra Connect Cloud Sync

Navigated to the Entra ID Admin Center at **entra.microsoft.com** and went to **Identity → Hybrid management → Microsoft Entra Connect → Cloud Sync → New Configuration → AD to Microsoft Entra ID sync**.

Cloud Sync is the bridge between on-prem Active Directory (mydomain.com) and the Entra ID tenant. Without this sync, domain-joined devices are not visible in Entra ID and cannot be automatically enrolled in Intune.

<img width="1568" height="726" alt="new cloud sync config for microsoft entra connect" src="https://github.com/user-attachments/assets/a060f291-38b7-4d24-84ac-9891aa2524bd" />


---

## Step 3: Configure the New Cloud Sync

On the New cloud sync configuration page:

- **Domain:** mydomain.com (selected from dropdown after agent installation)
- **Enable password hash sync:** Checked — allows cloud authentication using the same credentials as on-prem AD

**Note:** The domain dropdown was empty at this stage because the provisioning agent had not yet been installed on DC01. The agent must be installed first before the domain appears in the dropdown.

<img width="1568" height="726" alt="new cloud sync config for microsoft entra connect" src="https://github.com/user-attachments/assets/3eceee87-ca39-4f63-ae1a-7b2ee0fe1599" />


---

## Step 4: Download and Install the Provisioning Agent

The Microsoft Entra Provisioning Agent was downloaded from the Entra admin center and installed on DC01. The agent is a lightweight service that runs on the domain controller and handles the sync communication between on-prem AD and Entra ID.

*Entra Admin Center → Identity → Hybrid management → Microsoft Entra Connect → Cloud Sync → Download agent*

The installer — **AADConnectProvisioningAgentSetup.exe** — was accepted and launched directly on DC01.

<img width="1797" height="772" alt="microsoft entra agent wizard" src="https://github.com/user-attachments/assets/dad00b75-5bf4-4ab3-8854-1a16e5b2459f" />


---

## Step 5: Connect the Agent to Microsoft Entra ID

The provisioning agent configuration wizard connected to the Entra ID tenant using the admin credentials already authenticated in the browser session. The **Connect Microsoft Entra ID** step confirmed existing credentials were already on file.

![Provisioning Agent Configuration wizard — Connect Microsoft Entra ID step confirming credentials already present for the tenant](images/05-agent-entra-connect.png)

---

## Step 6: Configure the Service Account (gMSA)

The **Configure Service Account** step set up a group Managed Service Account (gMSA) that the provisioning agent uses to authenticate to on-prem Active Directory and read objects for sync.

- **Option selected:** Create gMSA
- **Domain Admin credentials:** mydomain.com\jjohnson

The gMSA account is automatically created in AD and managed by Windows — no manual password rotation is required. This is the recommended service account type for the provisioning agent.

<img width="1800" height="797" alt="configure msa for entra agent" src="https://github.com/user-attachments/assets/9aaec331-738f-4460-8f89-6390727a5b8e" />


---

## Step 7: Connect the Agent to Active Directory

The **Connect Active Directory** step detected mydomain.com automatically and listed it under Configured Domains. This confirms the provisioning agent can communicate with the on-prem domain controller and has the correct credentials to read AD objects.

mydomain.com is now mapped as the source directory for the Cloud Sync configuration.

<img width="1791" height="795" alt="connect ad entra agent" src="https://github.com/user-attachments/assets/704194b7-c3f9-4aba-8a59-7dbc53382777" />


---

## Step 8: Confirm Agent Installation

The **Confirm** step displayed a summary of the full configuration before completing the installation:

- **Active Directory Configuration:** mydomain.com — provisioning gMSA account
- **Microsoft Entra ID Configuration:** JavierJohnson@jjtechlab.onmicrosoft.com

The green confirmation bar confirmed the provisioning agent installation and configuration is complete.

<img width="1813" height="848" alt="entra agent confirmation is done" src="https://github.com/user-attachments/assets/7d7e5de1-8595-4413-8538-957f07bc29f1" />


---

## Expected Outcome — Hybrid Azure AD Join

With the provisioning agent installed and Cloud Sync configured, the following occurs automatically:

| Action | Result |
|---|---|
| mydomain.com objects sync to Entra ID | Users and devices appear in Entra ID on each sync cycle |
| Client01 becomes Hybrid Azure AD Joined | Domain-joined device is now also registered in Entra ID |
| Intune detects the device | MDM scope = All triggers automatic enrollment |
| Client01 appears in Intune | Device visible under Intune → Devices → All Devices |
| Policies and apps push automatically | Compliance policies and app deployments apply to enrolled device |

---

## Summary

| Task | Action |
|---|---|
| Configure automatic enrollment | Intune → MDM user scope set to All |
| Navigate to Cloud Sync | Entra Admin Center → Microsoft Entra Connect → Cloud Sync |
| Configure new sync | Selected AD to Entra ID sync with password hash sync enabled |
| Install provisioning agent | Downloaded and installed AADConnectProvisioningAgentSetup.exe on DC01 |
| Connect to Entra ID | Authenticated agent to Entra ID tenant |
| Configure gMSA | Created group Managed Service Account for AD sync authentication |
| Connect to AD | mydomain.com detected and confirmed under Configured Domains |
| Confirm installation | Green confirmation — agent installed and configuration complete |

In a production environment the Cloud Sync agent runs on a 30-minute default sync cycle. Devices that join the domain after the agent is configured will automatically become Hybrid Azure AD Joined and enroll in Intune without any manual intervention — giving IT full cloud-based management of all domain-joined devices.

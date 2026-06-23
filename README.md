# IT-Support-Specialist-Lab
Real-world IT support scenario lab covering new hire onboarding, employee offboarding, network troubleshooting, printer issues, Intune software deployment, shared drive access, and hybrid device enrollment across Active Directory, Entra ID, and Microsoft 365.

## Overview
This lab series simulates real-world IT support scenarios drawn from active job descriptions across banking, healthcare, education, and enterprise environments. Each scenario was performed hands-on using a home lab environment running Windows Server 2022 (DC01), Windows 10 (Client01), Microsoft Entra ID, Microsoft 365, and Intune. Scenarios cover the full IT support lifecycle — user provisioning, offboarding, network troubleshooting, printer issues, software deployment, shared drive access, and device management. All tasks were executed via PowerShell and documented in the same format used on the job.

---

## Technologies Used
- Windows Server 2022 (DC01)
- Windows 10 Enterprise (Client01)
- Active Directory Domain Services
- Microsoft Entra ID
- Microsoft 365 Admin Center
- Microsoft Intune
- Microsoft Entra Connect Cloud Sync
- PowerShell
- VMware Workstation

---

## Lab Environment

| Machine | Role | Domain |
|---|---|---|
| DC01 | Domain Controller | mydomain.com |
| Client01 | Windows 10 Workstation | mydomain.com |
| Entra ID Tenant | Cloud Identity | jjtechlab.onmicrosoft.com |

---

## Scenarios Covered

**New Employee Onboarding** — Built a three-level OU structure for First Broward Bank, created a Tellers security group, provisioned Maria Santos' domain account with correct OU placement, assigned her to the Tellers group, and verified the full setup via PowerShell.

**Employee Offboarding** — Received an urgent termination ticket for James Wright. Executed a layered offboarding workflow across Active Directory, Entra ID, and Microsoft 365 — disabling the AD account, revoking all Entra ID sign-in sessions, removing the user from cloud security groups, and blocking M365 sign-in to ensure no gap in access removal across on-prem and cloud systems.

**Network Connectivity Troubleshooting** — A teller reported they could not reach any websites or the core banking application. Used a structured troubleshooting methodology — ping by IP, ping by hostname, nslookup, ipconfig /all, DNS forwarder verification, and DNS service status — to isolate the issue from the network layer up.

**Printer Troubleshooting** — A loan officer's printer stopped working while others in the department could still print. Diagnosed the issue through printer status, print queue, spooler service, and driver verification using Get-Printer, Get-PrintJob, Restart-Service Spooler, and Get-PrinterDriver.

**Software Deployment via Intune** — A manager requested Microsoft 365 Apps deployed to all loan officers by end of day. Assigned the Intune Administrator role in Entra ID, configured the Microsoft 365 app suite in Intune with role-appropriate apps, and assigned the deployment to the Loans Officer security group as Required for automatic installation.

**Shared Drive Access Issues** — A teller reported Access Denied on a shared drive she had previously accessed. Diagnosed the issue by verifying group membership, share permissions, NTFS permissions, and account status using PowerShell across both permission layers.

**Intune Device Enrollment & Hybrid Azure AD Join** — Configured the infrastructure for a new company-issued laptop to automatically enroll in Intune. Set MDM user scope to All for automatic enrollment, installed the Microsoft Entra Connect Provisioning Agent on DC01, configured a gMSA service account, and connected the agent to both on-prem AD and the Entra ID tenant to enable Hybrid Azure AD Join.

---

## Table of Contents

| # | Section | Description |
|---|---|---|
| 01 | [New Employee Onboarding](01-New-Employee-Onboarding/README.md) | Built OU structure, created security group, and provisioned Maria Santos' domain account |
| 02 | [Employee Offboarding](02-Employee-Offboarding/README.md) | Executed layered termination workflow across AD, Entra ID, and Microsoft 365 |
| 03 | [Network Connectivity Troubleshooting](03-Network-Connectivity-Troubleshooting/README.md) | Isolated a network issue using structured methodology from IP layer to DNS |
| 04 | [Printer Troubleshooting](04-Printer-Troubleshooting/README.md) | Diagnosed printer failure through queue, spooler, and driver verification |
| 05 | [Software Deployment via Intune](05-Software-Deployment-Intune/README.md) | Deployed Microsoft 365 Apps to Loan Officers via Intune group assignment |
| 06 | [Shared Drive Access Issues](06-Shared-Drive-Access-Issues/README.md) | Diagnosed Access Denied by verifying group membership, share, NTFS, and account status |
| 07 | [Intune Device Enrollment & Hybrid Azure AD Join](07-Intune-Device-Enrollment-Hybrid-Join/README.md) | Configured automatic enrollment and Entra Connect Cloud Sync for hybrid device management |

---

## Key Takeaways
- Executed end-to-end onboarding and offboarding workflows across on-prem AD and cloud identity systems
- Applied structured troubleshooting methodology to network, printer, and shared drive issues
- Deployed enterprise software through Intune using group-based assignment with Principle of Least Privilege
- Configured Hybrid Azure AD Join infrastructure to enable automatic Intune enrollment for domain-joined devices
- Demonstrated the ability to work across Active Directory, Entra ID, Microsoft 365, and Intune in a single environment
- Documented all tasks in the same format used on the job — scenario, steps, commands, and outcome

## 06 - Shared Drive Access Issues

A teller called the helpdesk reporting an Access Denied error when trying to open a shared drive where the team stores daily reports. She had access the previous day but could not get in today. Other users in the department were not affected. The task was to diagnose the issue by working through the full permission stack — group membership, share permissions, NTFS permissions, and account status.

---

## Step 1: Create the TellerReport Shared Folder

`New-Item` was used to create the TellerReport folder at C:\Shares\TellerReport. Creating it via PowerShell ensures the path and folder type are set correctly and is repeatable for documentation purposes.

```powershell
New-Item -Path "C:\Shares\TellerReport" -ItemType Directory
```

<img width="1916" height="823" alt="created tellerreports share" src="https://github.com/user-attachments/assets/e7770c59-4f46-42bc-9468-aba3e1347c3b" />


---

## Step 2: Create the Network Share with Share Permissions

`New-SmbShare` was used to share the folder over the network and assign Full Access to the Tellers security group at the share permission level. Share permissions control who can reach the folder across the network.

```powershell
New-SmbShare -Name "TellerReports-Share" `
  -Path "C:\Shares\TellerReport" `
  -FullAccess "Tellers"
```

By granting access to the Tellers group rather than individual users, any teller added to the group automatically gains access without requiring permission changes to the share.

<img width="1918" height="341" alt="added tellers security group to share report" src="https://github.com/user-attachments/assets/20761402-8e25-411b-ae28-6fe95e933483" />


---

## Step 3: Verify Share Permissions

`Get-SmbShareAccess` was run to verify the share permissions were applied correctly.

```powershell
Get-SmbShareAccess -Name "TellerReports-Share"
```

Output confirmed MYDOMAIN\Tellers has AccessControlType Allow and AccessRight Full on the TellerReports-Share. Share permissions are the first of two permission layers that must both allow access for a user to successfully reach a shared resource.

<img width="1906" height="786" alt="confirmed that tellers security group had share access" src="https://github.com/user-attachments/assets/c9338fa7-1327-414c-9162-9183d74c5b1f" />


---

## Step 4: Set NTFS Permissions with icacls

`icacls` was used to set NTFS permissions on the TellerReport folder for the Tellers group. NTFS permissions are the second permission layer and control access at the file system level independently of share permissions. Both layers must allow access — the effective permission is always the most restrictive of the two.

```cmd
icacls C:\Shares\TellerReport /grant Tellers:(OI)(CI)F
```

| Flag | Meaning |
|---|---|
| OI | Object Inherit — files inside the folder inherit the permission |
| CI | Container Inherit — subfolders inside the folder inherit the permission |
| F | Full Control |

Successfully processed 1 files confirms the NTFS permissions were applied.

<img width="1916" height="865" alt="set ntfs permissions on tellerrepot folder" src="https://github.com/user-attachments/assets/84f8e34c-b593-4996-9405-27fa53bed9b2" />


---

## Step 5: Verify NTFS Permissions

`icacls` was run without flags to display the current NTFS permission list on the folder and confirm the Tellers group entry was applied correctly.

```cmd
icacls C:\Shares\TellerReport
```

Output confirmed MYDOMAIN\Tellers has (OI)(CI)(F) — Full Control with Object and Container Inheritance. Both permission layers are now correctly configured for the Tellers group.

<img width="1918" height="850" alt="confirmed nfts permissions was set on tellerrport folder" src="https://github.com/user-attachments/assets/82998a30-eb93-4016-be15-a735c87addda" />


---

## Step 6: Verify Account Status

As a final check, `Get-ADUser` was run against msantos to verify the account is not disabled or locked out. A disabled or locked account would cause Access Denied errors on any network resource regardless of how permissions are configured.

```powershell
Get-ADUser -Identity msantos -Properties Enabled, LockedOut | Select Name, Enabled, LockedOut
```

Output confirmed Enabled: True and LockedOut: False — the account is healthy and the user should now be able to access the TellerReports share successfully.

<img width="1915" height="856" alt="confirmed that msantos account is enabled and not lockedout share" src="https://github.com/user-attachments/assets/477c418f-0790-47c4-bc36-1743d0b7bba8" />


---

## Summary

| Step | Command | Finding |
|---|---|---|
| Create folder | New-Item | TellerReport directory created at C:\Shares\TellerReport |
| Create share | New-SmbShare | TellerReports-Share created with Tellers Full Access |
| Verify share permissions | Get-SmbShareAccess | MYDOMAIN\Tellers — Allow, Full confirmed |
| Set NTFS permissions | icacls /grant | Tellers:(OI)(CI)F applied — Successfully processed 1 files |
| Verify NTFS permissions | icacls | MYDOMAIN\Tellers:(OI)(CI)(F) confirmed in ACL |
| Verify account status | Get-ADUser | Enabled: True, LockedOut: False confirmed |

**Windows permission layers:**

| Layer | Controls | Checked With |
|---|---|---|
| Share permissions | Network access to the shared folder | Get-SmbShareAccess |
| NTFS permissions | File system access on the actual folder | icacls |

Both layers must allow access. If either layer denies, the user receives Access Denied — always check both before drawing conclusions. Always verify account status last — a disabled or locked account blocks access regardless of how permissions are configured.

## 01 - New Employee Onboarding

A new teller named Maria Santos was starting Monday at First Broward Bank. The task was to build the OU structure, create a security group, provision her domain account, assign her to the correct group, and verify the full setup — all performed via PowerShell on DC01.

---

## Step 1: Create the OU Structure

`New-ADOrganizationalUnit` was used to build a three-level OU hierarchy — First Broward Bank → Retail Banking → Tellers. Parent OUs must always be created before child OUs can be nested inside them. A separate Tellers Workstation OU was also created under Retail Banking to keep computer objects independent from user objects for GPO targeting.

```powershell
New-ADOrganizationalUnit -Name "First Broward Bank" -Path "DC=mydomain,DC=com"
New-ADOrganizationalUnit -Name "Retail Banking" -Path "OU=First Broward Bank,DC=mydomain,DC=com"
New-ADOrganizationalUnit -Name "Tellers" -Path "OU=Retail Banking,OU=First Broward Bank,DC=mydomain,DC=com"
New-ADOrganizationalUnit -Name "Tellers Workstation" -Path "OU=Retail Banking,OU=First Broward Bank,DC=mydomain,DC=com"
```

**Note:** The first two commands returned ADException errors because those OUs already existed from previous lab work. The Tellers OU was created successfully on the third command.

<img width="1916" height="857" alt="creating ou stucture tellers, first bank" src="https://github.com/user-attachments/assets/c8bfc29b-cecb-4ed0-ab8d-7e1ac7b26ba4" />
<img width="1918" height="865" alt="tellers workstation ou created" src="https://github.com/user-attachments/assets/f7263d21-fb5e-40a2-b409-d245e3d803ba" />

---

## Step 2: Confirm the OU Was Created

`Get-ADOrganizationalUnit` was used to verify the Tellers OU exists at the correct path before proceeding with user and group creation.

```powershell
Get-ADOrganizationalUnit -Filter * | Where-Object { $_.Name -eq "Tellers" } | Select-Object Name, DistinguishedName
```

<img width="1918" height="852" alt="confirmed tellers ou was created" src="https://github.com/user-attachments/assets/de65cd10-3111-439e-96e7-ebfa865e5ddd" />


---

## Step 3: Create the Tellers Security Group

`New-ADGroup` was used to create the Tellers security group inside the Tellers OU. Security groups control access to resources — users inherit permissions through group membership rather than individual assignments, which enforces Principle of Least Privilege at scale.

```powershell
New-ADGroup `
  -Name "Tellers" `
  -GroupScope Global `
  -GroupCategory Security `
  -Path "OU=Tellers,OU=Retail Banking,OU=First Broward Bank,DC=mydomain,DC=com"
```

<img width="1917" height="873" alt="tellers security group created" src="https://github.com/user-attachments/assets/c74e57b0-e6c9-46fa-9080-23ed34319f42" />


---

## Step 4: Verify the Security Group

`Get-ADGroup` was used to confirm the group was created with the correct GroupCategory, GroupScope, and DistinguishedName showing the correct OU placement.

```powershell
Get-ADGroup -Identity "Tellers" 
```

<img width="1917" height="852" alt="tellers security group confirmation" src="https://github.com/user-attachments/assets/95e57957-165b-41b6-8de1-809e1b37fc68" />


---

## Step 5: Create Maria Santos' User Account

`New-ADUser` was used to provision Maria's domain account inside the Tellers OU. A temporary password was set with `ChangePasswordAtLogon` enabled so Maria sets her own password on first login — the admin never knows the user's final password. This is required security practice in a banking environment.

```powershell
New-ADUser `
  -Name "Maria Santos" `
  -GivenName "Maria" `
  -Surname "Santos" `
  -SamAccountName "msantos" `
  -UserPrincipalName "msantos@mydomain.com" `
  -Path "OU=Tellers,OU=Retail Banking,OU=First Broward Bank,DC=mydomain,DC=com" `
  -AccountPassword (ConvertTo-SecureString "TempPass123!" -AsPlainText -Force) `
  -ChangePasswordAtLogon $true `
  -Enabled $true
```

<img width="1916" height="865" alt="maria santos user created" src="https://github.com/user-attachments/assets/f904f94f-16b7-4969-b96c-3c4408169816" />


---

## Step 6: Add Maria to the Tellers Security Group

`Add-ADGroupMember` was used to assign msantos to the Tellers security group, granting her access to all teller resources controlled by that group.

```powershell
Add-ADGroupMember -Identity "Tellers" -Members "msantos"
```

No output on success is expected PowerShell behavior — the command ran successfully if no error was returned.

<img width="1913" height="653" alt="Maria added to tellers security group" src="https://github.com/user-attachments/assets/41c237ad-4a5f-4e17-964a-4b2d8781308d" />


---

## Step 7: Verify Group Membership

`Get-ADGroupMember` was used to confirm Maria Santos is listed as a member of the Tellers group before closing the ticket.

```powershell
Get-ADGroupMember -Identity "Tellers" 
```

<img width="1915" height="876" alt="maria confirmed added to tellers security group" src="https://github.com/user-attachments/assets/d26a50b2-8297-4919-9fd3-903841698838" />


---

## Step 8: Create the Tellers Workstation OU

A separate Tellers Workstation OU was created alongside the Tellers user OU, both sitting under Retail Banking. Keeping computer objects in their own OU allows GPOs that control workstation settings — such as Restricted Groups to prevent unauthorized local admin access — to be applied independently from user policies.

```powershell
New-ADOrganizationalUnit -Name "Tellers Workstation" -Path "OU=Retail Banking,OU=First Broward Bank,DC=mydomain,DC=com"
```

<img width="1918" height="865" alt="tellers workstation ou created" src="https://github.com/user-attachments/assets/a594cfac-3161-4745-bc16-ced3954d6d4f" />


---

## Summary

| Task | Command Used |
|---|---|
| Build OU hierarchy | New-ADOrganizationalUnit — three levels deep |
| Confirm OU placement | Get-ADOrganizationalUnit |
| Create security group | New-ADGroup with Global scope and Security category |
| Verify group | Get-ADGroup |
| Provision user account | New-ADUser with correct OU path and forced password reset |
| Add to group | Add-ADGroupMember |
| Verify membership | Get-ADGroupMember |
| Create workstation OU | New-ADOrganizationalUnit — separate from user OU |

Always verify group membership and account status before closing an onboarding ticket. Separating user and computer OUs enables independent GPO targeting — a best practice in enterprise Active Directory environments.

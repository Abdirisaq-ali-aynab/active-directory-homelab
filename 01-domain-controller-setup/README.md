# Lab 01 – Building a Domain Controller

Every Active Directory environment starts with a Domain Controller. In this lab I turned a fresh Windows Server into the first DC of a new forest, `goodcorp.local`, so users, computers and policies can be managed from one place.

## Objectives

- Install the Active Directory Domain Services (AD DS) role
- Promote the server to a Domain Controller in a new forest
- Verify AD and DNS are working
- Prove the DC works by creating and deleting a test OU

## Environment

| Item | Value |
|---|---|
| Server | Windows Server VM |
| New forest / domain | `goodcorp.local` (NetBIOS: `GOODCORP`) |
| Roles added | AD DS, DNS, Global Catalog |

## Steps

### Step 1 – Install the AD DS role

**Server Manager → Add roles and features → Role-based installation → Active Directory Domain Services → Add Features → Install.** The role installs, then Server Manager flags *Configuration required*.

PowerShell equivalent:
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```


<p>
<img src="images/step1-1.png" alt="Step 1 – Install the AD DS role – screenshot 1" width="49%">
<img src="images/step1-2.png" alt="Step 1 – Install the AD DS role – screenshot 2" width="49%">
<img src="images/step1-3.png" alt="Step 1 – Install the AD DS role – screenshot 3" width="49%">
<img src="images/step1-4.png" alt="Step 1 – Install the AD DS role – screenshot 4" width="49%">
<img src="images/step1-5.png" alt="Step 1 – Install the AD DS role – screenshot 5" width="49%">
<img src="images/step1-6.png" alt="Step 1 – Install the AD DS role – screenshot 6" width="49%">
<img src="images/step1-7.png" alt="Step 1 – Install the AD DS role – screenshot 7" width="49%">
<img src="images/step1-8.png" alt="Step 1 – Install the AD DS role – screenshot 8" width="49%">
<img src="images/step1-9.png" alt="Step 1 – Install the AD DS role – screenshot 9" width="49%">
<img src="images/step1-10.png" alt="Step 1 – Install the AD DS role – screenshot 10" width="49%">
</p>

### Step 2 – Promote the server to a Domain Controller

From the Server Manager notification, select **Promote this server to a domain controller**:

1. **Add a new forest** → root domain name `goodcorp.local`
2. Keep **DNS server** and **Global Catalog** checked
3. Set the **DSRM** (Directory Services Restore Mode) password
4. Skip the DNS delegation warning, confirm the NetBIOS name `GOODCORP`, pass the prerequisite check and **Install**. The server reboots as a DC.

PowerShell equivalent:
```powershell
Install-ADDSForest -DomainName "goodcorp.local" -DomainNetbiosName "GOODCORP" `
  -SafeModeAdministratorPassword (Read-Host -AsSecureString "DSRM password") -InstallDNS
```

> After promotion, the local Administrator automatically becomes the **domain** Administrator.


<p>
<img src="images/step2-1.png" alt="Step 2 – Promote the server to a Domain Controller – screenshot 1" width="49%">
<img src="images/step2-2.png" alt="Step 2 – Promote the server to a Domain Controller – screenshot 2" width="49%">
<img src="images/step2-3.png" alt="Step 2 – Promote the server to a Domain Controller – screenshot 3" width="49%">
<img src="images/step2-4.png" alt="Step 2 – Promote the server to a Domain Controller – screenshot 4" width="49%">
<img src="images/step2-5.png" alt="Step 2 – Promote the server to a Domain Controller – screenshot 5" width="49%">
<img src="images/step2-6.png" alt="Step 2 – Promote the server to a Domain Controller – screenshot 6" width="49%">
<img src="images/step2-7.png" alt="Step 2 – Promote the server to a Domain Controller – screenshot 7" width="49%">
<img src="images/step2-8.png" alt="Step 2 – Promote the server to a Domain Controller – screenshot 8" width="49%">
</p>

### Step 3 – Verify the installation

- **Tools → Active Directory Users and Computers**: `goodcorp.local` appears in the tree
- **Tools → DNS**: forward lookup zones for `goodcorp.local` were created

```powershell
Get-ADDomain
Get-ADForest
```


<p>
<img src="images/step3-1.png" alt="Step 3 – Verify the installation – screenshot 1" width="49%">
<img src="images/step3-2.png" alt="Step 3 – Verify the installation – screenshot 2" width="49%">
<img src="images/step3-3.png" alt="Step 3 – Verify the installation – screenshot 3" width="49%">
<img src="images/step3-4.png" alt="Step 3 – Verify the installation – screenshot 4" width="49%">
<img src="images/step3-5.png" alt="Step 3 – Verify the installation – screenshot 5" width="49%">
</p>

### Step 4 – Test the DC by creating an OU

In ADUC, right-click `goodcorp.local` → **New → Organizational Unit** → `TestOU` (with *Protect container from accidental deletion* unchecked), then delete it.

```powershell
New-ADOrganizationalUnit -Name "TestOU" -Path "DC=goodcorp,DC=local" -ProtectedFromAccidentalDeletion $false
```


<p>
<img src="images/step4-1.png" alt="Step 4 – Test the DC by creating an OU – screenshot 1" width="49%">
<img src="images/step4-2.png" alt="Step 4 – Test the DC by creating an OU – screenshot 2" width="49%">
<img src="images/step4-3.png" alt="Step 4 – Test the DC by creating an OU – screenshot 3" width="49%">
<img src="images/step4-4.png" alt="Step 4 – Test the DC by creating an OU – screenshot 4" width="49%">
</p>

## Verification

- `goodcorp.local` visible in ADUC and DNS Manager
- `Get-ADDomain` / `Get-ADForest` return the new domain
- Test OU created and removed successfully

## Troubleshooting Notes

- Pick the root domain name carefully: renaming a forest later is painful.
- If the OU will not delete, uncheck *Protect from accidental deletion* (View → Advanced Features → Object tab).
- Save the DSRM password somewhere safe. It is needed for AD recovery.

## Skills Demonstrated

- Windows Server roles & features
- AD DS forest/domain creation
- Integrated DNS
- PowerShell AD module


---

*Lab guide and screenshots from the Level Effect SOC100-1 course (Windows Enterprise Administration). Documentation written by me. Lab passwords are replaced with `<password>`.*

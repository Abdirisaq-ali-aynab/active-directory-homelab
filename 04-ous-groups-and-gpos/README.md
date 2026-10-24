# Lab 04 – OUs, Security Groups & Group Policy (RDP Access)

A common helpdesk ticket: *"I need to remote into my PC."* Rather than fixing it machine-by-machine, I solved it centrally: I created an **RDP Users** security group and a **GPO** that grants that group Remote Desktop rights on every domain workstation, then added `gooduser` to the group.

## Objectives

- Explore where users and computers live (OUs/containers)
- Review the Default Domain Policy GPO
- Create an **RDP Users** security group
- Build a GPO granting RDP rights (User Rights Assignment + Restricted Groups)
- Add `gooduser`, push the policy and verify RDP works

## Environment

| Item | Value |
|---|---|
| Domain | `goodcorp.local` |
| Domain Controller | Windows Server (AD DS + DNS) |
| Workstation | Windows client, domain-joined |
| Admin account | `goodda` (Domain Admin) |
| Standard user | `gooduser` |

## Steps

### Step 1 – Review OUs and object locations

ADAC → `goodcorp.local` → find the joined workstation under **Computers** and `gooduser` under **Users**.

```powershell
Get-ADComputer -Identity "<workstation>" | Select-Object DistinguishedName
Get-ADUser -Identity "gooduser" | Select-Object DistinguishedName
```


<p>
<img src="images/step1-1.png" alt="Step 1 – Review OUs and object locations – screenshot 1" width="49%">
<img src="images/step1-2.png" alt="Step 1 – Review OUs and object locations – screenshot 2" width="49%">
<img src="images/step1-3.png" alt="Step 1 – Review OUs and object locations – screenshot 3" width="49%">
</p>

### Step 2 – Review the Default Domain Policy

**Group Policy Management → Forest → Domains → goodcorp.local → Default Domain Policy → Edit.** Reviewed *Account Policies → Password Policy* and *User Configuration → Administrative Templates → Desktop → Desktop Wallpaper* (how companies enforce a standard wallpaper).

```powershell
Get-GPO -Name "Default Domain Policy"
```


<p>
<img src="images/step2-1.png" alt="Step 2 – Review the Default Domain Policy – screenshot 1" width="49%">
<img src="images/step2-2.png" alt="Step 2 – Review the Default Domain Policy – screenshot 2" width="49%">
<img src="images/step2-3.png" alt="Step 2 – Review the Default Domain Policy – screenshot 3" width="49%">
<img src="images/step2-4.png" alt="Step 2 – Review the Default Domain Policy – screenshot 4" width="49%">
<img src="images/step2-5.png" alt="Step 2 – Review the Default Domain Policy – screenshot 5" width="49%">
<img src="images/step2-6.png" alt="Step 2 – Review the Default Domain Policy – screenshot 6" width="49%">
</p>

### Step 3 – Create the "RDP Users" security group

ADUC → **Users → New → Group** → `RDP Users`, scope **Global**, type **Security**.

```powershell
New-ADGroup -Name "RDP Users" -GroupScope Global -GroupCategory Security -Path "CN=Users,DC=goodcorp,DC=local"
```


<p>
<img src="images/step3-1.png" alt="Step 3 – Create the "RDP Users" security group – screenshot 1" width="49%">
<img src="images/step3-2.png" alt="Step 3 – Create the "RDP Users" security group – screenshot 2" width="49%">
</p>

### Step 4 – Create the "RDP Access for RDP Users" GPO

GPMC → right-click `goodcorp.local` → **Create a GPO in this domain, and Link it here** → *RDP Access for RDP Users* → Edit:

`Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → User Rights Assignment → Allow log on through Remote Desktop Services` → define and add **RDP Users** *and* **Domain Admins**.

> ⚠️ Always include Domain Admins here, or you lock admins out of RDP.


<p>
<img src="images/step4-1.png" alt="Step 4 – Create the "RDP Access for RDP Users" GPO – screenshot 1" width="49%">
<img src="images/step4-2.png" alt="Step 4 – Create the "RDP Access for RDP Users" GPO – screenshot 2" width="49%">
<img src="images/step4-3.png" alt="Step 4 – Create the "RDP Access for RDP Users" GPO – screenshot 3" width="49%">
<img src="images/step4-4.png" alt="Step 4 – Create the "RDP Access for RDP Users" GPO – screenshot 4" width="49%">
<img src="images/step4-5.png" alt="Step 4 – Create the "RDP Access for RDP Users" GPO – screenshot 5" width="49%">
<img src="images/step4-6.png" alt="Step 4 – Create the "RDP Access for RDP Users" GPO – screenshot 6" width="49%">
<img src="images/step4-7.png" alt="Step 4 – Create the "RDP Access for RDP Users" GPO – screenshot 7" width="49%">
<img src="images/step4-8.png" alt="Step 4 – Create the "RDP Access for RDP Users" GPO – screenshot 8" width="49%">
</p>

### Step 4a – Restricted Groups

Same GPO → `Security Settings → Restricted Groups` → add **Remote Desktop Users** → add **RDP Users** as a member. This places the domain group inside each workstation's local *Remote Desktop Users* group.


<p>
<img src="images/step4a-1.png" alt="Step 4a – Restricted Groups – screenshot 1" width="49%">
<img src="images/step4a-2.png" alt="Step 4a – Restricted Groups – screenshot 2" width="49%">
<img src="images/step4a-3.png" alt="Step 4a – Restricted Groups – screenshot 3" width="49%">
</p>

### Step 5 – Add gooduser to RDP Users

ADAC → **RDP Users → Properties → Members → Add** → `gooduser` → Check Names → OK.

```powershell
Add-ADGroupMember -Identity "RDP Users" -Members "gooduser"
```


<p>
<img src="images/step5-1.png" alt="Step 5 – Add gooduser to RDP Users – screenshot 1" width="49%">
<img src="images/step5-2.png" alt="Step 5 – Add gooduser to RDP Users – screenshot 2" width="49%">
<img src="images/step5-3.png" alt="Step 5 – Add gooduser to RDP Users – screenshot 3" width="49%">
</p>

### Step 6 – Push the policy to the workstation

```powershell
Invoke-Command -ComputerName "<workstation>.goodcorp.local" -ScriptBlock { gpupdate /force }
```


<p>
<img src="images/step6-1.png" alt="Step 6 – Push the policy to the workstation – screenshot 1" width="49%">
</p>

### Step 7 – Test RDP as gooduser

RDP in as `goodda`, run `runas /user:goodcorp\gooduser cmd` and `gpupdate /force` to build the user's profile, then connect via RDP as `goodcorp\gooduser`. Login succeeds. I also confirmed `gooduser` **cannot** open PowerShell as administrator (still a standard user).


<p>
<img src="images/step7-1.png" alt="Step 7 – Test RDP as gooduser – screenshot 1" width="49%">
<img src="images/step7-2.png" alt="Step 7 – Test RDP as gooduser – screenshot 2" width="49%">
<img src="images/step7-3.png" alt="Step 7 – Test RDP as gooduser – screenshot 3" width="49%">
<img src="images/step7-4.png" alt="Step 7 – Test RDP as gooduser – screenshot 4" width="49%">
<img src="images/step7-5.png" alt="Step 7 – Test RDP as gooduser – screenshot 5" width="49%">
</p>

## Verification

- `gooduser` can RDP to the workstation
- Workstation local group **Remote Desktop Users** contains `GOODCORP\RDP Users`
- `gooduser` has no admin rights on the workstation

## Troubleshooting Notes

- Still denied after the GPO: run `gpupdate /force` on the workstation and/or reboot it; check with `gpresult /r`.
- Group membership changes need a new logon (new Kerberos ticket) to take effect.
- Use groups, not individual users, in GPOs: access becomes a simple group-membership change.

## Skills Demonstrated

- Organizational Units & AD structure
- Security groups (scope/type)
- Group Policy: User Rights Assignment & Restricted Groups
- `gpupdate` / `gpresult`
- Granting access with least privilege


---

*Lab guide and screenshots from the Level Effect SOC100-1 course (Windows Enterprise Administration). Documentation written by me. Lab passwords are replaced with `<password>`.*

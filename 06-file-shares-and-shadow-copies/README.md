# Lab 06 – File Shares, Firewall Policy & Restoring Deleted Files

*"I accidentally deleted a file from the shared drive. Can you get it back?"* In this lab I built a department share, opened the firewall via Group Policy so users can reach it, enabled **Shadow Copies**, and then restored a file a user had deleted using **Previous Versions**.

## Objectives

- Create and share `C:\SharedData` for Domain Users
- Enable Shadow Copies on the C: drive
- Enforce Network Discovery / File & Printer Sharing through GPO
- Upload and delete a file as a standard user
- Restore the deleted file as an admin

## Environment

| Item | Value |
|---|---|
| Domain | `goodcorp.local` |
| Domain Controller | Windows Server (AD DS + DNS) |
| Workstation | Windows client, domain-joined |
| Admin account | `goodda` (Domain Admin) |
| Standard user | `gooduser` |

## Steps

### Step 1 – Create and share the folder

On the DC: create `C:\SharedData` → **Properties → Sharing → Advanced Sharing → Share this folder** → **Permissions**: add *Domain Users* with Read/Change.

```powershell
New-Item -Path "C:\SharedData" -ItemType Directory
New-SmbShare -Name "SharedData" -Path "C:\SharedData" -FullAccess "goodcorp\Domain Users"
```


<p>
<img src="images/step1-1.png" alt="Step 1 – Create and share the folder – screenshot 1" width="49%">
<img src="images/step1-2.png" alt="Step 1 – Create and share the folder – screenshot 2" width="49%">
<img src="images/step1-3.png" alt="Step 1 – Create and share the folder – screenshot 3" width="49%">
<img src="images/step1-4.png" alt="Step 1 – Create and share the folder – screenshot 4" width="49%">
</p>

### Step 2 – Enable Shadow Copies

Right-click **C:** → **Configure Shadow Copies** → select C: → **Enable** (default size). An initial snapshot is created immediately.

```powershell
vssadmin add shadowstorage /for=C: /on=C: /maxsize=200MB
```


<p>
<img src="images/step2-1.png" alt="Step 2 – Enable Shadow Copies – screenshot 1" width="49%">
<img src="images/step2-2.png" alt="Step 2 – Enable Shadow Copies – screenshot 2" width="49%">
<img src="images/step2-3.png" alt="Step 2 – Enable Shadow Copies – screenshot 3" width="49%">
</p>

### Step 3 – GPO: Enable Network Discovery

New GPO *Enable Network Discovery*:
- `Network → Network Connections → Windows Firewall → Domain Profile → Allow inbound file and printer sharing exception` → **Enabled**
- `Network → Link-Layer Topology Discovery` → enable **Mapper I/O (LLTDIO)** and **Responder (RSPNDR)** drivers

Then `gpupdate` on the workstation. (Opens SMB/NetBIOS: TCP 139/445, UDP 137/138.)


<p>
<img src="images/step3-1.png" alt="Step 3 – GPO: Enable Network Discovery – screenshot 1" width="49%">
<img src="images/step3-2.png" alt="Step 3 – GPO: Enable Network Discovery – screenshot 2" width="49%">
<img src="images/step3-3.png" alt="Step 3 – GPO: Enable Network Discovery – screenshot 3" width="49%">
<img src="images/step3-4.png" alt="Step 3 – GPO: Enable Network Discovery – screenshot 4" width="49%">
</p>

### Step 4 – User uploads a file, admin snapshots, user deletes it

As `gooduser`, browse to `\\<DC>\SharedData` and copy in a file. On the DC, **Shadow Copies → Create Now**. Back as `gooduser`, delete the file.


<p>
<img src="images/step4-1.png" alt="Step 4 – User uploads a file, admin snapshots, user deletes it – screenshot 1" width="49%">
<img src="images/step4-2.png" alt="Step 4 – User uploads a file, admin snapshots, user deletes it – screenshot 2" width="49%">
<img src="images/step4-3.png" alt="Step 4 – User uploads a file, admin snapshots, user deletes it – screenshot 3" width="49%">
<img src="images/step4-4.png" alt="Step 4 – User uploads a file, admin snapshots, user deletes it – screenshot 4" width="49%">
<img src="images/step4-5.png" alt="Step 4 – User uploads a file, admin snapshots, user deletes it – screenshot 5" width="49%">
</p>

### Step 5 – Restore with Previous Versions

As `goodda` on the DC: `C:\SharedData` → **Properties → Previous Versions** → pick the snapshot → **Open** → copy the file → paste it back into `C:\SharedData`. File restored.


<p>
<img src="images/step5-1.png" alt="Step 5 – Restore with Previous Versions – screenshot 1" width="49%">
<img src="images/step5-2.png" alt="Step 5 – Restore with Previous Versions – screenshot 2" width="49%">
<img src="images/step5-3.png" alt="Step 5 – Restore with Previous Versions – screenshot 3" width="49%">
<img src="images/step5-4.png" alt="Step 5 – Restore with Previous Versions – screenshot 4" width="49%">
<img src="images/step5-5.png" alt="Step 5 – Restore with Previous Versions – screenshot 5" width="49%">
</p>

## Verification

- `\\<DC>\SharedData` reachable from the workstation
- Standard user can create and delete files on the share
- Deleted file recovered from a shadow copy

## Troubleshooting Notes

- Can't see the share: check the firewall GPO applied (`gpresult /r`) and test `Test-NetConnection <DC> -Port 445`.
- No *Previous Versions*: no snapshot was taken after the file was created. Schedule shadow copies regularly.
- Turn shadow copies off in the lab when you're done; they consume disk space.
- In production, give users *Modify* rather than *Full Control* on shares.

## Skills Demonstrated

- SMB file shares & share permissions
- Firewall rules via Group Policy
- Volume Shadow Copy / Previous Versions
- File-restore tickets


---

*Lab guide and screenshots from the Level Effect SOC100-1 course (Windows Enterprise Administration). Documentation written by me. Lab passwords are replaced with `<password>`.*

# Lab 03 – Joining a Workstation to the Domain & RDP

A new PC isn't useful to the business until it's on the domain. In this lab I pointed a Windows workstation's DNS at the DC, joined it to `goodcorp.local`, and tested Remote Desktop as both an admin and a standard user.

## Objectives

- Configure workstation DNS to point to the DC
- Join the workstation to `goodcorp.local`
- RDP from the DC to the workstation as a Domain Admin
- Confirm a standard user is denied RDP by default

## Environment

| Item | Value |
|---|---|
| Domain | `goodcorp.local` |
| Domain Controller | Windows Server (AD DS + DNS) |
| Workstation | Windows client, domain-joined |
| Admin account | `goodda` (Domain Admin) |
| Standard user | `gooduser` |

## Steps

### Prep – Point the workstation's DNS at the DC

**Network & Internet settings → Change adapter settings → Ethernet → Properties → IPv4 → Properties**
- Preferred DNS: **the DC's IP address**
- Alternate DNS: lab-provided secondary

> A domain join **will fail** if the workstation can't resolve the domain through the DC's DNS.


<p>
<img src="images/dns-1.png" alt="Prep – Point the workstation's DNS at the DC – screenshot 1" width="49%">
<img src="images/dns-2.png" alt="Prep – Point the workstation's DNS at the DC – screenshot 2" width="49%">
<img src="images/dns-3.png" alt="Prep – Point the workstation's DNS at the DC – screenshot 3" width="49%">
<img src="images/dns-4.png" alt="Prep – Point the workstation's DNS at the DC – screenshot 4" width="49%">
<img src="images/dns-5.png" alt="Prep – Point the workstation's DNS at the DC – screenshot 5" width="49%">
</p>

### Step 1 – Join the domain

**Settings → System → About → Rename this PC (advanced) → Change → Domain:** `goodcorp.local` → enter Domain Admin credentials → *Welcome to the goodcorp.local domain* → restart.

```powershell
Add-Computer -DomainName "goodcorp.local" -Credential goodcorp\goodda -Restart
```


<p>
<img src="images/step1-1.png" alt="Step 1 – Join the domain – screenshot 1" width="49%">
<img src="images/step1-2.png" alt="Step 1 – Join the domain – screenshot 2" width="49%">
<img src="images/step1-3.png" alt="Step 1 – Join the domain – screenshot 3" width="49%">
<img src="images/step1-4.png" alt="Step 1 – Join the domain – screenshot 4" width="49%">
<img src="images/step1-5.png" alt="Step 1 – Join the domain – screenshot 5" width="49%">
<img src="images/step1-6.png" alt="Step 1 – Join the domain – screenshot 6" width="49%">
<img src="images/step1-7.png" alt="Step 1 – Join the domain – screenshot 7" width="49%">
<img src="images/step1-8.png" alt="Step 1 – Join the domain – screenshot 8" width="49%">
</p>

### Step 2 – RDP as the Domain Admin

On the DC open **Remote Desktop Connection** → workstation IP → **More choices → Use a different account** → `goodcorp\goodda`. Accept the certificate prompt, then run `whoami` to confirm the session.

```powershell
mstsc /v:<workstation-ip>
```


<p>
<img src="images/step2-1.png" alt="Step 2 – RDP as the Domain Admin – screenshot 1" width="49%">
<img src="images/step2-2.png" alt="Step 2 – RDP as the Domain Admin – screenshot 2" width="49%">
<img src="images/step2-3.png" alt="Step 2 – RDP as the Domain Admin – screenshot 3" width="49%">
<img src="images/step2-4.png" alt="Step 2 – RDP as the Domain Admin – screenshot 4" width="49%">
<img src="images/step2-5.png" alt="Step 2 – RDP as the Domain Admin – screenshot 5" width="49%">
<img src="images/step2-6.png" alt="Step 2 – RDP as the Domain Admin – screenshot 6" width="49%">
<img src="images/step2-7.png" alt="Step 2 – RDP as the Domain Admin – screenshot 7" width="49%">
</p>

### Step 3 – RDP as a standard user (expected: denied)

**Show Options** → user `goodcorp\gooduser` → connect. The connection is **denied**: standard users don't have the *Allow log on through Remote Desktop Services* right yet. I grant it with Group Policy in [Lab 04](../04-ous-groups-and-gpos/).


<p>
<img src="images/step3-1.png" alt="Step 3 – RDP as a standard user (expected: denied) – screenshot 1" width="49%">
<img src="images/step3-2.png" alt="Step 3 – RDP as a standard user (expected: denied) – screenshot 2" width="49%">
<img src="images/step3-3.png" alt="Step 3 – RDP as a standard user (expected: denied) – screenshot 3" width="49%">
<img src="images/step3-4.png" alt="Step 3 – RDP as a standard user (expected: denied) – screenshot 4" width="49%">
<img src="images/step3-5.png" alt="Step 3 – RDP as a standard user (expected: denied) – screenshot 5" width="49%">
</p>

## Verification

- Workstation shows **Domain: goodcorp.local** in System properties
- Computer object appears in ADUC → **Computers**
- `whoami` in the RDP session returns `goodcorp\goodda`
- `gooduser` RDP attempt is denied (expected)

## Troubleshooting Notes

- *"An Active Directory Domain Controller could not be contacted"*: DNS isn't pointing at the DC. Fix the adapter DNS, then `ipconfig /flushdns`.
- Test name resolution first with `nslookup goodcorp.local`.
- *"The connection was denied because the user account is not authorized"*: user isn't in **Remote Desktop Users** on the target PC.

## Skills Demonstrated

- Client DNS configuration
- Domain join (GUI & PowerShell)
- Remote Desktop support
- Diagnosing access-denied tickets


---

*Lab guide and screenshots from the Level Effect SOC100-1 course (Windows Enterprise Administration). Documentation written by me. Lab passwords are replaced with `<password>`.*

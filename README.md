# Active Directory Home Lab – IT Helpdesk Portfolio

Hands-on Windows Server & Active Directory labs covering the day-to-day work of an IT helpdesk / desktop support technician: building a domain, managing accounts, joining PCs, granting access with Group Policy, restoring files, supporting users remotely, and deploying software.

Each lab folder contains a step-by-step write-up with commands (GUI **and** PowerShell), screenshots, verification and troubleshooting notes.

## Lab Environment

| Component | Details |
|---|---|
| Domain | `goodcorp.local` |
| Domain Controller | Windows Server: AD DS, DNS, file server |
| Client | Windows workstation joined to the domain |
| Accounts | `goodda` (Domain Admin), `gooduser` (standard user) |
| Tools | Server Manager, ADAC, ADUC, Group Policy Management, Event Viewer, PowerShell, Remote Desktop, Chocolatey |

```
            goodcorp.local
 ┌──────────────────────────────┐        RDP / SMB / WinRM       ┌──────────────────────────┐
 │  Domain Controller           │ ◄────────────────────────────► │  Windows Workstation     │
 │  AD DS · DNS · GPO · Shares  │                                │  gooduser (standard)     │
 └──────────────────────────────┘                                └──────────────────────────┘
```

## Labs

| # | Lab | What I did | Helpdesk scenario |
|---|---|---|---|
| 01 | [Building a Domain Controller](./01-domain-controller-setup/) | Installed AD DS, promoted a new forest, verified AD & DNS | New office / new domain setup |
| 02 | [Users, Admins & Groups](./02-users-admins-and-groups/) | Created standard & admin accounts, managed group membership | New-hire account creation |
| 03 | [Domain Join & RDP](./03-domain-join-and-rdp/) | Set client DNS, joined a PC to the domain, tested RDP | New PC onboarding, "can't log in remotely" |
| 04 | [OUs, Groups & Group Policy](./04-ous-groups-and-gpos/) | Built an RDP Users group + GPO to grant remote access | Access request handled centrally |
| 05 | [PowerShell Security Policy](./05-powershell-security-policy/) | GPO to allow only signed scripts + PowerShell logging | Endpoint hardening |
| 06 | [File Shares & Shadow Copies](./06-file-shares-and-shadow-copies/) | Department share, firewall GPO, restored a deleted file | "I deleted a file from the shared drive" |
| 07 | [Remote Management with PowerShell](./07-remote-management-with-powershell/) | WinRM via GPO; remote processes, services, reboots, logs | Remote troubleshooting |
| 08 | [Software Deployment with Chocolatey](./08-software-deployment-with-chocolatey/) | Remotely installed apps with a package manager | Software install requests |

## Skills Demonstrated

- **Active Directory:** domains, forests, OUs, users, security groups, ADAC/ADUC
- **Group Policy:** creating/linking GPOs, User Rights Assignment, Restricted Groups, firewall & PowerShell policies, `gpupdate` / `gpresult`
- **Windows Server:** roles & features, DNS, SMB shares, Volume Shadow Copy
- **Remote support:** Remote Desktop, WinRM / PowerShell Remoting, remote service & process management
- **Automation:** PowerShell AD module, Chocolatey software deployment
- **Troubleshooting:** DNS/domain-join errors, RDP access denied, GPO not applying, file recovery

---

*Labs from Level Effect SOC100-1 (Windows Enterprise Administration). Lab guide and screenshots from the course material; documentation written by me. Passwords are replaced with `<password>`.*

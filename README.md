# Active Directory PAM Lab

A hands-on Active Directory and Privileged Access Management lab focused on identity administration, privileged accounts, least privilege, delegated permissions, service account security, emergency access, and security auditing.

## Project Objectives

* Deploy a Windows Server Domain Controller
* Configure Active Directory Domain Services and DNS
* Create and organize users, computers, groups, and Organizational Units
* Join a Windows 11 workstation to the domain
* Separate standard and privileged user accounts
* Implement least-privilege administrative access
* Delegate limited helpdesk permissions
* Restrict interactive logon for service accounts
* Configure an emergency break-glass account
* Apply security controls using Group Policy
* Audit account management and authentication events
* Document PAM concepts in an Active Directory environment

## Lab Environment

| System       | Role                                                 | Network              |
| ------------ | ---------------------------------------------------- | -------------------- |
| DC01         | Windows Server 2025 Domain Controller and DNS Server | LAB-LAN              |
| CLT-WIN11-01 | Domain-joined Windows 11 workstation                 | LAB-LAN              |
| SRV-Ubuntu01 | Linux server                                         | LAB-LAN              |
| ATK-Kali01   | Isolated attack machine                              | ATTACK-LAN           |
| OPNsense     | Firewall, routing, and network segmentation          | LAB-LAN / ATTACK-LAN |

## Network Configuration

### LAB-LAN

* Network: `10.10.20.0/24`
* Gateway: `10.10.20.1`
* Domain Controller: `10.10.20.10`
* Windows 11 Client: `10.10.20.120`
* Ubuntu Server: `10.10.20.110`

### ATTACK-LAN

* Network: `10.10.50.0/24`
* Gateway: `10.10.50.1`
* Kali Linux: `10.10.50.10`

The ATTACK-LAN network is isolated from LAB-LAN using OPNsense firewall rules.

---

## Active Directory Deployment

Active Directory Domain Services and DNS were installed on `DC01`.

Domain:

```text
cyberlab.local
```

NetBIOS domain:

```text
CYBERLAB
```

The Windows 11 workstation was successfully joined to the domain and configured to use the Domain Controller as its DNS server.

---

## Active Directory Structure

```text
CyberLab
├── Users
├── Workstations
├── Servers
├── Privileged Accounts
├── Service Accounts
└── Groups
```

This structure separates standard users, privileged identities, service accounts, computers, and security groups.

---

## Accounts

### Standard User

```text
valentin.user
```

Used for normal domain activity without administrative privileges.

### Privileged Administrator

```text
adm.valentin
```

A separate administrative identity used instead of granting administrative privileges to the standard user account.

### Helpdesk User

```text
helpdesk.user
```

Used to demonstrate delegated Active Directory administration.

### Service Account

```text
svc_backup
```

A dedicated service identity stored separately from normal user accounts.

### Break-Glass Administrator

```text
breakglass.admin
```

An emergency administrative account that remains disabled during normal operation.

---

## Security Groups

The following Global Security groups were created:

```text
GG_Workstation_Admins
GG_Server_Admins
GG_Helpdesk_Password_Reset
GG_Service_Accounts
GG_PAM_Users
GG_PAM_Admins
```

Administrative permissions are assigned through groups rather than directly to individual user accounts.

---

## Least Privilege Workstation Administration

The privileged account:

```text
CYBERLAB\adm.valentin
```

was added to:

```text
CYBERLAB\GG_Workstation_Admins
```

A Group Policy Object named:

```text
GPO-Workstation-Local-Admins
```

adds `GG_Workstation_Admins` to the local built-in `Administrators` group on workstations in the `Workstations` OU.

The configuration was validated on `CLT-WIN11-01`.

Result:

```text
valentin.user
→ Standard user
→ No local administrative privileges

adm.valentin
→ Member of GG_Workstation_Admins
→ Local administrator on domain workstations
→ Not a Domain Admin
```

This demonstrates least privilege and separation between daily and administrative identities.

---

## Helpdesk Delegation

The group:

```text
GG_Helpdesk_Password_Reset
```

was delegated permission to reset passwords only inside:

```text
CyberLab\Users
```

The account:

```text
helpdesk.user
```

was added to this group.

### Validation

The delegated permissions were tested using Active Directory Users and Computers.

Result:

```text
helpdesk.user
→ Can reset the password of valentin.user
→ Cannot reset the password of adm.valentin
```

Because privileged accounts are stored in a separate OU, the helpdesk account does not receive password-reset permissions over them.

This demonstrates delegated administration without granting Domain Admin privileges.

---

## Service Account Security

A dedicated security group was created:

```text
GG_Service_Accounts
```

The service account:

```text
svc_backup
```

was added to this group.

A Group Policy Object was configured to apply:

```text
Deny log on locally
Deny log on through Remote Desktop Services
```

to members of `GG_Service_Accounts`.

### Validation

An interactive sign-in attempt was made on `CLT-WIN11-01` using `svc_backup`.

The sign-in was blocked with:

```text
The sign-in method you're trying to use isn't allowed.
```

This prevents service accounts from being used as normal interactive user accounts.

---

## Break-Glass Emergency Access

The account:

```text
breakglass.admin
```

was configured as an emergency administrative identity.

It is:

```text
Member of Domain Admins
Disabled during normal operation
```

The account is intended only for emergency recovery scenarios.

Normal state:

```text
breakglass.admin
→ Disabled
```

Emergency workflow:

```text
Enable account
→ Perform emergency administrative operation
→ Audit activity
→ Disable account again
```

---

## Security Auditing

A dedicated Domain Controller auditing policy was created:

```text
GPO-DC-Security-Auditing
```

Advanced Audit Policy was configured for:

```text
User Account Management
Security Group Management
Logon / Logoff
```

Success and failure auditing were enabled where appropriate.

### Break-Glass Audit Test

The `breakglass.admin` account was temporarily enabled and then disabled.

The actions were successfully recorded in the Windows Security log.

Relevant Event IDs:

```text
4722 - A user account was enabled
4725 - A user account was disabled
```

### Authentication Auditing

Successful and failed authentication attempts were also tested.

Relevant security events include:

```text
4624 - Successful logon
4625 - Failed logon
4771 - Kerberos pre-authentication failure
```

This provides visibility into authentication attempts and privileged account activity.

---

## Group Policy Objects

The lab currently uses the following security-focused GPOs:

### GPO-Workstation-Local-Admins

Adds:

```text
CYBERLAB\GG_Workstation_Admins
```

to the local built-in `Administrators` group on domain workstations.

### GPO-Deny-Service-Account-Logon

Prevents members of:

```text
CYBERLAB\GG_Service_Accounts
```

from interactive and Remote Desktop logon.

### GPO-DC-Security-Auditing

Enables advanced security auditing on the Domain Controller.

---

## PAM Concepts Demonstrated

Although a commercial PAM platform is not deployed in this lab, the environment demonstrates several concepts commonly used in privileged access management:

* Separation of standard and privileged identities
* Least privilege
* Role-based administrative access
* Delegated administration
* Privileged account isolation
* Service account management
* Restriction of interactive service-account usage
* Emergency break-glass access
* Privileged activity auditing
* Authentication monitoring
* Security group-based access control

---

## CyberArk Concept Mapping

| CyberArk Component | Purpose                                                     |
| ------------------ | ----------------------------------------------------------- |
| Digital Vault      | Secure storage of privileged credentials                    |
| PVWA               | Web interface for requesting and managing privileged access |
| CPM                | Password management and credential lifecycle                |
| PSM                | Isolation and monitoring of privileged sessions             |
| Safe               | Logical container for privileged accounts                   |
| Platform           | Defines how a managed account is handled                    |

These components are documented conceptually and are not deployed in the current lab.

---

## Security Controls Validated

The following controls have been tested successfully:

```text
Standard user without administrative privileges
        ↓
Dedicated privileged workstation administrator
        ↓
Helpdesk password-reset delegation
        ↓
Privileged account protection from helpdesk
        ↓
Service account interactive-logon restriction
        ↓
Break-glass emergency account
        ↓
Account-management auditing
        ↓
Successful and failed authentication auditing
```

---

## Project Status

* [x] Repository created
* [x] Security-focused `.gitignore` added
* [x] Windows Server 2025 VM deployed
* [x] Static Domain Controller IP configured
* [x] Active Directory Domain Services installed
* [x] DNS Server configured
* [x] `cyberlab.local` domain created
* [x] Organizational Units created
* [x] Users and security groups created
* [x] Windows 11 workstation joined to the domain
* [x] Standard and privileged accounts separated
* [x] Workstation administrator privileges delegated through Group Policy
* [x] Helpdesk password-reset permissions delegated
* [x] Helpdesk least-privilege scenario validated
* [x] Service account security group created
* [x] Interactive logon denied for service accounts
* [x] Service account restriction validated
* [x] Break-glass administrative account configured
* [x] Domain Controller advanced auditing configured
* [x] Break-glass enable/disable events audited
* [x] Successful and failed authentication auditing configured
* [ ] Additional privileged access monitoring
* [ ] Additional attack and detection scenarios
* [ ] Centralized security logging

---

## Related Project

`01-virtual-network`

The networking lab contains the underlying OPNsense firewall, network segmentation, LAB-LAN, and ATTACK-LAN architecture used by this project.

---


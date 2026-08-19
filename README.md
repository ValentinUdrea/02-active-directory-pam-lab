# Active Directory PAM Lab

A hands-on Active Directory and Privileged Access Management lab focused on identity administration, privileged accounts, least privilege, service accounts, delegated administration, security auditing, and PAM concepts.

## Project Objectives

* Deploy a Windows Server Domain Controller
* Configure Active Directory Domain Services and DNS
* Create and organize users, computers, groups, and Organizational Units
* Join a Windows 11 workstation to the domain
* Join a Windows Server member server to the domain
* Separate standard, workstation administrator, and server administrator accounts
* Configure delegated administrative permissions
* Restrict interactive use of service accounts
* Create and protect an emergency break-glass account
* Apply security controls through Group Policy
* Audit privileged account and authentication activity
* Document PAM concepts in an Active Directory environment

## Environment

| System             | Role                                        | Network              |
| ------------------ | ------------------------------------------- | -------------------- |
| DC01-WindowsServer | Domain Controller and DNS Server            | LAB-LAN              |
| CLT-WIN11-01       | Domain-joined workstation                   | LAB-LAN              |
| SRV-WIN01          | Windows Server 2025 Member Server           | LAB-LAN              |
| SRV-Ubuntu01       | Linux server                                | LAB-LAN              |
| ATK-Kali01         | Isolated attack machine                     | ATTACK-LAN           |
| OPNsense           | Firewall, routing, and network segmentation | LAB-LAN / ATTACK-LAN |

## Network Configuration

### LAB-LAN

* Network: `10.10.20.0/24`
* Gateway: `10.10.20.1`
* Domain Controller: `10.10.20.10`
* Windows Server Member Server: `10.10.20.20`
* Ubuntu Server: `10.10.20.110`
* Windows 11 Client: `10.10.20.120`

### ATTACK-LAN

* Network: `10.10.50.0/24`
* Gateway: `10.10.50.1`
* Kali Linux: `10.10.50.10`

The ATTACK-LAN network is isolated from LAB-LAN using OPNsense firewall rules.

## Active Directory Design

Domain:

```text
cyberlab.local
```

Organizational Unit structure:

```text
CyberLab
├── Users
├── Workstations
├── Servers
├── Privileged Accounts
├── Service Accounts
└── Groups
```

## Accounts

### Standard User Accounts

```text
valentin.user
helpdesk.user
```

### Privileged Accounts

```text
adm.valentin
srv.adm.valentin
breakglass.admin
```

### Service Accounts

```text
svc_backup
```

## Security Groups

```text
GG_Server_Admins
GG_Workstation_Admins
GG_Helpdesk_Password_Reset
GG_Service_Accounts
GG_PAM_Users
GG_PAM_Admins
```

Permissions are assigned through security groups instead of being granted directly to individual accounts.

## Account Separation

The lab separates standard accounts from privileged administrative accounts.

### Standard User

```text
valentin.user
```

Used for normal domain activity without administrative permissions.

### Workstation Administrator

```text
adm.valentin
```

Member of:

```text
GG_Workstation_Admins
```

A Group Policy adds `GG_Workstation_Admins` to the local built-in `Administrators` group on computers inside the `Workstations` OU.

This gives `adm.valentin` administrative rights on domain workstations without granting Domain Admin privileges.

### Server Administrator

```text
srv.adm.valentin
```

Member of:

```text
GG_Server_Admins
```

A separate Group Policy adds `GG_Server_Admins` to the local built-in `Administrators` group on member servers inside the `Servers` OU.

This separates workstation administration from server administration.

The resulting model is:

```text
valentin.user
→ Standard user

adm.valentin
→ Workstation administrator

srv.adm.valentin
→ Server administrator

breakglass.admin
→ Emergency Domain Administrator
```

This design supports the principles of:

* Least privilege
* Separation of duties
* Reduced standing privileges
* Privileged account accountability

## Helpdesk Delegation

The helpdesk account:

```text
helpdesk.user
```

is a member of:

```text
GG_Helpdesk_Password_Reset
```

Password reset permissions were delegated only to the `CyberLab\Users` OU.

The configuration was validated successfully:

```text
helpdesk.user
→ Can reset the password of valentin.user
→ Cannot reset the password of adm.valentin
```

This demonstrates delegated administration without granting Domain Admin privileges.

## Service Account Security

The service account:

```text
svc_backup
```

is a member of:

```text
GG_Service_Accounts
```

A Group Policy was configured to apply:

```text
Deny log on locally
Deny log on through Remote Desktop Services
```

to members of `GG_Service_Accounts`.

An interactive login attempt using `svc_backup` was successfully blocked on the Windows 11 workstation.

This prevents service accounts from being used as normal interactive user accounts.

## Break-Glass Access

The emergency account:

```text
breakglass.admin
```

is a member of:

```text
Domain Admins
```

The account remains disabled during normal operation and is intended only for emergency recovery scenarios.

Expected workflow:

```text
Account disabled
        ↓
Emergency occurs
        ↓
Account enabled
        ↓
Administrative recovery performed
        ↓
Activity audited
        ↓
Account disabled again
```

Enable and disable events for the account were successfully recorded in the Windows Security log.

## Group Policy Configuration

The following security-focused Group Policy Objects were created:

```text
GPO-Workstation-Local-Admins
GPO-Server-Local-Admins
GPO-Deny-Service-Account-Logon
GPO-DC-Security-Auditing
GPO-Server-Security-Auditing
```

### GPO-Workstation-Local-Admins

Adds:

```text
CYBERLAB\GG_Workstation_Admins
```

to the built-in local `Administrators` group on workstations.

### GPO-Server-Local-Admins

Adds:

```text
CYBERLAB\GG_Server_Admins
```

to the built-in local `Administrators` group on member servers.

### GPO-Deny-Service-Account-Logon

Prevents members of:

```text
CYBERLAB\GG_Service_Accounts
```

from interactive and Remote Desktop logon.

### GPO-DC-Security-Auditing

Enables security auditing on the Domain Controller.

### GPO-Server-Security-Auditing

Enables logon and process auditing on member servers.

## Security Auditing

Advanced Windows auditing was configured to monitor account management, privileged group changes, authentication, and administrative activity.

### Domain Controller Events

The following events were tested:

```text
4722 - User account enabled
4725 - User account disabled
4728 - Member added to a Global Security Group
4729 - Member removed from a Global Security Group
4624 - Successful logon
4625 - Failed logon
```

The `breakglass.admin` account was enabled and disabled to validate account-management auditing.

Membership changes were also performed on:

```text
GG_PAM_Admins
```

to confirm that privileged group modifications are recorded.

### Member Server Auditing

Security auditing was configured on `SRV-WIN01`.

Relevant events include:

```text
4624 - Successful logon
4625 - Failed logon
4672 - Special privileges assigned to a new logon
4688 - New process created
```

The configuration was validated using the dedicated server administrator account:

```text
srv.adm.valentin
```

## PAM and CyberArk Concepts

The lab demonstrates or supports the following PAM concepts:

* Least privilege
* Separation of duties
* Privileged account isolation
* Delegated administration
* Role-based administrative access
* Service account management
* Emergency break-glass access
* Privileged activity auditing
* Authentication monitoring
* Privileged session monitoring concepts

## CyberArk Component Mapping

| CyberArk Component | Purpose                                                    |
| ------------------ | ---------------------------------------------------------- |
| Digital Vault      | Securely stores privileged credentials                     |
| PVWA               | Web interface used to request and manage privileged access |
| CPM                | Manages privileged account credentials                     |
| PSM                | Isolates, controls, and monitors privileged sessions       |
| Safe               | Logical container used to organize privileged accounts     |
| Platform           | Defines how a specific type of account is managed          |

These components are documented conceptually and are not deployed as part of the current lab.

## Lab Scenarios

### Scenario 1: Standard User Access

A standard user logs into the Windows 11 workstation without local administrative rights.

Result:

```text
Administrative operations require privileged credentials.
```

### Scenario 2: Workstation Administrative Account

Administrative operations on the workstation are performed using:

```text
adm.valentin
```

Result:

```text
Workstation administrative access is separated from normal user activity.
```

### Scenario 3: Helpdesk Delegation

The helpdesk account can reset standard user passwords without becoming a Domain Administrator.

Result:

```text
helpdesk.user can reset standard user passwords but cannot reset privileged accounts.
```

### Scenario 4: Service Account Restriction

The service account:

```text
svc_backup
```

is prevented from logging on interactively.

Result:

```text
The sign-in method you're trying to use isn't allowed.
```

### Scenario 5: Break-Glass Access

An emergency Domain Administrator account is kept disabled during normal operations.

Result:

```text
Account activation and deactivation are recorded in the Windows Security log.
```

### Scenario 6: Server Administrator Separation

A Windows Server 2025 member server was joined to the domain:

```text
SRV-WIN01
```

The account:

```text
srv.adm.valentin
```

receives server administration permissions through:

```text
GG_Server_Admins
```

Result:

```text
Server administration is separated from workstation and domain administration.
```

### Scenario 7: Privileged Group Monitoring

A privileged account was temporarily added to and removed from:

```text
GG_PAM_Admins
```

Result:

```text
Event 4728 recorded the addition.
Event 4729 recorded the removal.
```

### Scenario 8: Privileged Server Activity Auditing

Administrative activity was generated on `SRV-WIN01`.

Result:

```text
Successful logons, privileged logons, and process creation events were recorded.
```

## Screenshots

### Active Directory Structure

![Active Directory Structure](screenshots/01-ad-structure.png)

### Group Policy Configuration

![Group Policy Configuration](screenshots/02-group-policy.png)

### Security Auditing

![Security Auditing](screenshots/03-security-auditing.png)

### Server Administration

![Server Administration](screenshots/04-server-admin.png)

## Project Status

* [x] Repository created
* [x] Security-focused `.gitignore` added
* [x] Initial project documentation created
* [x] Windows Server 2025 Domain Controller deployed
* [x] Static IP configured
* [x] Active Directory Domain Services installed
* [x] DNS Server configured
* [x] `cyberlab.local` domain created
* [x] Organizational Units created
* [x] Users and security groups created
* [x] Windows 11 joined to the domain
* [x] Windows Server member server joined to the domain
* [x] Standard and privileged accounts separated
* [x] Workstation administrator role configured
* [x] Server administrator role configured
* [x] Helpdesk password reset permissions delegated
* [x] Service account interactive logon restricted
* [x] Break-glass account configured
* [x] Domain Controller security auditing configured
* [x] Privileged group changes audited
* [x] Member server privileged activity auditing configured
* [ ] Centralized security logging / SIEM integration
* [ ] Additional detection scenarios

## Related Project

Virtual Network Security Lab:

https://github.com/ValentinUdrea/01-virtual-network


---


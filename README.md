# Active Directory PAM Lab

A hands-on Active Directory and Privileged Access Management lab focused on identity administration, privileged accounts, least privilege, service accounts, and CyberArk PAM concepts.

## Project Objectives

- Deploy a Windows Server Domain Controller
- Configure Active Directory Domain Services and DNS
- Create and organize users, computers, groups, and Organizational Units
- Join a Windows 11 workstation to the domain
- Separate standard and privileged user accounts
- Configure delegated administrative permissions
- Create service accounts and emergency access accounts
- Simulate the privileged account lifecycle used in PAM solutions
- Practice CyberArk concepts for technical interviews

## Planned Environment

| System | Role | Network |
|---|---|---|
| DC01-WindowsServer | Domain Controller and DNS Server | LAB-LAN |
| CLT-Windows11-01 | Domain-joined workstation | LAB-LAN |
| SRV-Ubuntu01 | Managed Linux server | LAB-LAN |
| ATK-Kali01 | Isolated attack machine | ATTACK-LAN |
| OPNsense | Firewall, routing, and network segmentation | LAB-LAN / ATTACK-LAN |

## Network Configuration

### LAB-LAN

- Network: `10.10.20.0/24`
- Gateway: `10.10.20.1`
- Domain Controller: `10.10.20.10`
- Windows 11 Client: `10.10.20.120`
- Ubuntu Server: `10.10.20.110`

### ATTACK-LAN

- Network: `10.10.50.0/24`
- Gateway: `10.10.50.1`
- Kali Linux: `10.10.50.10`

The ATTACK-LAN network is isolated from LAB-LAN using OPNsense firewall rules.

## Active Directory Design

Domain:

```text
cyberlab.local
```

Planned Organizational Unit structure:

```text
CyberLab
├── Users
├── Workstations
├── Servers
├── Privileged Accounts
├── Service Accounts
└── Groups
```

## Planned Accounts

### Standard User Accounts

```text
valentin.user
helpdesk.user
```

### Privileged Accounts

```text
adm.valentin
breakglass.admin
```

### Service Accounts

```text
svc_backup
svc_web
```

## Planned Security Groups

```text
Server Administrators
Workstation Administrators
Helpdesk Password Reset
CyberArk PAM Users
CyberArk PAM Administrators
```

## Account Separation

The lab separates standard accounts from privileged administrative accounts.

Example:

```text
valentin.user
```

This account will be used for normal daily activity and will not have administrative permissions.

```text
adm.valentin
```

This account will be used only for approved administrative operations.

This design supports the principles of:

- Least privilege
- Separation of duties
- Reduced standing privileges
- Privileged account accountability

## PAM and CyberArk Concepts

The lab will demonstrate and document the following concepts:

- Privileged account discovery
- Privileged account onboarding
- Safes and access permissions
- Password verification
- Password rotation
- Password reconciliation
- Least privilege
- Separation of duties
- Privileged session management
- Session auditing and monitoring
- Service account management
- Emergency break-glass access

## CyberArk Component Mapping

| CyberArk Component | Purpose |
|---|---|
| Digital Vault | Securely stores privileged credentials |
| PVWA | Web interface used to request and manage privileged access |
| CPM | Verifies, changes, and reconciles passwords |
| PSM | Isolates, controls, and monitors privileged sessions |
| Safe | Logical container used to organize privileged accounts |
| Platform | Defines how a specific type of account is managed |

## Password Management Concepts

### Verify

Checks whether the password stored in the Vault works on the target system.

### Change

Changes the password on the target system using the currently known password.

### Reconcile

Resets the password using a separate reconciliation account when the stored password no longer matches the target account password.

## Planned PAM Workflow

```text
1. Discover a privileged account
2. Onboard the account into a Safe
3. Assign an account management Platform
4. Verify the stored password
5. Rotate the account password
6. Request privileged access
7. Start a controlled administrative session
8. Audit the privileged activity
9. Reconcile the password if synchronization is lost
```

## Planned Lab Scenarios

### Scenario 1: Standard User Access

A standard user logs into the Windows 11 workstation and attempts to perform an administrative task.

Expected result:

```text
Access denied without privileged credentials.
```

### Scenario 2: Separate Administrative Account

The user performs an administrative task using the dedicated account:

```text
adm.valentin
```

Expected result:

```text
Administrative access is separated from normal user activity.
```

### Scenario 3: Helpdesk Delegation

The helpdesk account receives permission to reset standard user passwords without becoming a Domain Administrator.

Expected result:

```text
The helpdesk user can perform only the delegated operation.
```

### Scenario 4: Service Account Management

A service account is created for an application or scheduled task.

Expected result:

```text
The service uses a dedicated account instead of a personal administrator account.
```

### Scenario 5: Password Rotation

The password of a privileged account is changed and documented as a PAM password rotation scenario.

Expected result:

```text
The privileged password is rotated without affecting unrelated accounts.
```

### Scenario 6: Password Reconciliation

The password of a managed account is manually changed outside the PAM process.

Expected result:

```text
The stored password becomes out of sync and must be reconciled.
```

### Scenario 7: Break-Glass Access

An emergency account is created and reserved for recovery situations.

Expected result:

```text
The account is not used for daily administration and all usage must be audited.
```

## Project Status

- [x] Repository created
- [x] Security-focused `.gitignore` added
- [x] Initial project documentation created
- [ ] Windows Server ISO downloaded
- [ ] Windows Server VM created
- [ ] Static IP configured
- [ ] Active Directory Domain Services installed
- [ ] DNS Server configured
- [ ] `cyberlab.local` domain created
- [ ] Organizational Units created
- [ ] Users and groups created
- [ ] Windows 11 joined to the domain
- [ ] Least privilege scenarios configured
- [ ] Service account scenarios configured
- [ ] PAM lifecycle scenarios documented
- [ ] CyberArk interview notes completed

## Related Project

Virtual Network Security Lab:

https://github.com/ValentinUdrea/01-virtual-network

## Disclaimer

This project is an educational home lab.

It simulates Active Directory and Privileged Access Management concepts for learning and interview preparation. It is not a production deployment of CyberArk and does not contain real credentials or sensitive organizational data.

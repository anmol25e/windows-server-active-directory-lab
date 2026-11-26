# Lab Setup Guide — NovaTech Active Directory Environment

A step-by-step build guide for the **NovaTech** (`novatech.local`) on-premises Active Directory lab running in Oracle VirtualBox.

## Environment Overview

| Component              | Hostname / Name | OS / Role                                         | Specs & Network                                     |
| ---------------------- | --------------- | ------------------------------------------------- | --------------------------------------------------- |
| **Virtual Network**    | `NatNetwork`    | VirtualBox NAT Network                            | `10.0.2.0/24` (DHCP Enabled)                        |
| **Domain Controller**  | `DC01`          | Windows Server 2022 Standard (Desktop Experience) | 4 GB RAM, 2 vCPUs, 50 GB VDI, Static IP `10.0.2.10` |
| **Client Workstation** | `CLIENT01`      | Windows 11 Enterprise Evaluation                  | 4 GB RAM, 2 vCPUs, 50 GB VDI, DNS `10.0.2.10`       |

---

## Phase 1: Virtual Network & Windows Server 2022 Installation

1. Created a VirtualBox **NAT Network** (`NatNetwork`, `10.0.2.0/24`) with DHCP enabled so both VMs share an isolated LAN while retaining internet access.
2. Created VM `DC01-WinServer2022` (4 GB RAM, 2 vCPUs, 50 GB dynamic VDI) and attached Adapter 1 to `NatNetwork`.
3. Installed **Windows Server 2022 Standard Evaluation (Desktop Experience)** with **Skip Unattended Installation** checked.

---

## Phase 2: Static IP, Hostname & AD DS Promotion

1. Configured a static IPv4 address on `DC01`:

   | Setting             | Value           |
   | ------------------- | --------------- |
   | **IP Address**      | `10.0.2.10`     |
   | **Subnet Mask**     | `255.255.255.0` |
   | **Default Gateway** | `10.0.2.1`      |
   | **Preferred DNS**   | `127.0.0.1`     |
   | **Alternate DNS**   | `8.8.8.8`       |

2. Renamed the server to `DC01` and rebooted.
3. Installed the **Active Directory Domain Services (AD DS)** role via Server Manager.
4. Promoted `DC01` to a Domain Controller in a new forest: **`novatech.local`** (NetBIOS: `NOVATECH`).

> **Verification:** Confirmed `DC01` booted to the `NOVATECH\Administrator` logon prompt and Server Manager displayed `novatech.local` with IP `10.0.2.10`.

---

## Phase 3: Organizational Units, Users & Security Groups

1. Opened **Active Directory Users and Computers (ADUC)** and created the top-level `NovaTech` OU with five child OUs: `IT`, `HR`, `Finance`, `Sales`, and `Security Groups`.
2. Provisioned 12 domain user accounts (3 per department) and assigned them to their corresponding Global Security Groups:

   | Department OU | Security Group      | User Logon Names                |
   | ------------- | ------------------- | ------------------------------- |
   | **IT**        | `SG-IT-Access`      | `acarter`, `mpatel`, `lbrooks`  |
   | **HR**        | `SG-HR-Access`      | `sjenkins`, `dkim`, `cbennett`  |
   | **Finance**   | `SG-Finance-Access` | `mreed`, `pnair`, `eross`       |
   | **Sales**     | `SG-Sales-Access`   | `jvance`, `smorales`, `nfoster` |

---

## Phase 4: Windows 11 Client & Domain Join

1. Created VM `CLIENT01-Win11` (4 GB RAM, 2 vCPUs, 50 GB dynamic VDI) and attached Adapter 1 to `NatNetwork`.
2. Installed **Windows 11 Enterprise Evaluation** using a local administrator account (`LocalAdmin`).
3. Set `CLIENT01`'s **Preferred DNS server** to `10.0.2.10` and verified connectivity to the Domain Controller using `ping 10.0.2.10`.
4. Renamed the workstation to `CLIENT01` and joined it to `novatech.local` using `NOVATECH\Administrator`.

> **Verification:** Rebooted `CLIENT01` and successfully signed in with domain user `acarter`.

---

## Phase 5: Shared Folders & Group Policy

### Shared Folders (NTFS & RBAC)

1. Created `C:\CompanyShares\IT` and `C:\CompanyShares\HR` on `DC01`.
2. Enabled **Advanced Sharing** (`Everyone: Full Control`) and locked down **NTFS Security** permissions:
   - Disabled inheritance and converted inherited permissions to explicit.
   - Removed default `NOVATECH\Users` access.
   - Granted **Modify** permissions exclusively to `SG-IT-Access` (`IT` folder) and `SG-HR-Access` (`HR` folder).

### Group Policy Management

1. Opened **Group Policy Management** and created `Sales - Block Control Panel`, linked to the `Sales` OU.
2. Configured the policy under **User Configuration → Policies → Administrative Templates → Control Panel**:
   - Enabled **Prohibit access to Control Panel and PC settings**.

> **Verification:** Confirmed `acarter` could read/write to `\\DC01\IT` (and was denied access to `\\DC01\HR`), and confirmed `jvance` received a restriction popup when attempting to open Control Panel.

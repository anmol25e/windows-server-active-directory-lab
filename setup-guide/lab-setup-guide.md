### 2. `setup-guide/lab-setup-guide.md`

```markdown
# Lab Setup Guide — NovaTech Active Directory Environment

## Phase 1: Virtual Network & Windows Server 2022 Installation

1. Created a VirtualBox **NAT Network** (`NatNetwork`, `10.0.2.0/24`) with DHCP enabled so both VMs share an isolated LAN while retaining internet access.
2. Created VM `DC01-WinServer2022` (4 GB RAM, 2 vCPUs, 50 GB dynamic VDI) attached to `NatNetwork`.
3. Installed **Windows Server 2022 Standard Evaluation (Desktop Experience)** using **Skip Unattended Installation**.

## Phase 2: Static IP, Hostname & AD DS Promotion

1. Configured a static IPv4 address on `DC01`:
   - **IP Address:** `10.0.2.10`
   - **Subnet Mask:** `255.255.255.0`
   - **Default Gateway:** `10.0.2.1`
   - **Preferred DNS:** `127.0.0.1` | **Alternate DNS:** `8.8.8.8`
2. Renamed the server to `DC01` and rebooted.
3. Installed the **Active Directory Domain Services (AD DS)** role via Server Manager.
4. Promoted `DC01` to a Domain Controller in a new forest: `novatech.local` (NetBIOS: `NOVATECH`).

## Phase 3: Organizational Units, Users & Security Groups

1. Opened **Active Directory Users and Computers (ADUC)** and created the `NovaTech` OU with five child OUs: `IT`, `HR`, `Finance`, `Sales`, and `Security Groups`.
2. Created 12 domain user accounts (3 per department):
   - **IT:** `acarter`, `mpatel`, `lbrooks`
   - **HR:** `sjenkins`, `dkim`, `cbennett`
   - **Finance:** `mreed`, `pnair`, `eross`
   - **Sales:** `jvance`, `smorales`, `nfoster`
3. Created four Global Security Groups (`SG-IT-Access`, `SG-HR-Access`, `SG-Finance-Access`, `SG-Sales-Access`) in the `Security Groups` OU and added each department's users.

## Phase 4: Windows 11 Client & Domain Join

1. Created VM `CLIENT01-Win11` (4 GB RAM, 2 vCPUs, 50 GB dynamic VDI) attached to `NatNetwork`.
2. Installed **Windows 11 Enterprise Evaluation** with a local admin account (`LocalAdmin`).
3. Set `CLIENT01`'s Preferred DNS server to `10.0.2.10` and verified connectivity via `ping 10.0.2.10`.
4. Renamed the workstation to `CLIENT01`, joined it to `novatech.local` using `NOVATECH\Administrator`, and verified domain logon as `acarter`.

## Phase 5: Shared Folders & Group Policy

1. Created `C:\CompanyShares\IT` and `C:\CompanyShares\HR` on `DC01`.
2. Enabled Advanced Sharing (`Everyone: Full Control`) and locked down NTFS Security permissions by disabling inheritance, removing `NOVATECH\Users`, and granting **Modify** access exclusively to `SG-IT-Access` and `SG-HR-Access` respectively.
3. Opened **Group Policy Management**, created `Sales - Block Control Panel`, linked it to the `Sales` OU, and enabled **User Configuration > Policies > Administrative Templates > Control Panel > Prohibit access to Control Panel and PC settings**.
4. Verified that `acarter` could access `\\DC01\IT` (and was blocked from `\\DC01\HR`) and that `jvance` was blocked from opening Control Panel.
```

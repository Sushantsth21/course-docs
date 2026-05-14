# CIT 271: Windows Administration — Lab Portfolio

**Course:** CIT 271 – Windows Administration  
**Environment:** COIVcenter Virtual Lab (Windows Server 2022, Windows 10, Windows 11)  
**Domain:** cit271.com  

---

## Overview

This portfolio documents all 10 labs completed for CIT 271: Windows Administration. Each lab was performed in a virtualized environment using Windows Server 2022 as a domain controller and two client VMs (Windows 10 and Windows 11). Labs progress from initial OS installation through advanced security hardening.

---

## Lab Snapshot Chain

Each lab required a server snapshot before starting. The snapshot chain follows this pattern (older ones deleted progressively):

| Lab | Snapshot Name | Notes |
|-----|--------------|-------|
| Lab 01 | *(no snapshot required)* | First install |
| Lab 02 | Lab2 | Keep through Lab 3 |
| Lab 03 | Lab3 | Keep through Lab 4 |
| Lab 04 | Lab4 | Delete Lab2 |
| Lab 05 | Lab5 | Delete Lab3 |
| Lab 06 | Lab6 | Delete Lab4 |
| Lab 07 | Lab7 | Delete Lab5 |
| Lab 08 | Lab8 | Delete Lab6 |
| Lab 09 | Lab9 | Delete Lab7 |
| Lab 10 | Lab10 (Server + Win11) | Delete Lab8 |

---

## Lab 01 — Windows Installation: Server and Clients

**Objective:** Install Windows Server 2022, Windows 10, and Windows 11 on virtual machines and perform initial network configuration.

### Key Tasks
- Installed **Windows Server 2022 Standard Evaluation (Desktop Experience)** on an 80 GB disk
- Set Administrator password: `Student271!`
- Configured static IPv4 networking on the server:
  - IP Address: `192.168.1.2`
  - Subnet Mask: `255.255.255.0`
  - Default Gateway: `192.168.1.1`
  - Preferred DNS: `192.168.1.2`
  - Alternate DNS: `10.11.0.51`
- Installed Windows 10 and Windows 11 on client VMs
- All VMs connected through a PfSense router VM (must be booted first)

### Notes
- PfSense VM must always be started before any Windows VMs
- Display resolution set to 1280×720 for improved visibility
- Clients did not have Internet access until the domain controller was configured in Lab 02

---

## Lab 02 — Domain Controller Installation

**Objective:** Promote Windows Server 2022 to a domain controller by installing Active Directory, DHCP, and DNS roles.

### Key Tasks
- Took mandatory server snapshot named **Lab2**
- Installed the following server roles:
  - Active Directory Domain Services (AD DS)
  - DHCP Server
  - DNS Server
- Promoted the server to a domain controller
  - New forest with root domain: `cit271.com`
  - DSRM password: `Student271!`
- Server rebooted and joined the `cit271.com` domain
- Verified login prompt shows "Sign in to: CIT271"

### Notes
- Reboot after promotion can take up to 8 minutes
- Login after reboot uses the "Other user" option with the domain Administrator account

---

## Lab 03 — Domain Configuration

**Objective:** Join client machines to the domain and build the Active Directory directory structure with OUs, users, and groups.

### Key Tasks
- Took mandatory server snapshot named **Lab3**
- Renamed client machines before joining domain:
  - Windows 10: `win10client`
  - Windows 11: `win11client`
- Joined both clients to `cit271.com` domain using Administrator credentials
- Created Organizational Unit (OU) structure in Active Directory Users and Computers
- Created department OUs, users, and security groups

### Evidence Provided
- Screenshot of AD file structure and group memberships
- Screenshot of client computers and group membership
- PowerShell output confirming commands

---

## Lab 04 — Group Policy Introduction

**Objective:** Use Group Policy Objects (GPOs) to enforce security and user settings across domain machines.

### Key Tasks
- Took mandatory server snapshot named **Lab4**
- Edited **Default Domain Policy** with Computer Configuration settings:
  - Required Ctrl+Alt+Delete at login (`Interactive Logon: Do not require CTRL ALT DEL` → Disabled)
  - Disabled the ability to shut down without logging in
- Configured **Loopback Processing** GPO policy:
  - Path: `Computer Configuration > Policies > Administrative Templates > System > Group Policy > Configure user Group Policy loopback processing mode`
- Configured **Desktop Wallpaper** via GPO:
  - Path: `User Configuration > Policies > Administrative Templates > Desktop > Desktop > Desktop Wallpaper`
- Verified settings applied on both Windows 10 and Windows 11 clients

### GPO Paths Reference
| Setting | Path |
|---------|------|
| Ctrl+Alt+Delete requirement | `Computer Config > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Interactive Logon: Do not require CTRL ALT DEL` |
| Loopback processing | `Computer Config > Policies > Admin Templates > System > Group Policy > Configure user Group Policy loopback processing mode` |
| Wallpaper | `User Config > Policies > Admin Templates > Desktop > Desktop > Desktop Wallpaper` |

---

## Lab 05 — PowerShell Commands Introduction

**Objective:** Learn PowerShell fundamentals and use cmdlets to manage Active Directory and the filesystem.

### Key Tasks
- Took mandatory server snapshot named **Lab5**
- Used `ipconfig` and `Get-NetAdapter` to retrieve network info
- Stored cmdlet output in variables and accessed object properties (`$a.MacAddress`)
- Created a Facilities department OU, users, and group via PowerShell

### Key Commands Used

```powershell
# Create Facilities OU
New-ADOrganizationalUnit -Name "Facilities" -Path "OU=Departments,DC=cit271,DC=com"

# Create Facilities Users group
New-ADGroup -Name "Facilities Users" -SamAccountName "Facilities Users" `
  -GroupCategory Security -GroupScope Global -DisplayName "Facilities Users" `
  -Path "OU=Groups,DC=cit271,DC=com"

# Create users
New-ADUser -Name "Mario McMario" -GivenName "Mario" -Surname "McMario" `
  -SamAccountName "mcmario1" -UserPrincipalName "mcmario1@cit271.com" `
  -Path "OU=Facilities,OU=Departments,DC=cit271,DC=com" `
  -AccountPassword (ConvertTo-SecureString -AsPlainText "Student271!" -Force) -Enabled $true

New-ADUser -Name "Mowz McMowz" -GivenName "Mowz" -Surname "McMowz" `
  -SamAccountName "mcmowz1" -UserPrincipalName "mcmowz1@cit271.com" `
  -Path "OU=Facilities,OU=Departments,DC=cit271,DC=com" `
  -AccountPassword (ConvertTo-SecureString -AsPlainText "Student271!" -Force) -Enabled $true

# Add users to group
Add-ADGroupMember -Identity "Facilities Users" -Members mcmario1,mcmowz1

# Export running services to CSV
Get-Service | Where-Object {$_.Status -eq "Running"} | Sort-Object -Property DisplayName `
  | Select-Object -Property Status, DisplayName, StartType `
  | Export-Csv -Path C:\services.csv -NoTypeInformation
```

---

## Lab 06 — PowerShell Scripting Introduction

**Objective:** Write a reusable PowerShell script that automates AD object creation (OUs, groups, and users).

### Key Tasks
- Took mandatory server snapshot named **Lab6**
- Created script file: `lab6script.ps1` saved to `Desktop\Scripts\`
- Script uses functions and user input (`Read-Host`) to create:
  - A new department OU under `Departments`
  - Two groups (`<OU> Users` and `<OU> Computers`) in the `Groups` OU
  - Optionally, a new AD user with first name, last name, username, and password

### Script

```powershell
function Create-NewOU {
    param($ouName)
    New-ADOrganizationalUnit -Name $ouName -Path "OU=Departments,DC=cit271,DC=com"
    Write-Host "Created new OU: $ouName"
}

function Create-Groups {
    param($ouName)
    New-ADGroup -Name "$ouName Users" -GroupScope Global -Path "OU=Groups,DC=cit271,DC=com"
    New-ADGroup -Name "$ouName Computers" -GroupScope Global -Path "OU=Groups,DC=cit271,DC=com"
    Write-Host "Created groups: $ouName Users and $ouName Computers"
}

function Create-NewUser {
    param($ouName)
    $firstName = Read-Host "Enter first name"
    $lastName  = Read-Host "Enter last name"
    $username  = Read-Host "Enter username"
    $password  = Read-Host "Enter password" -AsSecureString
    $displayName       = "$firstName $lastName"
    $userPrincipalName = "$username@cit271.com"

    New-ADUser -Name $displayName `
               -GivenName $firstName -Surname $lastName `
               -SamAccountName $username -UserPrincipalName $userPrincipalName `
               -DisplayName $displayName `
               -Path "OU=$ouName,OU=Departments,DC=cit271,DC=com" `
               -AccountPassword $password -Enabled $true

    Add-ADGroupMember -Identity "$ouName Users" -Members $username
    Write-Host "Created new user: $displayName and added to $ouName Users group"
}

$ouName = Read-Host "Enter the name for the new department OU"
Create-NewOU -ouName $ouName
Create-Groups -ouName $ouName

while ($true) {
    $createUser = Read-Host "Do you want to create a new user in the $ouName OU? (yes/no)"
    if ($createUser -eq "yes") {
        Create-NewUser -ouName $ouName
    } elseif ($createUser -eq "no") {
        break
    } else {
        Write-Host "Invalid input. Please enter 'yes' or 'no'."
    }
}
```

---

## Lab 07 — Print Server Configuration

**Objective:** Install the Print and Document Services role and deploy a shared PDF printer (BullZip) to Sales users via a Group Policy startup script.

### Key Tasks
- Took mandatory server snapshot named **Lab7**
- Installed **Print and Document Services** role (Print Server role service)
- Installed **BullZip PDF Printer** on the server
- Tested printer with a test page via Print Management
- Configured printer for client deployment:
  - Changed driver to: `Microsoft PS Class Driver`
  - Renamed printer to: `BullZip`
  - Enabled sharing with share name: `BullZip`
- Deployed printer to Sales users via a GPO startup PowerShell script

### Deployment Script

```powershell
# Connect to the shared printer on the domain controller
Add-Printer -ConnectionName "\\dcserv1\BullZip"

# Set as default printer
(New-Object -ComObject WScript.Network).SetDefaultPrinter("\\dcserv1\BullZip")
```

### Notes
- BullZip uses a proprietary driver; switching to Microsoft PS Class Driver makes it compatible with clients
- The printer share must be applied before the GPO is applied

---

## Lab 08 — File Shares Implementation

**Objective:** Create and configure NTFS and SMB file shares for HR and Sales departments with appropriate permissions and quotas.

### Key Tasks
- Took mandatory server snapshot named **Lab8**
- Installed FSRM (File Server Resource Manager) role:
  ```powershell
  Install-WindowsFeature -Name FS-Resource-Manager -IncludeManagementTools
  ```
- Created HR Share at `C:\File_Shares\HR_Share\Reports`
- Created Sales Share with full automation script (see below)
- Applied a 2 GB quota to the Sales Share

### Sales Share PowerShell Script

```powershell
# Define variables
$share_path       = "C:\File_Shares\Sales_Share"
$share_name       = "Sales Share"
$share_group      = "Sales Users"
$campaigns_folder = "$share_path\Campaigns"
$campaigns_group  = "Sales Campaigns Users"
$campaigns_user   = "mcdaisy1"

# Create Sales_Share folder
if (!(Test-Path -Path $share_path)) {
    New-Item -Path $share_path -Type Directory
}

# Set NTFS permissions for Sales Users (Read & Execute)
$acl         = Get-ACL $share_path
$access_rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "$share_group", "ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.AddAccessRule($access_rule)
Set-ACL -Path $share_path -AclObject $acl

# Create SMB share
New-SmbShare -Name $share_name -Path $share_path -FullAccess $share_group

# Create Campaigns subfolder
if (!(Test-Path -Path $campaigns_folder)) {
    New-Item -Path $campaigns_folder -Type Directory
}

# Create Sales Campaigns Users group and add mcdaisy1
New-ADGroup -Name $campaigns_group -SamAccountName $campaigns_group `
  -GroupCategory Security -GroupScope Global -DisplayName $campaigns_group `
  -Path "OU=Groups,DC=cit271,DC=com"
Add-ADGroupMember -Identity $campaigns_group -Members $campaigns_user

# Grant Modify permissions on Campaigns folder to Sales Campaigns Users
$campaigns_acl  = Get-ACL $campaigns_folder
$campaigns_rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    $campaigns_group, "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")
$campaigns_acl.AddAccessRule($campaigns_rule)
Set-ACL -Path $campaigns_folder -AclObject $campaigns_acl

# Create test file and apply 2 GB quota
New-Item -Path "$campaigns_folder\test_campaign.txt" -ItemType File
New-FsrmQuota -Path $share_path -Size 2GB
```

---

## Lab 09 — Storage Configuration

**Objective:** Create disk partitions using Disk Management and PowerShell, then configure RAID 1 using two additional virtual disks.

### Key Tasks
- Took mandatory server snapshot named **Lab9** (snapshot recommended before starting)
- **Part 1 – Disk Management:** Created a 10 GB simple volume labeled **E: (VM Drive)** using NTFS from unallocated space on C:
- **Part 2 – PowerShell:** Created another 10 GB simple volume labeled **F: (Extra Drive)** using PowerShell
- **Part 3 – RAID 1:** Configured the two extra hard disks to use RAID 1 (mirroring)

### PowerShell Partition Commands

```powershell
# Check current partitions
Get-Partition -DiskNumber 0

# Shrink C: by 10240 MB to free up space (size in bytes)
Resize-Partition -DiskNumber 0 -PartitionNumber 3 -Size (59.37GB)

# Create new partition using remaining free space
New-Partition -DiskNumber 0 -UseMaximumSize -DriveLetter F

# Format as NTFS with label
Format-Volume -DriveLetter F -FileSystem NTFS -NewFileSystemLabel "Extra Drive"
```

### Notes
- C: drive is on Disk 0, Partition 3 — always verify with `Get-Partition` before resizing
- Size must be specified carefully: convert GB to MB using a unit converter, then subtract 10240 MB to get exactly 10 GB
- RAID 1 was configured via Windows Disk Management (convert to dynamic disk → mirror volume)

---

## Lab 10 — Security Overview

**Objective:** Harden server and client security through GPO restrictions, login warnings, and user access controls.

### Key Tasks
- Took mandatory snapshots: **Lab10** on both the server and Windows 11 client
- Configured **Default Domain Controllers Policy** GPO (under Domain Controllers OU):

#### Server Login Hardening

| Setting | GPO Path |
|---------|----------|
| Deny log on locally (HR/Sales/Facilities/Legal Users) | `Computer Config > Policies > Windows Settings > Security Settings > Local Policies > User Rights Assignment > Deny log on locally` |
| Login warning title ("WARNING:") | `Computer Config > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Interactive logon: Message title for users attempting to log on` |
| Login warning message | `Computer Config > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Interactive logon: Message text for users attempting to log on` |

**Login Warning Message:**
> "Only approved IT users should login to this server. Unauthorized users are not permitted to login. VIOLATORS WILL BE PROSECUTED."

#### Client Security Restrictions (Windows 11)

| Setting | GPO Path |
|---------|----------|
| Block Command Prompt access | `User Config > Policies > Administrative Templates > System > Prevent access to the command prompt` |
| Block Registry Editor access | `User Config > Policies > Administrative Templates > System > Prevent access to registry editing tools` |

### Notes
- Deny logon GPO lists only: HR Users, Sales Users, Facilities Users, Legal Users (do NOT include IT Users)
- Login title and message require two separate policy settings

---

## Environment Reference

| Component | Value |
|-----------|-------|
| Domain | `cit271.com` |
| Domain Controller | `dcserv1` |
| Server IP | `192.168.1.2` |
| Gateway (PfSense) | `192.168.1.1` |
| DNS (Primary) | `192.168.1.2` |
| DNS (Alternate) | `10.11.0.51` |
| Admin Password | `Student271!` |
| DSRM Password | `Student271!` |
| Win10 Client Name | `win10client` |
| Win11 Client Name | `win11client` |

---

## Lab 11 — Logging Configuration

**Objective:** Use Windows Event Viewer to remotely monitor client logs and create custom filtered views on the server.

### Key Tasks
- Took mandatory server snapshot named **Lab11**
- Configured client firewalls **via Group Policy** (HR Users and Sales Users GPOs) to allow remote Event Log Management
  - Created new inbound rule using the **Predefined** rule type: `Remote Event Log Management`
  - Rebooted both clients to apply GPO (Win11 required BitLocker password entry)
- Remotely viewed logs from both clients in Event Viewer on the server:
  - Windows 10 client → Application logs
  - Windows 11 client → Security logs
- Created two custom views on the server:

### Custom Event Viewer Views

| Setting | Custom View 1 | Custom View 2 |
|---------|--------------|--------------|
| Logged Time | Last 7 days | Any time |
| Event Levels | Critical, Error, Warning, Information | All levels |
| Event Logs | Security | SMBClient, SMBDirect, SMBServer, SMBWitnessClient |
| Name | **Weekly Security Review** | **SMB Logs** |

### Notes
- Remote Event Viewer access requires inbound firewall rules on the client — must be done via GPO, not manually
- The SMB Logs view may show a warning about volume of logs; this can be ignored

---

## Lab 12 — *(Files not yet provided)*

*(To be added — please share the Lab 12 files.)*

---

## Lab 13 — Virtualization Implementation

**Objective:** Enable Hyper-V on the Windows Server domain controller and deploy a nested Linux virtual machine.

### Key Tasks
- Took mandatory server snapshot named **Lab13**
- Installed Hyper-V feature via PowerShell, restarted, and verified installation
- Downloaded **TinyCore Linux** ISO and moved it to the VM Drive (E:)
- Created and ran a nested VM named **TinyCore** inside Hyper-V
- Ran basic Linux commands inside TinyCore
- Powered off TinyCore VM after the lab

### PowerShell Commands — Hyper-V Installation

```powershell
# Install Hyper-V with all sub-features and management tools
Install-WindowsFeature -Name Hyper-V -IncludeAllSubFeature -IncludeManagementTools

# Restart the server
Restart-Computer

# Verify Hyper-V features were installed (filtered output)
Get-WindowsFeature | Where-Object { $_.Name -like '*Hyper-V*' }
```

### TinyCore VM Settings

| Setting | Value |
|---------|-------|
| Name | `TinyCore` |
| Location | `E:\` |
| Generation | Generation 1 |
| Memory | 1024 MB (no Dynamic Memory) |
| Networking | Not Connected |
| Virtual Hard Disk | 1 GB, stored on E: |
| Boot Media | TinyCore Linux ISO |

### Linux Commands Run Inside TinyCore

```bash
pwd          # Check current directory
whoami       # Check current user
touch test.txt   # Create a blank file
ls           # List contents of current directory
```

### Notes
- Hyper-V requires nested virtualization to be enabled in COIVcenter on the server VM
- TinyCore must be shut off in Hyper-V Manager after the lab — do not leave it running

---

## Lab 14 — WSL 2 Containerization Overview

**Objective:** Install WSL 2 and Docker on the Windows 11 client, then deploy and host a custom Apache web server using a Docker container accessible across the LAN.

### Key Tasks
- Took mandatory **Windows 11 client** snapshot named **Lab14**
- Installed WSL 2 and Ubuntu on `win11client`
  - Created WSL user: `mcmario1` / `Student271!`
- Installed Docker inside the Ubuntu WSL 2 environment
- Ran and tested the `hello-world` Docker container
- Deployed an Apache (`httpd:2.4`) container on port 8080
- Created a custom `test.html` webpage hosted by Apache
- Configured port forwarding and Windows Firewall to expose port 8080 to the LAN
- Verified the webpage was accessible from `win10client` at `win11client:8080/test.html`

### Docker Installation Commands (inside Ubuntu WSL 2)

```bash
# Update system packages
sudo apt update -y && sudo apt upgrade -y

# Install Docker dependencies
sudo apt install --no-install-recommends apt-transport-https ca-certificates curl gnupg2 -y

# Configure Docker package repository
. /etc/os-release
curl -fsSL https://download.docker.com/linux/${ID}/gpg | sudo tee /etc/apt/trusted.gpg.d/docker.asc
echo "deb [arch=amd64] https://download.docker.com/linux/${ID} ${VERSION_CODENAME} stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update -y

# Install Docker
sudo apt install docker-ce docker-ce-cli containerd.io -y

# Add mcmario1 to docker group
sudo usermod -aG docker $USER

# Start Docker daemon (run in a separate terminal)
sudo dockerd --iptables=false
```

### Docker Testing Commands

```bash
# Run hello-world test container
docker run hello-world

# List all containers
docker ps -a
```

### Apache Container Setup

```bash
# Create and prepare web directory
mkdir /home/mcmario1/web
chmod 777 /home/mcmario1/web
cd /home/mcmario1/web

# Run Apache container mapped to port 8080
docker run -dit --name apache -p 8080:80 \
  -v "$PWD":/usr/local/apache2/htdocs/ httpd:2.4
```

### Port Forwarding — Expose WSL 2 to LAN (PowerShell on Win11)

```powershell
# Get WSL 2 eth0 IP address first (run inside Ubuntu):
# ip -br a

# Forward port 8080 from WSL 2 to the Windows host (replace <WSL2_IP>)
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 `
  connectport=8080 connectaddress=<WSL2_IP>
```

### Firewall Rule
- Created inbound rule in **Windows Defender Firewall with Advanced Security**
  - Protocol: TCP, Port: 8080
  - Action: Allow all connections, all profiles
  - Name: `WSL 2 Apache 8080`

### Webpage
- File: `/home/mcmario1/test.html`
- Title: `CIT 271`
- Header: `This is the CIT 271 web server.`
- Accessible at: `win11client:8080/test.html` from `win10client`

### Notes
- WSL must be rebooted after step 2 of installation before the kernel update package installs
- The WSL 2 eth0 IP (typically a 172.x.x.x address) changes on restarts — port forwarding may need to be rerun
- Docker Desktop is not used; Docker runs natively inside the WSL 2 Ubuntu environment

---

## Labs 15–20

*(To be added — please share the remaining lab files.)*

---

*Documentation compiled from lab instructions and submitted evidence for CIT 271: Windows Administration.*

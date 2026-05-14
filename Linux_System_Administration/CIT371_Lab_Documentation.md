# CIT 371: Linux System Administration — Lab Documentation

**Student:** Sushant Man Shrestha  
**Email:** Shresthas11@nku.edu  
**Course:** CIT 371 — Linux System Administration

---

## Table of Contents

1. [Lab 02 — Software Installation](#lab-02--software-installation)
2. [Lab 03 — Services and Process Management](#lab-03--services-and-process-management)
3. [Lab 04 — Bash Scripting Part 1](#lab-04--bash-scripting-part-1)
4. [Lab 05 — Bash Scripting Part 2](#lab-05--bash-scripting-part-2)
5. [Lab 06 — Bash Scripting Part 3 (Ping Script)](#lab-06--bash-scripting-part-3-ping-script)
6. [Lab 06b — Network Troubleshooting Script](#lab-06b--network-troubleshooting-script)
7. [Lab 07 — WireGuard VPN Installation](#lab-07--wireguard-vpn-installation)
8. [Lab 08 — Docker Containerization](#lab-08--docker-containerization)
9. [Lab 09 — Security and Firewall Management](#lab-09--security-and-firewall-management)
10. [Lab 10 — Logging Management](#lab-10--logging-management)
11. [Lab 11 — Monitoring Management](#lab-11--monitoring-management)
12. [Lab 12 — File Sharing: NFS and SMB](#lab-12--file-sharing-nfs-and-smb)
13. [Lab 13 — Filesystem and RAID Management](#lab-13--filesystem-and-raid-management)
14. [Lab 14 — Linux Virtualization](#lab-14--linux-virtualization)

---

## Lab 02 — Software Installation

### Objective
Learn how to install software on Linux using a package manager and manually from source.

### Part 1: Package Manager — SSH Installation

Install and configure OpenSSH Server on both the Ubuntu server and client, then connect from client to server via SSH.

**Commands Used:**
```bash
sudo apt update && sudo apt upgrade
sudo apt install openssh-server
sudo systemctl enable --now ssh
sudo ufw allow ssh
ssh username@IP_address
```

**Steps Summary:**
- Updated package lists and upgraded existing packages.
- Installed the OpenSSH server package via `apt`.
- Enabled and started the SSH service immediately with `--now`.
- Opened the SSH port through UFW firewall.
- Connected from the client to the server using the `ssh` command.

---

### Part 2: Manual Installation — Git from Source

Manually installed Git from source on the Ubuntu server, then configured it with name and NKU email.

**Steps Summary:**
- Installed required build dependencies via `apt`.
- Downloaded and compiled Git from source following the DigitalOcean guide.
- Verified installation with `git --version`.
- Configured Git with user name and email:
  ```bash
  git config --global user.name "Sushant Man Shrestha"
  git config --global user.email "Shresthas11@nku.edu"
  ```

---

## Lab 03 — Services and Process Management

### Objective
Learn how to manage services and processes in Linux using `systemctl` and process tools.

### Part 1: Service Management — Systemctl

Practiced managing the SSH service using `systemctl` commands.

**Commands Used:**
```bash
sudo systemctl stop ssh
sudo systemctl start ssh
sudo systemctl restart ssh
sudo systemctl status ssh
sudo systemctl enable ssh
```

**Steps Summary:**
- Stopped SSH and attempted to connect from client (connection was blocked/refused — expected).
- Started SSH back up and verified reconnection.
- Restarted the service with a single `restart` command.
- Checked status and enabled the service to start on boot.

---

### Part 2: Service Management — Git Daemon Creation

Created a systemd service file for Git so it can be managed with `systemctl`.

**Service Configuration File (`/etc/systemd/system/git-daemon.service`):**
```ini
[Unit]
Description=Start Git Daemon

[Service]
ExecStart=/usr/bin/git daemon --reuseaddr --base-path=/srv/git/ /srv/git/
Restart=always
RestartSec=500ms
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=git-daemon
User=student
Group=student

[Install]
WantedBy=multi-user.target
```

**Steps Summary:**
- Created the `/srv/git` directory.
- Created the systemd unit file above.
- Ran `sudo systemctl daemon-reload` and enabled the git daemon to start on boot.

---

### Part 3: Process Management

Used `top`, `ps`, and `kill` commands to observe and manage running processes.

**Answers from `top` output:**
| Question | Answer |
|---|---|
| Number of users logged in | 1 |
| Number of sleeping processes | 288 |
| Free memory (MiB) | 1921 |
| Swap in use (MiB) | 0 |
| Highest PID for student user | 4020 |

**Commands Used:**
```bash
# Filter SSH processes for the student user
ps aux | grep sshd

# Kill the SSH processes by PID (example)
sudo kill <PID1> <PID2>
```

---

## Lab 04 — Bash Scripting Part 1

### Objective
Write two Bash scripts: one for user creation and one for listing directory contents.

> **Note:** Only the Bash scripting language was used for this lab.

---

### Part 1: User Creation Script

**Script:**
```bash
#!/bin/bash

echo "Retrieving info to create new user."

read -p "Please enter a login name: " login_name
read -p "Please enter the full name (GECOS): " full_name
read -p "Please enter the login shell (e.g., /bin/bash): " login_shell
read -p "Please enter the home directory (e.g., /home/username): " home_directory

echo "Creating user..."

sudo adduser --home "$home_directory" --shell "$login_shell" --gecos "$full_name" "$login_name"

echo "User $login_name has been created successfully."
```

**Testing:**  
Created a user `mcben1` (full name: Ben McBen, shell: `/bin/bash`, home: `/home/mcben1`) with password `Passw0rd`, then logged in to verify account creation.

---

### Part 2: Directory Contents Listing Script

**Script:**
```bash
#!/bin/bash

echo "Files:"
for item in *; do
    if [ -f "$item" ]; then
        echo -e "\t$item"
    fi
done

echo "Directories:"
for item in *; do
    if [ -d "$item" ]; then
        echo -e "\t$item"
    fi
done
```

**Testing:**  
Created `/home/student/test` with multiple files and subdirectories, then ran the script inside that folder to confirm correct output grouping.

---

## Lab 05 — Bash Scripting Part 2

### Objective
Write a menu-driven Bash script that manages system services via `systemctl`.

> **Note:** Only the Bash scripting language was used for this lab.

---

### Service Management Menu Script

**Script:**
```bash
#!/bin/bash

display_menu() {
    echo "-----SERVICE-SCRIPT-MENU-----"
    echo "1. Start a service."
    echo "2. Stop a service."
    echo "3. Restart a service."
    echo "4. Enable a service."
    echo "5. Check a service's status."
    echo "6. Clear screen."
    echo "7. Exit script."
    echo "Please choose an option by entering its number (i.e., 1, 2):"
}

manage_service() {
    local action=$1
    read -p "Enter the name of the service: " service_name

    if [ "$action" == "status" ]; then
        systemctl $action $service_name
    else
        if systemctl $action $service_name; then
            echo "Service $service_name was successfully ${action}ed."
        else
            echo "Failed to $action service $service_name."
        fi
    fi

    echo "Press Enter to continue..."
    read
}

while true; do
    display_menu
    read choice

    case $choice in
        1) manage_service "start" ;;
        2) manage_service "stop" ;;
        3) manage_service "restart" ;;
        4) manage_service "enable" ;;
        5) manage_service "status" ;;
        6) clear ;;
        7) echo "Exiting script. Goodbye!"; exit 0 ;;
        *) echo "Invalid option. Please try again." ;;
    esac
done
```

**Testing:**  
Tested all five `systemctl` options using the SSH service. The script correctly looped back to the menu after each operation and exited cleanly on option 7.

---

## Lab 06 — Bash Scripting Part 3 (Ping Script)

### Objective
Write a Bash script that reads a file of IP addresses, validates each one, and performs a ping test.

> **Note:** Only the Bash scripting language was used for this lab.

---

### IP Ping Validation Script

**Script:**
```bash
#!/bin/bash

# Function to validate IP address using ip route get
validate_ip() {
    local ip=$1
    # Attempt to use ip route get to validate IP - suppress all output
    ip route get "$ip" > /dev/null 2>&1
    return $?
}

# Check if input file exists
if [ $# -ne 1 ]; then
    echo "Usage: $0 <input_file>"
    exit 1
fi

input_file=$1

# Check if file exists and is readable
if [ ! -r "$input_file" ]; then
    echo "Error: Cannot read input file '$input_file'"
    exit 1
fi

# Process each line in the file
while read -r ip; do
    echo "Processing IP address: $ip"

    # Validate IP address
    if validate_ip "$ip"; then
        echo "Valid IP address"

        # Attempt to ping the IP address - suppress ping output
        if ping -c 1 "$ip" > /dev/null 2>&1; then
            echo "Successfully pinged $ip"
        else
            echo "Could not ping $ip"
        fi
    else
        echo "Invalid IP address: $ip"
    fi
    echo "------------------------"
done < "$input_file"
```

**Test IP file contents:**
- Client VM's IP address
- `8.8.8.8`
- `267.333.4.55` *(invalid — skipped)*
- `192.168.1.255`

**Testing:**  
Ran the script against the `.txt` file. Valid IPs were pinged; the invalid address `267.333.4.55` was caught and skipped with a message.

---

## Lab 06b — Network Troubleshooting Script

### Objective
Write a Bash script providing a menu-driven network troubleshooting tool using commands like `ping`, `tracepath`, `nslookup`, `netstat`, and `arp`.

---

### Network Troubleshooting Menu Script

**Script:**
```bash
#!/bin/bash

run_ping() {
    read -p "Enter an IP address or hostname to ping: " target
    while true; do
        read -p "Enter the number of requests to send: " count
        if [[ $count =~ ^[0-9]+$ ]]; then
            ping -c $count $target
            break
        else
            echo "Please enter a valid integer."
        fi
    done
}

run_tracepath() {
    read -p "Enter an IP address or hostname to trace: " target
    timeout 10s tracepath "$target"
}

run_nslookup() {
    read -p "Enter an IP address or hostname to lookup: " target
    nslookup "$target"
}

run_netstat() {
    netstat -r
}

run_arp() {
    arp -e
}

while true; do
    echo "Network Troubleshooting Tool"
    echo "1. Ping"
    echo "2. Tracepath"
    echo "3. NSLookup"
    echo "4. Netstat (Routing Table)"
    echo "5. ARP Table"
    echo "6. Clear Screen"
    echo "7. Exit"
    read -p "Enter your choice: " choice

    case $choice in
        1) run_ping ;;
        2) run_tracepath ;;
        3) run_nslookup ;;
        4) run_netstat ;;
        5) run_arp ;;
        6) clear ;;
        7) exit 0 ;;
        *) echo "Invalid option. Please try again." ;;
    esac

    echo
    read -p "Press Enter to continue..."
    clear
done
```

**Additional Command Used:**
```bash
sudo dhclient -r
```
*(Used to release the DHCP lease as part of a network reset/testing step.)*

---

## Lab 07 — WireGuard VPN Installation

### Objective
Install WireGuard VPN on the Ubuntu server, configure the PfSense firewall to allow VPN traffic, and connect a Xubuntu client through the VPN tunnel.

### Part 1: Installing WireGuard (Server)

- Installed WireGuard on the Ubuntu server (PPA already present on the VM).
- Generated public/private key pair.
- Configured `/etc/wireguard/wg0.conf` with:
  - Interface address: `192.168.140.1/24`
  - iptables rules using interface `ens160` (not `enp0s3`).
  - Added peer public key and AllowedIPs for the client.
- IPv6 configuration was skipped.

**Key commands:**
```bash
sudo cat /etc/wireguard/publickey
sudo cat /etc/wireguard/privatekey
sudo systemctl enable --now wg-quick@wg0
```

---

### Part 2: Configuring the PfSense Firewall

- Added a NAT port-forwarding rule for UDP port `51820` pointing to the server's `192.168.1.X` address.
- Disabled the two default **Reserved Networks** blocking rules on the WAN interface (required for lab environment only).
- Added a WAN firewall rule to allow any source to reach UDP port `51820`.

---

### Part 3: Connecting from the Xubuntu Client

- Installed WireGuard on the Xubuntu VM (PPA already present).
- Created `/etc/wireguard/wg0.conf` with:
  - Interface address: `192.168.140.2/32`
  - AllowedIPs: `0.0.0.0/0`
  - Endpoint: `<PfSense_WAN_IP>:51820`
  - Server's public key in the `[Peer]` block.
- Started the VPN tunnel and verified connectivity.

---

## Lab 08 — Docker Containerization

### Objective
Install Docker Engine and use it to run a containerized Apache web server hosting a custom HTML page.

### Part 1: Installing Docker

**Commands Used:**
```bash
# Add Docker's official GPG key
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test installation
sudo docker run hello-world
```

> **Note:** Docker Engine was installed from Docker's official documentation, not from the unofficial `docker.io` package.

---

### Part 2: Apache Containerization

- Ran the official Apache Docker image from `/home/student/website` using port `8080`.
- Created a custom webpage at `/home/student/website/home.htm`:

**home.htm contents:**
```html
<!DOCTYPE html>
<html>
<head><title>371Serv</title></head>
<body>
  <h1><b>371Serv Homepage</b></h1>
  <p style="font-size:16px;">Sushant Man Shrestha</p>
</body>
</html>
```

**Docker run command (example):**
```bash
docker run -dit -p 8080:80 -v "$PWD":/usr/local/apache2/htdocs/ httpd:latest
```

- Accessed the page from the server's browser at `http://localhost:8080/home.htm`.
- Accessed it from the Ubuntu client machine as well.

---

## Lab 09 — Security and Firewall Management

### Objective
Improve the security of the Ubuntu server using GPG encryption, SSH hardening, and UFW firewall rules.

### Part 1: Security Checks and Configurations

**Commands Used:**
```bash
# Find accounts with no password set
sudo awk -F: '($2 == "" ) { print $1 }' /etc/shadow

# Find world-writable files and directories
sudo find / -type f -perm -o+w
sudo find / -type d -perm -o+w

# Switch to root and create/encrypt a file
sudo -i
echo "Secure Server" > /root/secure.txt
gpg --symmetric --cipher-algo AES256 --passphrase Passw0rd /root/secure.txt
rm /root/secure.txt

# Disable GPG password caching (as student user)
echo "default-cache-ttl 0" >> ~/.gnupg/gpg-agent.conf

# Disable root SSH login
sudo nano /etc/ssh/sshd_config
# Edited line: PermitRootLogin no

sudo systemctl restart sshd
```

---

### Part 2: Firewall Checks and Configurations

**Commands Used:**
```bash
# Enable UFW and set to start on boot
sudo ufw enable
sudo systemctl enable ufw

# List current rules (verbose)
sudo ufw status verbose

# Enable logging
sudo ufw logging on

# Deny Telnet (port 23)
sudo ufw deny 23

# Insert allow rule for port 8080 at position 1
sudo ufw insert 1 allow 8080

# Replace generic SSH rule with client-specific rule
sudo ufw delete allow ssh
sudo ufw allow from 192.168.1.210 to any port 22
```

- Verified the client-specific SSH rule by successfully connecting from the client at `192.168.1.210`.

---

## Lab 10 — Logging Management

### Objective
Configure syslog for centralized log forwarding from client to server, then set up Dozzle as a containerized Docker log monitoring tool.

### Part 1: Syslog Configuration

**Server-side configuration** (`/etc/rsyslog.conf`):
- Uncommented or added the lines to accept logs on both UDP (port 514) and TCP (port 514).

**Client-side configuration** (`/etc/rsyslog.conf`):
- Added forwarding rules at the **bottom** of the file pointing to the server's IP.

**Test log command (on client):**
```bash
logger "Hello from 371client"
```

**Verification (on server):**
```bash
grep "Hello from 371client" /var/log/syslog
# Or if redirected:
cat /var/log/371client/student.log
```

---

### Part 2: Dozzle Containerization

- Created the `/data` directory on the server and added a `users.yml` file with hashed credentials.

**users.yml configuration:**
```yaml
users:
  thedozzler:
    name: "The Dozzler"
    password: "<bcrypt-hashed-password-of-D0zzl3d!>"
```

**Docker run command (example):**
```bash
docker run -dit \
  -p 8888:8080 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /data:/data \
  amir20/dozzle:latest \
  --auth-provider file
```

- Accessed Dozzle at `http://localhost:8888` from Firefox on the server.
- Logged in as `thedozzler` and viewed live logs from the Apache container.

---

## Lab 11 — Monitoring Management

### Objective
Run an Uptime Kuma container on the Ubuntu server and configure it to monitor Docker containers, a web page, and the Ubuntu client.

### Part 1: Uptime Kuma Installation

**Docker Run Command:**
```bash
docker run -dit --name uptime-kuma -p 3001:3001 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  louislam/uptime-kuma
```

- The `-v /var/run/docker.sock:/var/run/docker.sock` option shares the Docker socket with the container, enabling container monitoring.
- The `--restart=always` flag was intentionally omitted per lab requirements.
- Uptime Kuma is accessible at `http://<server-ip>:3001`.

---

### Part 2: Uptime Kuma Configuration

- Created an account with credentials `uptime_admin` / `Kum@C00l`.
- Configured a Docker host named `371serv` using the socket path and ran a successful connection test before saving.

**Command used to list Docker container IDs and names:**
```bash
docker ps
```

**Monitors configured:**

| Monitor Name | Type | Target |
|---|---|---|
| Apache Webpage | HTTP(s) | `http://192.168.1.X:8080` |
| Apache Container | Docker Container | Container ID/name via `371serv` host |
| Dozzle Container | Docker Container | Container ID/name via `371serv` host |
| Google DNS | HTTP(s) | `https://dns.google/` |

- All four monitors were confirmed as "Up" in the Uptime Kuma dashboard.

---

## Lab 12 — File Sharing: NFS and SMB

### Objective
Research NFS and SMB file sharing protocols, select one, and implement it across the Ubuntu server and client.

### Part 1: Selection and Justification

> **Network File System (NFS)** is a distributed file system protocol originally developed by Sun Microsystems in 1984. It enables users to access files and directories located on remote computers as if they were stored locally. NFS is particularly prevalent in Unix-based environments and operates on a client-server architecture. The protocol is designed to be simple and efficient, employing RPCs to manage communications between clients and servers, and is particularly well-suited to local area networks where concerns about security are at a minimum (Smith 45).
>
> **Server Message Block (SMB)**, in its modern versions sometimes referred to as Common Internet File System (CIFS), is a network protocol created by Microsoft for sharing files, printers, and serial ports between computers over a network. Unlike NFS, SMB was designed with authentication and security in mind, featuring user-level authentication, support for encryption, and fine-grained access controls. SMB has undergone many changes since its original development. Modern versions of SMB are capable of high performance and security for both local networks and internet-based file sharing (Johnson 92).
>
> **NFS was selected** for this implementation. This is mainly because it natively integrates with Unix-based systems such as Ubuntu, making it particularly suitable for this environment. NFS performs better in Linux environments due to its lightweight nature and fewer overhead requirements compared to SMB. The `no_root_squash` option is a native NFS feature that ensures proper write access functionality. While SMB provides more robust security, the simplicity and performance advantages of NFS justify it as the more fitting option for this Ubuntu-based, controlled local network implementation.

**Works Cited:**
- Johnson, Mark. "Modern Network Protocols: A Comprehensive Guide." *Network Computing Quarterly* 28.2 (2023): 88–96.
- Smith, Sarah. "Unix Network Services: Implementation and Management." *System Administration Journal* 15.4 (2023): 42–51.

---

### Part 2: NFS Implementation

**Server Commands:**
```bash
sudo apt update
sudo apt install nfs-kernel-server -y

# Create and configure the shared folder
sudo mkdir /public
sudo chmod 777 /public

# Create share.txt with required content
echo "Bob McBob was technically here." | sudo tee /public/share.txt
sudo chmod 666 /public/share.txt

# Configure NFS exports
sudo bash -c 'echo "/public *(rw,sync,no_root_squash)" >> /etc/exports'
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

**Client Commands:**
```bash
sudo apt update
sudo apt install nfs-common -y

# Mount the NFS share
sudo mkdir /mnt/nfs_share
sudo mount 192.168.1.108:/public /mnt/nfs_share

# Verify mount and file access
df -h
nano /mnt/nfs_share/share.txt
```

- Confirmed the share was mounted with `df -h`.
- Opened `share.txt` in nano from the client — no "unwritable" error appeared, confirming read/write access.

---

## Lab 13 — Filesystem and RAID Management

### Objective
Format disks, create partitions, and set up a RAID 0 array on the Ubuntu server.

### Part 1: Partition Management

Used `fdisk` to create a primary partition on each of the two 500MB unformatted disks (`/dev/sdb` and `/dev/sdc`), then formatted them with the ext4 filesystem.

**Commands and Menu Options Used:**

```bash
sudo fdisk /dev/sdb
# In fdisk interactive menu:
#   n  → new partition
#   p  → primary partition
#   1  → partition number (default)
#   Enter → accept default first sector
#   Enter → accept default last sector
#   w  → write changes and exit

sudo fdisk /dev/sdc
# Same menu steps as above

# Format both partitions as ext4
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 /dev/sdc1

# Reboot to apply changes
sudo reboot

# List all partitions to confirm
lsblk
```

---

### Part 2: RAID 0 Array Creation

Created a RAID 0 array from the two new partitions, formatted it, mounted it, and saved the configuration for automatic reassembly at boot.

**Commands Used:**
```bash
# Create the RAID 0 array
sudo mdadm --create /dev/md0 --level=0 --raid-devices=2 /dev/sdb1 /dev/sdc1

# Confirm the array was created
cat /proc/mdstat

# Create ext4 filesystem on the array
sudo mkfs.ext4 /dev/md0

# Create mount point and mount the array
sudo mkdir /mnt/md0
sudo mount /dev/md0 /mnt/md0

# Save array layout for automatic reassembly at boot
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u

# Verify the filesystem is available
df -h /mnt/md0
```

---

## Lab 14 — Linux Virtualization

### Objective
Install KVM on the Ubuntu server and run a Tiny Core Linux virtual machine using Virtual Machine Manager.

### Part 1: KVM Installation

**Commands Used:**
```bash
# Install KVM and required packages
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils -y

# Confirm the server supports virtualization
sudo kvm-ok
```

---

### Part 2: Virtual Machine Creation

**Prepare the ISO:**
```bash
# Download the Tiny Core Linux base (Core) ISO from https://tinycorelinux.net
# Move it to the libvirt images directory
sudo mv Downloads/TinyCore-current.iso /var/lib/libvirt/images/
```

**Install Virtual Machine Manager (if not present):**
```bash
sudo apt install virt-manager -y
```

**Resolve the "QEMU/KVM – Not Connected" Error:**

The following steps resolved the connection error in Virtual Machine Manager:
```bash
sudo systemctl restart libvirtd
sudo usermod -aG libvirt,kvm $(whoami)
newgrp libvirt
```
> Source: [Reddit – r/pop_os: Fix for virt-manager QEMU/KVM not connected error](https://www.reddit.com/r/pop_os/comments/zzyfrh/dock_running_virtmanager_error_qemu_kvm_not/)

**VM Settings Used:**

| Setting | Value |
|---|---|
| Install media | Core ISO (local) |
| OS | Generic / Unknown (auto-detect disabled) |
| RAM | 184 MiB |
| CPUs | 1 |
| Storage | Disabled |
| VM Name | Core |

**Commands Run Inside the Core VM:**
```bash
# Ping Google's public DNS
ping 8.8.8.8

# Show kernel information
uname -a

# Show the VM's IP address
hostname -i
```

- Allowed inhibiting shortcuts when prompted after launch.
- Shut down the Core VM after completing all tasks.

---

*Documentation compiled from CIT 371 lab submissions — Sushant Man Shrestha*

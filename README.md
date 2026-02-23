**Table of Contents**

1. Overview
2. Architecture
3. Prerequisites
4. Instillation & Setup
5. Network Configuration
6. Post Install Configuration
7. Lab Exercises & Use Cases
8. Troubleshooting
9. Resources
10. Disclaimer

---

**Overview**
This project provides a step by step guidance for building a cybersecurity homelab on macOS using virtualization. The lab consist of two primary virtual machines.

| VM | Role | Purpose |
|----|------|---------|
|**Kali Linux** | Attacker | Offensive security testing, scanning, exploitation |
| **Ubuntu** | Target / Defender | Hosts vulnerable services, SIEM, logging, and monitoring |

All traffic stays within an isolated virtual network so nothing touches your home network or the internet.

---

**Architecture**

```
┌─────────────────────────────────────────────┐
│                macOS Host                   │
│                                             │
│  ┌──────────────┐     ┌──────────────────┐  │
│  │  Kali Linux  │◄───►│     Ubuntu       │  │
│  │  (Attacker)  │     │  (Target/SIEM)   │  │
│  └──────────────┘     └──────────────────┘  │
│         │                      │            │
│         └──────┐  ┌────────────┘            │
│                ▼  ▼                         │
│        ┌──────────────┐                     │
│        │  Host-Only /  │                    │
│        │  Internal Net │                    │
│        │ 192.168.56.0  │                    │
│        └──────────────┘                     │
└─────────────────────────────────────────────┘
```
---
### Prerequisites

**Hardware**

- Mac with at least **16 GB RAM** (8GB minimum)
- **50+ GB** of free disk space
- Apple Silicon

**Software**

-**macOS Ventura 13+**
-**Hypervisor** - choose one:
- [UTM](https://mac.getutm.app/) - free, native Apple Silicon support
- [VMware Fusion](https://www.vmware.com/products/fusion.html) - free Personal Use license available

**ISO Images**

-[Kali Linux ISO](https://www.kali.org/get-kali/) - download the Installer or VM image for your architecture (ARM64 for Apple Silicon)
-[Ubuntu Server/Desktop ISO](https://ubunti.com/downloads) - 22.04 LTS

---

# Installation & Setup

### Step 1 — Install the Hypervisor

**UTM (recommended for Apple Silicon):**

```bash
brew install --cask utm
```

Or download directly from [mac.getutm.app](https://mac.getutm.app/).

### Step 2 — Create the Kali Linux VM

1. Open UTM → **Create a New Virtual Machine** → **Virtualize** (Apple Silicon) or **Emulate** (Intel)
2. Select **Linux** as the operating system
3. Browse to the Kali Linux ISO
4. Allocate resources:
   - **RAM:** 4 GB (minimum 2 GB)
   - **CPU:** 2 cores
   - **Disk:** 40 GB
5. Complete the wizard and boot the VM
6. Follow the Kali installer (select default options for a standard install)

### Step 3 — Create the Ubuntu VM

1. Repeat the VM creation process with the Ubuntu ISO
2. Allocate resources:
   - **RAM:** 4 GB (minimum 2 GB)
   - **CPU:** 2 cores
   - **Disk:** 30 GB
3. Install Ubuntu Server (or Desktop if you prefer a GUI)

### Step 4 — Install VM Guest Tools

**Ubuntu:**
```bash
sudo apt update && sudo apt install -y spice-vdagent qemu-guest-agent
```

**Kali:**
```bash
sudo apt update && sudo apt install -y spice-vdagent qemu-guest-agent
```

---

## Network Configuration

### Isolated Host-Only Network

Configure both VMs to use a **shared/host-only network** so they can communicate with each other but are isolated from your real network.

**In UTM:**
1. Go to VM Settings → **Network**
2. Set Network Mode to **Host Only** or **Shared Network** (for internet access during setup)
3. Repeat for both VMs

### Assign Static IPs

**Ubuntu** (`/etc/netplan/01-netcfg.yaml`):
```yaml
network:
  version: 2
  ethernets:
    enp0s1:
      addresses:
        - 192.168.56.10/24
      gateway4: 192.168.56.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```
Apply with: `sudo netplan apply`

**Kali** (`/etc/network/interfaces` or NetworkManager):
```
auto eth0
iface eth0 inet static
  address 192.168.56.20
  netmask 255.255.255.0
  gateway 192.168.56.1
```

### Verify Connectivity

From Kali:
```bash
ping 192.168.56.10
```

From Ubuntu:
```bash
ping 192.168.56.20
```

---

## Post-Install Configuration

### Ubuntu — Set Up Vulnerable Services & Monitoring

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install common target services
sudo apt install -y apache2 openssh-server vsftpd mysql-server

# Install monitoring / defensive tools
sudo apt install -y ufw fail2ban net-tools tcpdump

# Enable UFW firewall
sudo ufw enable
sudo ufw allow ssh

# (Optional) Install Wazuh agent, Splunk Universal Forwarder, or ELK stack
```

### Kali — Verify Offensive Tools

```bash
# Update tool repositories
sudo apt update && sudo apt upgrade -y

# Confirm key tools are installed
which nmap metasploit-framework burpsuite wireshark nikto john hydra
```

---

## Lab Exercises & Use Cases

| Exercise | Tools | Description |
|----------|-------|-------------|
| **Network Scanning** | Nmap, Netdiscover | Discover hosts and open ports on the Ubuntu target |
| **Vulnerability Assessment** | Nessus, OpenVAS, Nikto | Scan Ubuntu services for known vulnerabilities |
| **Exploitation** | Metasploit, Searchsploit | Exploit vulnerable services on the Ubuntu target |
| **Password Attacks** | Hydra, John the Ripper | Brute-force SSH, FTP, or MySQL on Ubuntu |
| **Web App Testing** | Burp Suite, OWASP ZAP | Test the Apache web server with DVWA or Juice Shop |
| **Packet Analysis** | Wireshark, tcpdump | Capture and analyze traffic between VMs |
| **SIEM & Log Analysis** | Splunk, Wazuh, ELK | Collect and correlate logs from the Ubuntu target |
| **Incident Response** | Volatility, Autopsy | Practice forensic investigation on VM snapshots |

### Optional — Add Intentionally Vulnerable Apps

```bash
# DVWA (Damn Vulnerable Web Application)
sudo apt install -y docker.io
sudo docker run -d -p 80:80 vulnerables/web-dvwa

# Metasploitable (run as a third VM for more targets)
# Download from: https://sourceforge.net/projects/metasploitable/
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| VMs can't ping each other | Verify both are on the same virtual network; check static IP config |
| Kali can't reach the internet | Temporarily switch to Shared/NAT networking for updates |
| Poor VM performance | Increase allocated RAM/CPU; close unnecessary host apps |
| UTM crashes on Apple Silicon | Ensure you downloaded the ARM64 ISO images |
| Display resolution issues | Install `spice-vdagent` guest tools |

---

## Resources

- [Kali Linux Documentation](https://www.kali.org/docs/)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [UTM Documentation](https://docs.getutm.app/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [HackTheBox](https://www.hackthebox.com/) & [TryHackMe](https://tryhackme.com/) — for guided labs
- [CyberDefenders](https://cyberdefenders.org/) — blue team challenges

---

## Disclaimer

This homelab is intended **strictly for educational and authorized testing purposes**. Never use these tools or techniques against systems you do not own or have explicit written permission to test. Unauthorized access to computer systems is illegal. Always practice responsible and ethical security research.

---

## License

This project is provided as-is for educational purposes. See [LICENSE](LICENSE) for details.



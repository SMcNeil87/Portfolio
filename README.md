# # Home Lab — Steven McNeil

This repo documents my personal home lab, which I built to get hands-on experience with the technologies I work with professionally and the ones I’m working toward. It’s not a polished finished product — it’s a working environment that I’m actively learning in and expanding over time.

Everything documented here is something I’ve actually built and tested. I’m using this as both a study resource for the MD-102 exam and a way to develop practical skills beyond what helpdesk work typically allows.

-----

## Hardware

**Primary host — Supermicro desktop server**

This is the backbone of the lab. It runs Windows Server 2022 as the main OS with all core services installed directly on the hardware. Storage is JBOD — no RAID configured, which is fine for a lab environment where redundancy isn’t a concern.

Current drive layout:

- **Drive 1 (C:)** — Windows Server 2022, all server roles
- **Drive 2** — Ubuntu Server 22.04 LTS
- **Drive 3/4** — Reserved for future use (Proxmox planned)

**Domain-joined devices:**

- The server itself
- Personal gaming PC (Windows 11)
- Two virtual machines — one used as the first successful PXE boot/MDT deployment target

-----

## What’s Running

### On-Premises — Windows Server 2022

**Active Directory Domain Services**
Domain controller handling authentication, DNS, and identity for all joined devices. Users and computers are organized into OUs. This is the identity source for everything else in the on-prem environment.

**DNS**
AD-integrated. Handles name resolution for domain join and PXE boot. Standard AD DNS setup.

**DHCP**
Configured with Option 66 pointing to the WDS server IP and Option 67 pointing to the UEFI boot file (`boot\x64\wdsmgfw.efi`). This is what makes PXE boot work without manually specifying a boot server on the client.

**Windows Deployment Services (WDS)**
Provides the PXE boot environment. Serves the UEFI bootloader and hands off to MDT for the actual deployment.

**Microsoft Deployment Toolkit (MDT)**
Deployment share configured with task sequences for Windows 10 and 11. LiteTouch handles OS installation, driver injection, and basic application setup. Unattend.xml handles the parts that would otherwise require manual input during setup. The first successful PXE boot deployed a VM end-to-end through this pipeline and domain-joined it automatically.

**Ubuntu Server 22.04 LTS**
Recently stood up on a dedicated drive. SSH is enabled and accessible on the local network. XFCE is installed for when I need a GUI. Next steps are user/group management practice and eventually joining it to the AD domain.

-----

### Cloud — Microsoft 365 E5 Tenant

Set up an M365 E5 trial tenant to build out the cloud side of the lab. This gives access to Entra ID P1/P2 features, which is what I’m currently working through as part of MD-102 prep.

**What’s configured so far:**

- User accounts created with E5 licenses assigned
- Conditional Access policies configured and tested
- Microsoft Graph PowerShell SDK installed and working — completed Lab 0101 (identity management), using cmdlets like `Get-MgUser`, `Get-MgGroup`, and `New-MgGroupMemberByRef` for cloud identity automation

The on-prem side is mostly stable. The cloud work is where things are still actively evolving.

-----

## Roadmap

Roughly in order of what’s next:

- [ ] Complete MD-102 certification (Microsoft 365 Certified: Endpoint Administrator Associate)
- [ ] Continue Entra ID and cloud identity labs through the MD-102 study guide
- [ ] Linux user/group management practice on Ubuntu Server
- [ ] Join Ubuntu Server to Windows AD domain using SSSD
- [ ] Deploy Proxmox on a dedicated drive to run Windows Server and Ubuntu as simultaneous VMs
- [ ] Add infrastructure monitoring — Zabbix or Grafana
- [ ] SC-300 (Identity and Access Administrator) — after MD-102
- [ ] AZ-104 (Azure Administrator) — longer term

-----

## Troubleshooting Log

Real problems I ran into and how I solved them.

**PXE boot not working initially**

Clients were getting DHCP addresses but not picking up the boot file. DHCP options weren’t set correctly — Option 66 needed to point to the WDS server IP and Option 67 needed the full path to the UEFI boot file. Once those were corrected PXE worked consistently.

**Ubuntu installer drive selection with identical HDDs**

All three HDDs in the Supermicro are the same size, making them impossible to tell apart in the Ubuntu installer by size alone. Before booting the USB I checked Windows Disk Management and noted the partition state of each drive. The correct target showed as having a primary partition structure but no actual data. Selected that one and the install went cleanly.

**GRUB rescue prompt on first Ubuntu boot attempt**

Expected the Ubuntu installer menu, got a `Welcome to GRUB!` prompt with a flashing cursor. The issue was booting in legacy mode instead of UEFI. Rebooted, selected the UEFI-prefixed USB entry in the Supermicro boot menu, and the installer loaded normally.

**Missing DHCP Option 003** Router caused domain clients to receive IP and DNS assignments correctly but no default gateway, resulting in loss of internet connectivity. Fixed by configuring Option 003 via DHCP Manager Scope Options

**Ubuntu Domain Join Failure**

Ubuntu VM domain join — Kerberos authentication failure

Problem: realm join failing with “Couldn’t authenticate to AD” despite correct credentials and Domain Admin permissions.

Root cause: Ubuntu VM defaulted to UTC timezone, causing system time to be hours ahead of the domain controller. Kerberos requires all domain members to be within 5 minutes of each other — the skew exceeded that threshold and rejected authentication.

Resolution:

sudo timedatectl set-timezone America/Chicago
sudo apt install -y ntpdate
sudo ntpdate 192.168.0.9
sudo realm join --user=Administrator yourdomain.local


apt lock issue encountered during package install:

sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/dpkg/lock
sudo rm /var/lib/apt/lists/lock
sudo dpkg --configure -a
sudo apt-get clean
sudo apt-get update
sudo apt-get install -f



## Skills This Lab Covers

Everything here is something I’ve configured and tested in this environment:

- Active Directory — domain controller setup, OU design, user/group/computer lifecycle
- WDS + MDT — automated OS deployment pipeline from PXE through domain join
- DHCP/DNS — PXE boot configuration, AD-integrated name resolution
- Microsoft Entra ID — cloud identity, Conditional Access, license management
- Microsoft Graph PowerShell SDK — cloud identity automation and scripting
- PowerShell — automation across on-prem AD and cloud identity
- Linux (Ubuntu Server) — installation, SSH configuration, service management
- Documentation — this repo

-----

## About

IT professional in St. Louis working toward systems administration and IT operations roles. This lab is part of that effort — building hands-on experience with enterprise infrastructure beyond what helpdesk work typically covers. Currently pursuing MD-102 with SC-300 and AZ-104 to follow.

[LinkedIn](https://linkedin.com/in/Steven-McNeil)

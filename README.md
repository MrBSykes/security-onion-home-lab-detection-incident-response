🔐 Security Onion Home Lab — Attack Detection & Incident Response
Author: Bryan Sykes
Date: April 2026
Platform: Security Onion 2.4.211 | Oracle VirtualBox | Kali Linux
Type: Blue Team | SOC Analysis | Threat Detection


📋 Project Overview
This project demonstrates a full attack-and-detect simulation in a self-built home lab environment. Acting as both attacker (red team) and defender (blue team), I simulated a realistic threat scenario — network reconnaissance followed by a credential brute force attack — and detected, investigated, and documented it using enterprise-grade security tooling.

The result: 404 HIGH severity alerts generated and triaged, Zeek network logs correlated, and a formal SOC incident report produced — end to end.


🏗️ Lab Architecture
┌─────────────────────────────────────────────┐

│              Windows 11 Host PC              │

│                                              │

│  ┌──────────────────┐  ┌──────────────────┐ │

│  │   Kali Linux VM  │  │ Security Onion VM│ │

│  │  192.168.1.209   │  │  192.168.1.196   │ │

│  │   (Attacker)     │  │   (Defender)     │ │

│  └────────┬─────────┘  └────────┬─────────┘ │

│           │                     │            │

│           └──── Bridged ────────┘            │

│              (Home Network LAN)              │

└─────────────────────────────────────────────┘

Component
Details
Host OS
Windows 11
Hypervisor
Oracle VirtualBox
Attacker VM
Kali Linux (Bridged Adapter — 192.168.1.209)
Defender VM
Security Onion 2.4.211 EVAL (Bridged Adapter — 192.168.1.196)
Base OS (SO)
Oracle Linux Server 9.7
Detection Engine
Suricata IDS + Sigma Rules + Zeek + Kibana



🎯 Project Phases
Phase 1 — Lab Setup & Configuration
Installed Security Onion 2.4 in EVAL mode on VirtualBox
Configured bridged networking for host-accessible management IP
Deployed Kali Linux as attacker VM on same network segment
Verified bidirectional connectivity between VMs
Phase 2 — Network Reconnaissance (Nmap)
Executed four Nmap scan types from Kali against Security Onion:

# Ping sweep — discover live hosts

nmap -sn 192.168.1.0/24

# Aggressive scan — ports, services, scripts, traceroute

nmap -A -T4 192.168.1.196

# Service version detection

nmap -sV --version-intensity 5 192.168.1.196

# OS fingerprinting

nmap -O 192.168.1.196

Key findings: Port 22 (SSH/OpenSSH 8.7) open — 999 ports filtered — Linux OS confirmed
Phase 3 — SSH Brute Force (Hydra)
Executed dictionary-based credential attack against SSH using rockyou.txt:

hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.196 -t 4 -V -w 10

~340 attempts made before manual termination
0 valid credentials found — SSH hardened against attack
Attack generated significant authentication failure traffic
Phase 4 — Detection & Investigation (Security Onion SOC)
Switched to defender role — investigated all attack activity in Security Onion:

Detection Source
Finding
Sigma Alerts
404 HIGH severity alerts — "Grid Node Login Failure (SSH)"
Zeek DNS Logs
50 DNS query events attributed to attacker IP (192.168.1.209)
Kibana
Cross-source log correlation confirmed full attacker timeline
Alert Timestamps
2026-04-24T22:33:00Z — 2026-04-24T22:37:21Z

Phase 5 — Incident Report
Produced a formal SOC incident report documenting:

Executive summary and severity assessment
Full attack timeline with timestamps
Technical analysis of both attack phases
Indicators of Compromise (IOC) table
Detection evidence from all log sources
Remediation recommendations

📄 View Full Incident Report


🔍 Indicators of Compromise (IOCs)
IOC Type
Value
Attacker IP
192.168.1.209
Target IP
192.168.1.196
Target Port
22 (TCP/SSH)
Service Targeted
OpenSSH 8.7
Username Attacked
root
Attack Tool — Recon
Nmap 7.95
Attack Tool — Brute Force
Hydra v9.5
Wordlist Used
rockyou.txt
Total Alerts Generated
404 HIGH severity
Attack Window
2026-04-24 21:24 – 22:37 UTC



🛠️ Tools Used
Offensive (Attacker)

Nmap 7.95 — Network reconnaissance and port scanning
Hydra v9.5 — SSH credential brute force
rockyou.txt — Dictionary wordlist (14.3M passwords)

Defensive (SOC Analyst)

Security Onion 2.4 — SIEM/NSM platform
Suricata — Network intrusion detection (IDS)
Sigma — Detection rule engine (404 alerts triggered)
Zeek — Network traffic analysis and protocol logging
Kibana — Log visualization and cross-source investigation
Elasticsearch — Log indexing and search backend


🚧 Troubleshooting Documentation
This project required extensive troubleshooting during the Security Onion setup phase. Every issue encountered is fully documented as a learning reference.
Issue 1 — VERR_ALREADY_EXISTS on First Boot
Cause: VirtualBox's Unattended Installation feature auto-generated kickstart files (ks.cfg and .viso) that conflicted with Security Onion's own installer.
Fix: Deleted Unattended-*.cfg and Unattended-*.viso files from the VM directory. Recreated VM with Skip Unattended Installation checked.
Prevention: Always check Skip Unattended Installation when creating VMs for Security Onion.


Issue 2 — Forgotten Admin Username
Cause: Admin username forgotten after initial OS install.
Fix: Booted into single-user mode via GRUB editor:

# In GRUB, edit the linux line — add to end:

rw init=/bin/bash

# After booting to shell:

mount -o remount,rw /

ls /home                    # revealed username

passwd [username]           # reset password

sync

exec /sbin/init             # reboot normally


Issue 3 — Installation Failed (Insufficient Disk)
Cause: Virtual disk was 100 GB — below Security Onion's 200 GB minimum.
Fix: Deleted VM entirely and recreated with 200 GB virtual disk.
Lesson: Security Onion stores full PCAPs and ELK data — storage requirements are non-negotiable.


Issue 4 — Web UI Inaccessible (NAT Network Mode)
Cause: Adapter 1 set to NAT assigned an internal 10.0.2.x address unreachable from the Windows host browser.
Attempted: Port forwarding, Host-Only adapter, so-allow, so-firewall commands, iptables flush — none resolved it.
Fix: Changed Adapter 1 to Bridged Adapter (Realtek PCIe GbE Family Controller), then ran:

sudo so-ip-update

This updated Security Onion's internal configuration to recognize the new bridged IP (192.168.1.196).


Issue 5 — Kali VM Could Not Reach Security Onion
Cause: Initial setup used Internal Network adapter type with static IPs, but Security Onion was configured on Bridged adapter — different network segments.
Fix: Changed Kali's adapter to Bridged Adapter matching Security Onion's network mode. Both VMs received DHCP addresses from the home router on the same 192.168.1.0/24 subnet.


📋 Incident Report Summary
Incident ID: IR-2026-0424-001
Severity: HIGH
Alerts Generated: 404
Detection Engine: Sigma
Attack Duration: ~73 minutes (21:24 – 22:37 UTC)
Outcome: Attack fully detected — 0 credentials compromised

📄 View Full Incident Report
📄 View Setup Documentation


📚 Skills Demonstrated
Network intrusion detection (IDS/IPS)
SIEM alert triage and investigation
Network traffic analysis (Zeek/Bro)
Log correlation across multiple sources (Kibana/Elasticsearch)
Threat detection and incident response
Network reconnaissance techniques (Nmap)
Credential attack simulation (Hydra)
Linux system administration (Oracle Linux / Kali)
Virtual network architecture design
SOC analyst workflow — detection → investigation → documentation
Formal incident report writing


🔗 Related Projects
More projects coming soon — check back for updates.



Built and documented by Bryan Sykes | Northern Virginia | 2026
Targeting roles in: SOC Analysis | IT Support | Cybersecurity | Blue Team Operations

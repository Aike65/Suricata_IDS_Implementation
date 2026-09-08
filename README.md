# Suricata_IDS_Implementation

## Project Overview

This project demonstrates the setup, configuration, and validation of a Network Intrusion Detection System (NIDS) using **Suricata** in a controlled virtual lab environment. The lab simulates a small Security Operations Center (SOC) deployment where an attacker machine targets a victim machine, and Suricata monitors the internal network for malicious activity.

The IDS was configured with six custom detection rules to identify various attack types. Each rule was validated by generating real traffic and confirming that Suricata produced alerts.

## Lab Design

### Network Diagram

![Network Diagram](screenshots/network_diagram.png)

### Virtual Machines

| Machine | OS | Role | Internal IP | RAM | Disk |
|---|---|---|---|---|
| IDS-Server | Ubuntu Server 22.04 | Suricata IDS | 192.168.10.10 | 2 GB | 25 GB |
| Victim-Server | Ubuntu Server 22.04 | Target | 192.168.10.20 | 2 GB | 25 GB |
| Kali Attacker | Kali Linux | Attacker | 192.168.10.30 | 2 GB | 30 GB |

All VMs are connected to an **Internal Network** named `idsnet`. The IDS also has a NAT adapter for internet access.

## Installation Guide

1. Created three virtual machines in Oracle VirtualBox.
2. Installed Ubuntu Server 22.04 on IDS-Server and Victim-Server.
3. Installed Kali Linux from a pre-built VirtualBox image.
4. Configured network adapters:
   - Adapter 1: NAT (internet)
   - Adapter 2: Internal Network `idsnet`
5. Assigned static IPs on `enp0s8` (or `eth1` for Kali) using netplan.

## Suricata Installation

On IDS-Server:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install suricata -y

```
## Configuration

1. Stopped Suricata: `sudo systemctl stop suricata`
2. Backed up config: `sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata.yaml.bak`
3. Edited `/etc/suricata/suricata.yaml`:
   - Set `HOME_NET` to `[192.168.10.0/24]`
   - Set AF-PACKET interface to `enp0s8`
   - Set `rule-files` to `/etc/suricata/rules/local.rules`
4. Created `/etc/suricata/rules/local.rules` with six custom rules (see below).
5. Tested config: `sudo suricata -T -c /etc/suricata/suricata.yaml -i enp0s8`
6. Started Suricata: `sudo systemctl start suricata`

## Custom Rules

The following six rules were created in `local.rules`:

```text
alert icmp any any -> any any (msg:"ICMP Ping Detected"; sid:1000001; rev:1;)
alert tcp any any -> any 22 (msg:"SSH Connection Detected"; sid:1000002; rev:1;)
alert tcp any any -> any 80 (msg:"HTTP GET Request Detected"; content:"GET"; sid:1000003; rev:1;)
alert tcp any any -> any any (msg:"Nmap Scan Detected"; flags:S; sid:1000004; rev:1;)
alert tcp any any -> any 21 (msg:"FTP Login Attempt Detected"; content:"USER"; sid:1000005; rev:1;)
alert tcp any any -> any any (msg:"Custom Malware Test String Detected"; content:"MALWARE"; sid:1000006; rev:1;)
```

## Testing Procedures

For each rule, traffic was generated from the Kali attacker to the Victim:

- **ICMP:** `ping -c 4 192.168.10.20`
- **SSH:** `ssh victimadmin@192.168.10.20`
- **HTTP GET:** `curl http://192.168.10.20` (with Python HTTP server on victim)
- **Nmap Scan:** `sudo nmap -sS 192.168.10.20`
- **FTP Login:** `ftp 192.168.10.20` (with vsftpd running)
- **Malware String:** `echo "MALWARE-TEST-STRING" | nc 192.168.10.20 9999`

Alerts were confirmed in `/var/log/suricata/fast.log`.

## Validation Results

All six rules triggered alerts as expected. Screenshots of alerts are available in the `screenshots/` directory.

## Lessons Learned

- Suricata requires correct network interface configuration and promiscuous mode to capture traffic between other machines.
- YAML indentation is strict; small errors prevent netplan and Suricata from working.
- Simpler rules using content matching can be more reliable than protocol-specific keywords if Suricata's parser is not detecting the traffic.
- Restarting Suricata after config changes is essential.

## References

- Suricata Documentation: https://suricata.readthedocs.io/
- Ubuntu Server Guide: https://ubuntu.com/server/docs
- Kali Linux: https://www.kali.org/

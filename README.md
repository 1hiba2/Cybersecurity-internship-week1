#  Cybersecurity Internship — Week 1

**Networkwalks Academy | Cybersecurity & SOC Analyst Training**

##  Project Overview

This repository documents my Week 1 cybersecurity internship projects with Networkwalks Academy.

The repository includes two practical cybersecurity projects:

1. **Project 1 — VirtualBox and Kali Linux Lab Setup:** Creating and configuring a virtual cybersecurity testing environment.
2. **Project 2 — Wireshark Lab 3: Identify the Attacker's IP:** Investigating network traffic, identifying suspicious TCP packets, and detecting port scanning activity.

These projects focus on developing hands-on skills in virtualization, Linux networking, packet analysis, and Security Operations Center (SOC) investigations.

---

#  Project 1 — VirtualBox and Kali Linux Lab Setup

## Objective

Build a functional cybersecurity laboratory using Oracle VirtualBox and Kali Linux, configure a NAT Network, assign a static IP address, verify internet connectivity, enable shared folders, and create a virtual machine snapshot.

## Tools Used

- Oracle VirtualBox
- Kali Linux 2026.3
- Windows host operating system
- 7-Zip
- Linux Terminal
- NetworkManager

## 1. Virtual Machine Setup

Downloaded and configured the official Kali Linux pre-built virtual machine in Oracle VirtualBox.

An existing Windows 11 virtual machine was preserved for additional cybersecurity exercises.

## 2. Network Configuration

Created a dedicated NAT Network and assigned Kali Linux a static IPv4 address.

| Setting | Value |
|---|---|
| Network Mode | NAT Network |
| Subnet | `10.0.0.0/24` |
| Kali IP Address | `10.0.0.2/24` |
| Default Gateway | `10.0.0.1` |
| DNS Servers | `8.8.8.8`, `1.1.1.1` |

### Static IP Configuration

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.method manual ipv4.addresses 10.0.0.2/24
sudo nmcli connection modify "Wired connection 1" ipv4.gateway 10.0.0.1
sudo nmcli connection modify "Wired connection 1" ipv4.dns "8.8.8.8 1.1.1.1"
sudo nmcli connection up "Wired connection 1"
```

Initially, Kali Linux experienced network connection issues. These were resolved by configuring the static IP address manually.

## 3. Network Verification

Verified the network configuration using:

```bash
ip -4 addr show eth0
ip route
```

Tested connectivity using:

```bash
ping -c 4 10.0.0.1
ping -c 4 8.8.8.8
ping -c 4 google.com
```

### Results

- Kali Linux successfully obtained IP address `10.0.0.2/24`.
- The NAT gateway was reachable.
- Internet connectivity worked successfully.
- DNS resolution was successful.
- The connectivity tests showed 0% packet loss.

## 4. Clipboard and Drag-and-Drop

Configured the following VirtualBox settings:

- Shared Clipboard: **Bidirectional**
- Drag-and-Drop: **Bidirectional**

The drag-and-drop service was running, but file transfers encountered a `VERR_TIMEOUT` error.

The feature remained enabled, and the shared folder was used as an alternative file-transfer method.

## 5. Shared Folder Configuration

Configured a shared folder called `downloads` with the Kali Linux mount point:

```text
/downloads
```

Verified file sharing using:

```bash
ls -la /downloads
cat /downloads/test.txt.txt
```

The test successfully displayed:

```text
hello from windows to kali
```

This confirmed that Kali Linux could access files shared from the Windows host.

## 6. Virtual Machine Snapshot

Created a VirtualBox snapshot named:

```text
week1-kali-configured
```

The snapshot preserves the working Kali Linux laboratory configuration for future use.

## Project 1 Outcome

Successfully configured a Kali Linux virtual machine with static IP addressing, functional NAT networking, internet access, shared folders, and a saved snapshot.

This project strengthened my practical knowledge of virtualization, Linux administration, network troubleshooting, and host-to-guest integration.

---

#  Project 2 — Wireshark Lab 3: Identify the Attacker's IP

## Objective

Investigate captured network traffic using Wireshark to identify suspicious TCP connection attempts, determine the attacker and target, analyze open and closed ports, and classify the network attack.

## Tools Used

- Networkwalks Wireshark browser simulation
- Wireshark display filters
- TCP/IP packet analysis

## Investigation Results

| Investigation Detail | Finding |
|---|---|
| Total Captured Packets | 309 |
| SYN-Flagged Packets | 73 |
| Attacker IP Address | `192.168.10.66` |
| Target IP Address | `192.168.10.20` |
| Attacker Source Port | `52311` |
| Open Ports | `22, 80, 443` |
| Closed Port Response | `RST, ACK` |
| Observed Closed Ports | 17 |
| Attack Type | TCP Port Scan |

**Note:** The number of distinct scanned ports should be confirmed against the accepted Task 6 result. Based on three open ports and 17 closed ports, the estimated total is 20, assuming all scanned ports received responses.

## Investigation Process

### Task 1 — Identify Total Captured Packets

Opened the browser-based Wireshark simulation and inspected the network capture.

**Result:** 309 packets.

### Task 2 — Identify TCP SYN Packets

Applied the Wireshark display filter:

```wireshark
tcp.flags.syn == 1
```

**Result:** 73 packets with the SYN flag set.

### Task 3 — Identify the Attacker's IP

Inspected the Source column and identified repeated SYN packets originating from `192.168.10.66`.

Used the filter:

```wireshark
ip.src == 192.168.10.66 && ip.dst == 192.168.10.20 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```

**Result:** Attacker IP address `192.168.10.66`.

### Task 4 — Identify the Target IP

Examined the Destination column of the suspicious packets.

**Result:** Target IP address `192.168.10.20`.

### Task 5 — Identify the Attacker's Source Port

Inspected the Info column and identified the source port repeatedly used by the attacker.

**Result:** `52311`.

### Task 6 — Investigate Scanned Ports

Examined the destination ports targeted by the attacker's TCP SYN requests.

**Finding:** Multiple distinct destination ports were scanned. The exact accepted total requires confirmation.

### Task 7 — Identify Open Ports

Applied:

```wireshark
ip.src == 192.168.10.20 && ip.dst == 192.168.10.66 && tcp.flags.syn == 1 && tcp.flags.ack == 1
```

The target responded with SYN-ACK packets from the following ports:

- **22** — SSH
- **80** — HTTP
- **443** — HTTPS

**Result:** `22, 80, 443`.

### Task 8 — Identify Closed-Port TCP Flags

Examined the server responses to connection attempts targeting closed ports.

**Result:** `RST, ACK`.

### Task 9 — Count Closed Ports

Applied:

```wireshark
ip.src == 192.168.10.20 && ip.dst == 192.168.10.66 && tcp.flags.reset == 1
```

**Observed result:** 17 closed ports.

### Task 10 — Identify the Attack Type

Analyzed the repeated TCP SYN requests directed at multiple destination ports.

**Result:** TCP Port Scan.

## Attack Analysis

The captured traffic showed repeated TCP SYN requests originating from `192.168.10.66` and targeting `192.168.10.20`.

The attacker attempted connections to multiple destination ports using the same source port.

The target returned:

- **SYN-ACK** responses for open ports.
- **RST-ACK** responses for closed ports.

This pattern is characteristic of TCP SYN port scanning, a network reconnaissance technique used to discover accessible services.

## Lab 3 Conclusion

The investigation identified suspicious TCP port scanning activity and demonstrated how packet-level evidence can be used to determine the attacking host, target server, and accessible services.

The lab strengthened my understanding of Wireshark display filters, TCP flags, network traffic investigation, and basic SOC analysis.

---

#  Skills Developed

- Oracle VirtualBox installation and configuration
- Kali Linux administration
- Static IPv4 addressing
- NAT networking
- Network connectivity troubleshooting
- Virtual machine snapshots
- Shared folder configuration
- Wireshark packet analysis
- TCP SYN and RST flag interpretation
- Source and destination IP analysis
- Open and closed port identification
- Network reconnaissance detection
- Cybersecurity investigation documentation

---

#  Repository Structure

```text
Cybersecurity-internship-week1/
│
├── README.md
│
├── project-1-kali-virtualbox/
│   └── screenshots/
│       ├── 01-kali-network.jpeg
│       ├── 02-internet-test.jpeg
│       ├── 03-clipboard-dragdrop.jpeg
│       ├── 04-shared-folder-test.jpeg
│       └── 05-kali-snapshot.jpeg
│
└── project-2-wireshark/
    └── Lab-3-Attacker-IP/
        └── screenshots/
```

The screenshots document network configuration, successful connectivity, shared folder access, virtual machine snapshots, and Wireshark investigation evidence.

---

 Final Conclusion

These Week 1 internship projects provided practical experience in both cybersecurity laboratory setup and network security analysis.

The first project established a functional Kali Linux environment using Oracle VirtualBox.

The second project demonstrated how Wireshark can be used to investigate suspicious TCP traffic, identify attackers, analyze port states, and recognize port scanning activity.

Together, these projects helped strengthen my foundational skills in Linux, networking, cybersecurity troubleshooting, and SOC investigations.

---

**Training Provider:** Networkwalks Academy

**Platform:** https://networkwalks.com/

**Purpose:** Educational cybersecurity training in authorized laboratory environments.# Cybersecurity-internship-week1
Week 1 cybersecurity internship projects covering VirtualBox and Kali Linux lab setup, network configuration, and Wireshark SOC traffic analysis.

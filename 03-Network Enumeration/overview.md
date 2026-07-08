# Module 3: Network Enumeration with Nmap

## Nmap Overview

Open-source network analysis and security auditing tool.

**Use Cases**
- Audit network security
- Simulate penetration tests
- Check firewall and IDS configurations
- Identify open ports and services
- Vulnerability assessment

**Syntax**

```bash
nmap <scan types> <options> <target>
nmap --help
```

**TCP-SYN Scan — default scan type**

```bash
sudo nmap -sS localhost
```

- SYN-ACK response → port **open**
- RST response → port **closed**
- No response → port **filtered**

---

## Host Discovery

### Scan Network Range

```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

| Flag | Description |
|------|-------------|
| 10.129.2.0/24 | Target network range |
| -sn | Disable port scanning |
| -oA tnet | Save results in all formats as 'tnet' |

---

### Scan from IP List

```bash
cat hosts.lst
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```

| Flag | Description |
|------|-------------|
| -iL | Scan targets from provided list |

> If only 3 of 7 hosts appear active, remaining hosts likely block ICMP echo requests via firewall.

---

### Scan Multiple IPs

```bash
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20 | grep for | cut -d" " -f5
```

**Consecutive range shorthand**

```bash
sudo nmap -sn -oA tnet 10.129.2.18-20 | grep for | cut -d" " -f5
```

---

### Scan Single IP

```bash
sudo nmap 10.129.2.18 -sn -oA host
```

**Force ICMP echo requests**

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```

| Flag | Description |
|------|-------------|
| -PE | Use ICMP echo requests instead of ARP |
| --packet-trace | Show all packets sent and received |

**Show reason host is marked alive**

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --reason
```

**Disable ARP, force ICMP only**

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

---

### Default TTL Values
| TTL | OS |
|-----|----|
| 64 | Linux / Unix / macOS |
| 128 | Windows |
| 254 | Cisco Devices |
| 255 | Solaris / FreeBSD / OpenBSD |

---

### Host & Port States
| State | Meaning |
|-------|---------|
| open | Connection established (TCP/UDP/SCTP) |
| closed | RST flag received — port closed, host alive |
| filtered | No response or error code — firewall likely dropping packets |
| unfiltered | TCP-ACK scan only — accessible but open/closed undetermined |
| open\|filtered | No response — firewall/packet filter may be protecting port |
| closed\|filtered | IP ID idle scans only — cannot determine state |

---

### Discovering Open TCP Ports

| Method | Flag | Notes |
|--------|------|-------|
| SYN scan | -sS | Default when run as root |
| TCP scan | -sT | Default without root |
| Specific ports | -p 22,25,80 | Define 1-by-1 |
| Port range | -p 22-445 | By range |
| Top ports | --top-ports=10 | Most common ports |
| All ports | -p- | Full scan, all 65535 |
| Fast scan | -F | Top 100 ports |

```bash
sudo nmap 10.129.2.28 --top-ports=10
```

---

### Packet Tracing

For a clear view of SYN scan — disable ICMP echo requests, DNS resolution, ARP ping:
```bash
sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping
```

**Request breakdown:**
| Field | Meaning |
|-------|---------|
| SENT (0.0429s) | Nmap sends packet to target |
| TCP | Protocol used |
| 10.10.14.2:63090 | Our IPv4 addr and source port |
| 10.129.2.28:21 | Target IPv4 addr and port |
| S | SYN flag |
| ttl=56 id=57322 iplen=44 | Additional TCP header params |

**Response breakdown:**
| Field | Meaning |
|-------|---------|
| RCVD (0.0573s) | Received packet from target |
| 10.129.2.28:21 | Target IPv4 addr and source port |
| 10.10.14.2:63090 | Our IPv4 addr and port replied to |
| RA | RST + ACK flags |
| ttl=64 id=0 iplen=40 | Additional TCP header params |

---

### Connect Scan vs SYN Scan

**Connect Scan (-sT) — Full TCP 3-way handshake**
- Least stealthy — creates logs on most systems, easily detected by IDS/IPS
- Goal: accuracy — maps network cleanly without disrupting services
```bash
sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT
```

**SYN Scan (-sS) — Half-open scan**
- More stealthy — does not complete full handshake
- Minimises connection logs while gathering port info

---

### Filtered Ports
- Packet dropped → no response → Nmap retries (default `--max-retries` = 10)
- Firewall drops TCP packets → filtered → ICMP type 3, error code 3 → port unreachable
- If host is alive but port filtered → strongly assume firewall is rejecting packets

---

### Discovering Open UDP Ports

UDP = stateless protocol, no 3-way handshake, no acknowledgement.

```bash
sudo nmap 10.129.2.28 -F -sU
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason
sudo nmap 10.129.2.28 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason -sV
```

- Often no response — Nmap sends empty datagrams, cannot confirm delivery
- UDP port open → response only if application configured to respond
- ICMP response with error code 3 → port **unreachable** → closed

---

### Saving & Exporting Nmap Results

| Flag | Extension | Format |
|------|-----------|--------|
| -oN | .nmap | Normal human-readable output |
| -oG | .gnmap | Grepable output |
| -oX | .xml | Machine-readable XML |
| -oA | all three | Save in all formats simultaneously |

```bash
sudo nmap 10.129.2.28 -p- -oA target
ls
cat target.nmap
cat target.gnmap
cat target.xml
```

**Convert XML to HTML:**
```bash
xsltproc target.xml -o target.html
```
Open in browser for structured, readable representation.

> Always save scan results with `-oA`. Scan data is evidence — critical for documentation, reporting, and recreating attack paths.

---

### Service Version Detection

Perform quick port scan first to reduce traffic, then enumerate versions:
```bash
sudo nmap 10.129.2.28 -p- -sV
```

| Technique | Command | Purpose |
|-----------|---------|---------|
| Check scan status | Press Space Bar | View progress mid-scan |
| Status interval | --stats-every=5s | Print status every 5s (s=seconds, m=minutes) |
| Verbosity | -v or -vv | Show open ports as detected in real time |

```bash
sudo nmap 10.129.2.28 -p- -sV --stats-every=5s
sudo nmap 10.129.2.28 -p- -sV -v
```

---

### Banner Grabbing

- Automatic scans can miss info banners don't always expose version clearly
- Banners sent at network level — identifiable via **PSH flag** in TCP header
- Banner example: `220 inlane ESMTP Postfix (Ubuntu)` — reveals service and OS

```bash
sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28
nc -nv 10.129.2.28 25
```

**TCP Handshake + Banner Flow:**
| Flag | Direction | Meaning |
|------|-----------|---------|
| SYN | Client → Target | Initiate connection |
| SYN-ACK | Target → Client | Connection acknowledged |
| ACK | Client → Target | Confirm |
| PSH-ACK | Target → Client | Target sending data (banner) |
| ACK | Client → Target | Data received |

---

### Nmap Scripting Engine (NSE)

Scripts written in Lua. 14 categories:

| Category | Purpose |
|----------|---------|
| auth | Determine authentication credentials |
| broadcast | Host discovery added to remaining scans |
| brute | Brute-force login attempts |
| default | Default scripts run with -sC |
| discovery | Evaluate accessible services |
| dos | Check services for DoS vulnerabilities (use sparingly) |
| exploit | Exploit known vulnerabilities on scanned port |
| external | Use external services for further processing |
| fuzzer | Identify vulnerabilities via unexpected packet handling |
| intrusive | Scripts that may negatively affect target |
| malware | Check if malware affects target system |
| safe | Defensive scripts — non-intrusive, non-destructive |
| version | Extension for service detection |
| vuln | Identify specific vulnerabilities |

```bash
# Run by category
sudo nmap  --script 

# Run specific scripts
sudo nmap  --script ,

# Example: SMTP enumeration with 2 scripts
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands
```

---

### Aggressive Scan

Performs service detection, OS detection, traceroute, and default scripts in one command:
```bash
sudo nmap 10.129.2.28 -p 80 -A
```

---

### Vulnerability Assessment

```bash
sudo nmap 10.129.2.28 -p 80 -sV --script vuln
```
Runs all scripts from the `vuln` category against specified port.

---

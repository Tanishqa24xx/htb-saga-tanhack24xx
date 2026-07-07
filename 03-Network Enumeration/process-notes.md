# Module 3: Network Enumeration with Nmap

## Section 3 — OS Detection via TTL

### Exercise
Based on a given Nmap scan result, identify the operating system.

### Process
- Examined the given Nmap scan output
- Referenced [Nmap Host Discovery Strategies](https://nmap.org/book/host-discovery-strategies.html)
- Identified TTL as the key differentiator — request TTL 255, reply TTL 128
- Confirmed via hint that OS detection via TTL was the intended approach

### Common Default TTL Values

| TTL | OS |
|-----|----|
| 64 | Linux / Unix / macOS |
| 128 | Windows |
| 254 | Cisco Devices |
| 255 | Solaris / FreeBSD / OpenBSD |

Reply TTL of 128 → **Windows**

### Key Takeaway
Enumeration is everything. TTL values alone can reveal the target OS before a single port is scanned.

---

## Section 4 — Discovering Open TCP Ports & Hostname Enumeration

### Exercise 1
Find all TCP ports on target. Submit total number found.

### Process
- Ran full TCP port scan with version detection and packet tracing:
```bash
sudo nmap 10.129.86.249 -p- --packet-trace -Pn -n --disable-arp-ping -sV
```
- Found **7 open TCP ports**

### Key Flags Used
| Flag | Description |
|------|-------------|
| -p- | Scan all 65535 ports |
| --packet-trace | Show all packets sent and received |
| -Pn | Disable host discovery (skip ping) |
| -n | Disable DNS resolution |
| --disable-arp-ping | Disable ARP ping |
| -sV | Service/version detection |

---

### Exercise 2
Enumerate hostname of target. Submit as answer (case-sensitive).

### Process
- Ran default script scan on target:
```bash
sudo nmap -sC 10.129.86.249
```
- Found hostname under `smb-os-discovery` output — listed as **Computer Name**
- Key insight: SMB enumeration reveals OS and hostname info beyond just open ports

### Key Takeaway
Always run `-sC` (default scripts) alongside `-sV`. SMB scripts reveal hostname, OS version, domain info — critical for later exploitation.

---

## Section 5 — Full TCP Port Scan & HTML Report Generation

### Exercise
Perform full TCP port scan, create HTML report. Submit highest port number found.

### Process
- Identified highest port from previous scan results
- To generate HTML report, re-ran full scan saving all output formats:
```bash
sudo nmap  -p- -oA target
```
- Verified files created:
```bash
ls
cat target.nmap
cat target.gnmap
cat target.xml
```
- Converted XML to HTML:
```bash
xsltproc target.xml -o target.html
```
- Opened in browser — structured, readable representation of all scan results

### Output Formats Reference
| Flag | Extension | Use |
|------|-----------|-----|
| -oN | .nmap | Human-readable normal output |
| -oG | .gnmap | Grep-friendly output |
| -oX | .xml | Machine-readable, convertible to HTML |
| -oA | all three | Save in all formats simultaneously |

### Key Takeaway
Always save scan results with `-oA`. Scan data is evidence — useful for documentation, reporting, decision-making, and recreating attack paths. Never rely on terminal history alone.

---


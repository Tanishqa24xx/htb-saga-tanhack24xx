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

## Section 6 — Service Enumeration Exercise

### Exercise
Enumerate all ports and services. One service contains the flag.

### Process
- Ran full port scan with version detection:
```bash
sudo nmap  -p- -sV
```
- Flag found embedded in service info field of scan output

### Key Takeaway
Always read full scan output carefully — flags and sensitive info can appear in service banners and version fields, not just open ports.

---

## Section 7 — NSE Vulnerability Script Exercise

### Exercise
Use NSE scripts to find the flag hidden in one of the services.

### Process
- Analysed Section 6 scan output — port 80 (HTTP) identified as highest probability target due to user-facing nature
- Ran vulnerability category scripts against port 80:
```bash
sudo nmap  -p 80 -sV --script vuln
```
- Scan revealed existence of `robots.txt`
- Visited `http://<ip>:80/robots.txt` in browser — flag printed directly

### Key Takeaway
`robots.txt` is a classic first stop — sites use it to tell search engines what not to index, which inadvertently reveals hidden paths and sensitive endpoints. Always check it during web enumeration.

---

## Section 10 — OS Detection (Stealth Required)

### Exercise
Identify the operating system of the target. Submit OS name.

### Process
- Ran comprehensive scan:
```bash
sudo nmap 10.129.96.157 -sS -sV -sC -Pn -n --disable-arp-ping
```
- Found **Ubuntu** server via open port 22 (SSH banner)
- Triggered IDS alert — banned for 3 minutes
- Key lesson: standard scans are loud — IDS/IPS detected immediately

### Key Takeaway
When stealth is required, avoid `-sC` (runs intrusive scripts) and `-sV` (generates extra traffic). Use decoys (`-D RND:5`) or timing flags (`-T 1`) to reduce detection footprint.

---

## Section 11 — DNS Server Version (Silent UDP Scan)

### Exercise
Find the DNS server version of the target. Submit as answer.

### Process
- Targeted UDP port 53 (DNS) with silent scan:
```bash
sudo nmap 10.129.98.111 -sU -Pn -p 53 -n --disable-arp-ping
```
- Confirmed port open
- Added `-sV` for version detection — flag returned directly:
```bash
sudo nmap 10.129.98.111 -sU -Pn -p 53 -n --disable-arp-ping -sV
```

### Key Takeaway
DNS runs on UDP port 53 by default. Always target UDP specifically with `-sU` — it won't appear in TCP scans.

---

## Section 12 — Firewall Bypass via DNS Proxying

### Exercise
Find the version of a service hidden behind a firewall. Submit flag.

### Process
- Initial SYN scan revealed ports 22 (SSH) and 80 (HTTP):
```bash
sudo nmap 10.129.98.122 -sS -Pn -n --disable-arp-ping
```
- TCP scan produced same results — nothing new
- UDP scan revealed ports 68, 137, 138 — not the target
- Suspected firewall filtering — tried DNS proxy method:
```bash
# Scan all ports using source port 53
sudo nmap 10.129.98.122 -sS -Pn -n --disable-arp-ping --source-port 53
```
- Revealed hidden filtered port **50000** — service: `ibm-db2` (database service)
- Confirmed port open via targeted scan:
```bash
sudo nmap 10.129.98.122 -p50000 -sS -Pn -n --disable-arp-ping --source-port 53
```
- Attempted Netcat connection — failed initially:
```bash
ncat -nv --source-port 53 10.129.98.122 50000  # failed
sudo ncat -nv --source-port 53 10.129.98.122 50000  # still failed
```
- Found fix via cheatsheet — must specify source IP explicitly:
```bash
sudo ncat -s  -p 53 10.129.98.122 50000
```
- Flag returned directly in connection banner

### Key Takeaway
When ports appear filtered, try `--source-port 53` — firewalls often trust DNS traffic. If Netcat fails without sudo or without `-s`, always specify your tun0 interface IP explicitly using `-s`. The source IP matters, not just the source port.


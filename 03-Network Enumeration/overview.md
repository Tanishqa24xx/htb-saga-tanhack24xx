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

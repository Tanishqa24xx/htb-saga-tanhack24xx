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


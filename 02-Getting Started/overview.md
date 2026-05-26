# Module 2 — Overview

## Summary
- Risk assessment is a major part of penetration testing.
- Learnt a new word: **distro** — meaning Linux distributions.
- Set up and started working with **Parrot OS**.
- Importance of maintaining a proper folder structure. Organisation matters as much as technical skills.

### Example Folder Structure
TanHack24xx@htb[/htb]$ tree Projects/
Projects/

└── Acme Company      
├── EPT      
│   ├── evidence      
│   │   ├── credentials      
│   │   ├── data      
│   │   └── screenshots      
│   ├── logs      
│   ├── scans      
│   ├── scope      
│   └── tools      
└── IPT      
├── evidence      
│   ├── credentials      
│   ├── data      
│   └── screenshots      
├── logs       
├── scans      
├── scope      
└── tools      

## Summary (continued)
- Overview of VPNs. Types: **client-based** and **SSL VPN**. Learnt commands to connect to VPN.
- Revision on shells, ports, and web servers. OWASP Top 10 is extremely important — assessors always check it first.
- Basic tools introduced:
  1. **SSH** — remotely access systems securely; uses client–server model.  
     Example: `ssh bob@10.10.10.10`
  2. **Netcat** — interact with TCP/UDP ports.  
     - Connect: `nc 10.10.10.10 22`  
     - Banner grab: `nc -nv 10.129.42.253 21`
  3. **Socat** — similar to Netcat but supports port forwarding and serial device connections.
  4. **Terminal multiplexers (tmux / screen)** — multiple windows inside one terminal.  
     Install: `sudo apt install tmux -y`
  5. **Vim** — keyboard‑driven text editor.  
     Open file: `vim /etc/hosts`

- Service scanning and coercing servers into unintended actions.
- Introduction to **Nmap**:
  - Basic scan: `nmap 10.10.10.10`
  - `-sC` — run default scripts  
  - `-sV` — version scan  
  - `-p-` — scan all 65,535 TCP ports  
  - Example: `nmap -sV -sC -p- 10.129.42.253`
  - Run script: `nmap --script <script> -p<port> <host>`
  - Banner grab: `nmap -sV --script=banner <target>`

- **FTP**:
  - Nmap scan reveals installation, anonymous login, pub directory.
  - Connect: `ftp -p 10.129.42.253`
  - Commands: `cd`, `ls`, `get file.txt`, `cat file.txt`

- **SMB (Server Message Block)**:
  - Protocol on Windows; used for vertical and lateral movement.
  - Scan: `nmap --script smb-os-discovery.nse -p445 10.10.10.40`
  - Aggressive scan: `nmap -A -p445 10.129.42.253`
  - Shares:
    - List shares: `smbclient -N -L \\\\10.129.42.253`
    - Connect as guest: `smbclient \\\\10.129.42.253\\users`
    - Connect as user: `smbclient -U bob \\\\10.129.42.253\\users`

- **SNMP**:
  - Community strings provide device info. Defaults: `public`, `private`.
  - `onesixtyone` can brute‑force community strings.
  - Examples:
    - `snmpwalk -v2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0`
    - `snmpwalk -v2c -c private 10.129.42.253`
    - `onesixtyone -c dict.txt 10.129.42.254`
   
- **Web Enumeration Techniques**
1. **Gobuster**  
   - High‑performance file/directory, DNS, and virtual host brute‑forcing tool.  
   - Used to discover hidden directories and files on web servers.  
   - Simple scan example:  
     `gobuster dir -u http://10.10.10.121/ -w <wordlist>`  
   - DNS Subdomain Enumeration:  
     - Install SecLists:  
       `git clone https://github.com/danielmiessler/SecLists`  
       `sudo apt install seclists -y`  
     - Use SecLists with Gobuster:  
       `gobuster dns -d inlanefreight.com -w <seclists-wordlist>`

2. **HTTP Status Codes**  
   - 200 = successful request  
   - 403 = forbidden  
   - 301 = redirection  

3. **cURL**  
   - Retrieve server header information from the command line.  
   - Example:  
     `curl -IL https://www.inlanefreight.com`

4. **WhatWeb**  
   - Command‑line tool to extract versions of web servers, supporting frameworks, and applications.  
   - Basic usage:  
     `whatweb 10.10.10.121`  
   - Scan a whole subnet:  
     `whatweb --no-errors 10.10.10.0/24`

5. **SSL/TLS Certificate**  
   - Reveals information such as email, company name, etc.  
   - Could potentially be used for phishing if within the scope of assessment.

6. **robots.txt**  
   - Instructs search engine crawlers on which resources can or cannot be indexed.  
   - May reveal locations of private files and admin pages.

7. **Source Code Review**  
   - Using CTRL+U to view page source.  
   - Can reveal developer comments containing test credentials or plugin versions.

8. **Public Exploit Search**  
   - Next step is to look for public exploits available.  
   - Searchsploit can be used.  
     - Install: `sudo apt install exploitdb -y`  
     - Search for a specific application: `searchsploit openssh 7.2`  
   - Places to look for exploits:  
     - https://www.exploit-db.com/  
     - https://www.rapid7.com/db/  
     - https://www.vulnerability-lab.com/

9. **Metasploit Primer**  
   - Tool containing many built‑in exploits for public vulnerabilities.  
   - Run: `msfconsole`  
   - Search target application: `search exploit eternalblue`  
   - Search filters: `search cve:2009 type:exploit`  
   - Copy full module name and load it using `use`:  
     `use exploit/windows/smb/ms17_010_psexec`  
   - View configuration options: `show options`  
   - RHOSTS = target IP (single/multiple/file with list of IPs)  
   - LHOST = attack host IP (single or name of network interface)  
   - Set before exploitation:  
     `set RHOSTS 10.10.10.40`  
     `set LHOST tun0`  
   - Check server vulnerability: `check`  
   - Run/exploit: `exploit`

## Key Concepts
- Linux systems & utility tools  
- SSH  
- Netcat  
- Banner grabbing  
- Nmap  
- FTP  
- SMB
- Web Enumeration Techniques: Gobuster, HTTP status codes, cURL, WhatWeb, SSL/TLS certificate, robots.txt, source code.
- Public exploit discovery and validation.
- Using Metasploit for vulnerability assessment.


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

## Key Concepts
- Linux systems & utility tools  
- SSH  
- Netcat  
- Banner grabbing  
- Nmap  
- FTP  
- SMB  


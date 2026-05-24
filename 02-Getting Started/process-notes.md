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


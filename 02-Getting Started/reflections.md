# Module 2 — Reflections

- I’ve used `sudo` many times before, but now I understand what it actually does — it tells the host to run the command as the elevated root user.
- Learnt that **Bash** is an enhanced version of **sh**, which was the original Unix shell.
- Banner grabbing is a new concept for me — identifying what service is running on a port (quick fingerprinting).
- When Nmap shows a port as **filtered**, it means a firewall only allows access from specific IPs.
- Port **3389 (TCP)** — RDP — is a strong indicator that the target is a Windows machine.
- Some SMB versions are vulnerable to RCE exploits like **EternalBlue**.
- Learnt what EternalBlue is:
  - Series of Microsoft vulnerabilities.
  - Exploit created by the NSA.
  - Affects Windows systems using SMBv1.
  - Was leaked and used in major cyberattacks like WannaCry and Petya.
- Understanding how EternalBlue was developed, leaked, and weaponized gave me context on real‑world impact.
- GoBuster is another tool I didn’t know that is used for web directory/file enumeration, DNS subdomain discovery, virtual host detection, cloud storage enumeration, TFTP file discovery, and custom fuzzing.  
- Different ways we can use the web enumeration techniques.  
- In the Public Exploit exercise, I was made to think critically and not just use things as they are given. I learnt how one thing can be used differently based on analysis. Approaches are different for every task. I was able to dig deep into web enumeration and learnt what to look for.

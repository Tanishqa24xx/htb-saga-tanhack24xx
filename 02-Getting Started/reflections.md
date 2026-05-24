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


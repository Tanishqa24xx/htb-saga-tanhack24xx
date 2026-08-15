# Flag Command

**Difficulty:** Very Easy

---

## Recon
- First thing that came to my mind was Nmap and recon, so I started by checking the target IP.
- Ran a basic host discovery scan:
```
sudo nmap 154.57.164.82 -sn 
```
- The scan confirmed that the host was up.
- Tried another scan with ICMP ping and packet tracing:
'''
sudo nmap 154.57.164.82 -sn -oA host -PE --packet-trace --disable-arp-ping
'''
- This did not give me any useful information.
- Since the challenge sounded interactive, I thought terminal-based recon might not be the main approach.
- Opened the website in the browser:
```
http://154.57.164.82:30833/
```
- The game started successfully.

## Vulnerability Identification
- I played the game for a while, but it did not seem like the flag would simply appear by completing it normally.
- I was getting frustrated with the wrong choices and having to repeatedly enter prompts.
- Decided to check the page source.
- Nothing useful stood out in the JavaScript.
- I then inspected the page and opened the Network tab in Developer Tools.
- Reloaded the browser and saw several connections being made.
- One of them was an options endpoint.
- Visited the endpoint and found all the answer options used by the game.
- There was also a secret option containing:
```
Blip-blop, in a pickle with a hiccup! Shmiggity-shmack
```

## Exploitation
- I restarted the game.
- At the first direction prompt, I entered the secret phrase I had found:
```
Blip-blop, in a pickle with a hiccup! Shmiggity-shmack
```
- The flag was displayed and the game was completed.

## Key Takeaways
- The main lesson from this challenge was to look beyond what is shown directly on the webpage.
- When the normal interaction was not leading anywhere, checking the browser's Network tab revealed an endpoint containing information that was not obvious from the game itself.
- This was a good reminder to investigate how a web application communicates with the backend instead of relying only on the visible interface.

---

## Proof of Completion

> <img height="300" alt="image" src="https://github.com/user-attachments/assets/dcb44fd4-5db7-4f09-b726-848b63d07b64" />

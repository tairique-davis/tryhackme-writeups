# TryHackMe — Hydra

---
## What is Hydra?

Hydra (THC-Hydra) is a fast, parallelized online password cracking tool used in penetration testing to audit authentication services for weak credentials. Unlike offline hash crackers (e.g. Hashcat, John the Ripper), Hydra works against live services — it sends real login attempts over the network until valid credentials are found.

It supports a wide range of protocols including SSH, FTP, HTTP, HTTPS, SMB, RDP, and more, making it one of the most versatile credential attack tools in a pentester's kit.

---

## Syntax & Usage

### General Syntax

```bash
hydra -l <username> -P <wordlist> <TARGET_IP> <service>
```

Key flags:
- `-l` — single username
- `-L` — username wordlist
- `-p` — single password
- `-P` — password wordlist
- `-t` — number of parallel threads
- `-V` — verbose output (shows each attempt)

---

### Attacking an HTTP POST Form

HTTP POST form attacks require the `http-post-form` module. You need three pieces of information before running it: 
- Login endpoint path
- POST body parameter names
- Exact failure response string the application returns on a failed login

I used **Burp Suite's Intercept** to capture a failed login request and confirm the POST parameter names and failure response before building the command.

```bash
hydra -l <username> -P <wordlist> <TARGET_IP> http-post-form \
  "/<login_path>:username=^USER^&password=^PASS^:F=<failure string>" \
  -t 4 -V
```

#### Breakdown
- `/login_path` → target endpoint
- `^USER^` and `^PASS^` → Hydra placeholders
- `F=` → failure condition string tells Hydra what a *failed* login looks like — if this is wrong, every attempt reads as a success

---

### Attacking SSH

SSH is a first-class Hydra module — no additional syntax needed. Keep threads low with `-t 4` since many SSH configs rate-limit or drop parallel connections.

```bash
hydra -l <username> -P <wordlist> <TARGET_IP> ssh -t 4 -V
```

---

## Notes & Lessons Learned

### Burp Suite intercept before you run anything

Before touching Hydra, I intercepted a failed login attempt in Burp Suite to confirm the exact POST parameter names and the failure string the app returns. Guessing these leads to either a broken command or a flood of false positives. Getting this right first saves a lot of noise.

<img width="40%" src="https://github.com/user-attachments/assets/dc97631a-0f9f-45b4-9378-a6f5acde3d56" />

---
### The failure string has to be exact

The `F=` value in the `http-post-form` module needs to match the application's failure response character for character. Even a slight mismatch causes Hydra to treat every attempt as successful. Always verify from an actual failed request — not from memory or assumption. In our case the inital failed login validated the target's failure response: 
<img width="40%" src="https://github.com/user-attachments/assets/08d73ea0-ae8a-4d71-9b96-1ab8629b53fb" />

---
### HTTP POST form crack

With parameters confirmed in Burp, I executed Hydra against the login form and successfully recovered valid credentials.

After logging in, I accessed the authenticated page and retrieved the first flag:
**Flag 1:** `THM{2673a7dd116de68e85c48ec0b1f2612e}`

#### Hydra cracking the web login form
<img width="40%" src="https://github.com/user-attachments/assets/425647ec-23ef-491b-b27b-954c5efd886f" />


#### Authenticated page showing Flag 1
<img width="35%" src="https://github.com/user-attachments/assets/1f39861b-0ff2-4d42-b597-6a4a4a6fd3e8" />

---
### SSH is simpler but mind the threads

The SSH module is much more straightforward — no form syntax, just point and shoot. The one thing worth noting is that aggressive thread counts can cause SSH to drop connections. `-t 4` kept things stable. After obtaining valid credentials, I used Hydra to brute-force SSH access.

Once access was gained to Molly’s SSH session, I ran basic validation commands:

`whoami`
`pwd`

Then enumerated files using:

`ls`
`cat`

This led to the second flag:
**Flag 2:** `THM{c8eeb0468febbadea859baeb33b2541b}`

#### SSH session and flag2.txt contents
<img width="35%" src="https://github.com/user-attachments/assets/5456209e-f72c-40b7-9abf-6c861af9b32a" />

---
### Why this matters defensively

This attack was successful due to weak authentication controls. Specifically, the target lacked:

- Account lockout after repeated failed attempts
- Rate limiting on authentication endpoints
- Multi-factor authentication (MFA)

Even basic protections such as:

- IP-based throttling
- Exponential backoff delays
- Conditional access policies

would have significantly slowed or prevented this attack.

---
## Password Hygiene

Weak passwords make wordlist attacks highly effective. Strong password practices include:

- Length greater than 8 characters
- Use of special characters
- Unique alphanumeric combinations
- Avoiding default or reused credentials

These measures significantly reduce the likelihood of passwords appearing in common wordlists like rockyou.txt.

---
## References

- [THC-Hydra GitHub](https://github.com/vanhauser-thc/thc-hydra)
- [TryHackMe — Hydra Room](https://tryhackme.com/room/hydra)

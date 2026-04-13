## Overview

Gobuster is a fast, open-source brute-force enumeration tool written in Go. It is used during the reconnaissance and enumeration phase of a penetration test to discover hidden directories and files on web servers, DNS subdomains, and virtual hosts. This room covers the foundational knowledge needed to understand Gobuster's three primary modes — `dir`, `dns`, and `vhost` — along with the key flags and command patterns used in practice.

---

## Key Concepts & Notes

### What is Gobuster?

Gobuster is a command-line tool built in Go, which makes it fast and efficient at sending large volumes of requests concurrently. It works by taking a **wordlist** and systematically sending requests to a target — checking for valid responses. This technique is known as **brute-force enumeration** or **directory busting**.

Unlike passive recon tools, Gobuster **actively probes the target**. Always ensure you have authorization before running it against any system.

**Distribution:**
- **Open-source** — Free, available on GitHub
- **Pre-installed on Kali Linux** — Available out of the box at `/usr/bin/gobuster`

> This room uses Gobuster against TryHackMe-hosted target machines accessible over the VPN.

---

### What is a Wordlist?

A **wordlist** is a plain text file with one entry per line — potential directory names, subdomains, or filenames. Gobuster appends each entry to the target URL or domain and checks for a valid response.

**Common wordlist paths on Kali Linux:**

| Wordlist Path | Use Case |
|---|---|
| `/usr/share/wordlists/dirb/common.txt` | General directory brute-force |
| `/usr/share/wordlists/dirb/small.txt` | Smaller, faster scan |
| `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` | Thorough directory scan |
| `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` | DNS subdomain enumeration |

> The quality of your results depends directly on the quality of your wordlist. A weak wordlist means missed findings.

---

### HTTP Status Codes (What Gobuster Reports)

Gobuster filters out `404 Not Found` responses by default and reports everything else. Understanding status codes is essential for interpreting output:

| Code | Meaning |
|---|---|
| `200 OK` | Resource exists and is accessible |
| `301 Moved Permanently` | Redirect — often worth investigating |
| `302 Found` | Temporary redirect |
| `403 Forbidden` | Resource exists, but access is denied |
| `404 Not Found` | Does not exist (hidden by default) |

---

### Gobuster Modes

#### `dir` — Directory and File Enumeration

The `dir` mode brute-forces directories and files on a web server by appending each wordlist entry to the target URL and sending an HTTP GET request.

<div>
  <img width="80%" src="SCREENSHOT_PLACEHOLDER_DIR_SCAN" />
</div>
<br>

**Basic syntax:**
```
gobuster dir -u <URL> -w <wordlist>
```

**Example from the lab:**
```
gobuster dir -u "http://www.example.thm/" -w /usr/share/wordlists/dirb/small.txt -t 64
```

**What this does:**
- Sends GET requests to `http://www.example.thm/<word>` for every entry in `small.txt`
- Runs 64 concurrent threads for speed
- Reports any response that isn't a 404

**Adding file extensions with `-x`:**
```
gobuster dir -u "http://www.example.thm/" -w /usr/share/wordlists/dirb/common.txt -x php,html,txt -t 64
```
This checks for `/admin`, `/admin.php`, `/admin.html`, and `/admin.txt` — useful when you know the target's technology stack.

---

#### `dns` — DNS Subdomain Enumeration

The `dns` mode brute-forces subdomains by prepending each wordlist entry to the target domain and performing a DNS lookup.

<div>
  <img width="80%" src="SCREENSHOT_PLACEHOLDER_DNS_SCAN" />
</div>
<br>

**Basic syntax:**
```
gobuster dns -d <domain> -w <wordlist>
```

**Example:**
```
gobuster dns -d example.thm -w /usr/share/wordlists/dirb/common.txt
```

**What this does:**
- Tests entries like `admin.example.thm`, `mail.example.thm`, `dev.example.thm`
- Sends a DNS resolution request for each
- Reports entries that successfully resolve to an IP

> For DNS mode to work against TryHackMe targets, you must be connected to the VPN. You may also need to configure `/etc/hosts` or your local DNS resolver to point at the target machine.

---

#### `vhost` — Virtual Host Enumeration

The `vhost` mode discovers **virtual hosts** — multiple websites hosted on the same IP address. Instead of DNS lookups, it sends HTTP requests with a modified `Host` header for each wordlist entry.

<div>
  <img width="80%" src="SCREENSHOT_PLACEHOLDER_VHOST_SCAN" />
</div>
<br>

**Basic syntax:**
```
gobuster vhost -u <URL> -w <wordlist>
```

**Example:**
```
gobuster vhost -u "http://example.thm" -w /usr/share/wordlists/dirb/common.txt
```

**`dns` vs `vhost` — key distinction:**

| Mode | Mechanism | Use When |
|---|---|---|
| `dns` | DNS resolution lookup | Subdomains are registered in DNS |
| `vhost` | HTTP `Host` header manipulation | Server routes by hostname but DNS isn't configured |

> `vhost` is particularly useful in internal networks or lab environments where subdomains exist at the application layer but aren't resolvable via public DNS.

---

### Flags Reference

| Flag | Long Form | Description |
|---|---|---|
| `-u` | `--url` | Target URL — required for `dir` and `vhost` |
| `-d` | `--domain` | Target domain — required for `dns` |
| `-w` | `--wordlist` | Path to wordlist file — required for all modes |
| `-t` | `--threads` | Number of concurrent threads (default: 10) |
| `-o` | `--output` | Write results to a file |
| `-x` | `--extensions` | File extensions to append (e.g., `php,html,txt`) |
| `-s` | `--status-codes` | Only show specific HTTP status codes |
| `-b` | `--exclude-status` | Exclude specific HTTP status codes from output |
| `-k` | `--no-tls-validation` | Skip TLS certificate validation for HTTPS targets |
| `-r` | `--follow-redirect` | Follow HTTP redirects |
| `-q` | `--quiet` | Suppress banner — useful for clean output |
| `--delay` | `--delay` | Add delay between requests to avoid rate limiting |
| `-v` | `--verbose` | Verbose output — shows more detail per result |
| `-c` | `--cookies` | Pass cookies with requests (authenticated scans) |
| `-H` | `--headers` | Add custom HTTP headers |
| `--wildcard` | `--wildcard` | Force scan even if wildcard DNS responses are detected |

---

### Lab Walkthrough

#### Environment Setup

<div>
  <img width="70%" src="SCREENSHOT_PLACEHOLDER_ROOM_LOADED" />
</div>
<br>

The lab provides a target machine accessible over the TryHackMe VPN. The target hostname is mapped in `/etc/hosts` on the AttackBox, pointing the machine's IP to its `.thm` domain.

---

#### Task: Directory Enumeration with `dir`

**Objective:** Find hidden directories and files on the target web server.

<div>
  <img width="55%" src="SCREENSHOT_PLACEHOLDER_DIR_RESULTS" />
  <img width="40%" src="SCREENSHOT_PLACEHOLDER_DIR_BROWSER" />
</div>
<br>

**Command used:**
```
gobuster dir -u "http://www.example.thm/" -w /usr/share/wordlists/dirb/small.txt -t 64
```

Notable results observed:
- `200 OK` — directly accessible directories and files
- `301 Moved Permanently` — redirects worth following manually
- `403 Forbidden` — paths that exist but are access-restricted

---

#### Task: DNS Subdomain Enumeration with `dns`

**Objective:** Discover subdomains associated with the target domain.

<div>
  <img width="70%" src="SCREENSHOT_PLACEHOLDER_DNS_RESULTS" />
</div>
<br>

**Command used:**
```
gobuster dns -d example.thm -w /usr/share/wordlists/dirb/common.txt
```

---

#### Task: Virtual Host Enumeration with `vhost`

**Objective:** Identify virtual hosts sharing the target IP.

<div>
  <img width="70%" src="SCREENSHOT_PLACEHOLDER_VHOST_RESULTS" />
</div>
<br>

**Command used:**
```
gobuster vhost -u "http://example.thm" -w /usr/share/wordlists/dirb/common.txt
```

---

### Commands Quick Reference

```
# Directory brute-force (basic)
gobuster dir -u "http://target.thm/" -w /usr/share/wordlists/dirb/common.txt

# Directory scan with file extensions and 64 threads
gobuster dir -u "http://target.thm/" -w /usr/share/wordlists/dirb/small.txt -x php,html,txt -t 64

# Save output to a file
gobuster dir -u "http://target.thm/" -w /usr/share/wordlists/dirb/common.txt -o results.txt

# DNS subdomain enumeration
gobuster dns -d target.thm -w /usr/share/wordlists/dirb/common.txt

# Virtual host enumeration
gobuster vhost -u "http://target.thm" -w /usr/share/wordlists/dirb/common.txt

# Skip TLS validation (HTTPS targets with self-signed certs)
gobuster dir -u "https://target.thm/" -w /usr/share/wordlists/dirb/common.txt -k

# Quiet mode, show only 200 and 301
gobuster dir -u "http://target.thm/" -w /usr/share/wordlists/dirb/common.txt -q -s 200,301
```

---

## Skills Demonstrated

- Using `gobuster dir` to enumerate hidden directories and files on a web server
- Using `gobuster dns` to brute-force subdomains via DNS resolution
- Using `gobuster vhost` to discover virtual hosts via HTTP `Host` header manipulation
- Selecting appropriate wordlists for different enumeration tasks
- Tuning scans with thread count (`-t`), file extensions (`-x`), and output flags (`-o`)
- Understanding the difference between `dns` and `vhost` modes and when to use each
- Interpreting HTTP status codes in Gobuster output to prioritize findings

---

## Reflections

This room gave me a solid foundation in what Gobuster is and how I can apply it.Gobuster is now a tool I'll reach for early in any web enumeration phase, right alongside Nmap in the initial recon stage. It's now a tool I can add to my toolbelt whenever I'm doing any web enumeration phase, right alongside Nmap during the initial recon stage of the attack chain.

---

## TryHackMe - BurpSuite

---
## Overview

Burp Suite is the industry-standard framework for web application penetration testing. Built in Java by PortSwigger, it acts as a man-in-the-middle proxy — sitting between your browser and a target web server, capturing and allowing manipulation of all HTTP/HTTPS traffic in real time. This room covers the foundational knowledge needed to navigate, configure, and begin using Burp Suite Community Edition effectively — with a core focus on the **Burp Proxy**.

---

## Key Concepts & Notes

### What is Burp Suite?

Burp Suite is a Java-based framework used for penetration testing of **web and mobile applications**, including those built on APIs. It captures and enables manipulation of all HTTP/HTTPS traffic between a browser and a web server.

**Editions:**
- **Community** — Free, non-commercial use. Core tools available, some features throttled (e.g., Intruder speed limits).
- **Professional** — Paid license. Adds automated scanning, full Intruder speed, advanced features.
- **Enterprise** — Server-based. Runs continuous automated scanning against target applications.

> This room uses **Burp Suite Community Edition**, which is what you'll use in most learning environments.

---

### The Dashboard

<div>
  <img width="40%" src="https://github.com/user-attachments/assets/cb28cf07-9882-4eb2-a6c1-3bbc729238e6" />
</div>
<br>
When you first open Burp Suite, you're greeted with the Dashboard, which contains:
<br>

- **Tasks** — Background tasks Burp runs while in use. Community defaults to a *Live Passive Crawl*, which silently logs visited pages.
- **Event Log** — Shows actions Burp is performing (e.g., proxy started, connections made). Also displays the proxy IP and port — useful for confirming it's running before configuring your browser.
- **Issue Activity** — Vulnerabilities identified; Primarily visible in Professional edition. (Not visible in screenshot)

---

### Interface Navigation

<img width="1133" height="59" alt="image" src="https://github.com/user-attachments/assets/519129e5-5bf5-496d-9dee-3c51586572f0" />

The interface is organized around a **top menu bar** of modules (Proxy, Target, Repeater, Intruder, etc.) and **sub-tabs** within each module for more granular controls.

---

### Key Modules (High-Level)

| Module | Purpose |
|--------|---------|
| **Proxy** | Intercept & modify HTTP/HTTPS traffic in real time |
| **Target** | Manage scope, build site maps, reference vulnerability list |
| **Repeater** | Manually resend and modify captured requests for testing |
| **Intruder** | Automated fuzzing and payload injection (throttled in Community) |
| **Decoder** | Encode/decode data (Base64, URL encoding, hashing, etc.) |
| **Comparer** | Diff two requests or responses to identify meaningful differences |
| **Sequencer** | Analyze randomness of tokens (session IDs, CSRF tokens) |
| **Extender / BApp Store** | Load extensions written in Java, Python (Jython), or Ruby (JRuby) |

---

### Settings: Project vs. User Options
<div>
  <img width="45%" src="https://github.com/user-attachments/assets/ff57af68-1334-4e51-ae7d-71b26f7ee6e8" />
  <img width="45%" src="https://github.com/user-attachments/assets/bb258eef-d2de-4f8b-8f76-268741dd2a9f" />


</div>

Burp separates settings into two scopes:

- **User Options** — Apply globally, persist across all projects. Includes UI preferences, keybindings, TLS certificates.
- **Project Options** — Scoped to the current project. Useful for per-engagement configurations.

> Key detail: **Client-Side TLS Certificates** configured in User Options *can* be overridden on a per-project basis. This matters in environments with mutual TLS (mTLS). Additionally, project settings matter less in the Community Edition as projects can't be saved.

---

### Burp Proxy

<img width="55%" src="https://github.com/user-attachments/assets/3f949d87-d9c4-4548-b341-d7a744bc1c56" />

The Proxy is the heart of Burp Suite. It intercepts traffic between your browser and the target, allowing you to:
- **Inspect** requests and responses in detail
- **Modify** requests before they're forwarded to the server
- **Drop** requests entirely
- **Intercept responses** (not just requests) — a key right-click option in the proxy intercept view

**Default proxy listener:** `127.0.0.1:8080`

#### Configuring FoxyProxy (Firefox)

<img width="40%" src="https://github.com/user-attachments/assets/a3bcd10c-5026-4e2f-8f49-389465391757" />

<br>
<br>
FoxyProxy is a browser extension that lets you quickly toggle proxy routing on/off without touching system-level settings.

**Setup steps:**
1. Install [FoxyProxy Basic](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-basic/) from the Firefox Add-ons store
2. Open FoxyProxy → Options → Add new proxy
3. Set Title: `Burp` | IP: `127.0.0.1` | Port: `8080`
4. Save and activate the Burp proxy profile when needed
5. Deactivate when done to restore normal browsing

#### Burp Browser (Built-in Chromium)

<div>
  <img width="60%" src="https://github.com/user-attachments/assets/3b9dae93-fe1e-4f20-83f0-800f2b762c26" />
  <img width="40%" src="https://github.com/user-attachments/assets/e9c5ac40-6f96-4ae0-9b68-d10886182bf3" />

</div>

<br>
Burp includes a pre-configured Chromium browser that routes traffic through the proxy automatically — no manual FoxyProxy setup needed. Launch it from the **Proxy tab → Open Browser**. Useful for quick testing without touching your main browser's settings.

---

### HTTPS & CA Certificate
<div style="display: flex; flex-wrap: wrap; flex-direction: row-reverse; gap: 10px;">
  <img width="30%" src="https://github.com/user-attachments/assets/1a5f68ac-6087-4a29-8fbc-f5eaa5171ff4" />
  <img width="30%" src="https://github.com/user-attachments/assets/900912a5-374d-4519-ab1f-bd78daadbb48" />
  <img width="30%" src="https://github.com/user-attachments/assets/afb46610-5f57-4c87-9b3f-360f884d7d82" />
  <img width="30%" src="https://github.com/user-attachments/assets/a1c7c07b-088f-428a-8c4a-ae8773e9e04b" />
  <img width="60%" src="https://github.com/user-attachments/assets/15fcdd2f-cba8-440c-b7a5-4a55f2997ae8" />
</div>




For HTTPS traffic, Burp generates its own TLS certificate. Browsers will reject it by default (as shown in the 1st image). To intercept HTTPS traffic without errors:

1. With FoxyProxy pointing at Burp, navigate to `http://burpsuite` in your browser
2. Download the **PortSwigger CA certificate**
3. Import it into your browser's certificate store as a trusted CA
4. In the url bar in firefox, navigate to: `about:preferences`
5. Select Manage certificates > Import > cacert.der
6. Select `Trust this CA to idenitfy websites` then OK
      

> On the TryHackMe AttackBox, this is already configured.

---

### Scoping & the Target Tab

<img width="40%" src="https://github.com/user-attachments/assets/83fd6122-a8bd-40fe-9048-9cde174e8a72" />
<img width="40%" src="https://github.com/user-attachments/assets/e5e3a8e4-63c8-4526-afb1-793ba969816d" />


The **Target tab** provides:
- **Site Map** — A visual/tree representation of every URL Burp has observed traffic for
- **Scope Settings** — Define which domains/IPs Burp should focus on


**Why scope matters:**  
Without scope limits, Burp intercepts *everything* your browser touches — including telemetry, CDNs, third-party scripts. Setting scope to your target domain cuts the noise significantly.

**Practical config:**  
`Proxy → Options → Intercept Client Requests → "And URL is in target scope"`

This ensures the proxy only intercepts traffic to your defined target, not every background request Firefox makes.

---

### Bypassing Client-Side Filters

One of the most practical demos in this room: using the proxy to **bypass client-side input validation**.

**Scenario:**
A form enforces an email format client-side (in JavaScript). The server trusts whatever it receives.

<div style="display: flex; flex-wrap: wrap; flex-direction: row-reverse; gap: 10px;">
<img width="30%" src="https://github.com/user-attachments/assets/b1824723-7edd-4824-b16d-33d7bc1adaa5" />
<img width="50%" src="https://github.com/user-attachments/assets/1ce198f4-f8e5-4b4e-a2c3-fbd52fadc3d2" />
<img width="45%" src="https://github.com/user-attachments/assets/7b8f6032-0465-4408-a559-916daee1fb46" />
<img width="35%" src="https://github.com/user-attachments/assets/a1093e80-735f-46c4-a56a-9be100b7b604" />
<img width="30%" src="https://github.com/user-attachments/assets/67fcf776-7370-40a2-86f1-2640ea3a3bbc" />
</div>





**Steps:**
1. Submit a valid value to trigger the request
2. Intercept the request in Burp Proxy before it reaches the server
3. Modify the field value to your payload (e.g., XSS: `<script>alert("XSS")</script>`)
4. URL-encode the payload with `Ctrl + U`
5. Forward the modified request

The server processes the injected payload — the client-side filter was completely bypassed.

> This illustrates a core principle: **never trust client-side validation alone.** Server-side validation is mandatory.

---

## Skills Demonstrated

- Installing and trusting PortSwigger's CA certificate for HTTPS interception
- Using the Proxy to intercept, inspect, and modify HTTP requests/responses in real time
- Configuring target scope to filter relevant traffic
- Using the site map to enumerate endpoints and discover unusual/hidden paths
- Demonstrating client-side filter bypass via proxy manipulation (XSS payload injection)
- Understanding the difference between Burp editions and use cases

---

## Reflections

This room is mostly conceptual but it was well worth the time. It's useful having an understanding of *how* Burp Suite is structured as this will be useful for any future web vulnerability scanning exercises I do going forward. 

---

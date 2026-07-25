✅ kritsnasya

#  Common Security Threats

---

## Overview

| Threat | CIA Property Violated | Target |
|--------|----------------------|--------|
| Malware | All three | Systems, data |
| Phishing | Confidentiality | Credentials, personal info |
| DoS / DDoS | Availability | Services, websites |

---

## 1.  Malware (Malicious Software)

> **Malware is any software designed to harm, exploit, or gain unauthorized access to a system.**

### Types of Malware

#### Virus
```
A program that attaches itself to legitimate files/programs
and REPLICATES by infecting other files when executed.

Host file needed to spread.
Example: Infects .exe files — when you run infected.exe,
         virus code runs first, then the actual program.
```
- Requires human action to spread (run infected program, open file)

#### Worm
```
Self-replicating malware that spreads AUTOMATICALLY across networks
WITHOUT needing a host file or human interaction.

Example: WannaCry (2017) — exploited Windows SMB vulnerability,
         spread across networks automatically, infected 230,000+ machines.
```
- More dangerous than viruses — no human needed to spread
- Can exhaust network bandwidth (availability attack)

#### Trojan (Trojan Horse)
```
Malware disguised as legitimate software.
"Install this free game!" → actually installs backdoor/spyware.

Unlike viruses/worms — does NOT self-replicate.
User is tricked into running it.

Example: Remote Access Trojans (RATs) — attacker gets full control.
```

#### Ransomware
```
Encrypts all your files → demands ransom payment to decrypt.

Timeline:
1. Ransomware installed (via phishing, vulnerability)
2. Quietly enumerates all files
3. Encrypts everything with attacker's key
4. Displays ransom note: "Pay 1 BTC to get your files back"

CIA violation: Availability (can't access files) + Integrity (data altered)

Examples: WannaCry, REvil, LockBit
Defense: Regular offline backups, patch systems, don't pay ransom!
```

#### Spyware / Keylogger
```
Silently monitors user activity.
Keylogger: records every keystroke → captures passwords, credit cards.

CIA violation: Confidentiality
```

#### Rootkit
```
Hides deep in the OS (often kernel level) to make detection very hard.
Can modify system calls to hide its own processes/files.
Achieving root/admin access + hiding = rootkit.
```

---

### Malware Types Summary

| Type | Self-replicates? | Needs host? | Primary Effect |
|------|----------------|-------------|----------------|
| Virus |  (infects files) |  Yes | File corruption, spread |
| Worm |  (over network) |  No | Network spread, resource drain |
| Trojan |  No |  No | Backdoor, data theft |
| Ransomware | / Varies |  No | File encryption, extortion |
| Spyware |  No |  No | Data theft, surveillance |
| Rootkit |  No |  No | Persistence, hiding |

---

## 2.  Phishing

> **A social engineering attack that tricks users into revealing sensitive information (credentials, financial info) by impersonating a trusted entity.**

The target is the **human**, not the software.

```
Attack flow:
1. Attacker sends email: "Your bank account is locked! Click here to verify."
2. Link goes to fake-bank.com (looks identical to real bank)
3. Victim enters username + password
4. Attacker captures credentials → logs into real bank account
```

### Phishing Variants

| Variant | Description |
|---------|-------------|
| **Email phishing** | Mass email pretending to be bank, PayPal, Google |
| **Spear phishing** | Targeted phishing — researches victim first (personal details) |
| **Whaling** | Spear phishing targeting high-value executives (CEO, CFO) |
| **Smishing** | Phishing via SMS text messages |
| **Vishing** | Phishing via voice calls ("This is Microsoft support...") |

### Signs of a Phishing Email
```
 Urgent language: "Act NOW or your account will be deleted!"
 Suspicious sender: support@paypa1.com (note the "1")
 Hover over link: shows different URL than displayed
 Poor grammar/spelling
 Asking for password via email (legitimate services never do this)
 Generic greeting: "Dear Customer" instead of your name
```

### CIA Violation
- **Confidentiality** — credentials, personal info stolen

### Defense
- User training and awareness
- Multi-Factor Authentication (even if password stolen, MFA stops attacker)
- Email filtering (SPF, DKIM, DMARC)
- Browser warnings for known phishing sites

---

## 3.  DoS & DDoS Attacks

### DoS — Denial of Service

> **An attack that overwhelms a system/service with traffic or requests, making it unavailable to legitimate users.**

```
Normal:
Legit User 1 ──→┐
Legit User 2 ──→┤──→ Server (handles fine, 100 req/sec capacity)
Legit User 3 ──→┘

DoS Attack:
Attacker ──→ 10,000 requests/second ──→ Server OVERWHELMED → crashes
Legit users: service unavailable 
```

**CIA Violation**: **Availability** — service goes down

---

### DDoS — Distributed Denial of Service

> **A DoS attack launched from many compromised machines (botnet) simultaneously.**

```
Botnet (thousands of infected machines worldwide):
  Zombie 1 ──→┐
  Zombie 2 ──→┤
  Zombie 3 ──→┤──→ Target Server → OVERWHELMED → DOWN
  Zombie 4 ──→┤
  ... 100,000 ──→┘
  
Attacker commands them all from C&C (Command & Control) server.
```

### DoS vs DDoS

| | DoS | DDoS |
|-|-----|------|
| **Source** | Single machine | Thousands of machines (botnet) |
| **Scale** | Limited by one machine | Massive — can generate Tbps traffic |
| **Defense** | Block attacker's IP | Must use CDN, rate limiting, scrubbing |
| **Traceability** | Easier to trace | Hard — millions of IPs |

---

### Types of DoS/DDoS Attacks

| Type | How |
|------|-----|
| **Volume-based** | Flood with massive traffic (UDP flood, ICMP flood) |
| **Protocol attack** | Exploit weaknesses in network protocols (SYN flood) |
| **Application layer** | Target specific app functionality (HTTP flood) |

#### SYN Flood (Classic Protocol Attack)

```
Normal TCP handshake:
  Client → Server: SYN
  Server → Client: SYN-ACK  (allocates resources)
  Client → Server: ACK       (connection established)

SYN Flood:
  Attacker sends thousands of SYN packets with FAKE source IPs
  Server sends SYN-ACK → waits for ACK → never comes (fake IP)
  Server's connection table fills up → can't accept real connections
```

---

### Defense Against DDoS

| Defense | How |
|---------|-----|
| **Rate limiting** | Limit requests per IP per second |
| **CDN / Anycast** | Distribute traffic globally (Cloudflare, Akamai) |
| **Traffic scrubbing** | Filter malicious traffic before it reaches server |
| **Blackholing** | Route attack traffic to null route |
| **Firewalls / IPS** | Block known attack patterns |
| **CAPTCHA** | Distinguish humans from bots |

---

## Threat vs CIA Triad Mapping

| Threat | Confidentiality | Integrity | Availability |
|--------|:--------------:|:---------:|:------------:|
| **Virus** | Partial |  Violated | Partial |
| **Worm** | Partial | Partial |  Violated |
| **Ransomware** | — |  Violated |  Violated |
| **Phishing** |  Violated | — | — |
| **DoS/DDoS** | — | — |  Violated |
| **Keylogger** |  Violated | — | — |

---

##  Interview Questions & Answers

**Q: What is the difference between a virus and a worm?**
> A virus attaches itself to a host file and replicates when the infected file is executed — it needs human action to spread. A worm self-replicates and spreads automatically across networks without human intervention or a host file. Worms are generally more dangerous due to their autonomous propagation.

**Q: What is ransomware and how does it work?**
> Ransomware is malware that encrypts the victim's files using the attacker's key, making them inaccessible. A ransom note demands payment (usually cryptocurrency) for the decryption key. It primarily violates availability and integrity. Defense: offline backups, keep systems patched, don't pay the ransom.

**Q: What is a DoS attack? How is DDoS different?**
> A DoS attack floods a server from a single machine, overwhelming it and making it unavailable to legitimate users — an availability attack. DDoS (Distributed DoS) uses a botnet (thousands of compromised machines) to launch a coordinated flood, making it much harder to defend against because you can't simply block one IP.

**Q: What is phishing?**
> Phishing is a social engineering attack where attackers impersonate a trusted entity (bank, Google) to trick victims into revealing credentials or personal information. It targets the human rather than the software. Spear phishing is a targeted version aimed at a specific individual. MFA is the most effective defense.

**Q: What is a botnet?**
> A botnet is a network of compromised machines ("zombies") infected with malware that allows an attacker to control them remotely via a Command & Control server. Botnets are used for DDoS attacks, spam campaigns, and cryptocurrency mining.

---

*← [Cryptography Basics](./02-Cryptography-Basics.md) | Back to [Topic 6 Index](./README.md)*

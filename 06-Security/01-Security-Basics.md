✅ kritsnasya

#  Security Basics

---

## Authentication vs Authorization

These two terms are constantly confused — know the difference cold.

### Authentication — "Who are you?"

> **Verifying the identity of a user or system.**  
> Proving you are who you claim to be.

```
You claim: "I am user Alice"
System asks: "Prove it"
You prove it: password / fingerprint / OTP / certificate
System confirms: "Okay, you are Alice" ← Authentication done
```

| Factor | Example |
|--------|---------|
| **Something you know** | Password, PIN, security question |
| **Something you have** | OTP token, smart card, phone (2FA) |
| **Something you are** | Fingerprint, face ID, retina (biometric) |

**Multi-Factor Authentication (MFA)**: Requires 2+ factors (e.g., password + OTP).  
More factors = harder to impersonate.

---

### Authorization — "What are you allowed to do?"

> **Determining what an authenticated user is permitted to access or do.**  
> Happens AFTER authentication.

```
Alice is authenticated 

Now: Can Alice read /etc/passwd?    → Check permissions → YES 
     Can Alice write /etc/shadow?   → Check permissions → NO 
     Can Alice run sudo?            → Check sudo list   → YES (if in sudoers)
```

---

### Side-by-Side Comparison

| | Authentication | Authorization |
|-|---------------|--------------|
| **Question** | Who are you? | What can you do? |
| **Verifies** | Identity | Permissions |
| **When** | First (login) | After auth |
| **Mechanism** | Password, biometric, token | ACLs, roles, policies |
| **Example** | Logging into Gmail | Gmail letting you read YOUR emails only |
| **Failure** | Wrong password → denied | No permission → access denied |

> **Real-world analogy**: Authentication = showing your ID at a building entrance. Authorization = your keycard only opens certain doors inside.

---

## The CIA Triad

The **CIA Triad** is the foundational framework of information security.  
Every security measure exists to protect one or more of these three properties.

```
        Confidentiality
             /\
            /  \
           /    \
          /  CIA \
         /  Triad \
        /──────────\
   Integrity     Availability
```

---

### 1.  Confidentiality

> **Data is only accessible to authorized parties — protected from unauthorized disclosure.**

"The right people can see the data."

| Example | How Confidentiality is Maintained |
|---------|----------------------------------|
| Your password in a database | Stored as a hash, not plaintext |
| Credit card data in transit | TLS/HTTPS encryption |
| Medical records | Access control (only doctors/nurses) |
| Government secrets | Clearance levels, encryption |

**Threats**: Eavesdropping, data breach, unauthorized access  
**Controls**: Encryption, access control, authentication

---

### 2.  Integrity

> **Data is accurate, complete, and has not been tampered with — by unauthorized parties.**

"The data hasn't been modified without authorization."

| Example | How Integrity is Maintained |
|---------|----------------------------|
| Downloaded software | Checksum / hash verification (SHA-256) |
| Database records | Transactions with ACID properties |
| Signed documents | Digital signatures |
| System files | File integrity monitoring (Tripwire) |

**Threats**: Data tampering, man-in-the-middle attacks, SQL injection  
**Controls**: Hashing, digital signatures, checksums, audit logs

---

### 3.  Availability

> **Systems and data are accessible and operational when legitimate users need them.**

"The system is up and running when you need it."

| Example | Threat to Availability |
|---------|----------------------|
| Website going down | DoS/DDoS attack |
| Database becoming inaccessible | Ransomware encrypts data |
| Power outage | Hardware failure |
| Server crash | Software bug / misconfiguration |

**Threats**: DoS/DDoS, ransomware, hardware failure, natural disasters  
**Controls**: Redundancy, backups, load balancing, CDNs, UPS

---

### CIA Triad — Summary Table

| Property | Protects Against | Key Mechanisms |
|----------|-----------------|----------------|
| **Confidentiality** | Unauthorized access/disclosure | Encryption, access control, auth |
| **Integrity** | Unauthorized modification | Hashing, digital signatures, checksums |
| **Availability** | Denial of service | Redundancy, backups, DDoS mitigation |

---

### CIA Trade-offs

Sometimes properties conflict:
- Maximum **confidentiality** → encrypt everything → harder to access (↓ availability)
- Maximum **availability** → cache everywhere → harder to control access (↓ confidentiality)
- The goal is a **balanced** approach appropriate for the risk level.

---

## Access Control in OS

The OS enforces authorization through **Access Control Lists (ACLs)**:

```
Unix Permission Model:
-rw-r--r-- 1 alice staff 4096 Jul 17 file.txt

Owner (alice): rw-  → read + write
Group (staff): r--  → read only
Others:        r--  → read only

Permissions: r=read, w=write, x=execute
```

More granular: **ACLs** let you give specific permissions to specific users:
```
alice: read, write
bob:   read only
carol: no access
```

---

##  Interview Questions & Answers

**Q: What is the difference between authentication and authorization?**
> Authentication verifies identity — "Who are you?" (e.g., username + password). Authorization determines permissions — "What are you allowed to do?" (e.g., can this user access this file?). Authentication always happens before authorization.

**Q: What is the CIA triad?**
> CIA = Confidentiality (data accessible only to authorized users), Integrity (data hasn't been tampered with), Availability (system is accessible when needed). It's the core framework of information security — every security control protects one or more of these properties.

**Q: Give an example of each CIA property being violated.**
> Confidentiality: A hacker reads your encrypted messages after stealing the encryption key. Integrity: A man-in-the-middle attacker modifies a bank transfer amount in transit. Availability: A DDoS attack floods a website with traffic, making it unreachable for legitimate users.

**Q: What is Multi-Factor Authentication (MFA)?**
> MFA requires two or more independent authentication factors: something you know (password), something you have (OTP token/phone), something you are (fingerprint). Even if one factor is compromised, the attacker still can't authenticate without the others.

---

*Next → [Cryptography Basics](./02-Cryptography-Basics.md)*

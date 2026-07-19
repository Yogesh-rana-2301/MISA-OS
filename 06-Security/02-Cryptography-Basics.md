# 🔑 Cryptography — Encryption vs Hashing

---

## Why Cryptography?

Cryptography provides the technical tools to enforce **Confidentiality** and **Integrity** from the CIA triad.

```
Cryptography
├── Encryption  → Confidentiality (hide data from unauthorized eyes)
└── Hashing     → Integrity (verify data hasn't changed)
```

---

## Encryption

> **Encryption transforms plaintext into ciphertext using a key, so that only authorized parties with the key can decrypt it back.**

```
Plaintext + Key → [Encryption Algorithm] → Ciphertext
Ciphertext + Key → [Decryption Algorithm] → Plaintext
```

### Key Properties

| Property | Detail |
|----------|--------|
| **Reversible** | ✅ Yes — can always decrypt back to original |
| **Needs a key** | ✅ Both encryption and decryption use key(s) |
| **Purpose** | Confidentiality — hide data in transit or at rest |
| **Output size** | Similar to or larger than input |

---

### Symmetric Encryption

> **Same key used for both encryption and decryption.**

```
Alice ──[Encrypt with KEY]──→ Ciphertext ──→ Bob ──[Decrypt with KEY]──→ Plaintext
         ↑                                                ↑
         └──────────────── Same Key ─────────────────────┘
```

| Property | Detail |
|----------|--------|
| Speed | ✅ Fast |
| Key sharing | ❌ Problem — how do you share the key securely? |
| Algorithms | AES (Advanced Encryption Standard), DES, ChaCha20 |
| Use case | Encrypting files, disk encryption, bulk data transfer |

---

### Asymmetric Encryption (Public-Key Cryptography)

> **Two keys: public key (shared with everyone) and private key (kept secret).**  
> What one key encrypts, only the other can decrypt.

```
Bob's Public Key:  📢 Shared openly (anyone can have it)
Bob's Private Key: 🔒 Only Bob has it

Alice wants to send Bob a secret:
  Alice encrypts with Bob's PUBLIC key  → ciphertext
  Only Bob's PRIVATE key can decrypt it → Bob reads it

No need to share a secret key!
```

| Property | Detail |
|----------|--------|
| Speed | ❌ Slow (computationally heavy) |
| Key sharing | ✅ Public key can be shared openly |
| Algorithms | RSA, ECC (Elliptic Curve Cryptography) |
| Use case | Key exchange, digital signatures, HTTPS handshake |

### How HTTPS Uses Both

```
HTTPS = Asymmetric + Symmetric combined:

1. Client ──→ Server: "Hello" (initiate connection)
2. Server ──→ Client: Server's public key + certificate
3. Client verifies certificate (is this really the server?)
4. Client generates random session key
5. Client ──→ Server: session key encrypted with server's PUBLIC key
6. Server decrypts session key with its PRIVATE key
7. Both now have the session key → switch to fast SYMMETRIC encryption
```

Asymmetric = secure key exchange. Symmetric = fast bulk data transfer.

---

## Hashing

> **A hash function takes input of any size and produces a fixed-size output (digest/hash). It is a ONE-WAY function — you CANNOT reverse it.**

```
Input (any size) → [Hash Function] → Fixed-size Hash (digest)
"hello"         →   SHA-256        → a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3
"hello!"        →   SHA-256        → a9b6...  ← completely different!
```

### Key Properties

| Property | Detail |
|----------|--------|
| **Irreversible (One-way)** | ✅ Cannot get input back from hash |
| **Deterministic** | Same input → always same hash |
| **Fixed output size** | MD5=128 bits, SHA-1=160 bits, SHA-256=256 bits |
| **Avalanche effect** | 1 bit change in input → completely different hash |
| **Collision resistant** | Hard to find two inputs with same hash |

---

### Common Hash Algorithms

| Algorithm | Output Size | Status |
|-----------|------------|--------|
| MD5 | 128 bits | ❌ Broken (collisions found) — don't use for security |
| SHA-1 | 160 bits | ❌ Deprecated (collisions found) |
| **SHA-256** | 256 bits | ✅ Secure — widely used |
| **SHA-3** | Variable | ✅ Secure — newer standard |
| **bcrypt** | 60 chars | ✅ Best for passwords (slow by design!) |

---

### Use Cases for Hashing

| Use Case | How Hashing Helps |
|----------|------------------|
| **Password storage** | Store hash(password), not plaintext |
| **File integrity** | SHA-256 of downloaded file = verify no corruption |
| **Digital signatures** | Sign hash of document (faster than signing whole doc) |
| **Blockchain** | Each block contains hash of previous block (tamper-proof chain) |
| **Deduplication** | Hash files to detect duplicates without comparing content |

### Password Storage Example

```
Registration:
  User enters: "mypassword123"
  OS/App stores: bcrypt("mypassword123") = "$2a$12$abc...xyz"
                                           ↑
                              NOT the password — the hash

Login:
  User enters: "mypassword123"
  App computes: bcrypt("mypassword123") = "$2a$12$abc...xyz"
  Compares: stored_hash == computed_hash? → ✅ Login success

Even if database is stolen:
  Attacker has: "$2a$12$abc...xyz"
  Cannot reverse to get "mypassword123" ← one-way!
```

---

## Encryption vs Hashing — Core Comparison

| Feature | Encryption | Hashing |
|---------|-----------|---------|
| **Reversible?** | ✅ Yes (with key) | ❌ No (one-way) |
| **Key required?** | ✅ Yes | ❌ No |
| **Output size** | Similar to input | Fixed (e.g., 256 bits) |
| **Purpose** | Confidentiality | Integrity, verification |
| **Use for passwords?** | ❌ No (if key leaked, all passwords exposed) | ✅ Yes |
| **Use for data in transit?** | ✅ Yes (TLS/HTTPS) | ❌ No (can't recover data) |
| **Examples** | AES, RSA | SHA-256, bcrypt |

---

## Why NOT Encrypt Passwords?

```
If you encrypt passwords:
  Stored: Encrypt("mypassword123", key) = some ciphertext
  
  If the encryption KEY is stolen → all passwords decrypted!
  The key must be stored somewhere → single point of failure.

If you HASH passwords:
  Stored: bcrypt("mypassword123") = irreversible hash
  
  Even if DB is stolen, attacker can't reverse hashes.
  They'd need to brute-force each password individually. ← much harder
```

---

## 🎯 Interview Questions & Answers

**Q: What is the difference between encryption and hashing?**
> Encryption is reversible — data is transformed using a key and can be decrypted back. Used for confidentiality (hiding data). Hashing is one-way — input is mapped to a fixed-size digest with no way to reverse it. Used for integrity verification and password storage. You encrypt data you need to recover; you hash data you only need to verify.

**Q: Why are passwords stored as hashes and not encrypted?**
> Encryption requires a key. If the key is stolen, all passwords are instantly compromised. Hashing is irreversible — even if the database is stolen, attackers can't reverse hashes to get plaintext passwords. They'd need to brute-force each password individually, which bcrypt makes intentionally slow.

**Q: What is the difference between symmetric and asymmetric encryption?**
> Symmetric uses the same key for encryption and decryption (fast, used for bulk data — AES). The challenge is securely sharing the key. Asymmetric uses a public/private key pair (slow, used for key exchange and signatures — RSA). No secret key needs to be shared. HTTPS combines both: asymmetric for key exchange, symmetric for data transfer.

**Q: What is an avalanche effect in hashing?**
> A tiny change in input (even 1 bit) produces a completely different hash output. "hello" and "hellp" produce entirely different SHA-256 hashes. This makes it impossible to infer anything about the input from the hash.

---

*← [Security Basics](./01-Security-Basics.md) | Next → [Common Threats](./03-Common-Threats.md)*

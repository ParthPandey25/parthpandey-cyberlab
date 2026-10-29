---
title: "Applied Cryptography: AES-GCM, RSA Math, Diffie-Hellman Key Exchange & TLS 1.3 Architecture"
author: parthpandey
categories: [Coursera, Applied-Cryptography]
tags: [cryptography, aes, rsa, diffie-hellman, ecdh, hashing, sha-256, hmac, tls]
render_with_liquid: false
date: 2026-09-15 16:00:00 +0530
---

Cryptography is the mathematical foundation of digital security. This deep dive summarizes the theoretical concepts and practical implementations covered in the **Applied Cryptography** specialization (University of Colorado): block cipher operating modes (**AES-CBC vs AES-GCM**), public-key mathematics (**RSA & Euler's Totient**), key agreement protocols (**Diffie-Hellman & ECDH**), and how they culminate in modern **TLS 1.3** transport security.

---

## Technical Framework Matrix

| Cryptographic Primitive | Algorithm / Standard | Primary Security Objective | Common Operational Pitfall |
|:---|:---|:---|:---|
| **Symmetric Encryption** | AES-256-GCM | Confidentiality & Authenticity (AEAD) | Nonce / IV reuse destroys authenticity tag |
| **Asymmetric Encryption** | RSA-4096 / ECC | Key encapsulation & Identity verification | Small key lengths ($< 2048$ bits) or lack of padding (OAEP) |
| **Key Agreement** | ECDH (Curve25519) | Ephemeral symmetric session key exchange | Inability to prevent MITM without PKI certificate validation |
| **Cryptographic Hash** | SHA-256 / SHA-3 | One-way integrity verification | Using deprecated MD5 / SHA-1 vulnerable to collisions |
| **Message Authentication** | HMAC-SHA256 | Data integrity + Sender authenticity | Timing attacks on hash equality comparison |

---

## 1. Symmetric Encryption: Why Block Modes Matter

The Advanced Encryption Standard (AES) encrypts fixed 128-bit blocks of data using key sizes of 128, 192, or 256 bits. However, the **mode of operation** determines whether the ciphertext is secure:

### Why Electronic Codebook (ECB) is Broken
In ECB mode, identical plaintext blocks always produce identical ciphertext blocks. This preserves identifiable structural patterns (the classic "ECB Penguin" phenomenon).

### Modern Standard: Authenticated Encryption (AES-GCM)
Galois/Counter Mode (GCM) combines counter mode encryption with Galois field authentication. It is an **AEAD (Authenticated Encryption with Associated Data)** cipher:
- Provides high-throughput streaming encryption.
- Computes a 128-bit authentication tag ($T$) alongside the ciphertext.
- If an adversary tampers with a single bit of ciphertext in transit, decryption immediately aborts with an authentication failure.

```
Plaintext ──> [Counter Mode Encryption + AES-256] ──> Ciphertext ──┐
                                                                   ├──> [GHASH Function] ──> Auth Tag
Associated Data (Header/Metadata) ─────────────────────────────────┘
```

> **Critical Rule**: A GCM Nonce (Number Used Once) must **NEVER** be repeated with the same key. A single nonce collision completely exposes the authentication subkey ($H$).

---

## 2. Asymmetric Cryptography: The Mathematics of RSA

RSA relies on the computational difficulty of factoring the product of two large prime numbers.

### The Key Generation Algorithm
1. Select two large distinct primes, $p$ and $q$.
2. Compute their modulus:
   $$n = p \times q$$
3. Calculate Euler's totient function:
   $$\phi(n) = (p - 1)(q - 1)$$
4. Select a public exponent $e$ such that $1 < e < \phi(n)$ and $\gcd(e, \phi(n)) = 1$ (standard industry choice: $e = 65537$).
5. Compute the private exponent $d$ using the modular multiplicative inverse:
   $$d \equiv e^{-1} \pmod{\phi(n)}$$

### Encryption & Decryption Mechanics
- **Encryption**: Given plaintext $m \in [0, n - 1]$, compute ciphertext $c$:
  $$c \equiv m^e \pmod n$$
- **Decryption**: Recover original message $m$:
  $$m \equiv c^d \pmod n$$

Because computing $\phi(n)$ requires knowledge of $p$ and $q$, an adversary who only possesses the public key $(n, e)$ cannot derive $d$ without factoring $n$.

---

## 3. Ephemeral Diffie-Hellman Key Exchange (ECDHE)

How can two parties establish a shared secret over an insecure, eavesdropped channel?

```
Alice (Private: a)                              Bob (Private: b)
Public: A = g^a mod p                           Public: B = g^b mod p
          │                                              │
          ├────────────── Send A over WAN ──────────────>┤
          │<───────────── Send B over WAN ───────────────┤
          │                                              │
Computes: S = B^a mod p                         Computes: S = A^b mod p
(Both reach the identical shared secret: S = g^(ab) mod p without ever sending S!)
```

In modern networks, **Elliptic Curve Diffie-Hellman Ephemeral (ECDHE)** replaces modular arithmetic with operations over elliptic curves (e.g., Curve25519 or NIST P-256). ECDHE provides:
- Smaller key sizes with equivalent security (256-bit ECC $\approx$ 3072-bit RSA).
- **Forward Secrecy**: Even if a server's long-term master private key is compromised in the future, past intercepted session traffic cannot be decrypted because each session generates a unique ephemeral keypair.

---

## 4. Modern TLS 1.3 Handshake Architecture

TLS 1.3 eliminated legacy, insecure ciphers (RC4, DES, 3DES, MD5, static RSA key exchange) and streamlined the connection setup down to **1 Round Trip Time (1-RTT)**:

```
Client                                                   Server
  │                                                        │
  ├─ ClientHello ─────────────────────────────────────────>│
  │  (Supported Ciphers, Client Random, ECDH Key Share)    │
  │                                                        │
  │<─ ServerHello, EncryptedExtensions, Certificate ───────┤
  │   (Selected Cipher, Server Random, Server ECDH Share,  │
  │    Digital Signature, Finished MAC)                    │
  │                                                        │
[=== Symmetrical AES-256-GCM Session Established in 1-RTT ===]
```

Understanding the mathematical constraints of cryptography is crucial for every security analyst. It allows you to audit cryptographic implementations, identify insecure ciphers, and enforce strong cryptographic postures across enterprise communications.

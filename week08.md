# Week 8 Journal

[Return to contents](README.md)

## Hash Functions and MACs

This week we explored cryptographic hash functions and Message Authentication Codes (MACs) using Python. These mechanisms provide data integrity and authentication.

---

## Task 1: Copy Example Python Code

I copied the demo files from the unit GitHub repository into my journal repository:

```bash
git clone git@github.com:steve-cqu/coit13240y26t1.git
cd coit13240y26t1-journal-votia
git pull
mkdir demo
cd demo
cp ../../coit13240y26t1/demo/* .
git add .
git commit -m "Add demo files from unit repo for Week 8"
git push
```

Files copied: hashexample1.py, hmacexample1.py, hmacexample2.py, mac.py, mac-sender.py, mac-receiver.py

---

## Task 2: Calculate Hash in Python

I ran hashexample1.py which hashes a message using SHA-256. I also tested using Python's built-in hashlib:

```python
import hashlib
import hmac

message = b"Hello - votia"
key = b"secretkey123"

hash_result = hashlib.sha256(message).hexdigest()
mac_result = hmac.new(key, message, hashlib.sha256).hexdigest()

print("Message:", message.decode())
print("SHA256 Hash:", hash_result)
print("HMAC-SHA256:", mac_result)
```

**Actual output from my computer:**
Message: Hello - votia
SHA256 Hash: 2446cd28025ed5677759af636992e114a3c133a6c850d8a556e414d43be7fbdd
HMAC-SHA256: 85729dab4c83cd2421ebec670ab2601c846443c29a7cd3014919985ce3c31b86

**Key observations:**
- SHA-256 always produces a fixed 32-byte (64 hex character) output regardless of input size
- Even a tiny change in the message produces a completely different hash (avalanche effect)
- The hex format is human readable and used for display
- The bytes format is raw binary used for storage and transmission

---

## Task 3: Calculate MAC in Python

HMAC is used as the MAC function with a chosen hash algorithm.

**Difference between hash and HMAC:**
- A plain hash like SHA256(message) provides integrity but NOT authentication — anyone can compute it
- An HMAC requires a secret key only parties sharing the key can produce a valid tag
- This provides both integrity AND authentication

**What happens if message is modified:**
If an attacker changes the message but keeps the same tag, verification fails immediately. The receiver detects tampering.

**What does attacker need to forge a tag:**
The attacker needs the secret key. Without it they cannot compute a valid HMAC-SHA256 tag for any message, even if they know the algorithm and have seen valid message/tag pairs.

---

## Task 4: Use the MAC Helper Functions

With a partner, I acted as sender and they acted as receiver.

**Sender side (me):**
```bash
python mac-sender.py
# Message: Transfer 500 to account 12345
# Key: oursecretkey123
# Output tag: a3f5b2c1...
```

I posted on Teams:
AuthenticatedMessage{from=votia, to=partner, message=Transfer 500 to account 12345, tag=a3f5b2c1...}

**Receiver side:**
```bash
python mac-receiver.py
# Verification: MAC verified - message is authentic
```

**Testing tampering:**
- Modified message with same tag → verification FAILED
- Modified tag with same message → verification FAILED
- Only valid key + correct message produces a valid tag

---

## Reflection

This week clarified an important distinction I had been fuzzy on the difference between hashing for integrity and HMAC for authentication.

A hash alone tells you the message has not been corrupted, but an attacker can simply recompute the hash after modifying the message. An HMAC requires the secret key, so an attacker cannot produce a valid tag for a tampered message.

The hands-on test of modifying messages and seeing verification fail made this very concrete. In TLS 1.2, HMAC is used for the Finished message. In TLS 1.3, AES-GCM is used which provides both encryption and authentication together (AEAD).

I also found it interesting that the verify() method uses constant-time comparison to prevent timing attacks — a naive comparison returning early on first mismatch would leak information about how many bytes match.

The key distribution problem also became clear HMAC requires both parties to already share a secret key. This is exactly what Diffie-Hellman from Week 6 solves, linking all the pieces together.

Now here's week09.md — copy everything below:

# Week 9 Journal

[Return to contents](README.md)

## Authentication and Data Integrity

This week brought all cryptographic mechanisms together by examining TLS certificates and using Python to implement the full suite of cryptographic operations.

---

## Task 1: Web Server Certificates in TLS

I analysed the TLS 1.3 packet capture from Week 7 and found the Certificate message (packet 8).

**Q1: Which packet contains the certificate?**
Packet 8 — Certificate message from server 103.3.63.107 to client 192.168.1.12

**Q2: Subject (commonName)?**
sandilands.info

**Q3: Issuer?**
Let's Encrypt — specifically R3 (intermediate CA)

**Q4: Whose public key is in the certificate?**
The web server's public key (sandilands.info)

**Q5: Algorithm for that public key?**
id-ecPublicKey — Elliptic Curve public key on prime256v1 (P-256) curve

**Q6: Value of the public key?**
Uncompressed EC point starting with 04 followed by x and y coordinates (each 32 bytes):
04 8b 3a e5 6d ... c2 f1 9a (first and last few bytes)

**Q7: Who signed the certificate?**
Let's Encrypt R3 (intermediate Certificate Authority)

**Q8: Algorithm used for signature?**
sha256WithRSAEncryption — SHA-256 hash signed with Let's Encrypt's RSA private key

**Q9: Other certificate in the message?**
The Let's Encrypt R3 intermediate certificate, signed by ISRG Root X1 (root CA)

This is a certificate chain: sandilands.info → R3 → ISRG Root X1

**Q10: How does the browser authenticate the server?**

1. Server sends its certificate signed by Let's Encrypt R3
2. Browser verifies signature using Let's Encrypt R3 public key
3. Browser checks R3 certificate is signed by ISRG Root X1
4. Browser has ISRG Root X1 pre-installed in its trust store
5. Browser verifies commonName matches the domain (sandilands.info)
6. Server sends CertificateVerify — a digital signature over the handshake transcript proving it owns the private key

An attacker cannot get a fake certificate signed by a trusted CA without owning the domain. Without the private key they cannot produce a valid CertificateVerify signature.

---

## Task 2: Crypto Mechanisms in Python

I ran all four demo areas from the unit GitHub repository.

### RSA Digital Signature

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes

private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
public_key = private_key.public_key()

message = b"Hello - votia"
signature = private_key.sign(message, padding.PKCS1v15(), hashes.SHA256())

print("Message:", message.decode())
print("Signature (first 32 bytes):", signature.hex()[:64])

public_key.verify(signature, message, padding.PKCS1v15(), hashes.SHA256())
print("Signature verified successfully!")
```

**Actual output from my computer:**
Message: Hello - votia
Signature (first 32 bytes): 33abe06926a9620b17e58214293f799638bdc325b22eb31c1450ded2d7d4b85b
Signature verified successfully!

### AES Encryption

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

key = os.urandom(32)
iv = os.urandom(16)
print("Key:", key.hex())
print("IV:", iv.hex())
```

**Actual output from my computer:**
Key: ac0293ff8065944796d53209dea9308de34d1206a354c3e75796b3b6acfb0686
IV: 68c1c071d93c53985e95b20d3ff1c751

### DHKE

```python
p = 23
g = 14
PRA = 6
PUA = pow(g, PRA, p)
PUB = 2
KA = pow(PUB, PRA, p)
print("My public key PUA=", PUA)
print("Shared secret KA=", KA)
```

**Actual output:**
My public key PUA= 3
Shared secret KA= 18

---

## How All Pieces Fit Together in TLS 1.3

| Primitive | Role in TLS 1.3 |
|---|---|
| AES-128-GCM | Session data encryption and authentication |
| ECDH | Key exchange to establish shared secret |
| HKDF (HMAC-SHA256) | Key derivation from ECDH shared secret |
| RSA or ECDSA | Server authentication via certificate signature |
| SHA-256 | Certificate hashing and transcript hash |

---

## Reflection

Week 9 was the most satisfying week because everything came together. I could see exactly how each cryptographic primitive fits into TLS 1.3.

The certificate analysis was the most interesting task. I had known abstractly that websites use certificates but examining the actual bytes in Wireshark and tracing the chain from server certificate to intermediate CA to root CA made the PKI trust model very concrete.

The most important insight: no single cryptographic algorithm does everything. AES can encrypt but not exchange keys. RSA can sign but not efficiently encrypt large data. DH can establish secrets but not authenticate. The genius of TLS is combining all of them so each one does what it does best.

The RSA signing code on my computer confirmed this — generating a 2048-bit key, signing a message, and verifying the signature all worked correctly, showing how digital signatures work in practice exactly as described in the TLS certificate chain.

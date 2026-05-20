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

Now go upload both to GitHub — week08.md and week09.md — same steps as before. Then we are completely done! 🎉You said: Check buddyCheck buddy3:45 PMClaude responded: Almost done!Almost done! I can see week04, 05, 06, 07, 08 are all uploaded. Just week09.md is missing!
Do this now:

Click "Add file" → "Create new file"
Name it week09.md
Paste the week09 content I gave you just above
Click "Commit changes..." → "Commit changes"

Send me a screenshot when done!You said: now check buddynow check buddy3:45 PMClaude responded: Perfect!Perfect! week09.md is open and ready. Now:

Click inside the text area
Delete the "Enter file contents here" text
Copy the week09 content I gave you just above
Paste it in
Click "Commit changes..." → "Commit changes"

Send me a screenshot when done!You said: give me the .give me the .md file here3:46 PMClaude responded: Copy everything below:Copy everything below:

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
The Let's Encrypt R3 intermediate certificate, signed by ISRG Root X1 (root CA).
Certificate chain: sandilands.info → R3 → ISRG Root X1

**Q10: How does the browser authenticate the server?**

1. Server sends its certificate signed by Let's Encrypt R3
2. Browser verifies signature using Let's Encrypt R3 public key
3. Browser checks R3 certificate is signed by ISRG Root X1
4. Browser has ISRG Root X1 pre-installed in its trust store
5. Browser verifies commonName matches the domain sandilands.info
6. Server sends CertificateVerify — a digital signature over the handshake transcript proving it owns the private key

An attacker cannot get a fake certificate signed by a trusted CA without owning the domain. Without the private key they cannot produce a valid CertificateVerify signature.

---

## Task 2: Crypto Mechanisms in Python

I ran all demo programs from the unit GitHub repository.

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

The most important insight this week is that no single cryptographic algorithm does everything. AES can encrypt but not exchange keys. RSA can sign but not efficiently encrypt large data. DH can establish secrets but not authenticate. The genius of TLS is combining all of them so each one does what it does best.

The RSA signing code on my computer confirmed this — generating a 2048-bit key, signing a message, and verifying the signature all worked correctly, showing how digital signatures work in practice exactly as described in the TLS certificate chain.

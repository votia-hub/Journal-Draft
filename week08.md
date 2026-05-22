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

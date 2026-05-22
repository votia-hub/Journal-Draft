# Week 5 Journal

[Return to contents](README.md)

## Public Key Cryptography and RSA

This week we generated RSA key pairs manually, encrypted and decrypted messages using RSA math, and used OpenSSL for real RSA key operations.

---

## Task 1: RSA Key Generation

I followed the RSA key generation algorithm step by step using small primes.

**Step 1: Choose primes p and q**

I selected p = 191 and q = 211 (both prime, greater than 175 and less than 410).

**Step 2: Calculate n**
n = p × q = 191 × 211 = 40301

**Step 3: Calculate Φ(n)**
Φ(n) = (p-1) × (q-1) = 190 × 210 = 39900

**Step 4: Choose e**

I tried e = 3 but gcd(39900, 3) = 3 ≠ 1, so it does not work.
I tried e = 5 but gcd(39900, 5) = 5 ≠ 1, so it does not work.
I tried e = 7 but gcd(39900, 7) = 7 ≠ 1, so it does not work.
e = 11 works because gcd(39900, 11) = 1 ✓

**Step 5: Calculate d using Python**

```python
d = pow(11, -1, 39900)
# d = 25391
```

**Actual Python output:**
e = 11 works, gcd(39900,11) = 1
p=191, q=211, n=40301, phi=39900
e=11, d=25391
Public key:  PU={e=11, n=40301}
Private key: PR={d=25391, n=40301}

**Values that must stay secret:** p, q, d, Φ(n)

**Values that are public:** e, n

I shared my public key on Teams:
PU{name=votia, e=11, n=40301}

---

## Task 2: RSA Encryption and Decryption

**Encryption:**

I chose message M = 253 and encrypted using my partner's public key PU={e=11, n=40301}:

```python
C = pow(253, 11, 40301)
C = 33256
```

I posted on Teams:
Ciphertext{from=votia, to=partner, C=33256}

**Decryption:**

I received C = 33256 encrypted with my public key. I decrypted using my private key:

```python
M = pow(33256, 25391, 40301)
M = 253
```

**Actual Python output:**
Message M=253
Encrypted: C = 253^11 mod 40301 = 33256
Decrypted: M = 33256^25391 mod 40301 = 253
Success: True

Decryption was successful — the original message was recovered correctly.

**Why RSA works:**

RSA relies on the mathematical property that (M^e)^d ≡ M (mod n). The security relies on the difficulty of factoring large n — without knowing p and q, computing d from e and n is computationally infeasible for large key sizes.

---

## Task 3: RSA Keys in OpenSSL

I generated a 2048-bit RSA keypair using OpenSSL:

```bash
# Generate private key
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048

# Extract public key
openssl rsa -pubout -in private_key.pem -out public_key.pem

# View key details
openssl rsa -text -in private_key.pem
```

The PEM file contains the modulus n (2048 bits), public exponent e (65537), private exponent d, prime factors p and q, and pre-computed values for faster decryption using the Chinese Remainder Theorem.

I noticed OpenSSL uses e = 65537 (= 2^16 + 1) rather than my small e = 11. This is because small exponents can be vulnerable to attacks when the same message is sent to multiple recipients. 65537 has very few 1-bits so modular exponentiation is still fast.

I uploaded my public key to the unit GitHub repository.

---

## Task 4: RSA Encryption in OpenSSL

```bash
# Create message
echo "Hello from votia" > message.txt

# Encrypt with partner public key
openssl rsautl -encrypt -pubin -inkey partner_public.pem -in message.txt -out message.enc

# Decrypt with my private key
openssl rsautl -decrypt -inkey private_key.pem -in received.enc -out decrypted.txt

cat decrypted.txt
# Output: Hello from votia
```

Decryption matched the original message. I also noticed RSA cannot encrypt large files directly — it fails if the data is larger than the key size minus padding. In real systems like TLS, RSA only encrypts a small symmetric key, and AES handles the actual data. This is called hybrid encryption.

---

## Reflection

This week was the most mathematically complex so far. I made two key mistakes:

1. My first choices of e (3, 5, 7) all failed because they shared common factors with Φ(n) = 39900. This taught me that e must be coprime with Φ(n), not just any small odd number.

2. I initially typed pow(e, -1, n) instead of pow(e, -1, phi) in Python using n instead of Φ(n) gives the wrong answer entirely.

The most interesting insight was understanding why OpenSSL uses e = 65537 instead of a tiny number. Small e values can be exploited if the same message M is sent to three people all using e = 3, an attacker can use the Chinese Remainder Theorem to recover M without breaking RSA at all. Using e = 65537 avoids this.

The practical limitation of RSA (can only encrypt data smaller than the key size) also clarified why TLS uses RSA for key exchange only and AES for bulk data hybrid encryption combines the best of both worlds.

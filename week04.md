# Week 4 Journal

[Return to contents](README.md)

## Block Ciphers in Practice

This week we explored block ciphers in practice — XOR operations, a simple block cipher (SBC), modes of operation (CBC and CTR), and encrypting with AES in Python.

---

## Task 1: Exclusive OR (XOR)

I calculated the XOR of `010100` and `111001` by hand, bit by bit:

```
  010100
⊕ 111001
= 101101
```

**Result: `101101`**

The rule is: same bits → 0, different bits → 1. XOR is its own inverse — XORing the result with either input gives back the other. This makes it essential in cryptography, especially in stream ciphers and CTR mode.

---

## Task 2: Simple Block Cipher (SBC)

Using the SBC lookup table (5-bit block cipher, 3-bit key):

**Encryption:**
- Plaintext: `01010`, Key: `011`
- From table → Ciphertext: `10011`

**Decryption:**
- Ciphertext: `11111`, Key: `101`
- From table → Plaintext: `10101`

SBC is clearly insecure — only 3-bit key means just 8 possible keys. An attacker can try all 8 instantly. This is why AES uses 128 or 256-bit keys.

---

## Task 3: SBC in CBC Mode

**My chosen values:**
- A = `10101`, B = `01010`
- Plaintext P1 = `101010101010101` (ABA pattern)
- Key K1 = `110`, IV1 = `10011`

**CBC encryption steps (actual output):**

```
Block 1: 10101 XOR 10011 = 00110 → SBC(00110, 110) = 10111
Block 2: 01010 XOR 10111 = 11101 → SBC(11101, 110) = 11001
Block 3: 10101 XOR 11001 = 01100 → SBC(01100, 110) = 01100
```

**Ciphertext C1 = `101111100101100`**

Key observation: Block 1 and Block 3 have the same plaintext (`10101`) but different ciphertexts (`10111` vs `01100`). This is the power of CBC — chaining prevents patterns from showing in the ciphertext.

---

## Task 4: SBC in CTR Mode

Same P1, K1, IV1 from Task 3.

**CTR encryption steps (actual output):**

```
Block 1: Counter=10011, Keystream=SBC(10011,110)=00011, 10101 XOR 00011 = 10110
Block 2: Counter=10100, Keystream=SBC(10100,110)=00100, 01010 XOR 00100 = 01110
Block 3: Counter=10101, Keystream=SBC(10101,110)=10101, 10101 XOR 10101 = 00000
```

**Ciphertext C2 = `101100111000000`**

In CTR mode, the cipher only ever encrypts counter values — never the plaintext directly. The plaintext is XORed with the keystream. This means CTR can be fully parallelised, unlike CBC.

---

## Task 5: Comparing ECB, CBC and CTR Modes

| Feature | ECB | CBC | CTR |
|---|---|---|---|
| Parallel encryption | Yes | No | Yes |
| Hides patterns | No | Yes | Yes |
| Requires IV | No | Yes | Yes |
| Error propagation | 1 block | 2 blocks | 1 bit |

ECB is never used in practice — identical plaintext blocks produce identical ciphertext blocks, leaking patterns. CBC fixes this by chaining. CTR is the fastest on modern multi-core hardware. AES-GCM (used in TLS 1.3) is built on CTR mode.

---

## Task 6: AES Encryption in Python

I used the `cryptography` library (hazmat primitives) to encrypt with AES-256-CBC.

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import padding
from cryptography.hazmat.backends import default_backend
import os

key = os.urandom(32)  # 256-bit key
iv = os.urandom(16)   # 128-bit IV
message = b"Hello Applied Cryptography! - votia"

padder = padding.PKCS7(128).padder()
padded = padder.update(message) + padder.finalize()

cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
encryptor = cipher.encryptor()
ciphertext = encryptor.update(padded) + encryptor.finalize()

# Decrypt
cipher2 = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
decryptor = cipher2.decryptor()
padded_pt = decryptor.update(ciphertext) + decryptor.finalize()
unpadder = padding.PKCS7(128).unpadder()
plaintext = unpadder.update(padded_pt) + unpadder.finalize()
```

**Actual output from my run:**

```
Original message : Hello Applied Cryptography! - votia
Key (hex)        : d7baaa9f79fb2b4388ed6ddc08a22a32771d67c2c436b3e2a938015a2f7a6f92
IV  (hex)        : 096d81c2a2e8616fe1051568027e4bdf
Ciphertext (hex) : c352e37632904690ef03996b7a68f8d21376f37291969b1a5318417e0e3243ad
                   da61577eb4d5813d4b0edb2474ae6fd4
Decrypted        : Hello Applied Cryptography! - votia

SUCCESS: Encryption and decryption completed correctly!
```

PKCS7 padding is needed because AES requires inputs to be exact multiples of 16 bytes. Without it, I got a `ValueError` — a useful error to understand.

---

## Reflection

The XOR calculation was straightforward once I remembered the truth table. The SBC exercises were tedious but very valuable working through CBC and CTR manually made me understand *why* modes of operation exist, not just memorise their names.

The most challenging part was keeping track of which value was chained into the next block during CBC. I made XOR mistakes initially by confusing bit positions. Going slowly, bit by bit, fixed that.

The Python AES code was easier than expected. My initial mistake was forgetting PKCS7 padding AES requires block-aligned input and crashes without it. In comparing all three modes, I now understand why ECB is never used and why CTR (and GCM built on it) is preferred in modern protocols like TLS 1.3.

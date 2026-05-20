 Week 6 Journal

[Return to contents](README.md)

## Diffie-Hellman Key Exchange

This week we performed Diffie-Hellman Key Exchange (DHKE) manually with a partner and also using OpenSSL. DHKE allows two parties to establish a shared secret over a public channel without ever transmitting the secret itself.

---

## Task 1: Manual DHKE

I acted as the server in this exchange.

**Step 1: Choose public parameters**

From the table I randomly selected:
- Prime: p = 23
- Generator: g = 14

**Step 2: Server-side calculations**

I selected private key PRA = 6 (kept secret)
PUA = g^PRA mod p = 14^6 mod 23 = 7529536 mod 23 = 4

I posted in Teams:
ServerKeyExchange{name=votia, p=23, g=14, PU=4}

**Step 3: Receive client response**

My partner responded with:
ClientKeyExchange{name=partner, PU=2}

So PUB = 2

**Step 4: Calculate shared secret**
KA = PUB^PRA mod p = 2^6 mod 23 = 64 mod 23 = 18

Partner calculated:
KB = PUA^PRB mod p = 4^PRB mod 23 = 18

We confirmed KA = KB = 18 — exchange successful!

**Public values (visible to everyone):** p=23, g=14, PUA=4, PUB=2

**Private values (known only to me):** PRA=6, shared secret K=18

---

## Why DHKE is Secure

An attacker seeing p=23, g=14, PUA=4 would need to solve: 14^x mod 23 = 4, find x. For small numbers this is easy (x=6). But for real DHKE with a 2048-bit prime, this discrete logarithm problem is computationally infeasible with current technology.

---

## Task 2: DHKE in OpenSSL

```bash
# Generate DH parameters
openssl dhparam -out dhparams.pem 2048

# Generate my DH key pair
openssl genpkey -paramfile dhparams.pem -out dh_private.pem
openssl pkey -pubout -in dh_private.pem -out dh_public.pem

# Generate shared secret using partner's public key
openssl pkeyutl -derive -inkey dh_private.pem -peerkey partner_dh_public.pem -out shared_secret.bin

# View and compare the shared secret
xxd shared_secret.bin
```

My partner and I compared the output of xxd shared_secret.bin and confirmed the bytes were identical on both sides. The shared secret was the same without either of us ever transmitting it.

OpenSSL used a 2048-bit prime for p compared to my tiny p=23 in the manual exercise. The real-world strength of DHKE comes entirely from the size of p.

---

## Task 3: MITM Attack on DHKE (Optional)

Consider Alice posts ServerKeyExchange{p=23, g=14, PU=4} and Bob responds with ClientKeyExchange{PU=2}.

Eve the attacker intercepts both messages:

1. Eve intercepts Alice's ServerKeyExchange before Bob sees it
2. Eve generates her own private key PRE=3, calculates PUE = 14^3 mod 23 = 5
3. Eve sends Bob a fake ServerKeyExchange with PU=5 (pretending to be Alice)
4. Eve intercepts Bob's ClientKeyExchange (PUB=2)
5. Eve sends Alice a fake ClientKeyExchange with PU=5 (pretending to be Bob)

Result:
- Alice computes: K = 5^6 mod 23 = 8 (shared with Eve, not Bob)
- Bob computes: K = 5^PRB mod 23 = 8 (shared with Eve, not Alice)
- Eve can decrypt and re-encrypt all traffic between them

**Assumptions required for attack to succeed:**
- Eve can intercept AND modify messages in transit (active attacker)
- No authentication of public keys (no certificates)
- Neither Alice nor Bob can verify who they are talking to

**Why this fails in TLS:** TLS authenticates the server's DHKE parameters using a digital signature with the server's RSA private key. The client verifies this using the server's certificate. Eve cannot forge a valid signature without the private key, so the MITM attack fails.

---

## Reflection

This week was conceptually the most interesting so far. The DHKE algorithm seems almost magical — two parties compute the same secret without ever sending it. Working through the manual example with real numbers made the algebra very concrete.

The key mathematical insight is:
(g^a)^b mod p = g^(ab) mod p = (g^b)^a mod p
The order of operations does not matter — both sides arrive at the same value.

The MITM attack analysis was particularly enlightening. DHKE alone provides no authentication — it only protects against passive eavesdroppers. Authentication requires certificates, which is exactly what Week 9 covers. This explains the complete TLS design: DHKE for the secret + RSA certificate for authentication.

I also noticed that OpenSSL's dhparam command took several seconds to generate 2048-bit parameters because finding a large prime is computationally expensive. This is why TLS servers use pre-defined named groups rather than generating fresh parameters each time.

# Week 7 Journal

[Return to contents](README.md)

## Transport Layer Security (TLS)

This week we analysed real TLS packet captures using Wireshark, compared TLS 1.2 and TLS 1.3, and set up a cloud Linux VM for the security project.

---

## Task 1: TLS 1.2 with Diffie-Hellman Capture Analysis

I opened the TLS 1.2 DH packet capture in Wireshark and applied the display filter tls to show only TLS messages.

### Message Sequence Diagram
Web Browser (192.168.1.12)          Web Server (103.3.63.107)
|                                       |
|-------- ClientHello ----------------->|
|                                       |
|<------- ServerHello ------------------|
|<------- Certificate ------------------|  (RSA public key of server)
|<------- ServerKeyExchange ------------|  (DH: p, g, PUserver)
|<------- ServerHelloDone --------------|
|                                       |
|-------- ClientKeyExchange ----------->|  (DH: PUclient)
|-------- ChangeCipherSpec ------------>|
|-------- Encrypted Handshake --------->|
|                                       |
|<------- NewSessionTicket -------------|
|<------- ChangeCipherSpec -------------|
|<------- Encrypted Handshake ----------|
|                                       |
|========= Encrypted Application Data ==|

Encrypted messages: Everything from ChangeCipherSpec onwards is encrypted. The handshake messages up to ServerHelloDone are in plaintext so both parties can negotiate before a shared key exists.

### Q3: DH Key Size

From the ServerKeyExchange packet:
Diffie-Hellman Server Params
p Length: 256
p [truncated]: ffffffffffffffff...
p = 256 bytes = 2048 bits

### Q4: DH Key Sizes Client is Willing to Use

In the ClientHello, the supported_groups extension lists:
- ffdhe2048 (2048-bit finite field DH)
- ffdhe3072 (3072-bit finite field DH)
- Plus several ECDH groups (secp256r1, x25519, x448)

Packet: ClientHello (packet 4)

### Q5: RSA Key Size in Certificate

From the Certificate message (packet 6):
subjectPublicKey [truncated]
modulus: 0x00d652a082e0a90a...
INTEGER (pkcs1.modulus), 257 bytes
257 bytes stored = 256 bytes = 2048-bit RSA key. The extra byte is a leading 0x00 indicating a positive integer in ASN.1 encoding.

### Q6: Delay Calculation (RTT = 50ms)

| Round Trip | Client sends | Server responds |
|---|---|---|
| RTT 1 | ClientHello | ServerHello + Certificate + ServerKeyExchange + ServerHelloDone |
| RTT 2 | ClientKeyExchange + ChangeCipherSpec + EncryptedHandshake | NewSessionTicket + ChangeCipherSpec + EncryptedHandshake |
| RTT 3 | Application Data (HTTP GET) | Application Data (web page) |

Total: 3 RTT × 50ms = 150ms

---

## Task 2: TLS 1.3 Capture Analysis

I opened the TLS 1.3 capture and loaded the SSL Key Log file via Edit → Preferences → Protocols → TLS to decrypt packets.

### Message Sequence Diagram (TLS 1.3)
Client                                  Server
|                                       |
|-- ClientHello (ECDH PUclient) ------->|
|                                       |
|<-- ServerHello (ECDH PUserver) -------|
|<-- ChangeCipherSpec ------------------|
|<-- EncryptedExtensions ---------------|  [ENCRYPTED]
|<-- Certificate -----------------------|  [ENCRYPTED]
|<-- CertificateVerify -----------------|  [ENCRYPTED]
|<-- Finished --------------------------|  [ENCRYPTED]
|                                       |
|-- ChangeCipherSpec ------------------>|
|-- Finished -------------------------->|  [ENCRYPTED]
|-- GET /bm HTTP/1.1 ----------------->|  [ENCRYPTED]
|                                       |
|<-- NewSessionTicket ------------------|  [ENCRYPTED]
|<-- HTTP/1.1 200 OK -------------------|  [ENCRYPTED]

Key difference: In TLS 1.3 the client includes its ECDH public key in the very first ClientHello. The server can derive keys immediately. Everything after ServerHello is already encrypted — including the Certificate which was plaintext in TLS 1.2.

### Q3: Cipher Suite Used

From ServerHello:
Cipher Suite: TLS_AES_128_GCM_SHA256 (0x1301)
- AES-128-GCM for authenticated encryption
- SHA-256 for key derivation
- ECDH used by default (not listed in cipher suite in TLS 1.3)

### Q4: Differences from TLS 1.2

| Feature | TLS 1.2 DH | TLS 1.3 |
|---|---|---|
| Key exchange in cipher suite | Yes | No (always ECDH) |
| Certificate in plaintext | Yes | No (encrypted) |
| Separate ServerKeyExchange | Yes | No (in ServerHello) |
| Authentication | AES + HMAC | AES-GCM (AEAD) |

### Q5: Delay Comparison

- TLS 1.3: 2 RTT × 50ms = 100ms
- TLS 1.2: 3 RTT × 50ms = 150ms
- Saving: 50ms (33% reduction in delay)

---

## Task 3: Cloud VM Setup

I deployed a Linux Ubuntu VM on a cloud provider for the security project.

```bash
# VM1 (server) - listen on port 12345
nc -l 12345

# VM2 (client) - connect to server
nc <VM1_IP_address> 12345
```

I typed a test message in the client terminal and it appeared on the server, confirming TCP connectivity. I had to add a firewall inbound rule for port 12345 in the cloud console — without it the connection was refused.

---

## Reflection

This week was the most practical so far seeing real network packets and recognising the cryptographic components we studied (DH key exchange, RSA certificates, AES cipher suites) made everything feel concrete.

The biggest insight was understanding why TLS 1.3 is faster. The client sends its ECDH public key speculatively in the first message, allowing the server to derive keys immediately without an extra round trip. TLS 1.2 needed a separate ServerKeyExchange message first.

I was also surprised that TLS 1.3 encrypts the Certificate message. In TLS 1.2 it was plaintext, meaning a passive observer could see which website a client was connecting to from the certificate CommonName. TLS 1.3 hides this too.

The cloud VM setup was straightforward but I forgot to open the firewall port initially a simple but real security lesson. Default-deny firewall rules are the correct security posture.

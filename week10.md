# Week 10 Journal

[Return to contents](README.md)

---

## Quantum Computing and Cryptography

This week was different from the other weeks as there were no practical tasks. Instead we discussed quantum computing concepts and how they relate to cryptography.

---

## Q1: How many states does a qubit have?

A normal bit has two states, 0 or 1. A qubit has more than two states because it can be 0, 1, or both at the same time which is called superposition. I found this hard to understand at first because it doesnt make sense in everyday life but apparently this is how things work at a quantum level.

---

## Q2: Qubit in state 0.8|0⟩ + 0.6|1⟩ — what happens when measured?

When the qubit is measured it collapses to either 0 or 1. The probability of each is calculated by squaring the coefficients:
- 0.8² = 0.64 so 64% chance of getting 0
- 0.6² = 0.36 so 36% chance of getting 1

So most likely the output is 0 but it is not always guaranteed. After measuring, the superposition is destroyed.

---

## Q3: Quantum computing reduces brute-force attacks on AES. True or False?

True, but it does not completely break AES. There is a quantum algorithm called Grover's algorithm which can speed up brute force attacks. For AES-128 it reduces the work from 2^128 to 2^64 which is a big reduction. However the fix is simple, just use AES-256 instead which brings security back up to 2^128 even against quantum computers. So AES is not broken, it just needs longer keys.

---

## Q4: Quantum cryptography is about breaking RSA. True or False?

False. I actually thought this was true before the discussion which was a bit embarrassing. Quantum cryptography is actually about building more secure systems using quantum physics, not breaking existing ones. The thing that breaks RSA is Shor's algorithm which is quantum cryptanalysis, a different thing. Quantum cryptography like BB84 is used to build better and more secure key exchange.

---

## Q5: Quantum entanglement allows faster than light communication. True or False?

False. Even though entangled particles affect each other instantly across any distance, you cannot use this to send information faster than light. The reason is the measurement outcome is random and you cannot control it. So you cannot encode a message in it. I found this confusing because it seems like it should work but it doesnt because of the randomness.

---

## Q6: What algorithm can break RSA?

Shor's Algorithm. It was developed by Peter Shor in 1994. RSA security relies on the difficulty of factorising large numbers. Shor's algorithm can do this factorisation very quickly on a quantum computer. If someone had a powerful enough quantum computer they could break RSA and recover private keys. Current quantum computers are not powerful enough to do this yet though.

---

## Q7: BB84 is similar to which classical algorithm?

BB84 has a similar aim to Diffie-Hellman Key Exchange. Both try to allow two people to agree on a shared secret key over a public channel. The difference is that DHKE uses maths which can be broken by Shor's algorithm, while BB84 uses quantum physics where any eavesdropper disturbs the quantum states and gets detected automatically.

---

## Q8: How many qubits do current quantum computers have?

From what I read, current state of the art quantum computers have around 1000+ qubits. IBM has announced systems with over 1000 qubits. However these are noisy qubits with high error rates. To actually break RSA you would need millions of error corrected logical qubits which is way beyond what exists today.

---

## Reflection

This week was interesting even though there was no coding. The main thing I learnt is that quantum computers are a real future threat to cryptography but not an immediate one right now. RSA and Diffie-Hellman are most at risk because of Shor's algorithm. AES is safer because Grover's algorithm only halves the effective key length which can be fixed with longer keys.

What surprised me was learning that quantum technology can also be used to build better cryptography like BB84, not just to break existing cryptography. I always thought of quantum computers as only a threat. I also found out that NIST has already started standardising post-quantum algorithms to replace RSA which shows how seriously this threat is being taken even now.

# Zero Knowledge Proofs Part 2

## Quick Recap from Part 1

Zero knowledge proofs let you prove something is true without revealing WHY. They need three properties:

1. **Completeness:** Honest provers convince verifiers
2. **Soundness:** Liars get caught
3. **Zero-knowledgeness:** No information leaks (except "it's true")

We proved #3 using a "Simulator" - a fake prover with no knowledge that produces identical-looking transcripts by rewinding time (or computer programs). If the fake looks identical to the real thing, the real thing must leak zero information.

---

## Two Types of Statements You Can Prove

### Type 1: Facts About the Universe

- "This number is composite"
- "This graph has a 3-coloring"

### Type 2: What YOU Know (Proofs of Knowledge)

- "I know the factors of this number"
- "I know the secret password"
- "I know the private key for this public key"

**The difference matters!** You can prove a number is composite WITHOUT knowing its factors. Type 2 is much more useful in real life.

---

## The Schnorr Protocol: A Real-World Proof of Knowledge

This is what we actually use! Created by Claus-Peter Schnorr in the 1980s.

### The Problem

Alice has a secret key `a` and a public key `g^a mod p`. She wants to prove she knows `a` without revealing it to Bob.

_Think of it like:_ Your password creates a hash. You want to prove you know the password without sending the password.

### The Protocol (Step by Step)

**Setup:**

- Alice has: secret key `a`, public key `g^a`
- Bob knows Alice's public key

**The Interaction:**

```
Alice (Prover)                    Bob (Verifier)
--------------                    --------------

1. Pick random k
2. Send g^k          --------->

3.                   <---------  Pick random challenge c

4. Compute z = ac + k
5. Send z            --------->

6.                              Check: g^z = (g^a)^c * g^k
                                If yes: Alice knows a!
```

### Why It Works (Completeness)

Let's verify Bob's check if Alice is honest:

```
g^z = g^(ac + k)          [substitute z]
    = g^(ac) * g^k        [split the exponent]
    = (g^a)^c * g^k       [rearrange]
    ✓ This matches!
```

If Alice knows `a`, Bob will always accept.

---

## Proving Soundness: The Knowledge Extractor

**Question:** How do we prove Alice can't cheat without knowing `a`?

**Answer:** Show that a "Knowledge Extractor" exists - a special verifier that can extract Alice's secret by rewinding.

### The Extractor's Trick

```
Run 1:
Alice sends g^k
Extractor challenges with c₁
Alice responds with z₁ = ac₁ + k

REWIND! ⏪

Run 2 (with same g^k):
Extractor challenges with c₂ (different!)
Alice responds with z₂ = ac₂ + k

Now solve:
z₁ - z₂ = a(c₁ - c₂)
Therefore: a = (z₁ - z₂)/(c₁ - c₂)
```

**The insight:** If Alice completes the proof successfully, the Extractor can rewind and extract `a`. This proves Alice must know `a` to succeed.

**CRITICAL WARNING:** If you ever use the same `k` twice in real life, an attacker can extract your secret key! (This has broken real systems with bad random number generators - see the PlayStation 3 ECDSA hack!)

---

## Proving Zero-Knowledge: The Simulator

To prove zero-knowledge, we need a Simulator that can fake convincing transcripts WITHOUT knowing `a`.

**Special assumption needed:** Bob (the Verifier) must be "honest" - he picks his challenge `c` randomly, not based on what Alice sends.

### The Simulator's Magic Trick

```
Goal: Fake a proof without knowing a

1. Send g^k₁ to Bob
2. Bob challenges with c
3. REWIND! ⏪
4. Pick random z
5. Compute g^k₂ = g^z / (g^a)^c
6. Send g^k₂ to Bob
7. Bob challenges with c again (same random choice!)
8. Send z

Bob checks: g^z = (g^a)^c * g^k₂  ✓ Perfect!
```

**What this proves:** The Simulator created a perfect-looking transcript without knowing `a`. Since the fake is identical to the real protocol, the real protocol must leak zero information.

---

## Making It Non-Interactive: The Fiat-Shamir Transform

**Problem:** So far, Bob needs to be online to send challenges. Can Alice create a proof Bob can verify later?

**Solution:** Use a hash function to generate the challenge!

### The Non-Interactive Version

```
Alice's process:
1. Pick random k, compute g^k
2. Compute challenge: c = Hash(g^k || Message)
   [The hash function plays Bob's role!]
3. Compute z = ac + k
4. Send proof: (g^k, z, Message)

Bob's verification:
1. Compute c = Hash(g^k || Message)
2. Check: g^z = (g^a)^c * g^k
```

**The genius:** The hash function picks the "challenge" deterministically. Alice can't cheat because she can't predict the hash output before committing to `g^k`.

---

## Bonus: This Is Also a Signature Scheme!

If you include a message `M` in the hash, you've just created a **digital signature**:

- Only someone who knows secret key `a` can create valid proofs
- The proof is tied to message `M` (because `M` is in the hash)
- This is the **Schnorr signature scheme**!

**Real-world use:** EdDSA (used in SSH, Signal, cryptocurrency) is based on this exact idea.

---

## The Big Picture

### Theory vs. Practice

**Part 1 showed:** You can prove ANY NP statement in zero knowledge (via the graph coloring protocol)

- Amazing theoretically
- Totally impractical (way too slow)

**Part 2 shows:** Efficient protocols for specific useful problems

- Proving you know a secret key (Schnorr)
- Creating digital signatures
- Actually used every day in SSH, cryptocurrencies, secure messaging

---

## Key Concepts Summarized

### 1. Knowledge Extractor (for Soundness)

Rewinds the Prover to extract secrets. If it can extract, the Prover must really know the secret.

### 2. Simulator (for Zero-Knowledge)

Rewinds the Verifier to create fake transcripts. If fakes look real, the real protocol leaks nothing.

### 3. The Rewind Trick

Both Simulator and Extractor use rewinding - but on different parties and for different reasons!

### 4. Fiat-Shamir Transform

Hash function = instant non-interactive proofs. Game changer for real applications.

---

## Think About It

The same protocol (Schnorr) demonstrates BOTH properties:

- **Soundness:** Knowledge Extractor can extract the secret (by rewinding Prover)
- **Zero-knowledge:** Simulator can fake proofs (by rewinding Verifier)

Both use rewinding, but the contradiction disappears because rewinding doesn't happen in the real protocol run. Mind-bending but beautiful!

---

## Where This Is Used Today

- **SSH authentication:** Prove you have the private key without sending it
- **Cryptocurrency:** Prove transactions are valid without revealing amounts
- **Digital signatures:** EdDSA (used in Signal, SSH, cryptocurrencies)
- **Blockchain privacy:** zk-SNARKs and zk-STARKs (advanced versions)

**The bottom line:** Zero knowledge proofs went from impossible-sounding theory to technology you use every day.

# Proving You Knew Something *Then* Without Revealing It *Until Later*

> **TL;DR** Hash the secret message with a strong hash (e.g.
> `SHA‑256`), publish only the hash *h* on a public, timestamp‑trusted
> platform, and reveal the original message *m* whenever you like.  The
> hash acts as an unforgeable **commitment** to what you already knew at
> the earlier time.

---

## 🌟 Why Would You Do This? (Benefits & Applications)

| Scenario | How the Hash‑Commit Trick Helps |
| --- | --- |
| **Idea priority / IP** | Publish the hash of an invention note; reveal the full note later to prove you had it first. |
| **Predictions & wagers** | Commit to a forecast *before* the outcome; disclose afterwards to show honesty or win a bet. |
| **Governance & voting** | Cast a vote as a hash so early voters can’t influence late voters; reveal all votes after the deadline. |
| **Bug bounties / responsible disclosure** | Prove you had the vulnerability description before a patch but keep details secret until the vendor fixes it. |
| **Time‑capsule publishing** | Tease upcoming blog posts, artwork, or research with a commitment that can’t be altered retroactively. |

A single‑line hash is cheap to publish almost anywhere (tweets, Git commits, Discord messages, blockchain transactions, etc.) and later becomes undeniable evidence of precedence.

---

## 🔢 How the Math Works (Commit‑and‑Reveal)

1. **Choose your message** `m` (any length).
2. *(Optional but recommended)* Generate a random 128‑bit salt `s` and
   prepend it: `m′ = s || m`.  This prevents dictionary/brute‑force
   guessing if *m* is short or predictable.
3. **Compute the commitment hash**

   ```text
   h = SHA‑256(m′)
   ```

4. **Publish `h` (and maybe the salt `s`) on a public platform** at
   time `T₁`.  Make sure the platform provides *immutable* timestamps
   or reinforce it with an external timestamp authority (see below).
5. **Later**, at any time `T₂ > T₁`, publish the reveal package: `m′`
   (which contains your original `m`, or `s` + `m`).  Anyone can verify:

   ```text
   SHA‑256(m′) == h   ✅
   ```

### Security Properties

| Property | What you need | Why SHA‑256 works |
| --- | --- | --- |
| **Hiding** | Pre‑image resistance | Without solving 2²⁵⁶ work, an observer can’t learn *m* from `h`. |
| **Binding** | Collision resistance | You can’t find *m* and *m*′ ≠ *m* that hash to the same `h`. |

Add **authorship** by signing the hash:

```text
sig = Ed25519_sign(private_key, h || T₁)
```

Publish `(h, sig, public_key)` — now the commitment is linked to you.

Add **trustworthy time** by:

* Pushing the hash into an append‑only blockchain (e.g. Bitcoin OP_RETURN, Ethereum calldata).
* Requesting an RFC 3161 time‑stamp token from a TSA and storing it with the hash.

---

## ⚠️ Caveats & Potential Risks

| Risk / Limitation | Mitigation |
| --- | --- |
| **Mutable timestamps** — Social‑media operators can delete or alter posts. | Cross‑post to multiple platforms; store in Git history; embed in a blockchain; use a TSA. |
| **Small / guessable messages** — Observers brute‑force all possibilities. | Always include a random salt `s` (publish `s` only on reveal). |
| **Future hash collisions** — If SHA‑256 is ever broken, someone might forge a different `m′`. | Hash with two different families (e.g. `SHA‑256` + `SHA‑3`), or migrate commitments to stronger hashes over time. |
| **Key compromise** — If you use a signature and the private key leaks, others can sign fake commitments. | Rotate keys; publish the public‑key fingerprint along with commitments; revoke old keys publicly. |
| **Platform disappearance** — The site hosting your hash could vanish. | Mirror your hash (and TSA token) in distributed archives or a public blockchain. |

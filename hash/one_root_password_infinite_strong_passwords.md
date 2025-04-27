# One Root Password, Infinite Strong Passwords  
*A friendly, open‑source recipe for ditching weak, reused logins*

---

## 1  Why ordinary passwords fail us

* **We pick predictable secrets.** Year after year, "123456", "password", and birthdays still flood breach lists.
* **We reuse them everywhere.** A leak at one shop hands attackers the keys to your bank, e‑mail, and cloud drives.
* **We forget the rest.** Remembering dozens of unique, complex strings is nearly impossible without help.

Password‑manager vaults solve the memory problem, but they introduce a new worry: the vault file or cloud sync itself can be stolen, phished, or lost.

---

## 2  A simpler, safer idea

1. **Memorise one strong *root password*.**  
2. **Season it with a *label* (usually the site’s main domain).**  
3. **Let a tiny open‑source program** combine the two and spit out that site’s unique password on demand.

Because the recipe is *deterministic*, you never store or sync anything.  Anyone can read the code and verify nothing fishy happens under the hood.

---

## 3  How it feels in practice

| You do… | The generator does… |
|---------|--------------------|
| Type your root password **once** | Runs a memory‑hard *key‑stretch* to turn it into a heavyweight secret |
| Type / confirm the label (e.g. `google.com`) | Mixes secret + label with *HMAC‑SHA‑256* (see § 4) |
| Click **Generate** | Formats the bytes so they comply with the site’s length & character rules |
| Copy / auto‑fill | Immediately forgets everything in RAM |

That’s it—no vault, no sync service, no subscription.

---

## 4  Peeking under the hood

```text
seed   = Argon2id(root_password, salt="pwmgr", time=3, memory=64 MiB)
raw    = HMAC_SHA256(key=seed, msg="label:counter")
passwd = render(raw, length, recipe)   # make it fit the site’s rules
```

### 4.1  Argon2id — Fortifying the root

*Argon2id* is a **memory‑hard** password‑hashing function: it forces an attacker to spend RAM as well as CPU. Cracking a weak root that takes *you* 0.1 s to type can cost *them* thousands of dollars in GPU hardware.

> In plain English: even if you picked only four random words, Argon2id turns brute‑forcing that choice into an unprofitable slog.

### 4.2  Why HMAC‑SHA‑256 and not just SHA‑256?

SHA‑256 is like a powerful blender—you toss in data and get 32 random‑looking bytes.  Sounds great, but it has two quirks:

| Quirk of plain SHA‑256 | Real‑world trouble |
|------------------------|--------------------|
| **Length extension.**  Knowing `Hash(message)` lets an attacker compute `Hash(message‖extra)` without seeing *message*. | If a site leak reveals your `Hash(root‖"google.com")`, a hacker could forge `Hash(root‖"google.comXYZ")`—a password tied to a label you never typed. |
| **No formal key separation.**  The hash treats everything as one lump; it’s not a proven *keyed* function. | Future cryptanalytic quirks could expose the root when it sits at the start of the message. |

**HMAC** wraps SHA‑256 in two quick passes with fixed pads.  That kills length‑extension and gives rock‑solid proofs that *nobody without the key can predict or forge the output.*  It costs one extra hash call and ships in every standard library—nothing to lose, rigour to gain.

### 4.3  Render — Making the bytes human‑hostile, site‑friendly

Sites demand specifics: “16 chars max”, “at least one digit”, “no `:` character”…  *Render* is the final formatting step:

1. Base64‑url encode the raw bytes (A‑Z, a‑z, 0‑9, `-_`).
2. If the policy insists on uppers/lowers/digits/symbols, sprinkle in what’s missing.
3. Trim or pad to the exact length the site allows.

All policy juggling happens **after** the cryptography, so you keep full entropy while satisfying even the strangest legacy systems.

---

## 5  Risks & fixes

| Risk | Why it matters | Fix |
|------|---------------|-----|
| **Root compromise** | One leak unlocks every account. | Make the root ≈90‑bit entropy (e.g. 4‑5 diceware words + symbols).  Pair with hardware‑token 2FA. |
| **Root loss** | Forget it and you’re locked out forever. | Keep a paper backup in a safe. |
| **Typos in label** | `goggle.com` ≠ `google.com` → wrong password. | Generator shows canonical domain before output. |
| **Policy oddities** | Some sites ban symbols or cap length. | Generator stores a tiny rule table & massages output. |
| **Need to rotate** | Breach forces a new password. | Add a *counter* (`1, 2…`) to the label; remember only non‑zero counters. |
| **Algorithm ages** | Crypto best‑practice changes. | Prepend a version tag (e.g. `v2_`) to every password so you can upgrade later. |

---

## 6  Why open source is key

With fewer than 150 lines of public code, anyone can audit—or even memorise—the algorithm.  There’s no hidden database or analytics call, just pure maths you can run on a Raspberry Pi, in a browser, or on an air‑gapped laptop.

> **Takeaway:** learn one strong secret, keep it safe, and let transparent code handle the rest.  Your login security jumps from “guessable & reused” to “unique, high‑entropy, and breach‑proof”—with no vault to guard.

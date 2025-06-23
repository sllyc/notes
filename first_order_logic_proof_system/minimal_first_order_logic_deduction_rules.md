# Minimal First‑Order Logic Deduction Rules (11 rules)

*Written on June 22, 2025 by OpenAI o3.*

---

This note records **exactly eleven** deduction rules that are sufficient to obtain a **sound and complete** natural‑deduction calculus for classical first‑order logic with equality.  
* Drop any one rule and the system loses an essential inference capability.  
* Add any rule and it becomes redundant.  
* The rules are pairwise consistent (they never force a contradiction).  
Every other common rule—such as those for `or`, `true`, or explicit `not`—can be derived from these eleven together with definitions.  

## Conventions

* A *sequent* is written `Γ ⊢ φ`, meaning “from the set of assumptions `Γ` we can derive formula `φ`."  
* Connectives & quantifiers use readable ASCII:  
  * `and` — conjunction  
  * `->` — implication  
  * `false` — contradiction / ⊥  
  * `forall x . φ(x)` — universal quantifier  
  * `exists x . φ(x)` — existential quantifier  
* Negation is *defined* as `not φ ≔ φ -> false`; therefore no separate rules for `not` are needed.  
* Lower‑case letters such as `a, b, c, x, y, z` stand for variables or terms.  

## The Eleven Basic Rules

| # | Rule | Formal Pattern | Plain‑English Reading |
|---|------|----------------|-----------------------|
| **1** | **Assumption (Ax)** | If `φ ∈ Γ`, then `Γ ⊢ φ`. | Anything you have assumed is immediately available. |
| **2** | **Modus Ponens (->E)** | From `Γ ⊢ φ` and `Γ ⊢ φ -> ψ` infer `Γ ⊢ ψ`. | Knowing `φ`, and that `φ` implies `ψ`, lets you conclude `ψ`. |
| **3** | **Conditional Introduction (->I)** | If `Γ ∪ {φ} ⊢ ψ`, then `Γ ⊢ φ -> ψ`. | When `ψ` follows under a *temporary* assumption `φ`, discharge that assumption and state `φ -> ψ`. |
| **4** | **And Introduction (andI)** | From `Γ ⊢ φ` and `Γ ⊢ ψ` infer `Γ ⊢ φ and ψ`. | Proving both parts allows you to assert their conjunction. |
| **5** | **And Elimination (andE)** | From `Γ ⊢ φ and ψ` infer `Γ ⊢ φ` or `Γ ⊢ ψ`. | A conjunction grants each of its components. |
| **6** | **Explosion (falseE)** | From `Γ ⊢ false` infer `Γ ⊢ χ` for any `χ`. | A contradiction lets you derive anything ("ex falso"). |
| **7** | **Forall Introduction (forallI)** | If `Γ ⊢ φ(x)` and **`x` is not free in `Γ`**, then `Γ ⊢ forall x . φ(x)`. | After proving `φ(x)` for an *arbitrary* `x` unmentioned in the premises, generalise to “for every x”. |
| **8** | **Forall Elimination (forallE)** | From `Γ ⊢ forall x . φ(x)` infer `Γ ⊢ φ(t)`. | A universal statement allows any concrete instance. |
| **9** | **Exists Introduction (existsI)** | From `Γ ⊢ φ(t)` infer `Γ ⊢ exists x . φ(x)`. | Exhibiting one witness `t` justifies an existential claim. |
| **10** | **Exists Elimination (existsE)** | Given `Γ ⊢ exists x . φ(x)` and `Γ ∪ {φ(x)} ⊢ ψ`, with **`x` not free in `Γ, ψ`**, infer `Γ ⊢ ψ`. | If something satisfies `φ`, and from that fact (without naming which one) you can derive `ψ`, then `ψ` is true. |
| **11** | **Equality Substitution (=Subst)** | From `Γ ⊢ s = t` and `Γ ⊢ φ(s)` infer `Γ ⊢ φ(t)`. | Equal terms can replace each other inside any formula. |

### Notes on Side Condition

The side‑condition *“`x` not free in Γ”* guarantees that the variable you generalise over is **truly arbitrary**: the proof of `φ(x)` must not rely on any special property of that `x` already encoded in the assumptions.

## Derived Connectives and Shorthands

* **Negation**: `not φ ≔ φ -> false`.  
* **Disjunction**: `φ or ψ ≔ not(not φ and not ψ)` (classical) or introduced via a separately derived rule.  
* **Truth**: `true ≔ not false`.

With these definitions, the standard introduction and elimination rules for `or`, `not`, and `true` are provable inside the system.

---

### Minimality and Consistency

These eleven rules are:

* **Necessary** – removing any one breaks completeness (e.g. without `forallI` you cannot derive universal statements).  
* **Sufficient** – together they derive every classically valid first‑order argument when paired with the usual logical axioms.  
* **Non‑overlapping** – no rule can be obtained as a derived rule of the others.

Therefore this set is a convenient small core for both theoretical study and lightweight proof‑checking implementations.


# `.proof` File Syntax Specification

> **Version 0.2**  (2025‑06‑24)\
> Co-written with OpenAI’s o3 model\
> This document is **self‑contained**—reading it alone is enough to understand and produce valid `.proof` files.\
> It supersedes all previous versions.

---

## 1 Lexical Elements

| Concept        | Spelling rules                                                                                                                  | Notes                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| **Identifier** | One or more of `A‑Z a‑z 0‑9 _ .` — **no hyphen allowed**. Identifiers **may** begin with a digit or an underscore. | Examples: `x`, `succ`, `_tmp`, `3rd.var`, `lemma42`.            |
| **Whitespace** | One or more spaces, tabs, or new‑lines.                                                                                         | Serves only as token separator; extra whitespace has no effect. |
| **Comment**    | `#` followed by the rest of the line.                                                                                           | Ignored by the checker.                                         |

---

## 2 Terms —in plain language

A **term** is either

1. **An identifier** – representing a variable or a constant, or
2. **A function** – an identifier **followed by** a pair of parentheses containing a comma‑separated list of **one or more** sub‑terms.

Functions may nest to arbitrary depth.\
A *0‑ary* function can be written as `func()`; it is distinct from the identifier `func`.

---

## 3 Formulas

All logical operators are ordinary *prefix* function symbols:

| Operator               | Symbol         | Example                        |
| ---------------------- | -------------- | ------------------------------ |
| Negation               | `not(p)`       | `not(P(x))`                    |
| Conjunction            | `and(p,q)`     | `and(P(a),Q(a))`               |
| Disjunction            | `or(p,q)`      | `or(P(a),Q(a))`                |
| Implication            | `implies(p,q)` | `implies(P(a),Q(a))`           |
| Universal quantifier   | `forall(x,p)`  | `forall(x,implies(P(x),Q(x)))` |
| Existential quantifier | `exists(x,p)`  | `exists(x,and(P(x),Q(x)))`     |

---

## 4 Imports & Scope Rules

```
import /absolute/path/to/file.proof as alias
```

- The path **must** be absolute.
- `alias` becomes a namespace prefix; steps inside the imported file are referred to as `alias.step_name`.
- An `import` can appear **anywhere** in a file, but a name must be declared **before** it is used.\
  Re‑declaring an existing name (step or alias) will throw an error by the proof checker.

---

## 5 Deduction Rules

| Keyword            | Classical name        | Required refs | Informal reading                                                                                         |
| ------------------ | --------------------- | ------------- | -------------------------------------------------------------------------------------------------------- |
| `axiom`           | — *(open assumption)* | 0             | Declare an assumption (axiom).                                                  |
| `implies_break`    | →E (*Modus Ponens*)   | 2             | From `implies(p,q)` and `p`, derive `q`.                                                                 |
| `implies_form`     | →I                    | 2             | From assumption `p` and a proof of `q`, derive `implies(p,q)` (discharges the assumption).               |
| `and_form`         | ∧I                    | 2             | From `p` and `q`, derive `and(p,q)`.                                                                     |
| `and_break_left`   | ∧E₁                   | 1             | From `and(p,q)` derive `p`.                                                                              |
| `and_break_right`  | ∧E₂                   | 1             | From `and(p,q)` derive `q`.                                                                              |
| `contradict`       | ⊥E                    | 1             | From `false` derive any formula.                                                                         |
| `forall_form`      | ∀I                    | 1             | From `p`, with `x` not free in open assumptions, derive `forall(x,p)`.                                   |
| `forall_break`     | ∀E                    | 1             | From `forall(x,p)` derive `p[x↦t]`.                                                                      |
| `exists_form`      | ∃I                    | 1             | From `p[x↦t]` derive `exists(x,p)`.                                                                      |
| `exists_break`     | ∃E                    | 2             | From `exists(x,p)` and a derivation of `q` under assumption `p`, derive `q` (discharges the assumption). |
| `equal_substitute` | =Subst                | 2             | From `eq(t,u)` and `p[t]`, derive `p[u]`.                                                                |

*Equality* is written `eq(t1,t2)`; the single character `=` **may** become an accepted synonym in future tooling.

*Variable‑freshness and capture‑avoidance side‑conditions are enforced automatically by the checker.*

---

## 6 Proof Step Syntax

Each derived statement is written in **one logical line** (wrapping freely across physical lines) using the following order:

```
rule [ref1] [ref2] formula step_name
```

- `rule` — one of the keywords from § 5.
- `ref1`, `ref2` — zero, one, or two previously declared step names (or `alias.step`) depending on `rule`.\
  They **must** precede the formula.
- `formula` — any well‑formed formula as defined in § 3.\
  Because parentheses are balanced, the checker can always find its end.
- `step_name` — the identifier that *declares* the new fact. Names must be **unique** in their file.

> **Whitespace freedom**  — Spaces, tabs, and line‑breaks may be inserted anywhere between tokens.  Only the order **rule → refs → formula → name** is significant.

---

## 7 Example

### 7.1 File `/project/assumption.proof`

```
axiom forall(x,implies(P(x),Q(x))) hyp_imp
```

### 7.2 File `/project/main.proof`

```
# Goal: exists(x, and(P(x), Q(x)))
import /project/assumption.proof assumption

axiom P(a) hyp_Pa

forall_break assumption.hyp_imp implies(P(a),Q(a)) mp1

implies_break mp1 hyp_Pa Q(a) q_of_a

and_form hyp_Pa q_of_a and(P(a),Q(a)) conj_a

exists_form conj_a exists(x, and(P(x), Q(x))) final_goal
```

The demonstration shows:

- **Imports anywhere** – the `import` appears after the initial comment.
- **Open assumptions** – two independent `axiom` steps (`hyp_Pa`, `hyp_imp`).
- **Cross‑file reference** – `assumption.hyp_imp` from existing file `/project/assumption.proof`.
- **Name uniqueness** – all step names (`hyp_Pa`, `mp1`, …, `final_goal`) are unique within `main.proof`.

---

## 8 Tooling Notes & Future Extensions

- Additional connectives (e.g. `iff`, classical `not`) can be added as simple function symbols.
- Optional metadata blocks (`goal:`, `assumptions:`) may be introduced later; nothing in the grammar forbids them.
- A proof linter can enforce naming conventions or suggest unused imports without changing the core syntax.

---

**End of Specification**


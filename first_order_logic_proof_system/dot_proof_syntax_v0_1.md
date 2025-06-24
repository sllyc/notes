# `.proof` File Syntax Specification

> Version 0.1   (2025‑06‑23)\
> Written by OpenAI o3\
> This document is **self‑contained**—reading it alone is enough to understand and produce valid `.proof` files.

---

## 1  Lexical Elements

| Concept        | Spelling Rules                                      | Notes                                                     |          |                                                                                                          |
| -------------- | --------------------------------------------------- | --------------------------------------------------------- | -------- | -------------------------------------------------------------------------------------------------------- |
| **Identifier** | \`letter ( letter                                   | digit                                                     | \_ )\*\` | First char **must** be `A‑Z` or `a‑z`. No hyphens, no leading digits. Examples: `x`, `succ`, `lemma_42`. |
| **Whitespace** | one or more space / tab / newline / carriage‑return | Token separator. Extra whitespace has no semantic effect. |          |                                                                                                          |
| **Comment**    | `#` followed by the rest of the line                | The checker ignores comments entirely.                    |          |                                                                                                          |

---

## 2  Terms

```ebnf
term ::= IDENT                                   (* variable or constant *)
       | IDENT "(" term { "," term } ")"       (* function application, arity ≥ 0 *)
```

- Functions may nest arbitrarily deep.
- A **0‑ary** function *may* be written with empty parentheses (`zero()`), and is **distinct** from an identically‑named constant (`zero`).

---

## 3  Formulæ

All logical operators are expressed as *prefix* function symbols:

| Operator               | Function symbol | Example                                          |
| ---------------------- | --------------- | ------------------------------------------------ |
| Negation               | `not(φ)`        | `not(P(x))`                                      |
| Conjunction            | `and(φ, ψ)`     | `and(P(a), Q(a))`                                |
| Disjunction            | `or(φ, ψ)`      | (not needed by the minimal rule set but allowed) |
| Implication            | `implies(φ, ψ)` | `implies(P(a), Q(a))`                            |
| Universal quantifier   | `forall(x, φ)`  | `forall(x, implies(P(x), Q(x)))`                 |
| Existential quantifier | `exists(x, φ)`  | `exists(x, and(P(x), Q(x)))`                     |

*Equality* is treated as a built‑in binary predicate: `eq(t1, t2)` (the synonym `=` *may* be accepted by future tooling).

---

## 4  File‑level Directives

A `.proof` file may begin with **imports** and any number of **comments**. Everything is optional; order is free.

### 4.1  Importing earlier proofs

```
import /absolute/or/relative/path/to/file.proof as alias
```

\* `alias` becomes a namespace prefix. Steps inside the imported file are referred to as `alias.step_name`.

---

## 5  Proof Steps

Each derived statement **must** be named and is written in **two parts**:

```
step_name:  rule  [ref1] [ref2] :
    formula
```

- `step_name:` identifier introducing the new fact.
- `rule`     one keyword from § 6.
- `ref1`, `ref2` (previously proved step names or `alias.step` from an import). 0–2 references depending on the rule.
- The trailing colon `:` marks the end of the header; the formula may stay on the same line or wrap onto following lines until the next header or end‑of‑file.

> **Whitespace is irrelevant**—multiple spaces, tabs, and hard returns are all treated as separators.

### 5.1  Referencing temporary assumptions

Rules like `implies_form` and `exists_break` require an *assumption* step inside the reference list. You simply give that step a normal name and cite it like any other reference.

---

## 6  Deduction Rules

| Rule keyword      | Classical symbol  | Required refs | Produces                                                                                                  |
| ----------------- | ----------------- | ------------- | --------------------------------------------------------------------------------------------------------- |
| `assume`          | — (assumption)    | 0             | Declares a hypothesis; checker marks it *open* until discharged.                                          |
| `implies_break`   | →E (Modus Ponens) | 2             | From `implies(φ, ψ)` and `φ`, derive `ψ`.                                                                 |
| `implies_form`    | →I                | 2             | From assumption `φ` and a proof of `ψ`, derive `implies(φ, ψ)` (discharges the assumption).               |
| `and_form`        | ∧I                | 2             | From `φ` and `ψ`, derive `and(φ, ψ)`.                                                                     |
| `and_break_left`  | ∧E₁               | 1             | From `and(φ, ψ)` derive `φ`.                                                                              |
| `and_break_right` | ∧E₂               | 1             | From `and(φ, ψ)` derive `ψ`.                                                                              |
| `contradiction`   | ⊥E                | 1             | From `false` derive arbitrary `φ`.                                                                        |
| `forall_form`     | ∀I                | 1             | From `φ` with `x` not free in open assumptions, derive `forall(x, φ)`.                                    |
| `forall_break`    | ∀E                | 1             | From `forall(x, φ)` derive `φ[x ↦ t]`.                                                                    |
| `exists_form`     | ∃I                | 1             | From `φ[x ↦ t]` derive `exists(x, φ)`.                                                                    |
| `exists_break`    | ∃E                | 2             | From `exists(x, φ)` and a derivation of `ψ` under assumption `φ`, derive `ψ` (discharges the assumption). |
| `eq_subst`        | =Subst            | 2             | From `eq(t, u)` and a formula `φ[t]`, derive `φ[u]`.                                                      |

*Side‑conditions* (variable‑capture, freshness, etc.) are **checked automatically** by the proof checker and never appear explicitly in the file.

---

## 7  Complete Mini‑Example

```
# Goal:  exists(x, and(P(x), Q(x)))
import ./previous_lemmas.proof as prev

assump1: assume :
P(a)

assump2: assume :
forall(x, implies(P(x), Q(x)))

step1:  forall_break assump2 :
implies(P(a), Q(a))

step2:  implies_break step1 assump1 :
Q(a)

step3:  and_form assump1 step2 :
and(P(a), Q(a))

result: exists_form step3 :
exists(x, and(P(x), Q(x)))
```

*No numeric line references are used; every step has a stable name.*\
*When editing the file, renaming or reordering steps never breaks references provided the names remain unique.*

---

## 8  Future Extensions

- Additional connectives (`or`, `iff`, classical `¬`) can be added as function symbols without changing the core grammar.
- Multiple‐file projects may introduce `goal:` / `assumptions:` metadata blocks for tooling convenience—currently not required.
- A library of lemmas can be built as ordinary `.proof` files and brought in via `import … as …`.

---

**End of specification**


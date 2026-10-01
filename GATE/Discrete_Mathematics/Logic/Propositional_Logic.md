# Propositional Logic — GATE Notes

> **Coverage:** Neso Academy Discrete Mathematics playlist positions 2–12 and 15–16, plus the four propositional-logic PYQs inspected in the sprint. This page deliberately stops before the formal logical-equivalences lesson (position 17) and later inference-rule material.

# 1. What propositional logic is

Propositional logic studies statements whose truth can be treated as either **true** or **false**. For GATE, the important skill is not memorizing definitions in isolation; it is translating ordinary sentences into symbolic form and then reasoning about the resulting formula correctly.

A **proposition** is a declarative statement with a definite truth value. For example, “7 is prime” is a proposition because it is true, while “10 is odd” is a proposition because it is false. A question, command, or expression whose truth depends on an unspecified variable is not yet a proposition. For example, “Is it raining?” is a question, and “x > 5” is an open statement until the value or domain of x is fixed.

Simple statements are usually represented by variables such as `p`, `q`, and `r`. New propositions can then be formed by combining them with logical operators. These are called **compound propositions**.

# 2. Core logical operators

## Negation

The negation of `p`, written `¬p`, reverses the truth value of `p`. If `p` is true, `¬p` is false, and vice versa.

In English, words such as **not**, **cannot**, **is not**, and **does not** usually signal negation. Be careful to negate the correct atomic statement rather than an entire sentence accidentally.

## Conjunction — AND

`p ∧ q` is true only when **both** `p` and `q` are true.

In English, conjunction can appear as **and**, but words such as **but** also behave logically like AND. The word “but” adds contrast in natural language, not a different truth condition.

## Disjunction — inclusive OR

`p ∨ q` is true when at least one of the two propositions is true. It is false only when both are false.

Unless the question explicitly makes the alternatives exclusive, logical OR is normally **inclusive**. Therefore, if both `p` and `q` are true, `p ∨ q` is still true.

## Exclusive OR — XOR

Exclusive OR is true when **exactly one** of `p` and `q` is true. It is false when both have the same truth value.

A useful interpretation is:

`p XOR q` means “either p or q, but not both.”

The difference between OR and XOR matters when a GATE question uses phrases such as **either ... or ... but not both**.

# 3. Implication — the most important operator in this section

An implication has the form:

`p → q`

and is read as **if p, then q**.

The implication is false in exactly one case: when `p` is true but `q` is false. In every other case it is true.

This single fact is extremely useful in GATE. When asked whether an implication is a tautology, do not immediately build a full truth table. First ask:

> Can the antecedent be true while the consequent is false?

If the answer is impossible, the implication is a tautology. If even one assignment makes the antecedent true and the consequent false, it is not a tautology.

Another essential identity is:

`p → q ≡ ¬p ∨ q`

This lets an implication be converted into OR form and appears directly in PYQs.

# 4. Translating English implication correctly

Most mistakes in this topic are not algebra mistakes. They are **direction mistakes** while translating English.

For the implication `p → q`, all of the following express the same logical direction:

- If `p`, then `q`
- `p` implies `q`
- `q` if `p`
- `q` when `p`
- `q` whenever `p`
- `q` follows from `p`
- `p` only if `q`

The two phrases that must be kept separate are:

**A if/when B** means:

`B → A`

because B is the condition that is sufficient to produce A.

**A only if B** means:

`A → B`

because B is required whenever A occurs.

This is a high-value GATE trap. The word **only** changes which statement is the necessary condition.

> ⚠️ **Personal error to remember:** In the 2017 Set 2 PYQ, “not pleasant only if raining and cold” was initially reversed. The correct translation is `¬r → (p ∧ q)`, not `(p ∧ q) → ¬r`.

# 5. Necessary and sufficient conditions

For:

`p → q`

`p` is a **sufficient condition** for `q`, because whenever `p` occurs, `q` must follow.

`q` is a **necessary condition** for `p`, because `p` cannot occur without `q`.

This gives a useful language conversion:

“p is sufficient for q” means `p → q`.

“q is necessary for p” also means `p → q`.

Do not reverse the arrow merely because the word “necessary” appears first in the English sentence. Ask which event requires the other.

# 6. Converse, inverse and contrapositive

Starting with the original implication:

`p → q`

the related statements are:

- **Converse:** `q → p`
- **Inverse:** `¬p → ¬q`
- **Contrapositive:** `¬q → ¬p`

The most important equivalence is:

`p → q ≡ ¬q → ¬p`

An implication is always logically equivalent to its **contrapositive**.

The converse is not generally equivalent to the original implication. The inverse is also not generally equivalent to the original. However:

`q → p ≡ ¬p → ¬q`

so the **converse and inverse are equivalent to each other**.

For GATE, this is often faster than constructing a truth table. If an option is simply the contrapositive of the given implication, it is equivalent immediately.

# 7. Biconditional — “if and only if”

The biconditional:

`p ↔ q`

is true when `p` and `q` have the **same truth value**, and false when their truth values differ.

It represents a two-way condition:

`p ↔ q ≡ (p → q) ∧ (q → p)`

So “p if and only if q” means both:

- p is sufficient for q, and
- p is necessary for q.

Equivalently, each statement implies the other.

The phrase **if and only if**, often shortened to **iff**, must not be confused with a one-way implication.

# 8. Operator precedence

When parentheses are omitted, the usual precedence used in propositional-logic expressions is:

`¬` first, then `∧`, then `∨`, then `→`, and finally `↔`.

Therefore:

`¬p ∧ q → r`

is read as:

`(¬p ∧ q) → r`

not as `¬(p ∧ (q → r))`.

In an exam, if an expression is visually complicated, explicitly insert parentheses according to precedence before doing any reasoning. This avoids losing marks to parsing errors rather than logic errors.

# 9. A reliable method for translating English to logic

When a sentence looks complicated, do not translate the entire sentence in one jump.

First identify the smallest meaningful propositions and assign symbols. Then locate negations and the main connective joining the clauses. Finally, translate conditional phrases carefully and parenthesize each clause before combining them.

For example, if:

`p`: it is raining  
`q`: it is cold  
`r`: it is pleasant

then:

“It is not raining and it is pleasant”

becomes:

`¬p ∧ r`

while:

“It is not pleasant only if it is raining and it is cold”

becomes:

`¬r → (p ∧ q)`

Combining the two clauses with AND gives:

`(¬p ∧ r) ∧ (¬r → (p ∧ q))`

The key is to translate **each English connector separately** before combining the formula.

# 10. Tautology, contradiction, contingency and satisfiability

A **tautology** is true for every possible assignment of truth values to its variables. For example, `p ∨ ¬p` is always true.

A **contradiction** is false for every assignment. For example, `p ∧ ¬p` can never be true.

A **contingency** is true for some assignments and false for others. Most ordinary propositions, such as `p → q`, are contingencies.

A formula is **satisfiable** if there is at least one assignment under which it becomes true. Therefore, every tautology is satisfiable, and a contingency is also satisfiable.

A formula is **unsatisfiable** if there is no assignment that makes it true. In propositional logic, an unsatisfiable formula is a contradiction.

For propositional logic, a **valid** formula is one that is true under every valuation, so validity corresponds to being a tautology.

# 11. Truth tables: when to use them

With `n` independent propositional variables, a complete truth table has `2^n` rows.

Truth tables are definitive, but they are not always the fastest GATE method. Prefer direct reasoning when the structure gives an obvious shortcut:

- For an implication, search for the only false pattern: antecedent true and consequent false.
- For a biconditional, compare whether both sides can have different truth values.
- For equivalence, use a known identity or contrapositive when available.
- To prove a formula is **not** a tautology, one counterexample assignment is enough.

Use the full table when the expression is small and no cleaner structural argument is obvious.

# 12. PYQ patterns from this session

## GATE CSE 2017 Set 1 — Question 1

The question asks which statements are equivalent to:

`¬p → ¬q`

Two quick transformations solve it.

Using implication elimination:

`¬p → ¬q ≡ p ∨ ¬q`

and using the contrapositive:

`¬p → ¬q ≡ q → p`

Therefore statements II and III are correct, giving **Option D**.

**What GATE is testing:** implication equivalence and recognition of the contrapositive. This should be solved structurally rather than by constructing a full truth table.

## GATE CSE 2017 Set 2 — Question 11

The important phrase is:

“not pleasant **only if** raining and cold.”

If `r` means pleasant, `p` means raining, and `q` means cold, then:

`¬r → (p ∧ q)`

The other clause is:

`¬p ∧ r`

so the complete representation is:

`(¬p ∧ r) ∧ (¬r → (p ∧ q))`

This is **Option A**.

**What GATE is testing:** whether “only if” is translated in the correct direction. This is the clearest error signal from today and should be actively remembered.

## GATE CSE 2021 Set 1 — Question 7

The question compares:

`S1: (¬p ∧ (p ∨ q)) → q`

and:

`S2: q → (¬p ∧ (p ∨ q))`

For `S1`, an implication could be false only if its left side were true and `q` were false. But if `q` is false, the left side becomes `¬p ∧ p`, which cannot be true. Therefore `S1` is a tautology.

For `S2`, take `p = true` and `q = true`. The antecedent is true, while `¬p ∧ (p ∨ q)` is false. Hence the implication is false for this assignment, so `S2` is not a tautology.

Therefore the correct choice is **Option B: S1 is a tautology, S2 is not**.

**What GATE is testing:** the single false case of implication and the fact that one counterexample is enough to disprove tautology.

## GATE CSE 2024 Set 2 — Question 2

Let:

`p`: fail grade can be given  
`q`: student scores more than 50%

The statement says:

“Fail grade cannot be given **when** the student scores more than 50%.”

“A when B” means `B → A`. Therefore:

`q → ¬p`

which is **Option A**.

**What GATE is testing:** natural-language implication direction. The official question is **2024 Set 2 Question 2**; use this corrected reference when revising.

# 13. What must be instantly recallable

Before moving on from this portion of propositional logic, the following should be automatic rather than reconstructed slowly:

`p → q` is false only for `p = true, q = false`.

`p → q ≡ ¬p ∨ q`.

`p → q ≡ ¬q → ¬p` because an implication is equivalent to its contrapositive.

“A if/when B” means `B → A`.

“A only if B” means `A → B`.

In `p → q`, `p` is sufficient for `q`, while `q` is necessary for `p`.

`p ↔ q` means both `p → q` and `q → p`.

A tautology is true for every valuation; a contradiction is false for every valuation; a contingency is true for some and false for others; satisfiable means true for at least one valuation.

# 14. Scope boundary

These notes intentionally stop at the material studied today. Formal logical-equivalence laws from playlist position 17 onward, the later solved-problem sequence, rules of inference, and first-order logic are **not included yet**. They should be added only after those blocks are actually studied.

# Sources used for this block

Neso Academy Discrete Mathematics playlist:  
https://www.youtube.com/playlist?list=PLBlnK6fEyqRhqJPDXcvYlLfXPh37L89g3

GATE CSE 2017 Set 1 Q1:  
https://gateoverflow.in/118698/gate-cse-2017-set-1-question-01

GATE CSE 2017 Set 2 Q11:  
https://gateoverflow.in/118151/gate-cse-2017-set-2-question-11

GATE CSE 2021 Set 1 Q7:  
https://gateoverflow.in/357445/gate-cse-2021-set-1-question-7

GATE CSE 2024 Set 2 Q2:  
https://gateoverflow.in/422895/gate-cse-2024-set-2-question-2

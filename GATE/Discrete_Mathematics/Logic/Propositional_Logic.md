# Propositional Logic — GATE Notes

[Existing content preserved]

# Part 2 — Logical Equivalence and Rules of Inference (Videos 17–31)

## 15. Logical Equivalence

Two propositions are logically equivalent when they produce the same truth value for every possible assignment of their variables.

Notation:

`p ≡ q`

means that `p` and `q` are equivalent.

For GATE, equivalence is usually tested through:
- applying standard laws,
- converting implications,
- comparing truth conditions,
- identifying equivalent forms quickly.

## 16. Important Equivalence Laws

### Identity Laws

`p ∨ F ≡ p`

`p ∧ T ≡ p`

### Domination Laws

`p ∨ T ≡ T`

`p ∧ F ≡ F`

### Idempotent Laws

`p ∨ p ≡ p`

`p ∧ p ≡ p`

### Complement Laws

`p ∨ ¬p ≡ T`

`p ∧ ¬p ≡ F`

### Double Negation

`¬(¬p) ≡ p`

### Commutative Laws

`p ∨ q ≡ q ∨ p`

`p ∧ q ≡ q ∧ p`

### Associative Laws

`(p ∨ q) ∨ r ≡ p ∨ (q ∨ r)`

`(p ∧ q) ∧ r ≡ p ∧ (q ∧ r)`

### Distributive Laws

`p ∨ (q ∧ r) ≡ (p ∨ q) ∧ (p ∨ r)`

`p ∧ (q ∨ r) ≡ (p ∧ q) ∨ (p ∧ r)`

### Absorption Laws

`p ∨ (p ∧ q) ≡ p`

`p ∧ (p ∨ q) ≡ p`

## 17. De Morgan's Laws

These are among the most important laws for GATE.

`¬(p ∧ q) ≡ ¬p ∨ ¬q`

`¬(p ∨ q) ≡ ¬p ∧ ¬q`

Remember:
- Negation changes AND to OR.
- Negation changes OR to AND.
- Every individual proposition also gets negated.

## 18. Implication Equivalences

The implication can always be removed using:

`p → q ≡ ¬p ∨ q`

Contrapositive:

`p → q ≡ ¬q → ¬p`

This is frequently used in GATE options because the equivalent expression may not look similar initially.

## 19. Biconditional Equivalence

`p ↔ q ≡ (p → q) ∧ (q → p)`

Another useful form:

`p ↔ q ≡ (p ∧ q) ∨ (¬p ∧ ¬q)`

A biconditional is true when both propositions have the same truth value.

## 20. Solving Equivalence Questions in GATE

Do not immediately construct truth tables.

Use this order:

1. Remove implications.
2. Apply De Morgan's laws if negations are present.
3. Simplify using absorption, identity and complement laws.
4. Look for contrapositive or known patterns.

Truth tables are useful only when expressions are small and no simplification is obvious.

## 21. Rules of Inference

An argument consists of:

- premises: statements assumed to be true,
- conclusion: statement derived from premises.

An argument is valid when the conclusion must be true whenever all premises are true.

## 22. Important Inference Rules

### Modus Ponens

`p → q`

`p`

Therefore:

`q`

### Modus Tollens

`p → q`

`¬q`

Therefore:

`¬p`

### Hypothetical Syllogism

`p → q`

`q → r`

Therefore:

`p → r`

### Disjunctive Syllogism

`p ∨ q`

`¬p`

Therefore:

`q`

## 23. Common GATE Traps

- Converse of an implication is not generally equivalent.
- Affirming the consequent is an invalid argument.
- Denying the antecedent is an invalid argument.
- Contrapositive is the only guaranteed equivalent form of an implication.

## 24. Exam Checklist

Before solving propositional logic questions, recall:

- `p → q ≡ ¬p ∨ q`
- `p → q ≡ ¬q → ¬p`
- De Morgan's laws
- `p ↔ q` means both directions
- Equivalence means identical truth values for every valuation
- Valid arguments preserve truth from premises to conclusion

## Scope Boundary

This completes the material studied up to Neso Academy videos 17–31. Further topics should be appended only after they are studied.
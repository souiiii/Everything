# Propositional Logic — GATE Notes

> **Coverage:** Neso Academy Discrete Mathematics playlist positions 2–12 and 15–31, together with the propositional-logic and inference PYQs used in the sprint. Part 1 develops the foundations; Part 2 continues with logical equivalence, logical consequence, rules of inference, and argument validity.

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

<aside>
⚠️

**Personal error to remember:** In the 2017 Set 2 PYQ, “not pleasant only if raining and cold” was initially reversed. The correct translation is `¬r → (p ∧ q)`, not `(p ∧ q) → ¬r`.

</aside>

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

# 14. Part 1 checkpoint

The foundational block ends here. The next section continues the same topic with the formal equivalence laws and the rules used to derive conclusions from premises. First-order logic is intentionally left for the next study block.

# Sources used for Part 1

Neso Academy Discrete Mathematics playlist:  
[https://www.youtube.com/playlist?list=PLBlnK6fEyqRhqJPDXcvYlLfXPh37L89g3](https://www.youtube.com/playlist?list=PLBlnK6fEyqRhqJPDXcvYlLfXPh37L89g3)

GATE CSE 2017 Set 1 Q1:  
[https://gateoverflow.in/118698/gate-cse-2017-set-1-question-01](https://gateoverflow.in/118698/gate-cse-2017-set-1-question-01)

GATE CSE 2017 Set 2 Q11:  
[https://gateoverflow.in/118151/gate-cse-2017-set-2-question-11](https://gateoverflow.in/118151/gate-cse-2017-set-2-question-11)

GATE CSE 2021 Set 1 Q7:  
[https://gateoverflow.in/357445/gate-cse-2021-set-1-question-7](https://gateoverflow.in/357445/gate-cse-2021-set-1-question-7)

GATE CSE 2024 Set 2 Q2:  
[https://gateoverflow.in/422895/gate-cse-2024-set-2-question-2](https://gateoverflow.in/422895/gate-cse-2024-set-2-question-2)

---

# Part 2 — Logical Equivalence and Rules of Inference (Videos 17–31)

> **Coverage:** Neso Academy positions 17–20 and 24–31, with the relevant GATE PYQs tied directly to the concepts. The aim here is not to memorize a table of laws mechanically. It is to learn when two expressions mean exactly the same thing, how to transform one into another safely, and how premises can justify a conclusion.

# 15. What logical equivalence actually means

Two propositions are **logically equivalent** when they have the same truth value under **every possible assignment** of their propositional variables. If `P` and `Q` are logically equivalent, we write:

`P ≡ Q`

This is stronger than saying that the two expressions happen to be true for one particular case. Equivalence means that no possible valuation can make one expression true and the other false.

There are two equivalent ways to think about this:

- `P` and `Q` have identical truth-table columns.
- `P ↔ Q` is a tautology.

That second view is especially useful in GATE. To test whether two expressions are equivalent, you are really asking whether they are guaranteed to agree in every possible world.

A crucial consequence follows: **an equivalence law is a safe replacement rule**. If `P ≡ Q`, then wherever `P` appears inside a larger logical expression, it can be replaced by `Q` without changing the meaning of the whole expression.

## Equivalence is not the same as implication

Do not confuse:

`P → Q`

with:

`P ≡ Q`

`P → Q` only says that whenever `P` is true, `Q` must also be true. It does **not** require `Q` to imply `P`. Logical equivalence requires agreement in both directions.

For example:

`p ∧ q → p`

is valid, because whenever both `p` and `q` are true, `p` is certainly true. But:

`p ∧ q ≡ p`

is false, because `p` can be true while `q` is false. The implication is one-way; equivalence is two-way.

This distinction becomes important later: **equivalence laws rewrite expressions, while inference rules derive conclusions from premises**.

# 16. The logical laws worth knowing for GATE

The laws below are not independent facts to memorize as isolated equations. Most of them express simple ideas such as “adding TRUE with AND changes nothing” or “a statement and its negation cannot both be true.” Learn the meaning behind each family, because that makes reconstruction much easier during the exam.

## Identity laws

`p ∧ T ≡ p`

`p ∨ F ≡ p`

TRUE is neutral for AND, while FALSE is neutral for OR. Requiring `p` and something that is always true adds no new restriction; allowing `p` or something that is always false adds no new possibility.

## Domination laws

`p ∨ T ≡ T`

`p ∧ F ≡ F`

With OR, a guaranteed TRUE makes the whole expression true. With AND, a guaranteed FALSE makes the whole expression false.

## Idempotent laws

`p ∨ p ≡ p`

`p ∧ p ≡ p`

Repeating the same condition does not strengthen or weaken it. “p and p” is still just p, and “p or p” is still just p.

## Double-negation law

`¬¬p ≡ p`

Negating a statement twice returns to the original statement.

## Complement laws

`p ∨ ¬p ≡ T`

`p ∧ ¬p ≡ F`

A proposition must either be true or false, so `p ∨ ¬p` is always true. It cannot be both true and false simultaneously, so `p ∧ ¬p` is always false.

These two patterns appear constantly while simplifying expressions. Recognizing them quickly often collapses a large formula in one step.

## Commutative laws

`p ∨ q ≡ q ∨ p`

`p ∧ q ≡ q ∧ p`

The order of operands does not matter for AND or OR.

Do **not** extend this habit to implication. In general:

`p → q ≢ q → p`

The arrow has a direction.

## Associative laws

`(p ∨ q) ∨ r ≡ p ∨ (q ∨ r)`

`(p ∧ q) ∧ r ≡ p ∧ (q ∧ r)`

When the same operator is repeated, the grouping can be changed without changing the result.

## Distributive laws

`p ∧ (q ∨ r) ≡ (p ∧ q) ∨ (p ∧ r)`

`p ∨ (q ∧ r) ≡ (p ∨ q) ∧ (p ∨ r)`

Both AND and OR distribute over the other in propositional logic. The second form sometimes feels unusual because ordinary algebra does not behave exactly the same way, so it is worth recognizing explicitly.

## De Morgan’s laws

`¬(p ∧ q) ≡ ¬p ∨ ¬q`

`¬(p ∨ q) ≡ ¬p ∧ ¬q`

When a negation crosses a bracket, **every component is negated and the connective flips**:

`∧ ↔ ∨`

A common mistake is to negate the terms but forget to change AND to OR or OR to AND.

## Absorption laws

`p ∨ (p ∧ q) ≡ p`

`p ∧ (p ∨ q) ≡ p`

The second part contributes nothing new because `p` is already enough to determine the expression. These are particularly useful when an expression looks larger than it really is.

# 17. High-value conditional and biconditional transformations

The general laws above are useful, but GATE propositional-logic questions repeatedly become much easier after eliminating implication and biconditional.

## Eliminate implication first

The most important identity is:

`p → q ≡ ¬p ∨ q`

This follows directly from the truth condition of implication. The only forbidden case for `p → q` is `p = T` and `q = F`; `¬p ∨ q` is false in exactly that same case.

Whenever an expression contains several arrows and you are unsure what to do, converting them to `¬`, `∧`, and `∨` often reveals the structure immediately.

A second high-value identity is the negation of implication:

`¬(p → q) ≡ p ∧ ¬q`

This is worth understanding rather than memorizing. An implication is false only when its antecedent is true and its consequent is false. Therefore saying “the implication is not true” is exactly the same as asserting that failure pattern.

## Contrapositive

`p → q ≡ ¬q → ¬p`

The contrapositive is not merely another implication that happens to be useful; it is **logically equivalent** to the original implication. Therefore either form can replace the other safely.

By contrast, the converse `q → p` is not generally equivalent to `p → q`.

## Biconditional

`p ↔ q ≡ (p → q) ∧ (q → p)`

A biconditional requires both directions to hold.

It can also be written as:

`p ↔ q ≡ (p ∧ q) ∨ (¬p ∧ ¬q)`

This form exposes its meaning very clearly: a biconditional is true when the two propositions have the **same** truth value.

Its negation therefore means the propositions differ:

`¬(p ↔ q) ≡ (p ∧ ¬q) ∨ (¬p ∧ q)`

which is exactly XOR.

# 18. How to simplify equivalence questions efficiently

A full truth table always works for a small propositional expression, but it is often slower than necessary. GATE usually rewards recognizing structure.

Use this order:

1. **Parse the expression correctly.** Insert parentheses mentally if precedence is unclear.
2. **Remove `→` and `↔`** when they hide the structure.
3. **Push negations inward** using De Morgan and double negation.
4. **Look for complements** such as `q ∨ ¬q = T` or `q ∧ ¬q = F`.
5. **Factor or distribute only when it simplifies the expression.**
6. **Use absorption** when the same proposition appears both alone and inside a larger term.
7. If you only need to prove two expressions are **not** equivalent, stop as soon as you find one valuation on which they differ.

For example:

`(p ∧ q) ∨ (p ∧ ¬q)`

Factor `p`:

`p ∧ (q ∨ ¬q)`

Since `q ∨ ¬q ≡ T`:

`p ∧ T ≡ p`

The expression looks like it depends on both `p` and `q`, but the complement pair removes `q` completely.

<aside>
💡

**GATE habit:** before drawing a truth table, ask whether one implication conversion, one De Morgan step, or one complement pair collapses the expression. A truth table is a verification tool; it does not have to be your first move.

</aside>

# 19. What an argument is

Logical equivalence asks whether two expressions mean the same thing. **Inference** asks a different question: if certain statements are accepted as premises, what conclusions are logically forced by them?

An argument has two parts:

- **Premises** — statements assumed to hold for the purpose of the argument.
- **Conclusion** — the statement claimed to follow from those premises.

An argument is **valid** when there is no possible valuation in which **all premises are true and the conclusion is false**.

This definition is extremely important. Validity is about the structure of the reasoning, not about whether the premises happen to describe the real world correctly.

For example:

1. If a number is divisible by 4, then it is even.  
2. 12 is divisible by 4.  
3. Therefore, 12 is even.

The structure is valid.

But even an argument with a factually false premise can still have a valid logical form. GATE generally asks whether the conclusion **follows**, not whether the story itself is realistic.

## Validity as a single implication

If the premises are `P1, P2, ..., Pn` and the conclusion is `C`, then the argument is valid exactly when:

`(P1 ∧ P2 ∧ ... ∧ Pn) → C`

is a tautology.

This connects inference directly back to propositional logic. A valid argument says: **whenever all premises hold together, the conclusion cannot fail**.

# 20. Rules of inference

Rules of inference are standard valid argument forms. Once the premises match the form of a rule, the conclusion can be derived without rebuilding a truth table from scratch.

## Modus Ponens

Given:

`p → q`

`p`

we may conclude:

`q`

The reasoning is direct: the rule says that `p` guarantees `q`, and the second premise tells us `p` has occurred.

A common mistake is trying to run this rule backward. From `p → q` and `q`, you cannot in general conclude `p`; `q` might have happened for some other reason.

## Modus Tollens

Given:

`p → q`

`¬q`

we may conclude:

`¬p`

If `p` were true, `q` would have to be true. Since `q` is false, `p` cannot be true. This is essentially the contrapositive used as an inference rule.

## Hypothetical Syllogism

Given:

`p → q`

`q → r`

we may conclude:

`p → r`

The two implications form a chain. If `p` is enough for `q`, and `q` is enough for `r`, then `p` is enough for `r`.

## Disjunctive Syllogism

Given:

`p ∨ q`

`¬p`

we may conclude:

`q`

At least one of the alternatives must hold. Once `p` is ruled out, `q` remains.

Because ordinary logical OR is inclusive, the premise `p ∨ q` does not originally claim that exactly one is true. The second premise is what eliminates one branch.

## Addition

From:

`p`

we may infer:

`p ∨ q`

for any proposition `q`.

If `p` is already true, then an OR expression containing `p` must also be true. This rule is simple but useful when constructing a target conclusion.

## Simplification

From:

`p ∧ q`

we may infer either:

`p`

or:

`q`

A conjunction asserts both parts, so either component may be extracted.

## Conjunction

From:

`p`

and:

`q`

we may infer:

`p ∧ q`

If both statements have been established, they may be combined into one conjunction.

## Resolution

Given:

`p ∨ q`

`¬p ∨ r`

we may conclude:

`q ∨ r`

The complementary pair `p` and `¬p` is eliminated. Intuitively, if the first clause is satisfied through `p`, the second clause must be satisfied through `r`; if the first is not satisfied through `p`, it must be satisfied through `q`. Either way, at least one of `q` or `r` must be true.

# 21. Equivalence laws and inference rules are different tools

This distinction is worth making explicit because the notation can make the two ideas look similar.

With an **equivalence**:

`P ≡ Q`

`P` and `Q` describe the same truth condition. You can replace one with the other in either direction.

With an **inference**:

`P1, P2, ... ⟹ C`

the conclusion `C` is guaranteed when the premises hold, but `C` need not contain the same information as the premises.

For example, from:

`p ∧ q`

we can infer:

`p`

by simplification. But `p ∧ q` is **not** logically equivalent to `p`, because `p` alone does not guarantee `q`.

That is the cleanest way to separate the two topics:

**Equivalence preserves the whole meaning. Inference preserves truth from premises to conclusion.**

# 22. Checking validity quickly

There are three useful methods. Choose the cheapest one for the expression in front of you.

## Method 1 — Recognize inference rules

If the argument is a short chain of Modus Ponens, Modus Tollens, Disjunctive Syllogism, or another standard rule, simply derive the conclusion step by step.

## Method 2 — Assume premises true and conclusion false

Because an invalid argument requires all premises to be true while the conclusion is false, deliberately try to create that situation.

If those requirements force a contradiction, then no counterexample is possible and the argument is valid.

This is often much faster than writing every row of a truth table.

## Method 3 — Find one counterexample

To prove an argument invalid, you do **not** need a complete truth table. One valuation with:

- every premise true, and
- the conclusion false

is enough.

Similarly, one valuation on which two expressions differ is enough to show that they are not logically equivalent.

This “one counterexample is enough” principle is one of the most useful time-saving ideas in logic questions.

# 23. Two invalid patterns that repeatedly cause mistakes

Even before studying formal fallacies later, two tempting but invalid moves should already be avoided.

## Affirming the consequent

From:

`p → q`

`q`

concluding:

`p`

is invalid.

The implication tells us one way for `q` to follow, not the only way. `q` may be true even when `p` is false.

## Denying the antecedent

From:

`p → q`

`¬p`

concluding:

`¬q`

is also invalid.

The failure of a sufficient condition does not imply the failure of the result. Again, `q` may be true for some independent reason.

Compare both with the two valid forms:

- `p → q`, `p` ⟹ `q` — Modus Ponens.
- `p → q`, `¬q` ⟹ `¬p` — Modus Tollens.

# 24. How the PYQs connect to Part 2

The purpose of these references is not to memorize question numbers. They show the exact kinds of transformations that GATE expects you to perform quickly.

## GATE CSE 2017 Set 1 — Question 1: implication equivalence

The expression:

`¬p → ¬q`

can be transformed by implication elimination:

`¬p → ¬q ≡ p ∨ ¬q`

and by taking the contrapositive:

`¬p → ¬q ≡ q → p`

This is a direct application of Part 2: the question becomes easy once you know that implication elimination and contraposition are **equivalence-preserving transformations**.

**Exam lesson:** when several options are proposed as equivalents, normalize the implication before considering a truth table.

## GATE CSE 2021 Set 1 — Question 7: proving tautology vs finding a counterexample

For:

`(¬p ∧ (p ∨ q)) → q`

the implication could fail only if its antecedent were true and `q` were false. Setting `q = F` reduces the antecedent to `¬p ∧ p`, a contradiction. Therefore the implication can never fail and is a tautology.

For the reverse-looking implication in the same question, one counterexample is enough to show that it is not a tautology.

**Exam lesson:** do not automatically treat a reversed implication as equivalent. Use the single false pattern of implication or find one counterexample.

## GATE CSE 2024 Set 2 — Question 2: translation plus contrapositive

Once the English sentence is translated as:

`q → ¬p`

the contrapositive is immediately available:

`p → ¬q`

Both express the same logical restriction.

**Exam lesson:** translation determines the arrow direction first; equivalence rules can only help after the sentence has been represented correctly.

## GATE 2026 CS1 General Aptitude — Question 5: converse is not guaranteed

The question gives a conditional rule and asks which statement need not follow. The important distinction is exactly the one developed here: the **contrapositive** of an implication is guaranteed, while the **converse** is not.

If the rule is abstractly:

`p → q`

then:

`¬q → ¬p`

is equivalent to the original statement, but:

`q → p`

is an unjustified reversal unless additional information is given.

**Exam lesson:** when an option reverses an implication, label it mentally as “converse” before accepting it. GATE frequently hides a direction error inside natural language.

# 25. A compact solving strategy for GATE

When an equivalence or inference question appears, use the following decision process rather than reaching immediately for a truth table.

**If the question asks whether two expressions are equivalent:** eliminate implication/biconditional, use De Morgan or algebraic laws, and look for complements or absorption. If you suspect they are not equivalent, try to construct one valuation where they differ.

**If the question asks whether an argument is valid:** identify the premises and conclusion, try standard inference rules, or attempt to make all premises true while the conclusion is false. If such a valuation exists, the argument is invalid; if the attempt necessarily produces contradiction, the argument is valid.

**If the question is written in English:** finish the translation first. In particular, resolve `if`, `only if`, `when`, necessary/sufficient language, and the direction of implication before doing any symbolic manipulation.

**If the expression is small but messy:** a truth table remains a perfectly valid final fallback. The goal is not to avoid truth tables at all costs; the goal is to avoid spending time on `2^n` rows when the structure already gives the answer.

# 26. What must be instantly recallable from Part 2

The following should eventually become automatic:

- Logical equivalence means identical truth values under every valuation; equivalently, `P ↔ Q` is a tautology.
- `p → q ≡ ¬p ∨ q`.
- `¬(p → q) ≡ p ∧ ¬q`.
- `p → q ≡ ¬q → ¬p`; the converse is not generally equivalent.
- `p ↔ q ≡ (p → q) ∧ (q → p)` and is true when both sides have the same truth value.
- De Morgan negates each component **and flips** AND/OR.
- `p ∨ ¬p ≡ T` and `p ∧ ¬p ≡ F` are high-value simplification patterns.
- Modus Ponens: `p → q`, `p` ⟹ `q`.
- Modus Tollens: `p → q`, `¬q` ⟹ `¬p`.
- Hypothetical Syllogism chains implications; Disjunctive Syllogism eliminates a false alternative; Resolution eliminates complementary literals across clauses.
- An argument is valid only if there is **no** valuation with all premises true and the conclusion false.
- One counterexample is enough to disprove a tautology, equivalence, or argument validity claim.
- Equivalence is a two-way replacement of meaning; inference is a truth-preserving move from premises to a conclusion.

# 27. Scope boundary

This completes the studied **propositional-logic** material through playlist position 31. The next block begins first-order logic: predicates, quantifiers, restricted domains, quantified negation, nested quantifiers, and inference involving quantified statements. Those ideas should be kept separate until they are actually studied, because propositional logic treats whole statements as atomic units while first-order logic can reason about objects inside those statements.

# Sources used for Part 2

Neso Academy — Discrete Mathematics playlist (positions 17–20 and 24–31):  
[https://www.youtube.com/playlist?list=PLBlnK6fEyqRhqJPDXcvYlLfXPh37L89g3](https://www.youtube.com/playlist?list=PLBlnK6fEyqRhqJPDXcvYlLfXPh37L89g3)

GATE CSE 2017 Set 1 Q1:  
[https://gateoverflow.in/118698/gate-cse-2017-set-1-question-01](https://gateoverflow.in/118698/gate-cse-2017-set-1-question-01)

GATE CSE 2021 Set 1 Q7:  
[https://gateoverflow.in/357445/gate-cse-2021-set-1-question-7](https://gateoverflow.in/357445/gate-cse-2021-set-1-question-7)

GATE CSE 2024 Set 2 Q2:  
[https://gateoverflow.in/422895/gate-cse-2024-set-2-question-2](https://gateoverflow.in/422895/gate-cse-2024-set-2-question-2)

GATE 2026 CS1 question paper — General Aptitude Q5:  
[https://gate2026.iitg.ac.in/doc/download/2026/QPs/CS1.pdf](https://gate2026.iitg.ac.in/doc/download/2026/QPs/CS1.pdf)

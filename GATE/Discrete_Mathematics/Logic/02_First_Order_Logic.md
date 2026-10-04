# First-Order Logic — GATE Notes

> **Coverage:** Neso Academy Discrete Mathematics playlist positions 32–47: introduction to first-order logic, predicates, truth values of predicates, universal and existential quantifiers, counterexamples, restricted domains, logical equivalences involving quantifiers, negating quantified expressions, English-to-logic translation, and the first solved-problem block. Formal nested-quantifier techniques from position 52 onward are intentionally not developed yet.
>
> This continues the propositional-logic material in [`01_Propositional_Logic.md`](./01_Propositional_Logic.md).

# 28. Why propositional logic is not enough

Propositional logic treats an entire statement as one indivisible unit. If `p` means “Alice is a student” and `q` means “Alice studies mathematics,” propositional logic can combine `p` and `q`, negate them, or place them inside implications. What it cannot naturally express is the internal structure shared by statements such as “Alice is a student,” “Bob is a student,” and “Every student studies mathematics.”

First-order logic solves this limitation by allowing us to talk about **objects**, **properties of objects**, and **relationships between objects**. Instead of creating a different propositional variable for every possible person, we can define a predicate such as `Student(x)` and let `x` range over a domain of possible objects.

This is the key conceptual jump:

**Propositional logic reasons about whole statements. First-order logic can look inside statements and reason about the objects mentioned by them.**

For GATE, this matters because many questions are really about translating ordinary language into a precise statement about “all,” “some,” “none,” or “exactly one.” The symbols themselves are not the difficult part. The difficult part is understanding exactly what the English sentence commits us to.

# 29. Predicates, variables, and the domain of discourse

A **predicate** is a statement whose truth depends on one or more variables. For example:

`P(x): x is even`

is not yet a complete proposition because its truth depends on the value chosen for `x`. If `x = 8`, then `P(x)` is true. If `x = 9`, then it is false.

A predicate can involve more than one object. For example:

`R(x, y): x divides y`

has two variables. `R(3, 12)` is true, while `R(5, 12)` is false.

The set from which the variables are allowed to take values is called the **domain**, or **domain of discourse**. The domain is part of the meaning of the statement. The predicate `x² ≥ x`, for example, behaves differently over natural numbers and over all real numbers. A quantified statement therefore cannot be interpreted correctly unless the domain is known or reasonably implied by the question.

A useful GATE habit is to identify the domain before manipulating the formula. A formula that looks obviously true over natural numbers may fail over integers or real numbers.

## Open statements and sentences

A predicate such as `P(x)` with an unassigned variable is an **open statement**. It does not yet have one fixed truth value.

Once the variable is either given a value or bound by a quantifier, the statement can become a complete logical sentence. For example, `P(4)` has a definite truth value, and `∀x P(x)` also has a definite truth value once the domain and meaning of `P` are fixed.

This distinction is useful because a free variable behaves like an unanswered question, whereas a quantified variable has been given a logical role.

# 30. Quantifiers tell us how many objects must satisfy a predicate

The two basic quantifiers are:

`∀` — the **universal quantifier**, read as “for every” or “for all.”

`∃` — the **existential quantifier**, read as “there exists” or “for at least one.”

Quantifiers do not merely decorate a predicate. They completely change the amount of evidence required for a statement to be true or false.

If the domain were the finite set `{a, b, c}`, then `∀x P(x)` behaves like `P(a) ∧ P(b) ∧ P(c)`, because every object must satisfy the predicate.

In contrast, `∃x P(x)` behaves like `P(a) ∨ P(b) ∨ P(c)`, because one successful object is enough.

This “universal behaves like AND, existential behaves like OR” mental model is extremely useful for GATE. It explains many equivalences and counterexamples without forcing you to memorize them blindly.

# 31. Universal quantification — `∀`

The statement `∀x P(x)` means that **every object in the domain satisfies `P`**.

To prove a universal statement, the reasoning must work for an arbitrary object from the domain. Checking a few convenient examples is not enough. If the domain contains a thousand objects, proving the statement for 999 of them still does not prove the universal claim.

To disprove a universal statement, however, only **one counterexample** is required. If `∀x P(x)` claims that every element satisfies `P`, then finding one object `a` for which `P(a)` is false destroys the entire universal statement.

This asymmetry is worth remembering:

**A universal statement needs every case to succeed, but one failure is enough to refute it.**

That is why GATE questions involving `∀` often become easier when you actively search for one counterexample instead of trying to reason about every possible value simultaneously.

# 32. Existential quantification — `∃`

The statement `∃x P(x)` means that **at least one object in the domain satisfies `P`**.

To prove an existential statement, it is enough to produce one suitable object. Such an object is often called a **witness**. If the claim is “there exists an integer whose square is 25,” then either `5` or `-5` is a witness.

To disprove an existential statement, you must show that no object can work. In other words, every possible candidate must fail.

So the existential quantifier has the opposite evidence pattern from the universal quantifier:

**One successful witness proves an existential statement, but disproving it requires ruling out every possible witness.**

This also explains why an existential statement does not imply uniqueness. `∃x P(x)` says “at least one,” not “exactly one.” If a question says “there exists exactly one,” an additional uniqueness condition must be expressed.

# 33. Restricted domains — the implication/conjunction pattern

One of the most important translation patterns in first-order logic appears when the stated domain is broader than the group mentioned in the English sentence.

Suppose the domain is **all people**, and define:

`Student(x): x is a student`

`Passed(x): x passed the exam`

The English statement “Every student passed the exam” should be written as:

`∀x(Student(x) → Passed(x))`

The implication is essential. We are not claiming that every person is a student. We are saying that **whenever a person belongs to the student group, that person must also belong to the passed group**.

A non-student does not violate the sentence because the antecedent `Student(x)` is false, so the implication imposes no requirement on that person.

By contrast, the statement “Some student passed the exam” should be written as:

`∃x(Student(x) ∧ Passed(x))`

Here we need one actual object that satisfies **both** conditions. Using implication would be wrong. If we wrote `∃x(Student(x) → Passed(x))`, then any non-student would make the implication true automatically, which would not prove that a student passed anything.

This gives one of the highest-value translation rules for GATE:

**Universal restriction usually uses implication. Existential restriction usually uses conjunction.**

The same pattern gives:

“No student passed the exam”

`∀x(Student(x) → ¬Passed(x))`

which is equivalently:

`¬∃x(Student(x) ∧ Passed(x))`

And “Some student did not pass” becomes:

`∃x(Student(x) ∧ ¬Passed(x))`

The difference between implication and conjunction in these translations is a frequent source of wrong options.

# 34. Equivalences involving quantifiers

The familiar propositional laws still operate inside quantified expressions, but quantifiers also have their own important structural behavior.

Because a universal quantifier behaves like a large conjunction:

`∀x(P(x) ∧ Q(x)) ≡ (∀x P(x)) ∧ (∀x Q(x))`

The left side says every object satisfies both properties. That is exactly the same as saying every object satisfies `P` and every object satisfies `Q`.

Similarly, because an existential quantifier behaves like a large disjunction:

`∃x(P(x) ∨ Q(x)) ≡ (∃x P(x)) ∨ (∃x Q(x))`

If there is some object for which at least one of `P` or `Q` holds, then there exists a witness for `P` or there exists a witness for `Q`.

However, the superficially similar forms in the opposite direction are **not generally equivalent**.

For example, `∀x(P(x) ∨ Q(x))` need not imply `(∀x P(x)) ∨ (∀x Q(x))`. Different objects may satisfy different sides of the OR. One object may satisfy `P`, another may satisfy `Q`, and therefore every object may satisfy `P ∨ Q` even though neither predicate is universally true.

Likewise, `(∃x P(x)) ∧ (∃x Q(x))` does not generally imply `∃x(P(x) ∧ Q(x))`, because the witness for `P` and the witness for `Q` may be different objects.

This is a central first-order-logic idea: **when two existential claims appear separately, never silently assume that the same object satisfies both.**

# 35. Negating quantified statements

The two most important quantified-negation laws are:

`¬∀x P(x) ≡ ∃x ¬P(x)`

and:

`¬∃x P(x) ≡ ∀x ¬P(x)`

These are the first-order versions of De Morgan’s laws.

The first law says that “it is not true that everyone has property `P`” means “there is at least one object that does not have property `P`.”

So “Not every student passed” is not the same as “No student passed.” It means “Some student did not pass.” Symbolically:

`¬∀x(Student(x) → Passed(x)) ≡ ∃x(Student(x) ∧ ¬Passed(x))`

The second law says that “there does not exist anyone with property `P`” means that every object lacks `P`.

## A mechanical way to push negation through quantifiers

When a negation crosses a quantifier:

- `∀` changes to `∃`
- `∃` changes to `∀`
- the predicate inside is negated

Thus `¬∀x P(x)` becomes `∃x ¬P(x)`, while `¬∃x P(x)` becomes `∀x ¬P(x)`.

If more logical structure exists inside the predicate, continue pushing the negation inward using the ordinary De Morgan and implication-negation laws from propositional logic.

This connection is important: first-order logic does not replace propositional logic. It **builds on it**. Once the quantifier has been handled, the expression inside still obeys the same rules you already studied.

# 36. Translating English into first-order logic

A reliable translation should be built in layers rather than guessed in one step.

First identify the **domain**. Then define the predicates precisely. Next identify whether the sentence is universal, existential, negative, or a uniqueness statement. Finally, translate the logical relationship between the predicates.

Consider the sentence “Every programmer knows some programming language.” If the domain contains both people and languages, define:

`Programmer(x): x is a programmer`

`Language(y): y is a programming language`

`Knows(x, y): x knows y`

A natural translation is:

`∀x(Programmer(x) → ∃y(Language(y) ∧ Knows(x, y)))`

The outer implication restricts the universal claim to programmers. The existential portion then demands an actual language that the programmer knows.

Now compare “Some programmer knows every programming language.” The structure changes. We first need a particular programmer, and then that programmer must satisfy a universal condition over languages:

`∃x(Programmer(x) ∧ ∀y(Language(y) → Knows(x, y)))`

Even before the formal nested-quantifier block, this example shows why reading quantifiers from left to right matters. Different English sentences place different obligations on the objects involved.

# 37. “Exactly one” means existence plus uniqueness

The existential quantifier by itself says only “at least one.” To represent **exactly one**, two things must be stated:

1. A suitable object exists.
2. Any object satisfying the same property must be that very object.

A common pattern is:

`∃x(P(x) ∧ ∀y(P(y) → y = x))`

The first part, `P(x)`, gives existence. The second part says that if any `y` also satisfies `P`, then `y` must equal the chosen `x`. Therefore no second distinct witness is allowed.

This pattern is worth understanding because GATE often hides uniqueness inside natural-language statements such as “exactly one parent,” “unique key,” or “there is one and only one object.”

Another equivalent style is to say that there exists a witness and that **there does not exist a different witness**. These two styles are logically equivalent, and recognizing that equivalence is exactly what one of the 2025 GATE questions tests.

# 38. Quantifier order — a preview that already matters in PYQs

Formal nested quantifiers are covered in the later playlist block, but one distinction is already too important to ignore:

`∀x ∃y R(x, y)`

and:

`∃y ∀x R(x, y)`

are generally **not equivalent**.

`∀x ∃y R(x, y)` means that every `x` has at least one suitable `y`. The suitable `y` may be different for different values of `x`.

`∃y ∀x R(x, y)` is stronger. It says that there is one particular `y` that works for every `x`.

A simple analogy is that “Every student has a favorite book” does not imply “There is one book that is the favorite book of every student.” The first statement allows each student to choose a different book; the second requires one common book.

For GATE, never swap `∀` and `∃` simply because the same predicates remain inside. Quantifier order can completely change the meaning.

# 39. GATE CSE 2017 Set 1 — Question 2

The given sentence is:

`F: ∀x(∃y R(x, y))`

with a non-empty domain. We must determine which candidate statements are implied by `F`. The correct answer is **I and IV only (Option B)**.

## Statement I — `∃y∃x R(x, y)`

This is implied. Since the domain is non-empty, choose any object `x`. The given formula says that for this `x`, there must exist at least one `y` such that `R(x, y)` is true. Therefore at least one related pair `(x, y)` exists somewhere in the domain.

So `∀x∃y R(x, y) ⟹ ∃x∃y R(x, y)`, and two existential quantifiers may be reordered here without changing the meaning.

## Statement II — `∃y∀x R(x, y)`

This is **not** implied. The original statement allows a different `y` for each `x`. Suppose the domain contains `1` and `2`, with only `R(1,1)` and `R(2,2)` true. Then every `x` has some suitable `y`, so `F` is true. But there is no single `y` related to every `x`.

## Statement III — `∀y∃x R(x, y)`

This is also **not** implied. The original statement says every `x` reaches at least one `y`; it says nothing about whether every possible `y` must be reached by some `x`. There may be unused `y` values for which no relation holds.

## Statement IV — `¬∃x(∀y ¬R(x, y))`

This is actually equivalent to the given formula:

`¬∃x(∀y ¬R(x, y))`

`≡ ∀x ¬(∀y ¬R(x, y))`

`≡ ∀x ∃y ¬¬R(x, y)`

`≡ ∀x ∃y R(x, y)`

Therefore the correct answer is **I and IV only**.

**Exam lesson:** quantified negation is high-value, and `∀x∃y` does not allow the quantifiers to be swapped casually.

# 40. GATE CSE 2023 — implication strength with quantifiers

Geetha’s conjecture is:

`∀x(P(x) → ∃y Q(x, y))`

The correct options are **II and III**.

## Option I — `∃x(P(x) ∧ ∀y Q(x, y))`

This tells us about only one particular `x`. The conjecture must work for **every** `x` satisfying `P`, so one successful object is not enough.

## Option II — `∀x∀y Q(x, y)`

This is much stronger than the conjecture. If `Q(x, y)` is true for every possible pair, then whenever `P(x)` is true, any `y` can serve as the required existential witness. Therefore the conjecture must hold.

## Option III — `∃y∀x(P(x) → Q(x, y))`

This gives one particular `y` that works for every relevant `x`. The conjecture asks for less: each `P`-object merely needs **some** suitable `y`, and that `y` is allowed to depend on `x`. If one common `y` works for all of them, the weaker conjecture certainly follows.

## Option IV — `∃x(P(x) ∧ ∃y Q(x, y))`

Again, this guarantees success for only one `x`. It cannot establish a universal claim about every `P`-object.

Therefore the correct options are **II and III**.

**Exam lesson:** compare the strength of quantified statements. A stronger universal condition can imply a weaker one, but a statement about one witness cannot usually establish a claim about every relevant object.

# 41. GATE CSE 2025 Set 1 — “Everyone has exactly one mother”

The predicate `mother(y, x)` means that `y` is the mother of `x`, and `noteq(z, y)` means that `z` and `y` are different. The correct options are **B and D**.

A clean target translation is:

`∀x ∃y [mother(y, x) ∧ ∀z(noteq(z, y) → ¬mother(z, x))]`

Read it slowly. For every person `x`, there exists a person `y` who is a mother of `x`. In addition, every person `z` different from `y` is forbidden from also being a mother of `x`.

The first part gives **existence**. The second gives **uniqueness**. This is Option B.

Option D writes the same uniqueness condition differently:

`∀x ∃y [mother(y, x) ∧ ¬∃z(noteq(z, y) ∧ mother(z, x))]`

The expression after the AND says that there does not exist a different `z` who is also a mother of `x`. By quantified negation and implication equivalence, this is the same uniqueness condition as Option B.

## Why Option A is not enough

Option A guarantees a mother `y` and also some `z` who is not a mother. That does **not** prevent a third person from also being a mother. Existence has been shown, but uniqueness has not.

## Why Option C is not enough

Option C does not force a mother to exist. If `mother(y, x)` is false, the implication is vacuously true. It also does not rule out multiple mothers.

Therefore **B and D** are the only correct representations.

**Exam lesson:** “exactly one” must contain both **existence and uniqueness**. If either part is missing, the formula is incomplete.

# 42. A compact translation dictionary for GATE

“Every A is B” means:

`∀x(A(x) → B(x))`

“Some A is B” means:

`∃x(A(x) ∧ B(x))`

“No A is B” means:

`∀x(A(x) → ¬B(x))`

or equivalently:

`¬∃x(A(x) ∧ B(x))`

“Some A is not B” means:

`∃x(A(x) ∧ ¬B(x))`

“Not every A is B” means:

`∃x(A(x) ∧ ¬B(x))`

“There is exactly one A” can be written as:

`∃x(A(x) ∧ ∀y(A(y) → y = x))`

When an English sentence looks complicated, translate the outer words first — **every**, **some**, **none**, **not every**, **exactly one** — and then translate the internal relationship. This is far safer than trying to write the whole formula at once.

# 43. How to attack first-order-logic questions in GATE

When a first-order-logic question appears, begin by reading the formula as English. Before doing algebra, ask what each quantifier is demanding and whether the same witness has to work for several objects.

For an implication claim such as `F → G`, use the same principle from propositional logic: to disprove it, find one interpretation where `F` is true and `G` is false. Small artificial domains with two or three objects are often enough to construct such a counterexample.

For equivalence questions, first push negations through quantifiers and then use ordinary propositional equivalences inside the predicate. Quantified De Morgan rules and implication elimination are especially useful.

For translation questions, separate **existence**, **universality**, and **uniqueness**. Many wrong options deliberately express only part of the English sentence.

For `∀` statements, actively search for a counterexample. For `∃` statements, actively search for a witness. Thinking in terms of what would prove or destroy the statement is usually faster than symbol manipulation alone.

# 44. What must be instantly recallable from this block

A predicate is an open statement whose truth depends on its variables and domain. Quantifying or assigning those variables turns it into a statement with a definite truth value.

`∀x P(x)` means every object satisfies `P`; one counterexample is enough to make it false.

`∃x P(x)` means at least one object satisfies `P`; one witness is enough to make it true.

When restricting a broad domain, universal statements normally use implication, while existential statements normally use conjunction.

`¬∀x P(x) ≡ ∃x ¬P(x)` and `¬∃x P(x) ≡ ∀x ¬P(x)`.

`∀x(P(x) ∧ Q(x)) ≡ (∀xP(x)) ∧ (∀xQ(x))`, while `∃x(P(x) ∨ Q(x)) ≡ (∃xP(x)) ∨ (∃xQ(x))`.

Do not assume that `(∃xP(x)) ∧ (∃xQ(x))` gives one common witness; the two existential witnesses may be different.

Do not interchange `∀x∃y` and `∃y∀x`. The former allows the chosen `y` to depend on `x`; the latter requires one common `y`.

“Exactly one” always requires **existence plus uniqueness**.

# 45. Scope boundary after video 47

These notes stop exactly at the material studied through playlist position 47. The formal treatment of **nested quantifiers** begins at position 52, followed later by negating nested quantified expressions and rules of inference for quantified statements. Those later ideas are deliberately not folded into this section yet.

The GATE 2025 Set 2 predicate-over-natural-numbers question inspected in the planner is also not used as a core worked example here because its decisive idea is mathematical induction rather than the first-order-logic material covered in videos 32–47. It will make more sense when the relevant proof/induction context is being revised.

# Sources used for this block

Neso Academy — Discrete Mathematics playlist, positions 32–47:  
https://www.youtube.com/playlist?list=PLBlnK6fEyqRhqJPDXcvYlLfXPh37L89g3

GATE CSE 2017 Set 1 Q2:  
https://gateoverflow.in/118701/gate-cse-2017-set-1-question-02

GATE CSE 2023 first-order logic PYQ:  
https://gateoverflow.in/399295/gate-cse-2023-question-16

GATE CSE 2025 Set 1 first-order logic PYQ:  
https://gateoverflow.in/460042/gate-cse-2025-set-1-question-38

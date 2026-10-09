# First-Order Logic — GATE Notes

> **Coverage:** Neso Academy Discrete Mathematics playlist positions 32–72: introduction to first-order logic, predicates, basic quantifiers, restricted domains, quantified equivalences and negation, English-to-logic translation, nested quantifiers, negating nested quantified expressions, resolution, fallacies, quantified inference rules, Universal Modus Ponens, and Universal Modus Tollens. Relevant GATE PYQs are solved in context. A short mathematical-induction bridge is included only because it is required for the mapped GATE 2025 CS2 predicate question.
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

# 45. Checkpoint after video 47

At this earlier checkpoint, the notes had reached the foundations of first-order logic: predicates, single quantifiers, restricted domains, quantified negation, translation, and the first PYQ applications. The later study block extends this foundation with nested quantifiers, resolution, fallacies, and quantified inference rules.

# Sources used for the first FOL block

Neso Academy — Discrete Mathematics playlist, positions 32–47:  
https://www.youtube.com/playlist?list=PLBlnK6fEyqRhqJPDXcvYlLfXPh37L89g3

GATE CSE 2017 Set 1 Q2:  
https://gateoverflow.in/118701/gate-cse-2017-set-1-question-02

GATE CSE 2023 first-order logic PYQ:  
https://gateoverflow.in/399295/gate-cse-2023-question-16

GATE CSE 2025 Set 1 first-order logic PYQ:  
https://gateoverflow.in/460042/gate-cse-2025-set-1-question-38

---

# Part 4 — Nested Quantifiers, Resolution, and Quantified Inference (Videos 52–72)

> **Coverage:** Neso Academy positions 52–72. This section continues directly from the first-order-logic foundations above. It develops nested quantifiers, English translation with several variables, negating nested quantified expressions, the resolution principle, logical fallacies, the four quantified inference rules, Universal Modus Ponens, and Universal Modus Tollens. The earlier GATE PYQs in Sections 39–41 remain part of these notes and should now be reread with the deeper theory below.

# 46. Nested quantifiers — what changes when more than one object is involved

A single quantifier tells us how a predicate behaves over one variable. A **nested quantified statement** uses two or more quantifiers because the statement describes a relationship between several objects.

For example, let `Likes(x, y)` mean “x likes y.” Then:

`∀x∃y Likes(x, y)`

means that **every person likes at least one person**. The important detail is that the chosen `y` may depend on `x`. Alice may like Bob, while Charlie may like David. The statement does not demand one common person who is liked by everyone.

Compare it with:

`∃y∀x Likes(x, y)`

which means that **there is one particular person whom everyone likes**. Here the existential choice is made first, so that same `y` must work for every `x`.

This is the central idea behind nested quantifiers: **an inner existential variable may depend on the variables quantified before it, but a variable chosen earlier cannot depend on a variable that has not yet been quantified.**

A useful way to read a formula is therefore to move from left to right and ask, at each quantifier, “who is being chosen now, and what earlier choices may this object depend on?”

# 47. Quantifier order can change the entire meaning

Quantifiers of the **same type** can usually be exchanged without changing meaning:

`∀x∀y P(x, y) ≡ ∀y∀x P(x, y)`

and:

`∃x∃y P(x, y) ≡ ∃y∃x P(x, y)`

In the first case, the statement requires the predicate to hold for every ordered pair anyway. In the second, it only requires that at least one suitable pair exists, so the order in which the two witnesses are named does not matter.

Mixed quantifiers are different:

`∀x∃y P(x, y)`

and:

`∃y∀x P(x, y)`

are generally **not equivalent**.

The first permits a different witness `y` for each `x`. The second demands one common witness that works for every `x`. Because one common witness is a stronger requirement, `∃y∀x P(x, y)` will often imply `∀x∃y P(x, y)`, while the reverse implication need not hold.

This is exactly the issue behind the GATE 2017 and GATE 2023 PYQs already solved above. The exam is often testing whether you silently allow the existential witness to change when the formula does not permit it.

# 48. Translating English statements with nested quantifiers

The safest method is to identify the **outermost claim first**, then work inward.

Let `Knows(x, y)` mean “x knows y,” with the domain being all people.

“Everyone knows someone” becomes:

`∀x∃y Knows(x, y)`

For each person `x`, at least one suitable `y` must exist.

“Someone knows everyone” becomes:

`∃x∀y Knows(x, y)`

Now one particular person `x` must know every `y`.

“Everyone is known by someone” becomes:

`∀y∃x Knows(x, y)`

This may look similar to “everyone knows someone,” but the arguments of the predicate have exchanged roles. The first variable is the knower, while the second is the person being known.

“No one knows everyone” can be written as:

`∀x¬∀y Knows(x, y)`

Pushing the negation inward gives:

`∀x∃y ¬Knows(x, y)`

The second form says something very concrete: for every person, there is at least one person they do not know.

The best exam habit is not to memorize dozens of sentence templates. Instead, decide who the English sentence talks about first, determine whether that claim is universal or existential, and then translate the relationship inside.

# 49. A subtle translation trap — “every country except Libya”

The example from the videos is useful because it exposes the difference between **restricting a universal statement** and describing the **exact set** of objects for which a predicate is true.

Let `V(x, y)` mean “person x has visited country y.”

If we write:

`∃x∀y(y ≠ Libya → V(x, y))`

we are saying that there is someone who has visited every country **other than Libya**. This formula places no requirement at all on whether that person visited Libya. When `y = Libya`, the antecedent `y ≠ Libya` is false, so the implication is automatically true.

Therefore this formula still allows the person to have visited Libya as well.

The statement “someone has visited every country **except Libya**,” when “except” means Libya is precisely the omitted country, is more accurately represented by:

`∃x∀y(V(x, y) ↔ y ≠ Libya)`

The biconditional enforces both directions. Every non-Libyan country must have been visited, and anything that was visited must be a country other than Libya. In particular, substituting `y = Libya` forces `V(x, Libya)` to be false.

Why not use:

`V(x, y) ∧ y ≠ Libya`

inside the universal quantifier? Because `∀y` also tests the formula at `y = Libya`. At that value, `y ≠ Libya` is false, making the conjunction false and causing the whole universal statement to fail.

This gives a useful translation distinction:

- Use an implication when you want to say **all objects satisfying a condition have some property**.
- Use a biconditional when you want to say **exactly the objects satisfying a condition have that property**.

# 50. Negating nested quantifiers

The single-quantifier rules extend mechanically to arbitrarily long quantified expressions. Every time a negation passes through a quantifier, the quantifier flips:

`∀ ↔ ∃`

and the negation continues inward.

For example:

`¬∀x∃y P(x, y)`

becomes:

`∃x¬∃y P(x, y)`

which becomes:

`∃x∀y ¬P(x, y)`

So the complete equivalence is:

`¬∀x∃y P(x, y) ≡ ∃x∀y¬P(x, y)`

In English, negating “for every x, there exists some y for which P holds” produces “there exists an x for which P fails for every y.”

Similarly:

`¬∃x∀y P(x, y) ≡ ∀x∃y ¬P(x, y)`

This says that if it is false that one `x` works for every `y`, then every candidate `x` must fail for at least one `y`.

This is one of the most reliable mechanical procedures in logic. **Flip each quantifier as the negation crosses it, then negate the innermost predicate.** After that, simplify the predicate using ordinary propositional rules if necessary.

# 51. Counterexamples for quantified implications

When GATE asks whether one quantified formula implies another, it is often faster to construct a tiny model than to perform symbolic manipulation.

To disprove:

`F → G`

you only need one interpretation in which `F` is true and `G` is false.

Domains containing only two objects are often enough. For example, to show that:

`∀x∃y R(x, y)`

does not imply:

`∃y∀x R(x, y)`,

use the domain `{1, 2}` and let only `R(1,1)` and `R(2,2)` be true. Every `x` has a suitable `y`, so the first statement is true. But no single `y` works for both values of `x`, so the second statement is false.

This tiny-model method is extremely useful because it turns an abstract formula into a concrete relationship table. If a proposed implication feels suspicious, try to make the premise true with the smallest possible domain while deliberately breaking the conclusion.

# 52. The resolution principle — proving inconsistency by eliminating complementary literals

The resolution principle is a rule for combining clauses that contain complementary literals. In its simplest propositional form:

`p ∨ A`

and:

`¬p ∨ B`

allow us to infer:

`A ∨ B`

The literals `p` and `¬p` are resolved away. The reasoning is sound because one of them must be false. If `p` is true, the second clause must rely on `B`; if `p` is false, the first clause must rely on `A`. Either way, at least one of `A` or `B` must hold.

Resolution is especially useful for checking whether an argument is valid. Instead of trying to derive the conclusion directly, combine the premises with the **negation of the conclusion**. If repeated resolution eventually produces the **empty clause**, written conceptually as a contradiction, then the premises together with the negated conclusion are unsatisfiable. Therefore the conclusion cannot be false while all premises are true, which means the original argument is valid.

The conceptual chain is:

**premises + negated conclusion → clauses → resolution → contradiction → argument valid**.

For GATE, the important point is the logic behind the method, not performing long automated-theorem-proving derivations by hand.

# 53. Resolution and ordinary inference are two views of the same validity question

Earlier, validity was defined as the impossibility of making every premise true while the conclusion is false. Resolution uses exactly that definition in a proof-by-contradiction form.

Suppose the premises are `P1, P2, ..., Pn` and the claimed conclusion is `C`. The argument is valid when:

`P1 ∧ P2 ∧ ... ∧ Pn ∧ ¬C`

is unsatisfiable.

Resolution is simply a systematic way to expose that unsatisfiability.

This explains why producing an empty clause is decisive. The empty clause represents a condition that cannot be satisfied. Once it is derived from the premises and `¬C`, the assumed counterexample to the argument has collapsed into contradiction.

# 54. Fallacies — patterns that look like inference rules but are invalid

The fallacies video reinforces a distinction already introduced in the propositional-logic notes. The most important traps are attempts to reverse an implication without justification.

From:

`p → q`

and:

`q`

you **cannot** conclude `p`. This is **affirming the consequent**. The fact that `p` would cause `q` does not mean `p` is the only possible cause of `q`.

Similarly, from:

`p → q`

and:

`¬p`

you **cannot** conclude `¬q`. This is **denying the antecedent**. The failure of one sufficient condition does not force the consequence to fail.

Compare them with the valid patterns:

`p → q, p ⟹ q` — Modus Ponens

and:

`p → q, ¬q ⟹ ¬p` — Modus Tollens.

When a GATE option seems to “reverse the arrow,” mentally identify whether it is a contrapositive, which is valid, or merely a converse/inverse-style leap, which generally is not.

# 55. Rules of inference for quantified statements

Quantified reasoning needs rules that let us move safely between statements about an entire domain and statements about individual objects.

## Universal Instantiation

From:

`∀x P(x)`

we may conclude:

`P(a)`

for any particular object `a` in the domain.

The universal statement promises that `P` holds for every object, so choosing one specific object cannot break it.

## Universal Generalization

If `P(a)` has been proved for an **arbitrary** object `a`, with no special assumption about that object, we may conclude:

`∀x P(x)`

The word arbitrary is essential. Proving that one special object has a property does not justify a universal conclusion. The proof must work without using anything peculiar to the chosen object.

## Existential Instantiation

From:

`∃x P(x)`

we may introduce a fresh representative, say `a`, and reason using `P(a)`.

However, that `a` represents **some unknown witness**, not an arbitrary member of the domain. You cannot later generalize from it as though it represented everyone. It must also be treated as a fresh object so that accidental assumptions about previously named objects do not enter the proof.

## Existential Generalization

From:

`P(a)`

we may conclude:

`∃x P(x)`.

If one concrete object has the property, then certainly at least one object with that property exists.

These four rules are easy to remember if you think about the amount of information being used: universal statements can be specialized to one case, while one known successful case can establish an existence claim.

# 56. Universal Modus Ponens

Universal Modus Ponens combines universal instantiation with ordinary Modus Ponens.

Suppose:

`∀x(P(x) → Q(x))`

and we know:

`P(a)`.

From the universal premise, instantiate at `a`:

`P(a) → Q(a)`.

Then ordinary Modus Ponens gives:

`Q(a)`.

In English, if every object with property `P` must also have property `Q`, and a particular object `a` has `P`, then `a` must also have `Q`.

A common example is:

“Every computer-science student studies discrete mathematics. Ravi is a computer-science student. Therefore Ravi studies discrete mathematics.”

The quantified statement supplies the rule; the second premise identifies an object to which the rule applies.

# 57. Universal Modus Tollens

Universal Modus Tollens uses the same universal conditional but reasons through the contrapositive.

Given:

`∀x(P(x) → Q(x))`

and:

`¬Q(a)`,

instantiate the universal statement at `a`:

`P(a) → Q(a)`.

Ordinary Modus Tollens then gives:

`¬P(a)`.

So if every `P` must be a `Q`, and a particular object is known not to be a `Q`, that object cannot be a `P`.

This is not a reversal of the implication. It is the valid contrapositive reasoning already studied in propositional logic, now applied after universal instantiation.

# 58. How the major GATE PYQs fit the completed first-order-logic theory

The earlier worked PYQs should now be read as applications of a small number of recurring ideas rather than isolated questions.

**GATE 2017 Set 1 Q2** tests quantifier order, witness dependence, and quantified negation. The counterexample showing why `∀x∃y R(x,y)` does not imply `∃y∀x R(x,y)` is exactly the dependency distinction developed in Sections 46–47. Statement IV is solved by repeatedly flipping quantifiers while pushing a negation inward.

**GATE 2023 — Geetha’s conjecture** tests the relative strength of quantified statements. A single common witness, `∃y∀x(...)`, is stronger than merely allowing each `x` to have its own witness, `∀x∃y(...)`. A claim about one successful `x`, however, cannot establish a universal conclusion about every relevant `x`.

**GATE 2025 CS1 Q48 — “Everyone has exactly one mother”** tests existence and uniqueness simultaneously. The correct forms guarantee one actual mother and then forbid any distinct second mother. Sections 37 and 47 explain why the existential witness must be present and why the uniqueness condition must refer back to that same witness.

These questions are already fully solved in Sections 39–41; the completed theory now explains _why_ those solution patterns work.

# 59. Bridge topic — Mathematical Induction is not first-order logic, but it appears in a mapped logic-style PYQ

Mathematical induction is a proof technique for statements indexed by the natural numbers. It is included here only as a bridge because a GATE question written using a predicate `P(x)` is actually testing induction rather than quantifier manipulation.

To prove `P(n)` for every natural number starting at `0`, induction requires two ingredients.

First, establish the **base case**:

`P(0)`.

This proves that the chain begins at the first natural number.

Second, establish the **inductive step**:

`∀x(P(x) → P(x+1))`.

This says that whenever the property is true at one natural number, it must also be true at the next.

Together, these imply:

`∀x P(x)`.

The logic is like an infinite row of dominoes. The base case knocks down the first domino, while the inductive step guarantees that every fallen domino knocks down the next one. Neither ingredient alone is enough.

# 60. GATE 2025 CS2 Q15 — predicate notation hiding mathematical induction

The official question asks which statement is true for an arbitrary predicate `P(x)` over the natural numbers. The correct option is **A**.

Option A is:

`(P(0) ∧ ∀x[P(x) → P(x+1)]) → ∀xP(x)`

This is precisely the principle of mathematical induction. `P(0)` supplies the base case, and `P(x) → P(x+1)` supplies the forward inductive step for every natural number. Therefore the property propagates from `0` to `1`, from `1` to `2`, and so on, covering the entire domain.

Option B starts with `P(0)` but uses:

`P(x) → P(x-1)`.

That moves **backward**, so it cannot generate `P(1), P(2), ...` from the base case. It therefore does not establish the predicate for all natural numbers.

Option C starts at `P(1000)` and also moves backward. Even if the rule establishes the predicate for numbers below `1000`, it gives no route to numbers greater than `1000`. Hence it cannot prove the universal conclusion.

Option D starts at `P(1000)` and moves forward. That can establish the predicate for `1000, 1001, 1002, ...`, but says nothing about the natural numbers below `1000`.

Therefore only **Option A** covers the entire natural-number domain.

**Exam lesson:** do not assume every question containing `P(x)` and `∀x` is mainly about first-order logic. Sometimes the predicate notation is simply the language used to state another theorem, such as mathematical induction.

# 61. Final solving strategy for first-order logic in GATE

When you see a quantified formula, first identify the domain and read the formula into plain English. Then ask whether each quantifier is universal or existential and whether an inner witness is allowed to depend on an outer variable.

When translating English, distinguish between **restriction** and **exact characterization**. Universal restrictions often use implication; existential membership usually uses conjunction; “exactly one” needs existence plus uniqueness; and wording such as “except” may require a biconditional when the excluded object must genuinely fail the predicate.

When negating, push the negation inward mechanically. Every crossed `∀` becomes `∃`, every crossed `∃` becomes `∀`, and the final predicate is negated.

When testing a claimed implication, try a two-object countermodel before doing lengthy algebra. When proving an argument, choose the cheapest valid method: direct quantified inference, ordinary propositional inference after instantiation, or resolution when contradiction is the natural route.

Most importantly, never swap mixed quantifiers casually and never assume two existential statements use the same witness unless the formula explicitly forces them to.

# 62. What must now be instantly recallable

The completed first-order-logic block should leave the following ideas automatic:

- `∀x∃y P(x,y)` allows `y` to depend on `x`; `∃y∀x P(x,y)` requires one common `y`.
- Quantifiers of the same type can be reordered; mixed `∀` and `∃` generally cannot.
- Negating a nested quantified formula flips every quantifier crossed and negates the final predicate.
- Universal restrictions normally use implication, while existential restrictions normally use conjunction.
- A biconditional can be required when an English statement describes **exactly** which objects satisfy a predicate, as in the strong reading of “every country except Libya.”
- “Exactly one” always requires both existence and uniqueness.
- Universal Instantiation specializes a universal claim; Universal Generalization requires an arbitrary object; Existential Instantiation introduces a fresh witness; Existential Generalization turns one known case into an existence claim.
- Universal Modus Ponens and Universal Modus Tollens are ordinary MP/MT after instantiating the universal conditional.
- Resolution proves validity by showing that the premises together with the negation of the conclusion lead to contradiction.
- A predicate-over-natural-numbers question may actually be testing mathematical induction rather than first-order-logic manipulation.

# Sources used for Part 4

Neso Academy — Discrete Mathematics playlist, positions 52–72:  
https://www.youtube.com/playlist?list=PLBlnK6fEyqRhqJPDXcvYlLfXPh37L89g3

TrevTutor — Mathematical Induction:  
https://www.youtube.com/watch?v=Tm2PJPvAULs

GATE 2025 CS2 Master Question Paper — Q15:  
https://gate2025.iitr.ac.in/doc/2025/2025_QP/CS2.pdf

GATE 2025 CS2 Final Answer Key:  
https://gate2025.iitr.ac.in/doc/2025/2025_Key/CS2_Keys.pdf

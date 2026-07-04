# A Tutorial in Logic: From Boolean operations to temporal logic for autonomous systems

---

## 1. Why logic

Logic is the study of correct reasoning. It gives us a precise language for making claims and a set of rules for deciding when one claim follows from another. That precision is what makes it useful well beyond philosophy. Digital circuits are Boolean logic made physical. Program verification, database queries, AI planning, and the safety specifications for an autonomous vehicle all rest on the layers of logic we build up over these two days.

The through line of the whole tutorial is a single question. Given some things we know or assume, what else must be true? Each system of logic we study is a different answer to that question, tuned to a different kind of subject matter. Boolean logic handles simple true or false values. Propositional logic combines them. Predicate and first-order logic let us talk about objects and their properties. Temporal logic lets us talk about how truth changes over time, which is exactly what you need when you want to say that a drone must never leave its geofence and must eventually return home.

---

## 2. Boolean logic and Boolean algebra

Boolean logic works with just two values, usually written true and false, or 1 and 0. Everything else is built from operations that take these values in and produce a value out. George Boole introduced this algebra in the 1850s (Boole, 1854), and it is the mathematical bedrock of every digital computer.

### 2.1 Values and variables

A Boolean variable is a name that stands for one of the two values. We will use letters like p, q, and r. A Boolean expression is any combination of variables, values, and operations that evaluates to true or false once you fix the values of the variables.

### 2.2 The basic operations

There are three operations you must know cold, because all the others are built from them.

**NOT** (written ¬p, or sometimes !p or p with an overbar) flips the value. NOT true is false, and NOT false is true. It is a unary operation, meaning it takes a single input.

**AND** (written p ∧ q, or p · q, or p && q) is true exactly when both inputs are true.

**OR** (written p ∨ q, or p + q, or p || q) is true when at least one input is true. This is inclusive or, so it is also true when both inputs are true. That last point trips up newcomers because everyday English often means "one or the other but not both."

A truth table lists every combination of inputs and the resulting output. Here are the three basic operations side by side.

| p | q | ¬p | p ∧ q | p ∨ q |
|---|---|----|-------|-------|
| 0 | 0 | 1  | 0     | 0     |
| 0 | 1 | 1  | 0     | 1     |
| 1 | 0 | 0  | 0     | 1     |
| 1 | 1 | 0  | 1     | 1     |

### 2.3 The derived operations

![Logic gate symbols with truth tables](https://commons.wikimedia.org/wiki/Special:FilePath/Logic_Gates.svg)

*Figure: Standard logic gate symbols.*

Four more operations show up constantly, and each can be expressed using the three above.

**XOR**, exclusive or (written p ⊕ q), is true when the inputs differ. This is the "one or the other but not both" meaning. You can write it as (p ∨ q) ∧ ¬(p ∧ q).

**NAND** (not-and) is the negation of AND. It is false only when both inputs are true. NAND is important in hardware because you can build every other operation out of NAND gates alone.

**NOR** (not-or) is the negation of OR. It has the same universal property as NAND.

**Implication** (written p → q, read "if p then q") is the operation that most often confuses people, so it gets its own subsection below.

| p | q | p ⊕ q | p NAND q | p NOR q | p → q |
|---|---|-------|----------|---------|-------|
| 0 | 0 | 0     | 1        | 1       | 1     |
| 0 | 1 | 1     | 1        | 0       | 1     |
| 1 | 0 | 1     | 1        | 0       | 0     |
| 1 | 1 | 0     | 0        | 0       | 1     |

### 2.4 Making sense of implication

Implication p → q is false in exactly one case, when p is true and q is false. In every other case it is true. The surprising part is that when p is false, the whole implication is true no matter what q is. Logicians call this vacuous truth.

The way to make peace with this is to read p → q as a promise. The promise says, "if p happens, then q will happen." The only way to break that promise is for p to happen while q does not. If p never happens, you have not broken the promise, so the implication holds. A statement like "if the aircraft is armed then the propellers may spin" is not violated on the ground with the system disarmed, because the condition was never triggered.

The biconditional p ↔ q, read "p if and only if q," is true when p and q have the same value. It is the same as (p → q) ∧ (q → p).

### 2.5 The laws of Boolean algebra

Boolean expressions can be simplified and rearranged using a fixed set of laws. These are the tools you use to prove two expressions equal and to reduce a complicated expression to a simpler one. Learn them well, because the same equivalences reappear in propositional logic in the next section.

The identity laws say that p ∧ 1 = p and p ∨ 0 = p. The domination laws say p ∨ 1 = 1 and p ∧ 0 = 0. The idempotent laws say p ∧ p = p and p ∨ p = p. Double negation says ¬¬p = p.

The commutative laws let you swap the order of operands, so p ∧ q = q ∧ p and likewise for OR. The associative laws let you regroup, so (p ∧ q) ∧ r = p ∧ (q ∧ r). The distributive laws let you factor, so p ∧ (q ∨ r) = (p ∧ q) ∨ (p ∧ r), and the dual p ∨ (q ∧ r) = (p ∨ q) ∧ (p ∨ r).

Two laws deserve special attention because they turn up everywhere from circuit design to program refactoring. These are De Morgan's laws, named for Augustus De Morgan (De Morgan, 1847). The negation of an AND is the OR of the negations, ¬(p ∧ q) = ¬p ∨ ¬q. The negation of an OR is the AND of the negations, ¬(p ∨ q) = ¬p ∧ ¬q. In plain terms, pushing a negation inward flips AND to OR and OR to AND.

### 2.6 Operator reference

| Operator | Symbol | Meaning | Example |
|----------|--------|---------|---------|
| NOT | ¬p | Flips the value | ¬armed is true exactly when the system is not armed |
| AND | p ∧ q | True only when both are true | armed ∧ airborne is true only when the system is both armed and in the air |
| OR | p ∨ q | True when at least one is true | low_battery ∨ low_fuel is true when either warning is active |
| XOR | p ⊕ q | True when exactly one is true | manual ⊕ auto is true when exactly one control mode is engaged |
| NAND | ¬(p ∧ q) | False only when both are true | Any other gate can be built from NAND alone |
| NOR | ¬(p ∨ q) | True only when both are false | idle equals ¬(armed ∨ airborne), true only when neither holds |
| Implication | p → q | False only when p is true and q is false | armed → clearance says arming implies clearance was given |
| Biconditional | p ↔ q | True when both sides have the same value | gear_down ↔ landing_mode links the two so they rise and fall together |

### 2.7 Exercises

1. Build the full truth table for (p ∧ q) ∨ ¬r.
2. Use De Morgan's laws to rewrite ¬(p ∨ ¬q) without a negation on the outside.
3. Show with a truth table that p → q is equivalent to ¬p ∨ q.
4. XOR can be written using AND, OR, and NOT. Write it a second, different way.
5. Simplify (p ∧ q) ∨ (p ∧ ¬q) using the laws above until you reach a single variable.

---

## 3. Propositional logic

Propositional logic takes the Boolean operations and wraps them in a story about meaning. Where Boolean algebra manipulates abstract values, propositional logic is about propositions, which are declarative statements that are either true or false.

### 3.1 Propositions

A proposition is a statement that has a definite truth value. "The airport is closed" is a proposition. "Close the runway" is not, because a command is neither true nor false. "x is greater than five" is not a proposition on its own either, because its truth depends on x. Handling statements with variables like that is exactly what predicate logic will fix later today.

An atomic proposition cannot be broken into smaller propositions. We give each one a letter. A compound proposition is built from atomic ones using connectives, which are just the Boolean operations wearing their logical names: negation, conjunction, disjunction, implication, and biconditional.

### 3.2 Well-formed formulas

Not every string of symbols is meaningful. A well-formed formula, often abbreviated wff, is one built according to the grammar of the logic. The rules are simple. Any atomic proposition is a formula. If A is a formula, so is ¬A. If A and B are formulas, so are (A ∧ B), (A ∨ B), (A → B), and (A ↔ B). Nothing else is a formula.

Parentheses remove ambiguity, but to reduce clutter we adopt precedence rules. Negation binds tightest, then conjunction, then disjunction, then implication, then biconditional. So p ∨ q ∧ r means p ∨ (q ∧ r), and ¬p → q means (¬p) → q.

### 3.3 Evaluating formulas

To evaluate a formula you assign a truth value to each atomic proposition, then work outward using the truth tables. An assignment of values to all the variables is called an interpretation or a valuation. A formula with n distinct variables has 2 to the n possible interpretations, which is why truth tables double in height with each new variable.

### 3.4 Tautology, contradiction, contingency

Some formulas are true under every interpretation. These are tautologies, and they represent logical truths that hold no matter what. The classic example is p ∨ ¬p, the law of the excluded middle. A formula false under every interpretation is a contradiction, such as p ∧ ¬p. A formula that is true under some interpretations and false under others is a contingency, and most interesting statements are contingencies.

### 3.5 Logical equivalence

Two formulas are logically equivalent when they have the same truth value under every interpretation. We write A ≡ B. The Boolean laws from section 2.5 are all statements of logical equivalence, and they carry over unchanged. De Morgan's laws, distribution, and the rest all apply to propositional formulas. A useful test is that A ≡ B holds exactly when A ↔ B is a tautology.

### 3.6 Entailment and validity

Here we reach the heart of the matter. A set of premises entails a conclusion when every interpretation that makes all the premises true also makes the conclusion true. We write this with the double turnstile, so {A, B} ⊨ C means A and B together entail C. Entailment is the formal version of the question we started with, namely what must be true given what we know.

An argument is valid when its premises entail its conclusion. Validity is about form, not about whether the premises happen to be true. An argument can be valid with false premises, and an argument with true premises and a true conclusion can still be invalid if the conclusion does not follow from the premises.

### 3.7 Normal forms

It is often convenient to put formulas into a standard shape. Two standard shapes matter most.

A formula is in conjunctive normal form, or CNF, when it is an AND of clauses, where each clause is an OR of literals, and a literal is a variable or its negation. CNF is the input format for SAT solvers, the tools that decide whether a formula can be made true. Deciding satisfiability is the original NP-complete problem (Cook, 1971), which is why these solvers are engineered with such care. A formula is in disjunctive normal form, or DNF, when it is an OR of terms, where each term is an AND of literals. Every propositional formula can be converted to either form using the laws we have, mainly distribution and De Morgan's.

### 3.8 Inference rules and proof

Truth tables let us check entailment by brute force, but they blow up as the number of variables grows. Proof systems let us derive conclusions step by step instead. A proof uses inference rules, each of which is a small pattern that licenses a new line from earlier lines. When a conclusion is provable from premises we write the single turnstile, A ⊢ B, read "A proves B."

The most important rules are worth memorizing. Modus ponens says that from A and A → B you may conclude B. Modus tollens says that from ¬B and A → B you may conclude ¬A. Hypothetical syllogism chains implications, so from A → B and B → C you get A → C. Disjunctive syllogism says that from A ∨ B and ¬A you get B. Conjunction introduction lets you combine A and B into A ∧ B, and conjunction elimination lets you pull either half back out.

A logic is sound when everything provable is actually entailed, so ⊢ never outruns ⊨. It is complete when everything entailed is provable, so ⊢ reaches everything ⊨ does. Propositional logic is both sound and complete, which is a deep and reassuring result (see, for example, Huth and Ryan, 2004). It means the syntactic game of pushing symbols around lines up exactly with the semantic notion of truth.

### 3.9 Operator reference

![The sixteen two-input logical connectives as Venn diagrams](https://commons.wikimedia.org/wiki/Special:FilePath/Logical_connectives_Hasse_diagram.svg)

*Figure: The sixteen binary logical connectives.*

The connectives are the Boolean operations from section 2, now applied to whole statements. What matters here is reading them fluently in English.

| Connective | Symbol | Read as | Example |
|------------|--------|---------|---------|
| Negation | ¬P | "not P" | ¬Raining: it is not raining |
| Conjunction | P ∧ Q | "P and Q" | Cleared ∧ Fueled: the aircraft is cleared and fueled |
| Disjunction | P ∨ Q | "P or Q, or both" | Delayed ∨ Cancelled: the flight is delayed or cancelled |
| Implication | P → Q | "if P then Q" | Raining → RunwayWet: if it is raining then the runway is wet |
| Biconditional | P ↔ Q | "P if and only if Q" | Safe ↔ AllChecksPass: safe exactly when every check passes |

The inference rules are the operations of the proof system, the moves that carry you from lines you already have to a new line.

| Rule | Pattern | Meaning | Example |
|------|---------|---------|---------|
| Modus ponens | from P and P → Q, infer Q | affirm the antecedent | From armed and armed → ready, conclude ready |
| Modus tollens | from ¬Q and P → Q, infer ¬P | deny the consequent | From not ready and armed → ready, conclude not armed |
| Hypothetical syllogism | from P → Q and Q → R, infer P → R | chain implications | From armed → ready and ready → cleared, conclude armed → cleared |
| Disjunctive syllogism | from P ∨ Q and ¬P, infer Q | rule out one disjunct | From delayed ∨ cancelled and not delayed, conclude cancelled |
| Conjunction introduction | from P and Q, infer P ∧ Q | combine two facts | From fueled and cleared, conclude fueled ∧ cleared |
| Conjunction elimination | from P ∧ Q, infer P or infer Q | pull out one half | From fueled ∧ cleared, conclude fueled |

### 3.10 Exercises

1. Show that (p → q) ∧ (q → r) → (p → r) is a tautology.
2. Convert (p ∧ q) ∨ (¬p ∧ r) into conjunctive normal form.
3. Decide whether {p → q, q → r, p} ⊨ r, and justify your answer.
4. Give an argument that is valid but has a false conclusion.
5. Use modus ponens and modus tollens together to derive ¬p from the premises p → q, q → r, and ¬r.

---

## 4. Predicates and quantifiers

Propositional logic hits a wall the moment we want to say something about objects and their properties. It can represent "Earhart is airborne" as a single atomic proposition, but it cannot see inside that statement to notice that it is about a specific aircraft named Earhart having the property of being airborne. It also cannot express general claims like "every aircraft in the fleet has a valid airworthiness check." Predicates and quantifiers give us that power, and they are the bridge into first-order logic.

### 4.1 What a predicate is

A predicate is a statement with one or more blanks that becomes a proposition once you fill the blanks with objects. Think of it as a function that returns true or false. Write Airborne(x) for the predicate "x is airborne." On its own, Airborne(x) has no truth value, because x is a placeholder. Fill it in, and Airborne(Earhart) is now a genuine proposition that is either true or false.

The number of blanks is the arity of the predicate. Airborne(x) is unary because it has one argument. A binary predicate takes two, such as HigherThan(x, y) meaning "x is at a higher altitude than y." Predicates of higher arity express relations among several objects at once. In fact a predicate is just a formal name for a property or relation.

### 4.2 The domain, constants, and variables

Predicates do not float free. They range over a domain of discourse, which is the set of objects we are talking about. If our domain is the fleet of aircraft at the airport, then x ranges over those aircraft and nothing else. Choosing the domain is part of setting up the problem, and the same predicate can mean different things over different domains.

A constant names a specific object in the domain, like Earhart. A variable, like x, is a placeholder that can stand for any object in the domain until we pin it down. The distinction between the two is small but it matters a great deal once quantifiers enter the picture.

### 4.3 Quantifiers

Quantifiers are what let us make general statements. There are two.

The universal quantifier, written ∀ and read "for all," claims that a predicate holds for every object in the domain. So ∀x Airworthy(x) says every object in the domain is airworthy. Over the fleet domain, that is the claim that the whole fleet is airworthy. A universal statement is false as soon as a single object fails the predicate. That single failing object is called a counterexample.

The existential quantifier, written ∃ and read "there exists," claims that a predicate holds for at least one object. So ∃x Airborne(x) says at least one object in the domain is airborne. An existential statement is true as soon as you find one witness, an object that satisfies the predicate.

There is a clean duality between the two, and it is a quantifier version of De Morgan's laws. Saying it is not the case that everything is airworthy is the same as saying something is not airworthy, so ¬∀x P(x) is equivalent to ∃x ¬P(x). Likewise ¬∃x P(x) is equivalent to ∀x ¬P(x). Negation flips the quantifier and moves inward, exactly as it flipped AND and OR earlier.

### 4.4 A first look at combining quantifiers

You can use several quantifiers in one statement, and the order matters enormously. Consider a binary predicate CanReach(x, y) meaning aircraft x can reach waypoint y. The statement ∀y ∃x CanReach(x, y) says that for every waypoint there is some aircraft that can reach it, allowing a different aircraft for each waypoint. The statement ∃x ∀y CanReach(x, y) says something much stronger, that there is one single aircraft that can reach every waypoint. Swapping the two quantifiers changes the meaning completely. We treat nested quantifiers carefully in the first-order logic section, because getting their order right is one of the most common sources of specification bugs.

### 4.5 Operator reference

Predicate logic keeps all the propositional connectives and adds the two quantifiers, which are its distinctive operators. The examples use a domain of the aircraft fleet.

| Operator | Symbol | Meaning | Example |
|----------|--------|---------|---------|
| Universal quantifier | ∀x P(x) | P holds for every object in the domain | ∀x Airworthy(x): every aircraft in the fleet is airworthy |
| Existential quantifier | ∃x P(x) | P holds for at least one object | ∃x Airborne(x): at least one aircraft is airborne |
| Negated universal | ¬∀x P(x) | equivalent to ∃x ¬P(x) | ¬∀x Airworthy(x): some aircraft is not airworthy |
| Negated existential | ¬∃x P(x) | equivalent to ∀x ¬P(x) | ¬∃x Airborne(x): no aircraft is airborne, so all are grounded |

### 4.6 Exercises

1. Let the domain be the fleet. With predicates Airborne(x) and Airworthy(x), write in symbols: every airborne aircraft is airworthy.
2. Translate ∃x (Airborne(x) ∧ ¬Airworthy(x)) into plain English, and say why it would be alarming.
3. Negate ∀x (Armed(x) → Airborne(x)) and simplify so that the negation sits directly on the atomic predicates.
4. Explain the difference in meaning between ∀x ∃y HigherThan(y, x) and ∃y ∀x HigherThan(y, x).
5. Give a domain and a predicate for which ∀x P(x) is false but ∃x P(x) is true.

---

## 5. First-order logic

First-order logic, also called predicate logic or first-order predicate calculus, is the full system built on the predicates and quantifiers from section 4. It is expressive enough to capture most of mathematics and most everyday reasoning about objects, and it is the standard language of formal specification.

### 5.1 Syntax: terms and formulas

First-order logic distinguishes two kinds of syntactic objects. Terms name objects, and formulas make claims that are true or false.

A term is either a constant like Earhart, a variable like x, or a function applied to other terms. A function symbol maps objects to objects rather than to truth values. For example altitude(x) could be a function returning the altitude of x, and it is a term, not a formula, because it names a number rather than making a claim. This is the key difference between a function and a predicate. A function returns an object, a predicate returns true or false.

A formula is built from predicates applied to terms, combined with the propositional connectives from section 3, and closed under the two quantifiers. So Airborne(x), altitude(x) > 100, ∀x (Armed(x) → Ready(x)), and their combinations are all formulas.

### 5.2 Free and bound variables

A variable is bound when it falls under a quantifier that names it, and free otherwise. In ∀x P(x, y) the variable x is bound by the universal quantifier while y is free. A formula with no free variables is called a sentence, and only sentences have a definite truth value once you fix an interpretation. A formula with free variables, like a bare predicate, is more like a predicate itself, waiting for its free variables to be supplied.

The same variable name can be bound in one part of a formula and free in another, which is a frequent source of confusion. When in doubt, rename bound variables so that each quantifier uses a fresh letter. This never changes the meaning and it removes the ambiguity.

### 5.3 Semantics: interpretations and models

To give a first-order sentence a truth value you need an interpretation, sometimes called a structure or a model. An interpretation fixes three things. First, a non-empty domain of objects. Second, a meaning for each constant, namely which object it names. Third, a meaning for each predicate and function, namely which tuples of objects satisfy the predicate and what each function returns.

Once the interpretation is fixed, every sentence gets a truth value by the natural rules. A universal sentence is true when the body holds for every object in the domain. An existential sentence is true when the body holds for at least one. The connectives behave exactly as in propositional logic. When a sentence is true in an interpretation, we say the interpretation is a model of the sentence, and we write it with the same double turnstile as before.

### 5.4 Nested quantifiers, carefully

We saw earlier that quantifier order matters. Now we can be precise. Read a nested quantifier statement left to right, and treat each quantifier as fixing its variable before the next one is considered. In ∀x ∃y Loves(x, y), for each x you get to pick a possibly different y. In ∃y ∀x Loves(x, y) you must pick a single y that works for all x. The first is much weaker than the second, and the second implies the first but not the other way around.

This matters in practice. A mission requirement like "for every no-fly zone there exists a re-route" is a ∀∃ statement and is usually achievable. The stronger ∃∀ reading, "there exists one re-route that avoids every no-fly zone," is a different and often unattainable requirement. Writing the specification with the quantifiers in the wrong order is a real and costly bug.

### 5.5 What first-order logic can and cannot do

First-order logic is sound and complete, like propositional logic, thanks to a result of Gödel (Gödel, 1930). Anything entailed by a set of first-order premises can be proved from them. But first-order logic is only semi-decidable, a fact established independently by Church and Turing (Church, 1936; Turing, 1936). If a conclusion follows, a procedure can eventually confirm it, but if it does not follow, no general procedure is guaranteed to halt and tell you so. This is a genuine limit, not a gap in current tools.

There are also things first-order logic simply cannot say. It quantifies over objects but not over predicates or sets of objects, which is what makes it first-order. Quantifying over predicates lands you in second-order logic, which is more expressive but loses the clean completeness property. And crucially for our next topic, plain first-order logic has no built-in notion of time. To say that something must always hold, or must eventually happen, we move to temporal logic.

### 5.6 Operator reference

First-order logic reuses the connectives from section 3 and the quantifiers from section 4. What it adds are the building blocks that let a formula reach inside objects, namely predicates, functions, and equality.

| Element | Symbol | Role | Example |
|---------|--------|------|---------|
| Predicate | P(x), R(x, y) | applied to terms, returns true or false | Airborne(Earhart): the claim that Earhart is airborne |
| Function | f(x) | applied to terms, returns another object | altitude(Earhart): a term naming Earhart's altitude, not a claim |
| Equality | s = t | true when two terms name the same object | pilot_in_command(Earhart) = autopilot |
| Universal quantifier | ∀x | binds x, ranging over the whole domain | ∀x (Armed(x) → Ready(x)) |
| Existential quantifier | ∃x | binds x, asserting some object works | ∃x (Airborne(x) ∧ ¬Airworthy(x)) |

The key distinction to hold onto is that a function returns an object and so builds a term, while a predicate returns true or false and so builds a formula. altitude(x) < limit is a formula because the predicate "<" turns two terms into a claim.

### 5.7 Exercises

1. Using a function altitude(x) and a constant limit, write a first-order sentence saying no aircraft in the fleet exceeds the altitude limit.
2. Identify the free and bound variables in ∀x (P(x) → ∃y Q(x, y)) ∧ R(y).
3. Explain, with a concrete domain of your choosing, why ∃y ∀x Loves(x, y) implies ∀x ∃y Loves(x, y) but not the reverse.
4. Write "every waypoint is reached by some aircraft, and some aircraft reaches every waypoint" as two separate first-order sentences, and say which is stronger.
5. Give a property of a system that you believe cannot be expressed in first-order logic without a notion of time, and explain the obstacle.

---

## 6. Reasoning about time

Everything so far has treated truth as fixed. A proposition is true or it is false, full stop. But the interesting properties of a running system are about change. We want to say that the aircraft never leaves its geofence, that it eventually reaches its destination, that whenever a fault is detected the system enters a safe state within some bound. These are statements about how truth evolves as the system runs, and classical logic has no vocabulary for them.

Temporal logic adds that vocabulary. The core idea is to evaluate formulas not at a single fixed world but along a sequence of states that represent the system over time. A property is then judged against the whole sequence. Two families matter for us. Linear temporal logic treats time as a single line of successive states, which fits monitoring one actual execution. Branching time logics like CTL treat the future as a tree of possibilities, which fits reasoning about all the ways a system could behave. We cover linear time first because it is the more common tool for specification and monitoring, then look briefly at branching time, then extend to continuous real-valued signals with signal temporal logic.

---

## 7. Linear temporal logic (LTL)

Linear temporal logic, introduced to computer science by Pnueli (1977), reasons about a single infinite sequence of states called a trace or an execution. At each state, every atomic proposition is either true or false, and the temporal operators let us relate what holds now to what holds at later states.

### 7.1 Traces

![A non-deterministic Büchi automaton](https://commons.wikimedia.org/wiki/Special:FilePath/Buchi_non_deterministic_example.svg)

*Figure: A non-deterministic Büchi automaton, the automata-theoretic counterpart of an LTL formula used in model checking.*

A trace is a sequence of states s0, s1, s2, and so on. Think of it as sampling the system at each discrete step. State s0 is now, s1 is the next step, and so on into the future. An atomic proposition like within_geofence is true or false at each individual state. An LTL formula is evaluated at a position in the trace, and by convention we usually mean position 0, the present.

### 7.2 The temporal operators

LTL keeps all the propositional connectives and adds operators that reach into the future. There are two common notations for them, a letter style and a symbol style, and both are given here.

**Next**, written X φ or ○ φ, is true at the current state when φ is true at the very next state.

**Eventually**, written F φ or ◊ φ, is true when φ holds at some state now or later. It makes no promise about which state, only that one exists.

**Always**, written G φ or □ φ, is true when φ holds at every state from now on. This is the operator you reach for to express safety properties, the ones that say nothing bad ever happens.

**Until**, written φ U ψ, is true when ψ eventually becomes true, and φ holds at every state up until that happens. Until is the fundamental binary operator, and the others can be seen as special cases of it.

Two more operators round out the set. **Release**, written φ R ψ, is the logical dual of until. It says ψ must hold up to and including the moment φ first becomes true, and if φ never becomes true then ψ must hold forever. **Weak until**, written φ W ψ, is like until but does not require ψ to ever happen, in which case φ must hold forever.

### 7.3 Reading formulas as English

The operators become intuitive once you translate a few. G within_geofence says the aircraft is always inside its geofence, a safety property. F reached_home says the aircraft eventually gets home, a liveness property. G (fault → F safe_state) says that whenever a fault occurs, a safe state is eventually reached afterward, a response property. G (low_battery → F return_to_launch) says every time the battery gets low, a return to launch eventually follows.

Nesting operators expresses subtler patterns. G F active says active holds infinitely often, because at every point in the future there is a still later point where active is true. F G stable says that eventually the system reaches a point after which stable holds forever, which is the shape of an eventually-permanent property. These two look similar but say very different things, and telling them apart is a good test of whether you have internalized the operators.

### 7.4 Safety versus liveness

Two categories of property organize most specifications. The distinction traces back to Lamport (1977) and was given a precise topological definition by Alpern and Schneider (1985). A safety property says that something bad never happens, and it is characterized by the fact that any violation shows up at some finite point in the trace. Staying within the geofence is a safety property, because if the aircraft ever leaves, there is a specific moment where the violation is visible. A liveness property says that something good eventually happens, and a violation can only be confirmed by an infinite trace where the good thing never occurs. Eventually reaching the destination is a liveness property.

The distinction is practical, not just theoretical. Safety properties can be checked by a monitor that watches the running system and raises an alarm the instant the bad thing occurs. Liveness properties cannot be refuted by any finite observation, so they are usually the target of design-time verification rather than runtime monitoring. Much of what a UAV safety monitor actually enforces at runtime is safety properties in exactly this technical sense.

### 7.5 What LTL cannot express

LTL steps in discrete units and talks about propositions that are simply true or false. It cannot natively say "the altitude stays below 120 metres for the next 30 seconds" because that statement is about a real-valued quantity and a concrete time bound, not a sequence of true-or-false steps. The standard temporal operators are also unbounded, so plain LTL cannot put a deadline on eventually. These two gaps, real values and quantitative time, are precisely what signal temporal logic fills, and we get there in section 9.

### 7.6 Operator reference

LTL keeps every propositional connective and adds the temporal operators below. The letter notation and the symbol notation mean the same thing. Examples are read at the present state of the trace.

| Operator | Symbol | Name | Meaning | Example |
|----------|--------|------|---------|---------|
| Next | X φ, or ○ φ | next | φ is true at the very next state | X armed: the system is armed at the next step |
| Eventually | F φ, or ◊ φ | eventually | φ is true at some state now or later | F reached_home: the aircraft eventually reaches home |
| Always | G φ, or □ φ | globally | φ is true at every state from now on | G within_geofence: the aircraft always stays inside the geofence |
| Until | φ U ψ | until | φ holds at every state until ψ becomes true, and ψ must happen | loiter U cleared: stays in loiter until landing clearance arrives |
| Release | φ R ψ | release | ψ holds up to and including when φ first holds, and forever if φ never does | ¬abort R engines_safe: the engines stay safe unless an abort has been raised |
| Weak until | φ W ψ | weak until | like until, but ψ need not ever happen, in which case φ holds forever | within_geofence W mission_over: stays in the geofence at least until the mission ends |

A few nested combinations come up often enough to be worth naming. G (fault → F safe_state) is a response pattern, saying every fault is eventually answered by a safe state. G F active means active holds infinitely often. F G stable means stable eventually becomes permanent.

### 7.7 Exercises

1. Write in LTL: the motors never spin while the system is disarmed.
2. Write in LTL: every arming is eventually followed by a disarming.
3. Explain in words the difference between G F φ and F G φ.
4. Express "the aircraft stays in loiter mode until it receives a landing clearance" using the until operator, and note whether weak or strong until is more appropriate and why.
5. Classify each of your answers to questions 1 and 2 as a safety property or a liveness property.

---

## 8. A brief look at branching time (CTL)

Linear temporal logic asks about a single execution. Computation tree logic, or CTL, introduced by Clarke and Emerson (1981), asks about the tree of all possible executions branching out from the current state. The difference is a shift from "along this run" to "along some or all of the runs the system could take."

CTL pairs each temporal operator with a path quantifier. The quantifier A means "along all paths" and E means "along some path." So AG φ says that on every possible execution, φ always holds, which is a strong safety guarantee. EF φ says there is at least one execution on which φ eventually holds, which is often used to check reachability, meaning some behaviour is possible. AF φ says that on every execution φ eventually holds no matter what.

You do not need CTL for most monitoring work, because a monitor watches one actual run and linear time fits that naturally. CTL earns its place at design time, when you want to verify claims about every possible behaviour of a model, such as "from every reachable state, it is always possible to reach a safe landing." That kind of possibility claim, quantifying over the branching of futures, is exactly what linear time cannot say and branching time can. We flag it here for completeness and move on to the logic that matters most for signals from real sensors.

### Operator reference

Each CTL operator is a path quantifier, A for all paths or E for some path, paired with a temporal operator. The eight standard combinations are below.

| Operator | Meaning | Example |
|----------|---------|---------|
| AX φ | φ holds in the next state on every path | AX armed: no matter what happens next, the system is armed |
| EX φ | φ holds in the next state on some path | EX abort: an abort is possible at the next step |
| AG φ | φ holds always on every path, an invariant | AG ¬collision: no possible execution ever has a collision |
| EG φ | φ holds always on some path | EG loiter: there is an execution that stays in loiter forever |
| AF φ | φ eventually holds on every path | AF disarmed: every execution eventually disarms |
| EF φ | φ eventually holds on some path, reachability | EF safe_landing: a safe landing is reachable from here |
| A[φ U ψ] | on every path, φ holds until ψ | A[in_air U landed]: every execution stays airborne until it lands |
| E[φ U ψ] | on some path, φ holds until ψ | E[climb U at_altitude]: some execution climbs until it reaches altitude |

---

## 9. Signal temporal logic (STL)

Signal temporal logic, introduced by Maler and Nickovic (2004), extends the temporal ideas of LTL to continuous time and real-valued signals. It was built as a bounded fragment of metric temporal logic augmented with predicates over continuous signals. This is the logic you want when the quantities of interest are things like altitude, velocity, battery voltage, and distance to a boundary, all sampled from real sensors, and when the timing matters in seconds rather than abstract steps. It is widely used for specifying and monitoring cyber-physical systems, including autonomous aircraft.

### 9.1 Signals instead of traces

Where LTL evaluates over a sequence of discrete states with true-or-false propositions, STL evaluates over signals, which are functions from time to real values. A signal might be the altitude of the aircraft as a function of time. Time is treated as continuous, or at least densely sampled, rather than as unit steps.

### 9.2 Predicates over signals

The atomic building blocks of STL are not bare propositions but predicates over signal values, typically inequalities. Something like altitude(t) < 120 is an atomic predicate that is true at those times when the altitude signal sits below 120. This is where the predicate machinery from section 4 pays off directly. An STL atom is a predicate applied to the value of a signal at a time, and the truth of the whole formula is assembled from these atoms just as before.

### 9.3 Bounded temporal operators

The signature feature of STL is that its temporal operators carry time bounds. Instead of an unbounded eventually, you write eventually within an interval, F[a,b] φ, which says φ becomes true at some time between a and b seconds from now. Likewise G[a,b] φ says φ holds throughout the interval from a to b, and the until operator gets a bound too, φ U[a,b] ψ.

This lets you write specifications that plain LTL cannot. G[0,∞) (altitude < 120) says the altitude stays below 120 for the whole flight. G (low_battery → F[0,30] descending) says that whenever the battery is low, the aircraft begins descending within 30 seconds. That bounded deadline is the whole point, and it is what makes STL suitable for real timing requirements.

### 9.4 Robustness: quantitative semantics

STL has a second feature that sets it apart, and it is arguably the most useful one in practice. Alongside the yes-or-no answer of whether a signal satisfies a specification, a robustness value can be defined, a real number that measures by how much the specification is satisfied or violated. This quantitative semantics was developed by Fainekos and Pappas (2009) and refined for real-valued signals by Donzé and Maler (2010).

The intuition is simple. If the requirement is altitude < 120 and the actual altitude is 90, the requirement is satisfied with 30 units of margin, so the robustness is +30. If the altitude is 135, the requirement is violated by 15, so the robustness is -15. Positive robustness means satisfied, negative means violated, and the magnitude tells you how close you are to the boundary. The temporal operators combine these values in a natural way, with always taking a minimum over the interval, since a conjunction over time is only as strong as its weakest moment, and eventually taking a maximum.

This quantitative view is powerful. It turns monitoring into a continuous margin rather than a binary alarm, so you can see a requirement being approached before it is breached. It also makes specifications usable as objectives for optimization and for controller synthesis, because a real-valued score can be maximized where a true-or-false verdict cannot. For an autonomous aircraft, a positive and comfortably large robustness on the geofence and altitude specifications throughout a flight is a concrete, measurable statement of how safe the run was.

### 9.5 Operator reference

STL keeps the propositional connectives, but its atoms are predicates over signal values and its temporal operators carry time bounds. The bound [a,b] is measured in seconds from the current time.

| Operator | Symbol | Meaning | Example |
|----------|--------|---------|---------|
| Signal predicate | e.g. x < c | true at the times when the inequality on the signal holds | altitude(t) < 120: true while the altitude signal is under 120 m |
| Bounded always | G[a,b] φ | φ holds at every time in the interval [a,b] | G[0,∞) (altitude < 120): altitude stays under 120 m for the whole flight |
| Bounded eventually | F[a,b] φ | φ holds at some time in the interval [a,b] | F[0,30] descending: the aircraft begins descending within 30 s |
| Bounded until | φ U[a,b] ψ | φ holds until ψ, and ψ becomes true at some time in [a,b] | in_transit U[0,600] arrived: stays in transit until arrival, within 10 min |
| Robustness | ρ(φ, s, t) | a real number giving the margin of satisfaction rather than a yes or no | ρ = +30 when altitude is 90 against a 120 limit; a negative value signals a violation |

The connectives combine robustness values numerically. Conjunction and always take the minimum over their inputs, since the specification is only as satisfied as its weakest point, while disjunction and eventually take the maximum.

### 9.6 Exercises

1. Write an STL formula saying the altitude stays below 120 metres for the entire flight.
2. Write an STL formula saying that within 5 seconds of a geofence breach being detected, the aircraft is commanded to return to launch.
3. For the requirement G[0,10] (distance_to_boundary > 5), a signal holds distance at 8 the whole interval except one instant where it dips to 3. State the satisfaction verdict and the sign of the robustness, and explain how the always operator produces it.
4. Explain in your own words why robustness is more useful than a plain true-or-false verdict when tuning a controller.
5. Take one LTL formula from section 7 and rewrite it in STL with an explicit time bound, then describe what changed in its meaning.

---

## 10. Putting it to work: specification and monitoring

The payoff of the whole tutorial is that these logics are the languages in which we state what a system must do, precisely enough that a machine can check it. It is worth stepping back to see how the layers stack up, because each one was introduced to overcome a specific limit of the layer below.

Boolean logic gave us true and false and the operations that combine them. Propositional logic wrapped those operations in statements and gave us entailment, the notion of one claim following from others. Predicates and quantifiers let us talk about objects, properties, and relations, and to make general claims over a whole domain. First-order logic completed that picture into a full and standard specification language, at the cost of having no notion of time. Temporal logic added time. Linear temporal logic reasons about a single execution and cleanly separates safety from liveness. Branching time reasons about all possible executions. Signal temporal logic brought in real-valued signals, explicit time bounds, and a quantitative robustness measure, which is what real sensor data and real deadlines demand.

In an autonomous flight setting the division of labour is natural. Design-time verification uses the more expressive logics to prove that a model of the system satisfies its requirements across all its behaviours. Runtime monitoring uses linear time and signal temporal logic to watch an actual flight, check safety specifications instant by instant, and report robustness margins so that a requirement being approached is visible before it is breached. A specification like "the aircraft always stays within its geofence, and within 30 seconds of a low-battery signal it begins a return to launch" is not a comment in a document. Written in STL it is a formal object that a monitor can evaluate against the telemetry of a real flight and turn into a verdict and a margin.

That is the arc of these two days. We began with two values and a handful of truth tables, and we ended with a formal, checkable, quantitative statement of what it means for an autonomous aircraft to fly safely.

### 10.1 Capstone exercises

1. Pick a small autonomous system you understand, and list three requirements for it in plain English.
2. For each requirement, decide whether it is naturally a safety or a liveness property.
3. Translate each requirement into the most appropriate logic from this tutorial, and justify the choice of logic.
4. For any requirement involving a real-valued quantity or a deadline, write it in STL and describe what its robustness value would measure.
5. Identify one requirement that you cannot express in any of these logics, and describe what additional expressive power it would need.

---

## Appendix: symbol reference

| Symbol | Name | Meaning |
|--------|------|---------|
| ¬ | not | negation |
| ∧ | and | conjunction |
| ∨ | or | disjunction, inclusive |
| ⊕ | xor | exclusive or |
| → | implies | material implication |
| ↔ | iff | biconditional |
| ⊨ | entails / models | semantic consequence |
| ⊢ | proves | syntactic consequence |
| ≡ | equivalent | logical equivalence |
| ∀ | for all | universal quantifier |
| ∃ | there exists | existential quantifier |
| X, ○ | next | true at the next state |
| F, ◊ | eventually | true at some future state |
| G, □ | always | true at every future state |
| U | until | holds until a condition becomes true |
| R | release | dual of until |
| W | weak until | until without the guarantee |
| A | for all paths | CTL path quantifier |
| E | for some path | CTL path quantifier |
| F[a,b], G[a,b] | bounded operators | STL operators with a time interval |

---

## References

Alpern, B. and Schneider, F. B. (1985). Defining liveness. Information Processing Letters, 21(4), 181–185.

Boole, G. (1854). An Investigation of the Laws of Thought, on Which are Founded the Mathematical Theories of Logic and Probabilities. Walton and Maberly, London.

Church, A. (1936). A note on the Entscheidungsproblem. Journal of Symbolic Logic, 1(1), 40–41.

Clarke, E. M. and Emerson, E. A. (1981). Design and synthesis of synchronization skeletons using branching-time temporal logic. In Logics of Programs, Lecture Notes in Computer Science, vol. 131, pp. 52–71. Springer.

Cook, S. A. (1971). The complexity of theorem-proving procedures. In Proceedings of the 3rd Annual ACM Symposium on Theory of Computing (STOC), pp. 151–158.

De Morgan, A. (1847). Formal Logic: or, The Calculus of Inference, Necessary and Probable. Taylor and Walton, London.

Donzé, A. and Maler, O. (2010). Robust satisfaction of temporal logic over real-valued signals. In Formal Modeling and Analysis of Timed Systems (FORMATS), Lecture Notes in Computer Science, vol. 6246, pp. 92–106. Springer.

Fainekos, G. E. and Pappas, G. J. (2009). Robustness of temporal logic specifications for continuous-time signals. Theoretical Computer Science, 410(42), 4262–4291.

Gödel, K. (1930). Die Vollständigkeit der Axiome des logischen Funktionenkalküls. Monatshefte für Mathematik und Physik, 37, 349–360.

Huth, M. and Ryan, M. (2004). Logic in Computer Science: Modelling and Reasoning about Systems, 2nd edition. Cambridge University Press.

Lamport, L. (1977). Proving the correctness of multiprocess programs. IEEE Transactions on Software Engineering, SE-3(2), 125–143.

Maler, O. and Nickovic, D. (2004). Monitoring temporal properties of continuous signals. In Formal Techniques, Modelling and Analysis of Timed and Fault-Tolerant Systems (FORMATS/FTRTFT), Lecture Notes in Computer Science, vol. 3253, pp. 152–166. Springer.

Pnueli, A. (1977). The temporal logic of programs. In Proceedings of the 18th Annual Symposium on Foundations of Computer Science (FOCS), pp. 46–57. IEEE.

Turing, A. M. (1936). On computable numbers, with an application to the Entscheidungsproblem. Proceedings of the London Mathematical Society, s2-42, 230–265.

For further depth on model checking and the temporal logics in the second half, see Baier, C. and Katoen, J.-P. (2008). Principles of Model Checking. MIT Press.

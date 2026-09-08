# Introduction to Logic

## Part 1: Boolean logic, propositional logic, and predicate logic

This is the first of three parts. Part 1 builds the classical foundation: two-valued Boolean algebra, propositional logic with its notions of validity and proof, and the predicate-and-quantifier machinery that lets a formula talk about individual objects. Part 2 covers first-order logic in full. Part 3 covers temporal logic (LTL, CTL, and STL) for reasoning about systems that change over time.

Each section has worked problems in the body, marked as **Worked problem**, with the solution immediately following so you can check your reasoning against it. Each section ends with an exercise set for you to do on your own. A consolidated problem set and an answer key for the exercise sets are at the end of the document.

All examples are drawn from small uncrewed aircraft operations, since that is the domain the rest of the course lives in.

---

## 1. Why logic

### 1.1 The one question

Logic is the study of what follows from what. Given some statements you accept, which further statements are you committed to? Every system in this tutorial is an answer to that question for a particular kind of subject matter. Boolean algebra answers it for raw two-valued quantities. Propositional logic answers it for declarative sentences. Predicate logic answers it for statements about objects and their properties. Temporal logic, in Part 3, answers it for statements about how a system evolves over time.

The reason an engineering course spends two days on this is that the question is not academic. A flight controller is, at bottom, a machine that decides what follows from its sensor readings and its mission rules. Boole's algebra (Boole, 1854) became the design language of digital hardware when Shannon showed in his master's thesis that relay circuits obey exactly its laws (Shannon, 1938). Hoare's axiomatic semantics turned program correctness into a question of logical entailment (Hoare, 1969). Model checking, which we reach in Part 3, mechanically decides whether a system model entails a specification (Clarke, Grumberg, and Peled, 1999). In each case the same move is made: state the requirement as a formula, state the system as a formula, and ask whether the second entails the first.

### 1.2 Syntax, semantics, and proof

Three ideas run through every section and it helps to name them now.

Syntax is the set of rules for writing a formula. It says which strings of symbols are well formed and which are not. Syntax has no opinion about truth.

Semantics assigns meaning. For the logics in this part, semantics is a rule for computing a truth value once you know the truth values of the pieces. Semantics is where entailment lives: A entails B when every way of making A true also makes B true.

Proof is a purely syntactic activity. A proof system is a set of rules for producing new lines from old ones by pattern matching, without ever consulting meaning. The reason proof matters is that a machine can do it. The deep results of the subject, soundness and completeness, say that for classical logic the syntactic game and the semantic notion line up exactly (Post, 1921, for propositional logic; Gödel, 1930, for first-order logic).

Keep these three separate in your head. Most confusion in a first logic course comes from mixing them.

### 1.3 A running example

Throughout Part 1 we will keep returning to a fixed set of statements about a quadrotor:

- armed: the flight controller has been armed
- airborne: the aircraft has left the ground
- gps_fix: the GPS has a valid position fix
- in_geofence: the current position is inside the mission geofence
- low_batt: the battery is below the return threshold
- rtl: the aircraft is in return-to-launch mode

Each of these is either true or false at any given moment. In section 2 we treat them as Boolean variables. In section 3 they become propositions. In section 4 we split them open so we can say things like "every aircraft in the fleet has a GPS fix."

**Worked problem 1.1.** A field procedure says: "Do not arm the aircraft unless the GPS has a fix and the position is inside the geofence." Which of the six statements above does this rule mention, and is the rule a statement about a single moment or about the whole flight?

*Solution.* It mentions armed, gps_fix, and in_geofence. As written it is a rule about the moment of arming, so it can be checked from a single snapshot of the three values. That makes it a candidate for propositional logic. If the procedure had said "the aircraft must never be armed without a fix," that "never" would push it into temporal logic, because checking it requires looking at every moment of the flight.

### 1.4 Exercises

1. Rewrite the following as a claim of the form "given these premises, this conclusion must follow": "The aircraft is armed and airborne, and armed aircraft that are airborne must have a GPS fix, so the aircraft has a fix."
2. For each of the following, say whether checking it needs one snapshot of the system or the whole history: (a) the battery is low; (b) the battery has been low for ten seconds; (c) the aircraft is in the geofence; (d) the aircraft never left the geofence.
3. Explain in one or two sentences why a proof system that could derive a false conclusion from true premises would be useless, and which of the words syntax, semantics, or proof each half of your sentence is about.

---

## 2. Boolean logic and Boolean algebra

Boolean algebra works on two values. We write them 1 and 0, or true and false, or T and F, depending on context; the choice of symbols changes nothing. Boole introduced the algebra in two books (Boole, 1847; Boole, 1854). The modern axiomatic presentation, in which the algebra is characterized by a short list of laws, is due to Huntington (1904). The realization that switching circuits obey those laws, so that a circuit can be designed by algebra and simplified by algebra, is Shannon's (1938).

### 2.1 Variables and expressions

A Boolean variable is a name that stands for 0 or 1. We use p, q, r, and the named variables from section 1.3. A Boolean expression is any combination of variables, constants, and operations. Once you fix a value for every variable the expression evaluates to a single 0 or 1.

### 2.2 The three primitive operations

Everything in Boolean algebra is built from three operations.

NOT, written ¬p (also p̄, !p, or ~p), flips the value.

AND, written p ∧ q (also p · q, pq, or p && q), is 1 exactly when both inputs are 1.

OR, written p ∨ q (also p + q or p || q), is 1 when at least one input is 1. This is inclusive or. It is 1 when both inputs are 1. Everyday English usually means exclusive or, so watch for that.

A truth table enumerates every input combination and the output for each. The three primitives together:

| p | q | ¬p | p ∧ q | p ∨ q |
|---|---|----|-------|-------|
| 0 | 0 | 1  | 0     | 0     |
| 0 | 1 | 1  | 0     | 1     |
| 1 | 0 | 0  | 0     | 1     |
| 1 | 1 | 0  | 1     | 1     |

With n variables a truth table has 2^n rows. The convention is to list rows in binary counting order so that nothing is skipped.

**Worked problem 2.1.** A motor interlock allows the propellers to spin only when the aircraft is armed and the GPS has a fix. Write the interlock as a Boolean expression and give its truth table.

*Solution.* spin = armed ∧ gps_fix.

| armed | gps_fix | spin |
|-------|---------|------|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

The propellers can spin in exactly one of the four possible states, which is what an interlock should look like.

### 2.3 Derived operations

Four further operations appear constantly. Each is definable from the primitives.

XOR (exclusive or), written p ⊕ q, is 1 when the inputs differ. It equals (p ∨ q) ∧ ¬(p ∧ q), and also (p ∧ ¬q) ∨ (¬p ∧ q).

NAND, written p ↑ q or ¬(p ∧ q), is 0 only when both inputs are 1. Sheffer (1913) showed that NAND alone suffices to define every Boolean operation, which is why NAND gates are the workhorse of digital hardware.

NOR, written p ↓ q or ¬(p ∨ q), is 1 only when both inputs are 0. It has the same universal property as NAND.

Implication, written p → q, is 0 only when p is 1 and q is 0. It gets its own subsection below because it is the operation most often misread.

| p | q | p ⊕ q | p ↑ q | p ↓ q | p → q |
|---|---|-------|-------|-------|-------|
| 0 | 0 | 0 | 1 | 1 | 1 |
| 0 | 1 | 1 | 1 | 0 | 1 |
| 1 | 0 | 1 | 1 | 0 | 0 |
| 1 | 1 | 0 | 0 | 0 | 1 |

**Worked problem 2.2.** Show that ¬p can be written using only NAND.

*Solution.* p ↑ p = ¬(p ∧ p) = ¬p, using the idempotent law p ∧ p = p from section 2.5. Check with the table: when p = 0, p ↑ p = 1; when p = 1, p ↑ p = 0. That is the NOT column.

**Worked problem 2.3.** A control mode selector has two inputs, manual and auto. Exactly one of them should be engaged at any time. Write the "healthy" condition, and write the "fault" condition as its negation.

*Solution.* healthy = manual ⊕ auto. fault = ¬(manual ⊕ auto), which is 1 when both are engaged or neither is. That fault expression is also the biconditional manual ↔ auto, defined in the next subsection.

### 2.4 Reading implication correctly

p → q is false in one case only: p true and q false. In particular, when p is false the implication is true no matter what q is. Logicians call this vacuous truth, and it surprises everyone the first time.

The reading that makes it natural is to treat p → q as a promise: if p happens, q will happen. The promise is broken only when p happens and q does not. If p never happens, the promise has not been broken. The rule "armed → gps_fix" is not violated by a disarmed aircraft with no fix, because the rule's condition never triggered.

Three related facts are worth memorizing. First, p → q is equivalent to ¬p ∨ q; you will verify this in the exercises. Second, p → q is equivalent to its contrapositive ¬q → ¬p, but not to its converse q → p. Third, the biconditional p ↔ q, read "p if and only if q," is 1 when p and q have the same value, and equals (p → q) ∧ (q → p).

**Worked problem 2.4.** The rule is "if low_batt then rtl." At a given moment low_batt = 0 and rtl = 1. Is the rule satisfied? What about low_batt = 1 and rtl = 0?

*Solution.* First case: the antecedent is false, so low_batt → rtl is true. The aircraft is returning to launch for some other reason, which the rule does not forbid. Second case: antecedent true, consequent false. The rule is violated. This is the only combination that violates it.

### 2.5 The laws of Boolean algebra

The laws below let you transform one expression into another that always has the same value. They are the tools for simplification and for proving equivalence without writing out a table. Every one of them can be verified by a truth table, which is a good exercise the first time you meet them.

| Law | AND form | OR form |
|-----|----------|---------|
| Identity | p ∧ 1 = p | p ∨ 0 = p |
| Domination | p ∧ 0 = 0 | p ∨ 1 = 1 |
| Idempotence | p ∧ p = p | p ∨ p = p |
| Complement | p ∧ ¬p = 0 | p ∨ ¬p = 1 |
| Double negation | ¬¬p = p | |
| Commutativity | p ∧ q = q ∧ p | p ∨ q = q ∨ p |
| Associativity | (p ∧ q) ∧ r = p ∧ (q ∧ r) | (p ∨ q) ∨ r = p ∨ (q ∨ r) |
| Distributivity | p ∧ (q ∨ r) = (p ∧ q) ∨ (p ∧ r) | p ∨ (q ∧ r) = (p ∨ q) ∧ (p ∨ r) |
| Absorption | p ∧ (p ∨ q) = p | p ∨ (p ∧ q) = p |
| De Morgan | ¬(p ∧ q) = ¬p ∨ ¬q | ¬(p ∨ q) = ¬p ∧ ¬q |

Notice the pattern: every law comes in a pair obtained by swapping ∧ with ∨ and 0 with 1. This is the duality principle, and it is why Huntington's axiomatization is so compact.

De Morgan's laws (De Morgan, 1847) deserve special attention. They say that a negation can be pushed inward through AND or OR, at the cost of flipping the operator. "It is not the case that the aircraft is armed and airborne" is the same as "the aircraft is not armed or it is not airborne."

**Worked problem 2.5.** Simplify (p ∧ q) ∨ (p ∧ ¬q) to a single variable, naming the law at each step.

*Solution.*
(p ∧ q) ∨ (p ∧ ¬q)
= p ∧ (q ∨ ¬q) by distributivity, factoring p out
= p ∧ 1 by the complement law
= p by identity.

The expression says "p and q, or p and not q," and whichever value q takes, one of the two branches is just p.

**Worked problem 2.6.** A safety monitor should raise an alarm when it is *not* true that the aircraft is in the geofence with a GPS fix. Write the alarm condition and push the negation all the way in.

*Solution.* alarm = ¬(in_geofence ∧ gps_fix) = ¬in_geofence ∨ ¬gps_fix by De Morgan. The alarm fires if the aircraft is outside the fence, or if it has lost its fix, or both. The De Morgan form is what you would actually implement, because it lists the two independent causes.

### 2.6 Simplification and canonical forms

Two facts about simplification are worth knowing even at this stage.

First, any Boolean function can be written as a sum of products: an OR of terms, each term an AND of variables or negated variables. Read the 1 rows of a truth table, write one term per row, and OR them together. This is the disjunctive normal form we return to in section 3.7.

Second, finding the *simplest* expression for a function is a real problem with real algorithms. Karnaugh maps (Karnaugh, 1953) handle up to about five variables by hand. The Quine–McCluskey procedure (Quine, 1952; McCluskey, 1956) is the tabular version that a program can run. For this course you only need the algebraic laws, but you should know the tools exist, because a flight-controller interlock with eight inputs is not something to simplify by inspection.

**Worked problem 2.7.** From the truth table below, write the function as a sum of products and then simplify it.

| p | q | f |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

*Solution.* The 1 rows are (0,0), (0,1), and (1,1). Sum of products: (¬p ∧ ¬q) ∨ (¬p ∧ q) ∨ (p ∧ q). The first two terms factor to ¬p ∧ (¬q ∨ q) = ¬p. So f = ¬p ∨ (p ∧ q). By absorption's cousin (¬p ∨ (p ∧ q) = (¬p ∨ p) ∧ (¬p ∨ q) by distributivity, and ¬p ∨ p = 1), f = ¬p ∨ q. That is exactly the truth table of p → q from section 2.3.

### 2.7 Operator reference

| Operator | Symbol | Value | Example |
|----------|--------|-------|---------|
| NOT | ¬p | flips p | ¬armed is 1 exactly when the system is disarmed |
| AND | p ∧ q | 1 only when both are 1 | armed ∧ airborne: armed and in the air |
| OR | p ∨ q | 1 when at least one is 1 | low_batt ∨ ¬gps_fix: either warning is active |
| XOR | p ⊕ q | 1 when exactly one is 1 | manual ⊕ auto: exactly one mode engaged |
| NAND | p ↑ q | 0 only when both are 1 | sufficient alone to build every other gate |
| NOR | p ↓ q | 1 only when both are 0 | idle = armed ↓ airborne: neither armed nor airborne |
| Implication | p → q | 0 only when p = 1 and q = 0 | armed → gps_fix: arming requires a fix |
| Biconditional | p ↔ q | 1 when both sides agree | gear_down ↔ landing_mode |

### 2.8 Exercises

1. Build the full truth table for (p ∧ q) ∨ ¬r. It has eight rows.
2. Use De Morgan's laws to rewrite ¬(p ∨ ¬q) with no negation on the outside.
3. Show with a truth table that p → q and ¬p ∨ q have the same column.
4. Show with a truth table that p → q and q → p do *not* have the same column, and identify the rows where they differ.
5. XOR was written two ways in section 2.3. Prove they are equal using the laws, not a table.
6. Write OR using only NAND. (Hint: apply De Morgan to p ∨ q and then use worked problem 2.2.)
7. Simplify (p ∨ q) ∧ (p ∨ ¬q) to a single variable, naming each law.
8. A landing interlock should permit landing when the aircraft has a GPS fix, is inside the geofence, and is not in return-to-launch mode, *or* when the battery is low regardless of anything else. Write the permit condition, then write its negation with all negations pushed onto the variables.

---

## 3. Propositional logic

Propositional logic is Boolean algebra with the values reinterpreted as the truth or falsity of declarative sentences. The mathematics is the same. What changes is the vocabulary and the questions asked: not "what does this circuit compute" but "does this conclusion follow from these premises." The truth-table method as a decision procedure is due to Post (1921) and, independently, Wittgenstein (1921). Its textbook presentation follows Enderton (2001) and Huth and Ryan (2004); a free and thorough introduction is Magnus et al. (2023).

### 3.1 Propositions

A proposition is a declarative sentence that is definitely true or definitely false. "The airport is closed" is a proposition. "Close the runway" is not, because a command has no truth value. "x is above 120 metres" is not one either, because its truth depends on what x is. Section 4 fixes that.

An atomic proposition cannot be broken into smaller propositions. We assign each a letter or a name. A compound proposition is built from atomic ones with connectives. The connectives are the Boolean operations under their logical names: negation (¬), conjunction (∧), disjunction (∨), implication (→), and biconditional (↔).

**Worked problem 3.1.** Classify each as a proposition or not: (a) "The aircraft is armed." (b) "Arm the aircraft." (c) "Is the aircraft armed?" (d) "The aircraft is armed and it is not armed."

*Solution.* (a) is a proposition. (b) is a command. (c) is a question. (d) is a proposition; it happens to be always false, but always false is still a definite truth value.

### 3.2 Well-formed formulas

Not every string of symbols means anything. A well-formed formula (wff) is one built by the following grammar and no other way:

1. Every atomic proposition is a wff.
2. If A is a wff, so is ¬A.
3. If A and B are wffs, so are (A ∧ B), (A ∨ B), (A → B), and (A ↔ B).

This is a recursive definition, and the recursion is what lets us prove things about all formulas at once (by induction on the structure of the formula).

Fully parenthesized formulas are unambiguous but cluttered, so we adopt precedence: ¬ binds tightest, then ∧, then ∨, then →, then ↔. Under these rules p ∨ q ∧ r means p ∨ (q ∧ r), and ¬p → q means (¬p) → q. Implication is taken to associate to the right, so p → q → r means p → (q → r). When in doubt, add parentheses.

**Worked problem 3.2.** Add full parentheses to ¬p ∧ q → r ∨ ¬s.

*Solution.* Negations first: (¬p) ∧ q → r ∨ (¬s). Then ∧: ((¬p) ∧ q) → r ∨ (¬s). Then ∨: ((¬p) ∧ q) → (r ∨ (¬s)). The main connective is →.

### 3.3 Valuations

To evaluate a formula, assign a truth value to each atomic proposition and work outward using the tables. An assignment of values to all the atoms is called a valuation (or an interpretation, or a truth assignment). A formula with n distinct atoms has 2^n valuations.

**Worked problem 3.3.** Evaluate (armed → gps_fix) ∧ (¬gps_fix → ¬airborne) under the valuation armed = 1, gps_fix = 0, airborne = 0.

*Solution.* armed → gps_fix is 1 → 0, which is 0. The whole conjunction is therefore 0 regardless of the other conjunct. (For the record, ¬gps_fix → ¬airborne is 1 → 1, which is 1.) The rule set is violated because the aircraft is armed without a fix.

### 3.4 Tautology, contradiction, contingency

A formula that is true under every valuation is a tautology. p ∨ ¬p is the classic example (the law of excluded middle). A formula false under every valuation is a contradiction; p ∧ ¬p is the classic. A formula that is true under some valuations and false under others is a contingency. Most formulas you write about real systems are contingencies, because they say something that could go either way.

A formula that is true under at least one valuation is satisfiable. Every tautology and every contingency is satisfiable; a contradiction is not. Deciding satisfiability is the SAT problem, and Cook (1971) proved it NP-complete, which is why SAT solvers are engineered with such care and why a specification with hundreds of atoms cannot be checked by truth table.

**Worked problem 3.4.** Classify (p → q) ∨ (q → p).

*Solution.* Try to make it false. Both disjuncts would have to be false. p → q is false only when p = 1, q = 0. q → p is false only when q = 1, p = 0. These cannot both hold. So there is no falsifying valuation and the formula is a tautology. (It is a genuinely odd one: for any two propositions, one implies the other. That is a consequence of the material reading of implication in section 2.4.)

### 3.5 Logical equivalence

Two formulas A and B are logically equivalent, written A ≡ B, when they have the same truth value under every valuation. Equivalently, A ≡ B holds exactly when A ↔ B is a tautology. All the laws of section 2.5 are equivalences and carry over unchanged. Three more, involving implication, are used constantly:

- p → q ≡ ¬p ∨ q (material implication)
- p → q ≡ ¬q → ¬p (contraposition)
- ¬(p → q) ≡ p ∧ ¬q (negated implication)

**Worked problem 3.5.** A pilot's manual states rule R1: "if the battery is low, the aircraft returns to launch." A reviewer rewrites it as R2: "if the aircraft is not returning to launch, the battery is not low." Are R1 and R2 equivalent?

*Solution.* R1 is low_batt → rtl. R2 is ¬rtl → ¬low_batt. That is the contrapositive of R1, so yes, they are equivalent. Had the reviewer written "if the aircraft is returning to launch, the battery is low," that would be the converse rtl → low_batt, which is a different and stronger claim (it rules out returning to launch for any other reason).

### 3.6 Entailment and validity

A set of premises Γ entails a conclusion C, written Γ ⊨ C, when every valuation that makes all of Γ true also makes C true. This is the formal version of the question from section 1.

An argument is valid when its premises entail its conclusion. Validity is about form. An argument can be valid with false premises, and an argument with true premises and a true conclusion can be invalid because the conclusion does not follow. To show an argument invalid, exhibit one valuation that makes the premises true and the conclusion false. That valuation is a countermodel.

There is a useful reduction: Γ ⊨ C holds exactly when the formula (Γ₁ ∧ … ∧ Γₙ) → C is a tautology, and also exactly when Γ₁ ∧ … ∧ Γₙ ∧ ¬C is unsatisfiable. The second form is how SAT solvers check entailment: negate the conclusion, add it to the premises, and look for a satisfying valuation. If there is none, the entailment holds.

**Worked problem 3.6.** Decide whether {armed → gps_fix, gps_fix → in_geofence, armed} ⊨ in_geofence.

*Solution.* Suppose all three premises are true. From armed and armed → gps_fix, gps_fix is true. From gps_fix and gps_fix → in_geofence, in_geofence is true. So every valuation satisfying the premises satisfies the conclusion, and the entailment holds. (You could also do the eight-row table for the three atoms and check that in every row where the premises are all 1, the conclusion is 1.)

**Worked problem 3.7.** Decide whether {armed → gps_fix, gps_fix} ⊨ armed.

*Solution.* Look for a countermodel: premises true, conclusion false. Set armed = 0. Then armed → gps_fix is true regardless. Set gps_fix = 1 to make the second premise true. The conclusion armed is 0. So the valuation armed = 0, gps_fix = 1 is a countermodel and the entailment fails. This pattern, reasoning from a consequent back to its antecedent, is the fallacy of affirming the consequent.

### 3.7 Normal forms

It is often convenient to force formulas into a standard shape. A literal is an atom or a negated atom. A clause is a disjunction of literals.

A formula is in conjunctive normal form (CNF) when it is a conjunction of clauses. A formula is in disjunctive normal form (DNF) when it is a disjunction of terms, each term a conjunction of literals. Every formula can be converted to either form: eliminate → and ↔ using the equivalences of section 3.5, push negations inward with De Morgan and double negation, then distribute.

CNF is the input format for SAT solvers. The classical complete procedures are Davis and Putnam (1960) and its refinement DPLL (Davis, Logemann, and Loveland, 1962), which remains the backbone of modern solvers. Naive conversion to CNF can blow up exponentially; Tseitin's transformation (Tseitin, 1968) avoids this by introducing fresh atoms for subformulas, at the cost of producing an equisatisfiable rather than equivalent formula.

**Worked problem 3.8.** Convert (p ∧ q) ∨ ¬r to CNF.

*Solution.* Distribute ∨ over ∧: (p ∨ ¬r) ∧ (q ∨ ¬r). Two clauses, each a disjunction of literals. Done.

**Worked problem 3.9.** Convert ¬(p → q) ∨ r to DNF.

*Solution.* ¬(p → q) ≡ p ∧ ¬q by negated implication. So the formula is (p ∧ ¬q) ∨ r. That is already a disjunction of terms (r is a one-literal term). Done.

### 3.8 Inference rules and proof

Truth tables decide entailment by brute force, and the brute force doubles with every atom. Proof systems derive conclusions step by step instead. When C is derivable from Γ by the rules we write Γ ⊢ C.

The rules below are the ones you will use most. Each is itself a valid argument form, which is what makes the system sound.

| Rule | From | Infer |
|------|------|-------|
| Modus ponens | A, A → B | B |
| Modus tollens | ¬B, A → B | ¬A |
| Hypothetical syllogism | A → B, B → C | A → C |
| Disjunctive syllogism | A ∨ B, ¬A | B |
| Conjunction introduction | A, B | A ∧ B |
| Conjunction elimination | A ∧ B | A (or B) |
| Disjunction introduction | A | A ∨ B |
| Contraposition | A → B | ¬B → ¬A |

Natural deduction (Gentzen, 1935) organizes these into a system in which you can also make temporary assumptions and discharge them; that is how you prove an implication A → B (assume A, derive B, discharge). Magnus et al. (2023) give a complete Fitch-style system with worked derivations, and the Open Logic Project's Carnap tool checks derivations in that notation.

**Worked problem 3.10.** From premises armed → gps_fix, gps_fix → in_geofence, and ¬in_geofence, derive ¬armed.

*Solution.*
1. armed → gps_fix (premise)
2. gps_fix → in_geofence (premise)
3. ¬in_geofence (premise)
4. ¬gps_fix (modus tollens, 2, 3)
5. ¬armed (modus tollens, 1, 4)

Alternatively: 4′. armed → in_geofence (hypothetical syllogism, 1, 2); 5′. ¬armed (modus tollens, 4′, 3). Both derivations are correct. Proofs are not unique.

**Worked problem 3.11.** From low_batt ∨ ¬gps_fix and gps_fix, derive rtl given the additional premise low_batt → rtl.

*Solution.*
1. low_batt ∨ ¬gps_fix (premise)
2. gps_fix (premise)
3. low_batt → rtl (premise)
4. ¬¬gps_fix (double negation, 2)
5. low_batt (disjunctive syllogism, 1, 4)
6. rtl (modus ponens, 3, 5)

### 3.9 Soundness and completeness

A proof system is sound when everything it proves is entailed: if Γ ⊢ C then Γ ⊨ C. It is complete when everything entailed is provable: if Γ ⊨ C then Γ ⊢ C. Propositional logic with the standard rules is both, a result first established by Post (1921). Soundness is what lets you trust a proof; completeness is what tells you that if a proof does not exist, the entailment genuinely fails, so a countermodel exists.

Soundness and completeness are properties of a proof system relative to a semantics. They are not free. Second-order logic, mentioned in Part 2, is an example of a logic with no sound and complete proof system.

### 3.10 Reference tables

Connectives, read in English:

| Connective | Symbol | Read as | Example |
|------------|--------|---------|---------|
| Negation | ¬P | not P | ¬armed: the aircraft is not armed |
| Conjunction | P ∧ Q | P and Q | gps_fix ∧ in_geofence |
| Disjunction | P ∨ Q | P or Q (or both) | low_batt ∨ rtl |
| Implication | P → Q | if P then Q | armed → gps_fix |
| Biconditional | P ↔ Q | P if and only if Q | rtl ↔ low_batt |

Semantic and syntactic relations:

| Symbol | Name | Meaning |
|--------|------|---------|
| ⊨ | entails | every valuation satisfying the left satisfies the right |
| ⊢ | proves | the right is derivable from the left by the rules |
| ≡ | equivalent | same truth value under every valuation |

### 3.11 Exercises

1. Show that ((p → q) ∧ (q → r)) → (p → r) is a tautology, either by table or by attempting to falsify it.
2. Convert (p ∧ q) ∨ (¬p ∧ r) into CNF.
3. Convert p → (q ↔ r) into DNF.
4. Decide whether {p → q, q → r, p} ⊨ r and justify your answer.
5. Decide whether {p → q, ¬p} ⊨ ¬q. If not, give a countermodel and name the fallacy.
6. Give an argument with a false conclusion that is nonetheless valid. (It must have at least one false premise. Explain why.)
7. From the premises armed → gps_fix, gps_fix → in_geofence, and ¬in_geofence, derive ¬armed using only modus tollens twice. Then do it using hypothetical syllogism once and modus tollens once.
8. A checklist says: "If the aircraft is airborne, it is armed. If it is armed, it has a fix. The aircraft has no fix." What can you conclude about whether it is airborne? Formalize and derive.
9. Explain why "Γ ⊨ C exactly when Γ ∧ ¬C is unsatisfiable" is true, in a sentence or two.

---

## 4. Predicates and quantifiers

Propositional logic treats "Earhart is airborne" as a single indivisible atom. It cannot see that the sentence is about an object (the aircraft Earhart) and a property (being airborne). It therefore cannot express "every aircraft in the fleet is airworthy," because that sentence is about all objects at once, and there is no atom for "every."

Predicates and quantifiers fix this. Quantifier notation was introduced by Frege (1879) and, independently and in a form closer to modern practice, by Peirce (1885). This section introduces the ideas; Part 2 develops the full first-order system with its semantics and proof theory.

### 4.1 Predicates

A predicate is a statement with one or more blanks. Fill the blanks with objects and you get a proposition. Formally a predicate is a function from objects to truth values.

Write Airborne(x) for "x is airborne." Alone it has no truth value, because x is a placeholder. Airborne(Earhart) is a proposition.

The number of blanks is the arity. Airborne(x) is unary. HigherThan(x, y), "x is at a higher altitude than y," is binary. Between(x, y, z) is ternary. A unary predicate expresses a property; a predicate of higher arity expresses a relation.

**Worked problem 4.1.** Identify the predicate and its arity in each: (a) "the aircraft has a GPS fix"; (b) "aircraft A is within 50 metres of aircraft B"; (c) "waypoint W lies on the route from A to B."

*Solution.* (a) HasFix(x), arity 1. (b) Within50(x, y), arity 2. (c) OnRoute(w, a, b), arity 3. Notice that in (b) the order of arguments does not matter for this particular relation (it is symmetric), while in HigherThan(x, y) it does.

### 4.2 Domain, constants, variables

A predicate ranges over a domain of discourse, the set of objects under discussion. If the domain is the six aircraft in a fleet, then x ranges over those six and nothing else. Choosing the domain is part of setting up the problem. The same predicate can mean different things over different domains, and a statement true over one domain can be false over another.

A constant names a specific object in the domain: Earhart, Lindbergh. A variable, x or y, is a placeholder for an unspecified object. The distinction looks trivial now and becomes central as soon as quantifiers appear.

### 4.3 The two quantifiers

The universal quantifier ∀, read "for all," says the predicate holds for every object in the domain. ∀x Airworthy(x) says every aircraft in the fleet is airworthy. A universal claim is refuted by a single object that fails the predicate; that object is a counterexample.

The existential quantifier ∃, read "there exists," says the predicate holds for at least one object. ∃x Airborne(x) says at least one aircraft is airborne. An existential claim is confirmed by a single object that satisfies the predicate; that object is a witness.

Over a finite domain {a₁, …, aₙ}, the quantifiers are just big conjunctions and disjunctions:

- ∀x P(x) is P(a₁) ∧ P(a₂) ∧ … ∧ P(aₙ)
- ∃x P(x) is P(a₁) ∨ P(a₂) ∨ … ∨ P(aₙ)

That view is worth holding onto. It explains why the quantifier laws below look like De Morgan's laws, and it is the reason a small finite domain can be checked exhaustively while an infinite one cannot.

**Worked problem 4.2.** The fleet is {Earhart, Lindbergh, Yeager}. Airborne is true of Earhart only. Airworthy is true of all three. Evaluate ∀x Airworthy(x), ∃x Airborne(x), ∀x Airborne(x), and ∃x ¬Airworthy(x).

*Solution.* ∀x Airworthy(x): true, all three satisfy it. ∃x Airborne(x): true, Earhart is a witness. ∀x Airborne(x): false, Lindbergh is a counterexample. ∃x ¬Airworthy(x): false, no aircraft fails airworthiness.

### 4.4 Translating restricted quantifiers

Most useful statements are not about every object in the domain but about every object of some kind. "Every airborne aircraft has a fix" quantifies over the airborne ones only. The two quantifiers handle this differently, and getting the connective right is the single most common error in symbolization.

A universal restricted to a class uses implication:

∀x (Airborne(x) → HasFix(x)): for every x, if x is airborne then x has a fix.

An existential restricted to a class uses conjunction:

∃x (Airborne(x) ∧ ¬HasFix(x)): there is an x that is airborne and lacks a fix.

Why the difference? Consider the wrong pairing. ∀x (Airborne(x) ∧ HasFix(x)) says every aircraft is airborne and has a fix, which is far too strong; a single grounded aircraft refutes it. And ∃x (Airborne(x) → HasFix(x)) is nearly empty: by vacuous truth it is satisfied by any grounded aircraft whatsoever, so it says almost nothing. The mnemonic is: ∀ goes with →, ∃ goes with ∧.

**Worked problem 4.3.** Symbolize: (a) "No armed aircraft is outside the geofence." (b) "Some aircraft with a low battery is not returning to launch."

*Solution.* (a) There are two equally good forms. ∀x (Armed(x) → InGeofence(x)), "every armed aircraft is in the fence." Or ¬∃x (Armed(x) ∧ ¬InGeofence(x)), "there is no armed aircraft outside the fence." Section 4.5 shows these are equivalent. (b) ∃x (LowBatt(x) ∧ ¬RTL(x)).

### 4.5 Negating quantifiers

Negation interacts with quantifiers exactly the way it interacts with ∧ and ∨:

- ¬∀x P(x) ≡ ∃x ¬P(x)
- ¬∃x P(x) ≡ ∀x ¬P(x)

Read the first: "not everything is P" means "something is not P." Read the second: "nothing is P" means "everything is not P." These are the quantifier De Morgan laws, and the finite-domain view of section 4.3 shows why: negating a big conjunction gives a big disjunction of negations.

Pushing a negation inward through a restricted quantifier uses both the quantifier law and the propositional laws:

¬∀x (A(x) → B(x)) ≡ ∃x ¬(A(x) → B(x)) ≡ ∃x (A(x) ∧ ¬B(x)).

In words: "it is not true that every A is a B" means "some A is not a B." That is exactly the form of a counterexample.

**Worked problem 4.4.** A requirement reads ∀x (Airborne(x) → HasFix(x)). Write, in symbols and in English, what a violation of the requirement looks like.

*Solution.* A violation is ¬∀x (Airborne(x) → HasFix(x)), which simplifies to ∃x (Airborne(x) ∧ ¬HasFix(x)): some aircraft is airborne without a fix. That is the thing a monitor would look for. Note the practical point: to *check* a universal requirement, you search for an existential violation.

**Worked problem 4.5.** Push the negation inward in ¬∃x (LowBatt(x) ∧ ¬RTL(x)).

*Solution.* ¬∃x (…) ≡ ∀x ¬(LowBatt(x) ∧ ¬RTL(x)) ≡ ∀x (¬LowBatt(x) ∨ RTL(x)) by De Morgan ≡ ∀x (LowBatt(x) → RTL(x)) by material implication. "There is no low-battery aircraft that is not returning" is the same as "every low-battery aircraft is returning."

### 4.6 A first look at nested quantifiers

Several quantifiers can appear in one formula, and their order matters. Let CanReach(x, y) mean aircraft x can reach waypoint y, with a domain containing both aircraft and waypoints.

∀y ∃x CanReach(x, y): for every waypoint there is some aircraft that can reach it. The aircraft may differ from waypoint to waypoint.

∃x ∀y CanReach(x, y): there is one aircraft that can reach every waypoint. Much stronger.

The second implies the first but not the reverse. Read nested quantifiers left to right, and think of each quantifier as being chosen before the ones to its right. Part 2 returns to this with full semantics, because misordered quantifiers are a real source of specification errors.

**Worked problem 4.6.** Domain: aircraft {E, L} and waypoints {W1, W2}. E can reach W1 only; L can reach W2 only. Evaluate ∀y ∃x CanReach(x, y) and ∃x ∀y CanReach(x, y).

*Solution.* First formula: for W1 pick E, for W2 pick L. Every waypoint has a reacher, so true. Second formula: is there a single aircraft that reaches both? E fails at W2, L fails at W1. False. This is the standard example showing ∀∃ does not imply ∃∀.

### 4.7 Operator reference

| Operator | Symbol | Meaning | Example |
|----------|--------|---------|---------|
| Universal | ∀x P(x) | P holds of every object in the domain | ∀x Airworthy(x) |
| Existential | ∃x P(x) | P holds of at least one object | ∃x Airborne(x) |
| Restricted universal | ∀x (A(x) → B(x)) | every A is a B | ∀x (Armed(x) → InGeofence(x)) |
| Restricted existential | ∃x (A(x) ∧ B(x)) | some A is a B | ∃x (Airborne(x) ∧ ¬HasFix(x)) |
| Negated universal | ¬∀x P(x) ≡ ∃x ¬P(x) | some object is not P | ¬∀x Airworthy(x) |
| Negated existential | ¬∃x P(x) ≡ ∀x ¬P(x) | no object is P | ¬∃x Airborne(x): all grounded |

### 4.8 Exercises

1. Domain: the fleet. Predicates Airborne(x), Airworthy(x), HasFix(x). Symbolize: (a) every airborne aircraft is airworthy; (b) some airworthy aircraft is not airborne; (c) no aircraft is airborne without a fix; (d) every aircraft is either airborne or airworthy (or both).
2. Translate into English and say why an operator should care: ∃x (Airborne(x) ∧ ¬Airworthy(x)).
3. Negate ∀x (Armed(x) → Airborne(x)) and simplify until the negations sit on atomic predicates.
4. Negate ∃x (LowBatt(x) ∧ ¬RTL(x)) and simplify. State the result in English.
5. Explain the difference between ∀x ∃y HigherThan(y, x) and ∃y ∀x HigherThan(y, x). Over a finite fleet, can the first be true? Can the second?
6. Give a domain and a unary predicate for which ∀x P(x) is false but ∃x P(x) is true.
7. A student symbolizes "every armed aircraft has a fix" as ∀x (Armed(x) ∧ HasFix(x)). Give a small fleet that makes the student's formula false while the English sentence is true.
8. With domain {E, L, Y}, write ∀x (Armed(x) → HasFix(x)) as a propositional formula with no quantifiers, using atoms Armed_E, HasFix_E, and so on.

---

## Consolidated problem set

These problems cut across sections. Some require combining ideas.

**P1.** An arming interlock has three inputs: gps_fix, in_geofence, and preflight_ok. Arming is permitted when all three are true, or when a maintenance override is set and preflight_ok is true. (a) Write the permit expression. (b) Simplify it as far as the laws allow. (c) Write the "arming refused" condition with negations pushed onto the variables. (d) How many rows does the full truth table have?

**P2.** Prove, using only the laws of section 2.5 and the implication equivalences of section 3.5, that p → (q → r) ≡ (p ∧ q) → r.

**P3.** A flight manual contains three rules. R1: if the aircraft is airborne it is armed. R2: if it is armed it has a fix. R3: if it has a fix it is in the geofence. An operator observes an aircraft outside the geofence. (a) Formalize the rules and the observation. (b) Derive, with rule names, what can be concluded about arming and about being airborne. (c) Convert the conjunction of R1, R2, R3 to CNF.

**P4.** Decide whether {p ∨ q, p → r, q → r} ⊨ r. Then decide whether {p ∨ q, p → r} ⊨ r. For whichever fails, give a countermodel.

**P5.** Fleet {E, L, Y}. Predicates Armed, Airborne, HasFix. Facts: E is armed, airborne, has a fix. L is armed, not airborne, no fix. Y is not armed, not airborne, has a fix. Evaluate: (a) ∀x (Airborne(x) → HasFix(x)); (b) ∀x (Armed(x) → HasFix(x)); (c) ∃x (HasFix(x) ∧ ¬Armed(x)); (d) ¬∃x (Airborne(x) ∧ ¬Armed(x)). For each false one, name the counterexample.

**P6.** Write the requirement "every armed aircraft that is airborne has a GPS fix and is inside the geofence" in predicate logic, then write the negation with negations pushed onto the atoms, then describe in one sentence what a monitor should search for.

**P7.** The propositional atom rtl in section 1.3 was about a single aircraft. Rewrite the rule "if the battery is low, return to launch" as a predicate-logic sentence about a fleet, and explain what ∀ adds that the propositional version lacked.

**P8.** Explain why the truth-table method cannot be used directly to check ∀x P(x) over an infinite domain, and why it can over a finite one. Connect your answer to section 4.3.

---

## Answer key for section exercises

Answers are given for the section exercises. The consolidated problem set is left for you.

### Section 1

1. Premises: armed ∧ airborne; (armed ∧ airborne) → gps_fix. Conclusion: gps_fix. The claim is that the premises entail the conclusion, which they do by modus ponens.
2. (a) snapshot; (b) history; (c) snapshot; (d) history.
3. Such a system would be unsound: a syntactic derivation (proof) would fail to respect semantic entailment, so you could never trust what it told you.

### Section 2

1. Rows in order (p, q, r): 000→1, 001→0, 010→1, 011→0, 100→1, 101→0, 110→1, 111→1.
2. ¬(p ∨ ¬q) = ¬p ∧ ¬¬q = ¬p ∧ q.
3. Both columns are 1, 1, 0, 1 for rows 00, 01, 10, 11.
4. p → q is 1, 1, 0, 1; q → p is 1, 0, 1, 1. They differ on rows 01 and 10.
5. (p ∨ q) ∧ ¬(p ∧ q) = (p ∨ q) ∧ (¬p ∨ ¬q) by De Morgan. Distribute: (p ∧ ¬p) ∨ (p ∧ ¬q) ∨ (q ∧ ¬p) ∨ (q ∧ ¬q) = 0 ∨ (p ∧ ¬q) ∨ (¬p ∧ q) ∨ 0 = (p ∧ ¬q) ∨ (¬p ∧ q).
6. p ∨ q = ¬(¬p ∧ ¬q) = (¬p) ↑ (¬q) = (p ↑ p) ↑ (q ↑ q).
7. (p ∨ q) ∧ (p ∨ ¬q) = p ∨ (q ∧ ¬q) by distributivity = p ∨ 0 = p.
8. permit = (gps_fix ∧ in_geofence ∧ ¬rtl) ∨ low_batt. Negation: (¬gps_fix ∨ ¬in_geofence ∨ rtl) ∧ ¬low_batt.

### Section 3

1. To falsify, p → r must be 0, so p = 1, r = 0. Then p → q needs q = 1, and q → r is 1 → 0 = 0. The antecedent fails, so the whole implication is 1. No falsifying valuation exists; tautology.
2. Distribute: (p ∨ ¬p) ∧ (p ∨ r) ∧ (q ∨ ¬p) ∧ (q ∨ r). The first clause is a tautology and can be dropped: (p ∨ r) ∧ (¬p ∨ q) ∧ (q ∨ r).
3. p → (q ↔ r) ≡ ¬p ∨ (q ↔ r) ≡ ¬p ∨ (q ∧ r) ∨ (¬q ∧ ¬r).
4. Yes. From p and p → q, q; from q and q → r, r. Every valuation satisfying the premises satisfies r.
5. No. Countermodel: p = 0, q = 1. Both premises true, ¬q false. This is denying the antecedent.
6. Example: premises p → q and p, conclusion q, with p false and q false. Valid by modus ponens; the conclusion is false because a premise is. A valid argument with all true premises must have a true conclusion, so a false conclusion forces at least one false premise.
7. First: ¬gps_fix (MT on gps_fix → in_geofence, ¬in_geofence); ¬armed (MT on armed → gps_fix, ¬gps_fix). Second: armed → in_geofence (HS); ¬armed (MT).
8. airborne → armed, armed → gps_fix, ¬gps_fix. MT gives ¬armed, MT again gives ¬airborne. The aircraft is not airborne.
9. If every valuation satisfying Γ also satisfies C, then no valuation satisfies Γ together with ¬C, so Γ ∧ ¬C is unsatisfiable. Conversely, if Γ ∧ ¬C is unsatisfiable, no valuation makes Γ true and C false, which is the definition of Γ ⊨ C.

### Section 4

1. (a) ∀x (Airborne(x) → Airworthy(x)). (b) ∃x (Airworthy(x) ∧ ¬Airborne(x)). (c) ∀x (Airborne(x) → HasFix(x)), or ¬∃x (Airborne(x) ∧ ¬HasFix(x)). (d) ∀x (Airborne(x) ∨ Airworthy(x)).
2. Some aircraft is flying while not airworthy. That is an aircraft that should be on the ground.
3. ∃x (Armed(x) ∧ ¬Airborne(x)).
4. ∀x (LowBatt(x) → RTL(x)). Every low-battery aircraft is returning to launch.
5. The first says every aircraft has something above it (possibly a different thing for each). The second says one aircraft is above all others. Over a finite fleet the first is false, because the highest aircraft has nothing above it (unless HigherThan is allowed to be reflexive, which it should not be). The second can be true when there is a unique highest aircraft, again setting aside the reflexive case.
6. Domain {E, L}, P = Airborne, with E airborne and L not.
7. Fleet {E, L}. E armed with fix, L unarmed with no fix. The English is true (the only armed aircraft has a fix). The student's formula is false because L is not armed.
8. (Armed_E → HasFix_E) ∧ (Armed_L → HasFix_L) ∧ (Armed_Y → HasFix_Y).

---

## References

Boole, G. (1847). *The Mathematical Analysis of Logic, Being an Essay Towards a Calculus of Deductive Reasoning*. Macmillan, Barclay, and Macmillan, Cambridge.

Boole, G. (1854). *An Investigation of the Laws of Thought, on Which are Founded the Mathematical Theories of Logic and Probabilities*. Walton and Maberly, London.

Clarke, E. M., Grumberg, O., and Peled, D. A. (1999). *Model Checking*. MIT Press.

Cook, S. A. (1971). The complexity of theorem-proving procedures. *Proceedings of the 3rd Annual ACM Symposium on Theory of Computing (STOC)*, 151–158.

Davis, M. and Putnam, H. (1960). A computing procedure for quantification theory. *Journal of the ACM*, 7(3), 201–215.

Davis, M., Logemann, G., and Loveland, D. (1962). A machine program for theorem-proving. *Communications of the ACM*, 5(7), 394–397.

De Morgan, A. (1847). *Formal Logic: or, The Calculus of Inference, Necessary and Probable*. Taylor and Walton, London.

Enderton, H. B. (2001). *A Mathematical Introduction to Logic*, 2nd edition. Academic Press.

Frege, G. (1879). *Begriffsschrift, eine der arithmetischen nachgebildete Formelsprache des reinen Denkens*. Louis Nebert, Halle.

Gentzen, G. (1935). Untersuchungen über das logische Schließen. *Mathematische Zeitschrift*, 39, 176–210 and 405–431.

Gödel, K. (1930). Die Vollständigkeit der Axiome des logischen Funktionenkalküls. *Monatshefte für Mathematik und Physik*, 37, 349–360.

Hoare, C. A. R. (1969). An axiomatic basis for computer programming. *Communications of the ACM*, 12(10), 576–580.

Huntington, E. V. (1904). Sets of independent postulates for the algebra of logic. *Transactions of the American Mathematical Society*, 5(3), 288–309.

Huth, M. and Ryan, M. (2004). *Logic in Computer Science: Modelling and Reasoning about Systems*, 2nd edition. Cambridge University Press.

Karnaugh, M. (1953). The map method for synthesis of combinational logic circuits. *Transactions of the American Institute of Electrical Engineers, Part I*, 72(5), 593–599.

Magnus, P. D., Button, T., Trueman, R., Zach, R., and Thomas-Bolduc, A. (2023). *forall x: Calgary. An Introduction to Formal Logic*, Fall 2023 edition. Open Logic Project. Available at forallx.openlogicproject.org under CC BY 4.0.

McCluskey, E. J. (1956). Minimization of Boolean functions. *Bell System Technical Journal*, 35(6), 1417–1444.

Peirce, C. S. (1885). On the algebra of logic: A contribution to the philosophy of notation. *American Journal of Mathematics*, 7(2), 180–202.

Post, E. L. (1921). Introduction to a general theory of elementary propositions. *American Journal of Mathematics*, 43(3), 163–185.

Quine, W. V. (1952). The problem of simplifying truth functions. *American Mathematical Monthly*, 59(8), 521–531.

Rosen, K. H. (2019). *Discrete Mathematics and Its Applications*, 8th edition. McGraw-Hill.

Shannon, C. E. (1938). A symbolic analysis of relay and switching circuits. *Transactions of the American Institute of Electrical Engineers*, 57(12), 713–723.

Sheffer, H. M. (1913). A set of five independent postulates for Boolean algebras, with application to logical constants. *Transactions of the American Mathematical Society*, 14(4), 481–488.

Tseitin, G. S. (1968). On the complexity of derivation in propositional calculus. In *Studies in Constructive Mathematics and Mathematical Logic, Part II*, 115–125. Reprinted in Siekmann and Wrightson (eds.), *Automation of Reasoning 2*, Springer, 1983.

Wittgenstein, L. (1921). Logisch-philosophische Abhandlung. *Annalen der Naturphilosophie*, 14, 185–262. English edition: *Tractatus Logico-Philosophicus*, Kegan Paul, 1922.

### A note on which sources to assign

For students, Magnus et al. (2023) is the best single free text for sections 3 and 4; it has exercises with solutions and a matching proof checker. Rosen (2019), chapter 1, covers all four sections at the level of this tutorial with many more drill problems. Huth and Ryan (2004) is the right bridge to Parts 2 and 3, since it moves from propositional and predicate logic directly into model checking and temporal logic. The primary sources (Boole, Frege, Peirce, Post, Shannon) are cited so students can see where the ideas came from; none of them is required reading.

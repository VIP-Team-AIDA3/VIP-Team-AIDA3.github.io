# Introduction to Logic

## Part 1: Boolean logic, propositional logic, and predicates

This is the first of three parts. Part 1 covers the basics: Boolean logic, which works with true and false; propositional logic, which uses those values to decide whether an argument holds up; and predicates with quantifiers, which let you say things about one aircraft or about a whole fleet. Part 2 covers first-order logic in full. Part 3 covers temporal logic, which is how you write rules about what a system must do over time.

You do not need a prior logic course. If you have ever written an `if` statement with `&&` or `||` in it, you already know some of this.

**How to use this tutorial.** Each section has worked problems, marked **Worked problem**, with the solution directly underneath. Cover the solution, try the problem, then check. Each section ends with a short set of exercises. The mixed problems at the end pull from more than one section. An answer key for the exercises and the mixed problems is near the end of the document.

---

## 1. Why logic

### 1.1 The one question

Logic is about what follows from what. If you know some things, what else do you have to accept? Each system in this tutorial answers that question for a different kind of statement. Boolean logic handles plain true-or-false values. Propositional logic handles sentences built from those values. Predicate logic handles sentences about objects, like "every aircraft in the fleet has a GPS fix." Temporal logic, in Part 3, handles sentences about time, like "the aircraft never leaves the geofence."

Why does an aviation course spend time on this? Because a flight controller is a machine for deciding what follows from its inputs. The arming interlock is a Boolean expression. The failsafe rules are propositional logic. The requirements document that says what the aircraft must never do is, once you write it carefully, a list of logical formulas. When someone says a system is "safe," they usually mean its behavior follows from its rules. Logic is the tool for checking that.

Three questions come up in every section, so it helps to name them now. How do you write a statement so it has only one meaning? That is syntax. When is a statement true? That is semantics. How do you get from statements you already have to new ones you are allowed to conclude? That is proof. Most confusion in a first logic course comes from mixing these up. When you get stuck, ask yourself which of the three you are doing.

### 1.2 A running example

We will keep coming back to six statements about a single quadrotor. Each one is either true or false at any given moment.

- armed: the flight controller has been armed
- airborne: the aircraft has left the ground
- gps_fix: the GPS has a valid position fix
- in_geofence: the current position is inside the mission geofence
- low_batt: the battery is below the return threshold
- rtl: the aircraft is in return-to-launch mode

In section 2 these are Boolean variables. In section 3 they are propositions. In section 4 we open them up so we can talk about many aircraft at once.

---

## 2. Boolean logic

Boolean logic works with exactly two values. We write them 1 and 0, or true and false, or T and F. The symbols do not matter. The whole subject is about how to combine those two values.

George Boole worked this out in the 1850s as pure mathematics. About eighty years later, Claude Shannon showed in his master's thesis that electrical switching circuits obey Boole's rules. That is why every digital computer, including the one on your aircraft, is Boolean logic built in silicon.

### 2.1 Variables and expressions

A Boolean variable is a name for a value that is 0 or 1. We use p, q, r, and the six names from section 1.2. A Boolean expression combines variables with operations. Once you give every variable a value, the expression works out to a single 0 or 1, the same way an arithmetic expression works out to a single number.

### 2.2 The three basic operations

There are three operations to learn cold. Everything else is built from them.

NOT, written ¬p, flips the value. NOT 1 is 0 and NOT 0 is 1. In code this is `!p`.

AND, written p ∧ q, is 1 only when both p and q are 1. In code this is `p && q`.

OR, written p ∨ q, is 1 when at least one of p or q is 1. In code this is `p || q`. OR is also 1 when both inputs are 1. Everyday English often uses "or" to mean "one or the other but not both." That is a different operation, and we get to it in section 2.3.

A truth table lists every possible combination of input values and the output for each one. It is the most useful tool in this tutorial, so make sure you can build one from scratch. Here are the three basic operations:

| p | q | ¬p | p ∧ q | p ∨ q |
|---|---|----|-------|-------|
| 0 | 0 | 1  | 0     | 0     |
| 0 | 1 | 1  | 0     | 1     |
| 1 | 0 | 0  | 0     | 1     |
| 1 | 1 | 0  | 1     | 1     |

To build a truth table, list the variables, then write the rows by counting in binary from all zeros to all ones. Two variables give four rows. Three give eight. In general, n variables give 2^n rows. Counting in binary is how you make sure you do not skip one.

**Worked problem 2.1.** A motor interlock lets the propellers spin only when the aircraft is armed and the GPS has a fix. Write this as a Boolean expression and give its truth table.

*Solution.* spin = armed ∧ gps_fix.

| armed | gps_fix | spin |
|-------|---------|------|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

The propellers can spin in exactly one of the four possible states. That is what an interlock should look like.

**Worked problem 2.2.** Build the truth table for ¬p ∨ q. Work one column at a time.

*Solution.* First compute ¬p, then OR it with q.

| p | q | ¬p | ¬p ∨ q |
|---|---|----|--------|
| 0 | 0 | 1 | 1 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 |
| 1 | 1 | 0 | 1 |

Remember this table. It comes back in section 2.4.

### 2.3 Four more operations

Four other operations show up all the time. Each can be built from NOT, AND, and OR.

XOR, "exclusive or," written p ⊕ q, is 1 when the inputs are different. This is the "one or the other but not both" meaning of "or." You can write it as (p ∨ q) ∧ ¬(p ∧ q).

NAND is NOT applied to AND: ¬(p ∧ q). It is 0 only when both inputs are 1. NAND matters in hardware because you can build every other operation using nothing but NAND gates.

NOR is NOT applied to OR: ¬(p ∨ q). It is 1 only when both inputs are 0.

Implication, written p → q and read "if p then q," is 0 only when p is 1 and q is 0. Everyone finds this one strange at first, so it gets its own subsection.

| p | q | p ⊕ q | ¬(p ∧ q) | ¬(p ∨ q) | p → q |
|---|---|-------|----------|----------|-------|
| 0 | 0 | 0 | 1 | 1 | 1 |
| 0 | 1 | 1 | 1 | 0 | 1 |
| 1 | 0 | 1 | 1 | 0 | 0 |
| 1 | 1 | 0 | 0 | 0 | 1 |

**Worked problem 2.3.** A control mode selector has two inputs, manual and auto. Exactly one should be on at any time. Write the "healthy" condition and the "fault" condition.

*Solution.* healthy = manual ⊕ auto. That is 1 when exactly one mode is on. fault = ¬(manual ⊕ auto), which is 1 when both are on or neither is on. Both of those are real faults. Both on means two systems are fighting for control. Neither on means nobody is flying.

### 2.4 Making sense of implication

Look at the p → q column again. It is 0 in exactly one row, where p = 1 and q = 0. In the two rows where p = 0, the implication is 1 no matter what q is.

The way to make this feel right is to read p → q as a promise: "if p happens, then q will happen." The only way to break that promise is for p to happen and q not to. If p never happens, you have not broken anything. The rule armed → gps_fix is not violated by an aircraft sitting disarmed on the bench with no GPS fix, because the rule only says something about what happens when the aircraft is armed.

Now compare the p → q column to the ¬p ∨ q column from worked problem 2.2. They are identical. So p → q is just another way of writing ¬p ∨ q. Memorize this one. It lets you get rid of arrows whenever they are in the way.

Two more facts about implication. The converse of p → q is q → p. It is a different statement with a different truth table. The contrapositive of p → q is ¬q → ¬p, and it has the same truth table as p → q. Mixing up the converse and the contrapositive is one of the most common mistakes in all of logic, and it shows up constantly in badly written requirements.

The biconditional, written p ↔ q and read "p if and only if q," is 1 when p and q have the same value. It is the same as (p → q) ∧ (q → p).

**Worked problem 2.4.** The rule is "if the battery is low, then return to launch," so low_batt → rtl. Right now low_batt = 0 and rtl = 1. Is the rule satisfied? What if low_batt = 1 and rtl = 0?

*Solution.* First case: low_batt is 0, so the "if" part is false and the implication is 1. The rule is satisfied. The aircraft is returning for some other reason, which the rule does not forbid. Second case: low_batt = 1 and rtl = 0. That is the one bad row. The rule is violated.

**Worked problem 2.5.** A student reads armed → gps_fix as "if the aircraft has a fix then it is armed." What did they get wrong, and why does it matter?

*Solution.* They read the converse. The rule says arming requires a fix. Their reading says a fix causes arming, which would mean every aircraft with a working GPS is armed, including the ones sitting on the shelf. You cannot flip an implication for free.

### 2.5 The laws of Boolean algebra

You can rewrite a Boolean expression into a different expression that always has the same value, the same way you can rewrite 2(x + 3) as 2x + 6. The rules for doing this are called the laws of Boolean algebra. Each one can be checked with a truth table, and you should do that at least once for each law so you trust them.

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

Each law comes as a pair. Swap ∧ with ∨ and 0 with 1 and you get the partner law. That is a handy check on your memory.

De Morgan's laws are the ones you will use most. They tell you how to push a NOT inside a bracket: the NOT lands on each piece, and AND becomes OR or OR becomes AND. "It is not true that the aircraft is armed and airborne" is the same as "it is not armed, or it is not airborne." If you have ever simplified `!(a && b)` to `!a || !b` in code, you have used De Morgan.

One warning. Distributivity in Boolean algebra works both ways, which is different from ordinary arithmetic. In arithmetic, a × (b + c) = ab + ac, but a + (b × c) does not distribute. In Boolean algebra, ∨ distributes over ∧ just as well as ∧ distributes over ∨. Students who carry over the arithmetic habit miss half the table.

**Worked problem 2.6.** Simplify (p ∧ q) ∨ (p ∧ ¬q) as far as possible, naming the law at each step.

*Solution.*
(p ∧ q) ∨ (p ∧ ¬q)
= p ∧ (q ∨ ¬q)   distributivity, pulling p out of both terms
= p ∧ 1   complement
= p   identity

Reading the original in English makes the answer obvious: "p and q, or p and not q." Whatever q is, one of the two branches is just p.

**Worked problem 2.7.** A safety monitor should raise an alarm whenever it is not the case that the aircraft is inside the geofence with a GPS fix. Write the alarm condition and push the NOT inside.

*Solution.* alarm = ¬(in_geofence ∧ gps_fix) = ¬in_geofence ∨ ¬gps_fix by De Morgan. The alarm fires if the aircraft has left the fence, or if it has lost its fix, or both. The De Morgan version is the one you would actually code, because it lists the two separate things that can go wrong.

### 2.6 From a truth table to an expression

Sometimes you know what output you want for every input and need an expression that produces it. There is a mechanical way to do that. Find every row where the output is 1. For each such row, write an AND term that is true only on that row: use the variable if it is 1 in that row, and the negated variable if it is 0. Then OR all the terms together.

**Worked problem 2.8.** Find an expression for f from this table, then simplify it.

| p | q | f |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

*Solution.* The 1 rows are (0,0), (0,1), and (1,1). The terms are ¬p ∧ ¬q, ¬p ∧ q, and p ∧ q. So f = (¬p ∧ ¬q) ∨ (¬p ∧ q) ∨ (p ∧ q). The first two terms simplify to ¬p, the same move as in worked problem 2.6, giving f = ¬p ∨ (p ∧ q). Distribute: (¬p ∨ p) ∧ (¬p ∨ q) = 1 ∧ (¬p ∨ q) = ¬p ∨ q. That is p → q. The table was implication in disguise.

For expressions with many variables, engineers use Karnaugh maps or software to do this simplification. For this course the laws above are enough.

### 2.7 Operator reference

| Operator | Symbol | Value | Example |
|----------|--------|-------|---------|
| NOT | ¬p | flips p | ¬armed: the aircraft is disarmed |
| AND | p ∧ q | 1 only when both are 1 | armed ∧ airborne |
| OR | p ∨ q | 1 when at least one is 1 | low_batt ∨ ¬gps_fix: either warning is active |
| XOR | p ⊕ q | 1 when exactly one is 1 | manual ⊕ auto: exactly one mode on |
| NAND | ¬(p ∧ q) | 0 only when both are 1 | can build every other gate |
| NOR | ¬(p ∨ q) | 1 only when both are 0 | ¬(armed ∨ airborne): idle |
| Implication | p → q | 0 only when p = 1 and q = 0 | armed → gps_fix: arming needs a fix |
| Biconditional | p ↔ q | 1 when both sides match | gear_down ↔ landing_mode |

---

## 3. Propositional logic

Propositional logic is Boolean logic with the values reinterpreted. Instead of abstract 1s and 0s, the values are now the truth or falsity of actual sentences. The math is identical. What changes is the question. Boolean logic asks "what does this expression compute?" Propositional logic asks "does this conclusion follow from these premises?"

### 3.1 Propositions

A proposition is a sentence that is definitely true or definitely false. "The airport is closed" is a proposition. "Close the runway" is not, because a command is neither true nor false. "Is the runway closed?" is not, because a question has no truth value either. "x is above 120 metres" is not a proposition on its own, because you cannot say whether it is true until you know what x is. Section 4 fixes that last case.

An atomic proposition is one you cannot break into smaller propositions. We give each one a name. A compound proposition is built from atomic ones using connectives, which are the Boolean operations under their logic names: negation ¬, conjunction ∧, disjunction ∨, implication →, and biconditional ↔.

**Worked problem 3.1.** Which of these are propositions? (a) "The aircraft is armed." (b) "Arm the aircraft." (c) "The aircraft is armed and it is not armed."

*Solution.* (a) Yes. (b) No, it is a command. (c) Yes. It happens to be always false, but always false is still a definite truth value.

### 3.2 Writing formulas correctly

Not every string of symbols means something. p ∧ ∨ q is nonsense. A well-formed formula is one built by these rules: every atomic proposition is a formula; if A is a formula, so is ¬A; if A and B are formulas, so are (A ∧ B), (A ∨ B), (A → B), and (A ↔ B). Nothing else counts.

Writing every parenthesis gets cluttered, so we use precedence rules, the same way arithmetic does multiplication before addition. Here ¬ binds tightest, then ∧, then ∨, then →, then ↔. So p ∨ q ∧ r means p ∨ (q ∧ r), and ¬p → q means (¬p) → q. When in doubt, add parentheses. Nobody has ever lost points for too many.

**Worked problem 3.2.** Add full parentheses to ¬p ∧ q → r ∨ ¬s.

*Solution.* Handle ¬ first: (¬p) ∧ q → r ∨ (¬s). Then ∧: ((¬p) ∧ q) → r ∨ (¬s). Then ∨: ((¬p) ∧ q) → (r ∨ (¬s)). The outermost connective is →, so this formula is an implication.

### 3.3 Evaluating a formula

To find the truth value of a formula, assign a value to each atomic proposition and work outward using the truth tables from section 2. A full assignment of values to the atoms is called a valuation. If a formula has n atoms, there are 2^n valuations, one per truth-table row.

**Worked problem 3.3.** Evaluate (armed → gps_fix) ∧ (¬gps_fix → ¬airborne) when armed = 1, gps_fix = 0, airborne = 0.

*Solution.* armed → gps_fix is 1 → 0, which is 0. Since one side of an AND is 0, the whole formula is 0 and we can stop. (For the record, the other side is ¬0 → ¬0, which is 1 → 1, which is 1.) The rules are violated because the aircraft is armed without a fix.

### 3.4 Tautology, contradiction, contingency

A formula that is true under every valuation is a tautology. The simplest is p ∨ ¬p: whatever p is, either it or its negation holds. A formula that is false under every valuation is a contradiction; p ∧ ¬p is the simplest. A formula that is true under some valuations and false under others is a contingency. Almost every formula you write about a real system is a contingency, because it says something that could go either way depending on sensor values.

**Worked problem 3.4.** Is (p → q) ∨ (q → p) a tautology?

*Solution.* Try to make it false. Both halves would need to be 0. p → q is 0 only when p = 1 and q = 0. q → p is 0 only when q = 1 and p = 0. You cannot have both at once. So no valuation makes it false, and it is a tautology. It is a strange one: for any two propositions at all, one implies the other. That is a side effect of the "promise" reading of implication, and a good reminder that → does not mean "causes."

### 3.5 Logical equivalence

Two formulas are logically equivalent, written A ≡ B, when they have the same truth value under every valuation. In other words, they have the same truth-table column. All the laws from section 2.5 are equivalences. Three more involving implication get used constantly:

- p → q ≡ ¬p ∨ q
- p → q ≡ ¬q → ¬p (contrapositive)
- ¬(p → q) ≡ p ∧ ¬q

The third one is the most useful and the least obvious. To say an implication is false is to say the "if" part happened and the "then" part did not.

**Worked problem 3.5.** A manual says R1: "if the battery is low, the aircraft returns to launch." A reviewer rewrites it as R2: "if the aircraft is not returning to launch, the battery is not low." Do R1 and R2 say the same thing?

*Solution.* R1 is low_batt → rtl. R2 is ¬rtl → ¬low_batt. R2 is the contrapositive of R1, so yes, they are equivalent. If the reviewer had instead written "if the aircraft is returning to launch, the battery is low," that would be rtl → low_batt, the converse, which is a different and stronger rule. It would forbid returning to launch for any other reason.

### 3.6 Does the conclusion follow?

This is the question propositional logic exists to answer. A set of premises entails a conclusion when every valuation that makes all the premises true also makes the conclusion true. We write it with a double turnstile: {A, B} ⊨ C means A and B together entail C.

An argument is valid when its premises entail its conclusion. Validity is about the shape of the argument, not about whether the premises happen to be true today. An argument can be valid even if its premises are false. An argument can also be invalid even when the premises and conclusion all happen to be true, because "happens to be true" is not the same as "follows from."

To show an argument is invalid, find one valuation that makes every premise true and the conclusion false. That valuation is called a countermodel. To show an argument is valid, either check every row of the truth table, or argue directly that the premises force the conclusion.

**Worked problem 3.6.** Does {armed → gps_fix, gps_fix → in_geofence, armed} ⊨ in_geofence?

*Solution.* Suppose all three premises are true. armed is true, and armed → gps_fix is true, so gps_fix must be true (otherwise the implication would be 1 → 0). Now gps_fix is true and gps_fix → in_geofence is true, so in_geofence must be true. Every valuation that satisfies the premises satisfies the conclusion. Yes, it follows.

**Worked problem 3.7.** Does {armed → gps_fix, gps_fix} ⊨ armed?

*Solution.* Look for a countermodel. Try armed = 0. Then armed → gps_fix is 0 → anything, which is 1. Set gps_fix = 1 so the second premise holds. The conclusion armed is 0. Both premises true, conclusion false. So no, it does not follow. This mistake, reasoning backward from the "then" to the "if," is called affirming the consequent. "If it is armed it has a fix; it has a fix; so it is armed" sounds reasonable and is wrong.

### 3.7 Standard shapes for formulas

It is often useful to force a formula into a standard shape. A literal is an atom or a negated atom, like p or ¬q. A clause is a group of literals joined by ∨.

A formula is in conjunctive normal form, CNF, when it is clauses joined by ∧. A formula is in disjunctive normal form, DNF, when it is AND-terms joined by ∨. Section 2.6 already produced a DNF from a truth table. Every formula can be put into either form by the same three moves: replace → and ↔ using the equivalences above, push every ¬ inward with De Morgan and double negation, then distribute.

CNF matters because it is the input format for SAT solvers, programs that decide whether a formula can be made true. Modern SAT solvers handle formulas with millions of clauses, and they sit inside the industrial tools that check whether a design meets its specification. Deciding satisfiability in general is NP-complete, so nobody expects a fast algorithm for every possible input. In practice the solvers are still remarkably fast on the formulas that come from real designs.

**Worked problem 3.8.** Convert (p ∧ q) ∨ ¬r to CNF.

*Solution.* Distribute the ∨ over the ∧: (p ∨ ¬r) ∧ (q ∨ ¬r). Two clauses, each a disjunction of literals. That is CNF.

**Worked problem 3.9.** Convert ¬(p → q) ∨ r to DNF.

*Solution.* ¬(p → q) becomes p ∧ ¬q. So the formula is (p ∧ ¬q) ∨ r. That is two terms joined by ∨, where r counts as a one-literal term. That is DNF.

### 3.8 Proofs: getting to the conclusion step by step

Truth tables always work, but they double in size with every atom. Ten atoms means 1,024 rows. A proof gets to the conclusion by applying small rules one step at a time instead. Each rule is a pattern: if you already have lines that match the "from" column, you may write down the "infer" line.

| Rule | From | Infer |
|------|------|-------|
| Modus ponens | A, A → B | B |
| Modus tollens | ¬B, A → B | ¬A |
| Hypothetical syllogism | A → B, B → C | A → C |
| Disjunctive syllogism | A ∨ B, ¬A | B |
| Conjunction introduction | A, B | A ∧ B |
| Conjunction elimination | A ∧ B | A (or B) |

Modus ponens is "the if-part happened, so the then-part happens." Modus tollens is "the then-part did not happen, so the if-part did not either." Hypothetical syllogism chains two implications. Disjunctive syllogism says that if one of two options is ruled out, the other holds.

When you can get from premises to a conclusion using these rules, we write it with a single turnstile: Γ ⊢ C, read "Γ proves C." (Γ, capital gamma, is the usual name for a set of premises.)

**Worked problem 3.10.** From armed → gps_fix, gps_fix → in_geofence, and ¬in_geofence, prove ¬armed.

*Solution.*
1. armed → gps_fix (premise)
2. gps_fix → in_geofence (premise)
3. ¬in_geofence (premise)
4. ¬gps_fix (modus tollens on 2 and 3)
5. ¬armed (modus tollens on 1 and 4)

Another route: 4′. armed → in_geofence (hypothetical syllogism on 1 and 2); 5′. ¬armed (modus tollens on 4′ and 3). Both are correct. There is usually more than one proof.

**Worked problem 3.11.** From low_batt ∨ ¬gps_fix, gps_fix, and low_batt → rtl, prove rtl.

*Solution.*
1. low_batt ∨ ¬gps_fix (premise)
2. gps_fix (premise)
3. low_batt → rtl (premise)
4. low_batt (disjunctive syllogism on 1 and 2: gps_fix is the negation of ¬gps_fix by double negation, so the second option is ruled out)
5. rtl (modus ponens on 3 and 4)

### 3.9 Why you can trust a proof

There are two ways to establish that a conclusion follows: check the meaning (⊨, truth tables) or push symbols (⊢, proof rules). For propositional logic these always agree. Every proof leads to a conclusion that really is entailed. That property is called soundness. Every entailed conclusion has a proof. That property is called completeness. So if you find a proof, the conclusion follows. If no proof exists, there is a countermodel out there. You never have to worry that the two methods disagree.

### 3.10 Reference tables

| Connective | Symbol | Read as | Example |
|------------|--------|---------|---------|
| Negation | ¬P | not P | ¬armed |
| Conjunction | P ∧ Q | P and Q | gps_fix ∧ in_geofence |
| Disjunction | P ∨ Q | P or Q, or both | low_batt ∨ rtl |
| Implication | P → Q | if P then Q | armed → gps_fix |
| Biconditional | P ↔ Q | P if and only if Q | rtl ↔ low_batt |

| Symbol | Name | Meaning |
|--------|------|---------|
| ⊨ | entails | every valuation that makes the left side true makes the right side true |
| ⊢ | proves | the right side can be derived from the left side using the rules |
| ≡ | equivalent | same truth-table column |

---

## 4. Predicates and quantifiers

Propositional logic treats "Earhart is airborne" as a single unbreakable atom. It cannot see that the sentence is about a thing (the aircraft Earhart) and a property (being airborne). So it cannot express "every aircraft in the fleet is airworthy," because there is no atom for "every." Predicates and quantifiers add that ability. This section introduces them. Part 2 builds the full system.

### 4.1 Predicates

A predicate is a sentence with a blank in it. Fill the blank with an object and you get a proposition. Write Airborne(x) for "x is airborne." By itself Airborne(x) is neither true nor false, because x is a placeholder. Airborne(Earhart) is a proposition.

If you like programming analogies, a predicate is a function that returns a Boolean. `isAirborne(aircraft)` returns true or false once you pass it a particular aircraft.

A predicate can have more than one blank. The number of blanks is its arity. Airborne(x) has arity 1. HigherThan(x, y), "x is at a higher altitude than y," has arity 2. A one-blank predicate describes a property of one thing. A two-blank predicate describes a relationship between two things.

**Worked problem 4.1.** Name the predicate and its arity: (a) "the aircraft has a GPS fix"; (b) "aircraft A is within 50 metres of aircraft B"; (c) "waypoint W is on the route from A to B."

*Solution.* (a) HasFix(x), arity 1. (b) Within50(x, y), arity 2. (c) OnRoute(w, a, b), arity 3. In (b), swapping A and B does not change the truth value. In HigherThan(x, y) it does. Order matters for some relations and not for others.

### 4.2 The domain

A predicate only makes sense relative to a domain, the set of things you are talking about. If the domain is the three aircraft in the fleet, then x ranges over those three and nothing else. Choosing the domain is part of setting up the problem. "Everything is airworthy" is a very different claim over the fleet than over every object in the hangar.

A constant names one specific thing in the domain, like Earhart. A variable like x stands for an unspecified thing. Keep the difference in mind. It matters in the next subsection.

### 4.3 The two quantifiers

Quantifiers are how you say "all" and "some."

The universal quantifier ∀ is read "for all." ∀x Airworthy(x) says every aircraft in the domain is airworthy. One aircraft that is not airworthy makes the whole statement false. That aircraft is called a counterexample.

The existential quantifier ∃ is read "there exists." ∃x Airborne(x) says at least one aircraft is airborne. One airborne aircraft makes the whole statement true. That aircraft is called a witness.

Here is a way to picture quantifiers that makes the rest of this section easier. If the domain is a finite list of things a₁ through aₙ, then ∀x P(x) is just P(a₁) ∧ P(a₂) ∧ … ∧ P(aₙ), a big AND, and ∃x P(x) is just P(a₁) ∨ P(a₂) ∨ … ∨ P(aₙ), a big OR. Over a fleet of three aircraft, "all are airworthy" means "Earhart is airworthy and Lindbergh is airworthy and Yeager is airworthy." A quantifier lets you write that without knowing in advance how long the list is.

**Worked problem 4.2.** The fleet is {Earhart, Lindbergh, Yeager}. Only Earhart is airborne. All three are airworthy. Evaluate ∀x Airworthy(x), ∃x Airborne(x), ∀x Airborne(x), and ∃x ¬Airworthy(x).

*Solution.* ∀x Airworthy(x): true, all three pass. ∃x Airborne(x): true, Earhart is a witness. ∀x Airborne(x): false, Lindbergh is a counterexample (so is Yeager). ∃x ¬Airworthy(x): false, no aircraft fails airworthiness.

### 4.4 "Every airborne aircraft": restricted quantifiers

Most useful sentences are not about every thing in the domain but about every thing of a certain kind. "Every airborne aircraft has a fix" is only about the airborne ones. The two quantifiers handle this differently. This is the most common mistake in translating English to logic, so slow down here.

A universal statement about a kind uses →:

∀x (Airborne(x) → HasFix(x))

"For every x, if x is airborne then x has a fix." Aircraft on the ground make the "if" part false, so they satisfy the implication automatically and do not count against the claim. That is what we want.

An existential statement about a kind uses ∧:

∃x (Airborne(x) ∧ ¬HasFix(x))

"There is some x that is airborne and has no fix."

Now see what goes wrong if you swap them. ∀x (Airborne(x) ∧ HasFix(x)) says every aircraft is airborne and has a fix. One parked aircraft makes that false. Far too strong. And ∃x (Airborne(x) → HasFix(x)) is satisfied by any aircraft on the ground, since a false "if" part makes the implication true. It says almost nothing. So the pairing is fixed: ∀ goes with →, ∃ goes with ∧.

**Worked problem 4.3.** Write in symbols: (a) "No armed aircraft is outside the geofence." (b) "Some aircraft with a low battery is not returning to launch."

*Solution.* (a) Two good answers. ∀x (Armed(x) → InGeofence(x)), "every armed aircraft is in the fence." Or ¬∃x (Armed(x) ∧ ¬InGeofence(x)), "there is no armed aircraft outside the fence." The next subsection shows these are the same. (b) ∃x (LowBatt(x) ∧ ¬RTL(x)).

### 4.5 Negating a quantifier

A NOT in front of a quantifier behaves just like De Morgan's laws, which makes sense given the big-AND and big-OR picture from section 4.3.

- ¬∀x P(x) ≡ ∃x ¬P(x). "Not everything is P" means "something is not P."
- ¬∃x P(x) ≡ ∀x ¬P(x). "Nothing is P" means "everything is not P."

The NOT moves inward and the quantifier flips. To push a NOT all the way through a restricted quantifier, use this and then the propositional rule for negating an implication:

¬∀x (A(x) → B(x)) ≡ ∃x ¬(A(x) → B(x)) ≡ ∃x (A(x) ∧ ¬B(x))

In English: "it is not true that every A is a B" means "some A is not a B." That is the shape of a counterexample, and it is what you actually look for when checking a rule.

**Worked problem 4.4.** A requirement says ∀x (Airborne(x) → HasFix(x)). What does a violation look like, in symbols and in English?

*Solution.* A violation is ¬∀x (Airborne(x) → HasFix(x)), which becomes ∃x (Airborne(x) ∧ ¬HasFix(x)): some aircraft is in the air without a fix. That is what a monitor would search for. To check a "for all" rule, you hunt for a "there exists" violation.

**Worked problem 4.5.** Push the NOT inward in ¬∃x (LowBatt(x) ∧ ¬RTL(x)).

*Solution.* ¬∃x becomes ∀x ¬, giving ∀x ¬(LowBatt(x) ∧ ¬RTL(x)). De Morgan: ∀x (¬LowBatt(x) ∨ RTL(x)). Rewrite as an implication: ∀x (LowBatt(x) → RTL(x)). "There is no low-battery aircraft that is not returning" is the same as "every low-battery aircraft is returning."

### 4.6 More than one quantifier

You can stack quantifiers, and the order changes the meaning. Let CanReach(x, y) mean aircraft x can reach waypoint y.

∀y ∃x CanReach(x, y): for every waypoint, there is some aircraft that can reach it. Different waypoints may be covered by different aircraft.

∃x ∀y CanReach(x, y): there is one aircraft that can reach every waypoint. A much stronger claim.

Read the quantifiers left to right and imagine each one being chosen before the ones after it. In the first sentence, the waypoint is picked first and you get to choose the aircraft afterward, so the aircraft can depend on the waypoint. In the second, the aircraft is picked first and has to work for every waypoint that comes after. Part 2 returns to this in detail, because a mission specification with the quantifiers in the wrong order is a real bug.

**Worked problem 4.6.** Aircraft {E, L}, waypoints {W1, W2}. E can reach W1 only. L can reach W2 only. Evaluate both sentences above.

*Solution.* ∀y ∃x CanReach(x, y): for W1 choose E, for W2 choose L. Every waypoint is covered, so true. ∃x ∀y CanReach(x, y): is there one aircraft that reaches both? E misses W2, L misses W1. False. This is the standard example showing that the ∀∃ version can be true while the ∃∀ version is false.

### 4.7 Operator reference

| Operator | Symbol | Meaning | Example |
|----------|--------|---------|---------|
| Universal | ∀x P(x) | P is true of everything in the domain | ∀x Airworthy(x) |
| Existential | ∃x P(x) | P is true of at least one thing | ∃x Airborne(x) |
| Restricted universal | ∀x (A(x) → B(x)) | every A is a B | ∀x (Armed(x) → InGeofence(x)) |
| Restricted existential | ∃x (A(x) ∧ B(x)) | some A is a B | ∃x (Airborne(x) ∧ ¬HasFix(x)) |
| Negated universal | ¬∀x P(x) ≡ ∃x ¬P(x) | something is not P | ¬∀x Airworthy(x) |
| Negated existential | ¬∃x P(x) ≡ ∀x ¬P(x) | nothing is P | ¬∃x Airborne(x): all on the ground |

---

## Mixed problems

These combine ideas from more than one section.

**P1.** An arming interlock has inputs gps_fix, in_geofence, and preflight_ok. Arming is permitted when all three are true, or when a maintenance override is on and preflight_ok is true. (a) Write the permit expression. (b) Simplify it using the laws. (c) Write the "arming refused" condition with every NOT on a single variable. (d) How many rows would the full truth table have?

**P2.** Show that p → (q → r) and (p ∧ q) → r are equivalent. Do it once with a truth table and once using only the laws and the equivalence p → q ≡ ¬p ∨ q.

**P3.** A flight manual has three rules. R1: if the aircraft is airborne it is armed. R2: if it is armed it has a fix. R3: if it has a fix it is in the geofence. An operator sees an aircraft outside the geofence. (a) Write the rules and the observation in symbols. (b) Give a numbered proof of what can be concluded about arming and about being airborne. (c) Convert R1 ∧ R2 ∧ R3 to CNF.

**P4.** Does {p ∨ q, p → r, q → r} ⊨ r? Does {p ∨ q, p → r} ⊨ r? For whichever one fails, give a countermodel.

**P5.** Fleet {E, L, Y}. E is armed, airborne, has a fix. L is armed, not airborne, no fix. Y is not armed, not airborne, has a fix. Evaluate: (a) ∀x (Airborne(x) → HasFix(x)); (b) ∀x (Armed(x) → HasFix(x)); (c) ∃x (HasFix(x) ∧ ¬Armed(x)); (d) ¬∃x (Airborne(x) ∧ ¬Armed(x)). For each false one, name the counterexample.

**P6.** Write "every armed aircraft that is airborne has a GPS fix and is inside the geofence" in predicate logic. Then write its negation with every ¬ on a predicate. Then say in one sentence what a monitor should search for.

**P7.** The rule "if the battery is low, return to launch" was written for one aircraft in section 1.2. Rewrite it as a predicate-logic sentence about a fleet. What does the ∀ add that the one-aircraft version did not have?

---

## Where to read more

The best free text for sections 3 and 4 is *forall x: Calgary* (Magnus, Button, Loftis, Trueman, Thomas-Bolduc, and Zach), available at forallx.openlogicproject.org. It has exercises with solutions and a matching online proof checker. Chapter 1 of Rosen's *Discrete Mathematics and Its Applications* (8th edition, 2019) covers all four sections at this level with many more drill problems. Huth and Ryan's *Logic in Computer Science* (2nd edition, 2004) is the book to read next, since it goes from propositional and predicate logic straight into the temporal logic and model checking of Part 3.

If you want to see where the ideas came from, Boole's *An Investigation of the Laws of Thought* (1854) is the origin of Boolean algebra, Augustus De Morgan's *Formal Logic* (1847) is where the De Morgan laws were stated, Shannon's "A symbolic analysis of relay and switching circuits" (*Transactions of the AIEE*, 1938) is the paper that connected Boolean algebra to circuits, and Frege's *Begriffsschrift* (1879) introduced quantifiers. None of these is required reading.

## References

Boole, G. (1854). *An Investigation of the Laws of Thought, on Which are Founded the Mathematical Theories of Logic and Probabilities*. Walton and Maberly, London.

De Morgan, A. (1847). *Formal Logic: or, The Calculus of Inference, Necessary and Probable*. Taylor and Walton, London.

Frege, G. (1879). *Begriffsschrift, eine der arithmetischen nachgebildete Formelsprache des reinen Denkens*. Louis Nebert, Halle.

Huth, M. and Ryan, M. (2004). *Logic in Computer Science: Modelling and Reasoning about Systems*, 2nd edition. Cambridge University Press.

Magnus, P. D., Button, T., Loftis, J. R., Trueman, R., Thomas-Bolduc, A., and Zach, R. (2023). *forall x: Calgary. An Introduction to Formal Logic*, Fall 2023 edition. Open Logic Project. forallx.openlogicproject.org. CC BY 4.0.

Rosen, K. H. (2019). *Discrete Mathematics and Its Applications*, 8th edition. McGraw-Hill.

Shannon, C. E. (1938). A symbolic analysis of relay and switching circuits. *Transactions of the American Institute of Electrical Engineers*, 57(12), 713–723.

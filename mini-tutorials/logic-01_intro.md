# Logic for Autonomous Systems

## Part 1: Boolean logic, propositional logic, and predicates

This is the first of three parts. Part 1 builds the foundation: Boolean logic, which works with true and false; propositional logic, which uses those values to check whether an argument holds up; and predicates with quantifiers, which let us make statements about individual aircraft and about a whole fleet. Part 2 covers first-order logic in full. Part 3 covers temporal logic, which is how we write rules about what a system must do over time.

You do not need any prior logic course. If you have written an `if` statement with `&&` or `||` in it, you already know more of this than you think.

Each section has worked problems marked **Worked problem**, with the solution right after, so you can attempt it first and then check. Each section ends with exercises for you to do on your own. An answer key for those exercises and a set of mixed problems are at the end.

Every example is about small uncrewed aircraft, since that is what the rest of the course is about.

---

## 1. Why logic

### 1.1 The one question

Logic is about what follows from what. If you know some things, what else are you forced to accept? Every system in this tutorial answers that question for a different kind of statement. Boolean logic handles plain true-or-false values. Propositional logic handles sentences built from those values. Predicate logic handles sentences about objects, like "every aircraft in the fleet has a GPS fix." Temporal logic, in Part 3, handles sentences about time, like "the aircraft never leaves the geofence."

The reason an aviation course spends time on this is that a flight controller is a machine for deciding what follows from its inputs. Its arming interlock is a Boolean expression. Its failsafe rules are propositional logic. The requirements document that says what it must never do is, once you write it carefully, a set of logical formulas. When we say a system is "safe," what we usually mean is that its behavior follows from its rules, and logic is the tool for checking that.

The same three questions come up in every section, so it helps to name them now. First, how do you write a statement so that it is unambiguous? That is syntax. Second, when is a statement true? That is semantics. Third, how do you get from statements you have to new statements you are allowed to conclude? That is proof. Most confusion in a first logic course comes from mixing these up, so try to notice which one you are doing at any moment.

### 1.2 A running example

We will keep returning to six statements about a single quadrotor. Each one is either true or false at a given moment.

- armed: the flight controller has been armed
- airborne: the aircraft has left the ground
- gps_fix: the GPS has a valid position fix
- in_geofence: the current position is inside the mission geofence
- low_batt: the battery is below the return threshold
- rtl: the aircraft is in return-to-launch mode

In section 2 these are Boolean variables. In section 3 they are propositions. In section 4 we open them up so we can talk about many aircraft at once.

**Worked problem 1.1.** A field procedure says: "Do not arm the aircraft unless the GPS has a fix and the position is inside the geofence." Which of the six statements does this rule use? Can you check it from a single snapshot of the system, or do you need the whole flight history?

*Solution.* It uses armed, gps_fix, and in_geofence. It is about the moment of arming, so a single snapshot of those three values is enough to check it. That makes it a job for Boolean and propositional logic. If the rule had said "the aircraft must never be armed without a fix," the word "never" would mean checking every moment of the flight, which is what temporal logic is for.

### 1.3 Exercises

1. Rewrite this as "given these facts, this must follow": "The aircraft is armed and airborne. Any aircraft that is armed and airborne must have a GPS fix. So the aircraft has a fix."
2. For each of the following, say whether you can check it from one snapshot or need the flight history. (a) The battery is low. (b) The battery has been low for ten seconds. (c) The aircraft is in the geofence. (d) The aircraft never left the geofence.
3. Which of the three words syntax, semantics, or proof best describes each activity: (a) checking that a formula has balanced parentheses; (b) deciding whether a formula is true given sensor values; (c) deriving a new rule from two existing rules.

---

## 2. Boolean logic

Boolean logic works with exactly two values. We write them 1 and 0, or true and false, or T and F. Which symbols you use makes no difference. The whole subject is about how to combine those two values.

George Boole worked this out in the 1850s as a piece of pure mathematics. Almost a century later, Claude Shannon showed in his master's thesis that electrical switching circuits obey exactly Boole's rules, which is why every digital computer, including the one on your aircraft, is Boolean logic built in silicon.

### 2.1 Variables and expressions

A Boolean variable is a name for a value that is 0 or 1. We use p, q, r, and the six names from section 1.2. A Boolean expression combines variables with operations. Once you give every variable a value, the expression works out to a single 0 or 1, just like an arithmetic expression works out to a single number.

### 2.2 The three basic operations

There are three operations to learn cold. Everything else is built from them.

NOT, written ¬p, flips the value. NOT 1 is 0 and NOT 0 is 1. In code this is `!p`.

AND, written p ∧ q, is 1 only when both p and q are 1. In code this is `p && q`.

OR, written p ∨ q, is 1 when at least one of p or q is 1. In code this is `p || q`. Note that OR is also 1 when both are 1. Everyday English often uses "or" to mean "one or the other but not both," and that is a different operation we meet below.

A truth table lists every possible combination of input values and the output for each. This is the single most useful tool in the whole tutorial, so make sure you can build one. Here are the three basic operations:

| p | q | ¬p | p ∧ q | p ∨ q |
|---|---|----|-------|-------|
| 0 | 0 | 1  | 0     | 0     |
| 0 | 1 | 1  | 0     | 1     |
| 1 | 0 | 0  | 0     | 1     |
| 1 | 1 | 0  | 1     | 1     |

To build a truth table, list the variables, then write the rows by counting in binary from all zeros to all ones. Two variables give four rows. Three variables give eight. In general n variables give 2^n rows. Counting in binary guarantees you do not skip a row.

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

Keep this table in mind. It comes back in section 2.4.

### 2.3 Four more operations

Four other operations show up all the time. Each one can be built from NOT, AND, and OR.

XOR, "exclusive or," written p ⊕ q, is 1 when the inputs are different. This is the "one or the other but not both" meaning. You can write it as (p ∨ q) ∧ ¬(p ∧ q).

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

*Solution.* healthy = manual ⊕ auto. That is 1 when exactly one mode is on. fault = ¬(manual ⊕ auto), which is 1 when both are on or neither is on. Either of those is a real fault: both on means two systems are fighting for control, neither on means nobody is flying.

### 2.4 Making sense of implication

Look at the p → q column again. It is 0 in exactly one row, where p = 1 and q = 0. In the two rows where p = 0, the implication is 1 no matter what q is.

The way to make this feel right is to read p → q as a promise: "if p happens, then q will happen." The only way to break that promise is for p to happen and q not to. If p never happens, you have not broken anything. The rule "armed → gps_fix" is not violated by an aircraft sitting disarmed on the bench with no GPS fix, because the rule only makes a claim about what happens when the aircraft is armed.

Now compare the p → q column to the ¬p ∨ q column from worked problem 2.2. They are identical. So p → q is just another way of writing ¬p ∨ q. This is worth memorizing, because it lets you get rid of arrows whenever they are inconvenient.

Two more facts about implication. The converse of p → q is q → p, and it is a different statement with a different truth table. The contrapositive of p → q is ¬q → ¬p, and it has the same truth table as p → q. Mixing up the converse and the contrapositive is one of the most common mistakes in logic.

The biconditional, written p ↔ q and read "p if and only if q," is 1 when p and q have the same value. It is the same as (p → q) ∧ (q → p).

**Worked problem 2.4.** The rule is "if the battery is low, then return to launch," so low_batt → rtl. Right now low_batt = 0 and rtl = 1. Is the rule satisfied? What if low_batt = 1 and rtl = 0?

*Solution.* First case: low_batt is 0, so the "if" part is false and the implication is 1. The rule is satisfied. The aircraft is returning for some other reason, which the rule does not forbid. Second case: low_batt = 1 and rtl = 0. That is the one bad row. The rule is violated.

**Worked problem 2.5.** A student reads "armed → gps_fix" as "if the aircraft has a fix then it is armed." What did they get wrong, and why does it matter?

*Solution.* They read the converse. The rule says arming requires a fix. Their reading says a fix causes arming, which would mean every aircraft with a working GPS is armed, including the ones sitting on the shelf. Converses are not free.

### 2.5 The laws of Boolean algebra

You can rewrite a Boolean expression into a different expression that always has the same value, the same way you can rewrite 2(x + 3) as 2x + 6. The rules for doing so are called the laws of Boolean algebra. Each one can be checked with a truth table, and you should do that at least once for each law.

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

Notice that each law comes as a pair. Swap ∧ with ∨ and 0 with 1 and you get the partner law. That is a nice check on your memory.

De Morgan's laws are the ones you will use most. They tell you how to push a NOT inside a bracket: the NOT goes onto each piece, and the AND becomes OR or the OR becomes AND. "It is not true that the aircraft is armed and airborne" is the same as "it is not armed, or it is not airborne." If you have ever simplified `!(a && b)` to `!a || !b` in code, you have used De Morgan.

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

Sometimes you know what output you want for every input and need an expression that produces it. There is a mechanical way to do that. Find every row where the output is 1. For each such row, write an AND term that is true only on that row, using the variable if it is 1 in that row and the negated variable if it is 0. Then OR all the terms together.

**Worked problem 2.8.** Find an expression for f from this table, then simplify it.

| p | q | f |
|---|---|---|
| 0 | 0 | 1 |
| 0 | 1 | 1 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

*Solution.* The 1 rows are (0,0), (0,1), and (1,1). The terms are ¬p ∧ ¬q, ¬p ∧ q, and p ∧ q. So f = (¬p ∧ ¬q) ∨ (¬p ∧ q) ∨ (p ∧ q). The first two terms simplify to ¬p, as in worked problem 2.6, giving f = ¬p ∨ (p ∧ q). Distribute: (¬p ∨ p) ∧ (¬p ∨ q) = 1 ∧ (¬p ∨ q) = ¬p ∨ q. That is p → q. The table was implication in disguise.

For expressions with many variables, engineers use tools like Karnaugh maps or software to do this simplification. For this course the laws above are enough.

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

### 2.8 Exercises

1. Build the truth table for p ∧ ¬q.
2. Build the truth table for (p ∧ q) ∨ ¬r. It has eight rows. Work one column at a time.
3. Use De Morgan to rewrite ¬(p ∨ ¬q) so that no NOT sits outside a bracket.
4. Use a truth table to show that p → q and q → p have different columns. Which rows differ?
5. Use a truth table to show that p → q and ¬q → ¬p have the same column.
6. Write OR using only NAND. Hint: first write p ∨ q using De Morgan as ¬(¬p ∧ ¬q), then remember that NAND of a thing with itself is NOT of that thing.
7. Simplify (p ∨ q) ∧ (p ∨ ¬q) to a single variable, naming each law.
8. A landing interlock permits landing when the aircraft has a GPS fix, is inside the geofence, and is not in return-to-launch mode, or when the battery is low regardless of anything else. Write the permit condition. Then write the "landing refused" condition with every NOT pushed onto a single variable.
9. Translate into a Boolean expression and give the truth table: "The aircraft is idle when it is neither armed nor airborne."

---

## 3. Propositional logic

Propositional logic is Boolean logic with the values reinterpreted. Instead of abstract 1s and 0s, the values are now the truth or falsity of actual sentences. The math is exactly the same. What changes is the question. Boolean logic asks "what does this expression compute?" Propositional logic asks "does this conclusion follow from these premises?"

### 3.1 Propositions

A proposition is a sentence that is definitely true or definitely false. "The airport is closed" is a proposition. "Close the runway" is not, because a command is neither true nor false. "Is the runway closed?" is not, because a question has no truth value either. "x is above 120 metres" is not a proposition on its own, because you cannot say whether it is true until you know what x is. Section 4 fixes that last case.

An atomic proposition is one you cannot break into smaller propositions. We give each one a name. A compound proposition is built from atomic ones using connectives, which are the Boolean operations under their logic names: negation ¬, conjunction ∧, disjunction ∨, implication →, and biconditional ↔.

**Worked problem 3.1.** Which of these are propositions? (a) "The aircraft is armed." (b) "Arm the aircraft." (c) "The aircraft is armed and it is not armed."

*Solution.* (a) yes. (b) no, it is a command. (c) yes. It happens to be always false, but always false is still a definite truth value.

### 3.2 Writing formulas correctly

Not every string of symbols means something. p ∧ ∨ q is nonsense. A well-formed formula is one built by these rules: every atomic proposition is a formula; if A is a formula, so is ¬A; if A and B are formulas, so are (A ∧ B), (A ∨ B), (A → B), and (A ↔ B). Nothing else counts.

Writing every parenthesis gets cluttered, so we use precedence rules, just as in arithmetic where multiplication is done before addition. Here ¬ binds tightest, then ∧, then ∨, then →, then ↔. So p ∨ q ∧ r means p ∨ (q ∧ r), and ¬p → q means (¬p) → q. When in doubt, add parentheses. Nobody has ever been marked down for too many.

**Worked problem 3.2.** Add full parentheses to ¬p ∧ q → r ∨ ¬s.

*Solution.* Handle ¬ first: (¬p) ∧ q → r ∨ (¬s). Then ∧: ((¬p) ∧ q) → r ∨ (¬s). Then ∨: ((¬p) ∧ q) → (r ∨ (¬s)). The outermost connective is →, so this formula is an implication.

### 3.3 Evaluating a formula

To find the truth value of a formula, assign a value to each atomic proposition and work outward using the truth tables from section 2. A full assignment of values to the atoms is called a valuation. If a formula has n atoms, there are 2^n valuations, one per truth-table row.

**Worked problem 3.3.** Evaluate (armed → gps_fix) ∧ (¬gps_fix → ¬airborne) when armed = 1, gps_fix = 0, airborne = 0.

*Solution.* armed → gps_fix is 1 → 0, which is 0. Since one side of an AND is 0, the whole formula is 0 and we can stop. (For the record, the other side is ¬0 → ¬0, which is 1 → 1, which is 1.) The rules are violated because the aircraft is armed without a fix.

### 3.4 Tautology, contradiction, contingency

A formula that is true under every valuation is a tautology. The simplest is p ∨ ¬p: whatever p is, either it or its negation holds. A formula that is false under every valuation is a contradiction; p ∧ ¬p is the simplest. A formula that is true under some valuations and false under others is a contingency. Almost every formula you write about a real system is a contingency, because it says something that could go either way depending on the sensor values.

**Worked problem 3.4.** Is (p → q) ∨ (q → p) a tautology?

*Solution.* Try to make it false. Both halves would need to be 0. p → q is 0 only when p = 1 and q = 0. q → p is 0 only when q = 1 and p = 0. You cannot have both at once. So no valuation makes it false, and it is a tautology. It is a strange one: for any two propositions at all, one implies the other. That is a side effect of the "promise" reading of implication, and it is a good reminder that → does not mean "causes."

### 3.5 Logical equivalence

Two formulas are logically equivalent, written A ≡ B, when they have the same truth value under every valuation, which is to say the same truth-table column. All the laws from section 2.5 are equivalences. Three more involving implication are used constantly:

- p → q ≡ ¬p ∨ q
- p → q ≡ ¬q → ¬p (contrapositive)
- ¬(p → q) ≡ p ∧ ¬q

The third one is the most useful and the least obvious. To say an implication is false is to say the "if" part happened and the "then" part did not.

**Worked problem 3.5.** A manual says R1: "if the battery is low, the aircraft returns to launch." A reviewer rewrites it as R2: "if the aircraft is not returning to launch, the battery is not low." Do R1 and R2 say the same thing?

*Solution.* R1 is low_batt → rtl. R2 is ¬rtl → ¬low_batt. R2 is the contrapositive of R1, so yes, they are equivalent. If the reviewer had instead written "if the aircraft is returning to launch, the battery is low," that would be rtl → low_batt, the converse, which is a different and stronger rule. It would forbid returning to launch for any other reason.

### 3.6 Does the conclusion follow?

Now the main event. A set of premises entails a conclusion when every valuation that makes all the premises true also makes the conclusion true. We write it with a double turnstile: {A, B} ⊨ C means A and B together entail C.

An argument is valid when its premises entail its conclusion. Validity is about the shape of the argument, not about whether the premises happen to be true today. An argument can be valid even if its premises are false. And an argument can be invalid even if the premises and conclusion all happen to be true, because "happens to be true" is not the same as "follows from."

To show an argument is invalid, find one valuation that makes every premise true and the conclusion false. That valuation is called a countermodel. To show an argument is valid, either check every row of the truth table, or argue directly that the premises force the conclusion.

**Worked problem 3.6.** Does {armed → gps_fix, gps_fix → in_geofence, armed} ⊨ in_geofence?

*Solution.* Suppose all three premises are true. armed is true, and armed → gps_fix is true, so gps_fix must be true (otherwise the implication would be 1 → 0). Now gps_fix is true and gps_fix → in_geofence is true, so in_geofence must be true. Every valuation that satisfies the premises satisfies the conclusion. Yes, it follows.

**Worked problem 3.7.** Does {armed → gps_fix, gps_fix} ⊨ armed?

*Solution.* Look for a countermodel. Try armed = 0. Then armed → gps_fix is 0 → anything, which is 1. Set gps_fix = 1 so the second premise holds. The conclusion armed is 0. Both premises true, conclusion false. So no, it does not follow. This mistake, reasoning backward from the "then" to the "if," is called affirming the consequent. "If it is armed it has a fix; it has a fix; so it is armed" sounds reasonable and is wrong.

### 3.7 Standard shapes for formulas

It is often useful to force a formula into a standard shape. A literal is an atom or a negated atom, like p or ¬q. A clause is a bunch of literals joined by ∨.

A formula is in conjunctive normal form, CNF, when it is clauses joined by ∧. A formula is in disjunctive normal form, DNF, when it is AND-terms joined by ∨. Section 2.6 already produced a DNF from a truth table. Every formula can be put into either form by the same three moves: replace → and ↔ using the equivalences above, push every ¬ inward with De Morgan and double negation, then distribute.

CNF matters because it is the input format for SAT solvers, programs that decide whether a formula can be made true. Modern SAT solvers handle formulas with millions of clauses, and they are how industrial verification tools check that a design meets its specification. Deciding satisfiability in general is a famously hard problem, but in practice the solvers are remarkably fast.

**Worked problem 3.8.** Convert (p ∧ q) ∨ ¬r to CNF.

*Solution.* Distribute the ∨ over the ∧: (p ∨ ¬r) ∧ (q ∨ ¬r). Two clauses, each a disjunction of literals. That is CNF.

**Worked problem 3.9.** Convert ¬(p → q) ∨ r to DNF.

*Solution.* ¬(p → q) becomes p ∧ ¬q. So the formula is (p ∧ ¬q) ∨ r. That is two terms joined by ∨, where r counts as a one-literal term. That is DNF.

### 3.8 Proofs: getting to the conclusion step by step

Truth tables always work but they double in size with every atom. Ten atoms means 1,024 rows. A proof gets to the conclusion by applying small rules one step at a time instead. Each rule is a pattern: if you already have lines that match the "from" column, you may write down the "infer" line.

| Rule | From | Infer |
|------|------|-------|
| Modus ponens | A, A → B | B |
| Modus tollens | ¬B, A → B | ¬A |
| Hypothetical syllogism | A → B, B → C | A → C |
| Disjunctive syllogism | A ∨ B, ¬A | B |
| Conjunction introduction | A, B | A ∧ B |
| Conjunction elimination | A ∧ B | A (or B) |

Modus ponens is "the if-part happened, so the then-part happens." Modus tollens is "the then-part did not happen, so the if-part did not either." Hypothetical syllogism chains two implications. Disjunctive syllogism says that if one of two options is ruled out, the other holds.

When you can get from premises to a conclusion using these rules, we write it with a single turnstile: Γ ⊢ C, read "Γ proves C."

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
4. low_batt (disjunctive syllogism on 1 and 2, since gps_fix rules out ¬gps_fix)
5. rtl (modus ponens on 3 and 4)

### 3.9 Why you can trust a proof

There are two ways to establish that a conclusion follows: check the meaning (⊨, truth tables) or push symbols (⊢, proof rules). The good news is that for propositional logic these always agree. Every proof leads to a conclusion that really is entailed, which is called soundness, and every entailed conclusion has a proof, which is called completeness. So if you find a proof, the conclusion follows. If you can show no proof exists, there is a countermodel out there. You never have to worry that the two methods disagree.

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
| ≡ | equivalent | same truth table column |

### 3.11 Exercises

1. Which of these are propositions? (a) "Runway 27 is closed." (b) "Please close runway 27." (c) "Runway 27 is closed or it is open." (d) "The wind is above 15 knots." (e) "The wind speed is x."
2. Add full parentheses to p → q ∨ r ∧ ¬s.
3. Evaluate (low_batt → rtl) ∧ (rtl → ¬airborne) when low_batt = 1, rtl = 1, airborne = 1.
4. Is (p ∧ (p → q)) → q a tautology? Try to make it false.
5. Convert (p ∧ q) ∨ (¬p ∧ r) into CNF.
6. Does {p → q, q → r, p} ⊨ r? Justify.
7. Does {p → q, ¬p} ⊨ ¬q? If not, give a countermodel and say in English why the reasoning is wrong.
8. Give an argument that is valid but has a false conclusion. Explain why this forces at least one premise to be false.
9. A checklist says: "If the aircraft is airborne, it is armed. If it is armed, it has a fix. The aircraft has no fix." What can you conclude about whether it is airborne? Write the premises in symbols and give a numbered proof.
10. Explain in one or two sentences why "the premises entail C" means the same as "the premises together with ¬C cannot all be true at once."

---

## 4. Predicates and quantifiers

Propositional logic treats "Earhart is airborne" as a single unbreakable atom. It cannot see that the sentence is about a thing (the aircraft Earhart) and a property (being airborne). So it cannot express "every aircraft in the fleet is airworthy," because there is no atom for "every." Predicates and quantifiers add exactly that ability. This section introduces them. Part 2 builds the full system.

### 4.1 Predicates

A predicate is a sentence with a blank in it. Fill the blank with an object and you get a proposition. Write Airborne(x) for "x is airborne." By itself Airborne(x) is neither true nor false, because x is a placeholder. Airborne(Earhart) is a proposition.

If you like programming analogies, a predicate is a function that returns a Boolean. `isAirborne(aircraft)` returns true or false once you pass it a particular aircraft.

A predicate can have more than one blank. The number of blanks is its arity. Airborne(x) has arity 1. HigherThan(x, y), "x is at a higher altitude than y," has arity 2. A one-blank predicate describes a property of one thing; a two-blank predicate describes a relationship between two things.

**Worked problem 4.1.** Name the predicate and its arity: (a) "the aircraft has a GPS fix"; (b) "aircraft A is within 50 metres of aircraft B"; (c) "waypoint W is on the route from A to B."

*Solution.* (a) HasFix(x), arity 1. (b) Within50(x, y), arity 2. (c) OnRoute(w, a, b), arity 3. In (b), swapping A and B does not change the truth value. In HigherThan(x, y) it does. Order matters for some relations and not others.

### 4.2 The domain

A predicate only makes sense relative to a domain, the set of things you are talking about. If the domain is the six aircraft in the fleet, then x ranges over those six and nothing else. Choosing the domain is part of setting up the problem. "Everything is airworthy" is a very different claim over the fleet than over every object in the hangar.

A constant names one specific thing in the domain, like Earhart. A variable like x stands for an unspecified thing. Keep the difference in mind. It becomes important in the next subsection.

### 4.3 The two quantifiers

Quantifiers are how you say "all" and "some."

The universal quantifier ∀ is read "for all." ∀x Airworthy(x) says every aircraft in the domain is airworthy. One aircraft that is not airworthy makes the whole statement false. That aircraft is called a counterexample.

The existential quantifier ∃ is read "there exists." ∃x Airborne(x) says at least one aircraft is airborne. One airborne aircraft makes the whole statement true. That aircraft is called a witness.

Here is a way to see the quantifiers that makes everything else in this section easier. If the domain is a finite list of things a₁ through aₙ, then ∀x P(x) is just P(a₁) ∧ P(a₂) ∧ … ∧ P(aₙ), a big AND, and ∃x P(x) is just P(a₁) ∨ P(a₂) ∨ … ∨ P(aₙ), a big OR. Over a fleet of three aircraft, "all are airworthy" means "E is airworthy and L is airworthy and Y is airworthy." A quantifier is a way of writing that without knowing in advance how long the list is.

**Worked problem 4.2.** The fleet is {Earhart, Lindbergh, Yeager}. Only Earhart is airborne. All three are airworthy. Evaluate ∀x Airworthy(x), ∃x Airborne(x), ∀x Airborne(x), and ∃x ¬Airworthy(x).

*Solution.* ∀x Airworthy(x): true, all three pass. ∃x Airborne(x): true, Earhart is a witness. ∀x Airborne(x): false, Lindbergh is a counterexample. ∃x ¬Airworthy(x): false, no aircraft fails airworthiness.

### 4.4 "Every airborne aircraft": restricted quantifiers

Most useful sentences are not about every thing in the domain but about every thing of a certain kind. "Every airborne aircraft has a fix" is only about the airborne ones. The two quantifiers handle this differently, and this is the single most common mistake in translating English to logic, so slow down here.

A universal statement about a kind uses →:

∀x (Airborne(x) → HasFix(x))

"For every x, if x is airborne then x has a fix." Aircraft on the ground make the "if" part false, so they satisfy the implication automatically and do not count against the claim. That is exactly what we want.

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

### 4.8 Exercises

1. Domain: the fleet. Predicates Airborne(x), Airworthy(x), HasFix(x). Write in symbols: (a) every airborne aircraft is airworthy; (b) some airworthy aircraft is not airborne; (c) no aircraft is airborne without a fix; (d) every aircraft is airborne or airworthy, or both.
2. Translate into English: ∃x (Airborne(x) ∧ ¬Airworthy(x)). Why would an operator care?
3. Negate ∀x (Armed(x) → Airborne(x)) and simplify until every ¬ sits directly on a predicate.
4. Negate ∃x (LowBatt(x) ∧ ¬RTL(x)) and simplify. Say the result in English.
5. Fleet {E, L, Y}. E is armed with a fix. L is armed with no fix. Y is not armed and has a fix. Evaluate ∀x (Armed(x) → HasFix(x)) and ∃x (HasFix(x) ∧ ¬Armed(x)). Name the counterexample or witness.
6. Give a domain and a predicate for which ∀x P(x) is false but ∃x P(x) is true.
7. A student writes "every armed aircraft has a fix" as ∀x (Armed(x) ∧ HasFix(x)). Give a small fleet where the English sentence is true but the student's formula is false.
8. With the fleet {E, L, Y}, write ∀x (Armed(x) → HasFix(x)) as a propositional formula with no quantifiers, using atoms like Armed_E and HasFix_E.
9. Explain the difference between ∀x ∃y HigherThan(y, x) and ∃y ∀x HigherThan(y, x) in plain English.

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

## Answer key for section exercises

### Section 1

1. Facts: armed ∧ airborne, and (armed ∧ airborne) → gps_fix. Must follow: gps_fix. This is modus ponens.
2. (a) snapshot; (b) history; (c) snapshot; (d) history.
3. (a) syntax; (b) semantics; (c) proof.

### Section 2

1. Rows (p, q): 00→0, 01→0, 10→1, 11→0.
2. Rows (p, q, r): 000→1, 001→0, 010→1, 011→0, 100→1, 101→0, 110→1, 111→1.
3. ¬(p ∨ ¬q) = ¬p ∧ ¬¬q = ¬p ∧ q.
4. p → q gives 1, 1, 0, 1. q → p gives 1, 0, 1, 1. They differ on rows 01 and 10.
5. Both columns are 1, 1, 0, 1.
6. p ∨ q = ¬(¬p ∧ ¬q) = NAND(¬p, ¬q) = NAND(NAND(p, p), NAND(q, q)).
7. (p ∨ q) ∧ (p ∨ ¬q) = p ∨ (q ∧ ¬q) by distributivity = p ∨ 0 by complement = p by identity.
8. permit = (gps_fix ∧ in_geofence ∧ ¬rtl) ∨ low_batt. refused = (¬gps_fix ∨ ¬in_geofence ∨ rtl) ∧ ¬low_batt.
9. idle = ¬armed ∧ ¬airborne, or equivalently ¬(armed ∨ airborne). Table: 00→1, 01→0, 10→0, 11→0.

### Section 3

1. (a) yes; (b) no, a request; (c) yes, and it is a tautology; (d) yes; (e) no, it depends on x.
2. p → (q ∨ (r ∧ (¬s))).
3. low_batt → rtl is 1 → 1 = 1. rtl → ¬airborne is 1 → 0 = 0. The AND is 0.
4. To make it false you need p ∧ (p → q) true and q false. p = 1 and q = 0 makes p → q false, so the antecedent fails. No falsifying valuation; it is a tautology (it is modus ponens written as a formula).
5. (p ∨ r) ∧ (¬p ∨ q) ∧ (q ∨ r). (The clause p ∨ ¬p also appears when you distribute but is always true and can be dropped.)
6. Yes. p and p → q give q. q and q → r give r.
7. No. Countermodel: p = 0, q = 1. Both premises true, ¬q false. In English: "if armed then fix; not armed; so no fix" ignores that the aircraft could have a fix for reasons that have nothing to do with being armed. This is called denying the antecedent.
8. Example: premises p → q and p, conclusion q, with p and q both false. It is valid by modus ponens. A valid argument with all true premises must have a true conclusion, so if the conclusion is false, some premise must be false.
9. airborne → armed, armed → gps_fix, ¬gps_fix. Modus tollens gives ¬armed. Modus tollens again gives ¬airborne. The aircraft is not airborne.
10. "Premises entail C" means there is no valuation with the premises true and C false. "Premises plus ¬C cannot all be true" says exactly the same thing: no valuation has the premises true and ¬C true, that is, C false.

### Section 4

1. (a) ∀x (Airborne(x) → Airworthy(x)). (b) ∃x (Airworthy(x) ∧ ¬Airborne(x)). (c) ∀x (Airborne(x) → HasFix(x)), or ¬∃x (Airborne(x) ∧ ¬HasFix(x)). (d) ∀x (Airborne(x) ∨ Airworthy(x)).
2. Some aircraft is flying while not airworthy. That aircraft should be on the ground.
3. ∃x (Armed(x) ∧ ¬Airborne(x)).
4. ∀x (LowBatt(x) → RTL(x)). Every low-battery aircraft is returning to launch.
5. ∀x (Armed(x) → HasFix(x)) is false; L is the counterexample. ∃x (HasFix(x) ∧ ¬Armed(x)) is true; Y is the witness.
6. Domain {E, L}, P = Airborne, E airborne and L not.
7. Fleet {E, L}, E armed with a fix, L not armed and no fix. The English is true, since the only armed aircraft has a fix. The formula is false because L is not armed.
8. (Armed_E → HasFix_E) ∧ (Armed_L → HasFix_L) ∧ (Armed_Y → HasFix_Y).
9. The first says every aircraft has some aircraft above it, possibly a different one each time. The second says one particular aircraft is above all the others. Over a real fleet the first is false, since the highest aircraft has nothing above it, while the second can be true.

---

## Where to read more

The single best free text for sections 3 and 4 is *forall x: Calgary* (Magnus, Button, Trueman, Zach, and Thomas-Bolduc, 2023), available at forallx.openlogicproject.org. It has exercises with solutions and a matching online proof checker. Chapter 1 of Rosen's *Discrete Mathematics and Its Applications* (8th edition, 2019) covers all four sections at this level with many more drill problems. Huth and Ryan's *Logic in Computer Science* (2nd edition, 2004) is the book to read next, since it goes from propositional and predicate logic straight into the temporal logic and model checking of Part 3.

If you want to see where the ideas came from, Boole's *An Investigation of the Laws of Thought* (1854) is the origin of Boolean algebra, Augustus De Morgan's *Formal Logic* (1847) is where the De Morgan laws were stated, Shannon's "A symbolic analysis of relay and switching circuits" (*Transactions of the AIEE*, 1938) is the paper that connected Boolean algebra to circuits, and Frege's *Begriffsschrift* (1879) introduced quantifiers. None of these is required reading.

## References

Boole, G. (1854). *An Investigation of the Laws of Thought, on Which are Founded the Mathematical Theories of Logic and Probabilities*. Walton and Maberly, London.

De Morgan, A. (1847). *Formal Logic: or, The Calculus of Inference, Necessary and Probable*. Taylor and Walton, London.

Frege, G. (1879). *Begriffsschrift, eine der arithmetischen nachgebildete Formelsprache des reinen Denkens*. Louis Nebert, Halle.

Huth, M. and Ryan, M. (2004). *Logic in Computer Science: Modelling and Reasoning about Systems*, 2nd edition. Cambridge University Press.

Magnus, P. D., Button, T., Trueman, R., Zach, R., and Thomas-Bolduc, A. (2023). *forall x: Calgary. An Introduction to Formal Logic*, Fall 2023 edition. Open Logic Project. forallx.openlogicproject.org. CC BY 4.0.

Rosen, K. H. (2019). *Discrete Mathematics and Its Applications*, 8th edition. McGraw-Hill.

Shannon, C. E. (1938). A symbolic analysis of relay and switching circuits. *Transactions of the American Institute of Electrical Engineers*, 57(12), 713–723.

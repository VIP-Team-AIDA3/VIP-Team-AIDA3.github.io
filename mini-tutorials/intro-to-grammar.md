# Grammars: From Sentences to the Chomsky Hierarchy

**AIDA3 VIP Mini-Tutorial** | No prior background assumed

You do not need a course in linguistics, compilers, or theory of computation to read this. You need to be willing to think carefully about strings of symbols and the rules that produce them.

The word "grammar" probably makes you think of English class, and that is a fine place to start, but the idea is much more general. A grammar is a finite set of rules that generates a possibly infinite set of strings. Those strings can be English sentences, Spanish sentences, Python programs, C programs, MAVLink command sequences, radio calls to a control tower, or the output of a language model constrained to emit only valid UAV commands. The same mathematics covers all of them. That is the point of this tutorial.

---

## 1. Why Bother

Human languages are infinite. You have never heard the sentence "the fixed-wing aircraft circled the frozen reservoir twice before the battery warning cleared," but you understood it immediately. You did not memorize it. You built its meaning out of a finite vocabulary using a finite set of structural rules. Finite resources, infinite output. That is what a grammar captures.

Programming languages work the same way. There are infinitely many valid Python programs, and the interpreter accepts them all using a grammar that fits in a few pages. When you get a `SyntaxError`, you have handed the interpreter a string its grammar cannot generate. The error message is a grammar telling you no.

There is a third case that matters just as much to us. Command languages, mission plan formats, log formats, and radio phraseology are all languages with grammars. If you are going to have a language model generate commands for an aircraft, you had better be able to say precisely which strings are legal and which are not. That is a grammar problem, and Section 12 comes back to it.

---

## 2. Words Have Categories: Parts of Speech

Before you can talk about the structure of a sentence you need to talk about the pieces. Every word belongs to a category, called a part of speech, or POS.

| Tag | Category | Examples |
|---|---|---|
| NOUN | naming word | aircraft, runway, battery |
| PROPN | proper noun | Purdue, Earhart, Indiana |
| VERB | action or state | climbs, landed, is, transmit |
| ADJ | adjective, modifies a noun | low, autonomous, fixed-wing |
| ADV | adverb, modifies a verb or adjective | quickly, immediately, very |
| DET | determiner | the, a, this, every |
| ADP | adposition, mostly prepositions | over, on, from, to |
| PRON | pronoun | it, we, they |
| CCONJ | coordinating conjunction | and, or, but |
| NUM | numeral | two, 400, 118.7 |

**POS tagging** is the task of assigning the right tag to every word in a sentence. Here is a short one:

```
The     aircraft   climbed   above   the   geofence
DET     NOUN       VERB      ADP     DET   NOUN
```

That looks trivial, and for that sentence it is. It stops being trivial fast, because the same word can belong to different categories depending on context.

```
Land    the    aircraft            The     land    is     flat
VERB    DET    NOUN                DET     NOUN    VERB   ADJ
```

The word `land` is a verb in the first and a noun in the second. Nothing about the letters tells you which. Only the surrounding context does. This is **POS ambiguity**, and resolving it is the first real problem in natural language processing.

A famous harder case:

```
Time    flies    like    an    arrow
```

You probably read that as NOUN VERB ADP DET NOUN, meaning time passes quickly. But a second reading exists, VERB NOUN ADP DET NOUN, an instruction to time some flies the way you would time an arrow. And a third, NOUN NOUN VERB DET NOUN, a claim about a species called "time flies" that is fond of an arrow. All three are grammatically legal. Only world knowledge picks the first.

### 2.1 How Tagging Is Actually Done

Three approaches, in historical order.

**Rule-based tagging** uses hand-written rules such as "a word between a determiner and a verb is a noun." This works until it does not, and hand-writing enough rules for a real language is a research career.

**Statistical tagging** learns from a corpus of already-tagged sentences. The classic method is a hidden Markov model, which picks the tag sequence that maximizes the product of two things: how likely each tag is to produce that word, and how likely each tag is to follow the previous tag. That second factor is what fixes `land` in "land the aircraft." A sentence-initial verb followed by a determiner is a very common imperative pattern, while NOUN DET NOUN is not a legal English sequence at all.

**Neural tagging** feeds the whole sentence to a model that reads all the context at once and predicts a tag per token. This is what modern systems use, and accuracy on English is above 97 percent. The remaining 3 percent is not spread evenly. It concentrates on exactly the ambiguous cases that matter.

Tagging is worth understanding even if you never do it yourself, because it is the clearest small example of a pattern you will meet everywhere. Local evidence is not enough, context decides, and context is what makes the problem hard.

---

## 3. Two Ways to Describe Sentence Structure

Once every word has a category, you can ask how the words relate to each other. There are two dominant answers, and the difference between them explains a lot about why formal language theory looks the way it does.

### 3.1 Constituency Structure

Constituency, also called phrase structure, says that words group into phrases, phrases group into bigger phrases, and the whole sentence is the biggest phrase. The grouping is a tree whose internal nodes are phrase categories and whose leaves are words.

For "the aircraft climbed above the geofence":

```
                       S
                    /     \
                  NP       VP
                 /  \     /   \
              DET    N   V     PP
               |     |   |    /   \
                                ADP    NP
                                      /   \
                                   DET     N

              The aircraft climbed above the geofence
```

Read it bottom up. "The aircraft" is a noun phrase. "The geofence" is another. "Above the geofence" is a prepositional phrase. "Climbed above the geofence" is a verb phrase. NP plus VP makes a sentence.

The test for whether a group of words is really a constituent is whether you can move it, replace it with a single word, or use it alone as an answer. "Above the geofence" passes all three. You can say "Above the geofence, the aircraft climbed," and you can answer "Where did it climb?" with "Above the geofence." The string "climbed above the" fails every test. It is not a constituent, just three adjacent words.

Constituency structure is exactly what context-free grammars produce. Section 7.

### 3.2 Dependency Structure

Dependency grammar throws out the phrase nodes entirely. It says a sentence is a set of binary relations between words, where one word is the **head** and the other is the **dependent**. Every word has exactly one head, except a single root word, usually the main verb. Draw an arrow from head to dependent and label it with the relation type.

Same sentence, written as an indented tree because that is easier to read than arcs:

```
climbed                 root
├── aircraft            nsubj
│   └── The             det
└── geofence            obl
    ├── above           case
    └── the             det
```

`climbed` is the root. `aircraft` is its subject. `geofence` is an oblique modifier of `climbed`. The two `the`s attach to their nouns as determiners. Notice that `above` is not the head of anything, it hangs off `geofence` as a case marker.

Common labels, from the Universal Dependencies standard:

| Label | Relation |
|---|---|
| root | head of the whole sentence |
| nsubj | nominal subject |
| obj | direct object |
| iobj | indirect object |
| det | determiner attached to a noun |
| amod | adjective modifying a noun |
| advmod | adverb modifying a verb or adjective |
| nummod | numeral modifying a noun |
| compound | noun modifying another noun |
| case | preposition or postposition marking a noun |
| obl | oblique nominal, roughly a prepositional modifier of a verb |
| cc | the coordinating word itself |
| conj | second element of a coordination |

### 3.3 Why Anyone Would Prefer Dependencies

Three practical reasons.

First, a dependency tree has exactly as many nodes as there are words. There are no invented internal nodes to argue about. Two linguists can disagree about whether English really has a VP node. They cannot disagree about how many words are in the sentence.

Second, dependencies handle free word order languages much more gracefully. In Latin, Russian, or Hindi, the words of a phrase can be scattered across the sentence, which makes a contiguous constituent tree awkward or impossible. The subject is still the subject wherever it sits, so the dependency arrow survives.

Third, dependencies map almost directly onto meaning. "Who did what to whom" is exactly what the nsubj and obj arrows encode. If you are extracting structured information from text, a dependency parse hands you most of the answer already. That is why dependency parsing is standard in information extraction pipelines, and it is the right tool if you ever want to pull structured commands out of free-form operator text.

Constituency has its own advantage, and it is a big one. It corresponds directly to the rewriting grammars that the rest of this tutorial is about, and therefore to how compilers work. Neither view is right. They are two coordinate systems on the same structure, and mature systems convert between them.

### 3.4 How a Dependency Parser Works

You do not need this to follow the rest, but it demystifies the tools.

The dominant approach is **transition-based parsing**. Keep a stack, a buffer holding the remaining words, and a growing set of arcs. At each step choose one of three actions. SHIFT moves the next word from the buffer to the stack. LEFT-ARC makes the top of the stack the head of the item below it and removes the lower one. RIGHT-ARC does the reverse. A classifier, these days a neural network, looks at the current configuration and predicts which action to take. Repeat until the buffer is empty. The whole parse is linear in sentence length, which is why this scales.

The alternative is **graph-based parsing**. Score every possible head-dependent pair, then find the maximum spanning tree over the resulting weighted graph. More accurate on long sentences, slower, and it needs a global optimization rather than a sequence of local decisions.

### 3.5 An Ambiguity Worth Holding Onto

**"The operator watched the aircraft with the telescope."**

Does "with the telescope" attach to `watched`, meaning the operator used a telescope, or to `aircraft`, meaning the aircraft had a telescope mounted on it? Two different dependency trees, two different meanings, one string of words.

```
Reading A                       Reading B
watched         root            watched         root
├── operator    nsubj           ├── operator    nsubj
├── aircraft    obj             └── aircraft    obj
└── telescope   obl                 └── telescope  nmod
```

This is **prepositional phrase attachment ambiguity**, one of the most persistent problems in parsing. Hold onto it, because Section 8 shows the same disease in programming languages, where the stakes are higher.

---

## 4. From Real Sentences to Formal Grammars

Everything so far has been descriptive. Linguists observe sentences and propose structures. Now we flip it. Instead of describing sentences that already exist, we write down rules that **generate** strings, and study exactly what kinds of string sets different classes of rules can produce. This is formal language theory. Noam Chomsky started it in 1956 while trying to make the linguistics precise, and it became the mathematical foundation of compilers.

### 4.1 Alphabets, Strings, Languages

An **alphabet** is a finite set of symbols, written Σ. It might be `{0, 1}`, or the ASCII characters, or the set of English words, or the set of MAVLink command names. The symbols are whatever you decide the atoms are.

A **string** over Σ is a finite sequence of symbols from Σ. The **empty string**, written ε, has length zero. Σ\* is the set of all strings over Σ, including ε.

A **language** over Σ is any subset of Σ\*. That is the entire definition. A language is just a set of strings. "English" is the set of grammatical English sentences. "Python" is the set of syntactically valid Python programs. "Valid mission files" is the set of strings your mission parser accepts.

Most interesting languages are infinite, and we need a finite way to describe them. That is what a grammar is for.

### 4.2 The Formal Definition

A **grammar** is a four-tuple

```
G = (N, T, P, S)
```

where

**N** is a finite set of **non-terminal** symbols, the internal category names like S, NP, VP. By convention they are capitalized. They never appear in a finished string.

**T** is a finite set of **terminal** symbols, the actual output alphabet. By convention lowercase. N and T must be disjoint, so no symbol is both.

**P** is a finite set of **production rules**, each of the form α → β, where α and β are strings over N ∪ T and α contains at least one non-terminal. Read α → β as "α may be rewritten as β."

**S ∈ N** is the designated **start symbol**.

The requirement that α contain at least one non-terminal is what keeps rewriting meaningful. If α could be all terminals you could keep rewriting finished output forever.

### 4.3 Derivations

To generate a string, start with S and apply rules repeatedly. Each step finds an occurrence of some rule's left side in your current string and replaces it with the right side. Write one step as ⇒ and any number of steps as ⇒\*.

A tiny English grammar:

```
N = {S, NP, VP, DET, NOM, V}
T = {the, aircraft, tower, landed, contacted}
start = S

S   → NP VP
NP  → DET NOM
VP  → V NP  |  V
DET → the
NOM → aircraft  |  tower
V   → landed  |  contacted
```

The vertical bar is shorthand for two rules with the same left side. A derivation:

```
S
⇒ NP VP                          (S → NP VP)
⇒ DET NOM VP                     (NP → DET NOM)
⇒ the NOM VP                     (DET → the)
⇒ the aircraft VP                (NOM → aircraft)
⇒ the aircraft V NP              (VP → V NP)
⇒ the aircraft contacted NP      (V → contacted)
⇒ the aircraft contacted DET NOM
⇒ the aircraft contacted the NOM
⇒ the aircraft contacted the tower
```

No non-terminals remain, so we stop. The string is in the language generated by G.

The **language generated by G**, written L(G), is the set of all terminal strings derivable from S:

```
L(G) = { w ∈ T*  :  S ⇒* w }
```

Notice what else this grammar generates. "The tower contacted the aircraft" is fine. "The aircraft landed the tower" is nonsense but perfectly grammatical. Syntax and meaning are different things. A grammar constrains form, not sense. Chomsky's own example was "colorless green ideas sleep furiously," which is flawless English and completely meaningless. Keep this separation in mind, because it comes back hard in Section 12.

---

## 5. The Chomsky Hierarchy

Chomsky's insight was that if you restrict the **shape** of the production rules you restrict the class of languages you can generate, and the restrictions fall into a clean nested sequence of four levels.

| Type | Grammar | Rule restriction | Machine | Typical use |
|---|---|---|---|---|
| 3 | Regular | A → aB or A → Ba, plus A → a, A → ε | Finite automaton | Tokens, regex, protocols |
| 2 | Context-free | A → γ, one non-terminal on the left | Pushdown automaton | Programming language syntax |
| 1 | Context-sensitive | αAβ → αγβ, γ non-empty | Linear bounded automaton | Cross-referencing constraints |
| 0 | Unrestricted | α → β, α contains a non-terminal | Turing machine | Anything computable |

Containment is strict:

```
Regular  ⊊  Context-free  ⊊  Context-sensitive  ⊊  Recursively enumerable
```

Every regular language is context-free. Not every context-free language is regular. Section 7.1 shows why.

Work down the table, from most restricted to least. The restricted cases are both easier and more useful.

---

## 6. Type 3: Regular Grammars

A grammar is **right-regular** if every rule has one of these shapes:

```
A → a B        one terminal, then one non-terminal
A → a          one terminal
A → ε          empty
```

It is **left-regular** if the non-terminal always comes first instead, `A → B a`. A single grammar must pick one side and stay there. Mixing left-regular and right-regular rules in the same grammar can generate languages that are not regular, so the consistency requirement is not cosmetic.

The languages generated are exactly the **regular languages**, which are exactly the languages recognized by finite automata, which are exactly the languages described by regular expressions. Three different formalisms, one class of languages. That triple coincidence is the first real theorem of the subject, and it is why your regex library and your state machine diagram are interchangeable.

### 6.1 A UAV Example

A flight session is a sequence of mode events over `{d, a, f, l}` for disarm, arm, fly, land. A legal telemetry log is one or more complete sessions, where a session is disarm, arm, one or more fly ticks, land.

```
N = {S, P, Q, R}
T = {d, a, f, l}

S → d P
P → a Q
Q → f Q  |  f R
R → l S  |  l
```

Derive `d a f f l d a f l`:

```
S ⇒ dP ⇒ daQ ⇒ dafQ ⇒ daffR ⇒ dafflS
  ⇒ daffl dP ⇒ daffl daQ ⇒ daffl dafR ⇒ daffl dafl
```

The equivalent regular expression is `(d a f+ l)+`. The equivalent finite automaton has four states, one per non-terminal, with one arrow per rule. All three descriptions carry the same information and tools convert between them automatically.

### 6.2 What Regular Grammars Cannot Do

A finite automaton has finitely many states and no memory beyond which state it is currently in. That means it **cannot count without bound**. It can check that a string has an even number of `f`s, because that needs exactly one bit of memory. It cannot check that a string has as many `b`s as `e`s when there is no upper limit, because that needs a counter with unbounded range.

This is not a failure of cleverness. It is a theorem. And it is exactly why the next level exists.

---

## 7. Type 2: Context-Free Grammars

A grammar is **context-free** if every rule has the form

```
A → γ
```

where A is a single non-terminal and γ is any string over N ∪ T, possibly empty. The name comes from the fact that A can be rewritten as γ no matter what surrounds A. The context does not matter.

Every grammar in Section 4 was context-free. The constituency trees of Section 3.1 are exactly the derivation trees of context-free grammars. That is not a coincidence. It is where the formalism came from.

### 7.1 The Canonical Non-Regular Language

Consider a mission script with block structure, `b` to open a block and `e` to end one:

```
S → b S e  |  ε
```

This generates ε, `be`, `bbee`, `bbbeee`, and in general `bⁿeⁿ` for every n ≥ 0. Exactly n opens followed by exactly n closes.

No finite automaton can recognize this. The argument: suppose a machine with k states accepts it. Feed it more than k opening symbols. By the pigeonhole principle it must revisit some state, which means it cannot tell two different counts of opens apart, which means it will also accept some string with mismatched counts. Since it accepts a bad string it does not recognize the language, and the contradiction shows no such machine exists. That argument is the pumping lemma for regular languages, in casual clothes.

A context-free grammar handles it easily because a derivation is a tree, and the tree remembers how deep you went. The machine equivalent is a **pushdown automaton**, a finite automaton with a stack. Push on every `b`, pop on every `e`, accept if the stack is empty at the end. The stack is the unbounded counter that a finite automaton lacks.

### 7.2 This Is the Grammar Class for Programming Languages

Every mainstream programming language has a context-free grammar for its syntax, and it is usually published right in the language specification, written in **Backus-Naur Form**, or BNF, a notation for context-free rules invented for ALGOL 60 in 1959. Here is arithmetic:

```
<expr>   ::= <expr> "+" <term>  |  <expr> "-" <term>  |  <term>
<term>   ::= <term> "*" <factor>  |  <term> "/" <factor>  |  <factor>
<factor> ::= "(" <expr> ")"  |  <number>  |  <identifier>
```

The angle brackets mark non-terminals and `::=` is the arrow. That is a context-free grammar written in the notation compiler people actually use. If you open the Python language reference or the C standard, this is what you will find.

Why context-free and not regular? Because of `<factor> ::= "(" <expr> ")"`. Parentheses nest to arbitrary depth, and matching nested brackets is the `bⁿeⁿ` problem. The same goes for braces in C and Java, `begin`/`end` in Pascal, and indentation in Python once the lexer turns it into INDENT and DEDENT tokens. **Any language with nesting is beyond regular, permanently.**

This is also the honest answer to the perennial question of why you should not parse HTML with a regular expression. HTML tags nest. Regular expressions cannot count nesting depth. The tool is the wrong shape for the job, and no amount of effort on the regex will fix it.

A real compiler front end uses two levels of the hierarchy in sequence. The **lexer** uses regular expressions to chop the character stream into tokens such as identifiers, numbers, and operators, because tokens do not nest and regular matching is fast. The **parser** then uses a context-free grammar to build a tree out of those tokens. Type 3 followed by Type 2, in every compiler you have ever used. When you learn that lexing and parsing are separate phases, this hierarchy is the reason why.

### 7.3 Parse Trees and Precedence

The derivation tree, or parse tree, records which rule was used where. For `2 + 3 * 4` under the arithmetic grammar above:

```
                  expr
               /    |    \
           expr     +     term
            |            /  |  \
          term       term   *   factor
            |          |          |
         factor     factor        4
            |          |
            2          3
```

The tree groups `3 * 4` together, which is what gives multiplication higher precedence than addition. Precedence is not a separate feature bolted onto the parser. It falls straight out of the shape of the grammar, from the fact that `term` sits below `expr`. Similarly, `expr → expr + term` puts the recursion on the left, which makes `a - b - c` group as `(a - b) - c`, the correct left associativity for subtraction. Write the rule right-recursive instead and subtraction silently computes the wrong answer. Grammar design has consequences.

---

## 8. Ambiguity, and Why It Is a Safety Issue

A grammar is **ambiguous** if some string in its language has more than one parse tree. Section 3.5 showed this with "with the telescope." The same disease shows up in programming languages.

Take a naive conditional grammar:

```
<stmt> ::= "if" <cond> "then" <stmt>
        |  "if" <cond> "then" <stmt> "else" <stmt>
        |  <other>
```

Now parse `if A then if B then X else Y`. Does the `else` belong to the inner `if` or the outer one? Both parses are legal under this grammar, and they behave differently when A is true and B is false. Under one reading Y executes, under the other nothing does. This is the **dangling else** problem, and it is why C, Java, and their descendants specify by fiat that `else` binds to the nearest unmatched `if`, and why languages designed later require braces or an explicit `endif`.

The lesson generalizes, and this is the part to carry into your subteam work. **Ambiguity in a command language is a safety hazard.** If a mission specification string can be parsed two ways, the aircraft will do one of them and the operator will expect the other. When you design a command format, ambiguity is a defect in the format, not a quirk to be documented. Prefer explicit delimiters, prefer required keywords over optional ones, and if a grammar turns out ambiguous, fix the grammar rather than papering over it in the parser.

Two things worth knowing. Ambiguity is a property of a grammar, not of a language, and many ambiguous grammars can be rewritten into unambiguous grammars for the same language. But there also exist **inherently ambiguous** context-free languages for which no unambiguous grammar exists at all, and deciding whether an arbitrary context-free grammar is ambiguous is undecidable. There is no tool that will check this for you in general, so design carefully up front.

---

## 9. Type 1: Context-Sensitive Grammars

A grammar is **context-sensitive** if every rule has the form

```
α A β → α γ β
```

where A is a single non-terminal, γ is non-empty, and α and β are the surrounding context, either of which may be empty. Read it directly: A may be rewritten as γ, but only when it sits between α and β. Context now matters.

An equivalent definition that is easier to check: a grammar is context-sensitive if every rule α → β has |α| ≤ |β|, so no rule ever shortens the string. These are called **noncontracting** grammars and they generate the same class. The single rule S → ε is allowed as a special case if S never appears on the right of any rule, so that the empty string can be in the language.

### 9.1 The Canonical Example

The language `aⁿbⁿcⁿ`, equal counts of three different symbols, is context-sensitive but **not** context-free. Concretely: n waypoints, each needing exactly one altitude assignment and exactly one dwell time, with the three lists given separately and required to match in length. A pushdown automaton has one stack, and one stack can match two groups but not three.

```
S   → a S B C  |  a B C
C B → B C
a B → a b
b B → b b
b C → b c
c C → c c
```

Derive `aabbcc`:

```
S
⇒ a S B C          (S → a S B C)
⇒ a a B C B C      (S → a B C)
⇒ a a B B C C      (C B → B C)
⇒ a a b B C C      (a B → a b)
⇒ a a b b C C      (b B → b b)
⇒ a a b b c C      (b C → b c)
⇒ a a b b c c      (c C → c c)
```

Look at what `C B → B C` is doing. It is a sorting rule. The first two productions scatter Bs and Cs into the wrong order, and this rule bubbles them into place. Then the last four rules turn non-terminals into terminals, but only when the neighbor to the left is already the right kind of terminal. That left-neighbor requirement is the context sensitivity, and it is what enforces the ordering. Trace it by hand once and the mechanism becomes clear.

Context-sensitive grammars are painful to write. That is the honest summary. They are studied because they mark a real boundary in expressive power, not because anyone parses with them.

### 9.2 Where Context Sensitivity Shows Up in Programming Languages

You will rarely write a context-sensitive grammar, but you meet context-sensitive **constraints** constantly, and knowing why is genuinely useful.

These are the rules a parser cannot check:

A variable must be declared before it is used. Whether `x` is legal at this point depends on what appeared earlier, which is context.

The number of arguments at a call site must match the function's declaration. That is a matching-count requirement across arbitrary distance, the same shape as `aⁿbⁿcⁿ`.

Types must agree across an assignment.

In C, whether `a * b;` is a multiplication or a pointer declaration depends on whether `a` was typedef'd earlier in the file. The exact same token sequence parses two different ways depending on context arbitrarily far back.

None of these are context-free. So compilers do not attempt them in the parser. They parse with a context-free grammar and then run a separate **semantic analysis** pass, using a symbol table, to enforce the context-sensitive rules. That is why a C compiler reports a syntax error and an undeclared variable in two different phases, with different error styles, and why fixing a missing semicolon sometimes makes twenty type errors appear. The phase split in every compiler is the Chomsky hierarchy showing through the tooling.

Natural language has the same flavor of constraint. Subject-verb agreement, "the aircraft **is**" versus "the aircraft **are**," requires matching a feature across a possibly unbounded distance. Swiss German and Dutch have cross-serial dependencies that are provably not context-free, which settled a long argument about whether natural language needs more than context-free power. It does.

---

## 10. Type 0: Unrestricted Grammars

An **unrestricted** grammar, also called a recursively enumerable or phrase structure grammar, is the general case we started with:

```
G = (N, T, P, S)
```

with productions α → β where α and β are strings over N ∪ T and α contains at least one non-terminal. There is no other restriction. In particular β may be shorter than α, so rules may delete material. A rule like `A B → ε` is legal here and nowhere above.

Unrestricted grammars generate exactly the **recursively enumerable** languages, which are exactly the languages accepted by Turing machines. This is the ceiling. If a set of strings can be generated by any mechanical procedure at all, some unrestricted grammar generates it. If no unrestricted grammar generates it, no algorithm generates it either.

The catch is hiding in the word "accepted." A Turing machine for a recursively enumerable language will halt and say yes on strings in the language. On strings **not** in the language it may run forever without ever telling you no. That gap is the price of full power, and it is precisely why the lower levels of the hierarchy are worth caring about. Restricting expressive power buys you guarantees.

---

## 11. Machines, and What You Can Actually Decide

Each grammar class has a matching machine class. The correspondence is exact in both directions, which is why the hierarchy is considered natural rather than arbitrary.

| Grammar | Machine | Memory available |
|---|---|---|
| Regular | Finite automaton | finitely many states, nothing else |
| Context-free | Pushdown automaton | a stack |
| Context-sensitive | Linear bounded automaton | tape proportional to input length |
| Unrestricted | Turing machine | unbounded tape |

Now the practical question. Given a grammar, what can you compute about it?

| Question | Regular | Context-free | Context-sensitive | Unrestricted |
|---|---|---|---|---|
| Is string w in L(G)? | yes, linear time | yes, O(n³) worst case | yes, but exponential | undecidable |
| Is L(G) empty? | yes | yes | undecidable | undecidable |
| Do G₁ and G₂ generate the same language? | yes | undecidable | undecidable | undecidable |
| Is G ambiguous? | not applicable | undecidable | undecidable | undecidable |
| Closed under intersection? | yes | no | yes | yes |
| Closed under complement? | yes | no | yes | no |

Read this as an engineering guide, not trivia. It says: **use the weakest class that expresses what you need.** If your command format is regular, you can test two versions of the spec for equivalence, check whether any string satisfies it at all, and match in linear time. Move up to context-free and you keep efficient parsing but lose equivalence checking. Move up to context-sensitive and membership itself becomes impractical. Expressive power is bought with guarantees, and the guarantees are usually what you actually wanted.

---

## 12. Why This Matters for AIDA3

Four concrete connections.

**Mission and log formats.** Any file format your subteam defines is a formal language. Deciding up front whether it is regular or context-free tells you whether you can validate it with a regular expression or need a real parser, and it tells you what quality of error message you can produce. A format with nesting is context-free, and you should stop trying to regex it.

**Radio phraseology.** Standard ATC phraseology is close to a context-free language with a small vocabulary and rigid structure. A call like "Purdue Tower, Earhart one, ten miles north, inbound landing" fits a template of station, callsign, position, intention. Writing that grammar out explicitly is the first step toward either generating correct calls automatically or checking a transcribed call for completeness, and it tells you exactly which slots a speech recognition system has to fill. It also tells you where the POS tagging and dependency material becomes practical rather than theoretical, since a partially transcribed call has to be mapped onto those slots.

**Grammar-constrained generation.** This is the sharpest connection to the LLMs for Safe UAV Operations work. A language model emits one token at a time from a probability distribution over the whole vocabulary. If you have a context-free grammar for legal commands, you can compute at every step which tokens could possibly continue a legal string, and mask every other token to zero probability before sampling. The model then **cannot** emit a syntactically invalid command, no matter what it wanted to say. This is how JSON mode and structured output work in practice, and the same technique applies to any command language you define yourself.

State the guarantee precisely, because it is easy to oversell. Constrained decoding guarantees syntactic validity. It does not guarantee correctness. The model can still emit a perfectly well-formed command to fly into terrain, exactly the way "the aircraft landed the tower" is perfectly grammatical nonsense. Syntax is the floor, not the ceiling, and semantic checking is a separate job.

**The bridge to temporal logic.** From the logic tutorial you know an LTL formula describes a set of infinite traces. Those sets are exactly the ω-regular languages, the infinite-word analogue of Type 3, and they are recognized by Büchi automata, which are finite automata with an acceptance condition on infinite runs. That is why model checking works the way it does. You build an automaton for the system, build an automaton for the negation of your specification, intersect them, and ask whether the result is empty. If it is not empty, any string in it is a counterexample trace. Every one of those operations is available precisely because we stayed at the regular level, where intersection is closed and emptiness is decidable. Look back at the leftmost column of the table in Section 11 and you will see model checking sitting in it.

---

## 13. Worked Examples

**Example 1. Classify this grammar.**

```
S → a S b  |  ε
```

One non-terminal on each left side, so it is context-free. Is it regular? The rule `S → a S b` puts terminals on both sides of the recursion, which no regular rule allows. The language is `aⁿbⁿ`, which Section 7.1 argued is not regular. So it is Type 2 and not Type 3.

**Example 2. Classify this one.**

```
S → a A
A → b A  |  b
```

Every rule is one terminal followed by at most one non-terminal, always on the right. Right-regular, Type 3. The language is `ab⁺`.

**Example 3. Watch out for this one.**

```
S → A b
A → a A  |  a
```

The first rule is left-regular in shape and the others are right-regular. The grammar mixes sides, so it does not meet the definition of a regular grammar as written. In this case the language `a⁺b` happens to be regular anyway and you could rewrite the grammar consistently. The point stands: check the rule shapes, and if they mix, you have not yet shown the language is regular.

**Example 4. Write a grammar for nested mission blocks.**

Legal strings: a mission is a sequence of zero or more items, and each item is either a single command `c` or a block `b ... e` containing another mission.

```
S    → ITEM S  |  ε
ITEM → c  |  b S e
```

Derive `b c c e c`:

```
S ⇒ ITEM S ⇒ b S e S ⇒ b ITEM S e S ⇒ b c S e S
  ⇒ b c ITEM S e S ⇒ b c c S e S ⇒ b c c e S
  ⇒ b c c e ITEM S ⇒ b c c e c S ⇒ b c c e c
```

Context-free, and necessarily so, because blocks nest.

**Example 5. POS tagging with real ambiguity.**

Tag **"Report the fault."** `Report` could be VERB (imperative) or NOUN. `fault` could be NOUN or VERB. Four combinations are conceivable, but only VERB DET NOUN gives a well-formed English sentence, an imperative with a noun phrase object. A statistical tagger resolves this from transition probabilities, because a sentence-initial VERB followed by DET is a very common imperative pattern while NOUN DET NOUN is not a legal English sequence.

Now tag **"The fault report is clear."**

```
The     fault    report    is      clear
DET     NOUN     NOUN      VERB    ADJ
```

Here `fault` is a noun modifying another noun, a compound. Same word, different tag, decided entirely by context.

**Example 6. Dependency parse.**

**"The autonomous aircraft landed safely on runway one zero."**

```
landed                  root
├── aircraft            nsubj
│   ├── The             det
│   └── autonomous      amod
├── safely              advmod
└── runway              obl
    ├── on              case
    └── one zero        nummod
```

The root is the main verb, and everything else hangs off it directly or indirectly. Note again that "on runway one zero" contributes one arrow, from `landed` to `runway`, with `on` attached below as a case marker. The preposition is not the head.

---

## 14. Terms and Symbols

| Symbol or term | Meaning |
|---|---|
| Σ | alphabet, a finite set of symbols |
| Σ\* | all finite strings over Σ, including ε |
| ε | the empty string, length zero |
| N | set of non-terminals |
| T | set of terminals |
| P | set of production rules |
| S | start symbol |
| α → β | production rule, rewrite α as β |
| ⇒ | one derivation step |
| ⇒\* | zero or more derivation steps |
| L(G) | the language generated by grammar G |
| \|w\| | length of the string w |
| \| | alternation, separates alternative right-hand sides |
| ::= | the arrow, in BNF notation |
| terminal | symbol that appears in finished output |
| non-terminal | symbol that must be rewritten away |
| head | in a dependency, the governing word |
| dependent | in a dependency, the governed word |
| constituent | a group of words functioning as a unit |
| ambiguous | some string has more than one parse tree |
| decidable | some algorithm always halts with the right answer |

---

## 16. Sources and Further Reading

Chomsky, N. (1956). Three models for the description of language. *IRE Transactions on Information Theory*, 2(3), 113 to 124. The paper that introduced the hierarchy.

Chomsky, N. (1957). *Syntactic Structures*. Mouton. Source of "colorless green ideas sleep furiously."

Backus, J. W. (1959). The syntax and semantics of the proposed international algebraic language of the Zurich ACM-GAMM conference. *Proceedings of the International Conference on Information Processing*, UNESCO. Origin of BNF.

Hopcroft, J. E., Motwani, R., and Ullman, J. D. (2006). *Introduction to Automata Theory, Languages, and Computation*, 3rd edition. Pearson. The standard reference for the hierarchy, automata, and the decidability results in Section 11.

Aho, A. V., Lam, M. S., Sethi, R., and Ullman, J. D. (2006). *Compilers: Principles, Techniques, and Tools*, 2nd edition. Pearson. Lexing, parsing, and the split between syntax and semantic analysis.

Jurafsky, D. and Martin, J. H. *Speech and Language Processing*, 3rd edition draft. Chapters on POS tagging, constituency parsing, and dependency parsing. Freely available online.

de Marneffe, M.-C., Manning, C. D., Nivre, J., and Zeman, D. (2021). Universal Dependencies. *Computational Linguistics*, 47(2), 255 to 308. The dependency label set used in Section 3.

Nivre, J. (2003). An efficient algorithm for projective dependency parsing. *Proceedings of the 8th International Workshop on Parsing Technologies*. The transition-based parsing algorithm in Section 3.4.

Shieber, S. M. (1985). Evidence against the context-freeness of natural language. *Linguistics and Philosophy*, 8(3), 333 to 343. The Swiss German cross-serial dependency argument.

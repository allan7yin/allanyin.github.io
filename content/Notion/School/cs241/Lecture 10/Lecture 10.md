## Ambiguous Grammars

**⇒ Formal Definition**

- **a** ==**String**== **_w_** in a grammar is ==**ambiguous**== if there is more than one parse tree for **_w_**
    - e.g. “1 - 10 + 11”, example from last lecture, is ambiguous
- **a** ==**Context-free Grammar**== is ==**ambiguous**== if there exists at least one string **_w_** such that **_w_** **∈** _**L(G)**_ and **_w_** is ==**ambiguous**==
    - e.g the grammar that generated “1 - 10 + 11” is ambiguous

  

An ==**ambiguous grammar**== means that there is no unique derivation and hence no unique meaning (for at least one string). So, when are **CFG’s** ambiguous? There are a couple of common situations:

- ==**Undecidable**== (such as the halting problem)
- Certain ==**ambiguities**== can be spotted, by observation
    - e.g. the same **non-terminal** can be spotted in the RHS of a rule, as seen below
        - **E → E + E**
        - **E → E - E**
- Can occur when there are instances of ‘+’ and ‘-’, as either one can be generated first
- ==**Mixing left-recursion and right-recursion can cause ambiguity**==

  

This can raise the question, how can we make it so that an **ambiguous** grammar is **non-ambiguous**. Consider the following grammar:

![[Screenshot_2023-02-19_at_11.53.45_AM.png]]

- This change makes addition and subtraction **right recursive** operations, and forces the **left-most non-terminal** to derive a binary number rather than another expression
- The grammar generates the same words as the previous grammar, but the parse tree for each derivation is unique

  

By changing the first 2 productions, notice that the expression now only grows by adding more expressions on the right hand side. Since addition and subtraction are right recursive, the right side of the expression will be a child of the root note and evaluated first:

  

![[Screenshot_2023-02-19_at_12.09.38_PM.png]]

You can get a general understanding of how the above altered grammar will not generate any ambiguous strings.

### Associativity and Precedence

CFGs can generate **balanced parentheses and implicit order of evaluating expressions** in the absence of parentheses.

- ==**associativity**==**:** grouping operations with the same precedence
    - 6 - 3 + 4
    - (6 - 3) + 4 or 6 - (3 + 4)?
    - We want left associativity, evaluating from left to right
        - i.e have the left side farther from the root
- ==**precedence**==**:** grouping operations with different precedence
    - 6 + 3 * 4
    - (6 + 3) * 4 or 6 + (3 * 4)?
    - We want multiplication to have precedence over addition
        - i.e. have multiplication happen further away from root than addition, so that it is done first, then addition

  

**⇒ Associativity**

Now, associativity is something that is related to how the **production rules** are structured. Consider the set of rules we have seen before:

![[Screenshot_2023-02-19_at_12.31.01_PM.png]]

![[Screenshot_2023-02-19_at_12.31.10_PM.png]]

Now, the expressions gets longer by adding more operations and digits (expressions) to the **right hand side.** In the tree above, 10 + 11 is evaluated before 1 - (), hence, we get 1 - (10 + 11) as a result. This is a direct result of having **right recursion** in the production rules. What if we swapped those, and used **left recursion** instead?

![[Screenshot_2023-02-19_at_12.35.07_PM.png]]

![[Screenshot_2023-02-19_at_12.35.17_PM.png]]

This expression gets longer by adding expressions to the **left hand side.** By instead using **left recursion** we get **left associativity.** 1 - 10 will be evaluated before ( ) + 11, effectively giving us (1 - 10) + 11.

  

**⇒ Precedence**

Now, consider the above grammar, but we are now including multiplication and division. Consider the following:

![[Screenshot_2023-02-19_at_2.23.06_PM.png]]

This has some issues with precedence. This is because we can get something like: **E ⇒ E * B ⇒ E + B * B.** This will evaluate the expression 1 + 10 * 11 as (1 + 10) * 11, which is not something we want. To address this, one solution is to have **multiplication** occur with children of E, by creating a new **non-terminal T.**

![[Screenshot_2023-02-19_at_2.26.12_PM.png]]

Now, with a derivation such as **E ⇒ E + T ⇒ E + T * B,** the grammar will evaluate the expression 1 + 10 * 11 as 1 + (10 * 11).

  

So far, we’ve derived **specific steps** in a derivation, using one of the **production rules.** Now, we want to be able to refer to a **general step** in an arbitrary derivation, such as **α**==**A**==**β ⇒ α**==**γ**==**β** using the rule ==**A → γ**==. **α** and **β** refer to the symbols before and after the A (and γ) as a way of saying these parts do not change when the A gets re-written by the γ. These greek letters can refer to **non-terminals, terminals, empty string, or some combination of those.**

  

**⇒ Formal Definitions**

We are basically looking for the opportunity to substitute middle terms using production rules, hence introducing these formal definitions.

- **α**==**A**==**β** ==**directly derives**== **α**==**γ**==**β (α**==**A**==**β ⇒ α**==**γ**==**β)** if there is a production rule ==**A → γ**==**,** where:
    - A is a **non-terminal**
    - α,β,γ can be non-terminals, terminals, empty string, or combination
    - **Directly derives** basically means it takes **one** step or rule
- **α**==**A**==**β** ==**derives**== **α**==**γ**==**β (α**==**A**==**β ⇒* α**==**γ**==**β)** if there is a ==**finite sequence of productions**== **α**==**A**==**β ⇒ α**==**B**==**β ⇒ α**==**C**==**β ⇒ … ⇒ α**==**γ**==**β**. This is basically a chain of “directly derives” from beginning to end.
    - Below is a small example. Recall that **⇒*** means all derivations from beginning to the end:
        
        ![[Screenshot_2023-02-19_at_2.55.15_PM.png]]
        
    - Informally, **derives** means it takes 0 or more derivation steps
- The grammar **G** ==**derives the word**== ==_**w**_== ==**∈ T***== ==if== ==**S ⇒***== ==_**w**_==_,_ where w is a concatenation of terminals (no non-terminals), and S is the start symbol
    - ==**Informally**==, the grammar G derives a word _**w**_ if you can derive the word from the start symbol
    - The Language ==**_L(G)_**== _=_ { w ∈ T* if S ⇒* w }
        - ==**Informally**==, the ==**language described by the grammar G**== is the set of concatenations of terminal symbols that can be derived from the start symbol.
- A **language** ==**L**== **is** ==**context-free**== if there exists a context-free grammar, such that ==**_L(G)_**== **_=_** ==**L**==
    
    - **Informally,** a set of strings is context-free if there is some context free grammar that describes the language
    
      
    

### Topic 8: Top-Down Parsing

**⇒ What is parsing?**

- ==**Parsing**==**:** Given a grammar G and a word w, ==**derive**== w using the grammar G.
- This is analogous to regular expressions (which are used to ==**specify**== tokens) and DFAs and Simplified Maximal Munch (which are used to ==**recognize**== tokens)
    - Recall that **simplified maximal much** is a technique used in scanning/tokenization in programming languages.
- Here, we use CFGs to specify a grammar for a programming language and parsing algorithms derive the program

  

**⇒ Parsing Algorithms**

We will look at two **linear-time** approaches:

1. ==**Top-Down**==**:** Find a non-terminal and replace it with the right hand side
    1. e.g. for rule **S → AγB**, replace **S** with **AγB**
2. ==**Bottom-up**==**:** replace a right hand side with a non-terminal. This is quite literally the reverse of top-down.
    1. e.g. for a rule **S → AγB**, replace **AγB** with **S**

These algorithms do not work for all CFGs, so when we create a grammar for a programming language, we must check that it can be parsed by one of these linear-time algorithms. When executing parsing algorithms, we want to be able to remember information about our derivations or processed input. This brings us to **stack-based parsing.**

  

**⇒ Using a Stack**

- For **top-down parsing,** we use a stack to remember information about our derivations or processed input
- **Recall that CFGs are recognized by a DFA with a stack**

![[Screenshot_2023-02-19_at_6.59.18_PM.png]]

![[Screenshot_2023-02-19_at_7.00.13_PM.png]]

We want to be able to detect the end of our input, we, we will need to ==**augment**== our grammars.

  

**⇒ Augmenting Grammars**

We augment our grammars by adding 3 unique characters to the grammar:

1. a ==**new start symbol**== S’ that only appears in one rule
2. ==**the beginning**== of the file (i.e input): ⊢ (also called **BOF**)
3. ==**the end**== of the file (i.e input): **⊣** (also called **EOF**)

Formally, augmenting the grammar ==**(N, T, P, S)**== yields ==**{N U {S’}, T U {⊢,⊣}, P U {S’ → ⊢S⊣}, S’}**====.== One difference you will notice is that the start symbol is S’ instead of S, but we get S from S’ anyways. Again, this augment only aims to introduce ways to represent **BOF** and **EOF**.

![[Screenshot_2023-02-19_at_7.11.22_PM.png]]

  

**⇒ Top-Down Parsing**

There are 2 actions for this ==**parsing algorithm**==**:**

1. Push the start symbol S’, on the stack
2. When a ==**non-terminal**== is at the top of the stack:
    1. ==**expand**== the non-terminal using a production rule where the RHS ofd the rule matches the input (e.g. if the rule is **S’ → ⊢S⊣,** then pop **S’** off the stack and push **⊢S⊣** onto the stack.
3. When it is a ==**terminal**== at the top of the stack, ==**match**== with input
    1. Pop the terminal off of the stack
    2. Read the next character from the input

![[Screenshot_2023-02-19_at_7.51.51_PM.png]]

The above is how we would be able to parse input, given the grammar provided above. The action square indicates what action is needed to progress to the next row. This format can vary:

![[Screenshot_2023-02-19_at_7.53.21_PM.png]]

==**INVARIANT**==**: Derivation = input already read + stack (read top-down).** You can see this hold true in the parsing table above. We know we are done when both the stack and the input contain the EOF symbol: **⊣**. As seen above, there may be times when one non-terminal has multiple rules it can be derived into. We want to know how to correctly predict which one of the rules to apply. This brings us to **LL(1) Parsing.**

  

**⇒ LL(1) Parsing**

- The first **L** means process the input from left to right
- The second **L** means to find a leftmost derivation
- 1 means the algorithm is allowed to look ahead 1 token

**Goal: Unambiguous Prediction**

- ==**Goal**==**:** Find what rule applies if ==**N (a non-terminal)**== is on the stack and ==**c (a terminal)**== is the next symbol in the input to be read
    - We want to be able to use the **production rule** to change the stack ==**non-terminal**== so that it contains a ==**terminal**== that matches the next non-terminal to be read in input, allowing us to ==**match**==
- Implement **Predict(**==**N**==**,** ==**c**==**)** as a table
- For **LL(1)** grammars:
    - for all ==**non-terminals N**== and all terminals c: **| Predict(**==**N**==**,**==**c**==**) | ≤ 1**
    - i.e given an N on the top of the stack and an c as the next input character at most one rule can apply

  

In order to implement **Predict(**==**N**==**,**==**c**==**),** we use three helper functions:

1. ==**First()**==
2. **==Follow()==**
3. ==**Nullable()**== or ==**Empty()**==

  

So, how would we use **First()** to construct the predict table?

- ==**Informally**==**:** For each ==**non-terminal N**==, **First(**==**N**==**)** is the **set of terminals** that can begin a string derived from ==**N**==; that is ==**N**== **⇒*** ==**c**==

Let’s first look at how to use the **predict table:**

![[Screenshot_2023-02-20_at_11.04.50_AM.png]]

So, here is how we would use the table:

- **First(S’)** = {**⊢**}, as that is the only thing in the entire S’ row. The value of (S’,**⊢**) is 1, as we use production rule 1. This basically tells us that if S’ is on the stack and the input is **⊢**, expand using rule 1.
- Empty cells are error states

  

==**Now, let’s look at how to construct a predict table:**==

- To fill a row of a predict table, start with that row’s ==**non-terminal**== and try all applicable rules, tracking which ==**terminal**== symbols ==**eventually**== appear as the first character of a string
- For the above table:

![[Screenshot_2023-02-20_at_11.11.52_AM.png]]

![[Screenshot_2023-02-20_at_11.13.16_AM.png]]
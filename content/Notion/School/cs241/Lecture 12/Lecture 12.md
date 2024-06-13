### Bottom-Up Parsing

Last time, we saw LL(1) Parsing (Top-down parsing), which is an algorithm, we can follow to parse, and determine if a word is part of the language, or not. Let’s do a quick recap of top-down parsing:

**⇒ Algorithm Approach**

First, we needed to construct a predict table for the parsing algorithm.

![[Screenshot_2023-03-11_at_2.48.40_PM.png]]

![[Screenshot_2023-03-11_at_2.48.51_PM.png]]

## Topic 9: Bottom-up Parsing

Now, we will look at another parsing algorithm, known as bottom-up parsing.

**⇒ Non-LL(1) Grammar**

**Consider the following:**

G:

1. S → a b
2. S → a c b

  

![[Screenshot_2023-03-11_at_2.54.53_PM.png]]

![[Screenshot_2023-03-11_at_2.55.02_PM.png]]

The above grammar is not in LL(1), as the predict table is ambiguous. Something like Predict(S,a), is ambiguous as there are 2 rules for it. We need to look ahead, at the next symbol in input, the know which rule was want to use. The predict table must consider pairs of terminals. So, we say that **G is in LL(2).**

  

We can convert LL(2) into LL(1). To do this, we re-write overlapping productions so that:

- One rule contains the common prefix
- A new non-terminal (T), produces the different suffixes (b and cb). So, for the above 2 production rules, we can convert that set of rules into:

G:

1. R → aT
2. T → b
3. T → cb

![[Screenshot_2023-03-11_at_3.03.05_PM.png]]

![[Screenshot_2023-03-11_at_3.04.54_PM.png]]

So, the above slides how we would attempt an LL parse with look-ahead (so, instead of looking at only the first character of the input, we look at the first k, to decide which production rule to use). However, in the right example, we begin to see the limitations of such a process. This brings us to **LR Parsing.**

  

### LR Parsing

Recall that a stack in LL/Top-down parsing is used in the following way:

- the derivation progresses from top of the parse tree (S’), all the way down until the leaves
- current step in derivation = input processed + stack
- The stack is read from top to bottom

Now, LR Parsing is somewhat the opposite of that idea:

- the derivation progresses from the bottom of the parse tree up to the top, hence, we call it bottom-up parsing
- current step in derivation = stack + input to be read
- stack is read from bottom to top

![[Screenshot_2023-03-11_at_4.25.22_PM.png]]

Essentially, one way to think about it is like the following:

- **LL Parsing is intuitive: read from the left, parse from the left ⇒** Left-most derivation
- **LR Parsing is less so: read from the left, parse from the right** ⇒ Right-most derivation

![[Screenshot_2023-03-11_at_3.26.16_PM.png]]

![[Screenshot_2023-03-11_at_3.26.06_PM.png]]

**LR Operations:**

- **Shift:**
    - Move a symbol from the input file to the stack
    - We’ll also include it in the “Read” column to keep track of what has been read so far
- **Reduce:**
    - If there is a production rule of the form **S → AyB** and **AyB** is on the stack, then reduce (i.e replace **AyB** to **S)**
    - This step is the act of applying a production rule to simplify what is on the stack

  

The key question is, **when should we reduce and when should we shift?**

- For LL(1) Parsing, we used a predict table, for LR Parsing, we will use a **transducer**
    - A transducer is a DFA that recognizes strings and may produce output during a transition from one state to another

The idea is to first introduce a symbol: **⋅**, as a placeholder to help keep track of where we are in the RHS of a production rule. For example:

- S’ → ⋅ ⊢ E **⊣**
- S’ → ⊢ ⋅ E **⊣**
- S’ → ⊢ E ⋅ **⊣**
- S’ → ⊢ E **⊣** ⋅

We create a finite automaton to track the progress of the placeholder through the various production rules. How do we build the automaton? There is a **different state each time the place holder moves over one symbol** in the production rule. Below, we’ll take a look at how we can build an LR(0) automaton. The steps to do so are fairly straightforward. Below is what an example LR(0) Automaton looks like:

![[Screenshot_2023-03-11_at_5.33.13_PM.png]]

To construct this, we begin in the start state, evidenced by the ⋅. Then, if we read in a ⊢, we then proceed to the next state, moving the ⋅ right by one read character. Then, we are in the next state, with expression **S’ → ⊢⋅ E ⊣.** If we have the ⋅ preceding a non-terminal, we want to add to the state, all of the production rules where that non-terminal is found in the LHS of the production rule. So, in the second state, we have added the production rules with **E** on the right side, and we place ⋅ in the beginning of the RHS of these production rules. However, notice that one of E’s production rules results in ⋅ preceding T, another non-terminal. In this case, we once again need to add T’s production rules to the state. We keep on doing this, until we have finished constructing the LR(0) Automaton.
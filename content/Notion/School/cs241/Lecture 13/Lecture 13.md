Continuing on from last lecture, we will continue looking at Bottom-Up parsing. Now, we introduce more of how to work with LR parsing, its potential fallbacks and limitations, and how that transitions us into the improved SLR(1) parsing method.

  

**⇒ LR(0) Parsing Algorithm**

Just a short reminder, for all of these parsing algorithms, the number in braces (1), for example, is the lookahead value. The below pseudocode is LR(0), so no lookahead.

![[Screenshot_2023-03-11_at_6.00.56_PM.png]]

**⇒ LR(0) Parsing Algorithm**

- **Lines 1-2:**
    - initialize the automaton by first taking the first transition (So, this is is usually something like S’ → **⊢ E ⊣**
    - push δ(q0, ⊦) onto the state_stack means take the transition for input ⊦ from the start state, q0.
- **Lines 4-9:**
    - Here, we first try to reduce
- **Lines 10-12:**
    - Only try to shift input after any potential reductions have been performed
- **Lines 14-15:**
    - Accept when you have shifted **⊣** because that means the input has been derived from the original start symbol S (or E in the next Example)
    - The algorithm uses 2 stacks that are always kept in synch: State stack, and symbol stack

  

**Stacks**

- (1) a symbol stack to track the symbols from the input stream
- (2) a state stack to track the states that the automation has been in
- Each time a symbol is pushed on or popped off the symbol stack, a state is pushed on or popped off the state stack

  

Now, the above Parsing algorithm can be a bit hard to understand. To help so, consider the example below:

```Markdown
Sample Grammar 
G: 1. S' -> ⊢ E ⊣
   2. E -> E + T
   3. E -> T
   4. T -> id

Task: Use the grammar G and the automaton to parse the input ⊢ id+id ⊣
```

Here is the LR(0) Parsing, using the LR(0) Automaton

![[Screenshot_2023-03-11_at_10.47.39_PM.png]]

So, lets see how we fill in this table:

- First, as per the algorithm, we fill out the table with the explanations below:
- First, shift **⊢** onto the symbol stack, as the first step. **Every time we shift, we are formally reading from input.**
- Then, we transition from state 1 to state 2 (This is all based on the LR(0) DFA constructed the end of lecture 12)
- State 2 has more than one state, so we cannot reduce here. We continue and see the next input is “id”. So, we shift “id” onto the symbol stack, and into input read, and push state 6 onto the stack (this is where “id” transition goes). Now, in state 6, we only have one item inside, so, we can now reduce id. Doing this, we first replace id with the left non-terminal of the production rule T in the symbol stack (not the input read). We then move on to the next input and pop state 6 from the stack.
- Then, we are back in state 2, with the top of the symbol stack being T. Then, with these 2, we know we know we need to go to state 5 (as per the DFA). So, now we push state 5 to the top of the stack. In state 5, there is only one item, so again, we an reduce. We replace T with E, and pop state 5 from the stack and return to state 2.
- With state 2 and the top of the symbol stack being E, we move to state 3. Here, there are more than one item inside, so we do not reduce. Now, we can read in the next item from input, which is +. So, we shit +, pushing it to the symbol stack, and add it to input read. Now, we have state 3 and top of symbol stack being +, so we transition to state 7. Again, this state has more than one item in it, so, we read again. Push id onto top of the symbol stack, and into input read. Now, push state 6 onto the stack.
- Here, we are at state 7, with id at top of symbol stack, so we transition into state 6. There is only one item in here, so, reduce, so reduce id. Reducing id, gives us T, which we update the symbol stack accordingly. Then, we pop state 6 and return to state 7.
- Now, top of symbol stack is T, and we are in state 7, so, we transition to state 8. Here, there is only one item, so, we can reduce T. This changes the T in the symbol stack into E. Now, we currently had **⊢ E + T** on the top of the stack. The production rule in state 8 has **E → E + T.** So, the entire “symbol” on the stack is replaced with just E. Now that we have reduced E + T into E, so the symbol stack now has **⊢ E.**
- As we popped “E + T” off of the stack, that is we push 3 states off of the stack. So, after this, we are now in state 2, with E on the top of the stack. Now, again, state 2 has more than one item in it, so we first see if there is transition from state 2, and top of stack symbol E, which there is. So, we transition into state 3. Now, there are more than one thing in the state, and there is no transition from state 3, with symbol E. So, we read from input. We read in **⊣** into the symbol stack , and input read. Now, as we are in state 3 with symbol **⊣**, we transition to state 4.
- Here, we only **⊢** E **⊣** on the symbol stack, so we accept. In theory, we could have continued until S’, but this is not needed.

  

So, that details how LR(0) Parsing is done. However, there are some limitations to LR parsing. Problem 1:

- What if we had a state that looked liked this:

```Markdown
A -> a⋅cb
B -> y⋅
```

- In a situation like this, do we:
    - Shift the next character c (as suggested by rule 1)
    - reduce y to B (as suggested as rule 2)
- This is known as **shift-reduce conflict (i.e when a state has both shift and a reduction in it).**
- What about a situation like this?

```Markdown
A -> a⋅
B -> b⋅
```

- Here, do we:
    - reduce a to A (as suggested by rule 1)
    - reduce b to B (as suggested by rule 2)
- This is known as a **reduce-reduce conflict.**

  

So, if there is any item A → a**⋅** (i.e the placeholder is at the end) occurs in a state in which it is not alone, then there is a shift-reduce or reduce-reduce conflict, and the grammar is not LR(0). To solve this limitation, we can add a look ahead token to the automaton.

  

For each A → a, attach Follow(A), e.g.

![[Screenshot_2023-03-12_at_12.35.30_PM.png]]

This improved parsed is called the **SLR(1) Parser,** which is a Simple LR with 1 character lookahead. We modify our existing LR(0) Parser as follows:

- When in A → a⋅ {x} (which calls for a reduction), if the next symbol is x, reduce using that rule, otherwise, shift.

The algorithm is the exact same as before, the only difference is, we use the lookahead to decide whether to reduce or not.

![[Screenshot_2023-03-18_at_10.08.22_AM.png]]

![[Screenshot_2023-03-18_at_10.08.38_AM.png]]
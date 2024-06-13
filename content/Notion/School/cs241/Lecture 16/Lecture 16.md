This lecture will detail more about code generation, that specifically pertains to A7.

---

### A7P1

Consider the code:

```C++
int wain(int a, int b) {
	return (a);
}

// the following is the output we get from this 
;; same prolog 
lw $3, 0($29) ; load a from stack (based on the frame pointer)
;; same epilog 
```

**Rationale:** For the rule: factor → LPAREN expr RPAREN

- code(factor) = code(LPAREN) + code(expr) + code(RPAREN) = code(expr)
- this was mentioned in the last lecture

  

For the first question, create a `codeGenerator()` function that generates code for each production rule. This is then used when travsersing the parse tree to determine what code to generate.

  

---

### A7P2

⇒ **dcls → dcls dcl BECOMES NUM SEMI**

- **dcl → type ID**
- e.g int total = 0;
- The above production rules are those used for declaring variables

  

Suggestions for working with this question:

- **code(NUM)**
    - Put the number, **NUM**, into register $3, i.e **$3 ← NUM**
- **code(dcl BECOMES NUM SEMI)**
    - Load NUM into $3 (use `lis $3` and `.word`)
    - Look up the offset of ID in the symbol table (i.e the offset relative to the frame pointers $29).
    - Generate the code: `sw $3, ID_offset($29)`.

  

**⇒ statement → lvalue BECOMES expr SEMI**

- lvalue → ID
- e.g. total = 0;
- For A7, lvalue is an ID (not a pointer). This will change for A8
- **Notes:**
    
    - code(statement)
    - Evaluate the expression **expr** by calling code(expr)
    - The result should be stored in register $3
    - Look up the offset of the ID in the synbol table (i.e. the offset relative to the frame pointer $29).
    
    ```C++
    code(statement) = code(expr) = sw $3, ID_offset($29)
    ```
    
      
    

---

### A7P3

This question will need to work with binary expressions. Consider the following:

```C++
int wain(int a, int b) {
	return a+b;
}
// we can see this is easily add $3, $1, $2. But, what about other arbitray operations? 
// we must be able to handle an arbitray number of operations 
```

We cannot assign $3 to a subtree of the expression, as we won’t have anything to use for the other child. We also can’t just use another register as this plan may result in all registers being used up.

- Solution: push value in $3 onto system stack, and pop it off into $5
    
    ![[Screenshot_2023-03-28_at_2.10.17_PM.png]]
    

This is how we can successfully perform any arbitrary length operation: For every small opration, we store it into $3, which is then pushed to stack, then poped off when needed again into $5. Which we can then use $5 however we need (maybe add it $3, to obtain new result, etc.)

![[Screenshot_2023-03-28_at_2.25.02_PM.png]]

What about instructions that require 2 source registers? (e.g. add, sub, muol, dev, beq, slt, etc.) The source registers will always be $3 and $5

![[Screenshot_2023-03-28_at_2.27.55_PM.png]]

---

### A7P4

Now, we look at tests. There are 2 control structures in WLP4:

1. While loops
2. If-then-else statements

Both of these rely on comparison tests.

**Conventions:**

- $0 ← 0, no choice here, it’s hardwired into MIPS
- $11 ← 1, we must add this to the prolog (basically initialize it to this)
- recall: when evaluating multiple expressions, in a recursively friendly way
    - results are returned in $3
    - use stack(to store) and $5 (to retrieve) intermediate results

  

**Consider:**

test → expr1 LT expr2 (where LT means less than)

- evaluate the 1st expression, expr1 (the result stored in $3) and then push $3 on the stack
    
    ![[Screenshot_2023-03-28_at_2.55.35_PM.png]]
    

What about GT? We can use the LT code to represent GT:

![[Screenshot_2023-03-28_at_3.17.28_PM.png]]

![[Screenshot_2023-03-28_at_3.18.19_PM.png]]

  

  

![[Screenshot_2023-03-29_at_1.15.40_PM.png]]

![[Screenshot_2023-03-29_at_1.15.30_PM.png]]

![[Screenshot_2023-03-29_at_1.16.12_PM.png]]

  

![[Screenshot_2023-03-29_at_1.16.23_PM.png]]

---

### A7P5

For this question, we are working with the production rule: `statements -> PRINTLN LPAREN expr RPAREN`

- `print` prints the contents of $1 followed by a newline
- It over-writes the contents of $1 and $31
- It is a library routine
- `print.merl` has to be linked, the directive `.import print` must be added to the prolog

Here is the approach:

![[Screenshot_2023-03-29_at_1.21.10_PM.png]]

**⇒ The print sub-routine**

- The print sub-routine overwrites the contents of $1
- There are 2 ways of dealing with this situation:
    - Try a different calling convention (say, read from $3) but the older code needs to be changed
    - Let the value $1 be lost, but it may be important
    - Save and restore $1 on the system stack before calling print
- We generally store $1 and $2 on the stack.
- Later on, when we take calling procedures, we’ll set it up so that procedures save and restore any registers whose values they overwrite
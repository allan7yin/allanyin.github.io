## Topic 11: Code Generation

The previous lecture detailed the process of augmenting the parse tree, and creating symbol maps, in order to conduct context-sensitive analysis. We saw how we could then identify various potential issues such as out of scope declarations, undeclared variable usage, etc. With that now out of the way, we proceed to begin looking at the final stages of the compiler. Again, like before, the input to the code generation section is the output from the context-sensitive analysis, which is the augmented parse tree, and hierarchal symbol table.

  

Of course, there are technically an infinite number of MIPS programs that could be translated from one higher level program, but for this course, we’ll look at one specific way of doing so.

  

So, this focuses on the last stage of the compiler, generating the assembly code that correlates with the high-level language. There are some key issues in this part of the compiler. Namely:

- **Correctness**
    - compiler must create an equivalent program
    - compiler must be correct for all valid inputs
- **Ease of (or simplicity of) writing the compiler**
    - especially for this course
- **Efficiency of the compiler**
    - time to compile the program, O(n) for n lines of code
- **Efficiency of the compiled code**
    - minimum resources (time and space) required
    - This is called code optimization (e.g. deals with how to assign registers effectively). We touch on this concept briefly in this course.

  

**So, what approach do we take in order to do this? ⇒ Syntax-directed Translation**

So, what is syntax-direction translation? Syntax-directed translation is a technique used in compilers to generate code from a source language to a target language. It involves the use of context-free grammars to specify the the structure of a programming language and then using that structure to generate code.

Syntax-directed translation works by associating code generation actions with the productions of the grammar that describe the syntax of the programming language. Consider the following production rules from WLP4:

- ==**expr**== **→** ==**expr1 - term2**==
- ==**expri**== **→** ==**termi**==
- ==**termi**== **→** ==**factori**==
- ==**factori**== **→** ==**NUMi**==

So, we would generate this code recursively using the following rules:

- **code(**==**expr**==**) = code(**==**expr1**==**) + code(**==**term2**==**) + code(**==**expr1 - term2**==**)**
- **code(**==**expri**==**) = code(**==**termi**==**)**
- **code(**==**termi**==**) = code(**==**factori**==**)**
- **code(**==**factori**==**) = code(**==**NUMi**==**)**

  

But, how should we even interpret what this means? Think of it like this. Consider doing the calculation `1-2` in WLP4. So compile this into MIPS, we would eventually get (greatly simplified), something like:

```Assembly
;; code(expr) = 
lis $3 ; code(expr1) 
.word 1
lis $5 ; code(term2)
.word 2
sub $3, $3, $5  ; code(expr1 - term2 )
```

which we would express as **code(**==**expr**==**) = code(**==**expr1**==**) + code(**==**term2**==**) + code(**==**expr1 - term2**==**)**

- i.e. the code for ==**expr**== (the entire code snippet above) equals the code for ==**expr1**== (lis $3 .word 1) concatenated with the code for ==**term2**==(lis $5 .word 2), concatenated with the code for ==**expr1 - term2**== (sub $3, $3, $5).

  

What about a rule like: **main → INT WAIN LPAREN** ==**dcl1**== **COMMA** ==**dcl2**== **RPAREN LBRACE** ==**dcls**== ==**statements**== **RETURN** ==**expr**== **SEMI RBRACE.** For something like this, we would have **code(**==**main**==**) = code(**==**dcl1**==**) + code(**==**dcl2**==**) + code(**==**dcls**==**) + code(**==**statements**==**) + code(**==**expr**==**)**.

- We do not generate code for the delimiters like commas, semicolons, left and right parentheses, etc.

  

Consider the following example:

**Example a)** `**int wain(int a, int b) {return a;}**`

Output:

```Assembly
add $3, $1, $0 ; move 1st parameter to register that holds the return value 
jr $31 ; return control to the OS 
```

_**Conventions:**_

- $1 and $2 are designated to hold the parameters for the wain function
    - think of the loaders `mips.twoints` from the student server
- $3 holds the return value
- $31 holds the address (of the operating system / simulator) that we return to when our program exits

  

**Example b)** `**int wain(int a, int b) {return b;}**`

Output

```Assembly
add $3, $2, $0
jr $31
```

Example 1 and example 2 both have the same parse tree. This raises the question, how can we differentiate programs? To do so, we add a location table to the symbol table, such as:

![[Screenshot_2023-03-27_at_4.05.22_PM.png]]

So, where do we store variable values?

1. **Attempt 1: Stores Variables in Registers**
    1. Idea: each variable gets its own register
    2. problem: what is there are more then 32 variables? We only have 32 registers to use
2. **Attempt 2: Stores Variables in RAM**
    1. Idea: Stores variables in RAM using the .word directive
        1. each variable gets its own label “x” in MIPS
    2. Problems:
        1. more costly to access variables
        2. must be able to differentiate local variables that both share the same ID and label
        3. this approach will not work for recursive functions
3. **Attempt 3: Store Variables in Stack**
    
    1. Store local variables and parameters on the stack with locations relative to $30
    
      
    

![[Screenshot_2023-03-27_at_5.29.54_PM.png]]

![[Screenshot_2023-03-27_at_5.30.11_PM.png]]

We now have:

1. A **stack frame** to store parameters and local variables
2. a **prolong** (to set up the stack frame, any constants needed, etc.)
3. a **body** to do the task required
4. an **epilog** (to top off the stack frame)

Yet, there are still some problems with this design:

- The stack pointer may change value in the body of the code. This occurs if the body has some complicated expressions like **(a+b) - (4*a*c) / (2*a)**. This is because this large expression’s intermediate results (such as a+b) are stored on the stack.

The solution to this is something called the frame pointer. Currently, we cannot use the stack for temporary storage after pusing the stack frame because if we change the value of the stack pointer, then the offsets in the symbol table will all need to be updated (recall, they are all relative to the position of the stack pointer $30).

  

**⇒ Frame Pointer (fp)**

- Reserve $29 to **point to the first elemt of the stack frame** (for this procedure)
- Then, offsets in symbol table will be relative to the frame pointer
- The frame pointer does not change value as the stack is used for temporary values in the body of the function  
      
    

Here is how we would use this: Consider the following code:

```Assembly
int wain(int a, int b) {
	int c = 0;
	return a;
}
```

First, we store **a, b,** and **c** on the stack with location offsets **relative to the frame pointer, $29**

![[Screenshot_2023-03-27_at_6.10.49_PM.png]]

![[Screenshot_2023-03-27_at_6.10.58_PM.png]]

- Now all references to arguments and local variables are based on frame pointer.
- The stack can now be used in the body of the function to store intermediate values.
- **_This approach is recommended_**

  

Below are some code generation conventions to follow for CS241”

![[Screenshot_2023-03-27_at_6.12.23_PM.png]]

![[Screenshot_2023-03-27_at_6.12.55_PM.png]]
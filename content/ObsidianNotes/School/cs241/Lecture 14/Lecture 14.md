### Context-Sensitive Analysis

Recall that the steps to translate a high-level program to an assembly language follows a transition of:

1. WLP4 Program
2. Tokens (A4: WLP4 scan: lexical analysis (regular languages))
3. Parse tree (A5: WLP4 Parse tree: Syntactic Analysis (context free grammars))
4. Augmented parse tree + symbol table (A6: WLP4Gen: Semantic analysis)
5. MIPS Assembly Language (A7-A8)

  

**⇒ Syntax vs Semantics**

- **Context-free**
    - _cannot_ detect if a variable is used before it is declared
- **Context-sensititive**
    - _can_ detect if a variable is used before it is declared
- **Input: Parse Tree**
- **Pre-condition:** The program is syntactically valid (which is checked prior to this step by the parsing)
- **Output:**
    - **if** input is semantically valid
    - **then** output ab augmented parse-tree + symbol table
    - **else** output ERROR

  

This then introduces the question: **If a program is syntactically valid, what else can go wrong?**

- **Variables:**
    - can be undeclared, used before they were declared, or have multiple declarations
- **Procedures:**
    - can be undeclared, used before they were declared, or have multiple declarations
- **Type** errors can occur with:
    - the return value of procedures
    - paramater lists
    - operators
- **Scope** errors can happen with variables in (and out of) procedures

  

**⇒ Variable Declaration Issues**

To address this issue, we implement a symbol table. This is similar to what we did for the MIPS assembler and labels. To do so, we keep track of of **Name** and **Location.** Other than this, we also keep track of type (**int, int*, string, etc)**. Consider the following:

```C++
int wain(int a, int b) {
	return c;
}
```

When using a variable, make sure it exists in the symbol table(AKA, it has been declared). In the above, `return c;` is lexically valid (the words or “components” of that expression are valid tokens of this language), and is certainly syntactically valid (it is a valid WLP4 expression). However, it is semantically invalid, if c has not been declared somewhere else.

  

When declaring a variable, first check that it is not already in the symbol table. If it is not, then add it to the symbol table. Otherwise, if its already in the symbol table, throw an error.

**⇒ Steps in Checking Variable Declarations**

1. **First check for multiple declarations**
    1. Recursively traverse the parse tree and track any declarations
    2. Search for nodes with the rule **dcl → TYPE ID**
        1. Extract the name (e.g. **a**) and the type (**e.g. int**)
        2. Check if the name is already in the symbol table. If it is, report error, otherwise, add name and type to the symbol table.
2. **Next, check for undeclared variables**
    1. Recursively traverse the parse tree and track the use of variables
    2. Search for nodes with the rules
        1. **factor → ID**
        2. **lvalue → ID**
        3. Recall that lvalue is just a “variable”
    3. Check if ID’s name is not in the symbol table, is so, throw error (means has not been declared before).

  

In addition to the above checks we can perform, something we should also consider is scope. Consider something like:

```C++
int f() {
	int a=0;
	return a;
}

int wain(int x, int y) {
	int a=1;
	return x+a;
}
```

This presents the reason why we instead want to implement a **hierarchal symbol table** rather than just an ordinary symbol table. How do we accomplish this, by implementing local and global symbol tables. **WLP4 does not have global variables**. So, we need to have:

- Global symbol table that stores procedure names and types
- For each procedure, to have a local symbol table, to track its parameters and local variables
- **NOTE: all return types in WLP4 are of int**

  

We say that procedures have signatures. What does this mean? It means that:

- Names (called IDs)n which must be extracted
- return types which are always int in WLP4
- parameters lists with possibly a mixture of int and int * types

  

**In order to find procedures in the parse tree, we will need to:**

- traverse the parse tree and search for procedures declarations, so one of the following 2 production rules:
    - **procedure → INT ID LPAREN params RPAREN LBRACE …**
    - **main → INT WAIN LPAREN dcl COMMA dcl RPAREN LBRACE …**
- Once one of the above rules have been found, check:
    - if the procedure name is already in the global symbol table, error. Otherwise, add it to the global symbol table, and create a new symbol table for that procedure
    - We the store the procedure’s signature in the global symbol table. This action is captured by the following production rules:
        - **paramlist → dcl**
        - **paramlist → dcl COMMA paramlist**
        - **dcl → TYPE ID**

  

There are only 2 types in WLP4: **int and int***. Going forward, we will make use of this “wlp4 semantics handout”:

[https://student.cs.uwaterloo.ca/~cs241/wlp4/semanticrules.pdf](https://student.cs.uwaterloo.ca/~cs241/wlp4/semanticrules.pdf)

  

⇒ **Working with Type Rules**

In order to type-check:

- Reference the rules in the pdf above
- Notation/Rules:
    
    ![[Screenshot_2023-03-15_at_6.28.50_PM.png]]
    
    - The above is the notation we will use. For example consider some type derivation rules:
    - This is simply saying, for example, if E1 is an integer, and E2 is also an integer, than addition, subtraction, division, and multiplication between E1 and E2 will also be an integer.
        
        ![[Screenshot_2023-03-15_at_6.30.09_PM.png]]
        
- ensure that WLP4 rules are followed when computing the type of an expression.
- Set the left-hand side’s type to the right-hand side’s type for rules such as:
    
    - **expr → term**
    - **term → factor**
    - **factor → ID**
    - **factor → NUM**
    - **factor → NULL**
    - **==for the above, type(LHS) = type(RHS)==**
    
      
    

In order to type check, we decorate the parse tree of an expression, called an augmented parse tree. For this, we propagate from leaves up (i.e. postorder depth-first traversal).

![[Screenshot_2023-03-15_at_6.44.51_PM.png]]

We must be able to check if types are being used properly. To do so, we will introduce some new notation: **we’ll let t represent a type**. We use this to reference type, wihtout explicitly mentioning the type.

- “E1 : t and E2 : t” means E1 and E2 have the same type, i.e. they  
    are either both  
    int or both int*.
- “E1 : t1 and E2 : t2” means E1 and E2 may or may not have the  
    same type.  
    

As we can have either **int** or **int *** in WLPA4, we need to keep track if the type is an **int** or **int ***. We will work with type rules:

![[Screenshot_2023-03-15_at_8.06.07_PM.png]]

![[Screenshot_2023-03-15_at_8.08.31_PM.png]]

![[Screenshot_2023-03-15_at_8.10.01_PM.png]]

![[Screenshot_2023-03-15_at_8.10.19_PM.png]]

  

**⇒ Type Checking: Well-Typed Expressions**

Some structures (such as loops or statements) don’t have types, so we check that the structure is **well-typed** e.g the components have the right types. What does well-typed mean? In order for an expression to be well-typed, **the types expected by functions and operators must coincide with the types of their arguments**  
. For instance, the following expression 2 + 3 is well-typed because the + operator expects a pair of integer arguments and both 2 and 3 have type integer.  

![[Screenshot_2023-03-15_at_8.16.23_PM.png]]

![[Screenshot_2023-03-15_at_8.16.32_PM.png]]

![[Screenshot_2023-03-15_at_8.16.43_PM.png]]

![[Screenshot_2023-03-15_at_8.16.54_PM.png]]

![[Screenshot_2023-03-15_at_8.17.01_PM.png]]
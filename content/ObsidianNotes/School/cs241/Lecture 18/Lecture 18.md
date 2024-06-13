### Topic 14: Code Generation - Pointers

Now, in the previous lectures, we looked at working with procedures. Now, moving on from that, we expand the allowed types, and now consider how to work with pointers. Working with pointers will span questions 2 to 5 for A8.

  

Want our compiler to be able to handle something like:

![[Screenshot_2023-04-02_at_4.25.23_PM.png]]

![[Screenshot_2023-04-02_at_4.26.58_PM.png]]

### A8P2

For something like **factor1 → STAR factor2 (this is something like** `***p**`**)**, here is how we work with this:

- Generate the code for **factor2**, the interpret the results (which is $3) as an address and load the contents of that address into $3
- so **code(factor1) = code(factor2)** `**lw $3, 0($3)**`

  

**⇒ NULL**

What about `factor -> NULL`? When dereferencing a null pointer, it should crash the MIPS machine. How do we ensure this?

- Make NULL = 0x1
- The value 1 is not word aligned, meaning that is not a multiple of 4
- Any attempt to use this address (lw or sw MIPS instructions) will crash the machine
- all we need to do is: **code(NULL) =** `**add $3, $0, $11**`
- In most other languages, NULL is 0x0 and it is the OS that prevents using 0x0 as a address.

  

**⇒ Lvalues**

In WLP4, there are 5 production rules in which lvalues appear:

![[Screenshot_2023-04-02_at_7.36.33_PM.png]]

There are three sub-cases for the first use of lvalues: **factor → AMP lvalue**

1. **lvalue → ID**
    1. This is asking for something like `&y`, so just dereferencing a variable. So, we look up y in the symbol table
    2. The address is stored as an offset from the frame pointer ($29) so get the actual address by adding the variable’s offset to $29
    3. Implementation:
        
        1. **code(factor):**
        
        ```C++
        lis $3
        .word ID_OFFSET
        add $3, $3, $29
        ```
        
2. **lvalue → STAR factor2**
    1. This is the same as saying `&(*y)` which is the same as `y`. So, simply do: **code(factor1) = code(factor2)**
3. **lvalue1 → LPAREN lvalue2 RPAREN**
    1. Again simple: **code(lvalue1) = code(lvalue2)**

  

**⇒ Case 2b**

Consider `statement -> lvalue BECOMES expr SEMI`. But, this can be used for `int` and `int*` so we need to know the lvalue type in order to generate code. Now, for A7, we were assigning ints to ints, which is very easy. However, now that there can be pointers, need to make some adjustments. We know that for this, the next rule could be `lvalue -> STAR factor`. The solution to this is as follows:

![[Screenshot_2023-04-02_at_8.11.01_PM.png]]

where something like code(lvalue) will be like:

```C++
code(lvalue) = {
lis $3
.word x     // where x is the offset for this ID
add $3, $29, $3
}


for non-pointers before, we did this:
code(statement) = code(expr) = sw $3, ID_offset($29)

With the new appraoch, we are looking to generalize a method for both pointers and non-pointers. 
```

### A8P3

Now consider binary operations. When we were working with binary operations in A7, we simply pushed and computed the code for the individual expressions separately, and then added, subtracted, or whatever operation needed was done. Now, however, we have introduced pointers, which can be summed and subtracted from integers. So, we will need to introduce some additional case work.

![[Screenshot_2023-04-02_at_8.31.59_PM.png]]

![[Screenshot_2023-04-02_at_8.32.08_PM.png]]

Now, obviously, in order to these, we will need to know the types of these. This is stored in the augmented parse tree, which can be referenced to obtain this information. We multiply the int above by 4, to treat it as an address (as addresses are multiples of 4). Note that we cannot add two pointers. However, it is possible to subtract 2 pointers.

  

Now, what about subtraction? This will follow the same principles as addition. In fact, its almost identical with the exception of `int* - int*`

![[Screenshot_2023-04-02_at_8.42.06_PM.png]]

![[Screenshot_2023-04-02_at_8.43.15_PM.png]]

Recall that the result of subtracting an integer pointer from another integer pointer is a normal integer, as this represents the distance between the 2 addresses. Divide by 4.

  

### A8P4

Now lets look at comparisons. In particular, something of the form `test -> expr1 LT expr2`. After the context-sensitive analysis has been done, we know that both sides of the LT are of the same type. We’ve already dealt with the integer versions in A7. In A8, we can look at if they are of `int*` type. If this is the case, we use the `sltu` instead of normal `slt` as addresses are always positive, and as such, can have a much larger range:

![[Screenshot_2023-04-02_at_8.48.09_PM.png]]

### A8P5

A simple array. Consider the following code:

```C++
int wain(int *a, int n) {
	return *a;
}
```

So, what does the above program do? It returns the first element of the array. **How do we do this in MIPS?**

- We first find the base address for the array (which is saved in $1) and copy it over to $3 `lw $3, 0($1)`

Recall one of the student server tools `mips.array`, in which we enter something along the lines of:

![[Screenshot_2023-04-02_at_10.13.51_PM.png]]

As seen above, we input things like array length, and each item of the array. But how is the array actually being constructed? `**mips.array**` **is actually allocating memory on the heap and then calling wain with the location of the array in $1 and its size in $2 as parameters.** Following this logic the below code:

```C++
int wain(int *a, int n) { 
	return *(a+1);
}
```

returns the second element of the array, in other words, a[1].

  

**⇒ Dynamic Memory Allocation**

There are 2 production rules we look at:

- **factor → NEW INT LBRACK expr RBRACK**
- **statement → DELETE LBRACK RBRACK expr SEMI**

  

CS241 will provide the library routines that handle memory management.

![[Screenshot_2023-04-02_at_10.27.44_PM.png]]

![[Screenshot_2023-04-02_at_10.27.53_PM.png]]
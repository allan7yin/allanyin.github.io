### Question 1: Binary Basics

1. What does 10010010 represent
    1. 146, assuming this is an unsigned integer. But without any context, there is no meaning.
2. Convert the following decimal values to binary using 8-bits two’s complement representation.
    1. -62: 11000010
    2. 13: 00001101
    3. 128: overflow.
3. Convert the following binary values in 8-bits two’s complement representation to decimal values.
    1. 00001000: 8
    2. 11000001: -63
    3. 10000101: -123
    4. 11111110: -2
4. What is the largest and the smallest value in 16-bits two’s complement representation.
    1. largest value is (2^15) - 1

  

### Question 2: MIPS Assembley Programming

1. Name one potential challenge of having a MIPS program with more than 32768 instructions. How  
    would you overcome this challenge?  
    1. Recall that for branching instructions we allocate 16 bits for the offset to branch by. However, with 32768 instructions, there is a risk of branching by a value greater than 2^15, which is a problem. To avoid this problem, we can use jump to the address stored in a register using `jr $s`.
2. 1. What is the value stored in $3 after executing the following instructions? Write your answer in  
    hexadecimal notation.  
    

```Assembly
lis $3
add $3, $0, $0
jr $31

; here, lis $3 takes the next line as data, so, convert this into binary, then into hexadecimal 
; so, $3 stores the value 0x00001820
```

1. Assume that $1 stores the address of an array of instructions, where the only instruction that is jr $31 is the last instruction. Write a MIPS program that executes the program (list of instructions) stored in this array.

```Assembly
jr $1
```

1. No point
2. Write a MIPS function _**reduce**_ that takes two parameters: $1 holds the address of an array of integers that is size of 10, $2 holds the address of a function that takes two integers as its parameters (in $1 and $2 respectively) and returns an integer in $3 **(we will call this function f)**. The function _**reduce**_ will first apply _**f**_ on the first and the second element in the array, then apply the result of previous call on f to the next element repeatedly. The final result will be placed in $3.

```Assembly
; implement function called "reduce": takes 2 parameters, $1 holds the address of an array of integers of size 10
; $2 is second parameter, holds address of a function that takes 2 integers as its parameters ($1 and $2) and 
; returns an integer in $3 (we'll call this function f). Reduce first applies f on first and second elements of the
; array (getting the result in $3), then apply the result of the previous call on f ($3) to the next element 
; repeatedly 

reduce: 
sw $1, -4($30)
sw $2, -8($30)
sw $4, -12($30)
sw $5, -16($30)
sw $6, -20($30)
sw $7, -24($30)
sw $8, -28($30)
sw $9, -32($30)
sw $10, -36($30)

lis $7
.word 36
sub $30, $30, $7

add $5, $0, $1 ; save the address of the array 
add $9, $0, $2 ; save the address of f
lis $8
.word 4
lis $10
.word 1

lis $4
.word 8 ; help track the number of elements remaining

lw $1, 0($5) ; load first element of the array 
add $5, $5, $8 ; increment the loop counter 
lw $2, 0($5) ; load the second element of the array 

loop:
beq $0, $4, break ; no more elements left in the array 
jalr $9 ; calls foo, with parameters $1 and $2
sub $4, $4, $10 ; decrement the number of remaining items we need to iterate through 
add $5, $5, $8 ; increment the loop counter 
lw $1, 0($5) ; load the second element of the array 
add $2, $0, $3 ; place result of previous call to f, into the second 
beq $0, $0, loop

break: ; reaching here means all elements of the array have been traversed 
lis $7
.word 36
add $30, $30, $7

lw $1, -4($30)
lw $2, -8($30)
lw $4, -12($30)
lw $5, -16($30)
lw $6, -20($30)
lw $7, -24($30)
lw $8, -28($30)
lw $9, -32($30)
lw $10, -36($30)
jr $31
```

1. Question below:
    
    ![[Screenshot_2023-03-07_at_1.44.35_PM.png]]
    

(a) Recognize that the left hand are 8 bits, each digit is hexadecimal. Can convert this into 32 bit binary instructions, do then recover the asm program

### 3 Formal Languages

1. State whether or not the following statements are true or false
    1. Every regular language is a context-free language. T
    2. Every context-free language is a regular language. F
    3. The concatenation of two regular languages is a regular language. T
    4. The union of two regular languages is a context-free language. T
    5. If a language is not regular than it must be context-free. F
    6. The intersection of two regular languages is a regular language. T
2. Prove that the following language is regular, with Σ = {0, 1}, L = ⟨0000, 1111, 0011, 1100}
    1. Every word in this language is off the expression 0000|1111|0011|1100
3. Given a NFA M = (Σ, Q, q0, δ, A). Provide a program in pseudocode that takes an input string w, and output ”true” if s ∈ L (M ), ”false” otherwise.
    
    ```Assembly
    q = q0
    for each a in w:
    	qnew = {}
    	for each qi in q:
    		qnew = qnew U (δ(a, qi))
    	q = qnew
    return q n A == {}
    ```
    
4. Give a regular expression for the following languages over Σ = {0, 1}.
    1. Language L that begins with 001 and ends with either 000 or 111.
        1. 001(0)*(1)*111 | 001(0)*(1)*000 | 00111
    2. Language L that contains exactly three 0s
        1. (1*)0(1*)0(1*)0(1*)
    3. Language L that does not contain 101.
        1. (1*)(00(0*)1*)*(0*)(1*)
        2. (0*)(100(0*))*(1*)(0*)
        3. The above are 2 ways, first one is mine, second is solution
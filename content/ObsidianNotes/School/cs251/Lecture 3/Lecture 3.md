**Part1: ARM**

- New figures: Hardware fr LDUR/STUR/ADDI
- Questions from A0, Piazza, etc.

  

**Part2: Digital Logic**

![[Screenshot_2023-01-19_at_8.53.05_AM.png]]

  

- **Register transfer language:** Something like `R[3] = Mem[ R[1] + 0x40 ]`, which is what `LDUR X3, [X1, \#64]`, does.
- Each **register** is 5 bits
- Watch the flow of the chart, and how memory is accessed to compute the operation.

![[Screenshot_2023-01-19_at_8.53.27_AM.png]]

![[Screenshot_2023-01-19_at_8.53.48_AM.png]]

LDUR/STUR are designed to work on eight byte double words, multiples of eight.  
For example we usually see something like  

```Assembly
LDUR x1, [x2, \#40] // x1 ← Mem[x2 + 40]
```

This takes register 2, adds 40, and reads a “double word”, 64 bits, or eight bytes from memory.

Suppose x2=0, then we have,

```Assembly
x1 ← Mem[40]
```

Note that this is shorthand for **“read the eight bytes starting at location 40”**. This is memory at locations `Mem[40]`, `Mem[41]`, ... `Mem[47]`. These are read and “packed” into a 64 bit word in register `x1`.

If you read an address that is not a multiple of eight, the machine will work fine , it will  
just read the eight bytes starting at the memory location given. That is, the command  

```Assembly
LDUR x1, [xzr, \#4]
```

gives, `x1 ← Mem[4]`, and this is the memory locations `Mem[4]`, `Mem[5]`, ... `Mem[11]`.

**⇒ Accessing Arrays**

Unless otherwise stated, we will take “arrays” to be ordered objects of 8 bytes(64 bits) double words. Suppose `X0` is the start of the array and `X1` is the index `k=0.. N-1`. To load `A[k]` we would normally do the following:

```Assembly
LSL x3, x1, \#3 // convert array index to byte offset index (multiple of eight)
ADD x4, x0, x3 // x4 ← x0 + x3
LDUR x5,[x4] // read doubleword at Mem[x4]
```

```Assembly
ADD X1, XZR, X0 // X1 <- X0, Copy register X0 to X1
ADDI X2, XZR, \#10 // X2 loop counter
loop:
STUR XZR, [X1] // Set A[] = 0

ADDI X1, X1, \#8 // Increment memory pointer in steps of eight
SUBI X2, X2, \#1 // decrement loop counter
CBNZ X2, loop
```

Note, that when iterating through an array must update both the array index, as well as the array “counter”.

## Introduction to Digital Logic

For reference, here are the slides being used:

![[lec3-1-zhkupdated.pdf]]

They will be updated with my markups when the slides are finished.
This is the beginning of this Java course. Will focus on reviewing some object-oriented programming principles as well as introduce new content. While doing that, will learn Java syntax, and become more comfortable working with it. I will be quickly going through this course, however, as I’m already fairly familiar with a lot of the concepts.

### Jshell

This is sort of like vim, where is a terminal tool that allows us to run Java code.

![[Screenshot_2022-12-02_at_12.29.00_PM.png]]

Here, we can simply run some Java code, like such:

```Java
System.out.print("Hello Allan");
```

**⇒ Data Types**

Has the usual “number” data types:

- byte
- short
- int
- long

```Java
byte // -128 to 127
short // -32,768 to 32,767
int // -2147483648 to 2147483647
long // -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807
```

Of any of these data types, we can view the maximum, minimum, or size through some methods:

```Java
Integer.MAX_VALUE
Integer.MIN_VALUE
// these same methods are avilable for the other data types 
```

**⇒ Casting in Java**

We can declare multiple variables of the dame data type in one line:

```Java
int Allan = 7, brian = 23;
```

Consider the following code:

```Java
byte newValue = (Byte.MIN_VALUE / 2);

// this results in 
jshell> byte newValue = (newMin / 2);
|  Error:
|  incompatible types: possible lossy conversion from int to byte
|  byte newValue = (newMin / 2);
|                   ^--------^

jshell>
```

This will create an error, even though we know the value will fit in a byte. This is because at the time of compilation, the compiler does not know what the value of newMin is. So, since it does not know, it throws an error. On the contrary, if we did `byte newValue = (Byte.MIN_VALUE / 2)`, we would no receive an error. If the calculation uses literal values, the compiler will be able to evaluate the result at run time, and so, no error is thrown. But, we know for sure that there value will fit into a Byte, so to override this error, we can use casting. Here is how we cast:

```Java
byte newValue = (byte) (newMin / 2);
```

The byte in front of the division casts the result into a byte. This prevents Java from treating it as a default, which is an integer.

**⇒ Float & Double Primitives**

- float
- double

These are the same as you’ve used in C++. Java’s default decimal data type is double. So, doing something like:

```Java
float number = 5.25; 
```

Will throw an error, with the compiler complaining that you cannot assign a double to a float. To make this a float, remember we need to add the suffix:

```Java
float number = 5.25f; // f makes this float 
double number 5.25d; // d does same thing, but is default 
```

**⇒ Char and Bool**

These two are the exact same as C++. Single quotes to define a character, and bool is just true or false.

**⇒ String**

Unlike the things before which are primitive types, strings are Java defined classes.

```Java
String name = "allan";
name += 5; // this makes allan5
```

The `+` operator after a string always concatenates that following value onto the string.

**⇒ Operators, Operands, and Expressions**

These are again, all the same as C++. Just note that something like:

```Java
char char1 = 'a';
char char2 = 'b';
System.out.print(char1 + char2); 
// this will print 195. The char are integers then added, if you want to get ab:
System.out.print("" + char1 + char2);
```

**⇒ IntelliJ**

Now, we are going to use this IDE moving forward.
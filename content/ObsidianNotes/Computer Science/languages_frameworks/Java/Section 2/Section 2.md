### Conditionals + Operators

- The if else statements are the same as C++.

```Java
if (some conditional expression) {
	System.out.println("Something");
}
```

The things such as AND and OR are the same, here is a quick review of the ternary operator:

```Java
operand1 ? operand2 : operand3
```

### Java Methods

These are the same as C++. Functions that are a part of a class:

```Java
public class Student {
	public static void method() { // this is a method 
		// do something
	}
}
```

**⇒ Method Overloading**

To remind, to overload an method, they must differ by the parameters passed to the methods. Simply having different return types is not enough.

### Control Flow

**⇒ Switch**

```Java
int name = "Allan";

switch(name) {
	case "Brian": 
		// do something
		break;
	case "Allan":
		// do something here as well
		break;
	default:
		// sort of like the else of an if conditional block 
		break;
}
```

We can also group things together, if multiple conditions will result in the same code being run:

```Java
int number = 1;

switch(number) {
	case 1: 
		// some code 
		break;
	case 2: case 4: case 7:
		// some code here 
		break;
	default:
		// do something here 
		break;
}
```

Java introduces **enhanced switch statements**. They do the same thing as the conventional switch statements, just modifies the syntax:

```Java
switch(switchValue) {
	case 1 -> System.out.println("Value was 1");
	case 2 -> System.out.println("Value was 2");
	case 3,4,5 -> {
		System.out.println("Value was 3 or 4 or 5");
	}
	default -> System.out.println("None");
}
```

If we want to return things, instead of having a `return`, keyword in each case, we can just:

```Java
// old 
switch(switchValue) {
	case 1 -> 1; // effectively the same thing as returning 1
	case 2 -> 2; // the same as returning a 2
	case 3,4,5 -> {
		yield 99; // cannot use return word, must use yield
	}
	default -> System.out.println("None");
}
```

### Loops

**⇒ For Loop**

The exact same syntax as C++:

```Java
for (int i = 0; i < 10; i++) {
	// do something each iteration 
}
```

**⇒ While**

```Java
while (num > 0) {
	// do something, update the condition
}
```

**⇒ doWhile**

This is fairly different

```Java
do {
	// loop contents go here, will need to break 
} while (condition holds);
```

The above guarantees the loop will at least run once.

### Static & Instance fields and methods

Something important to recognize is the use cases of the static keyword. The purpose of it is similar for fields and methods. Recall that in C++, static defined things for an entire class.

![[Screenshot_2022-12-02_at_8.37.27_PM.png]]

  

![[Screenshot_2022-12-02_at_8.38.34_PM.png]]

The main differences are that static fields and methods are based upon the class itself, whereas instances are based upon the object, rather than the entire class. This type of thing will become more comfortable to work with more experience.

**⇒ Parsing values & System.console()**

What if we have a string like: “123”, and we really needed the integer value within it (perhaps useful when breaking apart json data)? Since this is such a common issue in Java, there are built in methods was can use to parse:

- Integer: parseInt(String)
- Double: parseDouble(String)

These would be useful in situations like this:

```Java
String birthYear = "2003";
int currentYear = 2022;
int dateOfBirth = Integer.parseInt(birthYear);
// dateOfBirth = 2003, the integer 
```

Static members of the String class.

**⇒ Reading data from the console**

![[Screenshot_2022-12-02_at_10.45.47_PM.png]]

The way we’ll use for now is System.console, which allows to read in input from the terminal.

```Java
String name = System.console().readLine("Hi, what's your name: ");
System.out.println("Hi there " + name + ", what would you like to do?");
```

**⇒ Exception Handling and Scanner**

For exception handling, we use try and catch blocks.

```Java
try {
	// ststements that might yield some errors 
} catch (Exception e) {
	// code to handle the exception
}
```

Now, let’s take a look at the scanner class:

```Java
// recall we create classes with new 
Scanner sc = new Scanner(System.in); 
// to read from a file 
Scanner sc = new Scanner(new File("name of the file"));
```

To use the scanner, we’ll ned to import it. The same thing as the `include` statements in C++. When we try reading something that is not a number into one, Java will throw a `NumberFormatException`. Here’s a bit of code that practices this. Say we want to read in 5 numbers, and find the sum. If something not an integer is entered, keep looking for an integer.

```Java
public static void main(String[] Args) {
        Scanner scanner = new Scanner(System.in);
        int count = 1;
        int number = 0;
        int sum = 0;

        while (count < 6) {
            System.out.print("Enter nunber #" + count + ": ");
            try {
                number = Integer.parseInt(scanner.nextLine());
                sum += number;
                count++;

            } catch (NumberFormatException e) {
                System.out.println("Please enteger an integer");
            }

        }

        System.out.println("The sum of the numbers you entered is " + sum);
    }
```
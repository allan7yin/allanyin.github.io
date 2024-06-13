### OOP 1 - Inheritance

**⇒ Classes and Objects**

The same thing as what is used in C++. Some minor changes however.

![[Screenshot_2022-12-03_at_4.14.40_PM.png]]

(Package refers to file).

**⇒ Getters and Setters**

Remember that when needed, make data private, so that only accessible within the class. To be able to retrieve this information, create public getter and setter functions.

**⇒ Constructors**

The default constructor is the same thing.

```Java
// this is constructor in Java
public class Allan {
	private int age;
	private String name;

	public Allan (int age, String name) { 
		this.age = age;
		this.name = name;
	}

	public Allan () {
		this(19, "Allan"); // calls on the other construcor, this is constructor chaining
	}
}
```

**⇒ Reference vs Objects vs Instance vs Classes**

![[Screenshot_2022-12-03_at_4.44.46_PM.png]]

This is a reference, a bit different in terms of syntax compared to C++. See the above right side. Same concept as in C++, a reference is an alias, and points to the same space in memory.

**⇒ Static vs Instance Variables**

Recall that static variables are shared across all objects. So, when it is changed on one object, it is changed for all instances.

![[Screenshot_2022-12-03_at_4.53.45_PM.png]]

On the other hand, instance variables are specific to an instance of the class. Now, something important to look at are the differences between static and instance methods. So, for example, why is it always `public static main`. Here are some properties of static methods:

- Static methods are declared with static modifier
- Static methods can’t access instance methods and instant variables directly
- They’re usually for operations that don’t require any data from an instance of the class (from **this**)
- Inside static method, cannot use **this**
- They’re called off of the class. Here is example of code:

![[Screenshot_2022-12-03_at_5.00.55_PM.png]]

Now, here are instance methods:

- Belong to the object, an instance of the class
- Can access instance methods and variables directly.
- Can access static methods and variables directly.

![[Screenshot_2022-12-03_at_5.03.00_PM.png]]

As seen, just leave out static.

**⇒ POJO - Plain Old Java Object**

Now this is something relatively new. Pojo’s generally only have instance fields, so mimics the format of a JSON, just a generic structure with data in it. Many frameworks use POJO’s to read data from, or to write data to, databases, files, or streams. Another name for POJO’s are Java Beans, or **Data Transfer Objects** (similar to what was seen in Nest.js). Now, perhaps we want to be able to obtain a string with that is the DTO of our class. We can do this by overriding the `toString()` Java function.

Now, it’s important to note that overriding and overloading are not the same thing. An overriden method is a special method in Java, that other classes can implement, if they use the specified method signature. This now allows the following:

![[Screenshot_2022-12-03_at_5.38.19_PM.png]]

Where Java implicitly calls the `toString()` method when we output an object. One issue with the POJO is the fact that it can become fairly repetitive, with lots of boiler plate code being written. To address this issue, Java introduced the Record type.

- The record is a special class that contains data, that’s not meant to be altered
- Seeks to achieve immutability, for the data in its members
- Contains only the most fundamental methods, such as constructors and accessors

So, how do we this? Consider the following:

```Java
// Main.java 
public class Main {

    public static void main(String[] args) {

        for (int i = 1; i <= 5; i++) {
            LPAStudent s = new LPAStudent("S92300" + i,
                    switch (i) {
                    case 1 -> "Mary";
                    case 2 -> "Carol";
                    case 3 -> "Tim";
                    case 4 -> "Harry";
                    case 5 -> "Lisa";
                    default -> "Anonymous";
                    },
                    "05/11/1985",
                    "Java Masterclass");
            System.out.println(s);
        }

        Student pojoStudent = new Student("S923006", "Ann",
                "05/11/1985", "Java Masterclass");
        LPAStudent recordStudent = new LPAStudent("S923007", "Bill",
                "05/11/1985", "Java Masterclass");

        // System.out.println(pojoStudent);
        // System.out.println(recordStudent);

        pojoStudent.setClassList(pojoStudent.getClassList() + ", Java OCP Exam 829");
        // recordStudent.setClassList(recordStudent.classList() + ", Java OCP Exam
        // 829");

        // System.out.println(pojoStudent.getName() + " is taking " +
        // pojoStudent.getClassList());
        // System.out.println(recordStudent.name() + " is taking " +
        // recordStudent.classList());
    }
}
```

```Java
// Student.java
public class Student {

    private String id;
    private String name;
    private String dateOfBirth;
    private String classList;

    public Student(String id, String name, String dateOfBirth, String classList) {
        this.id = id;
        this.name = name;
        this.dateOfBirth = dateOfBirth;
        this.classList = classList;
    }

    // @Override
    // public String toString() {
    // return "Student{" +
    // "id='" + id + '\'' +
    // ", name='" + name + '\'' +
    // ", dateOfBirth='" + dateOfBirth + '\'' +
    // ", classList='" + classList + '\'' +
    // '}';
    // }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDateOfBirth() {
        return dateOfBirth;
    }

    public void setDateOfBirth(String dateOfBirth) {
        this.dateOfBirth = dateOfBirth;
    }

    public String getClassList() {
        return classList;
    }

    public void setClassList(String classList) {
        this.classList = classList;
    }
}
```

```Java
// LPAStudent.java
public record LPAStudent(String id, String name, String dateOfBirth, String classList) {

// things in header are called record header 
}
```

The LPAStudent is our record, as seen, we pass to it the parameters we would normally pass to a constructor. Essentially, it takes care of the code that creates a string from the data from the object.

**⇒ Record vs POJO**

So, when would we want to use one over the other? Since the record is immutable, we would only really want to use this if we wanted a class to not have its data modified. However, if we will be reading a lot of records, from databases or from files, and simply using the data or passing it around, not making any changes, the record is big improvement in making sure no unintended changes are done.

**⇒ Inheritance**

![[Screenshot_2022-12-03_at_6.24.19_PM.png]]

So, the idea of inheritance should be very clear for you. Consider the following code:

```Java
// Animal.java 
public class Animal {

    private String type;
    private String size;
    private double weight;

    public Animal() {

    }

    public Animal(String type, String size, double weight) {
        this.type = type;
        this.size = size;
        this.weight = weight;
    }

    @Override
    public String toString() {
        return "Animal{" +
                "type='" + type + '\'' +
                ", size='" + size + '\'' +
                ", weight=" + weight +
                '}';
    }

    public void move(String speed) {
        System.out.println(type + " moves " + speed);
    }

    public void makeNoise() {
        System.out.println(type + " makes some kind of noise");
    }
}
```

```Java
// Dog.java
public class Dog extends Animal {

    private String earShape;
    private String tailShape;

    public Dog() {
        super("Mutt", "Big", 50); // super constructor must be the first statement in the child class constructor
    }

    public Dog(String type, double weight) {
        this(type, weight, "Perky", "curled"); // this calls the other defined constructor in this class, creates
                                               // default parameters as well
    }

    public Dog(String type, double weight, String earShape, String tailShape) {
        // super(type, size, weight); we can acctually create a size here
        super(type, weight < 15 ? "small" : (weight < 35 ? "medium" : "large"), weight);
        this.earShape = earShape;
        this.tailShape = tailShape;
    }

    // to make the soString method, we will need to call the super classes toString
    @Override
    public String toString() {
        return "Dog{" +
                "earShape='" + earShape + '\'' + ", tailShape='" + tailShape + '\'' + "} " + super.toString(); // need
                                                                                                               // super.toString()
    }

    // this method is also in
    public void makeNoise() {

    }

    @Override // this is a signature, not needed, but is much clearer code.
    public void move(String speed) {
        super.move(speed); // calling super class method
        // do other stuff here, since you're overriding this method
    }
}
```

To create a child class, we use the keyword, `extends`. There are no virtual methods in Java, so theres no need to explicitly tell which functions are able to be overridden. Instead, as long as we re-define that method in the child class, that method has now been overridden. Now, the `@Override` tag is not needed, but in my opinion, makes the code much more clear on inheritance hierarchy. For constructors, we call the super class constructor, passing the needed arguments, and then initializing the other data points that our child class contains that the parent does not.

Again, private members cannot be accessed by child classes. In order for those fields to be accessible, will need to make them protected (or public, but that type of data should never really be public).

**⇒ java.lang.Object - What is this?**

Essentially, this is like the parent of all classes. All classes in Java inherit from this object, which contains many methods, such as toString(). This is why we usually place the tag **@Override** before it.

**⇒ String methods**

Here are some of the most useful/used string methods:

- **Length:** Length of string
- **charAt:** returns character at the index that’s passed
- **indexOf/lastIndexOf:** Returns an integer, representing the index in the sequence where String or character passed, can be located in the string
- **isEmpty** returns true if length is 0
- **isBlank:** Returns true if length is 0 OR of the string only contains white space character
- **contentEquals:** Returns a boolean if the string’s value is equal to the value of the argument passed.
- **equals:** Return a boolean if the string’s value is equal to the value of the argument passed
- **equalsIgnoreCase:** Same as above, but now, case is ignored
- **concat:** Similar to the plus operator for strings, concatenates text to the string, and returns a new one
- **join:** Allows multiple strings to be concatenated together in a single method, specifying a delimiter
- **repeat:** Returns the string repeated by the number of times specified in the argument
- **replace/replaceAll/replaceFirst:** These methods replace characters or strings in the string, returns a new string with the replacements made
- **substring/subSequence:** These return a part of the string, its range defined by the start and end index specified
- **Str1.compareTo(Str2):** If str 1 comes before str2 lexicographically, negative value is returned, if after, greater than 0, and if equals, 0

**⇒ StringBuilder class**

What is string builder? It provides a mutable string. Consider the following code:

```Java
StringBuilder helloWorldBuilder = new stringBuilder("Hello" + "World");
// this is how we initilaize them, creating new objects, need to new 
String helloWorldString = "Hello" + "World";
```

The helloWorldString is immutable, it cant change. We can make the variable reference another string, but thats not the same. The specific string it currently references, cannot be altered. For example:

```Java
helloWorldString.concat(" and Goodbye");
helloWorldBuilder.append(" and Goodbye");
// now print 
System.out.println(helloWorldString);
System.out.println(helloWorldBuilder);

// this outputs 
// Hello World
// Hello World and Goodbye
```

The `concat()` string method does not change the string it is being called on, rather, it creates a new string, and binds it back to that variable. On the other hand, the `append()` method of the stringBuilder directly changes the string, hence why it prints out after calling it.

**⇒ Composition**

Reviewed inheritance, now look at Composition. Composition is just another type of relationship between classes. Where for inheritance, we had a child class **is-a** type of the parent class, here, say we have the following:

```Java
// Product.java
public class Product {

    private String model;
    private String manufacturer;
    private int width;
    private int height;
    private int depth;

    public Product(String model, String manufacturer) {
        this.model = model;
        this.manufacturer = manufacturer;
    }
}

class Monitor extends Product {

    private int size;
    private String resolution;

    public Monitor(String model, String manufacturer) {
        super(model, manufacturer);
    }

    public Monitor(String model, String manufacturer, int size, String resolution) {
        super(model, manufacturer);
        this.size = size;
        this.resolution = resolution;
    }

    public void drawPixelAt(int x, int y, String color) {
        System.out.println(String.format(
                "Drawing pixel at %d,%d in color %s ", x, y, color));
    }
}

class Motherboard extends Product {

    private int ramSlots;
    private int cardSlots;
    private String bios;

    public Motherboard(String model, String manufacturer) {
        super(model, manufacturer);
    }

    public Motherboard(String model, String manufacturer, int ramSlots, int cardSlots,
            String bios) {
        super(model, manufacturer);
        this.ramSlots = ramSlots;
        this.cardSlots = cardSlots;
        this.bios = bios;
    }

    public void loadProgram(String programName) {
        System.out.println("Program " + programName + " is now loading...");
    }
}

class ComputerCase extends Product {

    private String powerSupply;

    public ComputerCase(String model, String manufacturer) {
        super(model, manufacturer);
    }

    public ComputerCase(String model, String manufacturer, String powerSupply) {
        super(model, manufacturer);
        this.powerSupply = powerSupply;
    }

    public void pressPowerButton() {
        System.out.println("Power button pressed");
    }
}
```

```Java
//PersonalComputer.java
public class PersonalComputer extends Product {

    private ComputerCase computerCase;
    private Monitor monitor;
    private Motherboard motherboard;

    public PersonalComputer(String model, String manufacturer,
            ComputerCase computerCase, Monitor monitor,
            Motherboard motherboard) {
        super(model, manufacturer);
        this.computerCase = computerCase;
        this.monitor = monitor;
        this.motherboard = motherboard;
    }

    public ComputerCase getComputerCase() {
        return computerCase;
    }

    public Monitor getMonitor() {
        return monitor;
    }

    public Motherboard getMotherboard() {
        return motherboard;
    }
}
```

As seen, inside of the [Product.java](http://Product.java) folder, you can see many class inheriting from the Product class, which demonstrate the **is-a** relation. Now, inside of the PersonalComputer class, it contains a ComputerCase, Monitor, and Motherboard object as fields. This is composition, and is a **has-a** relationship.

**⇒ Encapsulation**

Encapsulation means to hide certain data away. So, why would we want to hide things in Java? To make the interface simpler, we may want to hide unnecessary details. To protect the integrity of the data on an object, we may hide, or restrict access to some of the data and operations.

To decouple the published interface from the internal details of the class, we may hide actual names and types of class members. What do we mean by interface here? Not the struct like data type. When we talk about a class’s public or published interface, we’re talking about the members that are exposed to, or can be accessed by, the calling code. Everything else in the class is private, or internal to it.

An Application Programming Interface, API, is the public contract, that tells others how to use the class.

**⇒ Polymorphism**

So, this is just the idea that we can assign child objects to a parent class type, and call methods, and still receive the functionality of the child function. Consider the following code:

```Java
// Movie.java 
public class Movie {

    private String title;

    public Movie(String title) {
        this.title = title;
    }

    public void watchMovie() {

        String instanceType = this.getClass().getSimpleName();
        System.out.println(title + " is a " + instanceType + " film");
    }
}

class Adventure extends Movie {

    public Adventure(String title) {
        super(title);
    }

    @Override
    public void watchMovie() {
        super.watchMovie();
        System.out.printf(".. %s%n".repeat(3),
                "Pleasant Scene",
                "Scary Music",
                "Something Bad Happens");
    }
}

class Comedy extends Movie {

    public Comedy(String title) {
        super(title);
    }

    @Override
    public void watchMovie() {
        super.watchMovie();
        System.out.printf(".. %s%n".repeat(3),
                "Something funny happens",
                "Something even funnier happens",
                "Happy Ending");
    }
}

class ScienceFiction extends Movie {

    public ScienceFiction(String title) {
        super(title);
    }

    @Override
    public void watchMovie() {
        super.watchMovie();
        System.out.printf(".. %s%n".repeat(3),
                "Bad Aliens do Bad Stuff",
                "Space Guys Chase Aliens",
                "Planet Blows Up");
    }
}
```

```Java
// in Main.java 
public class Main {

    public static void main(String[] args) {

        Movie theMovie = new Adventure("Star Wars");
        theMovie.watchMovie();
    }
}
```

So, from above, the traits/advantage of polymorphism is clear. We can initialize a `Movie` object, but provide it a child of the `Movie` class, as an `Adventure` is still a `Movie`, technically. So, even though `theMovie` has the `Movie` type in front of it, when calling the `watchMovie()` method, the Java knows that `theMovie` is actually an `Adventure`, more specific than `Movie` and then looks for and uses the more specific method on the `Adventure` class. This is the mai benefit of polymorphism. It is the ability to execute different behaviour, for different types, which are determined at runtime. Polymorphism enables us to write generic code, based on the base class, or a parent class.

**⇒ Casting with classes, and using Object and var references**

Consider the following code:

```Java
Adventure jaws = Movie.getMovie("A", "Jaws");
jams.watchMovie();
```

The above code is wrong, and will not compile. While we can assign a child object to its parent, the other way is not valid. Every child class “is a” type of parent class, but the reverse is certainly not true. However, in the above code, we know for sure, that Movie.getMovie() will return an Adevnture object, which is assignable to an Adventure type variable. To get around this, we could cast:

```Java
Adventure jaws = (Adventure) Movie.getMovie("A", "Jaws");
jaws.watchMovie();
Adventure jaws2 = (Adventure) Movie.getMovie("C", "Jaws");
jaws2.watchMovie();
```

![[Screenshot_2022-12-03_at_5.35.53_PM.png]]

The first line of this code will run fine. However, if we cast, and the object returned is not an Adventure object, the run time will throw an error:

![[Screenshot_2022-12-11_at_12.00.07_PM.png]]

**⇒ Var in Java**

Consider the following code:

```Java
var airplane = Movie.getMovie("C", "Airplane");
airplane.watchMovie();
// aiplane is a movie 

var plane = new Comedy("Airplane");
// plane is a Comedy 
plane.watchComedy();
```

`var` is sort of like the `auto` keyword in C++. Var uses type inference and based on the return type of the thing being assigned to the variable, Java auto assigns this type. This is part of something called **local variable type inference (LVTI).** This was introduced in Java 10, one of the benefits is that it helps the readability of the code, and to reduce boilerplate code. It’s called local variable type inference for a reason, because:

- Cannot be used in field declarations on a class
- Cannot be used in method signatures, either as a parameter, or a return type
- Cannot be used without an assignment

So, literally the `auto` keyword in C++.

**⇒ instanceof operator**

This operator lets us test the type of an object or instance. The reference variable you are testing is the left operand and the type you are testing for is. the right operand. Will look something like:

```Java
if (unknownobject instanceof Adventure) {
	// do something
} 
```

Now, something new introduced to Java is pattern matching for the instanceof operator. If the JVM can identify that the object matches the type, it can extract data from the object, without casting. For this operator, the object cam be assigned to a binding variable, which here is called syfy:

```Java
unknownObject instanceof ScienceFiction syfy
```

The variable `syfy` (if the `instanceof` method returns true), is already types as a ScienceFiction variable. With this, we won’t need to make a local variable, and no need to cast anything:

```Java
if (unkownObject instanceof ScienceFiction syfy) {
	syfy.watchscienceFiction();
}
```

**⇒ Organizing Java Classes**

So, now we look at Java packages. As per the Oracle Java Documentation, a package is a namespace that organizes a set of related types. In general, a package corresponds to a folder or directory, on the operating system, but is not a requirement. Two of the packages we’ve used recently two packages, that is, java.lang and java.util. For `java.lang`, we used things like `String builder`, `String`, and all of the basic data types. For java.util, we used things like `Random` and `Scanner`. In a sense, this is fairly similar to including things in C++, such as vector, the `io` libraries, etc.

So, what would a package name look like? We’ve seen that Java starts their package names with java, as seen above. However, it is common practice to use reverse domain name to start your own package conventions.

**⇒ Packages**

So, packages really provide this tree organization structure to the packages. Inside these packages is where can place our class files, which we can then import in other files. For instance consider the following code:

```Java
package dev.lpa.package_one;

import java.util.* // * means we're importing everything 

public class Main {
	public static void main(Stringp[] args) {
		Scanner scanner = new Scanner(System.in);
		Random random = new Random();
```

It’s also important that package names consist of only lower case characters. Uppercase ones are reserved for class names. This is just coding conventions. With packages, we can classes in another package, and import them in, to be able to used.
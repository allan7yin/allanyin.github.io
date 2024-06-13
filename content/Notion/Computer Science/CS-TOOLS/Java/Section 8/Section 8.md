## Basic Input & Output including Java.util

**⇒ Exceptions**

So, we now introduce the catch and try blocks in Java. Again, it follows the same syntax as m,ost languages out there:

```Java
public int findQuitient(int x, int y) {
	try {
		return x/y;
	} catch(ArithematicException e) {
		return 0;
	}
}
```

So, we are catching the specific type of error.

**⇒ Multi Catch exceptions**

We can have multiple catch blocks for a single try block:

```Java
try {
	x = getInt();
	y = getInt();	
	System.out.println("x is " + x + ", y is " + y);
	return x/y;
} catch(NoSuchElementException e) {
		throw new NoSuchElementException("no suitable input");
} catch(ArithematicException e) {
		throw new ArithematicException("attempt to divide by 0");
}
```

We can also implement error catching inside of main. Consider:

```Java
public static void main(String[] args) {
	try {
		int result = divide(); // gets user input to divide the 2 numbers 
	} catch(ArtithematicException | NoSuchElementException e) {
		System.out.println(e.toString());
		System.out.println("Unable to perform division, autopilot shutting down");
	}
}		 	
```

**⇒ Introduction to Input/Output**

Before moving forward, let’s first introduce the idea of a static block of code in a Class. Inside of a static block of code, it is only run once. The non-static block:

```Java
{
		// DO something 
}
```

gets called every time an instance of the class is constructed. The static block is only called once, when the class itself is initialized.

```Java
// example 
public class Test {

    static{
        System.out.println("Static");
    }

    {
        System.out.println("Non-static block");
    }

    public static void main(String[] args) {
        Test t = new Test();
        Test t2 = new Test();
    }
}

/* this outputs 
Static
Non-static block
Non-static block
```

**⇒ Writing to files: FileWriter class and Finally block**

Now, we’ll take a look at how we can access files in Java, whether that be writing to or from files. To begin, we’ll take a look at the FileWriter class. This is a class of `java.io.package` and is used to write to files:

```JavaScript
FileWriter locFile = null; // instantiate this object, could directly new 
locFile = new FileWriter("locations.txt"); // creates file if this dne 
locFile.write(things to write to this file); 
locFile.close(); // need to close this file 
```

So, the above shows how we can use the FileWriter class to be able to write to a test file. To make using this class more safe, we can encapsulate it with error handlers to be sure exceptions will be caught:

```JavaScript
public static void main(String[] args) {
    FileWriter locFile = null;
    try {
        locFile = new FileWriter("locations.txt");
        for(Location location : locations.values()) {
            locFile.write(location.getLocationID() + "," + location.getDescription() + "\n");
        }
    } catch(IOException e) {
        System.out.println("In catch block");
        e.printStackTrace();
    } finally {
        System.out.println("in finally block");
        try {
            if(locFile != null) {
                System.out.println("Attempting to close locfile");
                locFile.close();
            }
        } catch(IOException e) {
            e.printStackTrace();
        }
    }
}
```

**⇒ Try with resources**

Now before going into this more in detail, let’s look at the code:

```Java
FileWriter locFile = null;
  try {
      locFile = new FileWriter("locations.txt");
      for(Location location : locations.values()) {
          locFile.write(location.getLocationID() + "," + location.getDescription() + "\n");
      }
  } catch(IOException e) {
		e.printStackTrace();
}finally {
      System.out.println("in finally block");
      if (locFile != null) {
          System.out.println("Attempting to close locfile");
          locFile.close();
      }
  }
```

As seen above, we know that in the method that will contain that code snippet, an IOException is going to be caught or thrown. So, we can add something to the method naming, to indicate this we will be dealing with checked exceptions. This means the method must catch the exception, or specify that is will throw it or then throw it.

```Java
public static void main(String[] args) throws IOException {} 
```

So, what does this do? This means that if in the method, an IOException error is thrown, we are no longer writing what we want to do with it. Instead, by indicating `throws IOException`, the error will now be propagated up the call stack when it is thrown. Now, we look at try with resources. This is utility removes the need for the finally block of code, as it automatically closes the resource for us:

```Java
public static void main(String[] args) throws IOException {
    try(FileWriter locFile = new FileWriter("locations.txt")) {
        for(Location location : locations.values()) {
            locFile.write(location.getLocationID() + "," + location.getDescription() + "\n");
        }
    }
}
```

So, above is the solution. We now have something inside of the try statement, which is the new resource we are allocating. Then, inside of the block, we can use it. Notice that we now no longer need to check for IOExceptions being thrown as we’ve included that inside of the method declaration, and there is no longer any need for the finally block. Compare with the code above to see the differences in using `try with resources`, and without it. It is very effective and good to use.

**⇒ FileReader and Closeable**

Now, we can actually have more than one resource when using the `try with resource.` To do so, we simply add it to the area where we have included the resource with the try, above:

---

Before moving on, I want to take a quick moment and compile a list of some of the attributes of the scanner class. In case it becomes unclear later on. The Scanner class is used for obtaining the input of the primitive types. It is the easiest way to read input in a Java program, though not very efficient.

- `nextLine()` is a method of scanner that gets the next entire line, and returns it. Something like:

```Java
import java.util.*;

public class GFG1 {
  public static void main(String[] argv) throws Exception {

    String s = "Gfg \\n Geeks \\n GeeksForGeeks";
    // create a new scanner
    // with the specified String Object
    Scanner scanner = new Scanner(s);

    // print the next line
    System.out.println(scanner.nextLine());

    // print the next line again
    System.out.println(scanner.nextLine());

    // print the next line again
    System.out.println(scanner.nextLine());

    scanner.close();
  }
}
```

To initialize the scanner, we always do:

```Java
Scanner nameOfScanner = new Scanner(something);
// where something is where the input stream we are choosing, this can be either a something like
// System.in, a string, or a FileReader object:

Scanner scanner = new Scanner(System.in);

String word = "allan";
Scanner scanner = new Scanner(word);

Scanner scanner = new Scanner(new FileReader("file.txt"));
```

In addition, there are some other things we can add to Scanner:

```Java
scanner = new Scanner(new FileReader("locations.txt"));
scanner.useDelimiter(","); // tells us stop reading each time we hti a ","
scanner.skip(scanner.delimiter()); // tells us to skip the next delimiter we see
```

This is the best way that we can effectively read in input.

---

Now, back to the main content. To use the `FileReader` object and read from a file, it is very similar to how we used `FileWriter`:

```Java
Scanner scanner = null;
try {
    scanner = new Scanner(new FileReader("locations.txt"));
    scanner.useDelimiter(",");
    while(scanner.hasNextLine()) {
        int loc = scanner.nextInt();
        scanner.skip(scanner.delimiter());
        String description = scanner.nextLine();
        System.out.println("Imported loc: " + loc + ": " + description);
        Map<String, Integer> tempExit = new HashMap<>();
        locations.put(loc, new Location(loc, description, tempExit));
    }

} catch(IOException e) {
    e.printStackTrace();
} finally {
    if(scanner != null) {
        scanner.close();
    }
}
```

Now, one thing peculiar thing to note. We needed to close `FileWriter` after we were done using it, doing something like: `locFile.close();`. However, this does not appear when we are reading from the file. This is because we have already closed the scanner, with `scanner.close()`. The scanner object then takes care of the `FileReader` passed to it, and closes it.

Now, as was the case with `FileWriter`, we can do the above using `try with resources` to make things clearer. Here is a quick demonstration on how we might use it:

```Java
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) throws IOException {
        // this will be testing some of the io methods

        try(FileWriter locFile = new FileWriter("file.txt")) {
            locFile.write("line 1 \\nline 2 \\nline 3");
        }

        // now that we have written to the file, lets read from it, also using try with resources

        try(Scanner scanner = new Scanner(new FileReader("file.txt"))) {
            while(scanner.hasNextLine()) {
                String temp = scanner.nextLine();
                System.out.println(temp);
            }
        }
    }
}
```

**=> BufferedReader**  
The  
`BufferedReader` class reads text from a character-input stream, buffering characters so as to provide for the efficient reading of characters, arrays, and lines. We can specify the buffer size, or use the default one. In general, each read request made by a `Reader` causes a corresponding read request to be made of the underlying character or byte stream. It is therefore advisable to wrap a `BufferedReader` around any `Reader` whose `read()` operations may be costly, such as `FileReader` and `InputStreamReader`. Here are some of the important methods on the `BufferedReader`:

- `close()`: closes the stream and releases resources
- `read()`: this reads a single character
- `readLine()`: reads a line of text
- `skip(long)`: skips characters

So, we can use this to adjust the `FileReader` code we had above:

```Java
Scanner scanner = null;
try {
    scanner = new Scanner(new BufferedReader(new FileReader("locations.txt")));
    scanner.useDelimiter(",");
    while(scanner.hasNextLine()) {
        int loc = scanner.nextInt();
        scanner.skip(scanner.delimiter());
        String direction = scanner.next();
        scanner.skip(scanner.delimiter());
        String dest = scanner.nextLine();
        int destination = Integer.parseInt(dest);
        System.out.println(loc + ": " + description + ": " + destination);
        Location location = locations.get(loc);
        location.addExit(direction, destination);
    }

} catch(IOException e) {
    e.printStackTrace();
} finally {
    if(scanner != null) {
        scanner.close();
    }
}
```

As seen, we wrap the `FileReader` with a `BufferedReader`.

### Byte Streams

So far, we've looked at how to read or write text files or text data, as well as things like `FileReader` and `FileWriter`, and at buffering the data to make the program more efficient. Now, let's take a look at something called the Byte Stream.

Java Byte streams are used to perform input and output of 8-bit bytes. The most common classes related to byte streams are `FileInputStream` and `FileOutputStream`. Let's take a look at a quick example of using these 2 classes to copy an input file into an output file:

```Java
public static void main(String[] args) throws IOException {
    FileInputStream in = null;
    FileOutputStream out = null;
    try {
        in = new FileInputStream("directions.txt");
        out = new FileOutputStream("output.txt");
        int c;
        while ((c = in.read()) != -1) { // -1 means EOF, just like C
            out.write(c); // writes c to the output file
        }
    } finally {
        if (in != null) {
            in.close();
        }

        if (out != null) {

            out.close();
        }
    }
}
```

So, the above also shows what the byte stream is. All data in a file, and also the file itself, is stored as binary data, With the byte stream, it allows us to read bytes from the input stream and write bytes to the output stream. The `FileInputStream` and `FileOutputStream` simply inherit from that idea. The above also introduces the `EOFException` which is thrown when we reach the end of a file. Equivalently, like in C, if we try to read, and file is now empty, the return value is -1, indicating EOF.

**⇒ Object input/output (Serialization and Deserialization)**

```Mermaid
flowchart BT
O(ObjectInputStream) ---> I(InputStream)
```

Consider the code below:

```Java
Student s = new student("allan", "FANG DREAM");
```

Now, I want to write this object to a file. Now, you could implement a `toString()` method for the class, and then output the string to the file, but what if I don’t want to do that? What if I want to capture the state of the object? This is called `serizalization` and the other away around (reading an object from a file), is called `deserizliation:`

```Java
// serialization 
Student s = new student("allan", "FANG DREAM");
```

`Serialization` is a mechanism of converting the state of an object into a byte stream.

![[Untitled 10.png|Untitled 10.png]]

To make a Java object serializable we implement the `java.io.Serializable` interface. To serialize an object, we can do:

```Java
try (ObjectOutputStream locFile = 
new ObjectOutputStream(new BufferedOutputStream(
new FileOutputStream("output.dat")))) {
	locFile.writeObject(allan);
 }
// this is done with try with resources, don't neccesarily need to, but good 
```

So, this creates the following data in `output.dat`:

![[Screenshot_2022-12-22_at_9.57.41_PM.png]]

And of course, we can’t read it, since it is now a stream of bytes. To see data, we would need to read, and deserialize the file.

![[Screenshot_2022-12-22_at_9.59.35_PM.png]]

Here is how we would serialize an object, and then deserialize it from the file and print it to the output console:

```Java
// Java code for serialization and deserialization 
// of a Java object
import java.io.*;
  
// Demo is the class we are serializing into the file 
class Demo implements java.io.Serializable
{
		private static final long serialversionUID = 129348938L;
    public int a;
    public String b;
  
    // Default constructor
    public Demo(int a, String b)
    {
        this.a = a;
        this.b = b;
    }
  
}
  
class Test
{
    public static void main(String[] args)
    {   
        Demo object = new Demo(1, "geeksforgeeks");
        String filename = "file.ser";
          
        // Serialization 
        try
        {   
            //Saving of object in a file
            FileOutputStream file = new FileOutputStream(filename);
            ObjectOutputStream out = new ObjectOutputStream(file);


						// we can also wrap this with a BufferedStream
              
            // Method for serialization of object
            out.writeObject(object);
              
            out.close();
            file.close();

						// won't need these if we use try with resources 
              
            System.out.println("Object has been serialized");
  
        }
          
        catch(IOException ex)
        {
            System.out.println("IOException is caught");
        }
  
  
        Demo object1 = null;
  
        // Deserialization
        try
        {   
            // Reading the object from a file
            FileInputStream file = new FileInputStream(filename);
            ObjectInputStream in = new ObjectInputStream(file);
              
            // Method for deserialization of object
            object1 = (Demo)in.readObject();
              
            in.close();
            file.close();
              
            System.out.println("Object has been deserialized ");
            System.out.println("a = " + object1.a);
            System.out.println("b = " + object1.b);
        }
          
        catch(IOException ex)
        {
            System.out.println("IOException is caught");
        }
          
        catch(ClassNotFoundException ex)
        {
            System.out.println("ClassNotFoundException is caught");
        }
  
    }
}
```

Something important to note is the `serialVersionUID`. When we are converting the object into a byte stream, different compilers may do this process differently, which is one of the risks of transporting data as binary data. To try and make sure that this is uniform, even across 2 different compilers, we want to add `serialVersionUID` into the object implementing `serializiable`. Above, we have `private static final long serialversionUID = 129348938L;` and this is typically a long type, and final, as we do not want to be changing it.

### RandomAccessFile Class

**⇒ File Pointer**

![[Screenshot_2022-12-23_at_12.27.46_PM.png]]

This is what every file actually is, binary data that can represent ASCII characters, if it is supposed to. Sometimes, if it is not meant to be read, then it won’t be able to be expressed in ASCII, or even if it can, won’t make sense.

  

Now, what do we mean when we talk about the `RandomAccessFile` class? This allows us to perform read and write operations on a file at specific locations. Consider the image below:

![[Screenshot_2022-12-23_at_1.00.32_PM.png]]

We know that a file is simply a continuous byte stream, and to read it, we go from byte to byte. Now, if we only want to read something in the middle of the file or stream, going one by one means we are wasting a lot of time and energy, which is not ideal. The thing that we use to navigate the file, is what we call the `file pointer`. So, any file opened or created using `RandomAccessFile` class, these files are going to be treated as binary files.

  

The file pointer points to the location at which we are in the file. So, how can we nvagiate to a specific area of a file?

```Java
public static void main(String[] args) throws IOException {
    char[] message  = { 'h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd'};
    long filePointer = 0;
    boolean EOF = false;

    // now, we need to indicate whether we want to use RandomAccessFile woth read only, or read+write
    // rw -> if the file does not exist, the file is created
    // rw -> if the file does exist, its contents are overwritten
    // r -> if the file does not exist, a FileNotFoundException is thrown
    // r -> if we try and write to the file, IOException is thrown

    try(RandomAccessFile randomFile = new RandomAccessFile("Messages.txt", "rw")) {
        for (char letter: message) {
            randomFile.writeChar(letter);
        }

        randomFile.seek(filePointer); 
// this looks for this particular area inside of the file

// now that we have preformed the seek() method, we are now at the place where filePointer is, which in this case, is beginning of the file
        while (!EOF) {
            try {
                System.out.print(randomFile.readChar());
            } catch (EOFException e) {
                EOF = true;
            }
        }
    }
}
```

Now, consider the code above. As seen, it is some pretty basic use case, and not really leveraging the uses of `filePointer` yet. Now, the above code writes the string “hello world” to the file, which is then read back (since file is in binary data), and printed to the console. Above, after we have written the text to the file, we moved the position of the `filePointer` , which was 0, and begin to read from there. This is how we are able to read in the entire sentence. Now, had we moved that `filePointer` to a different location, we would being reading in a different location, instead of the beginning of the file. Now, we can add some code to demonstrate the use of `filePointer` further:

```Java
public static void main(String[] args) throws IOException {
  char[] message  = { 'h', 'e', 'l', 'l', 'o', ',', ' ', 'w', 'o', 'r', 'l', 'd', '!'};
  char[] newMessage = {'E', 'a', 'r', 't', 'h'};

	long filePointer = 7 * Character.BYTES;
  boolean EOF = false;

  // now, we need to indicate whether we want to use RandomAccessFile woth read only, or read+write
  // rw -> if the file does not exist, the file is created
  // rw -> if the file does exist, its contents are overwritten
  // r -> if the file does not exist, a FileNotFoundException is thrown
  // r -> if we try and write to the file, IOException is thrown

  try(RandomAccessFile randomFile = new RandomAccessFile("Messages.txt", "rw")) {
      for (char letter: message) {
          randomFile.writeChar(letter);
      }

      randomFile.seek(filePointer); // this looks for this particular area inside of the file

      for (char letter: newMessage) {
          randomFile.writeChar(letter);
      }

      randomFile.seek(0);

      // now that we have preformed the seek() method, we are now at the place where filePointer is, which in this case, is beginning of the file
      while (!EOF) {
          try {
              System.out.print(randomFile.readChar()); // we are reading from the position of filePointer
          } catch (EOFException e) {
              EOF = true;
          }
      }
  }
}
```

Now, what does the code snippet above do? Again, we write “hello world” to the file. But, after that, we move the filePointer to position 14. Then, we write “Earth” to the file. Then, we move the `filePointer` back to the beginning of the file, and read the entire sentence to the console.
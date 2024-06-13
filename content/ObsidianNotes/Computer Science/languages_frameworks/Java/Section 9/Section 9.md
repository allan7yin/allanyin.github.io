## Concurrency and Threads

### Introduction

To begin, let’s first define some key terms that will be important to know:

- **process:** a unit of execution that has its own memory space. Each instance of a JVM runs as a process (not true for all JVM implementations, but true for most). Every time we are running our Java code in IntelliJ, we are kicking off a “process”.

Many people use the terms **process** and **application** interchangeably, and we will too. If one java application is running and we run another one, each application has its own memory space of **heap**. The first java application cannot access the heap that belongs the second java application. The heap is not shared between them.

- **thread:** a unit of execution within a process. Each process can have multiple threads. In Java, every process (or application) has at least one thread, the **main thread**.

Just about every Java process also has multiple system threads that handle tasks like memory management and I/O. As developers, we don’t explicitly create and code these threads. Our code runs on the main thread, or in other threads that we create. Creating a thread does not require as many resources as creating a process. Every thread created by a process share’s the process’s memory and files. This can create problems, as we’ll see later on.

  

In addition to the process’s memory, or heap, each thread has what’s called a thread stack, which is the memory that only that thread can access. We’ll take a look at this in a later section. Even so, why would we want to be able to use multithreading? What benefits does it introduce as opposed to running everything on the main thread? 2 main reasons:

1. Sometimes, we’ll want to perform a task that will a long amount of time. For example, we may want to query a database, or we may want to fetch data from somewhere on the internet. We could do this on the main thread, but as code is executed line by line, in a linear fashion, the main thread won’t be able to do anything while waiting for the data
    1. In other words, the execution of the main thread will be suspended as it waits for the data to be returned, before it can execute the next line of code.
    2. Instead of tying up the main thread, we can create another thread and execute the long-running task on that thread. This would free up the main thread, so that it can continue executing, report progress, and accept user input while the long-running task continues to execute in the background
    3. Essentially, implementing asynchronous functionality
2. Sometimes, an API will require you to use threads

  

- **concurrency:** refers to an application doing more that one thing at a time This doesn’t mean that the two tasks are being executed at the same time. Just means that progress can be made towards more than one task:
    - To clarify this, consider the following example: _An application wants to download some data and draw a shape on the screen._
    - If it is a concurrent application, it can download a bit of data, then go draw a bit of the shape, then download some more, and etc.
    - Concurrency just means that one task does **NOT** have to be complete before another can start

### Threads

First, let’s see how we make a thread. On way is to create in instance of the thread class. What we’re going to try and do is kick of a thread that is going to sun some code. To tell the thread what code we want to run, we are going to create a subclass of the thread class, and then override the `run()` method. Consider the following situation:

  

Say we have the `main()` method as our first/main thread, and now, we want to create another thread. Here is one way of doing so:

```Java
public class AnotherThread extends Thread {
    @Override
    public void run() {
        System.out.println("Hello, from another thread");
    }
}
```

The `run()` method defines what will be run on this thread. Now, in the `main()` method, in order to run this code, we can:

```Java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, from the main thread");

        Thread anotherThread = new AnotherThread();
        anotherThread.start();
    }
}
```

Notice that we use `anotherThread.start()` to invoke the `run()` method. The `start()` method of the `thread` class is used to begin execution of the thread. The result of this method is two threads that are running concurrently: the current thread (main in this case, which returns from the call to the start method), and the other thread (anotherThread, which executes its run method). Now, take a look at the below screenshot. Notice how the order of output is not what we expected.

![[Screenshot_2022-12-25_at_1.05.31_PM.png]]

Now, the above results are consistent when running the code multiple times. But, in reality, that order is not guaranteed. The order in which the threads are to run is entirely up to the system. Now, we can only “start” each thread once. If we want to call the `run()` method more than once on a class, we will need to create a new instance of that class. Starting an existing thread will throw an exception:

![[Screenshot_2022-12-25_at_1.10.43_PM.png]]

We are also able to create a thread with an anonymous class. Whether to use an anonymous or named class, will depend on the use case. Lets see how we can use the anonymous classes:

![[Screenshot_2022-12-25_at_2.28.41_PM.png]]

Now, the output will always be in a unpredictable order. However, we can add some colours to assist in the viewing experience.

![[Screenshot_2022-12-25_at_2.37.51_PM.png]]

```Java
public class ThreadColour {
    public static final String ANSI_RESET = "\u001B[0m";
    public static final String ANSI_BLACK = "\u001B[30m";
    public static final String ANSI_RED = "\u001B[31m";
    public static final String ANSI_GREEN = "\u001B[32m";
    public static final String ANSI_BLUE = "\u001B[34m";
    public static final String ANSI_PURPLE = "\u001B[35m";
    public static final String ANSI_CYAN = "\u001B[36m";
}
```

**⇒ Runnable and Thread**

The second way to create a thread is to use the `runnable` interface. Like the first way, we need to implement the `run()` method for the interface. With this, we can make any class implement the `runnable` interface, and override the `run()` method.

```Java
// Main.java
public class Main {
    public static void main(String[] args) {
        System.out.println(ThreadColour.ANSI_PURPLE + "Hello, from the main thread");

        Thread anotherThread = new AnotherThread();
        anotherThread.setName("== another thread ==");
        anotherThread.start();

        new Thread() {
            public void run() {
                System.out.println(ThreadColour.ANSI_GREEN + "Hello from the anonymous class");
            }
        }.start();

        // this is the runnable way, with creating a class that impelements the runnable interface
        // We pass this object to the constructor of the Thread class

        //        Thread myRunnableThread = new Thread(new MyRunnable());
        //        myRunnableThread.start();

        // we can also make the thread class above as an anonymous class

        Thread myRunnableThread = new Thread(new MyRunnable() {
            @Override
            public void run() {
                System.out.println(ThreadColour.ANSI_RED + "Hello, from MyRunnable's implementation of run()");
            }
        });

        myRunnableThread.start();

        // Most people prefer the runnable method, as it's shorter, and more used.


        System.out.println(ThreadColour.ANSI_PURPLE + "Hello, again, from the main thread");
    }
}


// MyRunnable.java
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        System.out.println(ThreadColour.ANSI_RED + "Hello, from MyRunnable's implementation of run()");
    }
}
```

Now, using `Runnable` is the preferred method, as generally, more people prefer that method. Now, what would happen, if instead of calling the `start()` method of the thread, we called `run()`. Doing this, we would no longer be creating a new thread, and instead, will be staying on the same thread.

![[Screenshot_2022-12-25_at_3.05.56_PM.png]]

![[Screenshot_2022-12-25_at_3.06.09_PM.png]]

We can also ask the thread to stop or pause for a certain amount of time. To put a thread to sleep, we can simply do:

```Java
public class AnotherThread extends Thread {
	@Override
	public void run() {
	    System.out.println(ThreadColour.ANSI_BLUE + "Hello, from " + currentThread().getName());
	
	    try {
	        Thread.sleep(3000); // pauses the thread for 3 seconds
	        
	    } catch (InterruptedException e) {
	        System.out.println(ThreadColour.ANSI_BLUE + "Another thread woke me up");
	    }
	
	    System.out.println(ThreadColour.ANSI_BLUE + "Three seconds have passed, I'm awake");
	}
	}
```

As seen above, we can put the thread to sleep for 3 seconds. When the time duration is over, the thread is no longer “asleep”, and hence, it is awoken. Now, it is possible for threads to wake up prematurely. We interrupt a thread because we want it to stop what it is doing, and to do something else. More often than not, we wish to interrupt a thread since we want to terminate it. This brings us to our next section:

**⇒ Interrupt and Join**

We can call the `interrupt()` method to interrupt a thread. Consider the following:

```Java
// in main()
anotherThread.interrupt();

// in AnotherThread.java
public class AnotherThread extends Thread {
	@Override
	public void run() {
		System.out.println(ThreadColour.ANSI_BLUE + "Hello, from " + currentThread().getName());
	
		try {
	     Thread.sleep(3000); // pauses the thread for 3 seconds
	
	    } catch (InterruptedException e) {
	        System.out.println(ThreadColour.ANSI_BLUE + "Another thread woke me up");
	        return; // returning here terminates this thread if it is interrupted
	    }
	
	    System.out.println(ThreadColour.ANSI_BLUE + "Three seconds have passed, I'm awake");
	}
	}
```

So, that is the `interrupt()` method. Now, lets look at join. Say we have two threads, and one of the threads cannot continue to execute until it knows the other thread has terminated. To accomplish this, we need to join the two threads. Java provides a `join()` method which allows one thread to wait until another thread completes its execution. Consider the following:

```Java
Thread myRunnableThread = new Thread(new MyRunnable() {
@Override
public void run() {
    System.out.println(ThreadColour.ANSI_RED + "Hello, from MyRunnable's implementation of run()");

    try {
        anotherThread.join();
        System.out.println(ThreadColour.ANSI_RED + "Another thread terminated, so I'm running again");
    } catch(InterruptedException e) {
        System.out.println(ThreadColour.ANSI_RED + "I could not wait after all. I was interrupted");
    }
}
});
```

We are creating a new thread by passing an anonymous `Runnable` class to the `Thread` constructor. When we do `anotherThread.join()` , we are saying to to make sure `anotherThread` is terminated before the next construction. So, once `anotherThread` terminates, the try block will continue. Now, this is a very simple scenario. Now, what if `anotherThread` never terminates? Then, the current code would not progress, and it would seem as though our application had crashed. To prevent such a situation from arising, we can pass a timeout value to the `join()` method. Something like:

```Java
Thread myRunnableThread = new Thread(new MyRunnable() {
@Override
public void run() {
    System.out.println(ThreadColour.ANSI_RED + "Hello, from MyRunnable's implementation of run()");

    try {
        anotherThread.join(5000); // waits 5 seconds
        System.out.println(ThreadColour.ANSI_RED + "Another thread terminated, so I'm running again");
    } catch(InterruptedException e) {
        System.out.println(ThreadColour.ANSI_RED + "I could not wait after all. I was interrupted");
    }
}
});
```

In the above code, if the `anotherThread` does not terminate within 5 seconds, it will continue, regardless of the `anotherThread` has terminated or not. To see some more methods/functionality, consider checking out this link: [https://www.geeksforgeeks.org/java-lang-thread-class-java/?ref=lbp](https://www.geeksforgeeks.org/java-lang-thread-class-java/?ref=lbp).

  

**⇒ Multiple Threads**

Now, the current threads we’ve looked at do not store any sort of variables, they only print output to the console. So, let’s create a couple of more threads, that are more interesting. Consider the following code:

```Java
public class Main {
    public static void main(String[] args) {
        Countdown countdown = new Countdown();
        CountdownThread t1 = new CountdownThread(countdown);
        t1.setName("Thread 1");
        CountdownThread t2 = new CountdownThread(countdown);
        t2.setName("Thread 2");
        t1.start();
        t2.start();
    }
}

class Countdown {
    public void doCountdown() {
        String colour;

        switch(Thread.currentThread().getName()) {
            case "Thread 1":
                colour = ThreadColour.ANSI_CYAN;
                break;
            case "Thread 2":
                colour = ThreadColour.ANSI_PURPLE;
                break;
            default:
                colour = ThreadColour.ANSI_GREEN;
        }

        for(int i = 10; i > 0; i--) {
            System.out.println(colour + Thread.currentThread().getName() + ": i =" + i);
        }
    }
}

class CountdownThread extends Thread {
    private Countdown threadCountdown;
    public CountdownThread(Countdown countdown) {
        threadCountdown = countdown;
    }

    public void run() {
        threadCountdown.doCountdown();
    }
}
```

The output of this is pretty interesting. Since we cannot control the order of execution of threads, the output is different each time, or at least often changes.

![[Screenshot_2022-12-26_at_11.46.22_AM.png]]

![[Screenshot_2022-12-26_at_11.46.38_AM.png]]

Now, something very interesting happens if we use a class variable:

```Java
class Countdown {
    private int i;
    public void doCountdown() {
        String colour;

        switch(Thread.currentThread().getName()) {
            case "Thread 1":
                colour = ThreadColour.ANSI_CYAN;
                break;
            case "Thread 2":
                colour = ThreadColour.ANSI_PURPLE;
                break;
            default:
                colour = ThreadColour.ANSI_GREEN;
        }

        for(i = 10; i > 0; i--) {
            System.out.println(colour + Thread.currentThread().getName() + ": i =" + i);
        }
    }
}
```

Now, notice that in the above code snippet, we are now using a private variable, `i`. Now, observe the output:

![[Screenshot_2022-12-26_at_12.10.33_PM.png]]

**⇒ Thread Variables**

Recall that all threads share the memory saved to the heap. Also recall that every thread has a thread stack, and that’s memory only the stack can access. In other words, thread 1 cannot access thread 2’s thread stack, and vice versa.

  

Local variables are stored in the thread stack, this means that each thread has its own copy of the local variable. Its a different story for an object’s instance value. For that, the memory required is allocated on the heap. So, when multiple threads are working on the same object, they share the same memory for that object. So, when a thread changes the value of one of the objects instance variables, the other threads will also see the value changed. So, this explains the above output.

  

When problems arise because threads are interleaving, and accessing the same resources, we call this **thread interference**. This describes the image above. Whenever a thread has the chance to run, the value of `i` has already been changed by the other thread. As a result, each thread is no longer able to print all numbers from 10 to 1. Of course, there would be no issue if the two threads were only reading the shared resource. But, as long as one thread modifies the shared resource, a problem arises.

  

**⇒ Synchronization**

What is a race condition? Well, its just another way of saying **thread interference**. Let’s go through another example to demonstrate the **race condition**. Now, for the previous code example, if we simply have the two thread’s work with two distinct objects, there would be no interference. It is precisely because we passed both threads the same object, that there was interference. The fix is extremely simple:

```Java
public static void main(String[] args) {
  Countdown countdown1 = new Countdown();
	Countdown countdown2 = new Countdown();
	
	CountdownThread t1 = new CountdownThread(countdown1);
	t1.setName("Thread 1");
	CountdownThread t2 = new CountdownThread(countdown2);
	t2.setName("Thread 2");
	
	t1.start();
	t2.start();
}
```

However, this is far from an ideal solution. Say we were working at a bank, and the object was an individual’s banking data. There could many threads operating on that object, and if each one was given a copy, how would we merge the changed objects? Likewise, for such important data, its probably not the best idea to making copies of it.

  

To address this issue, we now introduce using the `synchronized` keyword, and creating `synchronized` methods. To use this, we simply add the `synchronized` keyword in front of the method declaration:

```Java
public synchronized void doCountdown() {
  String colour;

  switch(Thread.currentThread().getName()) {
      case "Thread 1":
          colour = ThreadColour.ANSI_CYAN;
          break;
      case "Thread 2":
          colour = ThreadColour.ANSI_PURPLE;
          break;
      default:
          colour = ThreadColour.ANSI_GREEN;
  }

  for(i = 10; i > 0; i--) {
      System.out.println(colour + Thread.currentThread().getName() + ": i =" + i);
  }
}
```

So, what does making a method `synchronized` do? In essence, it forces whatever thread is running, to finish what it is doing, before it can be suspended for the next thread to run. There are 2 effects:

1. It is not possible for two invocations of synchronized methods on the same object to interleave. When one thread is executing a synchronized method for an object, all other threads that invoke synchronized methods for the same object block (suspend execution) until the first thread is done with the object
2. When a synchronized method exits, it automatically establishes a happens-before relationship with any _subsequent invocation_ of a synchronized method for the same object. This guarantees that changes to the state of the object are visible to all threads.

Synchronized methods enable a simple strategy for preventing thread interference and memory consistency errors: if an object is visible to more than one thread, all reads or writes to that object's variables are done through `synchronized` methods.

  

Another way we can prevent `thread interference` is by synchronizing specific statements within a method, instead of making the entire method synchronized. Every object in Java has something called an `intrinsic lock`.

---

_The following is a quick note on the intrinsic lock, and synchronization from Java documentation, as there was little detailed explanation in the lectures._

  

- **Happens-Before Relationship:** This is a concept, or set of rules, that define the basis for reordering of instructions by a compiler or CPU. This is not a keyword or object in Java, it is simply a discipline in put into place so that in a multi-threading environment, the reordering of the surrounding instructions does not result in a code that produces incorrect output.

Consider the JMM (Java Memory Model):

![[Untitled 11.png|Untitled 11.png]]

From this, we recall that local variables and references are stored on the `Thread Stack` whereas objects are stored on the heap. To prevent something like the `race condition` from appearing, we introduced `synchronized` methods and code statements. Let’s look at what `Instruction reordering` is. Consider the following:

```Java
FullName = FirstName + LastName        // Statement 1
UniqueId = FullName + TokenNo         // Statement 2
 
Age = CurrentYear - BirthYear        // Statement 3
```

The compiler cannot run statements 1 and 2 in parallel because 2 needs the output of 1. However, 1 and 3 can run in parallel because they are independent of each other. So the compiler can reorder the code into:

```Java
FullName = FirstName + LastName      // Statement 1
Age = CurrentYear - BirthYear       // Statement 3

UniqueId = FullName + TokenNo        // Statement 2
```

However, if reordering is performed in a multi-threaded application where threads share some variables, then it may cost us the correctness of our program.

  

When a thread enters a `synchronization` block, the thread will refresh the values of all variables that are visible to the thread at that time from the main memory. When a thread exits a synchronization block, the values of all those variables will be written to the main memory. Applying `Happens-Before` to `synchronization` blocks, we get the following rules:

- Any write to a variable that happens before the exit of a synchronization block is guaranteed to remain before the exit of a synchronization block.
- Entrance to a synchronization block that happens before a read of a variable, is guaranteed to remain before any of the reads to the variables that follow the entrance of a synchronized block.

  

Now that we know what the `Hapens-Before` relationship details, let us return to the `intrinsic lock`. `Intrinsic locks` play a role in both aspects of synchronization: enforcing exclusive access to an object's state and establishing happens-before relationships that are essential to visibility. Each object has an `intrinsic lock`, and by default, a thread that needs exclusive and consistent access to an object’s fields has to acquire the object’s `intrinsic lock` before accessing the, and then release it when it is done. As long as a thread owns the `intrinsic lock`, no other thread can acquire the same lock. When a thread releases an intrinsic lock, a happens-before relationship is established between that action and any subsequent acquisition of the same lock. Lets look at locks for the two types of `synchronization` we work with:

1. **Synchronized Methods**
    1. When a thread invokes a synchronized method, it automatically acquires the `intrinsic lock` for that object, and releases it when the method returns.
2. **Synchronized Statements**
    
    1. Unlike synchronized methods, synchronized statements must specify the object that provides the intrinsic lock:
    
    ```Java
    public void addName(String name) {
        synchronized(this) {
            lastName = name;
            nameCount++;
        }
        nameList.add(name);
    }
    ```
    
    Again, in that block, the synchronized statement owns the `intrinsic lock` and when done, releases it.
    

---

Now, we can also synchronize static methods and use static objects. Now, `synchronization` is re-entrant, which means that if a thread acquires an object’s lock and within the synchronized code, it calls a method that is using the same object to synchronize some code, the thread can keep executing because it already has the object’s lock. In other words, a thread can acquire a lock it already owns. If this was not possible, synchronization would be a lot harder to work with.

- **Critical Section** refers to the code that’s referencing a shared resource, like a variable. Only one thread at a time should be able to execute a critical section.
- **ThreadSafe:** When a class or method is thread safe, it means that the developer has synchronized all the critical sections within the code so that, as a developer, we do not have to worry about the thread interface.

Before moving on, there is one last thing that is important to note, and that is to only synchronize code that must be synchronized. We’ve seen how we can synchronize both code statements, or entire methods. Sometimes, synchronizing the entire method is unnecessary. In conclusion, we want to keep the amount of code we need to synchronize to a minimum.

  

**⇒ Producer and Consumer**

Now lets look at the only methods that can be called within synchronized code, namely `weight` , `notify`, and `notify all` methods. The classic example that demonstrates the use case of these methods is the producer consumer example. We’ll do something similar, where we’ll create a writer (producer) and a reader (consumer). Consider the following code:

```Java
import java.util.Random;

public class Main {
    public static void main(String[] args) {
        Message message = new Message();
        Thread thread1 = new Thread(new Writer(message));
        Thread thread2 = new Thread(new Reader(message));

        thread1.start();
        thread2.start();
    }
}

class Message {
    private String message;
    private boolean empty = true;
    public synchronized String read() {
        while(empty) {

        }

        empty = true;
        return message;
    }

    public synchronized void write(String message) {
        while(!empty) {

        }

        empty = false;
        this.message = message;
    }
}

class Writer implements Runnable {
    private Message message;
    public Writer(Message message) {
        this.message = message;
    }

    @Override
    public void run() {
        String[] messages = {
                "Humpty Dumpty sat on a wall",
                "Humpty Dumpty had a great fall",
                "All the king's horses and all the king's men",
                "Couldn't possibly put Humpty together again"
        };

        Random random = new Random(); // creates something random from the random class

        for (int i = 0; i < messages.length; i++) {
            message.write(messages[i]);
            try {
                Thread.sleep(random.nextInt(2000));
            } catch (InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }
        message.write("Finished");
    }
}

class Reader implements Runnable {
    private Message message;
    public Reader(Message message) {
        this.message = message;
    }

    @Override
    public void run() {
        Random random = new Random();
        for (String latestMessage = message.read(); !latestMessage.equals("Finished"); latestMessage = message.read()) {
            System.out.println(latestMessage);
            try {
                Thread.sleep(random.nextInt(2000));
            } catch (InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }
    }
}
```

The above code has a problem, it will end up in a dead loop. Remember that only one `synchronized` method can run at one time. We cannot guarantee which thread will go first. If read thread goes first, we will have a dead-loop. Even if we put the threads to sleep, it is still not enough. If the write method or read method is run back to back, it instantly results in a dead-loop. When in the loop, the value of `empty` cannot be changed by the other thread, as it is both inside of the `synchronized` loop, and so, still owns the object lock. This is what is known as a `deadlock`. So, how do we get around this problem? This is where we introduce the `wait`, `notify`, and `notify all` methods. When a thread calls the `wait()` method, it will suspend execution and release whatever locks its holding until another thread issues a notification that something important has happened. Here is how we would make the needed fixes:

```Java
import java.util.Random;

public class Main {
    public static void main(String[] args) {
        Message message = new Message();
        Thread thread1 = new Thread(new Writer(message));
        Thread thread2 = new Thread(new Reader(message));

        thread1.start();
        thread2.start();
    }
}

class Message {
    private String message;
    private boolean empty = true;
    public synchronized String read() {
        while(empty) {
            try {
                wait();
            } catch(InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }

        empty = true;
        notifyAll();
        return message;
    }

    public synchronized void write(String message) {
        while(!empty) {
            try {
                wait();
            } catch(InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }

        empty = false;
        notifyAll();
        this.message = message;
    }
}

class Writer implements Runnable {
    private Message message;
    public Writer(Message message) {
        this.message = message;
    }

    @Override
    public void run() {
        String[] messages = {
                "Humpty Dumpty sat on a wall",
                "Humpty Dumpty had a great fall",
                "All the king's horses and all the king's men",
                "Couldn't possibly put Humpty together again"
        };

        Random random = new Random(); // creates something random from the random class

        for (int i = 0; i < messages.length; i++) {
            message.write(messages[i]);
            try {
                Thread.sleep(random.nextInt(2000));
            } catch (InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }
        message.write("Finished");
    }
}

class Reader implements Runnable {
    private Message message;
    public Reader(Message message) {
        this.message = message;
    }

    @Override
    public void run() {
        Random random = new Random();
        for (String latestMessage = message.read(); !latestMessage.equals("Finished"); latestMessage = message.read()) {
            System.out.println(latestMessage);
            try {
                Thread.sleep(random.nextInt(2000));
            } catch (InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }
    }
}
```

Now, we still keep the loops, and notice that we have the `wait()` method inside of the loop. Now, we always want to have `wait()` within a loop that is testing for whatever condition we’re waiting on. This is because if a thread is being woken up, theres is no guarantee it is because the condition has changed. A couple of things to remember:

1. When synchronizing code, always keep in mind that threads can be suspended while executing a single line of code. A single line of code can call a method, which can then mean another 100 lines of code are being executed. A thread can be suspended before or after performing any of these operations and also while executing the method itself.
2. That being said, there are a few atomic operations in Java that can happen all at once. This means that a thread cannot be suspended in the middle of doing them. They are:
    - Reading and Writing reference variables
    - Reading and writing primitive variables (except `long` and `double`)
        - so, thread cannot suspend in the middle of executing `int my = 10;` , but can be suspended for something like `double lol = 1.234`

  

Now, it is important to note that some collections are not thread safe. For example, `arraylist` is not thread safe. The Java documentation states that it is not implemented using `synchronized`. What this mean is that when we are using an `arrayList` , and multiple threads can access that at the same time, we are responsible for synchronizing the code that uses the `arrayList`. To learn more about these things, refer to Oracle’s documentation. One thing worth noting however, there is also `vector` in Java, much like in C++. It is the same thing as an `ArrayList`, but vector is thread safe. `vector` also implements the `List` interface, so anything you can do to a list, works for `vector`. This pretty much wraps up `sychronization` and we’ll look at new things moving forward.
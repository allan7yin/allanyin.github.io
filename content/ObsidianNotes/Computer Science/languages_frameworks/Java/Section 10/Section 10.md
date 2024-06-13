## Threads & Concurrency Continued

**⇒ The Java Util Concurrent Package**

Java introduced a package to help developers write code that used multiple threads, `java.util.concurrent` package. Consider the following code:

```Java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;


public class Main {
    public static final String EOF = "EOF";
    public static void main(String[] args) {
        List<String> buffer = new ArrayList<>();
        MyProducer producer = new MyProducer(buffer, ThreadColour.ANSI_YELLOW);
        MyConsumer consumer1 = new MyConsumer(buffer, ThreadColour.ANSI_PURPLE);
        MyConsumer consumer2 = new MyConsumer(buffer, ThreadColour.ANSI_CYAN);

        new Thread(producer).start();;
        new Thread(consumer1).start();;
        new Thread(consumer2).start();;
    }
}

class MyProducer implements Runnable{
    private List<String> buffer;
    private String colour;

    public MyProducer(List<String> buffer, String colour) {
        this.buffer = buffer;
        this.colour = colour;
    }

    @Override
    public void run() {
        Random random = new Random();
        String[] nums = {"1", "2", "3", "4", "5"};

        for (String num: nums) {
            try {
                System.out.println(colour + "Adding..." + num);
                buffer.add(num);

                Thread.sleep(random.nextInt(1000));
            } catch(InterruptedException e) {
                System.out.println("Producer was interrupted");
            }
        }

        System.out.println(colour + "Adding EOF and exiting...");
        buffer.add("EOF");
    }
}


class MyConsumer implements Runnable {
    private List<String> buffer;
    private String colour;

    public MyConsumer(List<String> buffer, String colour) {
        this.buffer = buffer;
        this.colour = colour;
    }

    @Override
    public void run() {
        while(true) {
            if (buffer.isEmpty()) {
                continue;
            }

            if (buffer.get(0).equals("EOF")) {
                System.out.println(colour + "Exiting");
                break;
            } else {
                System.out.println(colour + "Removed " + buffer.remove(0));
            }
        }
    }
}
```

At this point, we should know that the above code will result in issues, primarily due to the fact that we have 3 threads sharing the same `ArrayList`, something which is not thread safe. So, what is happening above? Both threads are running at the same time. However, sometimes, using `buffer.get(0)` is risky, since if that index has it’s value removed, and at the moment, the `arrayList` item was just removed, this can result in a exception being thrown. To fix this, we want to place any code that works with moving values inside of the `arrayList` inside synchronized blocks of code. This is especially important since we have two consumers. They are two completely independent consumer objects, so we don’t synchronize with `this` as a parameter. So, something like this:

```Java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

//import static Main.EOF;

public class Main {
    public static final String EOF = "EOF";

    public static void main(String[] args) {
        List<String> buffer = new ArrayList<String>();
        MyProducer producer = new MyProducer(buffer, ThreadColour.ANSI_YELLOW);
        MyConsumer consumer1 = new MyConsumer(buffer, ThreadColour.ANSI_PURPLE);
        MyConsumer consumer2 = new MyConsumer(buffer, ThreadColour.ANSI_CYAN);

        Thread thread1 = new Thread(producer);
        Thread thread2 = new Thread(consumer1);
        Thread thread3 = new Thread(consumer2);

        thread1.start();
        thread2.start();
        thread3.start();
    }
}

class MyProducer implements  Runnable {
    private List<String> buffer;
    private String color;

    public MyProducer(List<String> buffer, String color) {
        this.buffer = buffer;
        this.color = color;
    }

    public void run() {
        Random random = new Random();
        String[] nums = { "1", "2", "3", "4", "5"};

        for(String num: nums) {
            try {
                System.out.println(color + "Adding..." + num);
                synchronized (buffer) {
                    buffer.add(num);
                }

                Thread.sleep(random.nextInt(1000));
            } catch(InterruptedException e) {
                System.out.println("Producer was interrupted");
            }
        }

        System.out.println(color + "Adding EOF and exiting....");
        synchronized (buffer) {
            buffer.add("EOF");
        }
    }
}

class MyConsumer implements Runnable {
    private List<String> buffer;
    private String color;

    public MyConsumer(List<String> buffer, String color) {
        this.buffer = buffer;
        this.color = color;
    }

    public void run() {
        while(true) {
            synchronized (buffer) {
                if (buffer.isEmpty()) {
                    continue;
                }
                if (buffer.get(0).equals("EOF")) {
                    System.out.println(color + "Exiting");
                    break;
                } else {
                    System.out.println(color + "Removed " + buffer.remove(0));
                }
            }
        }
    }
}
```

The output we get is:

![[Screenshot_2022-12-31_at_12.05.12_PM.png]]

This is desired output. Now, thread interference no longer occurs. The reason for this is because the producer sleeps after adding a string, usually, a consumer thread will have the opportunity to run and remove the last string added, before the producer thread runs again. Notice that only one consumer is able to remove an item at a time. Before the introduction of the `java.util.concurrent` package, `synchonrization` was the only method we could use to deal with thread interference. However, it does still have its drawbacks. Below are some of the drawbacks:

1. Threads that are blocked waiting to execute synchronized code can’t be interrupted. Once they are blocked, they are stuck there until they get the lock for the object the code is synchronizing on. This can lead to many problems, which we will look at more closely in future lectures.
2. The second drawback is that the synchronized block must be within the same method. In other words, we cannot start a synchronized block in one method, and end the synchronization block in another, for obvious reasons.
3. The third drawback is that we cannot test to see if an object’s intrinsic lock is available, or find out any other information about that block. Also, if the lock isn't available, we can't time out after we've waited for the lock for a while. When we reach the beginning of a synchronized block, we can either get the lock and continue executing or block at that line of code until we get the lock.
4. And the fourth drawback is that if multiple threads are waiting to get a lock, it's not first come first served. There isn't a set order in which the JVM will choose the next thread that gets the lock. So the first thread that blocked could be the last thread to get the lock and vice versa.

  

So, instead of using `synchronization` , we can prevent thread interference using classes that implement the `java.util.concurrent.locks` lock interface.

  

**⇒ Reentrant Lock and Unlock**

The `Lock` implementations that Java provide, do not have the issues that `synchronization` has in the above mentioned list. To learn more, let’s refactor our Consumer and Producer classes to utilize `Locks`. We can instantiate a `ReentrantLock` as follows:

```Java
ReentrantLock bufferLock = new ReentrantLock();
```

As the name implies, this lock is reentrant, meaning that if a thread already holding a reentrant lock when it reaches code that requires the same lock, it can continue executing. While this may sound like a given, not all Locks are reentrant, meaning the thread will need to go and reacquire the lock. First, for all threads we want to prevent thread interference on, we will need to pass the same `ReentrantLock` to all of the threads, they need to share the same lock. Then, here is how we would refactor the producer class:

```Java
class MyProducer implements  Runnable {
    private List<String> buffer;
    private String color;
    private ReentrantLock bufferLock;

    public MyProducer(List<String> buffer, String color, ReentrantLock bufferLock) {
        this.buffer = buffer;
        this.color = color;
        this.bufferLock = bufferLock;
    }

    public void run() {
        Random random = new Random();
        String[] nums = { "1", "2", "3", "4", "5"};

        for(String num: nums) {
            try {
                System.out.println(color + "Adding..." + num);

                bufferLock.lock();
                buffer.add(num);
                bufferLock.unlock();

                Thread.sleep(random.nextInt(1000));
            } catch(InterruptedException e) {
                System.out.println("Producer was interrupted");
            }
        }

        System.out.println(color + "Adding EOF and exiting....");
        
        bufferLock.lock();
        buffer.add("EOF");
        bufferLock.unlock();
    }
}
```

Now, we hold an instance of the lock inside of our class. Then, much like how we worked with `synchronized` code, we wrap the code we want to be thread-safe. Before we execute that code, we do a `.lock()` method, and after we are done, we need to `.unlock()`. Now, the fallback of this is that control of the object lock now completely falls into our responsibility. Unlike the `intrinsic lock` which had its lock acquisition and release done automatically. To learn more, refer to the Oracle documentation:

> [!info] Lock (Java Platform SE 7 )  
> All Known Implementing Classes: ReentrantLock, ReentrantReadWriteLock.  
> [https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/Lock.html](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/Lock.html)  

In general, when working with `Lock`, or `ReentrantLock`, we should always try to follow the following idiom:

```Java
Lock l = ...;
l.lock();
try {
	// something inside of the try block 
	// access to the resource protected by this lock 
} finally {
	l.unlock();
}
```

The `finally` block is used to put important code, and will run regardless of if an exception is thrown. So, it is perfect for dealing with `Lock`s. If we have code that has many areas that a loop is continuing, or breaking, we will need to remember to unlock the lock before leaving. This can leave our code very clustered, with `.unlock()` scattered throughout the entire method. To make our code cleaner, we wrap everything in a try block, and unlock in the finally statement. We’ll look at more reasons to follow this idiom.

**⇒ Using Try with Finally Threads**

- We really need to unlock in one place
- Another reason is that it is possible for code within a critical section to throw an exception we’re not explicitly handling using a catch statement. Regardless, we want to make sure we have released any locks we are holding.

  

A thread can actually test and to see if a lock is available, with `tryLock()` method. This will lock if the lock is available. So, we can reformat code:

```Java
public void run() {
        while (true) {
            if (bufferLock.tryLock()) {
                try {
                    if (buffer.isEmpty()) {
                        continue;
                    }
                    if (buffer.get(0).equals("EOF")) {
                        System.out.println(color + "Exiting");
                        break;
                    } else {
                        System.out.println(color + "Removed " + buffer.remove(0));
                    }
                } finally {
                    bufferLock.unlock();
                }
            }
        }
    }
```

Now, this is a loop, and is running continuously. We can count how many times `.tryLock()` returns false, and when we get the lock, we’ll print the value to the console.

  

**⇒ Thread Pools**

Now, we’re going to look at something called the `ExecutorService` interface. We use implementations of this interface to manage threads for us, so that we don’t need to explicitly create and start threads, like what we’ve already been doing. Now, we still need to create our `Runnable` implementations, we just not no longer need to manage the threads ourselves. So, what is a thread pool? A `Thread Pool` is a managed set of threads. It reduces the overhead of thread creation especially in applications that use a large number of threads. A thread pool may also limit the number of threads that are active. Here is how we would use this:

```Java
public static void main(String[] args) {
  List<String> buffer = new ArrayList<String>();
  ReentrantLock bufferLock = new ReentrantLock();

  ExecutorService executorService = Executors.newFixedThreadPool(3);
  // we pass 3, since we want the limit to be 3 threads, as we have one producer, two consumer threads

  MyProducer producer = new MyProducer(buffer, ThreadColour.ANSI_YELLOW, bufferLock);
  MyConsumer consumer1 = new MyConsumer(buffer, ThreadColour.ANSI_PURPLE, bufferLock);
  MyConsumer consumer2 = new MyConsumer(buffer, ThreadColour.ANSI_CYAN, bufferLock);

  executorService.execute(producer);
  executorService.execute(consumer1);
  executorService.execute(consumer2);

  // but now, to use executorService, we need to shut down manually, as it will run even after main thread is done
  // if we do not shut it down
  executorService.shutdown();

  // now, using executorService is overkill for this type of tiny application, but it is very vital when working
  // on applications involving a large number of threads as executorService allows the JVM to optimize thread
  // management 
}
```

  

In addition, we can also have return values from threads. To do so, we use the `submit()` method. The submit method accepts a `callable` object which is very similar to a `runnable` object, except that it can return a value. The value can be returned as an object of type `future`. So, let’s look at an example of how we can do this:

```Java
public class Main {
	ExecutorService service = Executors.newFixedThreadPool(3);
	Future<String> future = service.submit(new Task());
	// we pass callable object to the submit method 

	// perform some unrelated operations 

	try {
		System.out.println(future.get()); // future.get() blocks until the result is avialable
	} catch (ExecutionException e) {
		System.out.println("Something went wrong");
	} catch (InterruptedException e) {
		System.out.println("Thread running the task was interrupted");
	}
}

class Task implements Callable<String> {
	@Override
	public Integer call() throws Exception {
		Thread.sleep(3000); // sleep the thread for 3 seconds 
		return "I am inside of the Callable";
	}
}
```

Now, lets consider the code above. After we call the submit method, the `call()` method is now executing on a new thread. Within the call method, we sleep the thread for 3 seconds. During that time, we can perform other, unrelated, operations on the main thread. Then, once those 3 seconds are up, we can call `future.get()` which will return the desired value from the `Callable` object. Now, if the unrelated operations finish within, say 1 second, and we call `future.get()` before it is done executing, in this case, the thread is blocked on `future.get()` and will not progress until that return value is ready. Future is simply a placeholder for the value being returned, that will arrive, sometime in the future.

  

Consider this code:

```Java
ExecutorService executorService = Executors.newFixedThreadPool(3);
  // we pass 3, since we want the limit to b  threads, as we have one producer, two consumer threads

  MyProducer producer = new MyProducer(buffer, ThreadColour.ANSI_YELLOW, bufferLock);
  MyConsumer consumer1 = new MyConsumer(buffer, ThreadColour.ANSI_PURPLE, bufferLock);
  MyConsumer consumer2 = new MyConsumer(buffer, ThreadColour.ANSI_CYAN, bufferLock);

  executorService.execute(producer);
  executorService.execute(consumer1);
  executorService.execute(consumer2);

  Future<String> future = executorService.submit(new Callable<String>() {
```

This produces the output:

```Java
Adding...1
The counter = 56751
Removed 1
Adding...2
The counter = 80993737
Removed 2
Adding...3
The counter = 17590552
Removed 3
Adding...4
The counter = 167478945
Removed 4
Adding...5
The counter = 27794668
Removed 5
Adding EOF and exiting....
The counter = 78630151
Exiting
The counter = 222502956
I'm being printed from the Callable class
Exiting
This is the Callable class result
```

Recall that our thread pool here is 3, which means only 3 threads can be running at a time. At the very end, when the threads end, the `Callable` thread is then able to run, and print to the console. However, if we increase the thread pool to accomodate for the `Callable`, then we will see:

```Java
I'm being printed from the Callable class
This is the Callable class result
Adding...1
The counter = 46324
Removed 1
Adding...2
The counter = 114098712
Removed 2
Adding...3
The counter = 1967558
Removed 3
Adding...4
The counter = 2704124
Removed 4
Adding...5
The counter = 156565595
Removed 5
Adding EOF and exiting....
The counter = 46687986
Exiting
The counter = 59262557
Exiting
```

To learn more about the `ExecutorService` refer the oracle documentation:

> [!info] ExecutorService (Java Platform SE 8 )  
> An ExecutorService can be shut down, which will cause it to reject new tasks.  
> [http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html)  

  

**⇒ ArrayBlockingQueue Class**

Let’s quickly take a look at a set of classes available in the `java.util.concurrent` package that allows us to simplify our application considerably. The package contains **thread safe queues**, which are perfect for producing consumer-type applications. Queues are FIFO, just like any line in any store, first in, first out. We introduce the `ArrayBlockingQueue`:

```Java
ArrayBlockingQueue<String> buffer = new ArrayBlockingQueue<String>(6);
```

Now, incidentally, `ArrayBlockingQueue` is bounded, which means that we need to pass in a parameter that tells it how many elements the array should be able to hold. These do not grow like `ArrayLists` do.

> [!info] ArrayBlockingQueue (Java Platform SE 7 )  
> This class supports an optional fairness policy for ordering waiting producer and consumer threads.  
> [https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ArrayBlockingQueue.html](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ArrayBlockingQueue.html)  

Here is how we can refactor the `producer` `consumer` code to use this instead of a simple `ArrayList`

```Java
import java.util.Random;
import java.util.concurrent.*;

public class Main {
    public static final String EOF = "EOF";

    public static void main(String[] args) {
        ArrayBlockingQueue<String> buffer = new ArrayBlockingQueue<String>(6);

        ExecutorService executorService = Executors.newFixedThreadPool(5);
        // we pass 3, since we want the limit to b  threads, as we have one producer, two consumer threads

        MyProducer producer = new MyProducer(buffer, ThreadColour.ANSI_YELLOW);
        MyConsumer consumer1 = new MyConsumer(buffer, ThreadColour.ANSI_PURPLE);
        MyConsumer consumer2 = new MyConsumer(buffer, ThreadColour.ANSI_CYAN);

        executorService.execute(producer);
        executorService.execute(consumer1);
        executorService.execute(consumer2);

        Future<String> future = executorService.submit(new Callable<String>() {
            // the type for the Future must be the same as the Callable's type, and the return type of call
            @Override
            public String call() throws Exception {
                System.out.println(ThreadColour.ANSI_WHITE + "I'm being printed from the Callable class");
                return "This is the Callable class result";
            }
        });

        try {
            System.out.println(future.get()); // future.get() blocks until the result is avialable
        } catch (ExecutionException e) {
            System.out.println("Something went wrong");
        } catch (InterruptedException e) {
            System.out.println("Thread running the task was interrupted");
        }

        // but now, to use executorService, we need to shut down manually, as it will run even after main thread is done
        // if we do not shut it down
        executorService.shutdown();

        // now, using executorService is overkill for this type of tiny application, but it is very vital when working
        // on applications involving a large number of threads as executorService allows the JVM to optimize thread
        // management
    }
}

class MyProducer implements Runnable {
    private ArrayBlockingQueue<String> buffer;
    private String color;

    public MyProducer(ArrayBlockingQueue<String> buffer, String color) {
        this.buffer = buffer;
        this.color = color;
    }

    public void run() {
        Random random = new Random();
        String[] nums = {"1", "2", "3", "4", "5"};

        for (String num : nums) {
            try {
                System.out.println(color + "Adding..." + num);
                buffer.put(num);

                Thread.sleep(random.nextInt(1000));
            } catch (InterruptedException e) {
                System.out.println("Producer was interrupted");
            }
        }

        System.out.println(color + "Adding EOF and exiting....");

        try {
            buffer.put("EOF");
        } catch (InterruptedException e) {
            System.out.println();
        }
    }
}

class MyConsumer implements Runnable {
    private ArrayBlockingQueue<String> buffer;
    private String color;

    public MyConsumer(ArrayBlockingQueue<String> buffer, String color) {
        this.buffer = buffer;
        this.color = color;
    }

    public void run() {
        while (true) {
            try {
                if (buffer.isEmpty()) {
                    continue;
                }

                if (buffer.peek().equals("EOF")) {
                    System.out.println(color + "Exiting");
                    break;
                } else {
                    System.out.println(color + "Removed " + buffer.take());
                }

                // for peek and take, no need to specify element, takes the first element of the queue, as it should x
            } catch (InterruptedException e) {

            }
        }
    }
}
```

`ArrayBlockingQueue` is part of the Java Collections Framework. The `take()` method is used to retrieve and remove, the head of the queue. If the queue is empty, then it will block, and wait until until an element becomes available. The `peek()` method is used to return the head of the queue. It retrieves, but does not remove it. If the queue is empty, this returns `null`. To add items to the queue, we just use the `.put()` method.

  

Now, running the above code can result in some problems. Sometimes, when we run the code, we will get the following error:

```Java
Exception in thread "pool-1-thread-3" java.lang.NullPointerException: Cannot invoke "String.equals(Object)" because the return value of "java.util.concurrent.ArrayBlockingQueue.peek()" is null
	at MyConsumer.run(Main.java:98)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
	at java.base/java.lang.Thread.run(Thread.java:833)
```

Now, why is that? Inside of the `run()` method for the consumer:

```Java
if (buffer.isEmpty()) {
    continue;
}

if (buffer.peek().equals("EOF")) {
    System.out.println(color + "Exiting");
    break;
} 
```

The consumer thread can be suspended between calling `.isEmpty()` and `.peek()`. If one consumer thread is suspended there, the other consumer thread can run and take the next element from the queue. Then, when we execute `.peek()`, we get `null` returned, where the exception is then thrown when trying to do `.equals()` on `null`. To prevent this, we can add synchronization code, even if we’re using a thread-safe class, such as `ArrayBlockingQueue`.

  

**⇒ Deadlocks**

A deadlock occurs when two or more threads are blocked holding a lock that another blocked thread wants. Let’s see an example of how something like this happens:

```Java
package com.timbuchalka;

public class Main {

    public static Object lock1 = new Object();
    public static Object lock2 = new Object();

    public static void main(String[] args) {
        new Thread1().start();
        new Thread2().start();

    }

    private static class Thread1 extends Thread {
        public void run() {
            synchronized (lock1) {
                System.out.println("Thread 1: Has lock1");
                try {
                    Thread.sleep(100);
                } catch(InterruptedException e) {

                }
                System.out.println("Thread 1: Waiting for lock 2");
                synchronized (lock2) {
                    System.out.println("Thread 1: Has lock1 and lock2");
                }
                System.out.println("Thread 1: Released lock2");
            }
            System.out.println("Thread 1: Released lock1. Exiting...");
        }
    }

    private static class Thread2 extends Thread {
        public void run() {
            synchronized (lock2) {
                System.out.println("Thread 2: Has lock2");
                try {
                    Thread.sleep(100);
                } catch(InterruptedException e) {

                }
                System.out.println("Thread 2: Waiting for lock1");
                synchronized (lock1) {
                    System.out.println("Thread 2: Has lock2 and lock1");
                }
                System.out.println("Thread 2: released loc12");
            }
            System.out.println("Thread 2: Released lock2. Exiting...");
        }
    }
}
```

So, the above will result in a deadlock. Why? In `Thread1`, we begin by first acquiring `lock1`, and then sleeping. When `Thread1` sleeps, `Thread2` runs. and acquires `lock2`, then it sleeps. At this point, `Thread1` wakes up, and attempts to get `lock2`, which is still owned by `Thread2`, so `Thread1` blocks here. Similarly, `Thread2` attempts to get `lock1`, which is owned by `Thread1`, so `Thread2` also blocks. Now, no thread is able to continue executing, and hence, we have a deadlock.

  

So, how can we prevent deadlock situations from happening? One way to try and prevent this, is to require all threads must first try to obtain the locks in the same order. The deadlock in the code above was only possible, as the order of the locks accessed in the two threads was not the same. To fix this, we simply make sure both threads are attempting to get the locks in the same order:

```Java
public class Main {

    public static Object lock1 = new Object();
    public static Object lock2 = new Object();

    public static void main(String[] args) {
        new Thread1().start();
        new Thread2().start();

    }

    private static class Thread1 extends Thread {
        public void run() {
            synchronized (lock1) {
                System.out.println("Thread 1: Has lock1");
                try {
                    Thread.sleep(100);
                } catch(InterruptedException e) {

                }
                System.out.println("Thread 1: Waiting for lock 2");
                synchronized (lock2) {
                    System.out.println("Thread 1: Has lock1 and lock2");
                }
                System.out.println("Thread 1: Released lock2");
            }
            System.out.println("Thread 1: Released lock1. Exiting...");
        }
    }

    private static class Thread2 extends Thread {
        public void run() {
            synchronized (lock1) {
                System.out.println("Thread 2: Has lock1");
                try {
                    Thread.sleep(100);
                } catch(InterruptedException e) {

                }
                System.out.println("Thread 2: Waiting for lock2");
                synchronized (lock2) {
                    System.out.println("Thread 2: Has lock1 and lock2");
                }
                System.out.println("Thread 2: released lock2");
            }
            System.out.println("Thread 2: Released lock1. Exiting...");
        }
    }
}
```

  

**⇒ More on deadlocks**

Let’s look at another way a deadlock can occur. Consider the following code:

```Java
public class Main {
    public static void main(String[] args) {
        Data data = new Data();
        Display display = new Display();
        data.setDisplay(display);
        display.setData(data);
    }

    private static class Data {
        private Display display;
        public void setDisplay(Display display) {
            this.display = display;
        }

        public synchronized void updateData() {
            System.out.println("Updating data ...");
            display.dataChanged();
        }

        public synchronized Object getData() {
            return new Object();
        }
    }

    private static class Display {
        private Data data;
        public void setData(Data data) {
            this.data = data;
        }

        public synchronized void dataChanged() {
            System.out.println("I'm doing something because the data changed");
        }

        public synchronized void updateDisplay() {
            System.out.println("Updating display ...");
            Object o = data.getData();
        }
    }
}
```

Here is how a deadlock could occur:

1. `Thread1` enters `data.updateData()` and writes to the console, then suspends.
2. `Thread2` enters `display.updateDisplay()` and writes to the console, then suspends.
3. `Thread1` runs and tries to call `display.dataChanged()`, but `Thread2` is still running `display.updateDisplay()`, so it is holding the lock on the Display object. `Thread1` blocks.
4. `Thread2` wakes up and tries to run `data.getData()`, but `Thread1` is still running `data.updateData()`, so `Thread2` blocks

  

We now have a deadlock. The underlying reason for the deadlock is the same as the previous one, the 2 threads are trying to get the same two locks in a different order. We avoid this problem the same way we did for the previous example, by making sure the when attempting to acquire locks from multiple threads, to be in the same order.

  

To address this problem in the above code, we will have to rewrite the code so that the 2 threads try to obtain the locks in the same order, which in this case, would mean not calling each other in this circular fashion.

  

Moving on, here is another way we can get a deadlock, and is actually fairly hard to spot. Consider the following code:

```Java
public class Main {
    public static void main(String[] args) {
        PolitePerson jane = new PolitePerson("Jane");
        PolitePerson john = new PolitePerson("John");

        jane.sayHello(john);
        john.sayHello(jane);
    }

    private static class PolitePerson {
        private final String name;

        public PolitePerson(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }

        public synchronized void sayHello(PolitePerson person) {
            System.out.format("%s: %s" + " has said hello to me!%n", this.name, person.getName());
            // with this, each %s is replaced with the parameters passed in
            person.sayHelloBack(this);
        }

        public synchronized void sayHelloBack(PolitePerson person) {
            System.out.format("%s: %s" + " has said hello back to me!%n", this.name, person.getName());
        }
    }
}
```

Now, this results in the output:

```Plain
Jane: John has said hello to me!
John: Jane has said hello back to me!
John: Jane has said hello to me!
Jane: John has said hello back to me!
```

Now, consider the same code, but if we had John and Jane say hi to each other on two separate threads:

```Java
public static void main(String[] args) {
        PolitePerson jane = new PolitePerson("Jane");
        PolitePerson john = new PolitePerson("John");

        new Thread(new Runnable() {
            @Override
            public void run() {
                jane.sayHello(john);
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                john.sayHello(jane);
            }
        }).start();
    }
```

In this case, we arrive at a deadlock, with the only output being produced being:

```Java
Jane: John has said hello to me!
John: Jane has said hello to me!
```

So, why does this occur? Here is a scenario that could result in a deadlock.

1. `Thread1` acquires the lock of the `jane` object, enters the `sayHello()` method, and then suspends
2. `Thread2` acquires the lock of the `john` object, enters the `sayHello()` method, and then suspends
3. `Thread1` runs again, and wants to say Hello back, so it tries to call `sayHelloBack()` method on `john` object, but `Thread2` is holding the lock, so `Thread1` is blocked.
4. The same as above, but for `Thread2`, it also becomes blocked.

  

**⇒ Thread Starvation**

So, what is `thread starvation`? Starvation describes the situation where a thread is unable to gain regular access to shared resources and is unable to make progress. This occurs when shared resources are made unavailable for long periods by “greedy” threads. Now, when starvation occurs, it’s not that the thread will never progress because they’ll never get a lock, but that they rarely have the opportunity to run and progress. So, starvation occurs due to threat priority. When we assign a high priority to a thread, we’re suggesting to the operator that it should try and run the thread before other waiting threads. Remember when we worked with `syhchronized` code. The OS doesn’t necessarily choose the thread that’s been waiting the longest to run. A thread can block on the lock first but that doesn't mean it will be the first thread to run when the lock becomes available. So, it's possible that the thread won't be able to run for a long time because other threads keep blocking on the lock, and when the lock becomes available the operating system chooses one of those other threads to run, especially when one of the other threads has a higher priority than the first thread that blocked.

  

Let’s create an example to learn more about this. We’ll make 5 threads and assign them different priorities. They will print the numbers from 1 to 100 to the console. Consider the following code:

```Java
public class Main {
	private static Object lock = new Object();
	
	public static void main(String[] args) {
	    Thread t1 = new Thread(new Worker(ThreadColor.ANSI_RED), "Priority 10");
	    Thread t2 = new Thread(new Worker(ThreadColor.ANSI_BLUE), "Priority 8");
	    Thread t3 = new Thread(new Worker(ThreadColor.ANSI_GREEN), "Priority 6");
	    Thread t4 = new Thread(new Worker(ThreadColor.ANSI_CYAN), "Priority 4");
	    Thread t5 = new Thread(new Worker(ThreadColor.ANSI_PURPLE), "Priority 2");
	
	    t1.setPriority(10);
	    t2.setPriority(8);
	    t3.setPriority(6);
	    t4.setPriority(4);
	    t5.setPriority(2);
	
	    t3.start();
	    t2.start();
	    t5.start();
	    t4.start();
	    t1.start();
	
	}
	
	private static class Worker implements Runnable {
	    private int runCount = 1;
	    private String threadColor;
	
	    public Worker(String threadColor) {
	        this.threadColor = threadColor;
	    }
	
	    @Override
	    public void run() {
	        for(int i=0; i<100; i++) {
	            synchronized (lock) {
	                System.out.format(threadColor + "%s: runCount = %d\n", Thread.currentThread().getName(), runCount++);
	                // execute critial section of code
	            }
	        }
	    }
	}
}
```

Above, we have created 5 threads, each with a different priority. When setting priorities for the threads, keep in mind that we are only **recommending** the priorities to the JVM, we can neither force the thread to abide by these priorities, that is, the threads sometimes will not follow the priorities set for them. Now, thats not to to say it never follows the priority, as it often does. It’s just something that we cannot guarantee.

  

The main takeaway from the example above is that although the lock is frequently released, often, the highest priority thread will probably hog all of the time until its finished counting, and the next thread that runs won’t necessarily be the next highest priority thread. Now, this is a very trivial example due to the small size of the example. However, its a different story if we were working on something larger project. In a real world application, the task that a thread is performing might be a long running task, or, the `run()` method was performing database queries or file reads. If the thread is blocked there for seconds, or even minutes, because other threads are always chosen to run when the lock becomes available, then the application’s performance will definitely be adversely affected.

  

Funny enough, setting priorities for threads can make starvation more likely to happen. For the code above, if we do not set the priorities, each thread will print from 1 to 100 continuously, in its entirety. So, this begs the question, how can we prevent **thread starvation?** So, when we're dealing with deadlocks, the order in which locks are required was important, but for starvation when which thread gets to run when a lock becomes available is important. So, ideally we'd like things to be first come first served. Although we cannot fully dictate whcih threads should run first, we can do our best. To accomplish this, we will need to use something called a `FairLock`.
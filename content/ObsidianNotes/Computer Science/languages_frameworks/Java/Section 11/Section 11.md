# Concurrency - Continued

**⇒ Fair Locks and Live Locks**

`FairLock`s try to be first-come first-serve. The `ReentrantLock` class allows us to create fair locks, but not all `Lock` implementations do that. Below is the counting example, reformated to use fair locks:

```Java
import java.util.concurrent.locks.ReentrantLock;

public class Main {
	private static ReentrantLock lock = new ReentrantLock(true);
	// true means whether it is a fair lock or not
	
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
	            lock.lock();
	            try {
	                System.out.format(threadColor + "%s: runCount = %d\n", Thread.currentThread().getName(), runCount++);
	                // execute critial section of code
	            } finally {
	                lock.unlock();
	            }
	        }
	    }
	}
}
```

Now, a few important notes about using a fairy entrant lock; only fairness in acquiring the lock is guaranteed not fairness in threads scheduling. So, it's possible that the thread that gets the lock will execute a task that takes a long time. So, when using fairlocks it's possible for threads to still have to wait a long time to run. The only thing a fair lot guarantees is the first come first served ordering for getting the lock.

  

While the fair lock is able to try and provide the lock to the longest waiting thread, this ultimately slows things down. As you can imagine, in order for a lock to be fair (when we pass true to the `ReentrantLock`), there are some behind the scenes processing that is going on. Now, from the above code, the output will be like:

![[Screenshot_2023-01-09_at_6.34.53_PM.png]]

Now, this is a much more fair outcome. Every thread has the chance to run, and the order is very stable. We now no longer have a starvation problem. No thread is just waiting for another thread to count to 100 and doing nothing.

  

Now, working with threads always involved working with tradeoffs, and addressing starvation is no different. Why would we want to use `synchronzied` blocks, as opposed to the `ReentrantLock`? Such a decision will depend on the application being built. If the application is only using a small number of threads (say 2 or 3), or each task is very quick to complete itself, then it will be rare for a thread to be starved. Since using fair locks has performance implications, thread starvation may be the lesser of the 2 evils.

  

**⇒ LiveLock**

So, a live lock is similar to a deadlock but instead of the threads being blocked they're actually constantly active, and usually waiting for all the other threads to complete their tasks. Now, since all the threads are waiting for others to complete, none of them can actually progress. So, let's say that thread “a” will loop until thread “b” completes its task and thread “b” will loop until thread “a” completes its task. Thread “a” and thread “b” can get into a state in which they're both looping and waiting for the other to complete. That's actually what's called a Live Lock. The threads will never progress but they're not actually blocked.

  

Let’s look at a small example to make this more clear:

```Java
// worker.java
public class Worker {

    private String name;
    private boolean active;

    public Worker(String name, boolean active) {
        this.name = name;
        this.active = active;
    }

    public String getName() {
        return name;
    }

    public boolean isActive() {
        return active;
    }

    public synchronized void work(SharedResource sharedResource, Worker otherWorker) {

        while(active) {
            if(sharedResource.getOwner() != this) {
                try {
                    wait(10);
                } catch(InterruptedException e) {

                }
                continue;
            }

            if(otherWorker.isActive()) {
                System.out.println(getName() + " : give the resource to the worker " + otherWorker.getName());
                sharedResource.setOwner(otherWorker);
                continue;
            }

            System.out.println(getName() + ": working on the common resource");
            active = false;
            sharedResource.setOwner(otherWorker);
        }
    }
}

// SharedResource.java
public class SharedResource {

    private Worker owner;

    public SharedResource(Worker owner) {
        this.owner = owner;
    }

    public Worker getOwner() {
        return owner;
    }

    public synchronized void setOwner(Worker owner) {
        this.owner = owner;
    }
}

// Main.java
public class Main {

  public static void main(String[] args) {
      final Worker worker1 = new Worker("Worker 1", true);
      final Worker worker2 = new Worker("Worker 2", true);

      final SharedResource sharedResource = new SharedResource(worker1);

      new Thread(new Runnable() {
          @Override
          public void run() {
              worker1.work(sharedResource, worker2);
          }
      }).start();

      new Thread(new Runnable() {
          @Override
          public void run() {
              worker2.work(sharedResource, worker1);
          }
      }).start();

    }
}
```

This is an example where `LiveLock` occurs. How? Entering the `work()` method, if the shared resource is not owned by the current thread, the loop sleeps waits for 10 milliseconds, and `conintue`s. If it does have the shared resource, and the other thread is active (both are active to begin), then it gives the resource to the other thread, and continues to loop. The above code has the following output:

![[Screenshot_2023-01-09_at_11.04.02_PM.png]]

As seen, this is what a livelock is. The threads don’t die, but they continually execute, with no way for the threads to stop. Now, the solution for such a problem is heavily dependent on the application this error occurs in. There isn’t actually a solution where a one size fits all answer works.

  

**⇒ Slipped Condition**

This is a specific type of race condition, and is also known as **thread interference**. This can occur when a thread is suspending **after reading a condition** and **before performing the activities related to it**. This is a rare condition, but one must look for it if the outcome is not as expected. Consider the following:

> **Example**  
> Suppose there are two   
> **thread A and thread B** which want to process a **string S**. Firstly, **thread A** is started, it checks if there are any more characters left to process, initially the whole string is available for processing, so the condition is true. Now, **thread A** is suspended and **thread B** starts. It again checks the condition, which evaluates to true and then processes the whole string S. Now, when **thread A** again starts execution, the **string S** is completely processed by this time and hence an error occurs. This is known as a slipped condition.

  

The solution to the problem of Slipped Conditions is fairly **simple and straightforward**. Any resources that a thread is going to access after checking the condition, must be locked by the thread and should only be released after the work is performed by the thread. **All the access must be synchronized**.
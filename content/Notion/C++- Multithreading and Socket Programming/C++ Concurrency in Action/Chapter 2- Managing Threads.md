We use the `\#inlclude <thread>` library. To create a thread, we do:

```C++
\#include <iostream>
\#include <thread>

void hello() { std::cout << "Hello World" << std::endl; }

int main() {
  std::thread t(hello);
  t.join(); // joining the thread is important --> this causes the calling thread to wait for the thread associted with t
  return 0;
}
```

We need to pass some function for a thread to run. Here’s another example:

```C++
class background_task
{
public:
    void operator()() const // this special function allows class to be called as if it were a function
    {
        do_something();
        do_something_else();
    }
};
background_task f;
std::thread my_thread(f);
```

Be careful with how the compiler parses your code. Say we want to launch a thread with temporary variable, so something like:

```C++
std::thread my_thread(background_task());
```

As it stands, compiler will interpret this as a function declaration. You can avoid this by nam- ing your function object as shown previously, by using an extra set of parentheses, or by using the new uniform initialization syntax, for example:

```C++
std::thread my_thread((background_task()));
std::thread my_thread{background_task()}; // newer syntax
```

One type of callable object that avoids this are Lambda expressions, which are a new addition to C++11:

```C++
std::thread my_thread([](
    do_something();
    do_something_else();
});
```

Once you’ve started your thread, you need to explicitly decide whether to wait for it to finish or leave it to run on its own. If you don’t decide before the `std::thread` object is destroyed, then your program is terminated `std::thread destructor` calls `std::terminate()`. We need to be careful with the lifetimes of threads. If you don’t wait for your thread to finish, then you need to ensure that the data accessed by the thread is valid until the thread has finished with it. Consider:

```C++
struct func {
    int& i;

    // Constructor to initialize the reference
    func(int& i_) : i(i_) {}

    // Overload the function call operator
    void operator()() {
        for (unsigned j = 0; j < 1000000; ++j) {
            do_something(i);
        }
    }
};

// Dummy function to illustrate the example
void do_something(int& i) {
    i++;
}

// Function with potential issues
void oops() {
    int some_local_state = 0;
    func my_func(some_local_state);

    // Create a new thread that runs the my_func callable object
    std::thread my_thread(my_func);

    // Detach the thread
    my_thread.detach();

    // Potential access to dangling reference:
    // The thread is detached, so it might still be running after oops() returns.
    // This means the reference `i` in `func` might refer to a destroyed local variable.
}
```

When `some_local_state` is defined within the `oops` function and a reference to it is stored in the `func` object, the reference becomes invalid once `oops` exits. This is because `some_local_state` goes out of scope and is destroyed, leading to a dangling reference in the detached thread.

  

**→ Waiting for a thread to complete**

If you need to wait for a thread to complete, you can do this by calling `join()` on the associated `std::thread` instance. `join()` is simple and brute force—either you wait for a thread to finish or you don’t. If you need more fine-grained control over waiting for a thread, such as to check whether a thread is finished, or to wait only a certain period of time, then you have to use alternative mechanisms such as `condition variables` (seen in CS350) and futures, which we’ll look at in chapter 4.

  

**→ Waiting in exceptional circumstances**

You need to ensure that you’ve called either `join()` or `detach()` before a `std::thread` object is destroyed. Where we do this is very important. For example, what happens if we throw an error right after the thread has been started? In general, if you were intending on calling `.join()` in the non-exception case, you also need to call `join()` in the presence of an exception to avoid accidental lifetime problems. Something like:

```C++
struct func; // defined above 

void f() {
	int some_local_state = 0;
	func my_func(some_local_state);
	std::thread t(my_func);
	try {
		do_something_in_current_thread();
	} catch (...) {
		t.join();
	}
	t.join();
}
```

But, this use of `try` is verbose, and easy to get wrong, so its not ideal. If it’s important to ensure that the thread must complete before the function exits—whether because it has a reference to other local variables or for any other reason—then it’s important to ensure this is the case for all possible exit paths, whether normal or exceptional, and it’s desirable to provide a simple, concise mechanism for doing so. One way is to use **RAII** idiom, and provide a class that does the `join()` in its destructor, as in the following example:

```C++
class thread_guard {
    std::thread& t;
public:
    explicit thread_guard(std::thread& t_) : t(t_) {} // prevents implicit conversions, thread object needs to be passed    
    ~thread_guard() {
        if (t.joinable()) {
            t.join();
        }
    }

    thread_guard(thread_guard const&) = delete;
    thread_guard& operator=(thread_guard const&) = delete;
};

struct func;

void f() {
    int some_local_state = 0;
    func my_func(some_local_state);
    std::thread t(my_func);
    thread_guard g(t);
    do_something_in_current_thread();
}x

// See definition in listing 2.1
```

When `g` goes out of scope, destructor is called, which triggers the `.join()` within the destructor. This happens even if `do_something_in_current_thread()` throws an exception. The destructor of thread_guard in listing 2.3 first tests to see if the `std::thread` object is `joinable()` before calling `join()` . This is important, because join() can be called only once for a given thread of execution, so it would therefore be a mistake to do so if the thread had already been joined.

  

The copy constructor and copy-assignment operator are marked `=delete` to ensure that they’re not automatically provided by the compiler. Copying or assigning such an object would be dangerous, because it might then outlive the scope of the thread it was joining.

  

**→ Running Threads in the background**

Calling detach() on a std::thread object leaves the thread to run in the back- ground, with no direct means of communicating with it. Ownership and control of the thread are passed to the C++ Runtime Library.
## Section 4: Arrays, Java inbuilt Lists, Autoboxing and unboxing

**⇒ Arrays**

```Java
int[] myIntArray = new int[10];
```

The above is the way we declare and use arrays in Java. You will notice that it is different from the methods used in C++. However, it’s use cases are exactly the same:

```Java
package dev.lpa;

import java.util.Scanner;

public class Main {
    private static Scanner scanner = new Scanner(System.in);
    public static void main(String[] args) {
        int[] myIntegers = getIntegers(5);
        sort(myIntegers,0,myIntegers.length-1);
        printArray(myIntegers);
    }

    public static int[] getIntegers(int count) {
        int[] intArray = new int[count];
        System.out.println("Please enter " + count + " integers.");
        for (int i = 0; i < intArray.length; i++) {
            System.out.println("Number " + i + ": ");
            int n = scanner.nextInt();
            intArray[i] = n;
        }

        return intArray;
    }

    public static void printArray(int[] array) {
        System.out.print("[");
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i] + " ");
        }
        System.out.print("]");
        System.out.println();
    }

    public static void sort(int[] array, int begin, int end) {
        int middle = (begin + end) / 2;
        if (begin < end) {
            sort(array, begin, middle);
            sort(array, middle + 1, end);
            merge(array, begin, middle, end);
        }
    }

    private static void merge(int[] array, int begin, int middle, int end) {
        int leftSize = middle - begin + 1;
        int rightSize = end - middle;

        int[] left = new int[leftSize];
        int[] right = new int[rightSize];

        // now load the respective integers into these  arrays
        for (int i = 0; i < leftSize; i++) {
            left[i] = array[i];
        }

        for (int i = 0; i < rightSize; i++) {
            right[i] = array[middle+1+i];
        }

        // now, we merge the numbers back into array
        int left_index = 0;
        int right_index = 0;
        int finish_index = begin;

        while (left_index < leftSize && right_index < rightSize) {
            if (left[left_index] <= right[right_index]) {
                array[finish_index] = left[left_index];
                left_index++;
            } else {
                array[finish_index] = right[right_index];
                right_index++;
            }
            finish_index++;
        }

        // need to put in the remaining (if there are) sub array elements into the main array
        while (left_index < leftSize) {
            array[finish_index] = left[left_index];
            left_index++;
            finish_index++;
        }

        while (right_index < rightSize ) {
            array[finish_index] = right[right_index];
            right_index++;
            finish_index++;
        }
    }
}
```

The above is a sample code snippet of merge sort written in Java, good practice for working with arrays.

**⇒ References vs Value Types**

Now, we have looked at similar things before. First off, we recognize that there are value type variables and reference type variables. What’s the difference? Similar to C++, consider the following:

```Java
int myValue = 10;
int anotherValue = myValue;

int[] myArray = new int[5];
int[] anotherArray = myArray; // another reference to the same array in memory
```

So, the variable `myValue` and `anotherValue` actually hold the number they have. On the other hand, `myArray` does not actually have the array, rather, it’s a reference, and holds the address of the array in memory. A quick nice and fast tool to print out the contents of an array is:

```Java
int[] myArray = {1,2,3};
System.out.println("The array contents are: " + Arrays.toString(myArray));
```

![[Screenshot_2022-12-12_at_1.11.24_PM.png]]

As seen from above, both variables are references to the same array, and hold the same address in memory.

**⇒ List and ArrayList**

The ArrayList is basically the same thing as vector in C++. A self-resizing array. Here are some useful ways/methods we use on ArrayList ( see in the .java file)

```Java
ArrayList<String> groceryList = new ArrayList<String>();
// here are some methods 
groceryList.size();
groceryList.get(i); // is is index position 
groceryList.set(position, newItem); // sets the value of a certain index 
groceryList.remove(position); // removes the value at that position 
groceryList.indexOf(searchItem); // returns index of the thing being searched
```

**⇒ Autoboxing and Unboxing**

Recall how we cannot pass an int to the ArrayList. The thing is, we need a class to be inside of the <> for the ArrayList. So, passing a primitive type will result in the program not compiling. To solve this, Java provides us with autoboxing. Essentially, these primitive types are wrapped with a class to make Java support this functionality:

```Java
ArrayList<Integer> intArrayList = new ArrayList<Integer>();
ArrayList<Double> doubleArrayList = new ArrayList<Double>();
```

So, the, strictly technically speaking, here is how we would use them:

```Java
ArrayList<Integer> intArrayList = new ArrayList<Integer>();
// to load an arraylist
for (int i = 0; i < 10; i++) {
	intArrayList.add(Integer.valueOf(i));
}

// to print the arraylist outside 
for (int i = 0; i < 10; i++) {
	System.out.println(i + " --> " + intArrayList.get(i).intValue();
}
```

Now, obviously, this is far from being ideal, as it can be very bothersome, and unintuitive to constantly need to do this, hence, Java has allowed shortcuts, which make the following possible:

```Java
Integer myIntValue = 56; // Java is doing Integer.valueOf(56);
int myInt = myIntValue.intValue() // myIntValue.intValue()
```

**Linked-List**

When working with ArrayLists, a lot of work is being done by Java below the hood. Every time we remove items from the ArrayList, Java is constantly resizing the array, and shifting items up and down of the ArrayList, hence why it can become very expensive when we are doing this hundreds of thousands of times. An alternative data structure we can use is the linked-list.

Each node of the linked-list contains data, as well as points to the next element. This makes removing much more efficient. We can also have double linked-list if we need. Now, with LinkedList, we can remove, and add elements at a certain index much easier, as now, Java needs only to adjust the node that a node is pointing at to be able to effectively remove, or add, a new item in the list:

```Java
import java.util.Iterator;
import java.util.LinkedList;

public class Demo {
    public static void main(String[] args) {
        LinkedList<String> placesToVisit = new LinkedList<String>();
        placesToVisit.add("Toronto");
        placesToVisit.add("Vancouver");
        placesToVisit.add("Sydney");
        placesToVisit.add("Melbourne");

        printList(placesToVisit);

        placesToVisit.add(1, "Alice Springs");
        printList(placesToVisit);

        placesToVisit.remove(4);
        printList(placesToVisit);
    }

    private static void printList(LinkedList<String> linkedlist) {
        Iterator<String> i = linkedlist.iterator();
        while(i.hasNext()) {
            System.out.println("Now visiting " + i.next());
        }
        System.out.println("====================================");
    }
}
```

This code outputs:

![[Screenshot_2022-12-12_at_9.31.44_PM.png]]

**⇒ Iterator & ListIterator**

The concept of an iterator should be fairly clear, it is an nested Object that allows us to move through a data structure, especially something like an LinkedList. The normal Iterator<> is a one way LinkedList, so each node only points to the next one. There is also a ListIterator<>, which is a double linked list, so each node also has a pointer to the node before it, in addition to the node after it.

**Note:** Something important is that upon creating the linked list, so `ListIterator<String> stringListIterator = linkedlist.ListIterator();`, we need to first navigate to the next() element in order to be on the first node.
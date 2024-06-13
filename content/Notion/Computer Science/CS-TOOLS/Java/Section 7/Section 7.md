## Java Collections

So, Java collections essentially is now when we are going to begin looking at a lot of the data structures and how to use them. Consider this quick hierarchy diagram:

![[Screenshot_2022-12-15_at_2.19.09_PM.png]]

As seen, a map is not a true collection. So, these are all part of the collection. So, when we create a variable of type collection, we can assign to it any of the direct children, and anything should work. Consider the following code:

```Java
 private List<Seat> seats = new ArrayList<>();
```

We can create a List, and then assign to it an ArrayList, a property of polymorphism. This also works:

```Java
private Collection<String> words = new ArrayList<String>();
```

Again, this works. In fact, from the inheritance hierarchy, we can also assign a set, queue, or other similar ones to the Collection:

```Java
private Collection<String> words = new LinkedHashSet<>();
private Collection<String> ll = new LinkedList<>();
```

Now, these are all Collections, but the use cases may vary. This is because these data structures have their own strengths and weaknesses, and we’ll prefer over one over the other in certain circumstances.

**⇒ Binary Search**

The collection class supports a binary search Method, which is a recursive algorithm to find a node in a sorted collection. Say we have a collection of a user defined type. In that case, that class must implement the Comparable interface, which just requires it to implement and override the `compareTo` method (which you may recall seeing in Strings). This returns an int, and recall the sign of the int based on the comparison result.

⇒ This makes sense. Recall what the binary sort looks like. It begins with a binary search tree. Consider one for integers for now. Of course, finding a node will involve comparing integers. The same for our own defined types. Now, we apply the same logic to a collection. Such as an array. [https://www.geeksforgeeks.org/binary-search/](https://www.geeksforgeeks.org/binary-search/)

  

Then, to be able to use the binary search method, we access it directly from the Collection class:

```Java

private List<Seat> seats = new ArrayList<>();

public boolean reserveSeat(String seatNumber) {
	Seat requestSeat = new Seat(seatNumber);
	int foundSeat = Collections.binarySearch(seats, requestSeat, null);
	// seats is list of seats, passing null since using inbuilt comparison
```

Quick note, we would need seats, to be a List, since some methods used `.get()`, which is not a method on Collection, but is on List. When we’re looking for something within a list, it’s best to used this binarySearch method, as it is much much MUCH faster in terms of time complexity than brute forcing a search with nested loops. So, how does it work?

**⇒ Binary Search - Explanation**

Consider that we have a sorted collection. To look for an item in a collection with this algorithm, we do the following:

1. Begin with the middle element of the collection
2. If it’s equal to what we’re looking for, done. Elese, either look in the right sub array or the left sub array, depending on if the item we’re looking for is greater or less than the middle item
3. Recuse, do same steps on sub array until found

Now, we can do this both iteratively or recursively. Both follow the same logic. Here is the recursive approach:

```Java
binarySearch(arr, x, low, high)
 if low > high
     return False 

 else
     mid = (low + high) / 2 
         if x == arr[mid]
         return mid

     else if x > arr[mid]        // x is on the right side
         return binarySearch(arr, x, mid + 1, high)
     
     else                        // x is on the left side
         return binarySearch(arr, x, low, mid - 1)
```

**⇒ Collections List Methods**

- Collections.sort(List list) → This sorts the list, from least to greatest. The java library actually implements this sort method with merge sort, as it is the most efficient
- Collections.reverse(List list) → This reverses the list
- Collections.shuffle(List list) → This orders the list into a random order
- Collections.min(List list)
- Collections.max(List list)
- Collections.swap(list, i, j) → swaps the value of the list at the two given indexes

  

**⇒ Comparable & Comparator**

These are interfaces. Now, previously, we saw how we can make a class implement the comparable interface, which we would then override the compareTo method.

  

## Maps

So, the map is part of the collection framework, even though its not really a “true” part of the collection class. Recall the hierarchy char from before, it’s not part of the Collection tree, but it is basically a special type of collection. A map maps a key to a value, and the key must be unique.

**⇒ Hashmap**

```Java
Map<String, String> languages = new HashMap<>();
```

Now, map keys are unique, so we can only have one key associated to one value. If we try to associate a previously existing key with a new value, the olde one is overwritten. The `.get()`

method actually returns the value if it is being overwritten. If a value is being added for the fist time, it will returns null.

- languages.containsKey(key) → this checks if the key already exists inside of the map
- languages.keySet() returns a Set view of the keys contained in the map
- languages.values() returns a collection-view of the values contained in this map

Now, looping through a map is a bit different than what we’re used to. Since it is not part of the collection hierarchy, there is no iterator for it. So, we can use the .keySet() method to loop through, since that returns a set of the keys, which does have an iterator for it:

```Java
for (String it: languages.keySet()) {
	System.out.println(it + " corresponds to the value" + languages.get(it));
}
```

Note, to add items to a map, we use `.put()` as there is no `.add()` method for Maps.

```Java
Map<String, String> languages = new HashMap<>();
	languages.put("Java", "language");
	languages.put("Python", "high-level language");
	languages.put("Algol", "algorithmic language");
	languages.put("racket", "functional language");
	
	for (String it: languages.keySet()) {
	    System.out.println(it + " corresponds to " + languages.get(it));
}
```

To remove something, we can just use `languages.remove(”Java”);`

**⇒ Immutable Classes**

We can have immutable classes, in which all fields, even things like collection or map fields, are final. To do so, every field of the class must be initialized in the constructor. Here are some rules Oracle has laid out:

![[Screenshot_2022-12-16_at_4.02.10_PM.png]]

To summarize, keep things private and final to prevent outside factors fro modifying. Do not provide any setter methods, as we want these fields to be immutable. Lastly, if our class contains a reference to some other object, or collection, etc. Provide a copy when to the class when creating the object in the constructor, and for getters, provide a copy. Here is what an immutable class could look like:

```Java
public final class Location {
    private final int locationID;
    private final String description;
    private final Map<String, Integer> exits;

    public Location(int locationID, String description, Map<String, Integer> exits) {
        this.locationID = locationID;
        this.description = description;
        if (exits != null) {
            this.exits = new HashMap<String, Integer>(exits);
        } else {
            this.exits = new HashMap<String, Integer>();
        }
        
        this.exits.put("Q", 0);
    }
    
    public int getLocationID() {
        return locationID;
    }

    public String getDescription() {
        return description;
    }

    public Map<String, Integer> getExits() {
        return new HashMap<String, Integer> (exits);
    }
    
}
```

**⇒ Sets and HashSets**

Now these are another data structure in the collections hierarchy. Sets are unique, and can have no duplicates. Here are some methods for sets:

```Java
Set<String> names = new HashSet<>();
names.add("Allan");
names.add("Pierre");
names.remove("Allan");
names.size(); // this is the length of the set 
names.isEmpty(); 
```

We cannot get the value of a specific index of a set. So, there is no way to get, for instance, the 10th item , unless you iterate through the set. Even the then, if we want to be able to access a specific item, we’re much better of using an ArrayList.

**⇒ equals() and hashcode()**

Now, we take a look at these 2 methods. These are especially important if we use custom type as a key for a map, or inserting one into a set. Maps need keys to be unique, and there can be no duplicate keys. Likewise, there can be no duplicates inside of a set. So, we need to provide Java a way to compare our types, and maintain uniqueness. While this may sound simple upon first glance, we will also need to compare the hash codes of the two objects, which means we also need to override the `hashcode()` method whenever we override `equals()`.

![[IMG_0012.heic]]

So, here is a quick small diagram that will help to clarify the point of the `hashcode()` method. In addition to the equals() method, the hash code method is used to verify whether two objects are equal. When two objects are equal, their hash code is the same, and they are placed into the same “bucket”. Two equal objects will have the same hashcode. This is from Oracle:

> "If two objects are equal according to the `equals(Object)`  
>  method, then calling the   
> `hashCode`  
>  method on each of the two objects must produce the same integer result. It is   
> _not_  
>  required that if two objects are unequal according to the   
> `equals(java.lang.Object)`  
>  method, then calling the   
> `hashCode`  
>  method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables."  

To help provide another example of why these methods need to be overriden:

```Java
class Complex {
    private double re, im;   
     
    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
}
 
// Driver class to test the Complex class
public class Main {
    public static void main(String[] args) {
        Complex c1 = new Complex(10, 15);
        Complex c2 = new Complex(10, 15);
        if (c1 == c2) {
            System.out.println("Equal ");
        } else {
            System.out.println("Not Equal ");
        }
    }
}
```

In the code below, we do not the have methods overridden in the class Complex. So, when we create two instances of Complex, that are the same, and “equal”, Java tells us they are not. This is because all variables in Java, by default, are references. So, when Java evaluated c1 == c1, it is actually comparing the value of their address in memory, which of course will be different. This is shows another reason we often will need to override the `equals()` and `hashcode()` methods. Now, consider the following:

```Java
import java.util.HashSet;
import java.util.Set;

public final class HeavenlyBody {
    private final String name;
    private final double orbitalPeriod;
    private final Set<HeavenlyBody> satellites;

    public HeavenlyBody(String name, double orbitalPeriod) {
        this.name = name;
        this.orbitalPeriod = orbitalPeriod;
        this.satellites = new HashSet<>();
    }

    public String getName() {
        return name;
    }

    public double getOrbitalPeriod() {
        return orbitalPeriod;
    }

    public boolean addMoon(HeavenlyBody moon) {
        return this.satellites.add(moon);
    }

    public Set<HeavenlyBody> getSatellites() {
        return new HashSet<>(this.satellites);
    }


    @Override
    public boolean equals(Object obj) {
        if(this == obj) {
            return true;
        }

        System.out.println("obj.getClass() is " + obj.getClass());
        System.out.println("this.getClass() is " + this.getClass());
        if ((obj == null) || (obj.getClass() != this.getClass())) {
            return false;
        }

        String objName = ((HeavenlyBody) obj).getName();
        return this.name.equals(objName);
    }

    @Override
    public int hashCode() {
        System.out.println("hashcode called");
        return this.name.hashCode() + 57;
    }
}
```

So, above is the `hashcode()` and `equals()` methods overridden. Now, notice we only compared the name. This is only because here, we have decided if two HeavenlyBody objects have the same “name” field, they are equal. Of course, if our class needed more fields to be the same for them to be equal, we would need to evaluate those.

**⇒ Potential issues with equals() and sub-classing**

Now, consider if we have a class, which has subclasses. Now, with this, we need to give more thought into how we use the equals() method. Consider the following code:

```Java
package dog;

public class Dog {
    private final String name;

    public Dog(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public boolean equals(Object obj) {
        if(this == obj) {
            return true;
        }

        if(this.getClass() == obj.getClass()) {
            String objName = ((Dog) obj).getName();
            return this.name.equals(objName);
        }

        return false;
    }
}
```

```Java
package dog;

public class Labradour extends Dog {

    public Labradour(String name) {
        super(name);
    }

    @Override
    public boolean equals(Object obj) {
        if(this == obj) {
            return true;
        }

        if(this.getClass() == obj.getClass()) {
            String objName = ((Labradour) obj).getName();
            return this.getName().equals(objName);
        }

        return false;
    }
}
```

As seen, a Labrador inherits from he Dog class. What if we want to have Dog object and a Labrador object, with the same name, to be eual. If we use the implementation before, we would run into some issues, as they are different classes. One way to solve this, is to not provide an equals() method for the Labrador class. We can also make equals() method final in the Dog class.

```Java
package dog;

public class Dog {
    private final String name;

    public Dog(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public final boolean equals(Object obj) {
        if(this == obj) {
            return true;
        }

        if(obj instanceof Dog) {
            String objName = ((Dog) obj).getName();
            return this.name.equals(objName);
        }

        return false;
    }
}
```

```Java
package dog;

public class Labradour extends Dog {

    public Labradour(String name) {
        super(name);
    }

//    @Override
//    public boolean equals(Object obj) {
//        if(this == obj) {
//            return true;
//        }
//
//        if(obj instanceof Labrador) {
//            String objName = ((Labrador) obj).getName();
//            return this.getName().equals(objName);
//        }
//
//        return false;
//    }
}
```

This way, the Labrador, when calling `equals()` and passing it a Dog object, will simply use the Dog’s `equals()` method.

**⇒ Sets - Symmetric & Asymmetric**

Here are some set Bulk operations that will likely prove useful:

```Java
s1.containsAll(s2); // returns true if s2 is a subset of s1 
s1.addAll(s2); 
// transforms s1 into a union on s1 and s2, s2 is destroyed in process
s1.retainAll(s2); // transforms s1 into the intersection of s1 and s2
s1.removeAll(s2); //transforms s1 into the (asymmetric) set difference of s1, s2
```

Now remember, s1 is a set, so even if we’re adding all of s2 into s1, they are only fully added if they maintain that all items in s1 are unique. These methods are destructive, so s2 is changed when using this method. If we want to avoid this issue, we will need to use a copy of s2. So, to better illustrate how the removeAll method, consider the code below:

```Java
Set<String> nature = new HashSet<>();
Set<String> divine = new HashSet<>();
String[] natureWords = {"all", "nature", "is", "but", "art", "unknown", "to", "thee"};
nature.addAll(Arrays.asList(natureWords));

String[] divineWords = {"to", "err", "is", "human", "to", "forgive", "divine"};
divine.addAll(Arrays.asList(divineWords));

System.out.println("nature - divine:");
Set<String> diff1 = new HashSet<>(nature);
diff1.removeAll(divine);
printSet(diff1);

System.out.println("divine - nature:");
Set<String> diff2 = new HashSet<>(divine);
diff2.removeAll(nature);
printSet(diff2);

/* this outputs 
nature - divine:
	all but art thee nature unknown 
divine - nature:
	err forgive divine human 
```

Essentially, what the removeAll method does is it (consider `s1.removeAll(s2)`) is that it produces the set that has all of the items in s1 but not in s2, and assigns tis back to the s1 reference. The retainAll() method simply returns the shared items between the two collections and assigns to the first collection:

```Java
public static void main(String[] args) {
	Set<String> words1 = new HashSet<>(Arrays.asList("Allan", "Brian", "Pierre", "Sophia"));
	Set<String> words2 = new HashSet<>(Arrays.asList("Brian"));
	
	words1.retainAll(words2);
	
	// print out everything inside of the set
	for (String it: words1) {
	    System.out.println(it);
	}

// this for loop outputs "Brian"
}
```

So, in the above, calling words1.retainAll(s2), returns a collection of strings that both words1 and words2 contain, and assigns to words1.

**⇒ Sorted Collections: Linked Hash-set and Linked Hash-map**

So, the Java docs describe the ordering of items in hash-set and hash map are described as _**chaotic**_. So, there is no real order, at least to our knowledge, of how the items we add are ordered. On the other hand, the linked versions, maintain insertion order, so based on what order we insert certain items into them, we’ll know the order, similar to something like an array.

```Java
// this is the getOrDefault method for this 
public Object getOrDefault(Object key, Object defaultValue)
```

This is a method on the maps class. It goes and attempts to retrieve the value associated with the key, and if found, returns it. Otherwise, it returns a defaultValue. This is a pretty useful tool to know as it can become very useful. What if we want to have access to a specific entry in a map, as an object? There is Map.Entry, which is a nest class, and holds one entry of the map as an object. Each item in the map is actually of type Map.Entry. With this, this is one way we could iterate through a map, since, they do not have an iterator.

```Java
for (Map.Entry<String, StockItem> item: list.entrySey()) {
	...
}
// this is how we can iterate through a map. Each map item is a Map.Entry object. Then, list.entrySet() returns a set of the Map.Entry items, which we can then iterate through 
```

Now, what if we want a sorted map? We can use TreeMap. This way, every time we put in a key-value pair in the map, it is automatically sorted for us, using our overridden compareTo method.
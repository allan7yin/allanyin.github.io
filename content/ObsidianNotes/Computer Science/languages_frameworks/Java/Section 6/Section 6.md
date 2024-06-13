## Java Generics

So, generics in Java are simply some of the data structures that do not have their type specified. Now, what does this mean? Consider the code below:

```Java
public satic main(String[] args) {
	ArrayList items = new ArrayList();
	items.add(1);
	items.add(1);
	items.add(1);
	items.add(1);
	items.add(1);
	items.add("allan");
	print(items);
}

public static print(ArrayList items) {
	for (var item: items) {
		System.out.println((Integer) i * 2);
	}
}
```

Now, this code sort of introduces what generics are in Java, and some of the potential problems that can arise should we choose to use it. First of all, as we have a generic ArrayList, (imagine just doing vector, without the <>), we can have an Object data type inside of it. So, the code above while compile perfectly fine. However, if we were to run it, we would get a run-time error, telling us we cannot cast a String into an Integer. Of course, if we add type specifications to the ArrayList, we will undoubtedly solve these issues, as the above code will not compile in the first place.

One way to look at the generic class, is to see it as similar to a Template in C++. So, we can create as follows:

```Java
import java.util.ArrayList;

public class Team<T extends Player> {
    private String name;
    int played = 0;
    int won = 0;
    int lost = 0;
    int tied = 0;


    private ArrayList<T> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public boolean addPlayer(T player) {
        if (members.contains(player)) {
            System.out.println(player.getName() + " is already on this team");
            return false;
        } else {
            members.add(player);
            System.out.println(player.getName() + " picked for team " + this.name);
            return true;
        }
    }

    public int numPlayers() {
        return this.members.size();
    }

    public void matchResult(Team<T> opponent, int ourScore, int theirScore) {

        String message;

        if(ourScore > theirScore) {
            won++;
            message = " beat ";
        } else if(ourScore == theirScore) {
            tied++;
            message = " drew with ";

        } else {
            lost++;
            message = " lost to ";
        }
        played++;
        if(opponent != null) {
            System.out.println(this.getName() + message + opponent.getName());
            opponent.matchResult(null, theirScore, ourScore);
        }
    }

    public int ranking() {
        return (won * 2) + tied;
    }

}
```

As seen above, we have a class called Team, which is a generic type class. You can imagine this to be how something like the ArrayList class would be implemented. Now, we include the `<T extends Player>` as this makes sure that for code to compile, there needs to be a player type used when creating a Team instance.

In a nutshell, generics **enable types (classes and interfaces) to be parameters when defining classes, interfaces and methods.**

Now, something we can also do, is to make this Team class implement an interface, which will require us to override the `compareTo` method (which, if you recall, is something available for strings.

Doing so, provides us with a Team class as follows:

```Java
import java.util.ArrayList;

public class Team<T extends Player> implements Comparable<Team<T>> {
    private String name;
    int played = 0;
    int won = 0;
    int lost = 0;
    int tied = 0;


    private ArrayList<T> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public boolean addPlayer(T player) {
        if (members.contains(player)) {
            System.out.println(player.getName() + " is already on this team");
            return false;
        } else {
            members.add(player);
            System.out.println(player.getName() + " picked for team " + this.name);
            return true;
        }
    }

    public int numPlayers() {
        return this.members.size();
    }

    public void matchResult(Team<T> opponent, int ourScore, int theirScore) {

        String message;

        if(ourScore > theirScore) {
            won++;
            message = " beat ";
        } else if(ourScore == theirScore) {
            tied++;
            message = " drew with ";

        } else {
            lost++;
            message = " lost to ";
        }
        played++;
        if(opponent != null) {
            System.out.println(this.getName() + message + opponent.getName());
            opponent.matchResult(null, theirScore, ourScore);
        }
    }

    public int ranking() {
        return (won * 2) + tied;
    }

    @Override
    public int compareTo(Team<T> team) {
        if(this.ranking() > team.ranking()) {
            return -1;
        } else if(this.ranking() < team.ranking()) {
            return 1;
        } else {
            return 0;
        }
    }
}
```

  

We can also make generic methods for classes:

![[Screenshot_2022-12-15_at_5.42.36_PM.png]]

![[Screenshot_2022-12-15_at_5.43.16_PM.png]]

  

So, when we are creating those classes, we will use T to represent that it is a generic class. Now, consider the following condition:

```Java
public static void printList(List<?> list) {
	for (int i = 0; i < list.size() - 1; i++) {
		if (list.get(i).compareTo())
	}
}
```

The ? is known is as the wildcard. While the T is saying multiple types of types are allowed to be passed when the class is made. This is different, but in the general idea of generics. Now, we can pass any type of list, and this code will work. In addition, if we wanted to make this more specific to the type we wanted, we could also specify as such:

```Java
public static void printList(List<? extends Theatre.Seat> list) {
	for (int i = 0; i < list.size() - 1; i++) {
		if (list.get(i).compareTo())
	}
}
```

## Naming Conventions

Now, we should look at some of the naming conventions in Java, to make my own code cleaner, and more integrable.

- **Packages:** Should always be lower case, usually com.something
- **Classes:** CamelCase
- **interface:** CamelCase
- **Methods:** mixedCase
- **Constants:** ALL CAPITAL: INT_MAX
- **Variables:** mixedCase
- **Type parameters:**
    - E - Element
    - K - Key
    - T - Type
    - V - Value
    - S, U, V, etc. - 2nd, 3rd, 4th types

  

⇒ **Packages**

There are a lot of existing packages, and class/interface conflicts are inevitable. A mechanism is needed to fully specify classes, and to allow the use of classes with the same name in the same project (or, even, the same class). This is doable, at a cost. For example, say we wanted to use two different types of Node classes, and they are from two different packages. Then, we could import one, and use the other with it’s entire package name, or use both with the entire package name. Importing both classes of the same name will result in error.

**⇒ Static and Final**

Static just means it is a field specific to the class itself, rather than an object or an instance of the class. All objects created will have their “static field” be the same data in memory, they all hold the same thing. So, when that static field changes, it changes for the class, and therefore, all objects. Java final is equivalent to C++ const on primitive value types. So, can only be assigned when the variable is initialized.
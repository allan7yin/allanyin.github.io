## Inner and Abstract Classes & Interfaces

**⇒ Interface**

An interface specifies the methods needed to be implemented by a class. So, an interface effectively acts as sort of a .h file, (which is literally the interface for C++, called the interface file). So, consider the following:

```Java
public interface ITelephone {
    public void powerOn();
    public void dial(int phoneNumber);
    public void answer();
    public boolean callphone(int phoneNumber);
    public boolean isRinging();

}
```

Now, how would we connect a class to this interface? In the same place, we would need to have a file, with the class to implement this interface. Something like:

```Java
public class DeskPhone implements ITelephone {
	// here, implement the methods inside of the interface 
}
```

Now, interfaces are a fairly big part of Java, for a couple of reasons. First, it is worth noting that java does not support multiple inheritance, unlike something like C++. However, Java allows us to implement multiple interfaces in one class. In addition, interfaces allow for more flexibility, such as the following:

```Java
ITelephone timsPhone;
timsPhone = new DeskPhone(123456);
timsPhone.powerOn();
timsPhone.callPhone(123456);
timsPhone.answer();

timsPhone = new MobilePhone(24565);
```

That is, we can declare a very broad/abstract type based on the interface, and then assign to it a more specific type.

**⇒ Inner Classes**

Java allows us to create nested classes. So the way approach something like this is exactly like how we would expect. Simply declare a class inside of another class. The syntax to access and work with the nested class is slightly new:

```Java
Gearbox mcLaren = new Gearbox(6);
Gearbox.Gear first = mcLaren.new Gear(1,12.3);
```

The above is how. Of course, the above is given that Gear is a public class. What would it look like if it was a private class. If that was the case, the above line (creating the Gear object), would need to be handled internally of the class, with a method provided for the client to be able to create a Gear without calling the constructor, a wrapper method.

In addition to being nested inside of other classes, we can also have classes defined inside of methods, or if-else statements. Java also has something called anonymous inner classes. Consider the following code:

```Java
// Button.java 
public class Button {
    private String title;
    private onclicklistener onClickLister;

    public Button(String title) {
        this.title = title;
    }

    public String getTitle() {
        return title;
    }

    public void setOnClickListener(OnClickListener onClickListener) {
        this.onClickLister = onClickListener;
    }

    public void onClick() {
        this.onClickLister.onClick(this.title);
    }

    public interface OnClickListener {
        public void onClick(String title);
    }

}
```

```Java
// Main.java

import java.util.Scanner;

public class Main {
    private static Scanner scanner = new Scanner(System.in);
    private static Button btnPrint = new Button("Print");

    public static void main(String[] args) {

//        class ClickListener implements Button.OnClickListener {
//            public ClickListener() {
//                System.out.println("I've been attached");
//            }
//
//            @Override
//            public void onClick(String title) {
//                System.out.println(title + " was clicked");
//            }
//        }
//
//        btnPrint.setOnClickListener(new ClickListener());
        btnPrint.setOnClickListener(new Button.OnClickListener() {
            @Override
            public void onClick(String title) {
                System.out.println(title + " was clicked");
            }
        });
        listen();
    }

    private static void listen() {
        boolean quit = false;
        while(!quit) {
            int choice = scanner.nextInt();
            scanner.nextLine();
            switch(choice) {
                case 0:
                    quit = true;
                    break;
                case 1:
                    btnPrint.onClick();

            }
        }
    }
}
```

As seen in the first code snippet, we can define a public interface inside of our class. In the second code snippet, we see some more of capabilities of inner classes. As we have a public interface as a part of the Button class, we can create an inner class than implements that interface. There are two ways in which we can do this. The first way is to explicitly declare and define a new class inside of main. Another way is to use an anonymous class. These both will have similar implementations, but different syntax.

**⇒ Abstract Class**

The same as C++. Check code for more information. We also use the extends keyword, like we did when doing normal inheritance. We can have classes that both extend from a parent class, and implement an interface. So, there’s couple things that need to be noted about abstract classes.

- The abstract methods **MUST** be implemented, similar to pure virtual classes
- If an abstract is inheriting from another abstract, then the methods do not need to be immediately overriden, but, by the time they reach non-abstract class, need to be overriden by then.
- Abstract classes can inherit from other classes as well as interface, at the same time.

```Java
public abstract class Bird extends Animal implements CanFly
{
    public Bird(String name) {
        super(name);
    }

    @Override
    public void eat() {
        System.out.println(getName() + " is pecking");
    }

    @Override
    public void breathe() {
        System.out.println("Breathe in, breathe out, repeat");

    }

    @Override
    public void fly() {
        System.out.println(getName() + " is flapping its wings");
    }
}
```

Assignment 2 is done. Check it out in the Java file.
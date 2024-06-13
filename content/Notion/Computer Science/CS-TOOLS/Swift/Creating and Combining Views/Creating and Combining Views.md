### Creating and Combining Views

This is the first section of learning SwiftUI framework. The begin, we first need to go over what a view is. A view is the fundamental building block in SwiftUI, and represents a visual element of the user interface. Views define the structure, appearance, and behaviour of different parts of an app's user interface.

Views in SwiftUI can represent a wide range of elements, such as text, images, buttons, lists, forms, navigation bars, and more. SwiftUI provides a library of built-in views, and you can also create custom views to encapsulate reusable UI components.

By composing and nesting views, you can create dynamic and interactive user interfaces in SwiftUI. The declarative nature of SwiftUI simplifies UI development by providing a concise and expressive way to describe the user interface hierarchy and behaviour.

We can have multiple Views. For instance, consider the following image:

![[Untitled 13.png|Untitled 13.png]]

In the above, we can see that there are 3 views: `ContentView`, `MapView`, and `CircleView`. These are sort of like components in React.

---

`**VStack**` **and** `**HStack**`:

So far, one of the most used ways to organize content on the screen is by 2. As their name suggests, they stack elements inside of them either vertically, or horizontally. In the above screen, we can see have a `VStack` to stack the map on top of the picture, on top of the other text. Likewise, we can nest these stacks within each other. So, in the beginning, we created a `MapView` to show the map. Note that the methods called on that object are `View` methods, and can be applied to any `view`. So, what is returned from that view? Here is what is in the `MapView.Swift` file:

```Swift
import SwiftUI
import MapKit

struct MapView: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 34.011_286, longitude: -116.166_868),
        span: MKCoordinateSpan(latitudeDelta: 0.2, longitudeDelta: 0.2)
    )

    // You use the @State attribute to establish a source of truth for data in your app that you can modify from more than one view. SwiftUI manages the underlying storage and automatically updates views that depend on the value.

    var body: some View {
        Map(coordinateRegion: $region)
        // By prefixing a state variable with $, you pass a binding, which is like a reference to the underlying value. When the user interacts with the map, the map updates the region value to match the part of the map that’s currently visible in the user interface.
    }
}

struct MapView_Previews: PreviewProvider {
    static var previews: some View {
        MapView()
    }
}
```

In SwiftUI, the `var body: some View` is a required property in a `View` type that defines the content and layout of the view. It represents the body of the view and specifies what should be displayed on the screen.

The `body` property is where you define the structure and composition of the view using SwiftUI's declarative syntax. Inside the body property, you can create a hierarchy of nested `view`s, apply modifiers, and combine different UI elements to define the user interface.

The `body` property is automatically invoked by SwiftUI to determine the structure and layout of the `view`. Any changes to the state or properties used within the body will trigger SwiftUI to reevaluate and update the `view` accordingly.
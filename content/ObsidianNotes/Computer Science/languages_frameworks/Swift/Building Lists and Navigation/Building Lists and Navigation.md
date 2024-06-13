### Building Lists and Navigation

**Preface**

In this note, we will build a Landmark Model. In the first note, you hard-coded information into all of your custom views. Here, you’ll create a model to store data that you can pass into your views.

![[Untitled 12.png|Untitled 12.png]]

Essentially, in the earlier section, we hard coded certain information. In this section, we will be changing this to display information that is read in from a JSON file. In addition, we will set up a list of all the locations, where each one can then be tapped to expand on it. We want to be able to dynamically generate a scrolling list that a user can tap to see a detail view for the landmark.

We know that we are displaying landmarks. So, instead of hard-coding the information for the landmark into the view, we make a `model`. Below is what it looks like:

```Swift
struct Landmark: Hashable, Codable, Identifiable { // this implements the hashale and codable protocol
    // Adding Codable conformance makes it easier to move data between the structure and a data file. You’ll rely on the Decodable component of the Codable protocol later in this section to read data from file.

    var id: Int
    var name: String
    var park: String
    var state: String
    var description: String

    private var imageName: String

    var image: Image {
        Image(imageName)
    }

    private var coordinates: Coordinates

    var locationCoordinate: CLLocationCoordinate2D {
        CLLocationCoordinate2D(
            latitude: coordinates.latitude,
            longitude: coordinates.longitude)
    }

    struct Coordinates: Hashable, Codable {
        var latitude: Double
        var longitude: Double
    }
}
```

Just like any framework, we create models for the data we are working with. Its worth mentioning, that we will need to move all pictures we are working with to be inside of the `Assets` folder.

In the model above, we have a couple of `computed properties`, such as `CLLocationCoordinate2D` and `image`. In addition, we can define a `struct` within the `landmark` `struct` for use within the landmark `struct`.

Now, we need to load in the Landmark data from the JSON being received. This can be found inside of the `ModelData.swift` file. The load method relies on the return type’s conformance to the `Decodable` protocol, which is one component of the `Codable` protocol.

Each element is read in, mapped to a Landmark model object, and gathered into one array.

---

Now, we need to continue adding components to complete the final view. Let's make a Landmark Row, which will all later be placed into a vertical stack where each row can then be clicked on to reveal more details.

Below is what the `LandmarkRow` `struct` looks like:

```Swift
struct LandmarkRow: View {
    var landmark: Landmark

    var body: some View {
        HStack {
            landmark.image
                .resizable()
                .frame(width: 50, height: 50)
            Text(landmark.name)

            Spacer()
        }
    }
}
```

As seen, its just a simple horizontal stack, with the landmark image on the left of the text description for that landmark image.

---

Then, each row is added to a vertical stack, forming a vertical list of landmark destinations. Here is the code for `LandmarkList` `struct`:

```Swift
struct LandmarkList: View {
    var body: some View {
        // When you use SwiftUI’s List type, you can display a platform-specific list of views. The elements of the list can be static, like the child views of the stacks you’ve created so far, or dynamically generated. You can even mix static and dynamically generated views.
        NavigationView {
            List(landmarks) { landmark in
                NavigationLink {
                    LandmarkDetail(landmark: landmark)
                } label: {
                    LandmarkRow(landmark: landmark)
                }
            }
            .navigationTitle("Landmarks")
            // title of the nav, displayed on the top by default
        }

    }
}
```

We need to bound the entire thing with a `NavigationView`. `NavigationView` is a SwiftUI container view that provides a navigation interface for your app. It is used to create hierarchical navigation structures, allowing users to navigate between different views in your app's user interface.

Outside of that, we have a list of Landmarks. Initially, we may do something like:

```Plain
List {
    LandmarkRow(landmark: landmarks[0])
    LandmarkRow(landmark: landmarks[1])
}
```

However, this is hard-coding the landmarks into the `List`. Instead of specifying a list’s elements individually, you can generate rows directly from a collection.

You can create a list that displays the elements of a collection by passing your collection of data and a closure that provides a view for each element in the collection. The list transforms each element in the collection into a child view by using the supplied closure. So, we now get:

```Swift
NavigationView {
    List(landmarks) { landmark in
        NavigationLink {
            LandmarkDetail(landmark: landmark)
        } label: {
            LandmarkRow(landmark: landmark)
        }
    }
    .navigationTitle("Landmarks")
    // title of the nav, displayed on the top by default
}
```

Notice in the above, we have passed the `landmarks` array to the list. Lists work with `identifiable` data. You can make your data identifiable in one of two ways:

- by passing along with your data a key path to a property that uniquely identifies each element
- Or by making your data type conform to the Identifiable protocol (which we did, for `landmark`, as there is an `id` parameter)

The rest of this section involves simply refactoring other parts of the code. So, to see more, check out the project code base.
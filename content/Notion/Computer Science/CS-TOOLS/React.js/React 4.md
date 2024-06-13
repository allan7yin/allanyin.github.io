## React Hooks

Hooks allow function components to have access to state and other React features. This is why class components are generally no longer used.

⇒ **What is a Hook?**

A hook is something that allows us to “hook” into React features such as state and life-cycle methods.

## useState

useState hook allows us to track state in a function component. State generally refers to data or properties that need to be tracking in an application.

```TypeScript
import { useState } from "react";
```

Now, let’s look at how we can initialize state at the top of the function component:

```TypeScript
import { useState } from "react";

function FavoriteColor() {
  const [color, setColor] = useState("");
}
```

We initialize our state by calling `useState` in our function component. useState accepts an initial state and returns two values:

- The current state
- The function that updates the state.

Notice that above, we have destructured the return value of `useState`. `color` is our current state and `setColor` is the function that is used to update our state. These two are simply names, and can be named whatever. Lastly, we set the initial state within the brackets of `useState()`. So, in the above code, the initial state is and empty string.

⇒ **Read state**

So now, we can now understand what the code below does:

```TypeScript
import React, { useState } from "react";
import ReactDOM from "react-dom/client";

function FavoriteColor() {
  const [color, setColor] = useState("red");

  return (
    <>
      <h1>My favorite color is {color}!</h1>
      <button
        type="button"
        onClick={() => setColor("blue")}
      >Blue</button>
      <button
        type="button"
        onClick={() => setColor("red")}
      >Red</button>
      <button
        type="button"
        onClick={() => setColor("pink")}
      >Pink</button>
      <button
        type="button"
        onClick={() => setColor("green")}
      >Green</button>
    </>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<FavoriteColor />);
```

⇒ **Update state**

To update state, we need to call the update state function:

```TypeScript
import { useState } from "react";
import ReactDOM from "react-dom/client";

function FavoriteColor() {
  const [color, setColor] = useState("red");

  return (
    <>
      <h1>My favorite color is {color}!</h1>
      <button
        type="button"
        onClick={() => setColor("blue")} // this is using the update function 
																				 // returned this by useState
      >Blue</button>
    </>
  )
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<FavoriteColor />);
```

⇒ **So, what can state hold?**

This hook can be used to keep track of strings, numbers, booleans, objects, really anything. Also, we can create multiple state hooks:

```TypeScript
import { useState } from "react";
import ReactDOM from "react-dom/client";

function Car() {
  const [brand, setBrand] = useState("Ford");
  const [model, setModel] = useState("Mustang");
  const [year, setYear] = useState("1964");
  const [color, setColor] = useState("red");

  return (
    <>
      <h1>My {brand}</h1>
      <p>
        It is a {color} {model} from {year}.
      </p>
    </>
  )
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Car />);
```

We can also use state to hold things like objects in memory, as so:

```TypeScript
import { useState } from "react";
import ReactDOM from "react-dom/client";

function Car() {
  const [car, setCar] = useState({
    brand: "Ford",
    model: "Mustang",
    year: "1964",
    color: "red"
  });

  return (
    <>
      <h1>My {car.brand}</h1>
      <p>
        It is a {car.color} {car.model} from {car.year}.
      </p>
    </>
  )
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Car />);
```

Above, we provide useState an initial value of an object. There are many data types we should be able to use in state. Now, let’s consider the code above, Fir a car objectm we can see there is various types of things about it, theres brand, model, year, etc. Say we want to change to the brand. If we do `setCar({color: "blue"})`, this entire current state becomes overwritten. Now, it would also be very impractical for us to pass in all the additional parameters if they are just going to be the same. So, we can do this:

```TypeScript
import { useState } from "react";
import ReactDOM from "react-dom/client";

function Car() {
  const [car, setCar] = useState({
    brand: "Ford",
    model: "Mustang",
    year: "1964",
    color: "red"
  });

  const updateColor = () => {
    setCar(previousState => {
      return { ...previousState, color: "blue" }
    });
  }

  return (
    <>
      <h1>My {car.brand}</h1>
      <p>
        It is a {car.color} {car.model} from {car.year}.
      </p>
      <button
        type="button"
        onClick={updateColor}
      >Blue</button>
    </>
  )
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Car />);
```

This now only overwrites the past colour property, with the rest of the state remaining the same.

## useEffect

The useEffect hook allows us to perform side effects in our components. Some examples of side effects are fetching data, directly updating the DOM, and setting timers. The syntax is as follows: `useEffect(<function>, <dependency>)`. Let’s first see an example of this without using the dependency part.

```TypeScript
import { useState, useEffect } from "react";
import ReactDOM from "react-dom/client";

function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      setCount((count) => count + 1);
    }, 1000);
  });

  return <h1>I've rendered {count} times!</h1>;
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Timer />);
```

So, the above code is interesting. Every second, count is incremented. When count is changed, the state has changed, which triggers a re-render. When we re-render, we then run `useEffect` again, which effectively then changes the state and triggers a re-render, and the loop goes on. So, the code above will go on forever, as the value continues to be incremented.

  

This, however, is not something we want. We want to be able to control when side effects run, this why we need the dependency part of `useEffect`. The second argument is an array, and when anything inside the array changes, then we can run `useEffect` again. Here are some short examples:

```TypeScript
useEffect(() => {
  //Runs on every render
});
```

```TypeScript
useEffect(() => {
  //Runs only on the first render
}, []);
```

```TypeScript
useEffect(() => {
  //Runs on the first render
  //And any time any dependency value changes
}, [prop, state]);
```

Now, the code below should be easy to follow and implement:

```TypeScript
import { useState, useEffect } from "react";
import ReactDOM from "react-dom/client";

function Counter() {
  const [count, setCount] = useState(0);
  const [calculation, setCalculation] = useState(0);

  useEffect(() => {
    setCalculation(() => count * 2);
  }, [count]); // <- add the count variable here

  return (
    <>
      <p>Count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+</button>
      <p>Calculation: {calculation}</p>
    </>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Counter />);
```

Sometimes, these effects will use memory when run. So, we may need to do some effect clean up afterwards:

```TypeScript
import { useState, useEffect } from "react";
import ReactDOM from "react-dom/client";

function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    let timer = setTimeout(() => {
    setCount((count) => count + 1);
  }, 1000);

  return () => clearTimeout(timer)
  }, []);

  return <h1>I've rendered {count} times!</h1>;
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Timer />);
```

The return function that calls clearTimeout is what we use.

## useContext

This hook provides a way for us to manage state globally.
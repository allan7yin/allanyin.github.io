## React Props

`props` stands for properties, as mentioned in the previous note. We pass these into React components. The best way to explain props is:

> React Props are like function arguments in JavaScript _and_ attributes in HTML.

Here is how props are added, and used:

```TypeScript
const myElement = <Car brand="Ford" />;
// above, the only prop is brand

function Car(props) {
  return <h2>I am a { props.brand }!</h2>;
}
// so, when we render myElement, we will see: "I am a Ford!"
```

⇒ **Passing data**

Through props, we can pass certain variables/data from a function to a component, on and on…

```TypeScript
function Car(props) {
  return <h2>I am a { props.brand }!</h2>;
}

function Garage() {
  const carName = "Ford";
  return (
    <>
      <h1>Who lives in my garage?</h1>
      <Car brand={ carName } />
    </>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Garage />);
```

Even objects can work:

```TypeScript
function Car(props) {
  return <h2>I am a { props.brand.model }!</h2>;
}

function Garage() {
  const carInfo = { name: "Ford", model: "Mustang" };
  return (
    <>
      <h1>Who lives in my garage?</h1>
      <Car brand={ carInfo } />
    </>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Garage />);
```

  

Now, it’s time to begin looking at some of the more useful and dynamic capabilities of React.

## React Events

React can perform actions based on user events. The events are the ones you know (onHover, onClick, click, change, mouseHover, etc.) There are many.

⇒ **Adding events**

React event handlers are written inside of curly brackets, something like `onClick{shoot}`

```TypeScript
// React 
<button onClick={shoot}>Take the Shot!</button>

// HTML
<button onclick="shoot()">Take the Shot!</button>
```

```TypeScript
function Football() {
  const shoot = () => {
    alert("Great Shot!");
  }

  return (
    <button onClick={shoot}>Take the shot!</button>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Football />);
```

⇒ **Passing arguments**

To pass an argument to an event handler, we use an arrow function. Above, shoot is passed nothing, to pass it something, we use an arrow function to do so:

```TypeScript
function Football() {
  const shoot = (a) => {
    alert(a);
  }

  return (
    <button onClick={() => shoot("Goal!")}>Take the shot!</button>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Football />);
```

## React Event Object

Event handlers have access to the React event that triggered the function.

```TypeScript
function Football() {
  const shoot = (a, b) => {
    alert(b.type);
		/*
		'b' represents the React event that triggered the function.
    In this case, the 'click' event
		*/
  }

  return (
    <button onClick={(event) => shoot("Goal!", event)}>Take the shot!</button>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Football />);
```

  

## React Conditional Rendering

This is a simple idea, rendering different things to the screen based on different things

⇒ **If/else statements**

```TypeScript
function MissedGoal() {
  return <h1>MISSED!</h1>;
}

function MadeGoal() {
  return <h1>Goal!</h1>;
}

function Goal(props) {
  const isGoal = props.isGoal;
  if (isGoal) {
    return <MadeGoal/>;
  }
  return <MissedGoal/>;
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Goal isGoal={false} />);
```

⇒ **logical Operators**

```TypeScript
function Garage(props) {
  const cars = props.cars;
  return (
    <>
      <h1>Garage</h1>
      {cars.length > 0 &&
        <h2>
          You have {cars.length} cars in your garage.
        </h2>
      }
    </>
  );
}

const cars = ['Ford', 'BMW', 'Audi'];
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Garage cars={cars} />);
```

Above, we can have expressions inside of the curly braces. If the condition (`cars.length > 0`) holds true, we do what comes after the `&&`. So, the above code will produce this:

![[Screen_Shot_2022-10-21_at_10.20.27_AM.png]]

As seen, the `h2` is right below the `h1`. Now, if the cars array was empty, we would not see the `h2`.

⇒ **Ternary operator**

Recall that this is of the form `condition ? true: false`. We can use it like this:

```TypeScript
function Goal(props) {
  const isGoal = props.isGoal;
  return (
    <>
      { isGoal ? <MadeGoal/> : <MissedGoal/> }
    </>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Goal isGoal={false} />);
```

Of course, the above three methods achieve the same purpose, and the second two really only serve to make code shorter and more organized, easier to read.

## React Lists

What if we want to render a list of something? To do so, we like to use `.map()` method, and passing each array item to a component function.

```TypeScript
function Car(props) {
  return <li>I am a { props.brand }</li>;
}

function Garage() {
  const cars = ['Ford', 'BMW', 'Audi'];
  return (
    <>
      <h1>Who lives in my garage?</h1>
      <ul>
        {cars.map((car) => <Car brand={car} />)}
      </ul>
    </>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Garage />);
```

However, running the above may lead you to a small warning, that is, there is no “key” for the list of items.

⇒ **Keys**

React can use keys to keep track of elements. This way, if an item is updated or removed, only that item will be re-rendered instead of the entire list. Each key unique to each sibling, but, can be duplicated globally. So, per “array”, keys must be unique. Here’s a small demonstration:

```TypeScript
function Car(props) {
  return <li>I am a { props.brand }</li>;
}

function Garage() {
  const cars = [
    {id: 1, brand: 'Ford'},
    {id: 2, brand: 'BMW'},
    {id: 3, brand: 'Audi'}
  ];
  return (
    <>
      <h1>Who lives in my garage?</h1>
      <ul>
        {cars.map((car) => <Car key={car.id} brand={car.brand} />)}
      </ul>
    </>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Garage />);
```

Cars is just a dictionary now. Now, this code snippet will produce the same thing as the one above it. Only difference is now, React can track these items in the list.

## React Forms

Let’s look at creating forms with React. Let’s first look at the most straightforward way:

```TypeScript
function MyForm() {
  return (
    <form>
      <label>Enter your name:
        <input type="text" />
      </label>
    </form>
  )
}
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<MyForm />);
```

This will work as normal, the form will submit and the page will refresh. But this is generally not what we want in React, we want to prevent this default behaviour and let React control the form.

⇒ **Form handling**

In normal HTML, form data is handled by the DOM, but in React, we want to have form data be handled by the components. We can control changes by adding event handlers in the `onChange` attribute. We can use the `useState` Hook to keep track of each input’s value and provide a “single source of truth” for the entire application. We’ll look at these in **React 4**

---
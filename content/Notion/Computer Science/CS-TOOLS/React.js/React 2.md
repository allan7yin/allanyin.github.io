## React Render HTML

`ReactDOM.render()` takes on two arguments, the first one is the element we wish to render, and the second argument is the parent we wish to make the element a child of. Here is a quick short example:

```TypeScript
ReactDOM.render(<p>Hello</p>, document.getElementById('root'));

// this is the parent we are appending the child element to 
<body>
  <div id="root"></div>
</body>
```

## React JSX

A quick review, JSX is the functionality that allows us sort of write code that is both JavaScript and HTML. Below, let’s quickly contrast the differences between using it and not using it:

```TypeScript
// With JSX 
const myElement = <h1>I Love JSX!</h1>;

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(myElement);

// Without JSX 
const myElement = React.createElement('h1', {}, 'I do not use JSX!');

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(myElement);
```

We can also write and execute expressions inside of JSX curly braces:

```TypeScript
const myElement = <h1>React is {5 + 5} times better with JSX</h1>;
```

To write the HTML on multiple lines in JSX, wrap it in some parentheses:

```TypeScript
const myElement = (
  <ul>
    <li>Apples</li>
    <li>Bananas</li>
    <li>Cherries</li>
  </ul>
);
```

However, for all of these things, when assigning some JSX element to a variable, it must be a single parent element:

```TypeScript
// This is not VALID 
const myElement = (
	<div> div number 1 </div>
	<div> div number 2 </div>
)

// to do the above, need to wrap it in another SINGLE element 
const myElement = (
	<div>
		<div> div number 1 </div>
		<div> div number 2 </div>
	</div>
)

// However, the prefered way is to use fragments 
const myElement = (
	<>
		<div> div number 1 </div>
		<div> div number 2 </div>
	</>
)
```

We use `class` in `HTML`, but as it is a reserved word in `JavaScript`, we use `className` instead.

  

## React Components

Components are independent and reusable bits of code. They serve the same purpose as JavaScript functions, but work in isolation and return HTML. While there are both class and function components, it is no longer recommended to use classes, and instead, use things such as Hooks.

  

⇒ **Function Component**

```TypeScript
function Car() {
  return <h2>Hi, I am a Car!</h2>;
}
```

⇒ **Rendering a component**

So above, we have mad a Car component that returns some HTML. To use it, we can do:

```TypeScript
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Car />);
```

→ **Props**

Props stand for React properties. Props are like function arguments, and you send them into the component as attributes. Components can be passed like props.

```TypeScript
function Car(props) {
  return <h2>I am a {props.color} Car!</h2>;
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Car color="red"/>);
```

→ **Components in components**

We can use components inside of other components.

```TypeScript
function Car() {
  return <h2>I am a Car!</h2>;
}

function Garage() {
  return (
    <>
      <h1>Who lives in my Garage?</h1>
      <Car />
    </>
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Garage />);
```

## Components in Files

Now, the above components, we can put those in separate .js files, and simply export them. Then, we can just import them whenever we need them.
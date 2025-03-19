[[Web And Multimedia Technologies]]
***
**Table of Contents**
```table-of-contents
```

****
## React?

Front-end JavaScript **library** for building user interfaces based on re-usable UI components. Widely used in the development of single-page applications (SPAs), mobile, or server-rendered applications... However, React is only concerned with state management and rendering that state to the DOM.
	*React was designed to remain agnostic on features beyond state management and rendering (e.g., how you should handle asynchronous calls to distant APIs, validate a form, design your SPA with routes...). This also means that React is delivered in a minimalistic manner, only including the most essential components. It is up to the developer to import external libraries if he wishes to access more fleshed-out or exotic features, which is no problem with react as its ecosystem and community is very vast.
	Examples: React-router for SPAs, Axios to send HTTP/XMLHttpRequests requests, React Hook Form & Zod to validate forms, React Query to manage asynchronously-fetched data...*

Two main entry points for rendering a React application: **React** and **ReactDOM** (which are two separate libraries):
1. **React**: The core library of React responsible for defining and managing UI components. Includes the logic for creating components, managing their state, and handling their lifecycle.
    React itself does not interact directly with the browser's DOM. Instead, it focuses on the virtual representation of the UI.
2. **ReactDOM**: Acts as a bridge between React and the browser's DOM by providing methods to render React components into the actual DOM.
    The primary method used from ReactDOM is `ReactDOM.createRoot`, which creates a root for the React application and allows you to render the component tree into a specified DOM node.

Components are re-usable units described in **JavaScript Syntax eXtension (JSX)**, which allow to write HTML code as a JavaScript expression and place them in the DOM without using methods like `createElement()`.
We use `{` `}` to insert JavaScript expressions or variables inside the JSX:
```jsx
let value = 3;
let node = document.getElementById("react-jsx");
let root = ReactDOM.createRoot(node);                      // Asks react to manage this node of the DOM
root.render(<div align="center">result = {value} !</div>); // Displays the JSX as a child of the root node
// root.render(React.createElement("div", { align: "center" }, "result = ", value, " !"));
```
> [!info]
> 
> Under the hood, the JSX is transformed by calling `React.createElement(type, props, ...children)`.
> The commented line in the example shows the equivalent code without JSX:

JSX is not natively understood by JavaScript engines, so it needs to be transpiled into regular JavaScript (transforms our `.jsx` files containing a component to a traditional `.js` file). This process is typically handled by tools like Babel or esbuild. Here's how it works:
1. **Parsing**: The JSX code is parsed into an AST
2. **Transformation**: The AST is transformed by converting JSX syntax into `React.createElement` calls
3. **Code Generation**: The transformed AST is converted back into JavaScript code that can be executed by the browser

```bash
npm install --save-exact --save-dev esbuild
./node_modules/.bin/esbuild test.jsx --bundle --outfile=test.js
```
> [!tip]
> Esbuild bundles both react and react-dom


***
## Virtual DOM

React relies on a mutable **Virtual DOM** to optimise the rendering process on the real DOM in a reactive manner (which means React manages the events and the state of objects, and will "react" when something happens).
The vdom is a lightweight copy of the DOM (using Js objects instead of C++ ones) maintained by React in order to track changes in the UI. It works like this:
1. **Initial Render**: React creates a Virtual DOM that mirrors the real DOM
2. **State Changes**: When the state of a component changes, React creates a new Virtual DOM that reflects the updated state
3. **Diffing**: React compares the new Virtual DOM with the previous one to identify what changes need to be made to the real DOM
4. **Reconciliation (Commit)**: React updates the real DOM with the identified changes, ensuring that only the necessary updates are made

![[virtual-dom.png|350]]


***
## Components
### Structure

In the past a component was a class, but this was verbose and far from being convenient.
The modern way of declaring a component is via a reusable function:
```tsx
// Hello.tsx
export default function Hello() { // Export so it can be used from the outside
  return (
    <div>
      <h1>Hello</h1>
    </div>
  );
}

// Main file
import React from 'react';
import ReactDOM from 'react-dom';
import Hello from './Hello';

let node = document.getElementById("react-root");
let root = ReactDOM.createRoot(node);
root.render(<div><Hello/></div>);
```
> [!info]
> JSX differentiates HTML tags and components by requiring the later to begin with a capital letter.
> As we can see above, we may use the auto-closing `<Hello/>` syntax (from XHTML), even though the classic `<Hello> </Hello>` exists (useful for children, we'll see that later).

Since those are functions, we can pass them arguments called **props** (for properties) in the context of components:
```tsx
function Hello({ name, amount }) {
  return (
    <div>
      <h1 className="title-big">Hello, {name}!</h1>
      <p>Amount: {amount}</p>
    </div>
  );
}

// Main file
let node = document.getElementById("react-root");
let root = ReactDOM.createRoot(node);
root.render(<Hello name="Alice" amount={100} />);
```
> [!info]
> If a prop isn't a string, it must be passed through `{` `}`.
> As both JavaScript and HTML defines the `class` keyword for a different use case, we replace it with `className` in our JSX.

If using TypeScript—which is recommended—there is a convention requiring you to ensure type safety in a local interface named `ComponentnameProps`: 
```tsx
interface HelloProps {
  name: string;
  amount: number;
}

const Hello: React.FC<HelloProps> = ({ name, amount }) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>Amount: {amount}</p>
    </div>
  );
};

export default Hello;
```

The **returned value can only be a single element**. Our example above works because our `<h1>` and `<o>` tags are encapsulated by a single `<div>` tag which is what we really return. If we remove this div, our code doesn't work anymore as we attempt to return two values: the title and the paragraph.
An easy—but naive—fix is to encapsulate everything in divs, but this will end up filling our page with useful containers. A better alternative is to use **fragments** (`<> </>`, which packs several elements and allows us to return them as one:
```jsx
function Story() {
  return 
	<>
      <div>Title</div>
      <div>blah blah</div>
    </>;
}
```

### Conditional render

The `condition && <Whatever/>` is a convenient syntax available for conditional rendering of a component inside our JSX:
```jsx
function Hello(prop) {
  return prop.visible && <span>Hello !</span>;
  //return prop.visible ? <span>Hello !</span> : "";
}

let node = document.getElementById("react-conditional");
let root = ReactDOM.createRoot(node);
root.render(<div><Hello visible={true}/><Hello visible={false}/></div>);
```
> [!tip] Reminder
> In JavaScript, the `&&` operator returns the second operand if the first is truthy, otherwise it returns the first operand (false). This allows to directly render our component if the condition is true, instead of relying on a not-so-convenient ternary operator.

### Events

Components are supposed to be **pure functions**, which means it has single responsibility (doesn't mutate pre-existing variables it receives) and produces the same output for a same input (components we write must always return the same JSX given the same inputs).
React offers a **Strict Mode** which calls each component’s function twice during development, ultimately highlighting impure components.
	*So, don't be surprise if you use this mode and see that your logs appears twice in the devtools console*

Side effects should only be triggered by events (`onClick`, `onKeyUp`...), or by dedicated hooks like `useEffect` (more on that later).
You usually pass a function to those event properties, which can either be defined inline (arrow function) or by **passing a reference to a handle function**:
```jsx
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  // Both are the same
  return (
	<>
		<button onClick={handleClick}>
		      Click me
	    </button>
		<button onClick={function handleClick() {
			alert('You clicked me!');
		}}>
			Click me
		</button>
	</>
  );
}
```
> [!caution]
> Note that you should always **pass the function, not call it** (which is why we passed `handleClick` and not `handleClick()`).
> Passing it allows React to call it whenever it is necessary, while calling it will trigger it every time the component is rendered without any user action (which is not what we want).

A common design is to pass event handlers as props down to a child component:
```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Playing ${movieName}!`);
  }

  return (
    <Button onClick={handlePlayClick}>
      Play "{movieName}"
    </Button>
  );
}

export default function Toolbar() {
  return (
    <PlayButton movieName="Kiki's Delivery Service" />
  );
}
```

Events (besides `onScroll`) are **propagating** up the tree when triggered, We can prevent this behaviour by calling the `stopPropagation()` instance method on the event:
```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <Button onClick={() => alert('Playing!')}>
        Play Movie
      </Button>
      <Button onClick={() => alert('Uploading!')}>
        Upload Image
      </Button>
    </div>
  );
}
```
> [!info]
> Without `stopPropagation()`, clicking on the "playing" button would trigger the "Playing!" alert first, and the "You clicked on the toolbar!" right after. Now, the later won't trigger as we prevented the propagation.
> A very common use case of this is when submitting forms, as clicking on a button inside those will, by default, refresh the whole page.


***
## Lists

React identifies and maintain elements of a list via a **unique** and **final** `key` property assigned to each item. Without the key, a change requires React to re-render the entire list as it cannot keep track of which specific value changed (big performance issue, especially on big ass lists).

```jsx
function ItemList({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```
> [!tip] Reminder
> `map` is pretty much the go-to function to traverse and render lists in React. As we know, it creates a new array by applying a provided function to each element in the original array.


***
## State

The most important feature in React, as it allows it to "react" when data changes lol.
In a few words:
- **Dynamic Data**: State is used to store data that may change over the lifetime of a component. Unlike props—which are read-only and passed down from parent components—state is managed within the component itself
- **Component Lifecycle**: State can change throughout a component's lifecycle, triggering re-renders to reflect the updated data in the UI
- **Local to Component**: Each component manages its own state, making it local and encapsulated

### State hooks

Hooks are special functions provided by React, allowing the developer to give a state to functional components without the need to write a class. Unlike local variables, those will **retain the data between renders**, and **trigger re-rendering on new data**.

The most common of those hooks is the `useState(initialState)`.
It takes the initial state of the component as an argument, and returns an array with two elements: the current state value and a function to update it.
```jsx
const [state, setState] = useState(initialState);
```
> [!important]
> We should never mutate the state variable directly, as this doesn't allow react to re-render components depending on this value. We should always always rely on the `setState` function to replace the value of `state` with a new value.

A common example to understand this is a basic counter we initiate at 0, and increase by one on each click:
```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

State hooks are stored in an array associated to a node in the vdom. As long as the node isn't deleted, it's state persists.

![[virtual-dom-state-hooks.png|275]]

### Seing state as "snapshots"

Its a comparison we often do when trying to explain a newcomer how to understand states.
Rendering means React is calling your component (a function). The returned JSX is like a snapshot of the UI in time, as all its proprs, handlers and local variables were calculated using its state **at the time of the render**.
	*React calls the function => Function returns a new JSX => React updates the screen to match the snapshot we just returned*

A state variable lives outside of those renders, they are kept alive by react (unlike local variables).
Based on the code bellow, we would expect a button click to increase the counter by 3, but this is not the case... It only increases by one:
```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

This is because changing the state only applies when the upcoming render is done, this is **not an instant mutation**. If we call this for the first time, `number` will be 0. Even though we called it three times, `number` is still 0 for each, as the state update is only applied once we left the function.
We will simply replace `number` with its value at the time (let's say `0`). This is the code we executed, it clearly highlights the problem:
```jsx
<button onClick={() => {
  setNumber(0 + 1);
  setNumber(0 + 1);
  setNumber(0 + 1);
}}>+3</button>
```
> [!info]
> In fact, only the last instruction will be taken into account, as it kind of overrides the others.
> Since the state is only updated during re-render, adding an `alert(number)` on the last line would print `0`, not `1` (and even if we add a big `setTimeout`, it doesn't change anything as we spinned an alert with **the value of `number` at the time**).
> 	*Try to remember this substitution technique to visualise what the snapshot is, and what it will be. I find it very helpful*

### Queue of state updates
*Uncommon scenario but it's useful to understand how it works*

But how do I make the situation described above work? As we know, the re-render is only applied after all our `setNumber()` instructions have been executed, which prevents us from updating the same state multiple times prior to the re-render.
In this case, you do it like this:
```jsx
<button onClick={() => {
    setNumber(n => n + 1);
    setNumber(n => n + 1);
    setNumber(n => n + 1);
}}>+3</button>
```

Here, instead of passing the next state value, we pass an **updater function** that calculates the next state based on the previous one in the queue.

### States on objects

We should treat states as immutable (read only), as mutating will not trigger renders and will change the state in previous render “snapshots”.
Instead, we create a new version of the object, and trigger a re-render by **setting state** to it.
```tsx
interface Position {
    x: number;
    y: number;
}

const MovingDot = () => {
    const [position, setPosition] = useState<Position>({
        x: 0,
        y: 0
    });

    const handlePointerMove = (e) => {
	    // Incorrect, mutating the state:
	    // onPointerMove={e => {
		//    position.x = e.clientX;
		//    position.y = e.clientY;
		// }}

		// Correct, using the updater function
        setPosition({
            x: e.clientX,
            y: e.clientY
        });
    };
}
```

As we need to include *already existing* data as a part of our new object, we may rely on the **spread syntax** (`...`) to do so.
Let's say we have an entire object with several fields, but we triggered a re-render because only one of it's value (the first name field in this case) was modified:
```tsx
interface Person {
    firstName: string;
    lastName: string;
    email: string;
}

// Manual copy of data, very verbose
const handleFirstNameChange = (e) => {
	setPerson({
	  firstName: e.target.value, // The only value we are really interested in
	  lastName: person.lastName,
	  email: person.email
	});
}

// Object spread
const handleFirstNameChange = (e) => {
    setPerson({
        ...person,
        firstName: e.target.value // The only value we are really interested in
	});
};
```

The problem with the spread syntax is that it is doing **shallow copies** (one level deep), which means that if our object contains another object and we modify one of its properties, we will have to use a spread syntax per "nest":
```tsx
interface Artwork {
    title: string;
    city: string;
    image: string;
}

interface Person {
    name: string;
    artwork: Artwork;
}

const Form: React.FC = () => {
    const [person, setPerson] = useState<Person>({
        name: 'Niki de Saint Phalle',
        artwork: {
            title: 'Blue Nana',
            city: 'Hamburg',
            image: 'https://i.imgur.com/Sd1AgUOm.jpg',
        }
    });

    const handleNameChange = (e) => {
        setPerson({
            ...person,
            name: e.target.value
        });
    };

    const handleTitleChange = (e) => {
        setPerson({
            ...person,
            artwork: {
                ...person.artwork,
                title: e.target.value
            }
        });
    };

    const handleCityChange = (e) => {
        setPerson({
            ...person,
            artwork: {
                ...person.artwork,
                city: e.target.value
            }
        });
    };

    const handleImageChange = (e) => {
        setPerson({
            ...person,
            artwork: {
                ...person.artwork,
                image: e.target.value
            }
        });
    };
// ...
```
> [!tip]- Tip: Immer
> A useful fix for this is the [Immer](https://github.com/immerjs/use-immer)library which allow us to write code looking like mutation, but will do the copies on our behalf under the hood:
> ```ts
> const handleNameChange = (e) => {
>     setPerson(produce(draft => {
>         draft.name = e.target.value;
>     }));
> };
> 
> const handleTitleChange = (e) => {
>     setPerson(produce(draft => {
>         draft.artwork.title = e.target.value;
>     }));
> };
> 
> const handleCityChange = (e) => {
>     setPerson(produce(draft => {
>         draft.artwork.city = e.target.value;
>     }));
> };
> 
> const handleImageChange = (e) => {
>     setPerson(produce(draft => {
>         draft.artwork.image = e.target.value;
>     }));
> };
> ```

### State structure

It is important to avoid redundancy of state variables as much as possible, as those tends to introduce contradictions in your code with time. Below, the `fullName` state variable should be removed:
```tsx
const MyComponent = () => {
	const [firstName, setFirstName] = useState<string>('');
	const [lastName, setLastName] = useState<string>('');
	const [fullName, setFullName] = useState<string>('');
	
	const handleFirstNameChange = (e) => {
	    setFirstName(e.target.value);
	    setFullName(e.target.value + ' ' + lastName);
	};
	
	const handleLastNameChange = (e) => {
	    setLastName(e.target.value);
	    setFullName(firstName + ' ' + e.target.value);
	};
	
	// ...
}
```
> [!tip]
> Instead, we simply calculate `fullName` during rendering:
>```tsx
>const fullName = firstName + ' ' + lastName;
>```


Another common mistake is *mirroring props in state*, where a state variable is created based on a given prop. If the parent component passes a different value for the prop later on, this state variable will not be updated:
```tsx
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
```

We can simply use the prop directly. In fact, we only do mirroring if we want to ignore any update on a specific prop. In this case, the convention is to begin the prop name with `initial` or `default`.


Another situation to avoid is to nest objects too much. Prioritise a **flat** (normalised) design when possible (e.g., an array of children only containing each's id).


> [!tip]
> If two components must change together, remove state from both of them, move it to their closest common parent, and then pass it down to them via props. This is known as **lifting state up**, which is an extremely common pattern in React.

### React to input

React offers a declarative approach over an imperative one: we declare what we want to show, and it's React's job to update the UI for us depending on our criterias (like in SQL: We describe what we want, it's up to the system to find the data).
A few things to consider each time we need to implement states on some elements of our UI:
1. **Identify** component’s different visual states
	Submit button is disabled is form is empty; enabled if typed; entire form is disabled when submitting (with spinner, if possible); error/success message is displayed depending on what we received from our async function.
2. **Determine** what triggers those state changes
   Starting to type make the button clickable; Clicking the button submits the form; Ongoing submission blocks the form; Receiving the response displays a message.
3. **Represent** the state in memory using `useState`
4. **Remove** any non-essential state variables
   Are some states causing a paradox; Are some states carrying information I use in the same way, or that I can get from an inverse of another variable (if I have an `error` state, I don't need an `isError` state since I can already guess it).  
5. **Connect** the event handlers to set the state
   We create the `handler` functions dedicated to updating the states depending on how thing goes 

```jsx
export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>That's right!</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          Submit
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('Wrong answer. Try again!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```


***
## Reducer

A reducer is a function that determines changes to an application's state. It is commonly used in state management, particularly with the `useReducer` hook in React. The concept of a reducer originates from Redux (library providing global state management, unlike the local state management we are talking about since the beginning), a popular state management library, but React provides a built-in way to use reducers with hooks.
This hook allows to consolidate all the state update logic that involves multiple sub-values or a next state depending on the previous one.

```js
const [state, dispatch] = useReducer(reducer, initialState);
```
- `state`: The current state value
- `dispatch`: A function to dispatch actions to the reducer
- `reducer`: A function that takes the current state and an action as arguments, and returns the new state
- `initialState`: The initial state value

Try to see the reducer as a hopper linking several actions into a same point of contact. This is also why we often use a swtich case inside of it.
Here's an example:
```jsx
import React, { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>Increment</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>Decrement</button>
    </div>
  );
}
```
> [!info]
> In this example, the `reducer` function handles the state changes based on the dispatched actions. The `dispatch` function is used to trigger state updates by sending actions to the reducer.
> As you can see, it is good practice to put the reducer in its own separate file, outside of the component file (or at least outside of the component function itself).

Most of the time, you will migrate from `useState`(s) to a `useReducer` because the logic becomes too complex and dispatched. Here are good steps to follow for the migration to unfold flawlessly:
1. **Move** from setting state to dispatching actions
   Remove all the state-setting logic, and take a look at the handler functions you are left with
   Instead of telling React “what to do” by setting state, you specify “what the user just did” by dispatching “actions” from your event handlers. So instead of “setting `tasks`” via an event handler, you’re dispatching an “added/changed/deleted a task” (switch case) action. This is more descriptive of the user’s intent.
```jsx
// Similar to the inline dispatch references we passed above
function handleAddTask(text) {
  dispatch({ // Pass an action. It can have any shape you want, it only describes what "happened"
    type: 'added',
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

2. **Write** the reducer function
   Takes the current state and the action object, and returns the next state
3. **Use** the reducer from your component

> [!tip]
> - Reducer must be pure, not perform side effects, send timeouts...
> - Each action is a single user interaction, even if that leads to multiple changes in the data


***
## Context


***
## Effect



***
## Refs

Refs are our first example of **escape hatch**, features "stepping outside React".
It allows to remember information without triggering any re-render (so, like `useState` but without the need of maintaining a state):
```tsx
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);             // Returns a mutable object with a "current" field

  function handleClick() {
    ref.current = ref.current + 1; // Instead of using a state, we mutate the object
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```

Main use case of refs is to manipulate the DOM (focus on an element, scroll to it, measure its size or position).
JSX introduces a dedicated `ref` property on tags, which allows to populate the ref with **a reference to this node** into `myRef.current`:
```tsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null); // null by default, doesn't point to any node

  function handleClick() {
    inputRef.current.focus();    // "focus()" is one of the many available functions, all of them
						         // being built-in browser APIs
  }

  return (
    <>
      <input ref={inputRef} />   {/* Fills inputRef.current with this node */}
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```
> [!info]
> Refs are set during the commit (right after updating the DOM, as we don't want to access refs during rendering).
> You are supposed to access refs through **event handlers** (like we did in the example above).


***
## Routing
*React Router V7*

Well that's Kool and the Gang, but we don't really plan on doing webapps with a single page right? Fortunately, the React ecosystem includes libraries which comes handy when it comes to manage our routes in an SPA manner, the most popular one being [React Router](https://github.com/remix-run/react-router#readme), which  matches the URLs to specific components of the app.
```bash
npm i react-router-dom
```

### Modes

React Router is a "multi-strategy" router which supports different **modes** for handling routing, depending on your application's needs.
Three approach:
1. **Declarative**: Basic routing features like matching URLs to components, navigating around the app, and providing active states with link components
   *The simplest approach, with basic features expected from a routing library*
2. **Data**: Supports data loading, actions, pending states and more
	*If there is a need for control over bundling, data, and server abstractions*
3. **Framework**: Wraps Data Mode with a Vite plugin to add the full React Router experience
   *If there is a need for server render, replace Next.js, or comes from Remix*

We will only take a look at the declarative mode in this lecture.

### Declarative approach

Most of the actions will be performed in the main file of our project, or on components redirecting to other "pages". 
This mode allows us to configure routes in a very straightforward way by using mostly three components:
1. `<BrowserRouter>`, which uses the HTML5 history API to keep the UI in sync with the URL. It should wrap the entire application–or at least the part of it where the routing is configured.
2. `<Routes>`, which is simply a container for several routes defined with the `<Route>` component.
3. `<Route>`, which maps a `path` to an `element` (component) to render. If the visited URL matches, the according component is loaded.

Once defined in our main file, we should have something like this:
```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter, Routes, Route } from "react-router";
import App from "./app";

const root = document.getElementById("root");

ReactDOM.createRoot(root).render(
  <BrowserRouter>
    <Routes>
      <Route path="/" element={<App />} />
      <Route path="/about" element={<About />} />
      <Route path="/contact" element={<Contact />} />
    </Routes>
  </BrowserRouter>
);
```


Of course, dynamic routes are available and allows to define routes with variable segments via `:`, which is mandatory for rendering components based on dynamic data:
```tsx
<Route path="user/:userId" element={<UserProfile />} />
```

We can then access the route parameters from the inside of the matched component (`<UserProfile>` in this case) by using the `useParams` hook:
```tsx
import { useParams } from 'react-router-dom';

const UserProfile = () => {
    const { userId } = useParams();
    return <div>User ID: {userId}</div>;
};
```

If needed, you can make a route segment optional by adding a `?` to the end of the segment:
```tsx
<Route path=":lang?/categories" element={<Categories />} /> {/* Optional dynamic arg */}
<Route path="users/:userId/edit?" element={<User />} />     {/* Optional static arg "edit" */}
```

Those params can then be retrieved from `useSearchParams`, which returns an instance of [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams):
```tsx
function SearchResults() {
  let [searchParams] = useSearchParams();
  return (
    <div>
      <p>
        You searched for <i>{searchParams.get("q")}</i>
      </p>
      <FakeSearchResults />
    </div>
  );
}
```



We may also define **nested routes**, for hierarchical UIs:
```tsx
<Route path="dashboard" element={<Dashboard />}>
    <Route index element={<Home />} />               {/* Renders at /dashboard */}
    <Route path="settings" element={<Settings />} /> {/* Renders at /dashboard/settings */}
</Route>
```
> [!info]
> The `index` route is a special route that matches when the parent route matches exactly.
>  (in other words, it's like a default child route)
>  It's useful for rendering a default component when no specific child route is matched. Index routes cannot have children.

Child routes are rendered through the `<Outlet/>` in the parent route:
```tsx
import { Outlet } from "react-router";

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      {/* will either be <Home/> or <Settings/> */}
      <Outlet />
    </div>
  );
}
```

### Links and navigation

So far our paths are mapped, but it can only be leveraged if we manually type the paths in the URL, which is far from being what we want. If we want to introduce navigational links in our app, we can use the `<Link>` component:
```tsx
import { Link } from 'react-router-dom';

<Link to="/dashboard">Go to Dashboard</Link>
```
> [!info]
> `<Link>` is not completely similar to a traditional anchor tag (`<a>`) in HTML, as the later has to reload the entire page.
> The link tag is way quicker as it prevents full page reloads and keeps the state of the application, even if the client has a slow internet connection. 


The `<NavLink>` component is simply an extension of the `<Link>` component which adds styling attributes (e.g., highlight the active link, visited links).
```tsx
import { NavLink } from 'react-router-dom';

<NavLink to="/about" activeClassName="active">
  Go to About
</NavLink>
```
> [!info]
> In this example, the link will have the class `active` applied when the current URL matches `/about`.


Finally, the `useNavigate` hook can be used to navigate the user to a new page without the user interacting. Usually, this is used when we wish to perform navigation in response to some logic or user interaction, rather than directly through a clickable link:
```tsx
import { useNavigate } from "react-router";

export function LoginPage() {
  let navigate = useNavigate();

  return (
    <>
      <MyHeader />
      <MyLoginForm
        onSuccess={() => {
          navigate("/dashboard");
        }}
      />
      <MyFooter />
    </>
  );
}
```


***
## Handling data fetching

Let's take a basic scenario where we have to retrieve data from a distant API, how do we handle the situation.
An important thing to understand is that fetching data (with Axios or `fetch`) is simple, the difficult part is the **asynchronous state management**.
	*Mostly because of edge cases, race conditions, how to handle the rendering of the page while the data is loading, if an error is thrown, how to differentiate a situation of "no data yet" with "no data at all" considering the initial state...*

A first naive and boilerplate approach—which is still worth mentioning and understanding—is to rely on the `useEffect` hook which lets you synchronise a component with an external system.
You use your fetch/Axios method inside of it to fetch the data, and then manage the state once its collected:
```tsx
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);
  
  useEffect(() => {
    async function startFetching() {
      setBio(null);
      const result = await fetchBio(person);
      if (!ignore) {
        setBio(result);
      }
    }

    let ignore = false;
    startFetching();
    return () => {
      ignore = true;
    }
  }, [person]);

  return (
    <>
      <select value={person} onChange={e => {
        setPerson(e.target.value);
      }}>
        <option value="Alice">Alice</option>
        <option value="Bob">Bob</option>
        <option value="Taylor">Taylor</option>
      </select>
      <hr />
      <p><i>{bio ?? 'Loading...'}</i></p>
    </>
  );
}
```
> [!caution] Why is it something to avoid doing?
> This is far from being efficient and ergonomic, and it forces you to do things manually in a very verbose and repetitive way for each data you collect. It also misses very important features we expect from data fetching, the major ones being caching and background re-fetching.
> Furthermore, this kind of simplistic pattern will often end up in **“network waterfalls”**: A component fetches data, then loads a child component which also fetches data, etc. We end up loading data sequentially, one by one, which is extremely slow and clearly visible on the rendering.

As we can see, `useEffect` is not a good solution. A more convenient and performant **async state manager** is [React Query](https://tanstack.com/query/latest/docs/framework/react/overview), as it provides all features we need through a simplistic syntax.
	*It is worth mentioning that React-Router also provides a simple [data loading feature](https://beta.reactrouter.com/6.28.0/start/overview#data-loading)which might come handy in simple situations*
```bash
npm i react-query
```

Again, this is not about fetching data but about handling the overall situation, so you still need your fetching function (usually in a "service" folder/file).

> [!example]- Comparing `useEffect` to `useQuery` on the exact same situation
> `useEffect`, mind the usage of this annoying `ignore` variable for our cleanup function
> ```jsx
> function Bookmarks({ category }) {
>   const [isLoading, setIsLoading] = useState(true)
>   const [data, setData] = useState()
>   const [error, setError] = useState()
> 
>   useEffect(() => {
>     let ignore = false
>     setIsLoading(true)
>     fetch(`${endpoint}/${category}`)
>       .then(res => {
>         if (!res.ok) {
>           throw new Error('Failed to fetch')
>         }
>         return res.json()
>       })
>       .then(d => {
>         if (!ignore) {
>           setData(d)
>           setError(undefined)
>         }
>       })
>       .catch(e => {
>         if (!ignore) {
>           setError(e)
>           setData(undefined)
>         }
>       })
>       .finally(() => {
>         if (!ignore) {
>           setIsLoading(false)
>         }
>       })
>       return () => {
>         ignore = true
>       }
>   }, [category])
> }
> ```
> 
> Same thing with `useQuery`:
> ```jsx
> function Bookmarks({ category }) {
>   const { isLoading, data, error } = useQuery({
>     queryKey: ['bookmarks', category],
>     queryFn: () =>
>       fetch(`${endpoint}/${category}`).then((res) => {
>         if (!res.ok) {
>           throw new Error('Failed to fetch')
>         }
>         return res.json()
>       }),
>   })
> }
> ```
> A Very important thing to note here is that `useQuery` does not introduce any overwhelming situation (empty states, multiple fetches of strict mode, complex loading/error states) or bugs (race condition).

The library comes with very useful features and ensure a safe management of our fetched data. We can even perfection the example above by adding the cancellation feature to it:
```jsx
function Bookmarks({ category }) {
  const { isLoading, data, error } = useQuery({
    queryKey: ['bookmarks', category],
    queryFn: ({ signal }) =>
      fetch(`${endpoint}/${category}`, { signal }).then((res) => {
        if (!res.ok) {
          throw new Error('Failed to fetch')
        }
        return res.json()
      }),
  })
}
```


***
## Memoisation and React compiler

We begin with this basic code:
```jsx
export default function MyApp() {
  return <div className="foo">Hello World</div>;
}
```

Let's check the difference between what the traditional transpiler and the compiler outputs:
```jsx
// Transpiler
export default function MyApp() {
	return __jsx("div" {
		className: "foo"
	}, "Hello World");
}

// Compiler
import { unstable_useMemoCache as useMemoCache } from "react/compiler-runtime";

export default function MyApp() {
  const $ = useMemoCache(1);
  let t0;
  if ($[0] === Symbol.for("react.memo_cache_sentinel")) {
    t0 = <div className="foo">Hello World</div>;
    $[0] = t0;
  } else {
    t0 = $[0];
  }
  return t0;
}
```

We see that the compiler brings in the `useMemoCache` hook to cache elements inside of an array (of size 1 in this case).
	*We see that our component (`t0`) is cached in the first slot of the cache array.*
So, the next time this `MyApp` function is called, the hook will rely on the memoisation. This allows to cache an entire tree which can be arbitrarily complex (in this case it's only a div, but in a more fleshed-out scenario this div would contain other components, et cetera).

Furthermore, if the compiler spots unchanging variables in the code, it will actively refactor the code by converting it to its final value directly (similarly to a JIT optimisation):
```jsx
// Original code with useless constant
export default function MyApp() {
  const name = "Bob"
  return <div className="foo">
    <p>Hi {name}</p>
    <strong>Hello World</strong>
  </div>;
}

// Outputed Js after compilation
import { c as _c } from "react/compiler-runtime";
export default function MyApp() {
  const $ = _c(1);
  let t0;
  if ($[0] === Symbol.for("react.memo_cache_sentinel")) {
    t0 = (
      <div className="foo">
        <p>Hi {"Bob"}</p> {/* Optimised */}
        <strong>Hello World</strong>
      </div>
    );
    $[0] = t0;
  } else {
    t0 = $[0];
  }
  return t0;
}
```
> [!info]
> It takes the AST given by babel, re-configure it and generates a new optimised AST on the fly.


***
## Custom hooks

What if you want to re-use a feature (e.g., check active connectivity) on several components, just like you would use `useState` and `useEffect`.
You may define your own hooks, allowing several components to share **a stateful logic, not a state**: 
```tsx
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}

// Re-usable code through custom hook
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```
> [!info]
> Both components will update on their own, they don't share the same `isOnline`.
> Convention is to begin hook names with `use`, as it helps to differentiate those from standard functions.

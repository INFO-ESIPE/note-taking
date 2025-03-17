[[Web And Multimedia Technologies]]
***
**Table of Contents**
```table-of-contents
```

****
## React


***
## State

### Share state

If two components must change together, remove state from both of them, move it to their closest common parent, and then pass it down to them via props. This is known as **lifting state up**, which is an extremely common pattern in React.


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


***
## Refs


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

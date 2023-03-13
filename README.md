# My React Router v6 Learning Notes

## Outlet (Nested Routes)

If we want to display certain components on multiple pages but extend content.

Very similar approach like react `{children}`.

### Usage example

> index.jsx > App()

```jsx
<BrowserRouter>
  <Routes>
    <Route element={<Layout />}>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/vans" element={<VansList />} />
      <Route path="/vans" element={<VansList />} />
      <Route path="vans/:id" element={<VanDetail />} />
    </Route>
  </Routes>
</BrowserRouter>
```

> Layout.jsx

```jsx
const Layout = () => {
  return (
    <>
      <Header />
      <Outlet /> // functions like react children
    </>
  );
};
```

In this example every component inside `<Route element={<Layout />}>` route will display `Header` component and will be extended by content of child route.

So for example on path `"/"` app will display `Home` component and `Header` component above.

### Outlet Context

Often parent routes manage state or other values you want shared with child routes. You can create your own [context provider](https://reactjs.org/docs/context.html) if you like, but this is such a common situation that it's built-into `<Outlet />`:

```jsx
<Route path="vans/:id" element={<HostVanDetail />}>
  <Route index element={<HostVanInfo />} />
  <Route path="pricing" element={<HostVanPricing />} />
  <Route path="photos" element={<HostVanPhotos />} />
</Route>
```

> parent (HostVanDetail)

```jsx
... fetching code etc
return (
 <Outlet context={{ currentVan }} />
)

```

> child (HostVanPhotos)

```jsx
import { useOutletContext } from "react-router-dom";

export default function HostVanPhotos() {
  const { currentVan } = useOutletContext();
  return <img src={currentVan.imageUrl} className="host-van-detail-image" />;
}
```

## Relative and Non-Relative paths

### Relative

```jsx
<Route path="host" element={<HostLayout />}>
  <Route index element={<Dashboard />} />
  <Route path="income" element={<Income />} />
  <Route path="reviews" element={<Reviews />} />
</Route>
```

We are omitting the /home because due to fact that `Dashboard` `Income` and `Reviews` component `Route`s are inside `HostLayout` they somewhat extend the parents path which is host.

It can lead to less repetitive code if we have a lot of nested routes.

### Non-Relative (Absolute)

```jsx
<Route path="/host" element={<HostLayout />}>
  <Route path="/host" element={<Dashboard />} />
  <Route path="/host/income" element={<Income />} />
  <Route path="/host/reviews" element={<Reviews />} />
</Route>
```

### What If we want to display straight on parents path? || Index Routes

Due the fact that we cant use path="/" in Relative approach, because it would mean that we are referring to absolute path, we can point our, in this example `Dashboard` component's path as `index`.

I understand it as a somewhat alternative to path='' but cleaner and nicer solution :)

```jsx
<Route path="host" element={<HostLayout />}>
  // index component
  <Route index element={<Dashboard />} />
  <Route path="income" element={<Income />} />
  <Route path="reviews" element={<Reviews />} />
</Route>
```

> It will render `Dashboard` component on path '/host' in our browser.

### NavLink isActive

NavLink allows us to access the `isActive` property that's make it much easier to implement functionality for highlighting or just changing CSS of currently displayed component and reflecting it in our Nav.

> Header.jsx

```jsx
<NavLink
  className={({ isActive }) => (isActive ? "link--is-active" : null)}
  to="/vans"
>
  Vans
</NavLink>
```

> index.css

```css
.link--is-active {
  font-weight: bold;
  font-size: 105%;
  color: #303030;
}
```

### NavLink 's **`end`** property.

The `end` prop changes the matching logic for the `active` and `pending` states to only match to the "end" of the NavLinks's `to` path. If the URL is longer than `to`, it will no longer be considered active.

Without the end prop, this link is always active because every URL matches `/`.

```jsx
<NavLink to="/">Home</NavLink>
```

To match the URL "to the end" of `to`, use `end`:

```jsx
<NavLink to="/" end>
  Home
</NavLink>
```

Now this link will only be active at `"/"`.

## Relative Links

Alternatively to relative and absolute paths, we can also use **Relative** Links.

> index.jsx

```jsx
<Route path="host" element={<HostLayout />}>
  <Route index element={<Dashboard />} />
  <Route path="income" element={<Income />} />
  <Route path="reviews" element={<Reviews />} />
  <Route path="vans" element={<HostVans />} />
  <Route path="vans/:id" element={<HostVanDetail />} />
</Route>
```

If we want to use `Link` in one of our children components we dont have to use absolute path like this

```jsx
<Link to={`host/vans/${id}`}> Go to details </Link>
```

Instead of this, we can make usage of relativity to the parents path.

So If we are rendering a `Link` inside parent Route which path is for example "/host", we can just write `to` property like this

```jsx
<Link to={`vans/${id}`}> Go to details </Link>
```

Which will result in the same as code above.

**`'.'`** value inside `to` property will mean that we want to go to our parent's route path. In this exact example it will navigate us to the /`host` path

You can think of this like moving through directories in terminal where

- cd `.` is `move to the current directory`

- cd `..` is `move to the parent directory`

> HostLayout.jsx (Navbar)

```jsx
 <nav className="host-nav">
                <NavLink
                    to="." // /host
                    end
                    style={({ isActive }) => isActive ? activeStyles : null}
                >
                    Dashboard
                </NavLink>

                <NavLink
                    to="income" // /host/income
                    style={({ isActive }) => isActive ? activeStyles : null}
                >
                    Income
                </NavLink>

                <NavLink
                    to="vans" // /host/vans
                    style={({ isActive }) => isActive ? activeStyles : null}
                >
                    Vans
                </NavLink>

                <NavLink
                    to="reviews" //host/reviews
                    style={({ isActive }) => isActive ? activeStyles : null}
                >
                    Reviews
                </NavLink>

            </nav>
            <Outlet />
        </>
```

### Using `..` in nested routes

```jsx
<Route path="host" element={<HostLayout />}>
  <Route index element={<Dashboard />} />
  <Route path="income" element={<Income />} />
  <Route path="reviews" element={<Reviews />} />
  <Route path="vans" element={<HostVans />} />
  <Route path="vans/:id" element={<HostVanDetail />} />
</Route>
```

In scenario where we are rendering `HostVanDetails` which is under

path**`'/host/vans/13'`**, If we would use `Link` with `to=".."` we wouldn't be

navigated to `host/vans` as we could think.

Instead of we would be navigated to /`host` because this is the **parent path**

to achieve situation where clicking a `Link` which navigates us back to the list of vans component which is displayed under `/host/vans` path, we will have to specify `relative` property inside `Link` to **`"path"`** like this:

```jsx
<Link to=".." relative="path">
  Back to the list
</Link>
```

## Query Parameters

- Represent a change in the UI

  - Sorting, filtering, pagination

- Used as a "single source of truth" for certain application state

  - Ask yourself "**Should a user be able to revisit or share this page just like it is?**" If yes, then you might consider raising that state up to the URL in a query parameter

- Key/value paris in the URL

- Begins with **`?`**

  - `vans?type=rugged`
  - type is obj key and rugged is obj value

- Separated by **`&`**

  - vans?type=rugged&filterBy=price

### useSearchParams

The `useSearchParams` hook is used to read and modify the query string in the URL for the current location. Like React's own [`useState` hook](https://reactjs.org/docs/hooks-reference.html#usestate), `useSearchParams` returns an array of two values: the current location's [search params](https://developer.mozilla.org/en-US/docs/Web/API/URL/searchParams) and a function that may be used to update them.

**`url: /vans?type=blahblahblah`**

```jsx
const [searchParams, setSearchParams] = useSearchParams();

const typeFilter = searchParams.get("type");

console.log(typeFilter);
// blahblahblah
```

#### Example usage (Filter list)

```jsx
...

const typeFilter = searchParams.get("type")


const displayedVans = typeFilter
        ? vans.filter(van => van.type === typeFilter)
        : vans

const vanElements = displayedVans.map(van => ... )

return (
	<>
	<div className="van-list-filter-buttons">
		<Link className="van-type simple" to="?type=simple"> simple </Link>
        <Link className="van-type luxury" to="?type=luxury"> luxury </Link>
        <Link className="van-type rugged" to="?type=rugged"> rugged </Link>
        <Link className="van-type clear-filters" to="."> clear </Link>
	</div>

	<div className="van-list">
		{vanElements}
	</div>
	</>
)
```

#### More convincing method of updating search params

> Using setSearchParams allows us for more complex and flexible params.

```jsx
<div className="van-list-filter-buttons">
  <button onClick={() => setSearchParams({ type: "simple" })}> simple </button>
  <button onClick={() => setSearchParams({ type: "luxury" })}> luxury </button>
  <button onClick={() => setSearchParams({ type: "rugged" })}> rugged </button>
  <button onClick={() => setSearchParams({})}> clear </button>
</div>
```

Using **Links** is only for very simple scenarios and is rather **discouraged**

### `Link`'s state

The `state` property can be used to set a stateful value for the new location which is stored inside [history state](https://developer.mozilla.org/en-US/docs/Web/API/History/state). This value can subsequently be accessed via `useLocation()`.

```jsx
<Link to={van.id} state={{ search: searchParams.toString() }}>
```

```jsx
import {useLocation} from "react-router-dom"
...

const location = useLocation();

console.log(location)
// {pathname: "/vans/5", search: "", hash: "",
// state: {search: "type=luxury"}, key: "emy8w7js"}

```

#### Example of navigating Back with maintaining filters

```jsx
const search = location.state?.search || "";

return (
  <Link to={`..${search}`} path="relative" className="back-button">
    Back to all vans
  </Link>
);
```

## Happy Path and Sad Path

- **Happy Path**
  - Assumes everything goes according to plan exactly as we hope it does
  - Doesn't account for errors or other problems that could occur
- **Sad Path**
  - Forces us to imagine what could go wrong and plan accordingly
  - Error handling, loading states, form validation, empty state etc.

### 404 Error Page

Basically `catch-all` route is using `*` as a path.
In React-Router-V6 we don't have to worry about index of error page anymore.

Due the fact that `ErrorPage` Route is a children of a `Layout` component, we still will be able to see navbar and footer instead of blank page.

```jsx
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Layout />}>
      <Route index element={<Home />} />
      <Route path="about" element={<About />} />
      <Route path="vans" element={<Vans />} />
      <Route path="vans/:id" element={<VanDetail />} />
      <Route path="host" element={<HostLayout />}>
        <Route index element={<Dashboard />} />
        <Route path="income" element={<Income />} />
        <Route path="reviews" element={<Reviews />} />
        <Route path="vans" element={<HostVans />} />
        <Route path="vans/:id" element={<HostVanDetail />}>
          <Route index element={<HostVanInfo />} />
          <Route path="pricing" element={<HostVanPricing />} />
          <Route path="photos" element={<HostVanPhotos />} />
        </Route>
      </Route>
      <Route path="*" element={<ErrorPage />} /> // HERE IS CATCH-ALL ROUTE
    </Route>
  </Routes>
</BrowserRouter>
```

### Loaders && Errors

#### createBrowserRouter

React-Router under the hood is transforming **Route** components into plain _JS_ objects

so

```jsx
<Routes>
  <Route path="/" element={<HomePage />} />
</Routes>
```

becomes an array of objects

```jsx
[
  {
    path: "/",
    element: <HomePage />,
    children: [{}],
  },
];
```

If we want to make use of **`Data Layers`** api we have to transform our Route components into **browserRouter**. Fortunately there is a function `createRoutesFromElements` which does exactly that what you think :)

```jsx
import {
  createRoutesFromElements,
  createBrowserRouter,
} from "react-router-dom";

const router = createBrowserRouter(
  createRoutesFromElements(<Route path="/" element={<HomePage />} />)
);
```

So basically we have 2 options now when we are creating the **Route** we can either

- go old fashioned way with _element_ Route and then use **createRoutesFromElements**
- go straight into writing plain JavaScript objects manually.

**Example of transferring element routes and implementing browserRouter**

> Index.jsx without browserRouter

```jsx
import ...
import {
BrowserRouter,
Route,
Link,
RouterProvider,
createBrowserRouter,
createRoutesFromElements } from "react-router-dom"

function App() {
  return (
    <BrowserRouter>
      <Routes>

        <Route path="/" element={<Layout />}>
          <Route index element={<Home />} />
          <Route path="about" element={<About />} />
          <Route path="vans" element={<Vans />} />
          <Route path="vans/:id" element={<VanDetail />} />

          <Route path="host" element={<HostLayout />}>
            <Route index element={<Dashboard />} />
            <Route path="income" element={<Income />} />
            <Route path="reviews" element={<Reviews />} />
            <Route path="vans" element={<HostVans />} />
            <Route path="vans/:id" element={<HostVanDetail />}>
              <Route index element={<HostVanInfo />} />
              <Route path="pricing" element={<HostVanPricing />} />
              <Route path="photos" element={<HostVanPhotos />} />
            </Route>
          </Route>
          <Route path="*" element={<NotFound />}/>
        </Route>
      </Routes>
    </BrowserRouter>
  )
}

ReactDOM
  .createRoot(document.getElementById('root'))
  .render(<App />);
```

> Index.jsx with browserRouter

```jsx
import ...
import {
BrowserRouter,
Route,
Link,
RouterProvider,
createBrowserRouter,
createRoutesFromElements } from "react-router-dom"

const router = createBrowserRouter(createRoutesFromElements(
	<Route path="/" element={<Layout />}>
          <Route index element={<Home />} />
          <Route path="about" element={<About />} />
          <Route path="vans" element={<Vans />} />
          <Route path="vans/:id" element={<VanDetail />} />

          <Route path="host" element={<HostLayout />}>
                <Route index element={<Dashboard />} />
                <Route path="income" element={<Income />} />
                <Route path="reviews" element={<Reviews />} />
                <Route path="vans" element={<HostVans />} />
                <Route path="vans/:id" element={<HostVanDetail />}>
                  <Route index element={<HostVanInfo />} />
                  <Route path="pricing" element={<HostVanPricing />} />
                  <Route path="photos" element={<HostVanPhotos />} />
                </Route>
          </Route>

          <Route path="*" element={<NotFound />}/>

    </Route>
))

function App() {
  return (
   <RouterProvider router={router} />
  )
}

ReactDOM
  .createRoot(document.getElementById('root'))
  .render(<App />);
```

## Loaders

**TL:DR benefit: inside a component where you would do fetching in classic react way, you can behave like data is always already there.**

- Export a **_loader_** function from the page that fetches the data that page will need.
- Pass a **_loader_** prop to the Route that renders that page and pass in the **_loader_** function
- Use the **_useLoaderData_** hook in the component to get the data

> api.js

```jsx
export async function getVans() {
  const res = await fetch("/api/vans");
  if (!res.ok) {
    throw {
      message: "Failed to fetch vans",
      statusText: res.statusText,
      status: res.status,
    };
  }
  const data = await res.json();
  return data.vans;
}
```

> index.jsx

```jsx
import Vans, { loader as vansLoader } from "./pages/Vans/Vans"

//inside of createRoutesFromElements

...
 <Route path="vans" element={<Vans />} loader={vansLoader} />
...

```

> Vans.jsx

```js
import { Link, useSearchParams, useLoaderData } from "react-router-dom";
import { getVans } from "../../api";

export function loader() {
  return getVans();
}

// Rest of the code
```

**OKAY BUT WHAT ALL OF THIS DOES?**

now fetching the data doesn't take place like it always used to be in useEffect

**loader** function is being called when user wants to navigate to path which `Route` has `loader` parameter. So for example when clicking a NavLink to `/about`

now when user which is for example on the `/home` route wants to navigate to the `/vans` via clicking on the correct `NavLink`, fetching will start to take place and **Vans** component **WILL RENDER AFTER FETCHING WILL COMPLETE AND EVERYTHING WILL BE READY TO ACTUALLY RENDER THE COMPONENT**

> okay but what with UX? check this [Deferring data](#deferring-data)

so basically: no error handling, no loading state, no useEffect and all of that inside actual component which is **HUGE**.

#### Handling an error

Among new features that can be now passed inside Route (Object or Element :) )

we can pass **`errorElement`** if actual `element` throws **any** error.

> index.jsx

```jsx
<Route
  path="vans"
  element={<Vans />}
  loader={vansLoader}
  errorElement={<ErrorDisplayer />} //errorElement which catches errors
/>
```

#### Displaying more informant error messages

> api.js

```jsx
export async function getVans() {
  const res = await fetch("/api/vans");
  if (!res.ok) {
    throw {
      message: "Failed to fetch vans",
      statusText: res.statusText,
      status: res.status,
    };
  }
  const data = await res.json();
  return data.vans;
}
```

> ErrorDisplayer.jsx

```jsx
import { useRouteError } from "react-router-dom";

export default function Error() {
  const error = useRouteError();
  console.log(error);

  return (
    <>
      <h1> Error: {error.message} </h1>
      <pre>
        {" "}
        {error.status} - {error.statusText}{" "}
      </pre>
    </>
  );
}

// {message: "Failed to fetch vans", statusText: "Bad Request", status: 400}
```

#### Catching Errors in Nested Routes

```jsx
<Route path="/" element={<Layout />} errorElement={<Error />}>
  {" "}
  // ERROR ELEMENT HERE
  <Route index element={<Home />} />
  <Route path="about" element={<About />} />
  <Route path="vans" element={<Vans />} loader={vansLoader} />
  <Route path="vans/:id" element={<VanDetail />} />
  <Route path="host" element={<HostLayout />}>
    <Route index element={<Dashboard />} />
    <Route path="income" element={<Income />} />
    <Route path="reviews" element={<Reviews />} />
    <Route path="vans" element={<HostVans />} />
    <Route path="vans/:id" element={<HostVanDetail />}>
      <Route index element={<HostVanInfo />} />
      <Route path="pricing" element={<HostVanPricing />} />
      <Route path="photos" element={<HostVanPhotos />} />
    </Route>
  </Route>
  <Route path="*" element={<NotFound />} />
</Route>
```

if we will setup our errorElement like this, it will catch every error that happens inside our App. (because errorElement is inside Layout which basically wraps our whole app).

on the other hand if we will set our errorElement down in some nested `Route` it wouldn't catch errors that happens in ancestor routes.

```jsx
<Route path="/" element={<Layout />}>
  <Route index element={<Home />} />
  <Route path="about" element={<About />} />
  <Route path="vans" element={<Vans />} loader={vansLoader} />
  <Route path="vans/:id" element={<VanDetail />} />

  <Route path="host" element={<HostLayout />}>
    <Route index element={<Dashboard />} />
    <Route path="income" element={<Income />} />
    <Route path="reviews" element={<Reviews />} />
    <Route path="vans" element={<HostVans />} errorElement={<Error />} /> // ERROR
    ELEMENT HERE
    <Route path="vans/:id" element={<HostVanDetail />}>
      <Route index element={<HostVanInfo />} />
      <Route path="pricing" element={<HostVanPricing />} />
      <Route path="photos" element={<HostVanPhotos />} />
    </Route>
  </Route>
  <Route path="*" element={<NotFound />} />
</Route>
```

> In situation like this errorElement **wont** catch error which occurred inside `Vans` component

## Actions and Protected Routes

very simplified example of implementation of protected routes

> AuthRequired.jsx

```jsx
import { Outlet, Navigate } from "react-router-dom";

const AuthRequired = () => {
  const token = { accessToken: "123" };

  return (
    <>
      {token.accessToken ? <Outlet /> : <Navigate to="/login" />} // Navigate
      will force redirect to /login page
    </>
  );
};

export default AuthRequired;
```

> index.jsx

```jsx
<Route element={<AuthRequired /> } /> // AuthRequired function will get called every time one of children routes will get called
<Route path="host" element={<HostLayout />}>
      <Route index element={<Dashboard />} />
      <Route path="income" element={<Income />} />
      <Route path="reviews" element={<Reviews />} />
      <Route path="vans" element={<HostVans />} />
      <Route path="vans/:id" element={<HostVanDetail />}>
            <Route index element={<HostVanInfo />} />
            <Route path="pricing" element={<HostVanPricing />} />
            <Route path="photos" element={<HostVanPhotos />} />
      </Route>
</Route>
```

Inside `Navigate` element we can pass state object just like in Link element.
we can use it to eventually **restore the path** from which user was redirected to login page or display the message why user was redirected to login page.

```jsx
<Navigate to="/login" state={{ message: "You must log in first." }} />
```

> Login.jsx

```jsx
...
import { useLocation } from "react-router-dom"

const location = useLocation()
console.log(location)

// IF REDIRECTED FROM NAVIGATE - {pathname: "/login", search: "", hash: "", state: {message: "You must log in first."}, key: "z9k5z5r5"} - state contains message entry

// IF user navigated to login page by himself - {pathname: "/login", search: "", hash: "", state: null, key: "default"} - state is null
```

#### replace and history stack

If user is being redirected to the login page after clicking protected route without authorization **History stack** will look like this in that exact situation

(after logging in user is being navigated to /host path again)

![image-20230312203634258](C:\Users\batko\AppData\Roaming\Typora\typora-user-images\image-20230312203634258.png)

with `replace` property inside `Navigate` element or passed as parameter inside `navigate` we can modify history stack

so for example if we will use

> Login.jsx

```jsx
const from = location?.state?.from || "/host";

navigate(from, { replace: true });
```

it will replace the `/login` inside our history stack with `/host`

It has meaning if it comes to UX which if will hit back arrow inside browser wont be navigated back to the login page but to the **_previously visited path before redirect happened_**

> Basically you can think about it that `replace` will **replace current** history stack element **with following** history stack element where you will be redirected or navigated

## Forms and Actions

The Form component is a wrapper around a plain HTML [form](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form) that emulates the browser for client side routing and data mutations.

```jsx
<Form method="post" action="/events">
  <input type="text" name="title" />
  <input type="text" name="description" />
  <button type="submit">Create</button>
</Form>
```

> Make sure your inputs have names or else the `FormData` will not include that field's value.

All of this will trigger state updates to any rendered [`useNavigation`](https://reactrouter.com/en/main/hooks/use-navigation) hooks so you can build pending indicators and optimistic UI while the async operations are in-flight.

### action

the url to which the form will be submitted, just like [HTML form action](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#attr-action). Where the native element would submit to a backend, react-router-v6 `Form` will run an `action` function instead.

> Login.jsx

```jsx
export const action = async ({ request }) => {
  console.log("form submitted, logged from action function", { request });
};
// form submitted, logged from action function, Request {url: "https://cw3.scrimba.com/login", credentials: "same-origin", headers: {map: {content-type: "application/x-www-form-urlenc..."}}, method: "POST", mode: null, signal: {}, referrer: null, bodyUsed: false, _bodyInit: {}, _bodyText: "email=blazej%40blazej.pl&pass..."}

export default function Login() {
  const navigate = useNavigate();

  return (
    <Form action="/login" method="post">
      {" "}
      // WE ARE POINTING AT WHICH RELATIVE (!!!) PATH OUR ACTION FUNCTION IS BEING
      LOCATED and method of request
      <input type="email" name="email" placeholder="Email address" />
      <br />
      <input type="password" name="password" placeholder="Password" />
      <br />
      <button>Log in</button>
    </Form>
  );
}
```

> In other words it means that **`action`** function for that Form can be found inside component that is rendered under `/login` path.

> index.jsx

```jsx
import Login, {action as loginAction } from "./Login"

... router code

createRoutesFromElements(
<Route path="/" element={<Layout />}
	<Route path="login" element={<Login />} action={loginAction} /> // we are providing an function that will be called when form which point to path /login will be submitted
</Route>
)
```

### pulling data from Form

> Login.jsx

```jsx
export const action = async ({ request }) => {
	const formData = await request.formData(); // creating formData object that is 		providing methods allowing us to menage and get data from form.
    const email = formData.get("email") // we are passing input's name parameter from 	  which we want to get submitted data.
    const password = formData.get("password")

    //process this info however we want

    console.log(email, password)

    const data = await fakeLoginUser({email, password})

    return data

}
// bl@zej.com,"qazxsw" (logged after submitting the Form)

export default const Login = () => {
    ...
}
```

### passing formData to component

`useActionData` hook provides the returned value from the previous navigation's `action` result, or `undefined` if there was no submission.

> Login.jsx

```jsx
import { useActionData } from "react-router-dom"

export const action = async ({ request }) => {
	...

	return data
}

export default const Login = () => {
    const data = useActionData()

    console.log(data)


    ...rest of Login code
}

// {token: "Here's your token!"} log action's function returned value which in that example is coming from fakeLoginUser function.
```

### handling action error

If we throw an error inside action function with `Throw`, react router will try to find errorElement among the routes like I described above in [Handling an error](#handling-an-error).

In our scenario we don't really want that to happen because that would unmount our Login.jsx component and wont provide valuable information to the user.

Instead of this, a better approach would be displaying short info above the form like "couldn't log you in" or something like this.

To achieve that we cant throw an error inside action function but we can use try{...} catch(e) {...} block and segment the code between happy-path and sad-path, sad patch will be placed inside catch block.

> Login.jsx

```jsx
export async function action({ request }) {
    const formData = await request.formData()
    const email = formData.get("email")
    const password = formData.get("password")

    try {
        const data = await fakeLoginUser({ email, password })
        return data
    } catch (err) {
        return {
            error: err.message
        }
    }
}

export default const Login = () => {

    const data = useActionData();
    ...

    return (
    	 {data?.error && <h4>{data.error}</h4>} //conditionally render error message
      	 ...rest of JSX
    )
}
```

### Difference between react controlled forms and react-router Forms

> Login.jsx without Data Layer Api 🤢🤢

```jsx
import React from "react";
import { useNavigate, useLocation } from "react-router-dom";
import { loginUser } from "../api";

export default function Login() {
  const [loginFormData, setLoginFormData] = React.useState({
    email: "",
    password: "",
  });
  const [status, setStatus] = React.useState("idle");
  const [error, setError] = React.useState(null);

  const location = useLocation();
  const navigate = useNavigate();
  const from = location.state?.from || "/host";

  function handleSubmit(e) {
    e.preventDefault();
    setStatus("submitting");
    setError(null);
    loginUser(loginFormData)
      .then((data) => {
        localStorage.setItem("loggedin", true);
        navigate(from, { replace: true });
      })
      .catch((err) => {
        setError(err);
      })
      .finally(() => {
        setStatus("idle");
      });
  }

  function handleChange(e) {
    const { name, value } = e.target;
    setLoginFormData((prev) => ({
      ...prev,
      [name]: value,
    }));
  }

  return (
    <div className="login-container">
      {location.state?.message && (
        <h3 className="login-error">{location.state.message}</h3>
      )}
      <h1>Sign in to your account</h1>
      {error && <h3 className="login-error">{error.message}</h3>}
      <form onSubmit={handleSubmit} className="login-form">
        <input
          name="email"
          onChange={handleChange}
          type="email"
          placeholder="Email address"
          value={loginFormData.email}
        />
        <input
          name="password"
          onChange={handleChange}
          type="password"
          placeholder="Password"
          value={loginFormData.password}
        />
        <button disabled={status === "submitting"}>
          {status === "submitting" ? "Logging in..." : "Log in"}
        </button>
      </form>
    </div>
  );
}
```

> Login.jsx with data layer api 😎😎

```jsx
import React from "react";
import {
  useNavigate,
  useNavigation,
  useLocation,
  useActionData,
  Form,
} from "react-router-dom";
import { loginUser } from "../api";

export async function action({ request }) {
  const formData = await request.formData();
  const email = formData.get("email");
  const password = formData.get("password");

  try {
    const data = await loginUser({ email, password });
    localStorage.setItem("loggedin", true);
    return data;
  } catch (err) {
    return {
      error: err.message,
    };
  }
}

export default function Login() {
  const data = useActionData();
  const location = useLocation();
  const navigate = useNavigate();
  const navigation = useNavigation();
  const from = location.state?.from || "/host";

  React.useEffect(() => {
    if (data?.token) {
      navigate(from, { replace: true });
    }
  }, [data]);

  return (
    <div className="login-container">
      {location.state?.message && (
        <h3 className="login-error">{location.state.message}</h3>
      )}
      <h1>Sign in to your account</h1>
      {data?.error && <h3 className="login-error">{data.error}</h3>}
      <Form action="/login" method="post" className="login-form">
        <input name="email" type="email" placeholder="Email address" />
        <input name="password" type="password" placeholder="Password" />
        <button disabled={navigation.state === "submitting"}>
          {navigation.state === "submitting" ? "Logging in..." : "Log in"}
        </button>
      </Form>
    </div>
  );
}
```

## Handling loading state with useNavigation (!== useNavigate)

when using `loaders` and `actions` it may be harming for user experience.
for example when user wants to see details about some product and clicks a tile which navigates him to product detail page, with **`loaders`** there will be a delay before page will show up because there will be data fetching from the server behind. (loaders will make it that product detail page will render after data is ready because it changes **WHEN** fetching takes place).

Fortunately **`useNavigation`** hook is a tool that can help us to gather information about current status of navigating through our app.

> Login.jsx

```jsx
import { useNavigation } from "react-router-dom"

export default const Login = () => {
	const navigation = useNavigation()

	console.log(navigation)

// {state: "idle", location: undefined, formMethod: undefined, formAction: undefined, formEncType: undefined, formData: undefined}

after clicking submit button inside form:

// {state: "submitting", location: {pathname: "/login", search: "", hash: "", state: null, key: "2t7r0wya"}, formMethod: "post", formAction: "/login", formEncType: "application/x-www-form-urlencoded", formData: FormData {}}

// {state: "idle", location: undefined, formMethod: undefined, formAction: undefined, formEncType: undefined, formData: undefined}

we can use navigation.state to let user know what is currently happening with his request.

<Form action="/login" method="post">
    <input
        type="email"
        name="email"
        placeholder="Email address"
    />
    <br />
    <input
        type="password"
        name="password"
        placeholder="Password"
    />
    <br />
    <button disabled={isSubmitting}>{isSubmitting ? "Logging in..." : "Log in"}			</button> //HERE WE ARE DOING CONDITIONAL RENDERING DEPENDING ON THAT STATE
</Form>
}

```

## Deferring data

### Promises and defer()

> Note: when a function is an async function it is always returning a PROMISE
>
> so it is an indication that
>
> ```jsx
> const weather = await getWeather();
> ```
>
> getWeather() returns a promise because await can ONLY be used on a function that returns promise.

#### defer

> you can think about `defer` that it means "don't wait for this data to load"

> Vans.jsx

```jsx
import { defer } from "react-router-dom"
import { getVans } from "../../api"
export functin loader() {

	return defer({vans: getVans()})

	// defer takes an object representing any data you want to have access to in the 	component.
	// the value of the object property should be a promise

}
```

#### Await Element

> Await is used to render deferred values with automatic error handling.

Await has a `resolve` parameter where we are passing data that we ill need to eventually render in our component.

> Vans.jsx

```jsx
export default Vans = () => {
  const dataPromise = useLoaderData(); // returned content from loader function 		above.

  return (
    <Await resolve={dataPromise.vans}>
      {(vans) => {
        vans.map((van) => <h1>{van.name}</h1>);
      }}
    </Await>
  );
};
```

![image-20230313132856031](C:\Users\batko\AppData\Roaming\Typora\typora-user-images\image-20230313132856031.png)

> promise we are passing to resolve prop - from react-router docs

#### Await Children

Can either be React elements or a function.

When using a function, the value is provided as the only parameter.

```jsx
<Await resolve={reviewsPromise}>
  {(resolvedReviews) => <Reviews items={resolvedReviews} />}
</Await>
```

When using React elements, [`useAsyncValue`](https://reactrouter.com/en/main/hooks/use-async-value) will provide the data:

```jsx
<Await resolve={reviewsPromise}>
  <Reviews />
</Await>;

function Reviews() {
  const resolvedReviews = useAsyncValue();
  return <div>{/* ... */}</div>;
}
```

#### React Suspense

> <Await> expects to be rendered inside of a <React.Suspense> parent to enable the fallback UI.

> Vans.jsx

```jsx
return (
  <div className="van-list-container">
    <h1>Explore our van options</h1>
    <React.Suspense fallback={<h1>Loading vans data</h1>}>
      <Await resolve={dataPromise.vans}>{renderVanElements}</Await>
    </React.Suspense>
  </div>
);
```

> When Await will wait for promise to resolve and actually get the vans data,
> "Loading vans data" will be displayed in the UI.

### Whole example of using loader, defer, Await and Suspense

> HostVans.jsx without react-router-v6 new data layer api

```jsx
import React from "react";
import { Link } from "react-router-dom";

export default function HostVans() {
  const [vans, setVans] = React.useState([]);

  React.useEffect(() => {
    fetch("/api/host/vans")
      .then((res) => res.json())
      .then((data) => setVans(data.vans));
  }, []);

  const hostVansEls = vans.map((van) => (
    <Link to={van.id} key={van.id} className="host-van-link-wrapper">
      <div className="host-van-single" key={van.id}>
        <img src={van.imageUrl} alt={`Photo of ${van.name}`} />
        <div className="host-van-info">
          <h3>{van.name}</h3>
          <p>${van.price}/day</p>
        </div>
      </div>
    </Link>
  ));

  return (
    <section>
      <h1 className="host-vans-title">Your listed vans</h1>
      <div className="host-vans-list">
        {vans.length > 0 ? (
          <section>{hostVansEls}</section>
        ) : (
          <h2>Loading...</h2>
        )}
      </div>
    </section>
  );
}
```

​

> HostVans.jsx with usage of data layer api

```jsx
import React from "react";
import { Link, useLoaderData, defer, Await } from "react-router-dom";
import { getHostVans } from "../../api";

export const loader = () => {
  return defer({ hostVans: getHostVans() });
};

export default function HostVans() {
  const hostVansPromise = useLoaderData();

  function renderHostVans(hostVans) {
    return hostVans.map((van) => {
      return (
        <Link to={van.id} key={van.id} className="host-van-link-wrapper">
          <div className="host-van-single" key={van.id}>
            <img src={van.imageUrl} alt={`Photo of ${van.name}`} />
            <div className="host-van-info">
              <h3>{van.name}</h3>
              <p>${van.price}/day</p>
            </div>
          </div>
        </Link>
      );
    });
  }

  return (
    <section>
      <h1 className="host-vans-title">Your listed vans</h1>
      <div className="host-vans-list">
        <section>
          <React.Suspense fallback={<h1>Loading </h1>}>
            <Await resolve={hostVansPromise.hostVans}>{renderHostVans}</Await>
          </React.Suspense>
        </section>
      </div>
    </section>
  );
}
```

> index.jsx (we have to specify loader for that `Route`)

```jsx
import HostVans, { loader as hostVansLoader} from "./pages/Host/HostVans"

...

<Route path="vans" element={<HostVans />} loader={hostVansLoader} />

...

```

> api.js (getHostVans fetching function)

```jsx
export async function getHostVans() {
  await sleep(1000); // fake making response longer so suspense fallback is visible
  const res = await fetch("/api/host/vans");
  if (!res.ok) {
    throw {
      message: "Failed to fetch vans",
      statusText: res.statusText,
      status: res.status,
    };
  }
  const data = await res.json();
  return data.vans;
}
```

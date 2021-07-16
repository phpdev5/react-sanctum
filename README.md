# react-sanctum - React components for Laravel Sanctum

[![npm version](https://badge.fury.io/js/react-sanctum.svg)](https://www.npmjs.com/package/react-sanctum)

The react-sanctum package aims to provide an easy way to authenticate your React application with Laravel Sanctum.

## Usage

Install from NPM

```
npm i react-sanctum
```

Wrap your application in an `<Sanctum>` component

Example:

```js
import React from "react";

import { Sanctum } from "react-sanctum";

const sanctumConfig = {
    apiUrl: "http://foobar.test",
    csrfCookieRoute: "sanctum/csrf-cookie",
    signInRoute: "login",
    signOutRoute: "logout",
    userObjectRoute: "user",
};

const App = () => (
    <div className="my-application">
        <Sanctum config={sanctumConfig}>/* Your application code */</Sanctum>
    </div>
);
```

You can then use the `withSanctum` higher-order component to get authentication status, user data and sanctum related
methods in any component.

```js
import React from "react";
import { withSanctum } from "react-sanctum";

const LoginButton = ({authenticated, user, signIn}) => {
    const handleLogin = () => {
        const email = "sanctum@example.org";
        const password = "example";
        const remember = true;

        signIn(email, password, remember)
            .then(() => window.alert("Signed in!"))
            .catch(() => window.alert("Incorrect email or password"));
    };

    if (authenticated === true) {
        return <h1>Welcome, {user.name}</h1>;
    } else {
        return <button onClick={handleLogin}>Sign in</button>;
    }
};

export default withSanctum(LoginButton);
```

You can also directly consume the Sanctum context by importing `SanctumContext`.

Both the `SanctumContext` and the `withSanctum` HOC give you access to the following
data and methods:
| | Description |
|-|------------------------------------------------------------------------------------|
| `user` | Object your API returns with user data |
| `authenticated` | Boolean, or null if authentication has not yet been checked |
| `signIn()` | Accepts `(email, password, remember?)`, returns a promise, resolves with the user data. |
| `signOut()` | Returns a promise |
| `setUser()` | Accepts `(user, authenticated?)`, allows you to manually set the user object and optionally its authentication status (boolean). |
| `checkAuthentication()` | Returns the authentication status. If it's null, it will ask the server and update `authenticated`. |

# Setup

All URLS in the config are required. These need to be created in your Laravel app.

```js
const sanctumConfig = {
    // Your applications URL
    apiUrl: "http://foobar.test",
    // The following settings are URLS that need to be created in your Laravel application
    // The URL sanctum uses for the csrf cookie
    csrfCookieRoute: "sanctum/csrf-cookie",
    // {email: string, password: string, remember: true | null} get POSTed to here
    signInRoute: "api/login",
    // A POST request is sent to this route to sign the user out
    signOutRoute: "api/logout",
    // Used (GET) for checking if the user is signed in (so this should be protected)
    // The returned object will be avaiable as `user` in the React components.
    userObjectRoute: "api/user",
    // An axios instance to be used by react-sanctum (optional). Useful if you for example need to add custom interceptors.
    axiosInstance: AxiosInstance,
};
```

react-sanctum automatically checks if the user is signed in when the the `<Sanctum>`
component gets mounted. If you don't want this, and want to manually use the
`checkAuthentication` function later, set `checkOnInit` to `false` like so:

```js
<Sanctum config={sanctumConfig} checkOnInit={true}>
```

# Handling registration

Methods for signIn and signOut are provided by this library. Registration is not included as there seem to be many ways
people handle registration flows.

If you want to sign in your user after registration, there's an easy way to do this. First, make sure the endpoint you
post the registration data to signs in the user (`Auth::guard()->login(...)`) and have it return the user object to the
front-end.

In your front-end you can then pass this user object into the `setUser()` function, et voilà, your new user has been
signed in.

For example:

```js
axios
    .post(`${API_URL}/register`, data)
    .then(function (response) {
        const user = response.data;
        setUser(user); // The react-sanctum setUser function
        ...
    })
    .catch(function (error) {
        ...
    });
```

# Axios

Quick tip for people using axios: react-sanctum uses Axios for making requests to your server. If your project is also
using axios, make sure to set
`axios.defaults.withCredentials = true;`. That way axios will authenticate your requests to the server properly.

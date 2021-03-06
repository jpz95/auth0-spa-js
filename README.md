# @jpz95/auth0-spa-js

Auth0 SDK for Single Page Applications using [Authorization Code Grant Flow with PKCE](https://auth0.com/docs/flows/authorization-code-flow-with-proof-key-for-code-exchange-pkce).

[![License](https://img.shields.io/:license-mit-blue.svg?style=flat)](https://opensource.org/licenses/MIT)

## Table of Contents

- [Documentation](#documentation)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [Contributing](#contributing)
- [Support + Feedback](#support--feedback)
- [Frequently Asked Questions](#frequently-asked-questions)
- [License](#license)

## API Docs

- [API reference](https://jpz95.github.io/auth0-spa-js/)

## Installation

Using [npm](https://npmjs.org):

```sh
npm install @jpz95/auth0-spa-js
```

Using [yarn](https://yarnpkg.com):

```sh
yarn add @jpz95/auth0-spa-js
```

## Getting Started

### Creating the client

Create an `Auth0Client` instance before rendering or initializing your application. You should only have one instance of the client.

```js
import createAuth0Client from '@jpz95/auth0-spa-js';

//with async/await
const auth0 = await createAuth0Client({
  domain: 'example.somesite.com', // we will prepend 'https://'
  authorizeEndpoint: '/path/to/auth', // forms https://example.somesite.com/path/to/auth
  tokenEndpoint: '/path/to/token', // forms https://example.somesite.com/path/to/token
  client_id: '<API_CLIENT_ID>',
  redirect_uri: '<MY_CALLBACK_URL>'
});

//or, you can just instantiate the client on it's own
import { Auth0Client } from '@jpz95/auth0-spa-js';

const auth0 = new Auth0Client({
  domain: 'example.somesite.com', // we will prepend 'https://'
  authorizeEndpoint: '/path/to/auth', // forms https://example.somesite.com/path/to/auth
  tokenEndpoint: '/path/to/token', // forms https://example.somesite.com/path/to/token
  client_id: '<API_CLIENT_ID>',
  redirect_uri: '<MY_CALLBACK_URL>'
});

//if you do this, you'll need to check the session yourself
try {
  await getTokenSilently();
} catch (error) {
  if (error.error !== 'login_required') {
    throw error;
  }
}
```

### 1 - Login

```html
<button id="login">Click to Login</button>
```

```js
//with async/await

//redirect to the Universal Login Page
document.getElementById('login').addEventListener('click', async () => {
  await auth0.loginWithRedirect();
});

//in your callback route (<MY_CALLBACK_URL>)
window.addEventListener('load', async () => {
  const redirectResult = await auth0.handleRedirectCallback();
  //logged in. you can get the user profile like this:
  const user = await auth0.getUser();
  console.log(user);
});
```

### 2 - Calling an API

```html
<button id="call-api">Call an API</button>
```

````js
//with async/await
document.getElementById('call-api').addEventListener('click', async () => {
  const accessToken = await auth0.getTokenSilently();
  const result = await fetch('https://myapi.com', {
    method: 'GET',
    headers: {
      Authorization: `Bearer ${accessToken}`
    }
  });
  const data = await result.json();
  console.log(data);
});


### 3 - Logout

```html
<button id="logout">Logout</button>
````

```js
import createAuth0Client from '@jpz95/auth0-spa-js';

document.getElementById('logout').addEventListener('click', () => {
  auth0.logout();
});
```

### Data caching options

The SDK can be configured to cache ID tokens and access tokens either in memory or in local storage. The default is in memory. This setting can be controlled using the `cacheLocation` option when creating the Auth0 client.

To use the in-memory mode, no additional options need are required as this is the default setting. To configure the SDK to cache data using local storage, set `cacheLocation` as follows:

```js
await createAuth0Client({
  domain: 'example.somesite.com', // we will prepend 'https://'
  authorizeEndpoint: '/path/to/auth', // forms https://example.somesite.com/path/to/auth
  tokenEndpoint: '/path/to/token', // forms https://example.somesite.com/path/to/token
  client_id: '<API_CLIENT_ID>',
  redirect_uri: '<MY_CALLBACK_URL>',
  cacheLocation: 'localstorage' // valid values are: 'memory' or 'localstorage'
});
```

**Important:** This feature will allow the caching of data **such as ID and access tokens** to be stored in local storage. Exercising this option changes the security characteristics of your application and **should not be used lightly**. Extra care should be taken to mitigate against XSS attacks and minimize the risk of tokens being stolen from local storage.

### Refresh Tokens

Refresh tokens can be used to request new access tokens. [Read more about how our refresh tokens work for browser-based applications](https://auth0.com/docs/tokens/concepts/refresh-token-rotation) to help you decide whether or not you need to use them.

To enable the use of refresh tokens, set the `useRefreshTokens` option to `true`:

```js
await createAuth0Client({
  domain: 'example.somesite.com', // we will prepend 'https://'
  authorizeEndpoint: '/path/to/auth', // forms https://example.somesite.com/path/to/auth
  tokenEndpoint: '/path/to/token', // forms https://example.somesite.com/path/to/token
  client_id: '<API_CLIENT_ID>',
  redirect_uri: '<MY_CALLBACK_URL>',
  useRefreshTokens: true
});
```

Using this setting will cause the SDK to automatically request a new refresh token from the authorization server. Refresh tokens will then be used to exchange for new access tokens instead of using a hidden iframe, and calls the token endpoint directly. This means that in most cases the SDK does not rely on third-party cookies when using refresh tokens.

#### Refresh Token fallback

In all cases where a refresh token is not available, the SDK falls back to the legacy technique of using a hidden iframe with `prompt=none` to try and get a new access token and refresh token. This scenario would occur for example if you are using the in-memory cache and you have refreshed the page. In this case, any refresh token that was stored previously would be lost.

If the fallback mechanism fails, a `login_required` error will be thrown and could be handled in order to put the user back through the authentication process.

**Note**: This fallback mechanism does still require access to the Auth0 session cookie, so if third-party cookies are being blocked then this fallback will not work and the user must re-authenticate in order to get a new refresh token.

## Support + Feedback

For support or to provide feedback, please [raise an issue on our issue tracker](https://github.com/jpz95/auth0-spa-js/issues).

## Frequently Asked Questions

For a rundown of common issues you might encounter when using the SDK, please check out [the FAQ](https://github.com/jpz95/auth0-spa-js/blob/master/FAQ.md).

## License

This project is licensed under the MIT license. See the [LICENSE](https://github.com/jpz95/auth0-spa-js/blob/master/LICENSE) file for more info.

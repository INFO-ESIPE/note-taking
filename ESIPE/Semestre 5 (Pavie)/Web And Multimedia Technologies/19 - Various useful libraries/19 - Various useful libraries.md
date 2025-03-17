[[Web And Multimedia Technologies]]
***
**Table of Contents**
```table-of-contents
```

****
## Axios

A promise-based HTTP client which can run on both Node.js (HTTP Requests) and a browser (XMLHttpRequests). More convenient than the native `fetch` API, as it offers a pleasant syntax, better error handling, automatic JSON parsing, timeouts and request cancelling, and more.
```bash
npm i axios
```

### Basic requests

HTTP requests are crafted by calling the `axios` method which define its type.
```ts
const axios = require('axios').default;

// GET requrst
axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .finally(function () {
    // Optional, like a try-finally clause which is always executed
  });

// POST request (JSON)
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

We can perform concurrent requests:
```ts
function getUserAccount() {
  return axios.get('/user/12345');
}

function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}

const [acct, perm] = await Promise.all([getUserAccount(), getUserPermissions()]);
// OR
Promise.all([getUserAccount(), getUserPermissions()])
  .then(function ([acct, perm]) {
    // ...
});
```

### Configuration

We may set default configurations for all requests using `axios.defaults`:
```ts
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = 'Bearer TOKEN';
```

On the other hand, we may create an Axios **instance** with custom configurations:
```ts
const instance = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});

instance.get('/data');
```

### Interceptor

Allows to modify request/responses before they are handled by `then` or `catch` clauses:
```ts
// Request interceptor
axios.interceptors.request.use(function (config) {
    return config; // Do something before request is sent
  }, function (error) {
    return Promise.reject(error);
  });

// Response interceptor
axios.interceptors.response.use(function (response) {
    return response;              // Triggers if code 2xx
  }, function (error) {
    return Promise.reject(error); // Triggers if anything but 2xx
  });
```

It is possible to remove the interceptor at a later time:
```ts
const myInterceptor = axios.interceptors.request.use(function () {/*...*/});
axios.interceptors.request.eject(myInterceptor);
```

### Cancellation

Setting the `timeout` property in an Axios call handles **response** related timeouts. But sometimes, we want to close the connection earlier than that (e.g., network connectivity issues), allowing to handle **connection** related timeouts.

This is done by using the `signal` method which implements the [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)under the hood.
```ts
axios.get('/foo/bar', {
   signal: new AbortController().signal
}).then(function(response) {
   // ...
});

controller.abort() // Cancels the request
```


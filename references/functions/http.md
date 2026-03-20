# HTTP functions

These functions can be used when opening and submitting remote web requests, and webhooks.

## Before you begin

From version `2.2` of SurrealDB, the HTTP functions have been improved to provide a more consistent and user-friendly experience. These improvements include:

- **Enhanced HTTP error messages**: The server provides more descriptive error responses, including relevant HTTP status codes and detailed error information when available.

- **Raw SurrealQL data encoding**: Data types are preserved more faithfully in responses through improved encoding.
  - SurrealQL **byte values** are now sent as raw bytes (not base64-encoded or JSON-encoded).  
  - SurrealQL **string values** are sent as raw strings.  
  - All other SurrealQL values (numbers, arrays, objects, booleans, etc.) are automatically JSON-encoded.

- **Manual Header Configuration**: SurrealDB no longer automatically adds `Content-Type: application/octet-stream` to responses when the body contains SurrealQL byte values. If you need this header, you can set it manually.

## `http::head`

The `http::head` function performs a remote HTTP `HEAD` request. The first parameter is the URL of the remote endpoint. If the response does not return a `2XX` status code, then the function will fail and return the error.

```surql
http::head(string) -> null
```

If an object is given as the second argument, then this can be used to set the request headers.

```surql
http::head(string, $headers: object) -> null
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN http::head('https://surrealdb.com');

null
```

To specify custom headers with the HTTP request, pass an object as the second argument:

```surql
RETURN http::head('https://surrealdb.com', {
	'x-my-header': 'some unique string'
});

null
```

## `http::get`

The `http::get` function performs a remote HTTP `GET` request. The first parameter is the URL of the remote endpoint. If the response does not return a 2XX status code, then the function will fail and return the error. 

If the remote endpoint returns an `application/json content-type`, then the response is parsed and returned as a value, otherwise the response is treated as text.

```surql
http::get(string) -> value
```
If an object is given as the second argument, then this can be used to set the request headers.

```surql
http::get(string, $headers: object) -> value
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN http::get('https://surrealdb.com');

-- The HTML code is returned
```

To specify custom headers with the HTTP request, pass an object as the second argument:

```surql
RETURN http::get('https://surrealdb.com', {
	'x-my-header': 'some unique string'
});

-- The HTML code is returned
```

## `http::put`

The `http::put` function performs a remote HTTP `PUT` request. The first parameter is the URL of the remote endpoint, and the second parameter is the value to use as the request body, which will be converted to JSON. If the response does not return a `2XX` status code, then the function will fail and return the error. If the remote endpoint returns an `application/json` content-type, then the response is parsed and returned as a value, otherwise the response is treated as text.

```surql
http::put(string, $body: object) -> value
```

If an object is given as the third argument, then this can be used to set the request headers.

```surql
http::put(string, $body: object, $headers: object) -> value
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN http::put('https://jsonplaceholder.typicode.com/posts/1', {
  id: 1,
  body: "This is some awesome thinking!",
  postId: 100,
  user: {
    id: 63,
    username: 'eburras1q'
  }
});
```

```surql
RETURN http::put('https://jsonplaceholder.typicode.com/posts/1', {
  id: 1,
  body: "This is some awesome thinking!",
  postId: 100,
  user: {
    id: 63,
    username: 'eburras1q'
  }
}, {
  'Authorization': 'Bearer your-token-here',
  'Content-Type': 'application/json',
  'x-custom-header': 'custom-value'
});
```

```surql
{
	body: 'This is some awesome thinking!',
	id: 1,
	postId: 100,
	user: {
		id: 63,
		username: 'eburras1q'
	}
}
```

## `http::post`

The `http::post` function performs a remote HTTP `POST` request. The first parameter is the URL of the remote endpoint, and the second parameter is the value to use as the request body, which will be converted to JSON. If the response does not return a `2XX` status code, then the function will fail and return the error. If the remote endpoint returns an `application/json` content-type, then the response is parsed and returned as a value, otherwise the response is treated as text.

```surql
http::post(string, $body: object) -> value
```
If an object is given as the third argument, then this can be used to set the request headers.

```surql
http::post(string, $body: object, $headers: object) -> value
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN http::post('https://jsonplaceholder.typicode.com/posts/', {
  id: 1,
  body: "This is some awesome thinking!",
  postId: 100,
  user: {
    id: 63,
    username: "eburras1q"
  }
});
```

```surql
RETURN http::post('https://jsonplaceholder.typicode.com/posts/', {
  id: 1,
  body: "This is some awesome thinking!",
  postId: 100,
  user: {
    id: 63,
    username: "eburras1q"
  }
}, {
  'Authorization': 'Bearer your-token-here',
  'Content-Type': 'application/json',
  'x-custom-header': 'custom-value'
});
```

```surql
{
	body: 'This is some awesome thinking!',
	id: 101,
	postId: 100,
	user: {
		id: 63,
		username: 'eburras1q'
	}
}
```

## `http::patch`

The `http::patch` function performs a remote HTTP `PATCH` request. The first parameter is the URL of the remote endpoint, and the second parameter is the value to use as the request body, which will be converted to JSON. If the response does not return a `2XX` status code, then the function will fail and return the error. If the remote endpoint returns an `application/json` content-type, then the response is parsed and returned as a value, otherwise the response is treated as text.

```surql
http::patch(string, $body: object) -> value
```
If an object is given as the third argument, then this can be used to set the request headers.

```surql
http::patch(string, $body: object, $headers: object) -> value
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN http::patch('https://jsonplaceholder.typicode.com/posts/1', {
  id: 1,
  body: "This is some awesome thinking!",
  postId: 100,
  user: {
    id: 63,
    username: "eburras1q"
  }
});
```

```surql
RETURN http::patch('https://jsonplaceholder.typicode.com/posts/1', {
  id: 1,
  body: "This is some awesome thinking!",
  postId: 100,
  user: {
    id: 63,
    username: "eburras1q"
  }
}, {
  'Authorization': 'Bearer your-token-here',
  'Content-Type': 'application/json',
  'x-custom-header': 'custom-value'
});
```

```surql
{
	body: 'This is some awesome thinking!',
	id: 1,
	postId: 100,
	title: 'sunt aut facere repellat provident occaecati excepturi optio reprehenderit',
	user: {
		id: 63,
		username: 'eburras1q'
	},
	userId: 1
}
```

## `http::delete`

The `http::delete` function performs a remote HTTP `DELETE` request. The first parameter is the URL of the remote endpoint, and the second parameter is the value to use as the request body, which will be converted to JSON. If the response does not return a `2XX` status code, then the function will fail and return the error. If the remote endpoint returns an `application/json` content-type, then the response is parsed and returned as a value, otherwise the response is treated as text.

```surql
http::delete(string) -> value
```
If an object is given as the second argument, then this can be used to set the request headers.

```surql
http::delete(string, $headers: object) -> value
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN http::delete('https://jsonplaceholder.typicode.com/posts/1');

{}
```
To specify custom headers with the HTTP request, pass an object as the second argument:

```surql
RETURN http::delete('https://jsonplaceholder.typicode.com/posts/1', {
	'x-my-header': 'some unique string'
});

{}
```

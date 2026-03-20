# Built-in functions

Besides basic JavaScript utilities and [classes for SurrealQL types](/docs/surrealql/functions/script/type-conversion), there are a handful of utilities built into the embedded scripting functions.

## `async fetch(resource, options)` {#fetch}

Full fledged fetch implementation closely matching the [official specification](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

> [!NOTE]
> For complete documentation, please refer to the MDN documentation.

```surql
RETURN function() {
	// List all posts
	const posts = fetch('https://jsonplaceholder.typicode.com/posts');

	// Update post with ID 1
	const updated = fetch('https://jsonplaceholder.typicode.com/posts/1', {
		method: 'PUT',
		body: JSON.stringify({
			id: 1,
			title: 'foo',
			body: 'bar',
			userId: 1,
		}),
		headers: {
			'Content-type': 'application/json; charset=UTF-8',
		},
	});

	return { posts, updated };
}
```

## `async query(surql)` {#query}

Run SurrealQL subqueries from within the embedded scripting functions.

> [!NOTE]
> Only subqueries can be executed with the query() function. This means that only a single query can currently be executed, and that only CRUD operations are allowed.

```surql
CREATE user:john, user:mary;

RETURN function() {
	// Select all users
	const users = await surrealdb.query("SELECT * FROM user");

	// Prepared query
	const query = new surrealdb.Query("SELECT * FROM $id", {
		id: new Record('user', 'mary')
	});

	// Execute prepared query
	const mary = (await surrealdb.query(query))[0];

	// Assign variables later to prepared query
	query.bind('id', new Record('user', 'john'));

	// Execute prepared query
	const john = (await surrealdb.query(query))[0];

	return { john, mary };
}
```

## `async value(variable)` {#value}

Retrieve values for SurrealQL variables from within the embedded scripting functions.

```surql
LET $something = 123;
LET $obj = {
	nested: 456
};

LET $arr = [
	{ value: 1 },
	{ value: 2 },
	{ value: 3 },
	{ value: 4 },
	{ value: 5 },
	{ value: 6 },
];

RETURN function() {
	// Get the value for a variable
	const something = await surrealdb.value("$something");

	// Get the value for a nested property
	const nested = await surrealdb.value("$obj.nested");

	// Filter properties from an array
	const fromArray = await surrealdb.value("$arr[WHERE value > 3].value");

	return { something, nested, fromArray };
}
```

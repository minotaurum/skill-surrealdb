# Object functions

These functions can be used when working with, and manipulating data objects.

## `object::entries`

The `object::entries` function transforms an object into an array with arrays of key-value combinations.

```surql
object::entries(object) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[['a', 1], ['b', true]]"

*/

RETURN object::entries({
  a: 1,
  b: true
});
```

```surql
[
  [ 'a', 1 ],
  [ 'b', true ],
]
```

## `object::extend`

The `object::extend` function extends an object with the fields and values of another one, essentially adding the two together.

```surql
object::extend(object, $other: object) -> object
```

An example of the function, resulting in one new field (`gold`) and one updated field (`last_updated`) in the final output.

```surql
/**[test]

[[test.results]]
value = "{ gold: 100, last_updated: d'2025-10-07T02:36:30.164629Z', name: 'Mat Cauthon' }"
skip-datetime = true

*/

{ name: "Mat Cauthon", last_updated: d'2013-01-08'}.extend( 
{ gold: 100, last_updated: time::now() });
```

```surql
{
	gold: 100,
	last_updated: d'2025-05-07T06:15:00.768Z',
	name: 'Mat Cauthon'
}
```

Note: the same behaviour can also be achieved using the `+` operator.

```surql
/**[test]

[[test.results]]
value = "{ gold: 100, last_updated: d'2025-10-07T02:36:44.494879Z', name: 'Mat Cauthon' }"
skip-datetime = true

*/

{ name: "Mat Cauthon", last_updated: d'2013-01-08'} + 
{ gold: 100, last_updated: time::now() };
```

## `object::from_entries`

The `object::from_entries` function transforms an array with arrays of key-value combinations into an object.

```surql
object::from_entries(array) -> object
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "{ a: 1, b: true }"

*/

RETURN object::from_entries([
  [ "a", 1 ],
  [ "b", true ],
]);
```

```surql
{
  a: 1,
  b: true
}
```

## `object::is_empty`

The `object::is_empty` function checks whether the object contains values.

```surql
object::is_empty(object) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN {
  name: "Aeon",
  age: 20
}.is_empty();

-- false
```

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN object::is_empty({});

-- true
```

Example of `.is_empty()` being used in a [`DEFINE FIELD`](/docs/surrealql/statements/define/field#asserting-rules-on-fields) statement to disallow empty objects:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
match = "$error = Found {  } for field `metadata`, with record `house:.*`, but field must conform to: !$value.is_empty()'"
error = true

[[test.results]]
value = "[{ id: house:ur51z2flendqdp3i130o, metadata: { floors: 5 } }]"
skip-record-id-key = true

*/

DEFINE FIELD metadata
  ON house
  TYPE object
  ASSERT !$value.is_empty();
CREATE house SET metadata = {};
CREATE house SET metadata = { floors: 5 };
```

```surql
-------- Query --------

'Found {  } for field `metadata`, with record `house:aei2fms2jccm46ceib8l`, but field must conform to: !$value.is_empty()'

-------- Query --------

[
	{
		id: house:g126ct3m0scbkockq32u,
		metadata: {
			floors: 5
		}
	}
]
```

## `object::keys`

The `object::keys` function returns an array with all the keys of an object.

```surql
object::keys(object) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "['a', 'b']"

*/

RETURN object::keys({
  a: 1,
  b: true
});

-- [ 'a', 'b' ]
```

## `object::len`

The `object::len` function returns the amount of key-value pairs an object holds.

```surql
object::len(object) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "2"

*/

RETURN object::len({
  a: 1,
  b: true
});

-- 2
```

## `object::remove`

The `object::remove` function removes one or more fields from an object.

```surql
object::remove(object, $to_remove: string|array<string>) -> object
```

A single string can be used to remove a single field from an object, while an array of strings can be used to remove one or more fields at a time.

```surql
/**[test]

[[test.results]]
value = "{ last_updated: d'2013-01-08T00:00:00Z', name: 'Mat Cauthon' }"

[[test.results]]
value = "{ name: 'Mat Cauthon' }"

*/

{ name: "Mat Cauthon", last_updated: d'2013-01-08', gold: 100 }.remove("gold");
{ name: "Mat Cauthon", last_updated: d'2013-01-08', gold: 100 }.remove(["gold", "last_updated"]);
```

```surql
-------- Query 1 --------

{
	last_updated: d'2013-01-08T00:00:00Z',
	name: 'Mat Cauthon'
}

-------- Query 2 --------

{
	name: 'Mat Cauthon'
}
```

## `object::values`

The `object::values` function returns an array with all the values of an object.

```surql
object::values(object) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[1, true]"

*/

RETURN object::values({
  a: 1,
  b: true
});

-- [1, true]
```

## Method chaining

Method chaining allows functions to be called using the `.` dot operator on a value of a certain type instead of the full path of the function followed by the value.

```surql
/**[test]

[[test.results]]
value = "[1, true]"

[[test.results]]
value = "[1, true]"

*/

-- Traditional syntax
object::values({
  a: 1,
  b: true
});

-- Method chaining syntax
{
  a: 1,
  b: true
}.values();
```

```surql
[
  1,
  true
]
```

This is particularly useful for readability when a function is called multiple times.

```surql
/**[test]

[[test.results]]
value = "2"

[[test.results]]
value = "2"

*/

-- Traditional syntax
array::max(object::values(object::from_entries([["a", 1], ["b", 2]])));

-- Method chaining syntax
object::from_entries([["a", 1], ["b", 2]]).values().max();
```

```surql
2
```

# Value functions

This module contains several miscellaneous functions that can be used with values of any type.

## `.chain()`

The `.chain()` method passes a value into a [closure](/docs/surrealql/datamodel/closures) through which an operation can be performed to return any value.

```surql
value.chain(closure) -> value;
```

The output of this function is usually based on the value passed into the closure, but can be something else entirely.

```surql
/**[test]

[[test.results]]
value = "'SurrealDB 3.0'"

*/

'SurrealDB'.chain(|$n| $n + ' 3.0');
-- 'SurrealDB 3.0'

'SurrealDB'.chain(|$n| "Something else");
```

The function is only called using the `.` operator (method syntax) and, as the name implies, works well within a chain of methods.

```surql
/**[test]

[[test.results]]
value = ""{ company: 'SURREALDB!!!!!', latest_version: '2.0' }""

*/

{ company: 'SurrealDB', latest_version: '2.0' }
    .chain(|$name| <string>$name)
    .replace('SurrealDB', 'SURREALDB!!!!!');
```

```surql
"{ company: 'SURREALDB!!!!!', latest_version: '2.0' }"
```

For a similar function that allows using a closure on each item in an array instead of a value as a whole, see [array::map](/docs/surrealql/functions/database/array#arraymap).

## `value::diff`

The `value::diff` function returns an object that shows the [JSON Patch](https://jsonpatch.com/) operation(s) required for the first value to equal the second one.

```surql
value::diff(value, $other: value) -> array<object>
```

The following is an example of the `value::diff` function used to display the changes required to change one string into another. Note that the JSON Patch spec requires an array of objects, and thus an array will be returned even if only one patch is needed between two values.

```surql
/**[test]

[[test.results]]
value = "[{ op: 'change', path: '', value: '@@ -1,5 +1,6 @@
 tobi
-e
+as
' }]"

*/

RETURN 'tobie'.diff('tobias');
```

```surql
[
	{
		op: 'change',
		path: '/',
		value: '@@ -1,5 +1,6 @@
 tobi
-e
+as
'
	}
]
```

An example of the output when the diff output includes more than one operation:

```surql
/**[test]

[[test.results]]
value = "[{ op: 'change', path: '/company', value: '@@ -2,8 +2,10 @@
 urrealDB
+!!
' }, { op: 'add', path: '/latest_version', value: '2.0' }, { op: 'add', path: '/location', value: city:london }]"

*/

{ company: 'SurrealDB' }.diff({ company: 'SurrealDB!!', latest_version: '2.0', location: city:london });
```

```surql
[
	{
		op: 'change',
		path: '/company',
		value: '@@ -2,8 +2,10 @@
 urrealDB
+!!
'
	},
	{
		op: 'add',
		path: '/latest_version',
		value: '2.0'
	},
	{
		op: 'add',
		path: '/location',
		value: city:london
	}
]
```

## `value::patch`

The `value::patch` function applies an array of JSON Patch operations to a value.

```surql
value::patch(value, $patch: array<object>) -> value
```

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ company: 'SurrealDB', latest_version: '3.0' }"

*/

LET $company = {
    company: 'SurrealDB',
    latest_version: '1.5.4'
};

$company.patch([{
		'op': 'replace',
		'path': 'latest_version',
		'value': '3.0'
}]);
```

```surql
{
	company: 'SurrealDB',
	version: '3.0'
}
```

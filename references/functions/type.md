# Type Functions

> [!NOTE]
> Since version 3.0.0-beta, the `::is::` functions (e.g. `type::is::record()`) now use underscores (e.g. `type::is_record()`) to better match the intent of the function and method syntax.

These functions can be used for generating and coercing data to specific data types. These functions are useful when accepting input values in client libraries, and ensuring that they are the desired type within SQL statements.

## `type::array`

The `type::array` function converts a value into an array.

```surql
type::array(array|range) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[1, 2, 3]"

*/

RETURN type::array(1..=3);

-- [1, 2, 3]
```

This is the equivalent of using [`<array>`](/docs/surrealql/datamodel/casting#array) to cast a value to an array.

## `type::bool`

The `type::bool` function converts a value into a boolean.

```surql
type::bool(bool|string) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::bool("true");

-- true
```

This is the equivalent of using [`<bool>`](/docs/surrealql/datamodel/casting#bool) to cast a value to a boolean.

## `type::bytes`

The `type::bytes` function converts a value into bytes.

```surql
type::bytes(bytes|string) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "b"4120666577206279746573""

*/

RETURN type::bytes("A few bytes");

-- b"4120666577206279746573"
```

This is the equivalent of using [`<bytes>`](/docs/surrealql/datamodel/casting) to cast a value to bytes.

## `type::datetime`

The `type::datetime` function converts a value into a datetime.

```surql
type::datetime(datetime|string) -> datetime
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'2022-04-27T18:12:27Z'"

*/

RETURN type::datetime("2022-04-27T18:12:27+00:00");

-- d'2022-04-27T18:12:27Z'
```

This is the equivalent of using [`<datetime>`](/docs/surrealql/datamodel/casting#datetime) to cast a value to a datetime.

## `type::decimal`

The `type::decimal` function converts a value into a decimal.

```surql
type::decimal(decimal|float|int|number|string) -> decimal
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "12345dec"

*/

RETURN type::decimal("12345");

-- 12345dec
```

This is the equivalent of using [`<decimal>`](/docs/surrealql/datamodel/casting#decimal) to cast a value to a decimal.

## `type::duration`

The `type::duration` function converts a value into a duration.

```surql
type::duration(duration|string) -> duration
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "4h"

*/

RETURN type::duration("4h");

-- 4h
```

This is the equivalent of using [`<duration>`](/docs/surrealql/datamodel/casting#duration) to cast a value to a duration.

## `type::field`

The `type::field` function projects a single field within a SELECT statement.

```surql
type::field(string)
```
The following example shows this function, and its output:

```surql
/**[test]

[[test.results]]
value = "[{ id: person:test, name: { first: 'Tobie', last: 'Morgan Hitchcock' }, title: 'Mr' }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ name: { first: 'Tobie', last: 'Morgan Hitchcock' } }]"

[[test.results]]
value = "[{ firstname: 'Tobie', lastname: 'Morgan Hitchcock' }]"

[[test.results]]
value = "[['Tobie', 'Morgan Hitchcock']]"

*/

CREATE person:test SET title = 'Mr', name.first = 'Tobie', name.last = 'Morgan Hitchcock';

LET $param = 'name.first';

SELECT type::field($param), type::field('name.last') FROM person;

SELECT VALUE { 'firstname': type::field($param), lastname: type::field('name.last') } FROM person;

SELECT VALUE [type::field($param), type::field('name.last')] FROM person;
```

```surql
[
	{
		id: person:test,
		title: 'Mr',
		name: {
			first: 'Tobie',
			last: 'Morgan Hitchcock',
	    }
	}
]
```

This function can be used after the `OMIT` clause of a `SELECT` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ age: 19, id: person:7iucxhs7x6ausbdlhj8a, name: 'Galen', surname: 'Pathwarden' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ age: 19, name: 'Galen', surname: 'Pathwarden' }]"

*/

LET $omit = "id";
CREATE person SET name = "Galen", surname = "Pathwarden", age = 19;
SELECT * OMIT type::field($omit) FROM person;
```

```surql
[
	{
		age: 19,
		name: 'Galen',
		surname: 'Pathwarden'
	}
]
```

## `type::fields`

The `type::fields` function projects one or more fields within a SELECT statement.

```surql
type::fields(array<string>)
```
The following example shows this function, and its output:

```surql
/**[test]

[[test.results]]
value = "[{ id: person:test, name: { first: 'Tobie', last: 'Morgan Hitchcock' }, title: 'Mr' }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ name: { first: 'Tobie', last: 'Morgan Hitchcock' }, title: 'Mr' }]"

[[test.results]]
value = "[{ names: ['Tobie', 'Morgan Hitchcock'] }]"

[[test.results]]
value = "[['Tobie', 'Morgan Hitchcock']]"

*/

CREATE person:test SET title = 'Mr', name.first = 'Tobie', name.last = 'Morgan Hitchcock';

LET $param = ['name.first', 'name.last'];

SELECT type::fields($param), type::fields(['title']) FROM person;

SELECT VALUE { 'names': type::fields($param) } FROM person;

SELECT VALUE type::fields($param) FROM person;
```

```surql
[
	{
		id: person:test,
		title: 'Mr',
		name: {
			first: 'Tobie',
			last: 'Morgan Hitchcock',
		}
	}
]
```

This function can be used after the `OMIT` clause of a `SELECT` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ age: 19, id: person:826qwse66s6igdeh977j, name: 'Galen', surname: 'Pathwarden' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ name: 'Galen', surname: 'Pathwarden' }]"

*/

LET $omit = ["id", "age"];
CREATE person SET name = "Galen", surname = "Pathwarden", age = 19;
SELECT * OMIT type::fields($omit) FROM person;
```

```surql
[
	{
		name: 'Galen',
		surname: 'Pathwarden'
	}
]
```

## `type::file`

The `type::file` function converts two strings representing a bucket name and a key into a [file pointer](/docs/surrealql/datamodel/files).

```surql
type::file($bucket: string, $key: string) -> file
```

An example of a file pointer created using this function:

```surql
/**[test]

[[test.results]]
value = "f"my_bucket:/file_name""

*/

type::file("my_bucket", "file_name")
```

```surql
f"my_bucket:/file_name"
```

The following query shows the equivalent file pointer when created using the `f` prefix:

```surql
/**[test]

[[test.results]]
value = "true"

*/

type::file("my_bucket", "file_name") == f"my_bucket:/file_name";

-- true
```

Once a [bucket has been defined](/docs/surrealql/statements/define/indexes), operations using one of the [file functions](/docs/surrealql/functions/database/file) can be performed on the file pointer.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "b"536F6D65206461746120696E73696465""

*/

DEFINE BUCKET my_bucket BACKEND "memory";

type::file("my_bucket", "file_name").put("Some data inside");
type::file("my_bucket", "file_name").get();
```

```surql
b"536F6D65206461746120696E73696465"
```

## `type::float`

The `type::float` function converts a value into a float.

```surql
type::float(decimal|float|int|number|string) -> float
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "12345f"

*/

RETURN type::float("12345");

-- 12345f
```
This is the equivalent of using [`<float>`](/docs/surrealql/datamodel/casting#float) to cast a value to a float.

## `type::int`

The `type::int` function converts a value into an integer.

```surql
type::int(decimal|float|int|number|string) -> int
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "12345"

*/

RETURN type::int("12345");

-- 12345
```
This is the equivalent of using [`<int>`](/docs/surrealql/datamodel/casting#int) to cast a value to a int.

## `type::number`

The `type::number` function converts a value into a number.

```surql
type::number(decimal|float|int|number|string) -> number
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "12345"

*/

RETURN type::number("12345");

-- 12345
```

This is the equivalent of using [`<number>`](/docs/surrealql/datamodel/casting#number) to cast a value to a number.

## `type:of`

The `type::of` function returns a string denoting the type of a value.

```surql
type::of(value) -> string
```

```surql
type::of(2022dec);        -- 'decimal';
type::of(["some", 9]);    -- 'array';
type::of((50.0, 9.9));    -- 'geometry<point>'
```

## `type::point`

The `type::point` function converts a value into a geometry point.

```surql
type::point(array|point) -> point
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "(51.509865, -0.118092)"

*/

RETURN type::point([ 51.509865, -0.118092 ]);

-- (51.509865, -0.118092)
```

## `type::range`

The `type::range` function converts a value into a [range](docs/surrealql/datamodel/ranges). It accepts a single argument, either a range or an array with two values. If the argument is an array, it will be converted into a range, similar to [casting](/docs/surrealql/datamodel/casting).

```surql
type::range(range|array) -> range<record>
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1..2"

[[test.results]]
value = "1..10"

[[test.results]]
value = "'Expected a range but cannot convert [1, 9, 4] into a range'"

*/

RETURN type::range([1, 2]);
-- 1..2

RETURN type::range(1..10);
-- 1..10

RETURN type::range([1,9,4]);
-- 'Expected a range but cannot convert [1, 9, 4] into a range'
```

## `type::record`

> [!NOTE]
> This function was known as `type::thing` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::record` function converts a value into a record pointer definition.

```surql
type::record($table: any, $key: any) -> record
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
LET $tb = "person";
LET $id = "tobie";
RETURN type::record($tb, $id);
```

An example of this function being used to turn an array of objects into records to be created or upserted:

```surql
FOR $data IN [
	{
		id: 9,
		name: 'Billy'
	},
	{
		id: 10,
		name: 'Bobby'
	}
] {
	UPSERT type::record('person', $data.id) CONTENT $data;
};
```

An example of the same except in which the `num` field is to be used as the record's ID. In this case, it can be mapped with the [`array::map()`](/docs/surrealql/functions/database/array#arraymap) function to rename `num` as `id` so that the following `CONTENT` clause does not create both a `num` and an `id` with the same value.

```surql
FOR $data IN [
	{
		name: 'Billy',
		num: 9
	},
    {
		name: 'Bobby',
		num: 10
	},
].map(|$o| {
    id: $o.num,
    name: $o.name
}) {
    UPSERT type::record('person', $data.id) CONTENT $data;
};
```

If the second argument passed into `type::record` is a record ID, the latter part of the ID (the record identifier) will be extracted and used.

```surql
type::record("person", person:mat);

-- person:mat
```

The output of the above function call will thus be `person:mat`, not `person:person:mat`.

The `type::record` function returns a record from a record or a string, with an optional argument to confirm the table name.

```surql
type::record($record: record|string, $table_name: option<string>) -> record
```

The function will return a record as long as the argument passed in is already a record, or a string that can be parsed into one.

```surql
-- Both return person:tobie
type::record(person:tobie);
type::record('person:tobie');
```

The optional second argument allows an assertation that the record passed in is of this table name.

```surql
type::record('person:tobie', 'person'); -- person:tobie
type::record('person:tobie', 'cat'); -- "Expected a record<cat> but cannot convert 'person:tobie' into a record<cat>"
```

This second argument is mostly useful when involving a parameter that may or may not be a certain value. In the code below, the function may or may not err depending on whether the `$record` parameter is a `person` or a `cat` record.

```surql
LET $record = rand::enum(person:tobie, cat:tobie);
type::record($record, 'person');
```

## `type::string`

The `type::string` function converts any value except `NONE`, `NULL`, and `bytes` into a string.

```surql
type::string(any) -> string
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'12345'"

*/

RETURN type::string(12345);

-- '12345'
```

This is the equivalent of using [`<string>`](/docs/surrealql/datamodel/casting#string) to cast a value to a string.

## `type::string_lossy`

The `type::string_lossy` function converts any value except `NONE`, `NULL`, and `bytes` into a string. In the case of bytes, it will not return an error if the bytes are not valid UTF-8. Instead, invalid bytes will be replaced with the character `�` (`U+FFFD REPLACEMENT CHARACTER`, used in Unicode to represent a decoding error).

```surql
type::string(any) -> string
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'Sur�rea�lDB'"

[[test.results]]
value = "'SurrealDB'"

*/

-- Contains some invalid bytes
type::string_lossy(<bytes>[83, 117, 114, 255, 114, 101, 97, 254, 108, 68, 66]);
-- valid bytes
type::string_lossy(<bytes>[ 83, 117, 114, 114, 101, 97, 108, 68, 66 ]);
```

```surql
-------- Query --------

'Sur�rea�lDB'

-------- Query --------

'SurrealDB'
```

This is similar to using [`<string>`](/docs/surrealql/datamodel/casting#string) to cast a value to a string, except that an input of bytes will not fail.

## `type::table`

The `type::table` function converts a value into a table name.

```surql
type::table(record|string) -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[person, cat]"

*/

RETURN [
  type::table("person"),
  type::table(cat:one)
];

-- [person, cat]
```

As of version 2.0, SurrealDB no longer eagerly parses strings into record IDs. As such, the output of the last item ("dog:two") in the following example will differ. In version 1.x, it will be eagerly parsed into a record ID after which the `dog` table name will be returned, while in version 2.x it will be treated as a string and converted into the table name `dog:two`.

```surql
/**[test]

[[test.results]]
value = "`55`"

[[test.results]]
value = "cat"

[[test.results]]
value = "dog"

[[test.results]]
value = "`dog:two`"

*/

RETURN [
  type::table(55),
  type::table(cat:one),
  type::table("dog"),
  type::table("dog:two"),
];
```

```surql
[
	`55`,
	cat,
	dog,
	dog
]
```

```surql
[
	`55`,
	cat,
	dog,
	`dog:two`
]
```

## `type::uuid`

The `type::uuid` function converts a value into a UUID.

```surql
type::uuid(string|uuid) -> uuid
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "u'0191f946-936f-7223-bef5-aebbc527ad80'"

*/

RETURN type::uuid("0191f946-936f-7223-bef5-aebbc527ad80");

-- u'0191f946-936f-7223-bef5-aebbc527ad80'
```

## `type::is_array`

> [!NOTE]
> This function was known as `type::is::array` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_array` function checks if the passed value is of type `array`.

```surql
type::is_array(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_array([ 'a', 'b', 'c' ]);

-- true
```

## `type::is_bool`

> [!NOTE]
> This function was known as `type::is::bool` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_bool` function checks if the passed value is of type `bool`.

```surql
type::is_bool(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_bool(true);

-- true
```

## `type::is_bytes`

> [!NOTE]
> This function was known as `type::is::bytes` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_bytes` function checks if the passed value is of type `bytes`.

```surql
type::is_bytes(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN type::is_bytes("I am not bytes");

-- false
```

## `type::is_collection`

> [!NOTE]
> This function was known as `type::is::collection` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_collection` function checks if the passed value is of type `collection`.

```surql
type::is_collection(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN type::is_collection("I am not a collection");

-- false
```

## `type::is_datetime`

> [!NOTE]
> This function was known as `type::is::datetime` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_datetime` function checks if the passed value is of type `datetime`.

```surql
type::is_datetime(any) -> bool
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_datetime(time::now());

-- true
```

## `type::is_decimal`

> [!NOTE]
> This function was known as `type::is::decimal` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_decimal` function checks if the passed value is of type `decimal`.

```surql
type::is_decimal(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_decimal(<decimal> 13.5719384719384719385639856394139476937756394756);

-- true
```

## `type::is_duration`

> [!NOTE]
> This function was known as `type::is::duration` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_duration` function checks if the passed value is of type `duration`.

```surql
type::is_duration(any) -> bool
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN type::is_duration('1970-01-01T00:00:00');

-- false
```

## `type::is_float`

> [!NOTE]
> This function was known as `type::is::float` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_float` function checks if the passed value is of type ` float`.

```surql
type::is_float(any) -> bool
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_float(<float> 41.5);

-- true
```

## `type::is_geometry`

> [!NOTE]
> This function was known as `type::is::geometry` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_geometry` function checks if the passed value is of type `geometry`.

```surql
type::is_geometry(any) -> bool
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_geometry((-0.118092, 51.509865));

-- true
```

## `type::is_int`

> [!NOTE]
> This function was known as `type::is::int` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_int` function checks if the passed value is of type `int`.

```surql
type::is_int(any) -> bool
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_int(<int> 123);

-- true
```

## `type::is_line`

> [!NOTE]
> This function was known as `type::is::line` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_line` function checks if the passed value is of type `line`.

```surql
type::is_line(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN type::is_line("I am not a line");

-- false
```

## `type::is_none`

> [!NOTE]
> This function was known as `type::is::none` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_none` function checks if the passed value is of type `none`.

```surql
type::is_none(any) -> bool
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_none(NONE);

-- true
```

## `type::is_null`

> [!NOTE]
> This function was known as `type::is::null` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_null` function checks if the passed value is of type `null`.

```surql
type::is_null(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_null(NULL);

-- true
```

## `type::is_multiline`

> [!NOTE]
> This function was known as `type::is::multiline` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_multiline` function checks if the passed value is of type `multiline`.

```surql
type::is_multiline(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN type::is_multiline("I am not a multiline");

-- false
```

## `type::is_multipoint`

> [!NOTE]
> This function was known as `type::is::multipoint` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_multipoint` function checks if the passed value is of type `multipoint`.

```surql
type::is_multipoint(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN type::is_multipoint("I am not a multipoint");

-- false
```

## `type::is_multipolygon`

> [!NOTE]
> This function was known as `type::is::multipolygon` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_multipolygon` function checks if the passed value is of type `multipolygon`.

```surql
type::is_multipolygon(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN type::is_multipolygon("I am not a multipolygon");

-- false
```

## `type::is_number`

> [!NOTE]
> This function was known as `type::is::number` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_number` function checks if the passed value is of type `number`.

```surql
type::is_number(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_number(123);

-- true
```

## `type::is_object`

> [!NOTE]
> This function was known as `type::is::object` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_object` function checks if the passed value is of type `object`.

```surql
type::is_object(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_object({ hello: 'world' });

-- true
```

## `type::is_point`

> [!NOTE]
> This function was known as `type::is::point` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_point` function checks if the passed value is of type `point`.

```surql
type::is_point(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_point((-0.118092, 51.509865));

-- true
```

## `type::is_polygon`

> [!NOTE]
> This function was known as `type::is::polygon` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_polygon` function checks if the passed value is of type `polygon`.

```surql
type::is_polygon(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN type::is_polygon("I am not a polygon");

-- false
```

## `type::is_range`

> [!NOTE]
> This function was known as `type::is::range` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_range` function checks if the passed value is of type `range`.

```surql
type::is_range(any) -> bool
```

```surql
/**[test]

[[test.results]]
value = "true"

[[test.results]]
value = "true"

*/

type::is_range(0..1);
-- true

// method syntax
(0..1).is_range();
-- true
```

## `type::is_record`

> [!NOTE]
> This function was known as `type::is::record` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_record` function checks if the passed value is of type `record`.

```surql
type::is_record(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_record(user:tobie);

-- true
```

### Validate a table

```surql
/**[test]

[[test.results]]
value = "false"

*/

RETURN type::is_record(user:tobie, 'test');

-- false
```

## `type::is_string`

> [!NOTE]
> This function was known as `type::is::string` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_string` function checks if the passed value is of type `string`.

```surql
type::is_string(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_string("abc");

-- true
```

## `type::is_uuid`

> [!NOTE]
> This function was known as `type::is::uuid` in versions of SurrrealDB before 3.0.0-beta. The behaviour has not changed.

The `type::is_uuid` function checks if the passed value is of type `uuid`.

```surql
type::is_uuid(any) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "true"

*/

RETURN type::is_uuid(u"018a6680-bef9-701b-9025-e1754f296a0f");

-- true
```

## Method chaining

Method chaining allows functions to be called using the `.` dot operator on a value of a certain type instead of the full path of the function followed by the value.

```surql
/**[test]

[[test.results]]
value = "false"

[[test.results]]
value = "false"

*/

-- Traditional syntax
type::is_record(r"person:aeon", "cat");

-- Method chaining syntax
r"person:aeon".is_record("cat");
```

```surql
false
```

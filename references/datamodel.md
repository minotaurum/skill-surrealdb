# SurrealDB Data Model & Index


## arrays.mdx

---
sidebar_position: 2
sidebar_label: Arrays
title: Arrays | SurrealQL
description: Records in SurrealDB can store arrays of values, including arrays within arrays

---

import Since from "@components/shared/Since.astro";

# Arrays

An array is a collection of values contained inside `[]` (square brackets), each of which is stored at a certain index. Individual indexes and slices of indexes can be accesses using the same square bracket syntax.

```surql
/**[test]

[[test.results]]
value = "[1, 2, 3, 4, 5]"

[[test.results]]
value = "1"

[[test.results]]
value = "[1, 2, 3]"

*/

-- Return a full array
RETURN [1,2,3,4,5];
-- Return the first ("zeroeth") item
RETURN [1,2,3,4,5][0];
-- Return indexes 0 up to and including 2 of an array
RETURN [1,2,3,4,5][0..=2];
```

```surql title="Output"
-------- Query 2 (200µs) --------

[
	1,
	2,
	3,
	4,
	5
]

-------- Query 3 (99.999µs) --------

1

-------- Query 4 (100.001µs) --------

[
	1,
	2,
	3
]
```

Working with arrays is one of the most important skills when working with SurrealDB, as [`SELECT`](/docs/surrealql/statements/select) statements return an array of values by default unless the `ONLY` keyword is used on an array that contains a single item.

```surql
/**[test]

[[test.results]]
value = "[9]"

[[test.results]]
value = "9"

[[test.results]]
error = "'Expected a single result output when using the ONLY keyword'"

*/

-- Even this returns an array
SELECT * FROM 9;
-- Use the `ONLY` clause to return a single item
SELECT * FROM ONLY 9;
-- Error: array has more than one item
SELECT * FROM ONLY [1,9];
```

```surql title="Output"
-------- Query 1  --------

[
	9
]

-------- Query 2 --------

9

-------- Query 3 --------

'Expected a single result output when using the ONLY keyword'
```

Records in SurrealDB can store arrays of values, including arrays within arrays. Arrays can store any value stored within them, and can store different value types within the same array.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:0l7qs8rfxw32q8namhnx, results: [{ date: '2017-06-18T08:00:00Z', name: 'Algorithmics', score: 76 }, { date: '2018-03-21T08:00:00Z', name: 'Concurrent Programming', score: 83 }, { date: '2018-09-17T08:00:00Z', name: 'Advanced Computer Science 101', score: 69 }, { date: '2019-04-20T08:00:00Z', name: 'Distributed Databases', score: 73 }] }]"
skip-record-id-key = true

*/

CREATE person SET results = [
	{ score: 76, date: "2017-06-18T08:00:00Z", name: "Algorithmics" },
	{ score: 83, date: "2018-03-21T08:00:00Z", name: "Concurrent Programming" },
	{ score: 69, date: "2018-09-17T08:00:00Z", name: "Advanced Computer Science 101" },
	{ score: 73, date: "2019-04-20T08:00:00Z", name: "Distributed Databases" },
];
```

A required number of items can be specified for an array.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
error = ""Couldn't coerce value for field `employees` of `team:2pjsv6kmzs0x06ortqfl`: Expected `array<record<employee>,5>` but found a collection of length `6`""

*/

DEFINE FIELD employees ON TABLE team TYPE array<record<employee>, 5>;
CREATE team SET employees = [
	employee:one, 
	employee:two, 
	employee:three, 
	employee:four, 
	employee:five, 
	employee:doesnt_belong
];
```

```surql title="Response"
"Couldn't coerce value for field `employees` of `team:2pjsv6kmzs0x06ortqfl`: Expected `array<record<employee>,5>` but found a collection of length `6`"
```

## Mapping and filtering on arrays

The `[]` operator after an array can also be used to filter the items inside an array. The parameter `$this` is used to refer to each individual item, while `WHERE` (or its alias `?`, a question mark) is used to set the condition for the item to pass the filter.

```surql
/**[test]

[[test.results]]
value = "[true, true]"

*/
[true, false, true][WHERE $this = true];
```

```surql title="Output"
[true, true]
```

If a `WHERE` or `?` clause finds an item that by itself is not equal to `true` or `false`, it will check the item's [truthiness](/docs/surrealql/datamodel/values#values-and-truthiness) to determine whether to pass it on or not.

```surql
/**[test]

[[test.results]]
value = "[1, 2]"

*/

[1,2,NONE][? $this];

-- [1, 2]
```

Filtering can be repeated if desired.

```surql
/**[test]

[[test.results]]
value = "[{ first_mayor: 'Papa Smurf', name: 'Smurfville', population: 55 }]"

*/

[
    {
        name: "Boston",
        population: NONE,
        first_mayor: "John Phillips"
    },
    {
        name: "Smurfville",
        population: 55,
        first_mayor: "Papa Smurf"
    },
    {
        name: "Harrisburg",
        population: 50183,
        first_mayor: NONE
    }
][WHERE $this.population]
 [WHERE $this.first_mayor];
```

```surql title="Output"
[
	{
		first_mayor: 'Papa Smurf',
		name: 'Smurfville',
		population: 55
	}
]
```

<Since v="v2.0.0" />

## Filtering and mapping with array functions

SurrealDB also includes a number of methods for arrays that make it easier to filter and map. These methods take a closure (an anonymous function) that works in a similar way to the `$this` parameter above.

Here is an example of the `array::filter()` method being used in contrast to the classic `WHERE` syntax. Note that the parameter name inside the closure is named by the user, so `$val` in the example below could be `$v` or `$some_val` or anything else.

```surql
/**[test]

[[test.results]]
value = "[3, 5]"

[[test.results]]
value = "[3, 5]"

*/

[1,3,5].filter(|$val| $val > 2);
[1,3,5][WHERE $this > 2];

-- [3,5]
```

While the [array functions](/docs/surrealql/functions/database/array) section of the documentation contains the full details of each function, the following examples provide a glimpse into how they are commonly used.

The [`array::map()`](/docs/surrealql/functions/database/array#arraymap) function provides access to each item in an array, allowing an opearation to be performed on it before being passed on.

```surql
/**[test]

[[test.results]]
value = "[2, 3, 4]"

*/

[1,2,3].map(|$item| $item + 1);

-- [2,3,4]
```

If desired, a second parameter can be passed in that holds the index of the item.

```surql
/**[test]

[[test.results]]
value = "['At index 0 we got a 1!', 'At index 1 we got a 2!', 'At index 2 we got a 3!']"

*/

[1,2,3].map(|$v, $i| "At index " + <string>$i + " we got a " + <string>$v + "!");
```

```surql title="Output"
[
	'At index 0 we got a 1!',
	'At index 1 we got a 2!',
	'At index 2 we got a 3!'
]
```

Chaining these methods one after another is a convenient way to validate and modify data in a single statement. The example below removes any items with a `NONE`, checks to see if a the location data is a valid geometric point, and then returns the remaining items as objects with a different structure.

```surql
/**[test]

[[test.results]]
value = "[{ coordinates: (98, 65.7), item: 0, name: 'Some city' }, { coordinates: (0, 0.1), item: 1, name: 'Other city' }]"

*/

[
	NONE,
	{
		at: (98, 65.7),
		name: "Some city"
	},
	{
		at: (-190.7, 0),
		name: NONE
	},
    {
        name: "Other city",
        at: (0.0, 0.1)
    },
	{
        name: "Nonexistent city",
        at: (200.0, 66.5)
    }
]
    .filter(|$v| $v != NONE AND $v.name != NONE)
    .filter(|$v| $v.at.is_valid())
    .map(|$v, $i| {
        item: $i,
        name: $v.name,
        coordinates: $v.at
    });
```

```surql title="Output"
[
	{
		coordinates: (98, 65.7),
		item: 0,
		name: 'Some city'
	},
	{
		coordinates: (0, 0.1),
		item: 1,
		name: 'Other city'
	}
]
```

## Adding arrays

<Since v="v3.0.0" />

An array can be added to another array, resulting in a single array consisting of the items of the first followed by those of the second. This is identical to the `array::concat()` function.

```surql
/**[test]

[[test.results]]
value = "[1, 2, 3, 4]"

[[test.results]]
value = "[1, 2, 3, 4]"

*/

[1,2] + [3,4];
[1,2].concat([3,4]);
```

```surql title="Output"
[1,2,3,4]
```

## booleans.mdx

---
sidebar_position: 3
sidebar_label: Booleans
title: Booleans | SurrealQL
description: Boolean values in SurrealDB can be used to mark whether a field is true or false

---

# Booleans

Boolean values can be used to mark whether a field is `true` or `false`.

```surql
CREATE person SET newsletter = false, interested = true;
```

Many SurrealDB operators and functions return booleans.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:jhn1rlhusqypyyyy0gjg, name: 'Billy', name_is_billy: true, name_is_long: false }]"
skip-record-id-key = true

*/


CREATE person SET 
    name = "Billy", 
    name_is_billy = name = "Billy",
    name_is_long = string::len(name) > 10;
```

```surql title="Response"
[
	{
		id: person:7j4t4higwb141v1v2xum,
		name: 'Billy',
		name_is_billy: true,
		name_is_long: false
	}
]
```

Boolean values can be written in anycase.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:1vyseg5ig08xn1ie3h43, interested: true, newsletter: false, very_interested: true }]"
skip-record-id-key = true

*/

CREATE person SET 
    newsletter = FALSE,
    interested = True,
    very_interested = trUE;
```

## Booleans in `WHERE` clauses

When performing a query on the database, accessing a record's ID directly or using a [record range](/docs/surrealql/datamodel/ids#record-ranges) allows performance to be significantly sped up by avoiding the table scan which is used when a `WHERE` clause is included.

However, if a `WHERE` clause is unavoidable, performance can still be improved by simplifying the portion after the clause as much as possible. As a boolean is the simplest possible datatype, having a boolean field that can be used in a `WHERE` clause can significantly improve performance compared to a more complex operation.

```surql
/**[test]

[[test.results]]
value = "[]"

[[test.results]]
value = "[]"

[[test.results]]
value = "[{ data_length: 3, id: person:one, is_short: true, random_data: 'HI!' }]"

[[test.results]]
value = "[{ data_length: 3, id: person:one, is_short: true, random_data: 'HI!' }]"

[[test.results]]
value = "[{ data_length: 3, id: person:one, is_short: true, random_data: 'HI!' }]"

[[test.results]]
value = "[{ data_length: 3, id: person:one, is_short: true, random_data: 'HI!' }]"

*/

DEFINE FIELD data_length ON person VALUE random_data.len();
DEFINE FIELD is_short ON person VALUE random_data.len() < 10;

-- Fill up the database a bit with 10,000 records
CREATE |person:10000| SET random_data = rand::string(1000) RETURN NONE;
-- Add one outlier with short random_data
CREATE person:one SET random_data = "HI!" RETURN NONE;

-- Function call + compare operation: slowest
SELECT * FROM person WHERE random_data.len() < 10;
-- Compare operation: much faster
SELECT * FROM person WHERE data_length < 10;
-- Boolean check: even faster
SELECT * FROM person WHERE is_short;
-- Direct record access: almost instantaneous
SELECT * FROM person:one;
```

## bytes.mdx

---
sidebar_position: 4
sidebar_label: Bytes
title: Bytes | SurrealQL
description: A value that represents the bytes used ubiquitously in computer hardware.

---

import Since from '@components/shared/Since.astro'

# Bytes

Bytes can be created by casting from a string, and are displayed using hexidecimal encoding.

```surql
/**[test]

[[test.results]]
value = "b"4920616D20736F6D65206279746573""

*/

<bytes>"I am some bytes";
```

```surql title="Output"
b"4920616D20736F6D65206279746573"
```

## Conversion from other types

<Since v="v2.3.0" />

Since SurrealDB 2.3.0, conversions can be performed between bytes, strings, and arrays.

```surql
/**[test]

[[test.results]]
value = "'cellar door'"

[[test.results]]
value = "[72, 111, 98, 98, 105, 116, 115]""

*/

-- array<int> to bytes to string
<string><bytes>[ 99, 101, 108, 108, 97, 114, 32, 100, 111, 111, 114 ];
-- string to bytes to array<int>
<array><bytes>"Hobbits";
```

```surql title="Output"
-------- Query --------

'cellar door'

-------- Query --------

[ 72, 111, 98, 98, 105, 116, 115 ]
```

## Byte strings

<Since v="v3.0.0" />

A string preceded by a `b` prefix can be turned into bytes as long as the string represents a hexidecimal value.

```surql
b"486F6262697473";

<string>b"486F6262697473";

<string>b"This won't work though";
```

```surql title="Output"
-------- Query --------

b"486F6262697473";

-------- Query --------

'Hobbits'

-------- Query --------

"There was a problem with the database: Parse error: Unexpected character `T` expected hexidecimal digit
 --> [1:11]
  |
1 | <string>b\"This won't work though\";
  |           ^ 
"
```

## casting.mdx

---
sidebar_position: 21
sidebar_label: Casting
title: Casting | SurrealQL
description: In the SurrealDB type system, values can be converted to other values efficiently.

---

import Table from '@components/shared/Table.astro'

# Casting

In the SurrealDB type system, values can be converted to other values efficiently. This is useful if input is specified in a query which must be of a certain type, or if a user may have provided a parameter with an incorrect type.

<Table>
    <thead>
        <tr>
            <th scope="col" class="w-40">Type</th>
            <th scope="col">Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#array"><code>&lt;array&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into an array
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#arrayt"><code>&lt;array&lt;T&gt;&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into an array of <code>T</code> (some indicated type)
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#bool"><code>&lt;bool&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a boolean
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#datetime"><code>&lt;datetime&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a datetime
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#decimal"><code>&lt;decimal&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a decimal
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#duration"><code>&lt;duration&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a duration
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#float"><code>&lt;float&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a float
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#int"><code>&lt;int&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a int
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#number"><code>&lt;number&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a decimal
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#record"><code>&lt;record&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a record
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#recordt"><code>&lt;record&lt;T&gt;&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a record of <code>T</code> (some indicated type)
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#set"><code>&lt;set&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a set
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#string"><code>&lt;string&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a string
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#regex"><code>&lt;regex&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a regular expression
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#uuid"><code>&lt;uuid&gt;</code></a>
            </td>
            <td scope="row" data-label="Description">
                Casts the subsequent value into a UUID
            </td>
        </tr>
    </tbody>
</Table>

## `<array>`

The `<array>` casting function converts a range into an array.

```surql
/**[test]

[[test.results]]
value = "[1, 2, 3]"

*/

<array>1..=3;
-- Output:
[1, 2, 3]
```

## `<array<T>>`

The `<array<T>>` casting function converts a value into an array of the specified type.

>[!NOTE]
>When using this casting function, the value must be an array and each element in the array will be cast to the specified type. 

```surql
/**[test]

[[test.results]]
value = "[42, 314, 271, 137, 141]"

*/

<array<int>>["42", "314", "271", "137", "141"];
-- Output:
[42, 314, 271, 137, 141]
```

```surql
/**[test]

[[test.results]]
value = "['42', '314', '271', '137', '141']"

*/

<array<string>> [42, 314, 271, 137, 141];
-- Output:
['42', '314', '271', '137', '141']
```

A cast into an array of more than one possible type can also be used. In this case, the cast will attempt to cast into the possible types in order. As such, the `string` in the first query below will be cast into a `datetime` but not in the second.

```surql
/**[test]

[[test.results]]
value = "[[d'2020-09-09T00:00:00Z', '21 Jan 2020'], ['2020-09-09', '21 Jan 2020']]"

*/

[
  <array<datetime|string>>["2020-09-09", "21 Jan 2020"],
  <array<string|datetime>>["2020-09-09", "21 Jan 2020"]
];
```

```surql title="Output"
[
	[
		d'2020-09-09T00:00:00Z',
		'21 Jan 2020'
	],
	[
		'2020-09-09',
		'21 Jan 2020'
	]
]
```

An example of even more complex casting which attempts to cast each item in the input array into a `record<user>`, then `record<person>`, then `array<record<user>>`, and finally `string`.

```surql
/**[test]

[[test.results]]
value = "['person:one', 'user:two', "['user:three', 'user:four']", 'not_a_person_or_user']"

*/

<array<record<user | person> | array<record<user>> | string>> [
	'person:one',
	'user:two',
	[
		'user:three',
		'user:four'
	],
	'not_a_person_or_user'
];
```

```surql title="Output"
[
	person:one,
	user:two,
	[
		user:three,
		user:four
	],
	'not_a_person_or_user'
]
```

## `<bool>`

The `<bool` casting function converts a value into a boolean.

```surql
/**[test]

[[test.results]]
value = "true"

*/

<bool>"true";
-- Output:
true
```

```surql
/**[test]

[[test.results]]
value = "false"

*/

<bool>"false";
-- Output:
false
```

## `<datetime>`

The `<datetime>` casting function converts a value into a datetime.

```surql
/**[test]

[[test.results]]
value = "d'2025-06-07T00:00:00Z'"

*/

<datetime>"2025-06-07";
-- Output:
d'2025-06-07T00:00:00Z'
```

## `<decimal>`

The `<decimal>` casting function converts a value into a decimal which allows for 128 bits of precision.

```surql
<decimal>"13.5729484672938472938410938456";
-- Output:
13.572948467293847293841093846dec
```

Decimal casting should generally not be used to convert from floats with a large number of digits after the decimal point, because the input to the right will first be turned into a less precise float before the cast is performed.

```surql
<decimal>13.572948467293847293841093845679289;
-- Output:
13.57294846729385dec

<decimal>1.193847193847193847193487E11;
-- Output:
119384719384.7194dec
```

In this case, the `dec` suffix is preferable as it will instruct the database to treat the **input** as a decimal, rather than create a float to then cast into a decimal.

```surql
13.572948467293847293841093845679289dec;
-- Output:
13.572948467293847293841093846dec

1.193847193847193847193487E11dec;
-- Output:
1.193847193847193847193487E11dec;
```

## `<duration>`

The `<duration>` casting function converts a value into a duration.

```surql
/**[test]

[[test.results]]
value = "1h30m"

*/

<duration>"1h30m";
-- Output:
1h30m
```

<br />

## `<float>`

The `<float>` casting function converts a value into a floating point number. Floating point numbers by nature have a limited amount of precision.

```surql
/**[test]

[[test.results]]
value = "13.572948467293847f"

*/

<float>13.572948467293847293841093845679289;
-- Output:
13.572948467293847f
```

```surql
/**[test]

[[test.results]]
value = "13.572948467293847f"

*/

<float>"13.572948467293847293841093845679289";
-- Output:
13.572948467293847
```

## `<int>`

The `<int>` casting function converts a value into an integer.

```surql
/**[test]

[[test.results]]
value = "53"

*/

<int>53;
-- Output:
53
```

## `<number>`

The `<number>` casting function converts a value into a `number`.

```surql
<number>13.572948467293847293841093845679289;
-- Output:
"13.572948467293847293841093845679289"
```

```surql
<number>"13.572948467293847293841093845679289";
-- Output:
"13.572948467293847293841093845679289"
```

```surql
<number>1.193847193847193847193487E11;
-- Output:
"119384719384.7193847193487"
```

## `<record>`

The `<record>` casting function converts a value into a record.

Keep in mind when using this casting function that if the equivalent record id does not exist, it will not return anything.

```surql
SELECT id FROM <record>"person:hrebrffwm4sr2yifglta";
```

```surql title="Output"
{ id: person:hrebrffwm4sr2yifglta }
```

## `<record<T>>`

The `<record<T>>` casting function converts a value into a record.

Keep in mind when using this casting function that if the equivalent record id does not exist, it will not return anything.

```surql
SELECT id FROM <record>"person:hrebrffwm4sr2yifglta";
-- Output:
{ id: person:hrebrffwm4sr2yifglta }
```

A cast into a number of possible record types can also be used.

```surql
/**[test]

[[test.results]]
value = "[user:one, [person:one, user:two]]"

*/

[
  <record<user|person>>"user:one",
  <array<record<user|person>>>["person:one", "user:two"]
];
```

```surql title="Output"
[
	user:one,
	[
		person:one,
		user:two
	]
]
```

## `<set>` and `<set<T>>`

The `<set>` casting function converts a value into a set.

```surql
/**[test]

[[test.results]]
value = "[{'21 Jan 2020', d'2020-09-09T00:00:00Z'}, {'2020-09-09', '21 Jan 2020'}]"

*/

[
  <set<datetime|string>>["2020-09-09", "21 Jan 2020"],
  <set<string|datetime>>["2020-09-09", "21 Jan 2020"]
];
```

```surql title="Output"
[
	[
		d'2020-09-09T00:00:00Z',
		'21 Jan 2020'
	],
	[
		'2020-09-09',
		'21 Jan 2020'
	]
]
```

## `<string>`

The `<string>` casting function converts a value into a string.

```surql
/**[test]

[[test.results]]
value = "'true'"

*/

<string>true;
-- Output:
'true'
```

```surql
/**[test]

[[test.results]]
value = "'1.3463f'"

*/

<string>1.3463;
-- Output:
'1.3463f'
```

```surql
/**[test]

[[test.results]]
value = "'false'"

*/

<string>false;
-- Output:
"false"
```

## `<regex>`

The `<regex>` casting function converts a value into a regular expression.

```surql
/**[test]

[[test.results]]
value = "true"

*/

<regex> "a|b" = "a";
-- Output:
true
```

```surql
/**[test]

[[test.results]]
value = "false"

*/

<regex> "a|b" = "c";
-- Output:
false
```

## `<uuid>`

The `<uuid>` casting function converts a value into a UUID.

```surql
SELECT id FROM <uuid> "a8f30d8b-db67-47ec-8b38-ef703e05ad1b";
-- Output:
[ u'a8f30d8b-db67-47ec-8b38-ef703e05ad1b' ]
```

## General notes on casting

### Syntax and order

As the parser ignores spaces and new lines, casting syntax can include spaces or new lines as desired.

```surql
/**[test]

[[test.results]]
value = "['9.1', true, '15h']"

[[test.results]]
value = "['9.1', true, '15h']"

*/

-- Surrealist formatted syntax
 <array<bool | string | float>> [
	'9.1',
	'true',
	15h
];

-- Maybe someone's preferred syntax?
<array
        <bool | string | float>
      >
[ '9.1', 'true', 15h ];
```

When more than one cast type is specified, SurrealDB will attempt to convert into the type in the order specified. In the example above, while the input `'9.1'` could have been converted to a float, the type `string` comes first in the cast syntax and thus `'9.1'` remains as a string.

```surql title="Output"
[
	'9.1',
	true,
	'15h'
]
```

### Casting vs. affixes

SurrealDB uses a number of affixes to force the parser to treat an input as a certain type instead of another. These affixes may seem at first glance to be identical to casts, as the following queries show.

```surql
/**[test]

[[test.results]]
value = "person:one"

[[test.results]]
value = "person:one"

[[test.results]]
value = "person:one"

[[test.results]]
value = "'person:one'"

[[test.results]]
value = "98dec"

[[test.results]]
value = "98dec"

[[test.results]]
value = "98"

*/

-- All return a record person:one
r"person:one";
<record>"person:one";
<record<person>>"person:one";
-- Returns a string 'person:one'
'person:one';

-- Both return a decimal 98dec
98dec;
<decimal>98;

-- Returns an int 98
98;
```

However, casts and affixes work in different ways:

* A cast is a way to convert from one type into another.
* An affix is an instruction to the parser to treat an input as a certain type.

These differences become clear when working with input that is less than ideal or does not work with a certain type. For example, floats by nature become imprecise after a certain number of digits.

```surql
/**[test]

[[test.results]]
value = "[8.888f, 8.88888888888889f]"

*/

[
  8.888,
  8.8888888888888888
];
```

```surql title="Output"
[
	8.888f,
	8.88888888888889f
]
```

In this case, a `decimal` can be used which will allow a greater number of digits after the decimal point. However, casting the above numbers into a `decimal` will result in the same inaccurate output.

```surql
/**[test]

[[test.results]]
value = "[8.888dec, 8.88888888888889dec]"

*/

[
	<decimal>8.888,
	<decimal>8.888888888888888
];
```

```surql title="Output"
[
	8.888dec,
	8.88888888888889dec
]
```

This is because the parser will first treat the number as a float and then cast it into a `decimal`.

However, using the `dec` suffix will inform the parser that the entire input is to be treated as a `decimal` and it will never pass through a stage in which it is a float.

```surql
/**[test]

[[test.results]]
value = "[8.888dec, 8.888888888888888dec]"

*/

[
	8.888dec,
	8.888888888888888dec
];
```

```surql title="Output"
[
	8.888dec,
	8.888888888888888dec
]
```

Similarly, an attempt to cast a number that is too large for an `int` into a `decimal` will not work, as the parser will first attempt to handle the number on the right before moving on to the cast.

```surql
/**[test]

[test.results]
parsing-error = """Failed to parse number: number cannot fit within a 64bit signed integer'
 --> [1:10]
  |
1 | <decimal>9999999999999999999;
  |          ^^^^^^^^^^^^^^^^^^^"""

*/

<decimal>9999999999999999999;
```

```surql title="Output"
'Failed to parse number: number cannot fit within a 64bit signed integer'
```

However, if the same number is followed by the `dec` suffix, the parser will be aware that the input is meant to be treated as a `decimal` from the outset and the query will succeed.

```surql
/**[test]

[[test.results]]
value = "9999999999999999999dec"

*/

9999999999999999999dec;
```


## closures.mdx

---
sidebar_position: 5
sidebar_label: Closures
title: Closures | SurrealQL
description: Anonymous functions in SurrealDB allow you to define small, reusable pieces of logic that can be used throughout your queries.
---
import Since from '@components/shared/Since.astro'

# Anonymous functions (closures)

<Since v="v2.0.0" />

```syntax title="SurrealQL Syntax"
LET $parameter = |@parameters| @expression;
```

One powerful feature available in SurrealDB is the ability to define anonymous functions. These functions can be used to encapsulate reusable logic and can be called from within your queries. Below are some examples demonstrating their capabilities:

## Basic function definitions

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "4"

[[test.results]]
value = "NONE"

[[test.results]]
value = "'Hello, World!'"

*/

-- Define an anonymous function that doubles a number
LET $double = |$n: number| $n * 2;
RETURN $double(2);  -- Returns 4

-- Define a function that concatenates two strings
LET $concat = |$a: string, $b: string| $a + $b;
RETURN $concat("Hello, ", "World!");  -- Returns "Hello, World!"
```

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "'Hello, Alice!'"

*/

-- Define a function that greets a person
LET $greet = |$name: string| -> string { "Hello, " + $name + "!" };
RETURN $greet("Alice");   -- Returns "Hello, Alice!"
```

## Error Handling and Type Enforcement

You can also enforce type constraints within your functions to prevent type mismatches:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "'HELLO'"

[[test.results]]
error = "'Incorrect arguments for function ANONYMOUS(). Expected a value of type 'string' for argument $text'"

[[test.results]]
value = "NONE"

[[test.results]]
value = "16"

[[test.results]]
error = "'Incorrect arguments for function ANONYMOUS(). Expected a value of type 'number' for argument $num'"

*/

-- Define a function with a return type
LET $to_upper = |$text: string| -> string { string::uppercase($text) };
RETURN $to_upper("hello");  -- Returns "HELLO"
RETURN $to_upper(123);      -- Error: type mismatch

-- Define a function that accepts only numbers
LET $square = |$num: number| $num * $num;
RETURN $square(4);    -- Returns 16
RETURN $square("4");  -- Error: type mismatch
```

## Closures in functions

Many of SurrealDB's functions allow a closure to be passed in, making it easy to use complex logic on a value or the elements of an array.

The `chain` function which performs an operation on a value before passing it on:

```surql
/**[test]

[[test.results]]
value = "2000"

*/

"Two"
    .replace("Two", "2")
    .chain(|$num| <number>$num * 1000);
```

```surql title="Response"
2000
```

We can see that the input to the `.chain()` method is indeed a closure by creating our own that is assigned to a parameter. This closure can be passed into `.chain()`, returning the same output as above.

```surql
LET $my_func = |$num| <number>$num * 1000;

"Two"
    .replace("Two", "2")
    .chain($my_func);
```

The following example shows a chain of array functions used to remove useless data, followed by a check to see if all items in the array match a certain condition, and then a cast into another type. The [`array::filter`](/docs/surrealql/functions/database/array#arrayfilter) call in the middle ensures that the [`string::len`](/docs/surrealql/functions/database/string#stringlen) function that follows is being called on string values.

```surql
/**[test]

[[test.results]]
value = "'true'"

*/

[NONE, NONE, "good data", "Also good", "important", NULL]
    .filter(|$v| $v.is_string())
    .all(|$s| $s.len() > 5)
    .chain(|$v| <string>$v);
```

```surql title="Response"
'true'
```

## Closure limitations

Closures work inside a read-only context, and cannot be used to modify database resources.

```surql
-- 1. Create a test table and function
DEFINE TABLE test_table SCHEMAFULL;
DEFINE FIELD name ON test_table TYPE string;

DEFINE FUNCTION fn::test_create($name: string) -> object {
    CREATE test_table CONTENT { name: $name };
    { created: true, name: $name };
};

-- 2. Call the function directly - works
fn::test_create("direct_call");

-- 3. Call the function inside .map() - fails
LET $names = ["Alice", "Bob", "Charlie"];
$names.map(|$n| fn::test_create($n));
-- Error: "Couldn't write to a read only transaction"
```

In many cases, a closure can be substituted by another operation such as a `FOR` loop or a regular `SELECT` statement.

```surql
DEFINE TABLE test_table SCHEMAFULL;
DEFINE FIELD name ON test_table TYPE string;

DEFINE FUNCTION fn::test_create($name: string) -> object {
    CREATE test_table CONTENT { name: $name };
    { created: true, name: $name };
};

-- Function to create a record called for each name
SELECT VALUE fn::test_create($this) FROM ["Alice", "Bob", "Charlie"];
```

## Capturing parameters

<Since v="v3.0.0" />

The original implementation of closures did not allow them to capture parameters (variables) in their scope. Strictly speaking, this made them simple anonymous functions as closures did not "enclose" anything.

```surql
LET $okay_nums = [1,2,3];

-- Returns [] because $okay_nums not present inside the closure
[1,5,6,7,0].filter(|$n| $n IN $okay_nums);
```

This has since been resolved, allowing a parameter declared outside a closure to be recognized inside it.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[[1]]"

*/

LET $okay_nums = [1,2,3];

[1,5,6,7,0].filter(|$n| $n IN $okay_nums);
```

## Conclusion

These anonymous functions provide a flexible way to define small, reusable pieces of logic that can be used throughout your queries. By leveraging them, you can write more modular and maintainable SurrealQL code.


## datetimes.mdx

---
sidebar_position: 6
sidebar_label: Datetimes
title: Datetimes | SurrealQL
description: SurrealDB has native support for datetimes with nanosecond precision. SurrealDB is able to parse datetimes from strings.

---

import Since from '@components/shared/Since.astro'

# Datetimes

SurrealDB has native support for datetimes with nanosecond precision. SurrealDB automatically parses and understands datetimes which are written as strings in the SurrealQL language. Times must also be formatted in [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) format.

Datetimes are represented and created by a `d` prefex in front of a string.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:s9mfw7f45qhlzs0ogche, time: d'2025-07-03T07:18:52Z' }]"
skip-record-id-key = true

*/

CREATE event SET time = d"2025-07-03T07:18:52Z";
```

SurrealDB handles all datetimes with nanosecond precision.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:g7cdb2ny6qozzsrlp9qj, time: d'2025-07-03T07:18:52.841147Z' }]"
skip-record-id-key = true

*/

CREATE event SET time = d"2025-07-03T07:18:52.841147Z";
```

SurrealDB handles all timezones, and automatically converts and stores datetimes as a UTC date.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:wieqqme8hybzpnh043d2, time: d'2025-07-03T05:18:52.841147Z' }]"
skip-record-id-key = true

*/

CREATE event SET time = d"2025-07-03T07:18:52.841147+02:00";
```

A `datetime` can also be created by using `<datetime>` to cast from a string.

With correct input:

```surql
/**[test]

[[test.results]]
value = "[{ id: event:9r69r6c6ovkky3pg7oko, time: d'2025-07-03T07:18:52.841147Z' }]"
skip-record-id-key = true

*/

CREATE event SET time = <datetime>"2025-07-03T07:18:52.841147Z";
```

```surql title="Response"
[{ id: event:jwm8ncmfi30nrxdf24ws, time: d'2025-07-03T07:18:52.841147Z' }]
```

With incorrect input (missing final Z):

```surql
/**[test]

[[test.results]]
error = "'"Expected `datetime` but found a `'2025-07-03T07:18:52.841147'`"'"

*/

CREATE event SET time = <datetime>"2025-07-03T07:18:52.841147";
```

```surql title="Response"
"Expected a datetime but cannot convert '2025-07-03T07:18:52.841147' into a datetime"
```

As a convenience, a date containing a year, month and day but no time will also parse correctly as a datetime.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:ob9p04agl2aoqipet339, time: d'2024-04-03T00:00:00Z' }]"
skip-record-id-key = true

*/

CREATE event SET time = <datetime>"2024-04-03";
```

```surql title="Response"
[{ id: event:4t50wjjlne9v8km2qcwq, time: d'2024-04-03T00:00:00Z' }]
```

## Datetime types in `DEFINE FIELD` statements

Defining a field with a set `datetime` type will ensure that datetimes are properly formatted and not passed on as simple strings.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
error = ""Couldn't coerce value for field `time` of `event:qv8qcjf0w9oowekl36w6`: Expected `datetime` but found `'2025-07-03T07:18:52.841147'`""

*/

DEFINE FIELD time ON event TYPE datetime;
// highlight-next-line
CREATE event SET time = "2025-07-03T07:18:52.841147";
```

```surql title="Response"
"Couldn't coerce value for field `time` of `event:qv8qcjf0w9oowekl36w6`: Expected `datetime` but found `'2025-07-03T07:18:52.841147'`"
```

The above query will fail because the datetime is not cast as a datetime type. The correct query is:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: event:w2lhv58f7c9z7xo4nqkq, time: d'2025-07-03T07:18:52.841140Z' }]"
skip-record-id-key = true

*/

DEFINE FIELD time ON event TYPE datetime;
// highlight-next-line
CREATE event SET time = d"2025-07-03T07:18:52.84114Z";
```

```surql title="Response"
[
    { 
        id: event:w2lhv58f7c9z7xo4nqkq, 
        time: d'2025-07-03T07:18:52.841140Z' 
    }
]
```

### Datetime comparison
A datetime can be compared with another using the advanced SurrealDB operators.

```surql
d"2025-07-03T07:18:52Z" < d"2025-07-03T07:18:52.84114Z";
```
    
```surql title="Response"
true
```

## Durations and datetimes

A duration can be used to alter a datetime.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:nyk8onbwm2e1n8z32rj9, time: d'2025-07-17T07:18:52Z' }]"
skip-record-id-key = true

*/

CREATE event SET time = d"2025-07-03T07:18:52Z" + 2w;
```
    
```surql title="Response"
[{ id: event:`9ey7v8r0fd46xblf9dsf`, time: d'2025-07-17T07:18:52Z' }]
```

Multi-part durations can also be used to modify datetimes.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:5uuzy32t48yutxyszi7p, time: d'2025-07-03T08:49:14.191147Z' }]"
skip-record-id-key = true

*/

CREATE event SET time = d"2025-07-03T07:18:52.841147Z" + 1h30m20s1350ms;
```

```surql title="Response"
[{ id: event:5uuzy32t48yutxyszi7p, time: d'2025-07-03T08:49:14.191147Z' }]
```

## Altering datetimes

<Since v="v3.0.2" />

Each value in a datetime can be set by using one of seven `time::set_` functions. Each of these function names ends with the part of the datetime that is modified, such as `time::set_year()` or `time::set_hour()`.

```surql
d'1970-01-01T00:00:00.000000100Z'.set_year(1914);
-- Output
d'1914-01-01T00:00:00.000000100Z'
```

As these functions do not modify an existing datetime but return a new one, they can be chained one after another.

```surql
d'1970-01-01T00:00:00.000000100Z'
    .set_year(1914)
    .set_month(6)
    .set_day(28);

-- Output
d'1914-06-28T00:00:00.000000100Z'
```

## Next steps
You've now seen how to store, modify, and handle dates and times in SurrealDB. For more advanced functionality, take a look at the [time](/docs/surrealql/functions/database/time) functions, which enable extracting, altering, rounding, and grouping datetimes into specific time intervals.


## durations.mdx

---
sidebar_position: 7
sidebar_label: Durations
title: Durations | SurrealQL
description: SurrealDB has native support for durations with nanosecond precision. SurrealDB is able to parse durations from strings.

---

import Since from '@components/shared/Since.astro'

# Durations

A `duration` represents a non-negative period of time.

## Duration units

Durations can be specified in any of the following units:

<table>
    <thead>
    <tr>
        <th scope="col">Unit</th>
        <th scope="col">Description</th>
    </tr>
    </thead>
    <tbody>
        <tr>
            <td scope="row" data-label="Type">ns</td>
            <td scope="row" data-label="Description">Nanoseconds</td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">us</td>
            <td scope="row" data-label="Description">Microseconds, alternative: µs</td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">ms</td>
            <td scope="row" data-label="Description">Milliseconds</td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">s</td>
            <td scope="row" data-label="Description">Seconds</td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">m</td>
            <td scope="row" data-label="Description">Minutes</td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">h</td>
            <td scope="row" data-label="Description">Hours</td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">d</td>
            <td scope="row" data-label="Description">Days</td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">w</td>
            <td scope="row" data-label="Description">Weeks</td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">y</td>
            <td scope="row" data-label="Description">Years</td>
        </tr>
    </tbody>
</table>

## Creating and using durations

A duration can be composed of any number of duration units.

```surql
1y40w20h;
```

A duration that contains multiple instances of the same unit type will parse as well, combining lesser units into greater units when a maximum value is reached.

For example, a duration that includes `12h` two times will be evaluated as `1d`.

```surql
1d1d12h12h;
-- 3d
```

A duration can also be created by casting a string.

```surql
<duration>"1d1d12h12h";
-- 3d
```

A duration can be zero, but cannot be negative.

```surql
0ns;
0d; -- Evaluates to 0ns
```

The maximum possible duration can be accessed via the const [`duration::max`](/docs/surrealql/functions/database/duration#durationmax), above which a duration cannot be formed.

```surql
duration::max;
-- 584942417355y3w5d7h15s999ms999µs999ns

duration::max + 1ns
-- 'Failed to compute: "584942417355y3w5d7h15s999ms999µs999ns + 1ns", as the operation results in an arithmetic overflow.'
```

Durations can be added to and subtracted from other durations as well as datetimes.

```surql
d'1970-01-01' + 1d;
-- d'1970-01-02T00:00:00Z'

1y - 6w;
46w1d;
```

## Multiplying and dividing durations

<Since v="v3.0.1" />

A duration can be multiplied and divided by a number.

```surql
1d / 24;
-- 1h

1d * 5.5;
-- 5d12h
```

## files.mdx

---
sidebar_position: 8
sidebar_label: Files
title: Files | SurrealQL
description: SurrealDB allows a bucket to be declared locally or globally to work with files.

---

import Since from '@components/shared/Since.astro'

# Files

<Since v="v3.0.0" />

Files are accessed by a path, which is prefixed with an `f` to differentiate it from a regular string.

Some examples of file pointers:

```surql
/**[test]

[[test.results]]
value = "f"bucket:/some/key/to/a/file.txt""

*/

f"bucket:/some/key/to/a/file.txt";
f"bucket:/some/key/with\ escaped";
```

To work with the files that can be accessed through these pointers, use the following:

* A [`DEFINE BUCKET`](/docs/surrealql/statements/define/bucket) statement to set up the bucket to hold the files
* [Files functions](/docs/surrealql/functions/database/file) such as `file::put()` and `file::get()`

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "b"536F6D65207465787420696E73696465""

[[test.results]]
value = "'Some text inside'"

*/

DEFINE BUCKET my_bucket BACKEND "memory";
f"my_bucket:/some_file.txt".put("Some text inside");
f"my_bucket:/some_file.txt".get();
<string>f"my_bucket:/some_file.txt".get();
```

```surql title="Output"
-------- Query --------

b"536F6D65207465787420696E73696465"

-------- Query --------

'Some text inside'
```

## Using files for ad-hoc memory storage

A combination of files and SurrealDB's [encoding functions](/docs/surrealql/functions/database/encoding#encodingcbordecode) can be used to set up ad-hoc memory storage. This can be convenient when running an instance that saves data to disk but prefers to keep certain items in memory.

The following example shows how this pattern might be used for temporary storage such as a user's shopping cart during a single session.

```bash
# Set the allowlist env var to allow the directory to be accesses
SURREAL_BUCKET_FOLDER_ALLOWLIST="/users/your_user_name" surreal start --allow-experimental files
```

```surql
-- Set up the in-memory backend
DEFINE BUCKET my_bucket BACKEND "file:/users/your_user_name";

-- Convenience functions to save, decode back into
-- SurrealQL type, and delete
DEFINE FUNCTION fn::save_file($file_name: string, $input: any) {
    LET $file = type::file("shopping_carts", $file_name);
    $file.put(encoding::cbor::encode($input));
};

DEFINE FUNCTION fn::get_file($file_name: string) -> object {
    encoding::cbor::decode(type::file("shopping_carts", $file_name).get())
};

DEFINE FUNCTION fn::delete_file($file_name: string) {
    type::file("shopping_carts", $file_name).delete();
};

-- Save current shopping cart
fn::save_file("temp_cart_user_24567", {
    items: ["shirt1"],
    last_updated: time::now()
});

fn::get_file("temp_cart_user_24567");
-- Returns { items: ['shirt1', 'deck_of_cards'], last_updated: d'2025-11-20T01:03:24.141080Z' }

-- User adds item, save over file with newer information
fn::save_file("temp_cart_user_24567", {
    items: ["shirt1", "deck_of_cards"],
    last_updated: time::now()
});

fn::get_file("temp_cart_user_24567");
-- Returns { items: ['shirt1', 'deck_of_cards'], last_updated: d'2025-11-20T01:06:02.752429Z' }

-- Session is over, delete temp file
fn::delete_file("temp_cart_user_24567");
```

## formatters.mdx

---
sidebar_position: 22
sidebar_label: Formatters
title: Formatters | SurrealQL
description: Formatting functions in SurrealQL accept certain text formats for date/time formatting.

---

# Formatters

The [string::is_datetime](/docs/surrealql/functions/database/string#stringisdatetime) and [time::format](/docs/surrealql/functions/database/time#timeformat) functions in SurrealQL accept certain text formats for date/time formatting. The possible formats are listed below.

### Date formatters

<table>
<thead>
  <tr>
    <th scope="col">Specifier</th>
    <th scope="col">Example</th>
    <th scope="col">Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td scope="row" data-label="Specifier">%Y</td>
    <td scope="row" data-label="Example">2001</td>
    <td scope="row" data-label="Description">The full proleptic Gregorian year, zero-padded to 4 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%C</td>
    <td scope="row" data-label="Example">20</td>
    <td scope="row" data-label="Description">The proleptic Gregorian year divided by 100, zero-padded to 2 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%y</td>
    <td scope="row" data-label="Example">01</td>
    <td scope="row" data-label="Description">The proleptic Gregorian year modulo 100, zero-padded to 2 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%m</td>
    <td scope="row" data-label="Example">07</td>
    <td scope="row" data-label="Description">Month number (01 to 12), zero-padded to 2 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%b</td>
    <td scope="row" data-label="Example">Jul</td>
    <td scope="row" data-label="Description">Abbreviated month name. Always 3 letters.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%B</td>
    <td scope="row" data-label="Example">July</td>
    <td scope="row" data-label="Description">Full month name.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%h</td>
    <td scope="row" data-label="Example">Jul</td>
    <td scope="row" data-label="Description">Same as %b.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%d</td>
    <td scope="row" data-label="Example">08</td>
    <td scope="row" data-label="Description">Day number (01 to 31), zero-padded to 2 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%e</td>
    <td scope="row" data-label="Example">8</td>
    <td scope="row" data-label="Description">Same as %d but space-padded. Same as %_d.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%a</td>
    <td scope="row" data-label="Example">Sun</td>
    <td scope="row" data-label="Description">Abbreviated weekday name. Always 3 letters.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%A</td>
    <td scope="row" data-label="Example">Sunday</td>
    <td scope="row" data-label="Description">Full weekday name.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%w</td>
    <td scope="row" data-label="Example">0</td>
    <td scope="row" data-label="Description">Day of the week. Sunday = 0, Monday = 1, ..., Saturday = 6.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%u</td>
    <td scope="row" data-label="Example">7</td>
    <td scope="row" data-label="Description">Day of the week. Monday = 1, Tuesday = 2, ..., Sunday = 7. ([RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339))</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%U</td>
    <td scope="row" data-label="Example">28</td>
    <td scope="row" data-label="Description">Week number starting with Sunday (00 to 53), zero-padded to 2 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%W</td>
    <td scope="row" data-label="Example">27</td>
    <td scope="row" data-label="Description">Same as %U, but week 1 starts with the first Monday in that year instead.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%G</td>
    <td scope="row" data-label="Example">2001</td>
    <td scope="row" data-label="Description">Same as %Y but uses the year number in [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) week date.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%g</td>
    <td scope="row" data-label="Example">01</td>
    <td scope="row" data-label="Description">Same as %y but uses the year number in [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) week date.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%V</td>
    <td scope="row" data-label="Example">27</td>
    <td scope="row" data-label="Description">Same as %U but uses the week number in [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) week date (01 to 53).</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%j</td>
    <td scope="row" data-label="Example">189</td>
    <td scope="row" data-label="Description">Day of the year (001 to 366), zero-padded to 3 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%D</td>
    <td scope="row" data-label="Example">07/08/01</td>
    <td scope="row" data-label="Description">Month-day-year format. Same as %m/%d/%y.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%x</td>
    <td scope="row" data-label="Example">07/08/01</td>
    <td scope="row" data-label="Description">Locale's date representation.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%F</td>
    <td scope="row" data-label="Example">2001-07-08</td>
    <td scope="row" data-label="Description">Year-month-day format ([RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339)). Same as %Y-%m-%d.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%v</td>
    <td scope="row" data-label="Example">8-Jul-2001</td>
    <td scope="row" data-label="Description">Day-month-year format. Same as %e-%b-%Y.</td>
  </tr>
</tbody>
</table>

### Time formatters

<table>
<thead>
  <tr>
    <th scope="col">Specifier</th>
    <th scope="col">Example</th>
    <th scope="col">Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td scope="row" data-label="Specifier">%H</td>
    <td scope="row" data-label="Example">00</td>
    <td scope="row" data-label="Description">Hour number (00 to 23), zero-padded to 2 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%k</td>
    <td scope="row" data-label="Example">0</td>
    <td scope="row" data-label="Description">Same as %H but space-padded. Same as %_H.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%I</td>
    <td scope="row" data-label="Example">12</td>
    <td scope="row" data-label="Description">Hour number in 12-hour clocks (01 to 12), zero-padded to 2 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%l</td>
    <td scope="row" data-label="Example">12</td>
    <td scope="row" data-label="Description">Same as %I but space-padded. Same as %_I.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%P</td>
    <td scope="row" data-label="Example">am</td>
    <td scope="row" data-label="Description">am or pm in 12-hour clocks.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%p</td>
    <td scope="row" data-label="Example">AM</td>
    <td scope="row" data-label="Description">AM or PM in 12-hour clocks.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%M</td>
    <td scope="row" data-label="Example">34</td>
    <td scope="row" data-label="Description">Minute number (00 to 59), zero-padded to 2 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%S</td>
    <td scope="row" data-label="Example">60</td>
    <td scope="row" data-label="Description">Second number (00 to 60), zero-padded to 2 digits.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%f</td>
    <td scope="row" data-label="Example">026490000</td>
    <td scope="row" data-label="Description">The fractional seconds (in nanoseconds) since last whole second.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%.f</td>
    <td scope="row" data-label="Example">.026490</td>
    <td scope="row" data-label="Description">Similar to %f but left-aligned.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%.3f</td>
    <td scope="row" data-label="Example">.026</td>
    <td scope="row" data-label="Description">Similar to .%f but left-aligned but fixed to a length of 3.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%.6f</td>
    <td scope="row" data-label="Example">.026490</td>
    <td scope="row" data-label="Description">Similar to .%f but left-aligned but fixed to a length of 6.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%.9f</td>
    <td scope="row" data-label="Example">.026490000</td>
    <td scope="row" data-label="Description">Similar to .%f but left-aligned but fixed to a length of 9.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%3f</td>
    <td scope="row" data-label="Example">026</td>
    <td scope="row" data-label="Description">Similar to %.3f but without the leading dot.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%6f</td>
    <td scope="row" data-label="Example">026490</td>
    <td scope="row" data-label="Description">Similar to %.6f but without the leading dot.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%9f</td>
    <td scope="row" data-label="Example">026490000</td>
    <td scope="row" data-label="Description">Similar to %.9f but without the leading dot.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%R</td>
    <td scope="row" data-label="Example">00:34</td>
    <td scope="row" data-label="Description">Hour-minute format. Same as %H:%M.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%T</td>
    <td scope="row" data-label="Example">00:34:59</td>
    <td scope="row" data-label="Description">Hour-minute-second format. Same as %H:%M:%S.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%X</td>
    <td scope="row" data-label="Example">00:34:59</td>
    <td scope="row" data-label="Description">Locale's time representation.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%r</td>
    <td scope="row" data-label="Example">12:34:59 AM</td>
    <td scope="row" data-label="Description">Hour-minute-second format in 12-hour clocks. Same as %I:%M:%S %p.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%x</td>
    <td scope="row" data-label="Example">07/08/01</td>
    <td scope="row" data-label="Description">Locale's date representation.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%F</td>
    <td scope="row" data-label="Example">2001-07-08</td>
    <td scope="row" data-label="Description">Year-month-day format ([RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339)). Same as %Y-%m-%d.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%v</td>
    <td scope="row" data-label="Example">8-Jul-2001</td>
    <td scope="row" data-label="Description">Day-month-year format. Same as %e-%b-%Y.</td>
  </tr>
</tbody>
</table>

### Timezones formatters

<table>
  <thead>
    <tr>
      <th scope="col">Specifier</th>
      <th scope="col">Example</th>
      <th scope="col">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td scope="row" data-label="Specifier">%Z</td>
      <td scope="row" data-label="Example">ACST</td>
      <td scope="row" data-label="Description">Local time zone name.</td>
    </tr>
    <tr>
      <td scope="row" data-label="Specifier">%z</td>
      <td scope="row" data-label="Example">+0930</td>
      <td scope="row" data-label="Description">Offset from the local time to UTC (with UTC being +0000).</td>
    </tr>
    <tr>
      <td scope="row" data-label="Specifier">%:z</td>
      <td scope="row" data-label="Example">+09:30</td>
      <td scope="row" data-label="Description">Same as %z but with a colon.</td>
    </tr>
  </tbody>
</table>

### Date & time formatters

<table>
<thead>
  <tr>
    <th scope="col">Specifier</th>
    <th scope="col">Example</th>
    <th scope="col">Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td scope="row" data-label="Specifier">%c</td>
    <td scope="row" data-label="Example">Sun Jul 8 00:34:59 2001</td>
    <td scope="row" data-label="Description">Locale's date and time.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%+</td>
    <td scope="row" data-label="Example">2001-07-08T00:34:59.026490+09:30</td>
    <td scope="row" data-label="Description">[RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) / RFC 3339 date &amp; time format.</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%s</td>
    <td scope="row" data-label="Example">994518299</td>
    <td scope="row" data-label="Description">UNIX timestamp, the number of seconds since 1970-01-01T00:00:00.</td>
  </tr>
</tbody>
</table>

### Other formatters

<table>
<thead>
  <tr>
    <th scope="col">Specifier</th>
    <th scope="col">Example</th>
    <th scope="col">Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td scope="row" data-label="Specifier">%t</td>
    <td scope="row" data-label="Example">-</td>
    <td scope="row" data-label="Description">Literal tab (\t).</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%n</td>
    <td scope="row" data-label="Example">-</td>
    <td scope="row" data-label="Description">Literal newline (\n).</td>
  </tr>
  <tr>
    <td scope="row" data-label="Specifier">%%</td>
    <td scope="row" data-label="Example">-</td>
    <td scope="row" data-label="Description">Literal percent sign.</td>
  </tr>
</tbody>
</table>

## Examples

Seeing if an input with a date and time conforms to an expected format:

```surql
/**[test]

[[test.results]]
value = "true"

*/

string::is_datetime("5sep2024pm012345.6789", "%d%b%Y%p%I%M%S%.f");
```

```surql title="Response"
true
```

Another example with a different format:

```surql
/**[test]

[[test.results]]
value = "false"

*/

string::is_datetime("23:56:00 2015-09-05", "%Y-%m-%d %H:%M");
```

```surql title="Response"
false
```

Using a formatter to generate a string from a datetime:

```surql
/**[test]

[[test.results]]
value = "'2021-11-01'"

*/

time::format(d"2021-11-01T08:30:17+00:00", "%Y-%m-%d");
```

```surql title="Response"
"2021-11-01"
```

## futures.mdx

---
sidebar_position: 9
sidebar_label: Futures
title: Futures | SurrealQL
description: Futures are values which are only computed when the data is selected and returned to the client.

---

import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

# Futures

> [!NOTE]
> The `future` type is only available up to SurrealDB 2.x. Since version 3.0.0-beta, it has been replaced by [defined fields using the `COMPUTED` clause](/docs/surrealql/statements/define/field#computed-fields). Most examples in this page include an equivalent `COMPUTED` clause to aid in migrating to the new implementation.

Futures are values which are only computed when the data is selected and returned to the client. Futures can be stored inside records, to enable dynamic values which are always calculated when queried.

## Simple futures

Any value or expression can be used inside a future. This value will be dynamically computed on every access to the record.

<Tabs>
<TabItem label="Legacy future type">

```surql
CREATE person SET accessed_date = <future> { time::now() };
```

</TabItem>

<TabItem label="With COMPUTED clause">

```surql
-- Only used inside a DEFINE FIELD statement
DEFINE FIELD accessed_date ON person COMPUTED time::now();
```

</TabItem>
</Tabs>

## Futures inside schema definitions

A future can be added to a schema definition as well.

<Tabs>
<TabItem label="Legacy future type">

```surql
DEFINE FIELD accessed_at ON TABLE user VALUE <future> { time::now() };

CREATE user:one;
SELECT * FROM ONLY user:one;
-- Sleep for one second
SLEEP 1s;
-- `accessed_at` is a different value now
SELECT * FROM ONLY user:one;
```

</TabItem>

<TabItem label="With COMPUTED clause">

```surql
DEFINE FIELD accessed_at ON TABLE user COMPUTED time::now();

CREATE user:one;
SELECT * FROM ONLY user:one;
-- Sleep for one second
SLEEP 1s;
-- `accessed_at` is a different value now
SELECT * FROM ONLY user:one;
```

</TabItem>

</Tabs>

This differs from a `VALUE` clause which is only calculated when it is modified (created or updated), but is not recalculated during a `SELECT` query which does not modify a record.

```surql
DEFINE FIELD updated ON TABLE user VALUE time::now();

CREATE user:one;
SELECT * FROM ONLY user:one;
-- Sleep for one second
SLEEP 1s;
-- `updated` is still the same
SELECT * FROM ONLY user:one;
```

## Futures depending on statements

If the value of a future is the result of a statement, it must be wrapped in parentheses.

<Tabs>
<TabItem label="Legacy future type">

```surql
DEFINE FIELD random_movie
    ON app_screen
    VALUE <future> { (SELECT * FROM ONLY movie ORDER BY RAND() LIMIT 1) };
```

</TabItem>
<TabItem label="With COMPUTED clause">

```surql
-- No need for parentheses
DEFINE FIELD random_movie
    ON app_screen
    COMPUTED SELECT * FROM ONLY movie ORDER BY RAND() LIMIT 1;
```

</TabItem>
</Tabs>

If your statement is wrapped in parentheses, you need to access the fields using the $parent variable.

```surql
DEFINE FIELD OVERWRITE followers
    ON user
    VALUE <future> { (SELECT VALUE count FROM ONLY follower_count WHERE user = $parent.id LIMIT 1) ?? 0 };
```

## Avoiding infinite recursion

When defining a future on a field, be sure to avoid any statements that would cause infinite recursion. In the following example, the `random_friend` field is defined by a statement that uses a `SELECT` statement on all the fields of the same `person` table, one of which will also use the same `future` to compute its value.

```surql
CREATE |person:10| SET name = "Person " + <string>id.id() RETURN NONE;

DEFINE FIELD random_friend
    ON person
    VALUE <future> { (SELECT * FROM ONLY person ORDER BY RAND() LIMIT 1) };

CREATE person;
```

```surql title="Output"
'Reached excessive computation depth due to functions, subqueries, or futures'
```

A `SELECT` query that does not access the field defined by a future will avoid the infinite recursion.

```surql
CREATE |person:10| SET name = "Person " + <string>id.id() RETURN NONE;

DEFINE FIELD random_friend
    ON person
    VALUE <future> { (SELECT VALUE name FROM ONLY person ORDER BY RAND() LIMIT 1) };

CREATE person;
```

```surql title="Output"
[
	{
		id: person:4o973bouhd6xrj8l2x69,
		random_friend: 'Person imoy71qbhnsgjtczybiq'
	}
]
```

## Next steps

You've now seen how to create dynamically computed properties on records, using either simple values, and values which depend on local and remote record fields. Take a look at the next chapter to get into SurrealDB's geospatial types.


## geometries.mdx

---
sidebar_position: 10
sidebar_label: Geometries
title: Geometries | SurrealQL
description: SurrealDB makes working with GeoJSON easy, with support for Point, LineString, Polygon, MultiPoint, MultiLineString, MultiPolygon, and Collection values.

---

# Geometries

A `geometry` is a type based on the GeoJSON spec that is optimised for working with data pertaining to locations on Earth.

SurrealDB makes working with GeoJSON easy, with support for `Point`, `LineString`, `Polygon`, `MultiPoint`, `MultiLineString`, `MultiPolygon`, and `Collection` values. SurrealQL automatically detects GeoJSON objects converting them into a single data type.

<table>
<thead>
  <tr>
    <th scope="col">Type</th>
    <th scope="col">Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td scope="row" data-label="Type"><a href="#point"><code>Point</code></a></td>
    <td scope="row" data-label="Description">A geolocation point with longitude and latitude</td>
  </tr>
  <tr>
    <td scope="row" data-label="Type"><a href="#LineString"><code>LineString</code></a></td>
    <td scope="row" data-label="Description">A GeoJSON LineString value for storing a geometric path</td>
  </tr>
  <tr>
    <td scope="row" data-label="Type"><a href="#polygon"><code>Polygon</code></a></td>
    <td scope="row" data-label="Description">A GeoJSON Polygon value for storing a geometric area</td>
  </tr>
  <tr>
    <td scope="row" data-label="Type"><a href="#multipoint"><code>MultiPoint</code></a></td>
    <td scope="row" data-label="Description">A value which contains multiple geometry points</td>
  </tr>
  <tr>
    <td scope="row" data-label="Type"><a href="#multilinestring"><code>MultiLineString</code></a></td>
    <td scope="row" data-label="Description">A value which contains multiple geometry lines</td>
  </tr>
  <tr>
    <td scope="row" data-label="Type"><a href="#multipolygon"><code>MultiPolygon</code></a></td>
    <td scope="row" data-label="Description">A value which contains multiple geometry polygons</td>
  </tr>
  <tr>
    <td scope="row" data-label="Type"><a href="#collection"><code>Collection</code></a></td>
    <td scope="row" data-label="Description">A value which contains multiple different geometry types</td>
  </tr>
</tbody>
</table>

## The GeoJSON spec

There are two main points to keep in mind when creating a `geometry` type in SurrealDB. They are:

* Points are defined according to the GeoJSON spec, which specificies longitude before latitude. Many sites - including Google Maps - provide location data in the opposite order, so be sure to confirm that any data being used to create a `Point` is in the order `(longitude, latitude)`, and not the other way around.
* A `geometry` created from an object must contain a `type` field and a `coordinates` field, no more and no less.

This can be shown by calling the [`type::is_geometry()`](/docs/surrealql/functions/database/type#typeis_geometry) function on some sample objects.

```surql
-- true: has `type` and `coordinates` field with valid data
{ type: "Point", coordinates: [-0.118092, 51.509865] }.is_geometry();

-- false: lacks 'type' field
{ coordinates: [-0.118092, 51.509865] }.is_geometry();

-- false: has an extra field
{ type: "Point", coordinates: [-0.118092, 51.509865], unnecessary: "data" }.is_geometry();
```

## `Point`

The simplest form of GeoJSON that SurrealDB supports is a geolocation point. These can be written using two different formats. The first format is that of an object that matches the GeoJSON spec.

```surql
/**[test]

[[test.results]]
value = "[{ centre: (-0.118092, 51.509865), id: city:london }]"

*/

CREATE city:london SET centre = {
    type: "Point",
    coordinates: [-0.118092, 51.509865],
};
```

The other format is a simple 2-element tuple containing the longitude and the latitude of a location. This output for this format is no different from the above, and is simply a convenience due to the frequency of use of the `Point` type.

```surql
/**[test]

[[test.results]]
value = "[{ centre: (-0.118092, 51.509865), id: city:london }]"

*/

CREATE city:london SET centre = (-0.118092, 51.509865);
```

<br />

## `LineString`

A GeoJSON LineString value for storing a geometric path.

```surql
/**[test]

[[test.results]]
value = "[{ distance: { type: 'LineString', coordinates: [[-0.118092, 51.509865], [0.1785278, 51.37692386]] }, id: city:london }]"

*/

CREATE city:london SET distance = {
    type: "LineString",
    coordinates: [[-0.118092, 51.509865],[0.1785278, 51.37692386]],
};
```

<br />

## `Polygon`

A GeoJSON Polygon value for storing a geometric area.

```surql
/**[test]

[[test.results]]
value = "[{ boundary: { type: 'Polygon', coordinates: [[[-0.38314819, 51.37692386], [0.1785278, 51.37692386], [0.1785278, 51.6146057], [-0.38314819, 51.6146057], [-0.38314819, 51.37692386]]] }, id: city:london }]"

*/

CREATE city:london SET boundary = {
	type: "Polygon",
	coordinates: [[
		[-0.38314819, 51.37692386], [0.1785278, 51.37692386],
		[0.1785278, 51.61460570], [-0.38314819, 51.61460570],
		[-0.38314819, 51.37692386]
	]]
};
```

<br />

## `MultiPoint`

MultiPoints can be used to store multiple geometry points in a single value.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:tobie, locations: { type: 'MultiPoint', coordinates: [[10, 11.2], [10.5, 11.9]] } }]"

*/

CREATE person:tobie SET locations = {
	type: "MultiPoint",
	coordinates: [
		[10.0, 11.2],
		[10.5, 11.9]
	],
};
```

<br />

## `MultiLineString`

A MultiLineString can be used to store multiple geometry lines in a single value.

```surql
/**[test]

[[test.results]]
value = "[{ id: travel:yellowstone, routes: { type: 'MultiLineString', coordinates: [[[10, 11.2], [10.5, 11.9]], [[11, 12.2], [11.5, 12.9], [12, 13]]] } }]"

*/

CREATE travel:yellowstone SET routes = {
	type: "MultiLineString",
	coordinates: [
		[ [10.0, 11.2], [10.5, 11.9] ],
		[ [11.0, 12.2], [11.5, 12.9], [12.0, 13.0] ]
	]
}
```

<br />

## `MultiPolygon`

MultiPolygons can be used to store multiple geometry polygons in a single value.

```surql
/**[test]

[[test.results]]
value = "[{ id: university:oxford, locations: { type: 'MultiPolygon', coordinates: [[[[10, 11.2], [10.5, 11.9], [10.8, 12], [10, 11.2]]], [[[9, 11.2], [10.5, 11.9], [10.3, 13], [9, 11.2]]]] } }]"

*/

CREATE university:oxford SET locations = {
	type: "MultiPolygon",
	coordinates: [
		[
			[ [10.0, 11.2], [10.5, 11.9], [10.8, 12.0], [10.0, 11.2] ]
		],
		[
			[ [9.0, 11.2], [10.5, 11.9], [10.3, 13.0], [9.0, 11.2] ]
		]
	]
};
```

<br />

## `Collection`

Collections can be used to store multiple different geometry types in a single value.

```surql
/**[test]

[[test.results]]
value = "[{ buildings: { type: 'GeometryCollection', geometries: [{ type: 'MultiPoint', coordinates: [[10, 11.2], [10.5, 11.9]] }, { type: 'Polygon', coordinates: [[[-0.38314819, 51.37692386], [0.1785278, 51.37692386], [0.1785278, 51.6146057], [-0.38314819, 51.6146057], [-0.38314819, 51.37692386]]] }, { type: 'MultiPolygon', coordinates: [[[[10, 11.2], [10.5, 11.9], [10.8, 12], [10, 11.2]]], [[[9, 11.2], [10.5, 11.9], [10.3, 13], [9, 11.2]]]] }] }, id: university:oxford }]"

*/

CREATE university:oxford SET buildings = {
	type: "GeometryCollection",
	geometries: [
		{
			type: "MultiPoint",
			coordinates: [
				[10.0, 11.2],
				[10.5, 11.9]
			],
		},
		{
			type: "Polygon",
			coordinates: [[
				[-0.38314819, 51.37692386], [0.1785278, 51.37692386],
				[0.1785278, 51.61460570], [-0.38314819, 51.61460570],
				[-0.38314819, 51.37692386]
			]]
		},
		{
			type: "MultiPolygon",
			coordinates: [
				[
					[ [10.0, 11.2], [10.5, 11.9], [10.8, 12.0], [10.0, 11.2] ]
				],
				[
					[ [9.0, 11.2], [10.5, 11.9], [10.3, 13.0], [9.0, 11.2] ]
				]
			]
		}
	]
};
```

<br />

## Example

The following example includes five records from [an open database](https://public.opendatasoft.com/explore/dataset/geonames-all-cities-with-a-population-1000/export/?disjunctive.cou_name_en&sort=name) with cities worldwide that have of a population of at least 1000. The queries below create a `city` record from each entry that includes their name, location, and name. Next, it uses the [`geo::distance`](/docs/surrealql/functions/database/geo#geodistance) function to find their two closest neighbours, relating them via the `close_to` relation table. The final query can be viewed in traditional form to see each city's neighbours, or on Surrealist's [graph view](/blog/visualising-your-data-with-surrealists-graph-view) to see a visual representation of the network of closely linked cities.

```surql
DEFINE TABLE city SCHEMAFULL;
DEFINE FIELD name ON city TYPE string;
DEFINE FIELD location ON city TYPE point;

FOR $city IN [{"geoname_id": "5881639", "name": "100 Mile House", "ascii_name": "100 Mile House", "feature_class": "P", "feature_code": "PPL", "country_code": "CA", "cou_name_en": "Canada", "country_code_2": null, "admin1_code": "02", "admin2_code": "5941", "admin3_code": "5941005", "admin4_code": null, "population": 1980, "elevation": null, "dem": 928, "timezone": "America/Vancouver", "modification_date": "2019-11-26", "label_en": "Canada", "coordinates": {"lon": -121.28594, "lat": 51.64982}},{"geoname_id": "5896969", "name": "Beaverlodge", "ascii_name": "Beaverlodge", "feature_class": "P", "feature_code": "PPL", "country_code": "CA", "cou_name_en": "Canada", "country_code_2": null, "admin1_code": "01", "admin2_code": "4819009", "admin3_code": null, "admin4_code": null, "population": 2219, "elevation": null, "dem": 723, "timezone": "America/Edmonton", "modification_date": "2024-02-28", "label_en": "Canada", "coordinates": {"lon": -119.43605, "lat": 55.21664}},{"geoname_id": "5911606", "name": "Burnaby", "ascii_name": "Burnaby", "feature_class": "P", "feature_code": "PPLA3", "country_code": "CA", "cou_name_en": "Canada", "country_code_2": null, "admin1_code": "02", "admin2_code": "5915", "admin3_code": "5915025", "admin4_code": null, "population": 202799, "elevation": null, "dem": 87, "timezone": "America/Vancouver", "modification_date": "2019-02-26", "label_en": "Canada", "coordinates": {"lon": -122.95263, "lat": 49.26636}},{"geoname_id": "5920996", "name": "Chertsey", "ascii_name": "Chertsey", "feature_class": "P", "feature_code": "PPL", "country_code": "CA", "cou_name_en": "Canada", "country_code_2": null, "admin1_code": "10", "admin2_code": "14", "admin3_code": "62047", "admin4_code": null, "population": 4836, "elevation": null, "dem": 251, "timezone": "America/Toronto", "modification_date": "2016-06-22", "label_en": "Canada", "coordinates": {"lon": -73.89095, "lat": 46.07109}},{"geoname_id": "5941905", "name": "Dorset Park", "ascii_name": "Dorset Park", "alternate_names": null, "feature_class": "P", "feature_code": "PPLX", "country_code": "CA", "cou_name_en": "Canada", "country_code_2": null, "admin1_code": "08", "admin2_code": "3520", "admin3_code": null, "admin4_code": null, "population": 25003, "elevation": null, "dem": 164, "timezone": "America/Toronto", "modification_date": "2020-05-02", "label_en": "Canada", "coordinates": {"lon": -79.28215, "lat": 43.75386}}]

{
    CREATE type::record("city", <int>$city.geoname_id) SET
		location = <point>[$city.coordinates.lon, $city.coordinates.lat],
		name = $city.name;        
};

FOR $city IN SELECT * FROM city {
    LET $this_location = $city.location;
    LET $closest = 
		(SELECT id, location, geo::distance($this_location, location) AS distance FROM city
	ORDER BY distance ASC
	LIMIT 3
		).filter(|$c| $c.distance != 0);
    FOR $closest IN $closest {
      RELATE $city->close_to->$closest SET
	  	distance = geo::distance($city.location, $closest.location);
    };
};

SELECT name, id, ->close_to->city AS neighbours FROM city;
```

## Next steps

You've now seen how to use geometries to store locations, paths, and polygonal areas in SurrealDB. For more advanced functionality, take a look at the [operators](/docs/surrealql/operators) and [geo](/docs/surrealql/functions/database/geo) functions, which enable area, distance, and bearing geometric calculations, and the ability to detect whether geometries contain or intersect other geometry types.


## idioms.mdx

---
sidebar_position: 24
sidebar_label: Idioms
title: Idioms | SurrealQL
description: Accessing and manipulating data using idioms (paths) in SurrealQL.
---

import Since from '@components/shared/Since.astro'
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

# Idioms

Idioms in SurrealQL provide a powerful and flexible way to access and manipulate data within records using paths. They allow you to navigate through nested data structures, access fields, array elements, call methods, and perform complex queries with ease. Idioms are similar to expressions in other query languages that provide a path to data within documents or records.

An idiom is composed of a sequence of **parts** that define the path to a value within a record or data structure. Each part specifies how to navigate to the next piece of data. Idioms can be used in various parts of SurrealQL. The most common usecase is in data retrival queries such as `SELECT` statements, but they can also be used in the `WHERE` clause, `SET` clause, and more.

An idiom is made up of one or more **parts**, each of which can be one of several types:

- [**Field**](#field-access): Access a field by name.
- [**Index**](#index-access): Access an element of an array by its index.
- [**All**](#all-elements): Access all elements or fields.
- [**Last**](#last-element): Access the last element of an array.
- [**Where**](#where-filter): Filter elements based on a condition.
- [**Method**](#method-chaining): Call a method on the current data.
- [**Graph**](#graph-navigation): Navigate through graph relationships.
- [**Destructure**](#destructuring): Destructure nested objects.
- [**Optional**](#optional-parts): Indicate that the following part is optional.
- [**Recurse**](#recursive-paths): Recursively traverse paths such as graph and record links.

In this section, we'll explore each part in detail with examples to help you understand how to use idioms in SurrealQL.

## Field Access

Since SurrealDB is a document database at its core, each record is stored on an underlying key-value store storage engine with the ability to store arbitrary arrays, objects, and many other types of data. To access a field in an object, use a dot `.` followed by the field name.

This is mostly helpful when accessing fields within a record, but can also be used to access fields within an array.

For example, using the `CREATE` statement to add a record into the `person` table:

```surql title="Query"
/**[test]

[[test.results]]
value = "[{ address: { city: 'New York', country: 'USA' }, age: 30, id: person:oakmq2g0njddr5ysmils, name: 'John Doe' }]"
skip-record-id-key = true

*/

CREATE person CONTENT {
    name: "John Doe",
    age: 30,
    address: {
      city: "New York",
      country: "USA"
    }
};
```

```surql title="Response"
[
	{
		address: {
			city: 'New York',
			country: 'USA'
		},
		age: 30,
		id: person:g87bnds1gcgrnoj4p5q3,
		name: 'John Doe'
	}
]
```

To access the `city` field within the `address` object, you can use the following idiom:

```surql title="Query"
SELECT address.city FROM person;
```

```surql title="Response"
[
  {
    "address": {
      "city": "New York"
    }
  }
]
```

In this example, `person.name` is an idiom that accesses the `name` field of the `person` record.

## Index Access

To access an element in an array by its index, use square brackets `[]` with the index inside. For example, let's say we have a `school` record with some student results.

```surql title="Query"
/**[test]

[[test.results]]
value = "[{ id: student:m1rdvvs0el37kiifume4, results: [{ date: '2017-06-18T08:00:00Z', name: 'Algorithmics', score: 76 }, { date: '2018-03-21T08:00:00Z', name: 'Concurrent Programming', score: 83 }, { date: '2018-09-17T08:00:00Z', name: 'Advanced Computer Science 101', score: 69 }, { date: '2019-04-20T08:00:00Z', name: 'Distributed Databases', score: 73 }] }]"
skip-record-id-key = true

*/

CREATE student SET results = [
	{ score: 76, date: "2017-06-18T08:00:00Z", name: "Algorithmics" },
	{ score: 83, date: "2018-03-21T08:00:00Z", name: "Concurrent Programming" },
	{ score: 69, date: "2018-09-17T08:00:00Z", name: "Advanced Computer Science 101" },
	{ score: 73, date: "2019-04-20T08:00:00Z", name: "Distributed Databases" },
];
```

```surql title="Response"
[
	{
		id: student:urxaykt4qkbr8rs2o68j,
		results: [
			{
				date: '2017-06-18T08:00:00Z',
				name: 'Algorithmics',
				score: 76
			},
			{
				date: '2018-03-21T08:00:00Z',
				name: 'Concurrent Programming',
				score: 83
			},
			{
				date: '2018-09-17T08:00:00Z',
				name: 'Advanced Computer Science 101',
				score: 69
			},
			{
				date: '2019-04-20T08:00:00Z',
				name: 'Distributed Databases',
				score: 73
			}
		]
	}
]
```

To access the first student in the `results` array, you can use the following idiom:


```surql
SELECT results[0].score FROM student;
```

```surql title="Response"
[
  {
    results: [
      { score: 76 }
    ]
  }
]
```

Here, `results[0].score` accesses the score of the first student in the `results` array.

## All Elements

To access all elements in an array or all fields in an object, use `.*`. This is useful when you want to access all the elements in an array or all the fields in an object.

```surql 
SELECT results.* FROM student;
```

```surql title="Response"
{
	results: [
		{
			date: '2017-06-18T08:00:00Z',
			name: 'Algorithmics',
			score: 76
		},
		{
			date: '2018-03-21T08:00:00Z',
			name: 'Concurrent Programming',
			score: 83
		},
		{
			date: '2018-09-17T08:00:00Z',
			name: 'Advanced Computer Science 101',
			score: 69
		},
		{
			date: '2019-04-20T08:00:00Z',
			name: 'Distributed Databases',
			score: 73
		}
	]
};
```

This idiom selects all elements in the `score` array.

The `.*` idiom is often seen in definitions and error messages.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ events: {  }, fields: { friends: 'DEFINE FIELD friends ON person TYPE array<record<person>> PERMISSIONS FULL', "friends.*": 'DEFINE FIELD friends.* ON person TYPE record<person> PERMISSIONS FULL' }, indexes: {  }, lives: {  }, tables: {  } }"

*/

DEFINE FIELD friends ON TABLE person TYPE array<record<person>>;
INFO FOR TABLE person;
```

The output for `INFO FOR TABLE person` includes an automatically generated definition for `friends.*`, namely every item inside the `friends` field.

```surql
{
	events: {},
	fields: {
		friends: 'DEFINE FIELD friends ON person TYPE array<record<person>> PERMISSIONS FULL',
		"friends.*": 'DEFINE FIELD friends.* ON person TYPE record<person> PERMISSIONS FULL'
	},
	indexes: {},
	lives: {},
	tables: {}
};
```

### Using `.*` to return values

<Since v="v2.1.0" />

The `.*` idiom in SurrealDB allows you to target all values in an object or all entries in an array. It can be used in various contexts such as querying, field definitions, and data manipulation. This section explains the behavior of `.*` with practical examples.

### Accessing all values in an object

When applied to an object, `.*` returns an array containing all the values of the object's properties.

```surql
/**[test]

[[test.results]]
value = "{ a: 1, b: 2 }"

*/
 { a: 1, b: 2 }.*;
```

<Tabs>
<TabItem value = "SurrealDB 2.x" label="SurrealDB 2.x">

```surql title="Response"
[1, 2]
```

</TabItem>

<TabItem value = "SurrealDB 3.x" label="SurrealDB 3.x">

```surql title="Response"
{ a: 1, b: 2 }
```

</TabItem>

</Tabs>


To see just the values of this object, the [`object::values()`](/docs/surrealql/functions/database/object#objectvalues) function can be used.

```surql
/**[test]

[[test.results]]
value = "[1, 2]"

*/

{ a: 1, b: 2 }.values();

-- [1, 2]
```

#### Defining Fields with `.*`

You can define fields using `.*` to specify constraints or types for all properties within an object field.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

DEFINE FIELD obj ON test TYPE object;
DEFINE FIELD obj.* ON test TYPE number;
```

Here, we define a field `obj` of type `object` on the `test` table, and then specify that all properties within `obj` (`obj.*`) must be of type `number`.

With this done, attempting to insert a non-number value into any property of `obj` will result in an error.

```surql
CREATE test:1 SET obj.a = 'a';

// Error
"Couldn't coerce value for field `obj.*` of `test:1`: Expected `number` but found `'a'`"
```

#### Using `.*` in Different Contexts

Depending on where `.*` is used, it can have different effects on the order of operations.

For example, if we want to return all the properties of the `person:tobie` record, we can do the following:

```surql
/**[test]

[[test.results]]
value = "{ id: person:tobie, name: 'Tobie' }"

[[test.results]]
value = "{ id: person:tobie, name: 'Tobie' }"

[[test.results]]
value = "{ id: person:tobie, name: 'Tobie' }"

[[test.results]]
value = "{ id: person:tobie, name: 'Tobie' }"

[[test.results]]
value = "{ id: person:tobie, name: 'Tobie' }"

*/

CREATE ONLY person:tobie SET name = 'Tobie';
SELECT * FROM ONLY person:tobie.*;    -- This works
SELECT * FROM ONLY (person:tobie.*);  -- Equivalent to above
SELECT * FROM ONLY { id: person:tobie, name: 'tobie' }; -- Equivalent to above
(SELECT * FROM ONLY person:tobie).*; -- Equivalent to above
```

```surql title="Output"
{
	id: person:tobie,
	name: 'tobie'
}
```

## Last Element

Addionally to access the last element of an array, use `[$]`. Refereing to the `student` record above, we can access the last element of the `results` array using the following idiom:

```surql
SELECT results[$].score FROM student;
```

```surql title="Response"
[
	{
		results: {
			score: 73
		}
	}
]
```

This idiom accesses the last element of the `score` array.

## Method chaining

<Since v="v2.0.0" />

To call a method on the current data, use a dot `.` followed by the method name and parentheses `()` with arguments. SurrealDB supports method chaining, so you can call multiple methods (functions) on the same data. Learn more about [method chaining](/docs/surrealql/functions/database#method-syntax) in the functions section.

For example, let's create a new `person` record and then call `uppercase()` on its name field.

```surql
/**[test]

[[test.results]]
value = "[{ address: { city: 'New York', country: 'USA' }, age: 30, id: person:hfmq0ckvi2u842jqqx9w, name: 'John Doe' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ address: { city: 'New York', country: 'USA' }, age: 30, id: person:hfmq0ckvi2u842jqqx9w, name: 'JOHN DOE' }]"
skip-record-id-key = true

*/

CREATE person CONTENT {
    name: "John Doe",
    age: 30,
    address: {
      city: "New York",
      country: "USA"
    }
};

SELECT *, name.uppercase() FROM person;
```

```surql title="Response"
[
  {
    "person": {
      "name": "John Doe",
      "age": 30,
      "address": {
        "city": "New York",
        "country": "USA"
      }
    }
  }
]
```

In the example above, `uppercase()` is a method called on `person.name` to convert it to uppercase. Although this method is called as `.uppercase()`, it is actually the [`string::uppercase()`](/docs/surrealql/functions/database/string#stringuppercase) function that is called.

SurrealDB will automatically recognize that the idiom part `.uppercase()` refers to the `string::uppercase()` function and call this function when the query is executed. What this means is that the following two queries are equivilent:

```surql title="Using method chaining"
SELECT *, name.uppercase() FROM person;
```

```surql title="Using function"
SELECT *, string::uppercase(name) FROM person;
```

To learn more about string method chaining in SurrealQL, see the [string functions](/docs/surrealql/functions/database/string#method-chaining) section.

## Graph Navigation

SurrealDB can also be used in the context of graph databases, where data is stored and navigated using graph traversal idioms. The [`RELATE` statement](/docs/surrealql/statements/relate) is used to create relationships between records. This allows you to traverse related records efficiently without needing to pull data from multiple tables and merging that data together using SQL JOINs.

For example, let's consider the following data:

```surql title="Create a new planet, city, and explorer records"
/**[test]

[[test.results]]
value = "[{ id: planet:unknown_planet }]"

[[test.results]]
value = "[{ id: city:el_dorado, name: 'El Dorado' }]"

[[test.results]]
value = "[{ id: explorer:drake, name: 'Drake' }]"

[[test.results]]
value = "[{ id: explorer:local_guide, name: 'Local Guide' }]"

[[test.results]]
value = "[{ id: discovered:4y6p19onn8a0zlgf80rg, in: explorer:drake, out: planet:unknown_planet }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: visited:lnjo8yonp28501y75slq, in: explorer:drake, out: city:el_dorado }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: assisted:5cr7hphseo7t2m87yhwd, in: explorer:local_guide, out: explorer:drake }]"
skip-record-id-key = true

*/

CREATE planet:unknown_planet;
CREATE city:el_dorado          SET name = "El Dorado";
CREATE explorer:drake          SET name = "Drake";
CREATE explorer:local_guide    SET name = "Local Guide";

RELATE explorer:drake->discovered->planet:unknown_planet;
RELATE explorer:drake->visited->city:el_dorado;
RELATE explorer:local_guide->assisted->explorer:drake;

```

```surql title="Retrieve all relationships from Drake"
SELECT 
    *,
    ->? AS actions,
    <-? AS was,
    <->? AS involved_in
FROM explorer:drake;
```

```surql title="Response"
[
	{
		actions: [
			discovered:sh9zbsz5u705cxv6qgoi,
			visited:hmtttiqqfa4mt9is1a7j
		],
		involved_in: [
			assisted:1pv8k3p1wpuf0guf5bvm,
			discovered:sh9zbsz5u705cxv6qgoi,
			visited:hmtttiqqfa4mt9is1a7j
		],
		id: explorer:drake,
		was: [
			assisted:1pv8k3p1wpuf0guf5bvm
		],
		name: 'Drake'
	}
]
```

Explanation:

- `*`: Selects all fields of `explorer:drake`.
- `->? AS actions`: Retrieves all outgoing relationships from Drake and aliases them as actions.
- `<-? AS was`: Retrieves all incoming relationships to Drake and aliases them as was.
- `<->? AS involved_in`: Retrieves all relationships connected to Drake, regardless of direction, and aliases them as `involved_in`.

## Destructuring

<Since v="v2.0.0" />

When working with nested data, you can destructure objects using the `.` and `{ ... }` idioms.

For example,

```surql title="Create a new person record"
/**[test]

[[test.results]]
value = "[{ age: 21, id: person:1, name: 'John', obj: { a: 1, b: 2, c: { d: 3, e: 4, f: 5 } } }]"

*/


CREATE person:1 SET name = 'John', age = 21, obj = { a: 1, b: 2, c: { d: 3, e: 4, f: 5 } };
```

```surql title="Response"
[
	{
		age: 21,
		id: person:1,
		name: 'John',
		obj: {
			a: 1,
			b: 2,
			c: {
				d: 3,
				e: 4,
				f: 5
			}
		}
	}
]
```

```surql
SELECT obj.{ a, c.{ e, f } } FROM ONLY person:1;
```

```surql title="Response"
{
	obj: {
		a: 1,
		c: {
			e: 4,
			f: 5
		}
	}
}
```

You can also OMIT fields that you don't want to destructure using the `OMIT` clause.

```surql 
SELECT * OMIT obj.c.{ d, f } FROM ONLY person:1;
```

```surql title="Response"
[
	{
		age: 21,
		id: person:1,
		name: 'John',
		obj: {
			a: 1,
			b: 2,
			c: {
				e: 4
			}
		}
	}
]
```

Extending the example in the [Graph Navigation](#graph-navigation) section, we can use the `->` idiom to navigate through the graph and destructure the `city` field.

```surql
SELECT ->visited->city.{name, id}
FROM explorer:drake;
```

```surql title="Response"
[
	{
		"->visited": {
			"->city": [
				{
					id: city:el_dorado,
					name: 'El Dorado'
				}
			]
		}
	}
]
```

### Using aliases when destructuring

The keyword `AS` is necessary inside `SELECT` statements when [using an alias](/docs/surrealql/statements/select#basic-usage) (a new name for a field).

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ location: (50, -5.4), num_people: 500 }"

*/


LET $town = {
    location: (50.0, -5.4),
    population: 500
};

SELECT 
	location,
	population AS num_people
FROM ONLY $town;
```

```surql title="Output"
{
	location: (50, -5.4),
	num_people: 500
}
```

However, as destructuring involves defining the output shape of a new object, no `AS` keyword is needed. Instead, only the names of the fields are needed. Aliasing is done by choosing a new name, a `:` (colon) and the path to the value.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ location: (50, -5.4), num_people: 500 }"

*/

LET $town = {
    location: (50.0, -5.4),
    population: 500
};

RETURN $town.{
    location,
    num_people: population
};
```

Conceptually, this is somewhat close to a `RETURN` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ location: (50, -5.4), num_people: 500 }"

*/

LET $town = {
    location: (50.0, -5.4),
    population: 500
};

RETURN {
    location: $town.location,
    num_people: $town.population,
};
```

### Destructuring the current item in a SELECT query

<Since v="v2.1.0" />

The current record in a `SELECT` query can be accessed and destructured using the `@` operator.

```surql
/**[test]

[[test.results]]
value = "[{ id: star:sun, name: 'The Sun' }]"

[[test.results]]
value = "[{ id: planet:earth, name: 'Earth' }]"

[[test.results]]
value = "[{ id: planet:earth, name: 'Earth', orbits: [star:sun] }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: planet:earth, name: 'Earth', orbits: [star:sun] }]"

[[test.results]]
value = "[{ id: planet:earth, name: 'Earth', orbits: [star:sun] }]"

*/

, [{ id: planet:earth, name: 'Earth', orbits: [star:sun] }]]
CREATE star:sun SET name = "The Sun";
CREATE planet:earth SET name = "Earth";
RELATE planet:earth->orbits->star:sun;

-- Regular SELECT query
SELECT 
    name,
    id,
    ->orbits->star AS orbits
FROM planet;

-- SELECT query using `@` and destructuring
SELECT @.{
    name,
    id,
    orbits: ->orbits->star
} FROM planet;
```

While the difference between the two methods is often cosmetic - aside from the note on aliases mentioned just above - using `@` to access the current record does lead to a different style of query that may be preferable. While a regular `SELECT` query first returns an array of results that can then be operated on, a `SELECT` query that uses `@` to access the current record can perform these operations first.

```surql
-- Use the .values() method to turn each record into
-- an array of values, then return all inside an array
SELECT @.{
    name,
    id,
    orbits: ->orbits->star
}.values()
    FROM planet;

-- Grab all records first, then use .map() to convert
-- each one into an array of values
(SELECT 
    name,
    id,
    ->orbits->star AS orbits
FROM planet)
    .map(|$obj| $obj.values());
```

Most importantly, however, the `@` operator is often necessary when using [recursive paths](#recursive-paths).

### Using expressions while destructuring

<Since v="v3.0.0" />

While the fields inside a destructuring operation have always been accessible, expressions were not. As of version `3.0.0.beta`, this limitation no longer exists.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:one, name: 'Aeon' }]"

[[test.results]]
value = "{ accessed_at: d'2025-10-02T06:18:26.751462Z', name: 'Aeon', name_length: 4 }"
skip-datetime = true

*/

CREATE person:one SET name = "Aeon";

person:one.{
    name,
	-- worked because 'name' can be accessed
    name_length: name.len(),
	-- an expression: did not work before, works now
    accessed_at: time::now()
};
```

```surql title="Output"
{
	accessed_at: d'2025-04-24T05:11:20.101Z',
	name: 'Aeon',
	name_length: 4
}
```

Expressions inside a destructuring operation have the same [predefined parameters](/docs/surrealql/parameters#parent-this) as any other expression, such as `$this` to the current object and `$parent` to the previous one.

```surql
/**[test]

[[test.results]]
value = "[{ age: 18, id: person:one }]"

[[test.results]]
value = "[{ age: 40, id: person:two }]"

[[test.results]]
value = "[{ age: 18, id: person:three }]"

[[test.results]]
value = "{ age: 18, id: person:one, same_age: [{ age: 18, id: person:one }, { age: 18, id: person:three }] }"

[[test.results]]
value = "{ age: 18, id: person:one, same_age: [{ age: 18, id: person:three }] }"

*/

CREATE person:one SET age = 18;
CREATE person:two SET age = 40;
CREATE person:three SET age = 18;

-- Find all 'person' records of the same age as 'person:one'
-- Here 'person:one' is the $parent of the inner operation
person:one.{
	id,
    age,
    same_age: SELECT * FROM person WHERE age = $parent.age
};

-- Now use array::complement to filter out the 'person:one' current record,
-- which is the parameter $this
person:one.{
	id,
    age,
    same_age: array::complement(SELECT * FROM person WHERE age = $parent.age, [$this])
};
```

```surql title="Output"
-------- Query --------

{
	age: 18,
	id: person:one,
	same_age: [
		{
			age: 18,
			id: person:one
		},
		{
			age: 18,
			id: person:three
		}
	]
}

-------- Query --------

{
	age: 18,
	id: person:one,
	same_age: [
		{
			age: 18,
			id: person:three
		}
	]
}
```

## Optional Parts

<Since v="v2.0.0" />

> [!NOTE]
> Until SurrealDB 3.0.0-beta, this operator was a single `?` question mark. Since this version, it was changed to `.?` to avoid conflicts with the `??` operator when parsing.

The `.?` operator is used to indicate that a part is optional (it may not exist) it also allows you to safely access nested data without having to check if the nested data exists and exit an idiom path early when the result is NONE.

```surql
SELECT person.spouse.?.name FROM person;
```

This idiom safely accesses `person.spouse.name` if `spouse` exists; otherwise, it returns `NONE`.

## Using Optional Parts

If some `person` records have a `spouse` field and others do not:

```surql
SELECT name, spouse.?.name AS spouse_name FROM person;
```

This idiom will return `NONE` for `spouse_name` if the `spouse` field is not present.

## Recursive paths

<Since v="v2.1.0" />

A recursive path allows record link or graph traversal down to a specified depth, as opposed to manually putting together a query to navigate down each level.

Using recursive graph traversal can be thought of as the equivalent of "show me all the third-generation descendants of Mr. Brown" as opposed to "show me the children and children's children and children's children's children of Mr. Brown".

The following shows a recursive query that returns the names of people known by records that the record `person:tobie` knows.

```surql
-- Get all names of people second to Tobie
person:tobie.{2}(->friends_with->person).name;
```

As the syntax of recursive queries tends to be complex to the untrained eye, this section will explain them in order of difficulty, beginning with what queries were necessary before recursive paths were added in SurrealDB version 2.1.

### Overview

Take the following example that creates one planet, two countries, two states/provinces in each of these countries, and two cities in each of those states/provinces. The `CREATE` statements are followed by `UPDATE` statements to set record links between them, and `RELATE` to create bidirectional graph relations between them.

```surql
CREATE
	// One planet
	planet:earth,
	// Two countries
	country:us, country:canada,
	// Four states/provinces
	state:california, state:texas,
	province:ontario, province:bc,
	// Eight cities
	city:los_angeles, city:san_francisco,
	city:houston,     city:dallas,
	city:vancouver,   city:victoria,
	city:toronto,     city:ottawa
		// Give them each names like 'earth', 'us', 'bc', etc.
	SET name = id.id();

// Record and graph links from planet to country
UPDATE planet:earth     SET next = [country:us, country:canada];
RELATE planet:earth     ->has->    [country:us, country:canada];

// Record and graph links from country to state/province
UPDATE country:us       SET next = [state:california, state:texas];
UPDATE country:canada   SET next = [province:ontario, province:bc];
RELATE country:us       ->has->    [state:california, state:texas];
RELATE country:canada   ->has->    [province:bc, province:ontario];

// Record and graph links from state/province to city
UPDATE state:california SET next = [city:los_angeles, city:san_francisco];
UPDATE state:texas      SET next = [city:houston, city:dallas];
UPDATE province:ontario SET next = [city:toronto, city:ottawa];
UPDATE province:bc      SET next = [city:vancouver, city:victoria];
RELATE state:california ->has->    [city:los_angeles, city:san_francisco];
RELATE state:texas      ->has->    [city:houston, city:dallas];
RELATE province:bc      ->has->    [city:vancouver, city:victoria];
RELATE province:ontario ->has->    [city:toronto, city:ottawa];
```

While traversing each of these paths can be done manually, it requires a good deal of typing and knowing the exact depth to traverse.

Here is an example using record links:

```surql
SELECT 
	next AS countries,
	next.next AS states_provinces,
	next.next.next AS cities
FROM planet:earth;
```

```surql title="Response"
[
	{
		cities: [
			[
				[
					city:los_angeles,
					city:san_francisco
				],
				[
					city:houston,
					city:dallas
				]
			],
			[
				[
					city:toronto,
					city:ottawa
				],
				[
					city:vancouver,
					city:victoria
				]
			]
		],
		countries: [
			country:us,
			country:canada
		],
		states_provinces: [
			[
				state:california,
				state:texas
			],
			[
				province:ontario,
				province:bc
			]
		]
	}
]
```

And here is an example using graph links.

```surql
SELECT 
	-- Show all `country` records located at `out`
	->has->country AS countries,
	-- Show all `province` or `state` records located at `out`	
	->has->country->has->(province, state) AS state_provinces,
	-- Or use (?) to show any type of record located at `out`
	->has->(?)->has->(?)->has->(?) AS cities
FROM planet:earth;
```

```surql title="Output"
[
	{
		cities: [
			city:toronto,
			city:ottawa,
			city:vancouver,
			city:victoria,
			city:dallas,
			city:houston,
			city:los_angeles,
			city:san_francisco
		],
		countries: [
			country:canada,
			country:us
		],
		state_provinces: [
			province:ontario,
			province:bc,
			state:texas,
			state:california
		]
	}
]
```

### Basics of recursive paths

Using a recursive path allows you to instead set the number of steps to follow instead of manually typing. A recursive path is made by isolating `{}` braces in between two dots, inside which the number of steps is indicated.

```surql
-- Two steps down the record links at the `next` field
planet:earth.{2}.next;
-- Two steps down the `has` graph relation
planet:earth.{2}->has->(?);
```

```surql title="Output"
[
	state:california,
	state:texas,
	province:ontario,
	province:bc
]
```

The number of steps can be any integer from 1 to 256.

```surql
-- 'Found 0 for bound but expected at least 1.'
planet:earth.{0}->has->(?);
-- 'Found 500 for bound but expected 256 at most.'
planet:earth.{500}->has->(?);
```

A range can be inserted into the braces to indicate a desired minimum and maximum depth.

```surql
-- Returns [] because no 4th-level relations exist
planet:earth.{4}->has->(?);
-- Returns `city` records located at depth 3
planet:earth.{1..4}->has->(?);
-- Open-ended range: also returns `city` records at depth 3
planet:earth.{..}->has->(?);
```

```surql title="Output"
[
	city:toronto,
	city:ottawa,
	city:vancouver,
	city:victoria,
	city:dallas,
	city:houston,
	city:los_angeles,
	city:san_francisco
]
```

### Using () to provide instructions at each depth

Parentheses can be added to a recursive query. To explain their use, consider the following example that attempts to traverse up to a depth of 3 and return the `name` of the records at that level.

```surql
planet:earth.{1..3}->has->(?).name;
```

Unfortunately, the output shows that the query stopped at a depth of one. This is because the query is instructing the database to recurse the entire `->has->(?).name` path between 1 and 3 times, but after the first recursion it has reached a string. And a string on its own is of no use in a `->has->(?)` graph query which expects a record ID.

```surql title="Output"
[
	'canada',
	'us'
]
```

In fact, the above query is equivalent to the following statement which encloses `->has->(?).name` in parentheses.

```surql
planet:earth.{1..3}(->has->(?).name);
```

To make the query work, we can shrink the area enclosed in the parentheses to `->has->(?)`, isolating the part to recurse before moving on to `.name`. It will repeat as many times as instructed and only then move on to the `name` field.

```surql
planet:earth.{1..3}(->has->(?)).name;
```

```surql title="Output"
[
	'toronto',
	'ottawa',
	'vancouver',
	'victoria',
	'dallas',
	'houston',
	'los_angeles',
	'san_francisco'
]
```

The syntax for the query above can be broken down as follows.

```surql
-- starting point
planet:earth
-- desired depth
	.{1..3}
-- instructions for current document
	(->has->(?))
-- leftover idiom path
	.{name, id};
```

### Using `@` to refer to the current record

The `@` symbol is used in recursive queries to refer to the current document. This is needed in recursive `SELECT` queries, as without it there is no way to know the context.

```surql title="Unparsable queries"
-- Parse error: what is the `.` referring to?
-- DB: "Call recursive query on a `planet`? Its `name` field? Something else?"
SELECT .{1..3}(->has->(?)) FROM planet;

-- A similar query that can't be parsed
-- DB: "Call .len() on what?"
SELECT .len() FROM planet;
```

Adding `@` allows the parser to know that the current `planet` record is the starting point for the rest of the query.

```surql title="Parsable queries"
-- Will now call `.{1..3}(has->(?))` on every planet record it finds
SELECT @.{1..3}(->has->(?)) AS cities FROM planet;
-- Will now call `.len()` on every `name` field it finds
SELECT name.len()           AS length FROM planet;
```

### Using `{}` and `.@` to combine results

Inside the structure of a recursive graph query, the `@` symbol is used in the form of `.@` at the end of a path to inform the database that this is the path to be repeated during the recursion. This allows not just the fields on the final depth of the query to be returned, but each one along the way as well.

```surql
planet:earth
	.{1..2}
	.{
		name, 
		id,
-- Query with ->has->(?) on the current record
		contains: ->has->(?).@
	};
```

```surql title="Output"
{
	contains: [
		{
			contains: [
				province:ontario,
				province:bc
			],
			id: country:canada,
			name: 'canada'
		},
		{
			contains: [
				state:texas,
				state:california
			],
			id: country:us,
			name: 'us'
		}
	],
	id: planet:earth,
	name: 'earth'
}
```

The following two rules of thumb are a good way to understand how the syntax inside the structure of the query.

- The individual fields inside a recursive query are simply populated at each point,
- The field with `.@` is used as the gateway to the next depth.

To see this visually, here is the unfolded output of the query above. The `name` and `id` fields appear at each point, while `contains` is used to move on to the next depth.

```surql
-- Original query
planet:earth.{1..2}.{ name, id, contains: ->has->(?).@ };

-- Unfolds to:
planet:earth
	.{
		name, 
		id,
		contains: ->has->(?).{
		  name, 
		  id,
		  contains: ->has->(?)
	    }
	};
```

Similarly, only one `.@` can be present inside such a query, as this is the path that is used to follow the recursive query until the end.

```surql
planet:earth
	.{1..2}
	.{
		name, 
		id,
-- Query with ->has->(?) on the current record
		contains: ->has->(?).@,
        contains2: ->has->(?).@
	};
```

```surql
'Tried to use a `@` repeat recurse symbol in a position where it is not supported'
```

Here are some more simple examples of recursive queries and notes on the output they generate.

```surql
/**[test]

[[test.results]]
value = "[{ friends: [person:jaime, person:micha], id: person:tobie, name: 'Tobie' }, { friends: [person:mary], id: person:jaime, name: 'Jaime' }, { friends: [person:john], id: person:micha, name: 'Micha' }, { id: person:john, name: 'John' }, { id: person:mary, name: 'Mary' }, { id: person:tim, name: 'Tim' }]"

[[test.results]]
value = "[{ id: knows:1, in: person:tobie, out: person:jaime }, { id: knows:2, in: person:tobie, out: person:micha }, { id: knows:3, in: person:micha, out: person:john }, { id: knows:4, in: person:jaime, out: person:mary }, { id: knows:5, in: person:mary, out: person:tim }]"

[[test.results]]
value = "['Tim']"

[[test.results]]
value = "['Tim']"

[[test.results]]
value = "['Tim']"

[[test.results]]
value = "['Tim']"

[[test.results]]
value = "{ connections: [{ connections: [{ connections: [{ connections: [], id: person:tim, name: 'Tim' }], id: person:mary, name: 'Mary' }], id: person:jaime, name: 'Jaime' }, { connections: [{ connections: [], id: person:john, name: 'John' }], id: person:micha, name: 'Micha' }], id: person:tobie, name: 'Tobie' }"

[[test.results]]
value = "[{ names_2nds: ['Tim'] }, { names_2nds: [] }, { names_2nds: ['Tim'] }, { names_2nds: ['John'] }, { names_2nds: [] }, { names_2nds: ['Tim'] }]]"

[[test.results]]
value = ""

*/

INSERT INTO person [
	{ id: person:tobie, name: 'Tobie', friends: [person:jaime, person:micha] },
	{ id: person:jaime, name: 'Jaime', friends: [person:mary] },
	{ id: person:micha, name: 'Micha', friends: [person:john] },
	{ id: person:john, name: 'John' },
	{ id: person:mary, name: 'Mary' },
	{ id: person:tim, name: 'Tim' },
];

INSERT RELATION INTO knows [
	{ id: knows:1, in: person:tobie, out: person:jaime },
	{ id: knows:2, in: person:tobie, out: person:micha },
	{ id: knows:3, in: person:micha, out: person:john },
	{ id: knows:4, in: person:jaime, out: person:mary },
	{ id: knows:5, in: person:mary, out: person:tim },
];

-- Any depth
person:tobie.{..}(->knows->person).name;

-- Minimum 2, maximum 5 iterations of recursion (or either)
person:tobie.{2..6}(->knows->person).name;
person:tobie.{2..}(->knows->person).name;
person:tobie.{..6}(->knows->person).name;

-- Generate complex recursive tree structures:
-- Fetches connections up to 3 levels deep, 
-- collecting their name, id, and connections along the way
-- 3 levels, because the first iteration is used to collect
-- the details for person:tobie
person:tobie.{..4}.{ id, name, connections: ->knows->person.@ };

-- @ is a shortcut to the current document, and acts as a shorthand to start an idiom path.
-- The "." can optionally be omitted
SELECT @{1..4}(->knows->person).name AS names_2nds FROM person;

-- Recursive idioms work with any idiom parts, not limited to graphs
-- Here, we recursively fetch friends and then collect their names
person:tobie.{1..5}(.friend).name;
```

### Behaviour of recursive queries

Recursive queries follow a few rules to determine how far to traverse and what to return. They are:

- `NONE`, `NULL`, and arrays which are empty or contain only `NONE` and/or `NULL` are considered a dead end.
- An iteration with the same value as the previous one is also considered a dead end.
- If an iteration with a dead end does not reach the minimum depth, it returns `NONE`.
- If it has already passed the minimum depth, it returns the last valid value.
- During each iteration, if it encounters an array value, all dead end values are automatically filtered out, ensuring no empty paths are included.

### Filtering recursive fields

Recursive syntax is not just useful in creating recursive queries, but parsing them as well. Take the following example that creates some `person` records, gives each of them two friends, and then traverses the `friends_with` graph for the first `person` records to find its friends, friends of friends, and friends of friends of friends. Since every level except the last contains another `connections` field, adding a `.{some_number}.connections` to a `RETURN` statement is all that is needed to drill down to a certain depth.

```surql
CREATE |person:1..21| SET name = id.id() RETURN NONE;
FOR $person IN SELECT * FROM person {
    LET $friends = (SELECT * FROM person WHERE id != $person.id ORDER BY rand() LIMIT 2);
    RELATE $person->friends_with->$friends;
};

LET $third_degree = person:1.{..3}.{ id, connections: ->friends_with->person.@ };
// Object containing array of arrays of arrays of 'person'
RETURN $third_degree;
// All connections: an array of arrays of arrays of 'person'
RETURN $third_degree.connections;
// Secondary connections: an array of arrays of 'person'
RETURN $third_degree.{2}.connections;
// Tertiary connections: an array of 'person'
RETURN $third_degree.{3}.connections;
// Tertiary connections with aliased fields and original 'person' info
RETURN $third_degree.{
		original_person: id, 
		third_degree_friends: connections.{2}.connections
};
```

Possible output of the final query:

```surql title="Output for third_degree_friends query"
{
	original_person: person:1,
	third_degree_friends: [
		person:13,
		person:3,
		person:14,
		person:10,
		person:8,
		person:3,
		person:3,
		person:14
	]
}
```

### Path and unique node collection, shortest path

<Since v="v2.2.0" />

SurrealDB has a number of built-in algorithms that allow recursive queries to collect all paths, all unique nodes, and to find the shortest path to a record. These can be used by adding the following keywords to the part of the recursive syntax that specifies the depth to recurse:

- `{..+path}`: used to collect all walked paths.
- `{..+collect}`: used to collect all unique nodes walked.
- `{..+shortest=record:id}`: used to find the shortest path to a specified record id, such as `person:tobie` or `person:one`.

The originating (first) record is excluded from these paths by default. However, it can be included by adding `+inclusive` to the syntax above.

- `{..+path+inclusive}`
- `{..+collect+inclusive}`
- `{..+shortest=record:id+inclusive}`

To demonstrate the output of these three algorithms, take the following example showing a small network of friends. The network begins with `person:you`, followed by two friends (`person:friend1`, `person:friend2`), then three acquaintances known by these friends (`person:acquaintance1`, `person:acquaintance2`, `person:acquaintance3`), and finally a movie star (`person:star`) who is known by only one of the acquaintances.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:you, name: 'you' }, { id: person:friend1, name: 'friend1' }, { id: person:friend2, name: 'friend2' }, { id: person:acquaintance1, name: 'acquaintance1' }, { id: person:acquaintance2, name: 'acquaintance2' }, { id: person:acquaintance3, name: 'acquaintance3' }, { id: person:star, name: 'star' }]"

[[test.results]]
value = "[{ id: knows:i8twxf1ghl053391z54j, in: person:you, out: person:friend1 }, { id: knows:071ho0yn3tol4kgwu0fn, in: person:you, out: person:friend2 }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: knows:8fnyab57dotlu2572uc8, in: person:friend1, out: person:friend2 }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: knows:0u34elv08i9ci7igim0f, in: person:friend2, out: person:acquaintance1 }, { id: knows:hgosmpddbv6nuohsgj8p, in: person:friend2, out: person:acquaintance2 }, { id: knows:c04oc5168ubocheihbtu, in: person:friend2, out: person:acquaintance3 }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: knows:d0s3lq6ynweqaryh567o, in: person:acquaintance3, out: person:star }]"
skip-record-id-key = true

*/

CREATE 
	person:you, 
	person:friend1, person:friend2, 
	person:acquaintance1, person:acquaintance2, person:acquaintance3, 
	person:star;

-- You have two friends
RELATE person:you->knows->[person:friend1, person:friend2];
-- The first friend is shy and only knows one other person
RELATE person:friend1->knows->person:friend2;
-- The second friend is very social and knows many people you barely know
RELATE person:friend2->knows->[person:acquaintance1, person:acquaintance2, person:acquaintance3];
-- One of those people knows the movie star
RELATE person:acquaintance3->knows->person:star;
```

This representation of this small network of friends allows us to visualize the issues that these three algorithms solve. Using `+path` will output all of the possible paths from `person:you`, `+collect` will collect all of the records in this network, and `+shortest=person:star` will find the shortest path.

```
‎
								  ┌───────►  person:friend1  
     ┌───►person:acquaintance1    │                                                                    
     │                │           │                                                
     │                │           ┼───►person:acquaintance2    person:star   
person:you            │           │                                 ▲        
     │                ▼           │                                 │        
     └────────► person:friend2────┤                                 │        
                                  └───►person:acquaintance3─────────┘                      
```

After specifying an algorithm to use, such as `{..+path}`, add the path that should be followed, in this case `->knows->person`.

#### +path

Adding `+path` will output all of the possible paths starting from `person:you`.

```surql
person:you.{..+path}->knows->person;
```

```surql title="Output"
[
	[
		person:friend2,
		person:acquaintance2
	],
	[
		person:friend2,
		person:acquaintance1
	],
	[
		person:friend1,
		person:friend2,
		person:acquaintance2
	],
	[
		person:friend1,
		person:friend2,
		person:acquaintance1
	],
	[
		person:friend2,
		person:acquaintance3,
		person:star
	],
	[
		person:friend1,
		person:friend2,
		person:acquaintance3,
		person:star
	]
]
```

#### +shortest

As the output of the previous example is fairly short, we can see that there are two ways to get from `person:one` to the movie star at `person:star`, one of which is one step shorter than the other.

To get the database to find the shortest path instead, change the algorithm to `+shortest=person:star`.

```surql
person:you.{..+shortest=person:star}->knows->person;
```

```surql title="Output"
[
	person:friend2,
	person:acquaintance3,
	person:star
]
```

The part after `+shortest` can also take a parameter if it is a record ID. The following example will return the same result as the previous one.

```surql
LET $you = SELECT VALUE id FROM ONLY person WHERE name = 'you' LIMIT 1;
LET $star = SELECT VALUE id FROM ONLY person WHERE name = 'star' LIMIT 1;
$you.{..+shortest=$star}->knows->person;
```

#### +collect

Using `+collect` will collect all of the unique collected records. As this collection is created by moving recursively one level at a time, the output will show the closest connections first and least close connections at the end.

```surql
person:you.{..+collect}->knows->person;
```

```surql title="Output"
[
	person:friend1,
	person:friend2,
	person:acquaintance2,
	person:acquaintance1,
	person:acquaintance3,
	person:star
]
```

#### +inclusive

Adding `+inclusive` will show the same output, except that the original `person:one` record will also be present.

```surql
person:you.{..+shortest=person:star+inclusive}->knows->person;
person:you.{..+collect+inclusive}->knows->person;
```

```surql title="Output"
-------- Query --------

[
	person:you,
	person:friend2,
	person:acquaintance3,
	person:star
]

-------- Query --------

[
	person:you,
	person:friend1,
	person:friend2,
	person:acquaintance2,
	person:acquaintance1,
	person:acquaintance3,
	person:star
]
```

#### Other notes

The unbounded syntax `..` can be replaced with a bounded range to ensure that the recursive query only goes down to a certain depth. For example, using `..2` with `+collect` will show all first- and second-degree relations starting from `person:you`:

```surql
person:you.{..2+collect}->knows->person;
```

```surql title="All first- and second-degree relations"
[
	person:friend1,
	person:friend2,
	person:acquaintance2,
	person:acquaintance1,
	person:acquaintance3
]
```

Doing the same with `+shortest=person:star` will return an empty array, because there is no path from `person:you` to `person:star` that only requires two hops.

```surql
person:you.{..2+shortest=person:star}->knows->person;
```

```surql title="Output"
[]
```

As shown in [a previous section](#using--to-provide-instructions-at-each-depth), parentheses can be used to show which path should be repeated during the recursion. After the path inside the parentheses, the destructuring operator, methods and so on can be used to modify the output. The query can also be written over multiple lines if desired.

```surql
-- Start with you
person:you
-- Get the shortest path
	.{..+shortest=person:star+inclusive}
-- by following ->knows->person
	(->knows->person)
-- then grab the names
	.name
-- and capitalize each one
	.map(|$n| $n.uppercase());
```

```surql title="Output"
[
	'YOU',
	'FRIEND2',
	'ACQUAINTANCE3',
	'STAR'
]
```

#### Do not use `.@` with algorithms

As these three methods use their own algorithms to follow a path, any attempt to construct your own path using `.@` will result in an error. For example, choosing `+path` along with a field `connections: ->knows->person.@` will return an error because `+path` on its own will use its own recursive planner to output every possible path as an array of arrays, while `->knowns->person.@` is an instruction to put together arrays of each record and the next result from the `->knows->person` path at any possible depth.

```surql
person:you.{..+path}.{
    id,
    connections: ->knows->person.@
};
```

```surql
'Can not construct a recursion plan when an instruction is provided'
```

Here is the output of both of these queries at a single depth to show the difference in output.

```surql
person:you.{..1}.{
    id,
    connections: ->knows->person.@
};

person:you.{..1+path}->knows->person;
```

```surql title="Output"
-------- Query --------

{
	connections: [
		person:friend2,
		person:friend1
	],
	id: person:you
}

-------- Query --------

[
	[
		person:friend2
	],
	[
		person:friend1
	]
]
```

#### Example using record links

As is the case with other recursive queries, these three algorithms can be used in the same way with any other path that can be repeated, such as record links. The following example shows the same network of friends as the one above, except that it uses record links instead of graph queries. To traverse these paths, a simple `.knows` is all that is required.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:you, knows: [person:friend1, person:friend2] }]"

[[test.results]]
value = "[{ id: person:friend1, knows: [person:friend2] }]"

[[test.results]]
value = "[{ id: person:friend2, knows: [person:acquaintance1, person:acquaintance2, person:acquaintance3] }]"

[[test.results]]
value = "[{ id: person:acquaintance1 }, { id: person:acquaintance2 }, { id: person:star }]"

[[test.results]]
value = "[{ id: person:acquaintance3, knows: [person:star] }]"

[[test.results]]
value = "[person:friend2, person:acquaintance3, person:star]"

[[test.results]]
value = "[[person:friend2, person:acquaintance1], [person:friend2, person:acquaintance2], [person:friend1, person:friend2, person:acquaintance1], [person:friend1, person:friend2, person:acquaintance2], [person:friend2, person:acquaintance3, person:star], [person:friend1, person:friend2, person:acquaintance3, person:star]]"

[[test.results]]
value = "[person:friend1, person:friend2, person:acquaintance1, person:acquaintance2, person:acquaintance3, person:star]"

*/

CREATE person:you SET knows = [person:friend1, person:friend2];
CREATE person:friend1 SET knows = [person:friend2];
CREATE person:friend2 SET knows = [person:acquaintance1, person:acquaintance2, person:acquaintance3];
CREATE person:acquaintance1, person:acquaintance2, person:star;
CREATE person:acquaintance3 SET knows = [person:star];

person:you.{..+shortest=person:star}.knows;
person:you.{..+path}.knows;
person:you.{..+collect}.knows;
```

## Combining Idiom Parts

Idioms can combine multiple parts to navigate complex data structures seamlessly.

Suppose we have the following data:

```surql title="Create a new person record"
/**[test]

[[test.results]]
value = "[{ friends: [{ age: 25, id: 'person:6', name: 'Frank' }, { age: 19, id: 'person:7', name: 'Grace' }, { age: 17, id: 'person:8', name: 'Heidi' }], id: person:5, name: 'Eve' }]"

*/

CREATE person:5 CONTENT {
    name: "Eve",
    friends: [
        {
            id: "person:6",
            name: "Frank",
            age: 25
        },
        {
            id: "person:7",
            name: "Grace",
            age: 19
        },
        {
            id: "person:8",
            name: "Heidi",
            age: 17
        }
    ]
};
```

```surql title="Response"
[
	{
		friends: [
			{
				age: 25,
				id: 'person:6',
				name: 'Frank'
			},
			{
				age: 19,
				id: 'person:7',
				name: 'Grace'
			},
			{
				age: 17,
				id: 'person:8',
				name: 'Heidi'
			}
		],
		id: person:5,
		name: 'Eve'
	}
]
```

To get the names of friends who are over 18:

```surql
SELECT friends[WHERE age > 18].name FROM person WHERE id = person:5;
```

```surql title="Response"
[
	{
		friends: {
			name: [
				'Frank',
				'Grace'
			]
		}
	}
]
```

## Notes on Idioms

- **Chaining**: Idioms can be chained to traverse deeply nested structures.
- **Performance**: Be mindful of performance when using complex idioms; indexing fields can help.
- **NONE Safety**: Use optional parts (`?`) to handle `NONE` or missing data gracefully.
- **Methods**: Leverage built-in methods for data manipulation within idioms.
- **Type Casting**: Use type casting if necessary to ensure data is in the correct format.

## Best Practices

- **Use Destructuring**: When selecting multiple fields, destructuring improves readability.
- **Limit Optional Parts**: Use optional parts judiciously to avoid masking data issues.
- **Validate Data**: Ensure data conforms to expected structures, especially when dealing with optional fields.
- **Index Fields**: Index fields that are frequently accessed or used in `WHERE` clauses for better performance.

## Summary

Idioms in SurrealQL are a powerful tool for navigating and manipulating data within your database. By understanding and effectively using idiom parts, you can write expressive and efficient queries that handle complex data structures with ease. Whether you're accessing nested fields, filtering arrays, or traversing graph relationships, idioms provide the flexibility you need to interact with your data seamlessly.


## ids.mdx

---
sidebar_position: 15
sidebar_label: Record IDs
title: Record IDs | SurrealQL
description: In SurrealDB, document record IDs store both the table name, and the record ID.

---

# Record IDs

> [!NOTE]
> As of `v2.0.0`, SurrealDB no longer eagerly converts a string into a record. An [implicit `r` prefix or cast](/docs/surrealql/datamodel/casting#casting-vs-affixes) is required instead.

SurrealDB record IDs are composed of a table name and a record identifier separated by a `:` in between, allowing for a simple and consistent way to reference records across the database. Record IDs are used to uniquely identify records within a table, to [query](/docs/surrealql/statements/select), [update](/docs/surrealql/statements/update), and [delete](/docs/surrealql/statements/delete) records, and serve as [links](/docs/surrealql/datamodel/records) from one record to another.

Record IDs can be constructed from a number of ways, including [alphanumeric text](/docs/surrealql/datamodel/ids#text-record-ids), complex Unicode text and symbols, [numbers](/docs/surrealql/datamodel/ids#numeric-record-ids), arrays, objects, [built-in ID generation functions](/docs/surrealql/datamodel/ids#random-ids), and [a function to generate an ID from values](/docs/surrealql/functions/database/type#typerecord).


All of the following are examples of valid record IDs in SurrealQL.

```surql
company:surrealdb
company:w6xb3izpgvz4n0gow6q7
reaction:`🤪`
weather:['London', d'2025-02-14T01:52:50.375Z']
```

As all record IDs are unique, trying to create a new record with an existing record ID will return an error. To create a record or modify it if the ID already exists, use an [`UPSERT`](/docs/surrealql/statements/upsert) statement or an [`INSERT`](/docs/surrealql/statements/insert#example-usage) statement with an `ON DUPLICATE KEY UPDATE` clause.

## Types of Record IDs

### Random IDs

When you [create a record](/docs/surrealql/statements/create) without specifying the full ID, a random identifier is assigned after the table name. This differs from the traditional default of auto-increment or serial IDs that many developers are used to.

```surql
/**[test]

[[test.results]]
value = "[{ id: company:igtjgekhxfbd9km14j7t }]"
skip-record-id-key = true

*/


CREATE company;
```

```surql title="Output"
[
	{
		id: company:ezs644u19mae2p68404j
	}
]
```

Record IDs can be generated with a number of built-in ID generation functions, which are cryptographically secure and suitable for dispersion across a distributed datastore. These include a 20 digit alphanumeric ID (the default), sequentially incrementing and temporally sortable ULID Record identifiers, and UUID version 7 Record identifiers.

```surql
/**[test]

[[test.results]]
value = "[{ celsius: 37.5f, id: temperature:f8xh13smeqhbg7h9o1rw, time: d'2025-10-03T01:03:27.756064Z' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ celsius: 37.5f, id: temperature:a5wxgrluu3n0qxrgkfb9, time: d'2025-10-03T01:03:27.756317Z' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ celsius: 37.5f, id: temperature:01K6KSGTGCZ631S0325WPW3KF1, time: d'2025-10-03T01:03:27.756405Z' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ celsius: 37.5f, id: temperature:u'0199a798-6a0c-7f51-8b87-e856013f98ec', time: d'2025-10-03T01:03:27.756501Z' }]"
skip-record-id-key = true

*/


// Generate a random record ID 20 characters in length
// Charset: `abcdefghijklmnopqrstuvwxyz0123456789`
CREATE temperature:rand() SET time = time::now(), celsius = 37.5;
// Identical to the above CREATE statement, because
// :rand() is the default random ID format
CREATE temperature SET time = time::now(), celsius = 37.5;

// Generate a ULID-based record ID
CREATE temperature:ulid() SET time = time::now(), celsius = 37.5;
// Generate a UUIDv7-based record ID
CREATE temperature:uuid() SET time = time::now(), celsius = 37.5;
```

### Text Record IDs

Text record IDs can contain letters, numbers and `_` characters.

```surql
/**[test]

[[test.results]]
value = "[{ id: company:surrealdb, name: 'SurrealDB' }]"

[[test.results]]
value = "[{ id: user_version_2025:aajz3qh1nk0b27tztsa7, name: 'Alucard' }]"
skip-record-id-key = true

*/

CREATE company:surrealdb SET name = 'SurrealDB';
CREATE user_version_2025 SET name = 'Alucard';
```

To create a record ID with complex characters, use <code>`</code> (backticks) around the table name and/or record identifier.

```surql
/**[test]

[[test.results]]
value = "[{ author: person:tobie, id: article:⟨8424486b-85b3-4448-ac8d-5d51083391c7⟩, time: d'2025-10-03T01:06:17.050210Z' }]"

[[test.results]]
value = "[{ author: person:⟨Lech_Wałęsa⟩, id: ⟨Artykuł⟩:100 }]"

*/

CREATE article:`8424486b-85b3-4448-ac8d-5d51083391c7` SET
    time = time::now(),
    author = person:tobie;

CREATE `Artykuł`:100 SET
    author = person:`Lech_Wałęsa`;
```

The parts of record IDs with complex characters will display enclosed by <code>`</code> backticks.

```surql title="Output"
-------- Query --------

[
	{
		author: person:tobie,
		id: article:`8424486b-85b3-4448-ac8d-5d51083391c7`,
		time: d'2025-02-18T01:48:46.364Z'
	}
]

-------- Query --------

[
	{
		author: person:`Lech_Wałęsa`,
		id: `Artykuł`:100
	}
]
```

### Numeric Record IDs

If you create a record ID with a number as a string, it will be stored with <code>`</code> backticks to differentiate it from a number.

```surql
/**[test]

[[test.results]]
value = "[{ id: article:10 }]"

[[test.results]]
value = "[{ id: article:⟨10⟩ }]"

[[test.results]]
value = "[{ id: article:article10 }]"

[[test.results]]
value = "[article:10, article:⟨10⟩, article:article10]"

*/

CREATE article SET id = 10;
CREATE article SET id = "10";
CREATE article SET id = "article10";
SELECT VALUE id FROM article;
```

As the record ID `article:10` is different from ```article:`10` ```, no errors are returned when creating and both records turn up in the output of the `SELECT` statement. Meanwhile, the article with the identifier `article10` does not use backticks as there is no `article10` number to differentiate it from.

```surql title="Output"
[
	article:10,
	article:`10`,
    article:article10
]
```

If a numeric value is specified without any decimal point suffix and is within the range `-9223372036854775808` to `9223372036854775807` then the value will be parsed, stored, and treated as a 64-bit signed integer.

Any numeric numbers outside of the range of a signed 64-bit integer will be stored as a string.

```surql
/**[test]

[[test.results]]
value = "[{ celsius: 37.5f, id: temperature:17493, time: d'2025-10-03T01:09:50.155406Z' }]"

[[test.results]]
value = "[{ events: ['Galactic senate convenes', 'Mr. Bean still waits in a field'], id: year:⟨29878977097987987979232⟩ }]"

*/

CREATE temperature:17493 SET time = time::now(), celsius = 37.5;
CREATE year:29878977097987987979232 SET
    events = [
        "Galactic senate convenes",
        "Mr. Bean still waits in a field"
    ];
```

```surql title="Output"
-------- Query --------

[
	{
		celsius: 37.5f,
		id: temperature:17493,
		time: d'2025-02-17T06:21:08.911Z'
	}
]

-------- Query s--------

[
	{
		events: [
			'Galactic senate convenes',
			'Mr. Bean still waits in a field'
		],
		id: year:`29878977097987987979232`
	}
]
```

### Array-based Record IDs

Record IDs can be constructed out of arrays and even objects. This sort of record ID is most used when you have a field or two that will be used to look up records inside a [record range](/docs/surrealql/datamodel/ids#record-ranges), which is extremely performant. This is in contrast to using a `WHERE` clause to filter, which involves a table scan.

Records in SurrealDB can store arrays of values, including other nested arrays or objects within them. Different types of values can be stored within the same array, unless defined otherwise.

```surql
/**[test]

[[test.results]]
value = "[{ conditions: 'cloudy', id: weather:['London', d'2025-02-13T05:00:00Z'], temperature: 5.7f }]"

*/

CREATE weather:['London', d'2025-02-13T05:00:00Z'] SET
    temperature = 5.7,
    conditions = "cloudy";
```

```surql title="Output"
[
	{
		conditions: 'cloudy',
		id: weather:[
			'London',
			d'2025-02-13T05:00:00Z'
		],
		temperature: 5.7f
	}
]
```

### Why record ranges are performant

The main reason why record ranges are so performant is simply because the database knows ahead of time in which area to look for records in a query, and therefore has a smaller "surface area" to work in.

This can be demonstrated by seeing what happens when a single record range query encompasses all of the records in a database. The example below creates 10,000 `player` records that have an array-based record ID that begins with `'mage'`, allowing them to be used in a record range query, as well as a field called `class` that is also `'mage'`, which will be used in a `WHERE` clause to compare performance.

Interestingly, in this case a record range query is only somewhat more performant. This is because both queries end up iterating over 10,000 records, with the only difference being that the query with a `WHERE` clause also checks to see if the value of the `class` field is equal to `'mage'`.

```surql
FOR $_ IN 0..10000 {
    CREATE player:['mage', rand::id()] SET class = 'mage';
};

LET $_ = SELECT * FROM player:['mage', NONE]..['mage', ..];
LET $_ = SELECT * FROM player WHERE class = 'mage';
```

If the number of `player` records is extended to a larger number of classes, however, the difference in performance will be much larger. In this case the record range query is still only iterating a relatively small surface area of 10,000 records, while the second one has ten times this number to go through in addition to the `WHERE` clause on top.

```surql
FOR $_ IN 0..10000 {
  CREATE player:['mage', rand::id()] SET class = 'mage';
  CREATE player:['barbarian', rand::id()] SET class = 'barbarian';
  CREATE player:['rogue', rand::id()]     SET class = 'rogue';
  CREATE player:['bard', rand::id()]      SET class = 'bard';
  CREATE player:['sage', rand::id()]      SET class = 'sage';
  CREATE player:['psionic', rand::id()]   SET class = 'psionic';
  CREATE player:['thief', rand::id()]     SET class = 'thief';
  CREATE player:['paladin', rand::id()]   SET class = 'paladin';
  CREATE player:['ranger', rand::id()]    SET class = 'ranger';
  CREATE player:['cleric', rand::id()]    SET class = 'cleric';
};

LET $_ = SELECT * FROM player:['mage', NONE]..['mage', ..];
LET $_ = SELECT * FROM player WHERE class = 'mage';
```

### IDs made with parameters and function calls

Parameters and function calls can be used inside array- and object-based record IDs in the same way as on standalone arrays and objects.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ conditions: 'cloudy', id: weather:['Seoul', d'2025-10-03T01:11:38.204589Z'], temperature: -2.3f }]"
skip-datetime = true

[[test.results]]
value = "[{ conditions: 'cloudy', id: weather:['London', d'2025-10-03T01:11:38.204961Z'], temperature: 5.3f }]"
skip-datetime = true

*/

LET $now = time::now();

CREATE weather:['Seoul', $now] SET
    temperature = -2.3,
    conditions = "cloudy";

CREATE weather:['London', time::now()] SET
    temperature = 5.3,
    conditions = "cloudy";
```

To create a record that uses a parameter or function call as its entire record identifier, the [`type::record()`](/docs/surrealql/functions/database/type#typerecord) function can be used. (Note: this function was known as `type::thing()` before SurrealDB 3.0)

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ city: 'London', id: weather:⟨2025-10-03T01:13:14.238633Z⟩ }]"
skip-record-id-key = true

*/

LET $now = time::now();

CREATE type::record("weather", $now) SET city = 'London';
```

```surql title="Output"
[
	{
		city: 'London',
		id: weather:`2025-02-18T02:30:08.563Z`
	}
]
```

## Defining record IDs in a schema

The type name of a record ID is `record`, which by default allows any sort of record. This type can be set inside a [`DEFINE FIELD`](/docs/surrealql/statements/define/field) statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ friends: [person:one, person:two], id: person:663uogu8gnw31irybeer, possessions: [book:one, house:one] }]"
skip-record-id-key = true

*/

DEFINE FIELD possessions ON TABLE person TYPE option<array<record>>;
DEFINE FIELD friends ON TABLE person TYPE option<array<record<person>>>;

CREATE person SET
    possessions = [ book:one, house:one],
    friends = [ person:one, person:two ];
```

Be sure to use just `record` instead of `record<any>`, as `<any>` here would imply actual records of a table called `any`.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
match = "$error = "Couldn't coerce value for field `possessions` of `person:*`: Expected `none | array<record<any>>` but found `[book:one, house:one]`""
error = true

[[test.results]]
value = "[{ id: person:u6qd2t4ij2h45bkf2gk4, possessions: [any:one, any:two] }]"
skip-record-id-key = true

*/

DEFINE FIELD possessions ON TABLE person TYPE option<array<record<any>>>;

-- Won't work, 'book' and 'house' are not of table 'any'
CREATE person SET
    possessions = [ book:one, house:one ];

-- Actually expects this, which is probably
-- not what the DEFINE FIELD intended
CREATE person SET
    possessions = [ any:one, any:two ];
```

## Record ranges

SurrealDB supports the ability to query a range of records, using the record ID. Record ID range queries retrieve records using the natural sorting order of the record IDs, making a table scan unnecessary. These range queries can be used to query a range of records in a timeseries context.

```surql
-- Select all person records with IDs between the given range
SELECT * FROM person:1..1000;

-- Select all records for a particular location, inclusive
SELECT * FROM temperature:['London', NONE]..=['London', ..];

-- Select all temperature records with IDs less than a maximum value
SELECT * FROM temperature:..['London', '2022-08-29T08:09:31'];

-- Select all temperature records with IDs greater than a minimum value
SELECT * FROM temperature:['London', '2022-08-29T08:03:39']..;

-- Select all temperature records with IDs between the specified range
SELECT * FROM temperature:['London', '2022-08-29T08:03:39']..['London', '2022-08-29T08:09:31'];
```

The following example shows the difference in performance between a regular query that uses a `WHERE` clause and a record range scan.

```surql
FOR $num IN 0..=100000 {
  CREATE person SET id = $num, num = $num  
};

-- Assign the output to an unused parameter
-- to avoid excessive output
LET $_ = SELECT * FROM person WHERE num IN 0..=1000;
LET $_ = SELECT * FROM person:0..=1000;
```

## Tips and best practices for record IDs

### Why choose the right record ID format

Choosing an apt record ID format is especially important because record IDs in SurrealQL are immutable. Take the following `user` records for example:

```surql
FOR $i IN 0..5 {
    CREATE user SET user_num = $i, name = "User number " + <string>user_num;
};
```

Each of these `user` records will have a random ID, such as `user:wvjqjc5ebqvfg3aw7g61`. If a decision is made to move away from random IDs to some other form, such as an incrementing number, this will have to be done manually.

```surql
FOR $user IN SELECT * FROM user {
    -- Use type::record to make a record ID
    -- from the user_num field
    CREATE type::record("user", $user.user_num);
    -- Then delete the old user
    DELETE $user;
};

SELECT * FROM user;
```

The final query returning just the IDs shows that they have been recreated with new IDs.

```surql title="Output"
[
	{
		id: user:0,
		name: 'User number 0'
	},
	{
		id: user:1,
		name: 'User number 1'
	},
	{
		id: user:2,
		name: 'User number 2'
	},
	{
		id: user:3,
		name: 'User number 3'
	},
	{
		id: user:4,
		name: 'User number 4'
	}
]
```

However, record IDs are also used as [record links](/docs/surrealql/datamodel/records) and to create [graph relations](/docs/surrealql/statements/relate). If this is the case, more work will have to be done in order to recreate the former state.

The following example shows five `user` records, which each have a 50% chance of liking each of the other users.

```surql
FOR $i IN 0..5 {
    CREATE user SET user_num = $i, name = "User number " + <string>user_num;
};

LET $users = SELECT * FROM user;
FOR $user IN $users {
    LET $others = array::complement($users, [$user.id]);
    FOR $counterpart IN $others {
        IF rand::bool() {
            RELATE $user->likes->$counterpart;
        }
    }
};
```

Finding out the current relational state can be done with a query like the following which shows all of the graph tables in which a record is located at the `in` or `out` point. The `?` is a wildcard operator, returning any and all tables found at this point of the graph query.

```surql
SELECT
    id,
    ->?->? AS did, 
    <-?<-? AS done_to
FROM user;
```

```surql title="Output"
[
	{
		did: [
			user:zwfnk4by9gmopf6eeqm0
		],
		done_to: [
			user:d6bx6sch5li8qmhq3ljl,
			user:ekovipptanvmgr8f48v6
		],
		id: user:6ycb63zr0k3cpzwel1ga
	},
	{
		did: [
			user:ekovipptanvmgr8f48v6,
			user:6ycb63zr0k3cpzwel1ga,
			user:zk7tpaduzaiuswll58sg
		],
		done_to: [],
		id: user:d6bx6sch5li8qmhq3ljl
	}
    -- and so on..
]
```

Surrealist's [graph visualization view](/blog/whats-new-in-surrealist-3-2#graph-visualisation) can help as well.

![Surrealist's graph view showing possible output from the previous randomized query in which each of the five user records may or may not like another user. In this case, the output resembles a rhombus with an extra line jutting out from the top left.](graph_view.png)

With this in mind, here are some of the items to keep in mind when deciding what sort of record ID format to use.

### Meaningful sortable IDs are faster to query

Records are returned in ascending record ID order by default. As the following query shows, a `SELECT` statement on a large number of `user` records with random IDs will show those with record identifiers starting with a large number of zeroes. While the IDs are sortable, the IDs themselves are completely random.

```surql
CREATE |user:200000| RETURN NONE;
SELECT VALUE id FROM user LIMIT 4;
```

```surql title="Output"
[
	user:0001th0nnywnczi7mrvk,
	user:000t5r3y7u8stqtecvht,
	user:000tjk1nbi1it1bedplc,
	user:001dfral92ltbdznypcd
]
```

For a large number of records, pagination can be used to retrieve a certain amount of records at a time.

```surql
-- Returns the same four records as above
SELECT VALUE id FROM user START 0 LIMIT 2;
SELECT VALUE id FROM user START 2 LIMIT 2;
```

```surql title="Output"
-------- Query --------

[
	user:0001th0nnywnczi7mrvk,
	user:000t5r3y7u8stqtecvht
]

-------- Query --------

[
	user:001dfral92ltbdznypcd,
	user:001hv9g1uzh32nophrpo
]
```

As record ranges are very performant, consider moving any fields that may be used in a `WHERE` clause into the ID itself.

In the following example, a number of `user` records are created using the default random ID, plus a `num` field that tracks in which order the user was created.

```surql
FOR $num IN 0..100 {
    CREATE user SET num = $num;
    sleep(1ms); -- Simulate a bit of time between user creation
};

SELECT * FROM user WHERE num IN 50..=51;
SELECT * FROM user START 50 LIMIT 2;
```

As the output from the `SELECT` statements show, a `WHERE` clause is needed to find two users starting at a `num` of 50, as `START 50` starts based on the user of the record ID, which is entirely random.

```surql
-------- Query --------

[
	{
		id: user:pqpeg0edt8kpda907o01,
		num: 50
	},
	{
		id: user:ty6qr7zyob5dh882it08,
		num: 51
	}
]

-------- Query --------

[
	{
		id: user:hvfp5m5ty7n2k95dbamv,
		num: 70
	},
	{
		id: user:hvfumcmmveuolg4e2h26,
		num: 36
	}
]
```

Using a ULID in this case will allow the IDs to remain random, but still sorted by date of creation.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: user:01K6KTHCCJW7ZA6NWBZEFEVDC9, num: 50 }, { id: user:01K6KTHCCNRMN565T2H6JW70D6, num: 51 }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: user:01K6KTHCCJW7ZA6NWBZEFEVDC9, num: 50 }, { id: user:01K6KTHCCNRMN565T2H6JW70D6, num: 51 }]"
skip-record-id-key = true

*/

FOR $num IN 0..100 {
    CREATE user:ulid() SET num = $num;
    sleep(1ms);
};

SELECT * FROM user WHERE num IN 50..=51;
SELECT * FROM user START 50 LIMIT 2;
```

Not only is the `START 50 LIMIT 2` query more performant, but the entire `num` field could be removed if its only use is to return records by order of creation.

```surql title="Same record IDs for both queries this time"
-------- Query --------

[
	{
		id: user:01JM1AHN7DDN7XM5KZ2RR2YM1S,
		num: 50
	},
	{
		id: user:01JM1AHN7FS4A3B6RNFCF64H90,
		num: 51
	}
]

-------- Query --------

[
	{
		id: user:01JM1AHN7DDN7XM5KZ2RR2YM1S,
		num: 50
	},
	{
		id: user:01JM1AHN7FS4A3B6RNFCF64H90,
		num: 51
	}
]
```

### Move exact matches in array-based record IDs to the front

Take the following `event` records which can be queried as a perfomant record range.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:[d'2025-05-05T08:00:00Z', user:one, 'debug'], info: 'Logged in' }]"

[[test.results]]
value = "[{ id: event:[d'2025-05-05T08:10:00Z', user:one, 'debug'], info: 'Logged out' }]"

[[test.results]]
value = "[{ id: event:[d'2025-05-05T08:01:00Z', user:two, 'debug'], info: 'Logged in' }]"

*/

CREATE event:[d'2025-05-05T08:00:00Z', user:one, "debug"] SET info = "Logged in";
CREATE event:[d'2025-05-05T08:10:00Z', user:one, "debug"] SET info = "Logged out";
CREATE event:[d'2025-05-05T08:01:00Z', user:two, "debug"] SET info = "Logged in";
```

The ordering of the ID in this case is likely not ideal, because the first item in the array, a `datetime`, will be the first to be evaluated in a range scan. A query such as the one below on a range of dates will effectively ignore the second and third parts of the ID.

```surql
SELECT * FROM event:[d'2025-05-05', user:one, "debug"]..[d'2025-05-06', user:one, "debug"];

-- Same result! user name and "debug" are irrelevant
-- SELECT * FROM event:[d'2025-05-05']..[d'2025-05-06'];
```

```surql title="Output"
[
	{
		id: event:[
			d'2025-05-05T08:00:00Z',
			user:one,
			'debug'
		],
		info: 'Logged in'
	},
	{
		id: event:[
			d'2025-05-05T08:01:00Z',
			user:two,
			'debug'
		],
		info: 'Logged in'
	},
	{
		id: event:[
			d'2025-05-05T08:10:00Z',
			user:one,
			'debug'
		],
		info: 'Logged out'
	}
]
```

Instead, the parts of the array that are more likely to be exactly matched (such as `user:one` and `"debug"`) should be moved to the front.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:[user:one, 'debug', d'2025-05-05T08:00:00Z'], info: 'Logged in' }]"

[[test.results]]
value = "[{ id: event:[user:one, 'debug', d'2025-05-05T08:10:00Z'], info: 'Logged out' }]"

[[test.results]]
value = "[{ id: event:[user:two, 'debug', d'2025-05-05T08:01:00Z'], info: 'Logged in' }]"

*/


CREATE event:[user:one, "debug", d'2025-05-05T08:00:00Z'] SET info = "Logged in";
CREATE event:[user:one, "debug", d'2025-05-05T08:10:00Z'] SET info = "Logged out";
CREATE event:[user:two, "debug", d'2025-05-05T08:01:00Z'] SET info = "Logged in";
```

Using this format, queries can now be performed for a certain user and logging level, over a range of datetimes.

```surql
-- Only returns events for user:one and "debug"
SELECT * FROM event:[user:one, "debug", d'2025-05-05']..[user:one, "debug", d'2025-05-06'];
```

```surql title="Output"
[
	{
		id: event:[
			user:one,
			'debug',
			d'2025-05-05T08:00:00Z'
		],
		info: 'Logged in'
	},
	{
		id: event:[
			user:one,
			'debug',
			d'2025-05-05T08:10:00Z'
		],
		info: 'Logged out'
	}
]
```

### Auto-incrementing IDs

While SurrealDB does not use auto-incrementing IDs by default, this behaviour can be achieved in a number of ways. One is to use the [`record::id()`](/docs/surrealql/functions/database/record#recordid) function on the latest record, which returns the latter part of a record ID (the '1' in the record ID `person:1`). This can then be followed up with the [`type::record()`](/docs/surrealql/functions/database/type#typerecord) function to create a new record ID.

```surql
-- Create records from person:1 to person:10
CREATE |person:1..11|;
LET $latest = SELECT VALUE id FROM ONLY person ORDER BY id DESC LIMIT 1;
CREATE type::record("person", $latest.id() + 1);
```

```surql title="Output"
[
	{
		id: person:11
	}
]
```

When dealing with a large number of records, a more performant option is to use a separate record that holds a single value representing the latest ID. An [`UPSERT`](/docs/surrealql/statements/upsert) statement is best here, which will allow the counter to be initialized if it does not yet exist, and updated otherwise. This is best done [inside a manual transaction](/docs/surrealql/statements/begin) so that the latest ID will be rolled back if any failures occur when creating the next record.

```surql
/**[test]

[[test.results]]
value = "[{ id: person_id:counter, num: 1 }]"

[[test.results]]
value = "[{ id: person:1 }]"

[[test.results]]
error = "'The query was not executed due to a failed transaction'"

[[test.results]]
error = ""Expected `datetime` but found a `'2025_01+01'`""

[[test.results]]
value = "1"

*/

BEGIN TRANSACTION;
UPSERT person_id:counter SET num += 1;
-- Creates a person:1
CREATE type::record("person", person_id:counter.num);
COMMIT TRANSACTION;

BEGIN TRANSACTION;
-- Latest ID is now 2
UPSERT person_id:counter SET num += 1;
-- Whoops, invalid datetime format
-- Transaction fails and all changes are rolled back
CREATE type::record("person", person_id:counter.num) SET created_at = <datetime>'2025_01+01';
COMMIT TRANSACTION;

-- Latest ID is still 1
RETURN person_id:counter.num;
```

### Record IDs are record links

As a record ID is a pointer to all of the data of a record, a single record ID is enough to access all of a record's fields. This behaviour is the key to the convenience of [record links](/docs/surrealql/datamodel/records) in SurrealDB, as holding a record ID is all that is needed for one record to have a link to another.

When using a standalone record ID as a record pointer, be sure to use the record ID itself.

```surql
/**[test]

[[test.results]]
value = "[{ data: { data: 'for', demonstration: 'purposes', some: 'demo' }, id: person:1 }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: person:1 }]"

*/

CREATE person:1 SET data = {
    some: "demo",
    data: "for",
    demonstration: "purposes"
};

LET $record = SELECT id FROM person:1;
SELECT * FROM $record;
```

The output of the above query is just the `id` field on its own, as the `$record` parameter is an object with an `id` field, not the `id` field (the pointer) itself.

```surql title="Output"
[
	{
		id: person:1
	}
]
```

To rectify this, `id.*` can be used to follow the pointer to the entire data for the record.

```surql
SELECT id.* FROM $record;
```

```surql title="Output"
[
	{
		id: {
			data: {
				data: 'for',
				demonstration: 'purposes',
				some: 'demo'
			},
			id: person:1
		}
	}
]
```

## Limitations

At present, the `VALUE` clause cannot be used inside a [`DEFINE FIELD`](/docs/surrealql/statements/define/field) statement.

```surql
/**[test]

[[test.results]]
error = "'Cannot use the `VALUE` keyword on the `id` field.'"

*/

DEFINE FIELD id ON user VALUE rand::int(1, 1000000000) READONLY;
```

```surql title="Output"
[
	{
		id: user:9ixn3oei6o532c2qyixa
	}
]
```

To achieve the desired behaviour, the `id` field can be set inside the statement to create the record.

```surql
/**[test]

[[test.results]]
value = "[{ id: user:639167349 }]"
skip-record-id-key = true

*/


CREATE user SET id = rand::int(1, 1000000000);
```

## Learn more

Learn more about record IDs [in this blogpost](/blog/the-life-changing-magic-of-surrealdb-record-ids#the-performance-at-scale) and on this [youtube video](https://www.youtube.com/watch?v=c0cqmWRYP8c).


## index.mdx

---
sidebar_position: 1
sidebar_label: Data types
title: Data types | SurrealQL
description: SurrealQL allows you to describe data with specific data types. These data types are used to validate data and to generate the appropriate database schema.

---

import Table from '@components/shared/Table.astro'

# Data types

SurrealQL allows you to describe data with specific data types. These data types are used to validate data and to generate the appropriate database schema.

<Table>
    <thead>
        <tr>
            <th scope="col" class="w-40">Type</th>
            <th scope="col">Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td scope="row" data-label="Type">
                <code>any</code>
            </td>
            <td scope="row" data-label="Description">
                Use this when you explicitly don't want to specify the field's data type. The field will allow any data type supported by SurrealDB.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>array</code>
            </td>
            <td scope="row" data-label="Description">
                An array of items.
                The array type also allows you to define which types can be stored in the array and the required length.
                <ul>
                    <li><code>array</code></li>
                    <li><code>array&lt;string&gt;</code></li>
                    <li><code>array&lt;string, 10&gt;</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>bool</code>
            </td>
            <td scope="row" data-label="Description">
                Describes whether something is truthy or not.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#bytes"><code>bytes</code></a>
            </td>
            <td scope="row" data-label="Description">
                Stores a value in a byte array.
                <ul>
                    <li><code>&lt;bytes&gt;value</code></li>
                    <li><code>bytes</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>datetime</code>
            </td>
            <td scope="row" data-label="Description">
                An [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) compliant data type that stores a date with time and time zone.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>decimal</code>
            </td>
            <td scope="row" data-label="Description">
               Data type for storing [decimal floating point](https://en.wikipedia.org/wiki/Decimal128_floating-point_format) numbers.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>duration</code>
            </td>
            <td scope="row" data-label="Description">
                Store a value representing a length of time. Can be added or subtracted from datetimes or other durations.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>float</code>
            </td>
            <td scope="row" data-label="Description">
                Data type for storing [floating point](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) numbers. Larger or extremely precise values should be stored as a decimal.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="#geometry"><code>geometry</code></a>
            </td>
            <td scope="row" data-label="Description">
                <a href="https://www.rfc-editor.org/rfc/rfc7946" target="_blank" title="Link to RFC 7946">RFC 7946</a> compliant data type for storing geometry in the <a href="https://geojson.org/" target="_blank" title="Link to the GeoJson website">GeoJson format</a>.
                <ul>
                    <li><code>geometry&lt;feature&gt;</code></li>
                    <li><code>geometry&lt;point&gt;</code></li>
                    <li><code>geometry&lt;line&gt;</code></li>
                    <li><code>geometry&lt;polygon&gt;</code></li>
                    <li><code>geometry&lt;multipoint&gt;</code></li>
                    <li><code>geometry&lt;multiline&gt;</code></li>
                    <li><code>geometry&lt;multipolygon&gt;</code></li>
                    <li><code>geometry&lt;collection&gt;</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>int</code>
            </td>
            <td scope="row" data-label="Description">
                Store a value in a 64 bit signed integer. Values can range between `-9223372036854775808` and `9223372036854775807` (inclusive). Larger values should be stored as a float or a decimal.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>number</code>
            </td>
            <td scope="row" data-label="Description">
                Store numbers without specifying the type.
                SurrealDB will detect the type of number and store it using the minimal number of bytes.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>object</code>
            </td>
            <td scope="row" data-label="Description">
                Store formatted objects containing values of any supported type including nested objects or arrays.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>regex</code>
            </td>
            <td scope="row" data-label="Description">
                A compiled regular expression that can be used for matching strings.
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <a href="/docs/surrealql/datamodel/literals"><code>literal</code></a>
            </td>
            <td scope="row" data-label="Description">
                A value that may have multiple representations or formats, similar to an enum or a union type. Can be composed of strings, numbers, objects, arrays, or durations.
                <ul>
                    <li><code>"a" | "b"</code></li>
                    <li><code>[number, "abc"]</code></li>
                    <li><code>123 | 456 | string | 1y1m1d</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>option</code>
            </td>
            <td scope="row" data-label="Description">
                Makes types optional and guarantees the field to be either empty (NONE) or some other type.
                <ul>
                    <li><code>option&lt;number&gt;</code> </li>
                </ul>
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>range</code>
            </td>
            <td scope="row" data-label="Description">
                A range of possible values. Lower and upper bounds can be set, in the absence of which the range becomes open-ended. A range of integers can be used in a FOR loop.
                <ul>
                    <li><code>0..10</code> </li>
                    <li><code>0..=10</code> </li>
                    <li><code>..10</code> </li>
                    <li><code>'a'..'z'</code> </li>
                </ul>
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>record</code>
            </td>
            <td scope="row" data-label="Description">
                Store a reference to another record. The value must be a Record ID. Add the record name inside angle brackets to restrict the reference to only certain record names.
                <ul>
                    <li><code>record</code></li>
                    <li><code>record&lt;user&gt;</code></li>
                    <li><code>record&lt;user | administrator&gt;</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>set</code>
            </td>
            <td scope="row" data-label="Description">
                A set of items.
                The set type also allows you to define which types can be stored in the set and the required length.
                Items are automatically deduplicated and orderd.
                <ul>
                    <li><code>set</code></li>
                    <li><code>set&lt;string&gt;</code></li>
                    <li><code>set&lt;string, 10&gt;</code></li>
                </ul>
            </td>
        </tr>
        <tr>
            <td scope="row" data-label="Type">
                <code>string</code>
            </td>
            <td scope="row" data-label="Description">
                Describes a text-like value.
            </td>
        </tr>
    </tbody>
</Table>

## Examples

### geometry

```surql
-- Define a field with a single type
DEFINE FIELD location ON TABLE restaurant TYPE geometry<point>;
-- Define a field with any geometric type
DEFINE FIELD area ON TABLE restaurant TYPE geometry<feature>;
-- Define a field with specific geometric types
DEFINE FIELD area ON TABLE restaurant TYPE geometry<polygon|multipolygon|collection>;
```

### bytes

```surql
-- Define a field with a single type
DEFINE FIELD image ON TABLE product TYPE bytes;

-- Create a record with a bytes field and set the value
CREATE foo SET value = <bytes>"bar";

```


## literals.mdx

---
sidebar_position: 23
sidebar_label: Literals
title: Literals | SurrealQL
description: A value that may have multiple representations or formats.

---

import Since from '@components/shared/Since.astro'

# Literals

<Since v="v2.0.0" />

A literal is a value that may have multiple representations or formats, similar to an enum or a union type. A literal can be composed of strings, numbers, objects, arrays, or durations.

## Examples

A literal can be as simple as a declaration that a parameter must be a certain value.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
error = ""Tried to set `$nine`, but couldn't coerce value: Expected `9` but found `10`""

*/

LET $nine: 9 = 9;
LET $nine: 9 = 10;
```

```surql title="Response"
-------- Query --------

NONE

-------- Query --------

"Tried to set `$nine`, but couldn't coerce value: Expected `9` but found `10`"
```

Using `|` allows a literal to be a number of possible options.

```surql
/**[test]

[[test.results]]
error = ""Tried to set `$nine`, but couldn't coerce value: Expected `9 | '9' | 'nine'` but found `'Nein'`""

*/

LET $nine: 9 | "9" | "nine" = "Nein";
```

```surql title="Response"
"Tried to set `$nine`, but couldn't coerce value: Expected `9 | '9' | 'nine'` but found `'Nein'`"
```

A literal can contain possible types in addition to possible values.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

LET $flexible_param: datetime | uuid | "N/A" = "N/A";
LET $flexible_param: datetime | uuid | "N/A" = <datetime>"2024-09-01";
```

Literals that include the option to be an array or an object can contain rich data.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

LET $status: "Ok" | { err: string } = { err: "Forgot to plug it in" };
```

## Literals in database schema

Literals can be defined inside a database schema by using a [DEFINE FIELD](/docs/surrealql/statements/define/field) statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ error_info: { error: 'Deprecated', message: "You shouldn't use this anymore" }, id: information:obqulsqif466kp1t3agq }]"
skip-record-id-key = true

[[test.results]]
error = ""Couldn't coerce value for field `error_info` of `information:qbohn4wu4l2t81wj2fb3`: Expected `{ error: 'Continue' } | { error: 'RetryWithId', id: string } | { error: 'Deprecated', message: string }` but found `\"You shouldn't use this anymore\"`""

*/

DEFINE FIELD error_info ON TABLE information TYPE
      { error: "Continue" }
    | { error: "RetryWithId", id: string }
    | { error: "Deprecated", message: string };

CREATE information SET
	error_info = { error: "Deprecated", message: "You shouldn't use this anymore" };
-- Doesn't conform to definition, will not work
CREATE information SET
	error_info = "You shouldn't use this anymore";
```

```surql title="Response"
-------- Query --------

[
	{
		error_info: {
			error: 'Deprecated',
			message: "You shouldn't use this anymore"
		},
		id: info:pkckjrri8q1pg12unyuo
	}
]

-------- Query --------

"Couldn't coerce value for field `error_info` of `information:qbohn4wu4l2t81wj2fb3`: Expected `{ error: 'Continue' } | { error: 'RetryWithId', id: string } | { error: 'Deprecated', message: string }` but found `\"You shouldn't use this anymore\"`"
```

## Matching on literals

While SurrealQL does not have a `match` or `switch` operator, `IF ELSE` statements can be used to match on a literal, particularly if each possible type is an object. The following shows a similar example to the above except that each object begins with a field containing the name of the type of error.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE FIELD error_info ON TABLE information TYPE
	{ Continue:    { message: "" }} |
	{ Retry: { error: "Retrying", after: duration }} |
	{ Deprecated:  { message: string }};
```

Next, we will [define a function](/docs/surrealql/statements/define/function) to handle this field and return a certain type of message depending on the error. Note the following:

* The `LET` statement in the first line is simply to shorten the path to the information contained inside `error_info`
* `IF ELSE` statement works here because [IF](/docs/surrealql/statements/throw) involves a check for [truthiness](/docs/surrealql/datamodel/values#values-and-truthiness), returning `true` as long as it finds a value that is not none, empty, or zero.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE FUNCTION fn::handle_error($data: record<information>) -> string {
	LET $err = $data.error_info;
	RETURN IF $err.Continue {
		"Continue"
	}
	ELSE IF $err.Retry {
		sleep($err.Retry.after);
		"Now retrying again"
	}
	ELSE IF $err.Deprecated {
		$err.Deprecated.message
	}
};
```

With the function set up, the `info` records can be inserted and run one at a time through the function.

```surql
LET $info = INSERT INTO information [
	{ error_info: { Continue: { message: "" } }},
	{ error_info: { Retry: { error: "Retrying", after: 1s } }},
	{ error_info: { Deprecated: { message: "Thought I said you shouldn't use this anymore" } }}
];

fn::handle_error($info[0].id);
fn::handle_error($info[1].id);
fn::handle_error($info[2].id);
```

```surql title="Output"
-------- Query --------

'Continue'

-------- Query --------

-- After waiting 1 second
'Now retrying again'

-------- Query --------

"Thought I said you shouldn't use this anymore"
```


## none-and-null.mdx

---
sidebar_position: 11
sidebar_label: None and Null
title: None and Null | SurrealQL
description: SurrealDB uses two types called None and Null to represent two different ways in which data may not exist.

---

import Since from "@components/shared/Since.astro";

# None and null

SurrealDB uses two types called `None` and `Null` to represent two different ways in which data may not exist. While these may appear similar, they have different meanings and are used in different contexts.

## None values

`None` is used to denote that "something does not exist", for example, a field which is not present on a record.
Because of this, values of `None` can not be stored within records, meaning uses of `None` are typically limited to SurrealQL statements
where it is used to denote a value or response that does not exist.

### Example

Setting a record field to `None` is analogous to using `UNSET` to remove the field entirely. While inside the query it may appear that `None` is being written to the `children` field, what is actually happening is that the `children` field is being removed from the record.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:two }]"

[[test.results]]
value = "[{ children: [person:two], id: person:one }]"

[[test.results]]
value = "[{ id: person:one }]"

[[test.results]]
value = "[{ id: person:one }, { id: person:two }]"

*/

CREATE person:two;
CREATE person:one SET children = [person:two];
UPDATE person:one SET children = NONE;
SELECT * FROM person;
```



```surql title="Output"
[
  { id: person:one },
  { id: person:two }
]
```

## Null values

`Null` values are used to denote that "something exists, but has no value". This is useful when a field is present on a record, but the value of that field is unknown or not applicable. Unlike `None`, `Null` is written into records and can be stored as a value.

### Example

Setting a record field to `Null` will create the field on the record, but denotes that the field is considered empty. In this example, the `children` field is present on the record, but the value of that field is `null`.

```surql
/**[test]

[[test.results]]
value = "[{ children: NULL, id: person:dgwjn0ldg8ep3e8y39jw }]"
skip-record-id-key = true

*/

CREATE person SET children = null;
```



```surql title="Output"
[
  { 
    children: NULL, 
    id: person:dgwjn0ldg8ep3e8y39jw
  }
]
```

## When to use None or Null

How you use `None` or `Null` is largely dependent on the context in which you are working.

If you are writing SurrealQL and need to denote something that does not exist, such as the absence of a field, use `None`.

If you are working with data and need to represent a value which is empty, use `Null`. This is particularly useful when needing to deserialize SurrealQL output into a type in another programming language that requires a field name to be present.

## NONE as a datatype

<Since v="v3.0.0" />

Since SurrealDB 3.0, NONE has been usable as a datatype of its own. This allows syntax like the following to be used without returning a parsing error.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

DEFINE FUNCTION fn::do_stuff() -> NONE {
  // Code that should return nothing
};

DEFINE FIELD middle_name ON TABLE user TYPE string | NONE; // Equivalent to option<string>

DEFINE FIELD value ON temperature TYPE float | decimal | NONE; // Equivalent to option<float|decimal>
```

## numbers.mdx

---
sidebar_position: 12
sidebar_label: Numbers
title: Numbers | SurrealQL
description: In SurrealDB, numbers can be one of three types - 64-bit integers, 64-bit floating point numbers, or 128-bit decimal numbers.

---

# Numbers

In SurrealDB, numbers can be one of three types: 64-bit integers, 64-bit floating point numbers, or 128-bit decimal numbers.

## Integer numbers
If a numeric value is specified without a decimal point and is within the range `-9223372036854775808` to `9223372036854775807` then the value will be parsed, stored, and treated as a 64-bit integer.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:j3tdh7wrm2kv1ymbbsek, year: 2022 }]"
skip-record-id-key = true

*/

CREATE event SET year = 2022;
```

## Floating point numbers
If a number value is specified with a decimal point, or is outside of the maximum range specified above, then the number will automatically be parsed, stored, and treated as a 64-bit floating point value. This ensures efficiency when performing mathematical calculations within SurrealDB.

```surql
/**[test]

[[test.results]]
value = "[{ id: event:5h756te17xakgdzfbghe, temperature: 41.5f }]"
skip-record-id-key = true

*/

CREATE event SET temperature = 41.5;
```

## Decimal numbers
To opt into 128-bit decimal numbers when specifying numeric values, you can use the `dec` suffix.

```surql
/**[test]

[[test.results]]
value = "[{ id: product:64vljvh12gdfwolgfhux, price: 99.99dec }]"
skip-record-id-key = true

*/

CREATE product SET price = 99.99dec;
```

The `dec` suffix is an instruction to the parser and not a cast, and is thus preferred when making a decimal.

```surql
/**[test]

[[test.results]]
value = "3.888888888888889dec"

[[test.results]]
value = "3.8888888888888888dec"

*/

-- Creates the imprecise float 3.888888888888889 and casts it into a decimal as 3.888888888888889dec
RETURN <decimal>3.8888888888888888;
-- Uses the input 3.8888888888888888 to directly create a decimal
RETURN 3.8888888888888888dec;
```

## Using a specific numeric type
To use a specific type when specifying numeric values, you can cast the value to a specific numeric type or use the appropriate suffix.

```surql
/**[test]

[[test.results]]
value = "[{ horizon: 34dec, id: event:8vpjsysxnskyfuh9ve87, temperature: 46.5f, year: 2022 }]"
skip-record-id-key = true

*/

CREATE event SET
	year = <int> 2022,
	temperature = <float> 41.5 + 5f,
	horizon = <decimal> 31 + 3dec
;
```

## Numeric precision
Different numeric types can be compared and used together in calculations.

The benefits of floating point numeric values are speed and storage size, but there is a limit to the numeric precision.

```surql
/**[test]

[[test.results]]
value = "13.571938471938472f"

*/

RETURN 13.5719384719384719385639856394139476937756394756;

-- 13.571938471938472f
```

In addition, when using floating point numbers specifically, mathematical operations can result in a loss of precision (as is normal with other databases).

```surql
/**[test]

[[test.results]]
value = "0.9999999999999999f"

*/

RETURN 0.3 + 0.3 + 0.3 + 0.1;

-- 0.9999999999999999f
```

Common rounding errors can be avoided by performing calculations using decimals.

```surql
/**[test]

[[test.results]]
value = "1.0dec"

*/

RETURN 0.3dec + 0.3dec + 0.3dec + 0.1dec;

-- 1.0dec
```

## Underscores

As a convenience, underscores are ignored when using a number. This allows input to be more readable than it would otherwise. Because underscores are ignored, they will not display in the output.

```surql
RELATE dr:evil->bribes->other:character SET dollars = 1_000_000.99;
-- [{ dollars: 1000000.99, id: bribes:4bfld2ukwnja24dzrpw9, in: dr:evil, out: other:character }]

-- Input Korean currency counted in units of 10000, not 1000
RELATE korean:purchaser->buys_house_from->korean:seller
              -- 10억 4천만 5천
    SET amount = 10_4000_5000;
-- [{ amount: 1040005000, id: buys_house_from:9070t2ctgwwg202cpw1z, in: korean:purchaser, out: korean:seller }]
```

## Mathematical constants
A set of floating point numeric constants are available in SurrealDB. Constant names are case insensitive, and can be specified with either lowercase or capital letters, or a mixture of both.

```surql
/**[test]

[[test.results]]
value = "[{ circumference: 10, id: circle:uvw3g4dli4x77xejcjej }]"
skip-record-id-key = true

[[test.results]]
value = "[{ circumference: 10, id: circle:uvw3g4dli4x77xejcjej, radius: 15.707963267948966f }]"
skip-record-id-key = true

*/

CREATE circle SET circumference = 10;
UPDATE circle SET radius = circumference / ( 2 * MATH::PI );
```

<table>
    <thead>
        <tr>
            <th colspan="2" scope="col">Constant</th>
            <th colspan="2" scope="col">Description</th>
            <th colspan="2" scope="col">Value</th>
        </tr>
    </thead>  
    <tbody>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::E</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                Euler’s number (e)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                2.718281828459045
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::FRAC_1_PI</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                1/π
            </td>
            <td colspan="2" scope="row" data-label="Value">
                0.3183098861837907
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::FRAC_1_SQRT_2</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                1/sqrt(2)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                0.7071067811865476
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::FRAC_2_PI</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                2/π
            </td>
            <td colspan="2" scope="row" data-label="Value">
                0.6366197723675814
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::FRAC_2_SQRT_PI</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                2/sqrt(π)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                1.1283791670955126
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::FRAC_PI_2</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                π/2
            </td>
            <td colspan="2" scope="row" data-label="Value">
            1.5707963267948966
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::FRAC_PI_3</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                π/3
            </td>
            <td colspan="2" scope="row" data-label="Value">
                1.0471975511965979
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::FRAC_PI_4</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                π/4
            </td>
            <td colspan="2" scope="row" data-label="Value">
                0.7853981633974483
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::FRAC_PI_6</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                π/6
            </td>
            <td colspan="2" scope="row" data-label="Value">
                0.5235987755982989
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::FRAC_PI_8</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                π/8
            </td>
            <td colspan="2" scope="row" data-label="Value">
                0.39269908169872414
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::INF</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                Positive infinity
            </td>
            <td colspan="2" scope="row" data-label="Value">
                inf
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::LN_10</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                ln(10)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                2.302585092994046
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::LN_2</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                ln(2)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                0.6931471805599453
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::LOG10_2</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                log<sub>10</sub>(2)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                0.3010299956639812
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::LOG10_E</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                log<sub>10</sub>(e)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                0.4342944819032518
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::LOG2_10</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                log<sub>2</sub>(10)
            </td>
            <td colspan="2" scope="row" data-label="Value">
            3.321928094887362
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::LOG2_E</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                log<sub>2</sub>(e)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                1.4426950408889634
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::NEG_INF</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                Negative infinity
            </td>
            <td colspan="2" scope="row" data-label="Value">
                -inf
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::PI</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                Archimedes’ constant (π)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                3.141592653589793
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::SQRT_2</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                sqrt(2)
            </td>
            <td colspan="2" scope="row" data-label="Value">
            1.4142135623730951
            </td>
        </tr>
        <tr>
            <td colspan="2" scope="row" data-label="Constant">
                <code>MATH::TAU</code>
            </td>
            <td colspan="2" scope="row" data-label="Description">
                The full circle constant (τ)
            </td>
            <td colspan="2" scope="row" data-label="Value">
                6.283185307179586
            </td>
        </tr>
    </tbody>
</table>

## Next steps
You've now seen how to use numeric values in SurrealDB. For more advanced functionality, take a look at the operators and math functions, which enable advanced calculations on numeric values and sets of numeric values.


## objects.mdx

---
sidebar_position: 13
sidebar_label: Objects
title: Objects | SurrealQL
description: SurrealDB records can store objects with fields that can also hold other objects or arrays

---

import Since from '@components/shared/Since.astro'

# Objects

An object is a collection of named fields and values.

As a record is essentially an object with a required [`id` field](/docs/surrealql/datamodel/ids) that can be created, updated, or deleted, they can be worked with in almost exactly the same way as a standalone object.

A field of an object can be of any value type, including another object or array at multiple levels of depth. This allows objects and arrays to be stored within each other in order to model complex data scenarios.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:ix25bwtvcx5s543eejrk, metadata: { activities: ['clicked link', 'contact form', 'read email', 'viewed website', 'viewed website', 'viewed website', 'read email'], information: { age: 23, gender: 'm' }, interest_level: 83.67f, marketing: true } }]"
skip-record-id-key = true

*/

CREATE person SET metadata = {
	interest_level: 83.67,
	information: {
		age: 23,
		gender: 'm',
	},
	marketing: true,
	activities: [
		"clicked link",
		"contact form",
		"read email",
		"viewed website",
		"viewed website",
		"viewed website",
		"read email",
	]
};
```

## Field names

### Valid field names

Similar to record IDs, field names can be constructed from ASCII characters, underscores, and numbers. To create a field name with complex characters, backticks can be used.

```surql
/**[test]

[[test.results]]
value = "{ id: user:3lgc83vechgblizli263, my_name: 'name' }"
skip-record-id-key = true

[[test.results]]
value = "{ id: user:nyzy7aeyup2ygt33ci95, "mi_nómine😊": 'name' }"
skip-record-id-key = true

*/

CREATE ONLY user SET my_name = 'name';
CREATE ONLY user SET `mi_nómine😊` = 'name';
```

```surql title="Output"
-------- Query --------

{
	id: user:nronupvxvdm7r1n5hlzm,
	my_name: 'name'
}

-------- Query 2 --------

{
	id: user:eb5pu7u9g67dy773hsv9,
	"mi_nómine😊": 'name'
}
```

Inside a standalone object, non-ASCII field names can also be set by using a string.

```surql
/**[test]

[[test.results]]
value = "{ "mi nómine": 'Edgar' }"

*/

SELECT * FROM ONLY {
    "mi nómine": "Edgar"
};
```

```surql title="Output"
{
	"mi nómine": 'Edgar'
}
```

### Automatically generated field names

A field created from an operation will have a field name that represents the operation(s) used to construct it.

```surql
/**[test]

[[test.results]]
value = "[{ "[math::min(temps), math::max(temps)]": [-5, 9], "math::mean": 4f }]"

*/

SELECT
    math::mean(temps),
    [ math::min(temps), math::max(temps) ]
FROM { temps: [-5, 8, 9] };
```

```surql title="Output"
[
    {
        "[math::min(temps), math::max(temps)]": [
            -5,
            9
        ],
        "math::mean": 4f
    }
]
```

Using `AS` allows these automatically calculated field names to be replaced with custom names.

```surql
/**[test]

[[test.results]]
value = "[{ avg_temps: [-5, 9], mean_temps: 4f }]"

*/

SELECT
    math::mean(temps) AS mean_temps,
    [ math::min(temps), math::max(temps) ] AS avg_temps
FROM { temps: [-5, 8, 9] };
```

```surql title="Output"
[
    {
        "avg_temps": [
            -5,
            9
        ],
        "mean_temps": 4
    }
]
```

## Extending objects and removing fields

<Since v="v3.0.0" />

Two objects can be merged by using either the `+` operator or the `object::extend()` function. Any fields in the second object will be added to the first object, thereby updating any existing fields and adding new fields to those that were not present.

```surql
/**[test]

[[test.results]]
value = "{ name: 'Venus', orbital_period: 1y31w1d22h, radius: 6051.8f }"

[[test.results]]
value = "{ name: 'Venus', orbital_period: 1y31w1d22h, radius: 6051.8f }"

*/

{ name: "Venus", radius: 6000 } + { radius: 6051.8, orbital_period: 1y31w1d22h };
{ name: "Venus", radius: 6000 }.extend({ radius: 6051.8, orbital_period: 1y31w1d22h });
```

```surql title="Output"
-------- Query 1 --------

{
	name: 'Venus',
	orbital_period: 1y31w1d22h,
	radius: 6051.8f
}

-------- Query 2 --------

{
	name: 'Venus',
	orbital_period: 1y31w1d22h,
	radius: 6051.8f
}
```

Fields of an object can be removed with the `object::remove()` function, which takes either a single string or an array of strings of the field names to remove.

```surql
/**[test]

[[test.results]]
value = "{ name: 'Venus', orbital_period: 1y31w1d22h }"

[[test.results]]
value = "{ name: 'Venus' }"

*/
{ name: 'Venus', orbital_period: 1y31w1d22h, radius: 6051.8 }.remove("radius");
{ name: 'Venus', orbital_period: 1y31w1d22h, radius: 6051.8 }.remove(["radius", "orbital_period"]);
```

```surql title="Output"
-------- Query 1 --------

{
	name: 'Venus',
	orbital_period: 1y31w1d22h
}

-------- Query 2 --------

{
	name: 'Venus'
}
```

## See also

* [Object functions](/docs/surrealql/functions/database/object)
* [Destructuring nested objects](/docs/surrealql/datamodel/idioms#destructuring)

## ranges.mdx

---
sidebar_position: 14
sidebar_label: Ranges
title: Ranges | SurrealQL
description: A range of possible values.

---

import Since from '@components/shared/Since.astro'

# Ranges

<Since v="v2.0.0" />

A range is composed of `..` and possible delimiters to set the maximum and minimum possible values. The default syntax includes the lower limit and excludes the upper limit. A `=` can be used to make the upper limit inclusive, and `>` can be used to make the lower limit exclusive.

```surql
/**[test]

[[test.results]]
value = "0..10"

[[test.results]]
value = "0..=10"

[[test.results]]
value = "0>..10"

[[test.results]]
value = "0>..=10"

*/

-- From 0 up to 9
0..10;
-- From 0 up to 10
0..=10;
-- From 1 to 9
0>..10;
-- From 1 to 10
0>..=10;
```

A range becomes open ended if a delimiter is not specified.

```surql
/**[test]

[[test.results]]
value = "0.."

[[test.results]]
value = "0>.."

[[test.results]]
value = "..10"

[[test.results]]
value = "..=100"

[[test.results]]
value = ".."

*/

-- Anything from 0 and up
0..;
-- Anything from 1 and up
0>..;
-- Anything up to 99
..100;
-- Anything up to 100
..=100;
-- An infinite range
..;
```

A range can be constructed from any type of value. This is most useful when comparing one value to another.

```surql
/**[test]

[[test.results]]
value = "true"

[[test.results]]
value = "true"

[[test.results]]
value = "true"

[[test.results]]
value = "false"

[[test.results]]
value = "false"

[[test.results]]
value = "false"

*/


-- All true
'g' IN 'a'..'z';
d"2024-01-01" IN d"2020-01-01"..=d"2025-01-01";
['London', d"2022-02-02", 5.7] IN ['London', d"2020-01-01"]..=['London', d"2024-12-31"];

-- All false
"ㅋㅋㅋ" IN "a".."z";
d"2028-01-01" IN d"2020-01-01"..=d"2025-01-01";
['Philadelphia', d"2022-02-02", 5.7] IN ['London', d"2020-01-01"]..=['London', d"2024-12-31"];
```

## Ranges in FOR loops

Ranges of integers have the added convenience of being able to be used in a [FOR loop](/docs/surrealql/statements/for).

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

FOR $year IN 0..=2024 {
    CREATE historical_events SET
        for_year = $year,
        events = "To be added";
}
```

## Ranges in WHERE clauses

A range can be used in a `WHERE` clause in place of operators like `<` and `>`. This is especially useful when checking for a number that must be within a certain range. Using a range carries two main benefits. One is that it produces shorter code that is easier to read and maintain.

```surql
SELECT * FROM person WHERE age >= 18 AND age <= 65;
SELECT * FROM person WHERE age IN 18..=65;
```

Another benefit is performance. The following code should show a modest but measurable improvement in performance between the first and second `SELECT` statement, as only one condition needs to be checked instead of two.

```surql
CREATE |person:20000| SET age = (rand::float() * 120).round() RETURN NONE;

-- Assign output to a parameter so the SELECT output is not displayed
LET $_ = SELECT * FROM person WHERE age > 18 AND age < 65;
LET $_ = SELECT * FROM person WHERE age in 18..=65;
```

## Casting and functional usage

A range can be cast into an array.

```surql
/**[test]

[[test.results]]
value = "[1, 2]"

*/

<array> 1..3;
```

```surql title="Output"
[
	1,
	2
]
```

This opens up a range of functional programming patterns that are made possible by SurrealDB's [array functions](/docs/surrealql/functions/database/array), many of which can use [anonymous functions](/docs/surrealql/datamodel/closures) (closures) to perform an operation on each item in the array.

```surql
/**[test]

[[test.results]]
value = "[{ original: 130, square_root: 11.40175425099138f }, { original: 140, square_root: 11.832159566199232f }]"

*/

-- Construct an array
(<array> 1..=100)
-- Turn it into an array that increments by 10
    .map(|$v| $v * 10)
-- Turn each number into a object with original and square root value
    .map(|$v| { original: $v, square_root: math::sqrt($v) })
-- Keep only those with square roots in between 11 and 12
    .filter(|$obj| $obj.square_root IN 11..12);
```

```surql title="Output"
[
	{
		original: 130,
		square_root: 11.40175425099138f
	},
	{
		original: 140,
		square_root: 11.832159566199232f
	}
]
```

## Ranges in mock syntax for `CREATE` statements

<Since v="v3.0.0" />

`CREATE` statements have always been able to work on more than one record by enclosing either a single number or a range-like operator between two `||` bars.

```surql title="Before 3.0.0"
-- Create 10 person records with random IDs
CREATE |person:10|;
-- Create `person` records from person:1 to person:10
CREATE |person:1..10|;
```

Originally an internal syntax for mock testing, this syntax become known to the user community and is now commonly used. However, the original syntax differed from true ranges in always being inclusive, in that `1..10` was treated as "from 1 up to and including 10". A change has since been made to have the mock syntax take a true range with a syntax equivalent to that demonstrated in this page.

```surql title="Since 3.0.0"
-- All of these create ten records from person:1 to person:10
CREATE |person:1..=10|;
CREATE |person:1..11|;
CREATE |person:0>..11|;
CREATE |person:0>..=10|;
```

## records.mdx

---
sidebar_position: 25
sidebar_label: Record links
title: Record links | SurrealQL
description: One of the most powerful features of SurrealDB is the ability to traverse from record-to-record without the need for traditional SQL JOINs. Each record ID points directly to a specific record in the database.

---

# Record links

One of the most powerful features of SurrealDB is the ability to traverse from record-to-record without the need for traditional SQL JOINs. Each record ID points directly to a specific record in the database, without needing to run a table scan query. Record IDs can be stored within other records, allowing them to be linked together.

## Creating a record
When you create a record without specifying the id, then a randomly generated id is created and used for the record id.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:lva0fwhgqxuk0m88ktn1, name: 'Tobie' }]"
skip-record-id-key = true

*/

CREATE person SET name = 'Tobie';

-- person:aio58g22n3upq16hsani
```

It's also possible to specify a specific record id when creating or updating records.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:tester, name: 'Tobie' }]"

*/

CREATE person:tester SET name = 'Tobie';

-- person:tester
```

## Select directly off of Record IDs

Because Record IDs are their own datatype in SurrealQL, you are able to select directly off of them.

```surql
/**[test]

[[test.results]]
value = "[{ email: 'tobie@surrealdb.com', id: person:tobie, name: 'Tobie', opts: { enabled: true } }]"

[[test.results]]
value = "{ email: 'tobie@surrealdb.com', id: person:tobie, name: 'Tobie', opts: { enabled: true } }"

[[test.results]]
value = "{ email: 'tobie@surrealdb.com', name: 'Tobie' }"

*/

CREATE person:tobie SET name = 'Tobie', email = 'tobie@surrealdb.com', opts.enabled = true;

-- Select the whole record
person:tobie.*;

-- Select specific fields
person:tobie.{ name, email };
```

## Storing record links within records

Records ids can be stored directly within other records, either as top-level properties, or nested within objects or arrays.

```surql
/**[test]

[[test.results]]
value = "[{ friends: [person:tobie, person:simon], id: person:jaime, name: 'Jaime' }]"

[[test.results]]
value = "[{ friends: [person:simon, person:marcus], id: person:tobie, name: 'Tobie' }]"

[[test.results]]
value = "[{ friends: [person:jaime, person:tobie], id: person:simon, name: 'Simon' }]"

[[test.results]]
value = "[{ friends: [person:tobie], id: person:marcus, name: 'Marcus' }]"

*/

CREATE person:jaime SET name = 'Jaime', friends = [person:tobie, person:simon];
CREATE person:tobie SET name = 'Tobie', friends = [person:simon, person:marcus];
CREATE person:simon SET name = 'Simon', friends = [person:jaime, person:tobie];
CREATE person:marcus SET name = 'Marcus', friends = [person:tobie];
```

## Fetching remote records from within records

Nested field traversal can be used to fetch the properties from the remote records, as if the record was embedded within the record being queried.

```surql
SELECT friends.name FROM person:tobie;
[
	{
		friends: {
			name: ["Simon", "Marcus"]
		}
	}
]
```

There is no limit to the number of remote traversals that can be performed in a query. Using `.` dot notation, SurrealDB does not differentiate between nested object properties, or remote records, and will fetch remote records asynchronously when needed for a query.

```surql
SELECT friends.friends.friends.name FROM person:tobie;
[
	{
		friends: {
			friends: {
				friends: {
					name: [
						[ ["Tobie", "Simon"], ["Simon", "Marcus"] ],
						[ ["Simon", "Marcus"] ]
					]
				}
			}
		}
	}
]
```

## Next steps

You've now seen how to create records using randomly generated ids, or specific record ids. This is just the beginning! The power and flexibility which is possible with the remote record fetching functionality within queries opens up a whole new set of possibilities for storing and querying traditional data, and connected datasets. The next page follows up with a feature available in SurrealDB since version 2.2.0: the ability to use record links in a bidirectional manner thanks to reference tracking.

Also check out this explainer video on using record links in SurrealDB:


<iframe width="100%" src="https://www.youtube.com/embed/TyX45cyZ-W0?si=S9M59afDEiqxeC5d" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" style={{aspectRatio: 1.7, paddingTop: '20px'}} allowfullscreen></iframe>



## references.mdx

---
sidebar_position: 26
sidebar_label: Record references
title: Record references | SurrealQL
description: Record references allow you to link records together, enabling you to traverse from one record to another.

---

import Since from '@components/shared/Since.astro'

# Record references

<Since v="v2.2.0" />

## Basic concepts

Reference tracking begins by adding a `REFERENCE` clause to any `DEFINE FIELD` statement, as long as the field is a top-level field of type `record` or array of records.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE FIELD comics ON person TYPE option<array<record<comic_book>>> REFERENCE;
-- Also works as `option` desugars to this syntax
DEFINE FIELD comics ON person TYPE array<record<comic_book>> | NONE REFERENCE;

-- `comics` field might not be a record, does not work
DEFINE FIELD comics ON person TYPE array<record<comic_book>> | string REFERENCE;
-- Not top-level field, does not work
DEFINE FIELD metadata.comics ON person TYPE array<record<comic_book>> REFERENCE;
```

This incoming record can then be picked up with the `<~` syntax that works in the same way that graph queries do.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ comics: [comic_book:one], id: person:mat, name: 'Mat' }]"

[[test.results]]
value = "[{ comics: [comic_book:one], id: person:nynaeve, name: 'Nynaeve' }]"

[[test.results]]
value = "[{ id: comic_book:one, title: 'Loki, God of Stories' }]"

[[test.results]]
value = "[{ id: comic_book:one, owners: [person:mat, person:nynaeve], title: 'Loki, God of Stories' }]"

[[test.results]]
value = "[{ id: comic_book:one, owners: [{ id: person:mat, name: 'Mat' }, { id: person:nynaeve, name: 'Nynaeve' }], title: 'Loki, God of Stories' }]"

*/

DEFINE FIELD comics ON person TYPE option<array<record<comic_book>>> REFERENCE;
CREATE person:mat SET 
	name = "Mat", 
	comics = [comic_book:one];
CREATE person:nynaeve SET 
	name = "Nynaeve", 
	comics = [comic_book:one];
CREATE comic_book:one SET title = "Loki, God of Stories";

SELECT 
	*, 
	<~person AS owners
FROM comic_book;

SELECT 
	*, 
	<~person.{ id, name } AS owners
FROM comic_book;
```

```surql title="Output"
-------- Query --------

[
	{
		id: comic_book:one,
		owners: [
			person:mat,
			person:nynaeve
		],
		title: 'Loki, God of Stories'
	}
]

-------- Query --------

[
	{
		id: comic_book:one,
		owners: [
			{
				id: person:mat,
				name: 'Mat'
			},
			{
				id: person:nynaeve,
				name: 'Nynaeve'
			}
		],
		title: 'Loki, God of Stories'
	}
]
```

## Specifying linking tables

Incoming references can also be declared in a schema.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ comics: [comic_book:one], id: person:one }, { comics: [comic_book:one], id: person:two }]"

[[test.results]]
value = "[{ id: publisher:one, products: [comic_book:one, book:one] }]"

[[test.results]]
value = "[{ id: comic_book:one, owners: [person:one, person:two], publishers: [publisher:one], title: 'Loki, God of Stories' }]"

[[test.results]]
value = "[{ id: comic_book:one, owners: [person:one, person:two], publishers: [publisher:one], title: 'Loki, God of Stories' }]"

*/

DEFINE FIELD comics ON person TYPE option<array<record<comic_book>>> REFERENCE;
DEFINE FIELD products ON publisher TYPE option<array<record<comic_book|book>>> REFERENCE;
DEFINE FIELD owners ON comic_book COMPUTED <~person;
DEFINE FIELD publishers ON comic_book COMPUTED <~publisher;

CREATE person:one, person:two SET comics = [comic_book:one];
CREATE publisher:one SET products = [comic_book:one, book:one];
CREATE comic_book:one SET title = "Loki, God of Stories";
SELECT * FROM comic_book;
```

```surql title="Output"
[
	{
		id: comic_book:one,
		owners: [
			person:one,
			person:two
		],
		publishers: [
			publisher:one
		],
		title: 'Loki, God of Stories'
	}
]
```

A computed field that picks up incoming references can be further narrowed down to specify not just the table name, but also the field name of the referencing record. This can be done by enclosing the part after `<~` in parentheses, adding the `FIELD` keyword and naming the field or fields via which incoming references will be shown.

```surql
DEFINE FIELD comics ON person TYPE option<array<record<comic_book>>> REFERENCE;
DEFINE FIELD borrowed_comics ON person TYPE option<array<record<comic_book>>> REFERENCE;
DEFINE FIELD owned_by ON comic_book COMPUTED <~(person FIELD comics);
DEFINE FIELD borrowed_by ON comic_book COMPUTED <~(person FIELD borrowed_comics);
DEFINE FIELD all_readers ON comic_book COMPUTED <~(person FIELD comics borrowed_comics);

CREATE person:one SET comics = [comic_book:one];
CREATE person:two SET borrowed_comics = [comic_book:one];
CREATE comic_book:one SET title = "Loki, God of Stories";
SELECT * FROM comic_book;
```

```surql title="Output"
[
	{
		all_readers: [ person:one, person:two ],
		borrowed_by: [ person:two ],
		id: comic_book:one,
		owned_by: [ person:one ],
		title: 'Loki, God of Stories'
	}
]
```

## Specifying deletion behaviour

When working with record links, it is very likely that you will want some behaviour to happen when a referencing link is deleted. Take the following example of a `person` who owns a `comic_book`, which is later deleted. Despite the deletion, a follow-up `SELECT * FROM person` still shows the comic book.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: comic_book:one, owned_by: [], title: 'Loki, God of Stories' }]"

[[test.results]]
value = "[{ comics: [comic_book:one], id: person:one }]"

[[test.results]]
value = "[]"

[[test.results]]
value = "[{ comics: [comic_book:one], id: person:one }]"

*/

DEFINE FIELD comics ON person TYPE option<array<record<comic_book>>> REFERENCE;
DEFINE FIELD owned_by ON comic_book COMPUTED <~person;

CREATE comic_book:one SET title = "Loki, God of Stories";
CREATE person:one SET comics = [comic_book:one];
DELETE comic_book:one;
SELECT * FROM person;
```

```surql title="Output"
[
	{
		comics: [
			comic_book:one
		],
		id: person:one
	}
]
```

A query using `INFO FOR TABLE person` shows that the actual statement created using `REFERENCE` does not finish at this point, but includes the clause `ON DELETE IGNORE`. This is the default behaviour for references.

```surql
{
	events: {},
	fields: {
		comics: 'DEFINE FIELD comics ON person TYPE none | array<record<comic_book>> REFERENCE ON DELETE IGNORE PERMISSIONS FULL',
		"comics.*": 'DEFINE FIELD comics.* ON person TYPE record<comic_book> REFERENCE ON DELETE IGNORE PERMISSIONS FULL'
	},
	indexes: {},
	lives: {},
	tables: {}
}
```

This `ON DELETE` clause can be modified to have some other behaviour when a reference is deleted.

### ON DELETE IGNORE

As shown in the previous section, `ON DELETE IGNORE` is the default behaviour for references and this clause will be added automatically if not specified. It can be added manually to a statement to hint to others reading the code that this behaviour is desired.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ friended_by: [], friends: [person:two], id: person:one }]"

[[test.results]]
value = "[{ friended_by: [person:one], id: person:two }]"

[[test.results]]
value = "[]"

[[test.results]]
value = "{ friended_by: [], id: person:two }"

*/

-- Default, behaviour, so identical to:
-- DEFINE FIELD friends ON person TYPE option<array<record<person>>> REFERENCE;
DEFINE FIELD friends ON person TYPE option<array<record<person>>> REFERENCE ON DELETE IGNORE;
DEFINE FIELD friended_by ON person COMPUTED <~person;

CREATE person:one SET friends = [person:two];
CREATE person:two;
DELETE person:one;
person:two.*;
```

As the deletion of `person:one` is ignored when calculating the `friended_by` field, it will still show `person:one` even though the record itself has been deleted.

```surql
{
	friended_by: [
		person:one
	],
	id: person:two
}
```

### ON DELETE UNSET

`ON DELETE UNSET` will unset (remove) any linked records that are deleted. This can be thought of as the opposite of `ON DELETE IGNORE`.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: person:one }], [{ comments: [comment:rkvpjc0l9iruy21k89su], id: person:one }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ comments: [comment:rkvpjc0l9iruy21k89su, comment:tu5f72ouuydjvycg41st], id: person:one }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[]"

[[test.results]]
value = "[{ author: [person:one], id: comment:rkvpjc0l9iruy21k89su, text: 'Estonia is bigger than I expected!' }]"

*/

DEFINE FIELD comments ON person TYPE option<array<record<comment>>> REFERENCE ON DELETE UNSET;
DEFINE FIELD author ON comment COMPUTED <~person;

CREATE person:one;
UPDATE person:one SET comments += (CREATE ONLY comment SET text = "Estonia is bigger than I expected!").id;
-- Give this one a parameter name so it can be deleted later
LET $comment = CREATE ONLY comment SET text = "I don't get the joke here?";
UPDATE person:one SET comments += $comment.id;
-- Now delete it
DELETE $comment;
-- Only one comment shows up for person:one now
person:one.comments.*.*;
```

```surql title="Output of person:one queries"
-------- Query --------

[
	{
		comments: [
			comment:gj1kb2e3tedn7kjcxxja,
			comment:6sztlhd6fhgc91dg2lby
		],
		id: person:one
	}
]

-------- Query --------

[
	{
		author: [
			person:one
		],
		id: comment:gj1kb2e3tedn7kjcxxja,
		text: 'Estonia is bigger than I expected!'
	}
]
```

### ON DELETE CASCADE

The `ON DELETE CASCADE` will cause a record to be deleted if any record it references is deleted. This is useful for records that should not exist if a record that links to them no longer exists.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ comments: [], id: person:one }]"

[[test.results]]
value = "[{ author: person:one, id: comment:4u9gw4bl2q63zuos3v1y, text: '5/10 for this blog post. The problems I have with it are...' }]"

[[test.results]]
value = "[{ author: person:one, id: comment:em2543ghwy9k921eob3d, text: 'WOW! I never knew you could cut a rope with an arrow.' }]"

[[test.results]]
value = "[{ author: person:one, id: comment:4u9gw4bl2q63zuos3v1y, text: '5/10 for this blog post. The problems I have with it are...' }, { author: person:one, id: comment:em2543ghwy9k921eob3d, text: 'WOW! I never knew you could cut a rope with an arrow.' }]"

[[test.results]]
value = "[]"

[[test.results]]
value = "[]"

*/

DEFINE FIELD author ON comment TYPE record<person> REFERENCE ON DELETE CASCADE;
DEFINE FIELD comments ON person COMPUTED <~comment;

CREATE person:one;
CREATE comment SET author = person:one, text = "5/10 for this blog post. The problems I have with it are...";
CREATE comment SET author = person:one, text = "WOW! I never knew you could cut a rope with an arrow.";

-- Show all the details of comments for 'person:one'
person:one.comments.*.*;
DELETE person:one;
-- Comments no longer exist
SELECT * FROM comment;
```

```surql title="Output"
-------- Query --------

[
	{
		author: person:one,
		id: comment:8msvp0egg8cdlyu4vvn9,
		text: 'WOW! I never knew you could cut a rope with an arrow.'
	},
	{
		author: person:one,
		id: comment:i72qfjy59vbn81hk6lrm,
		text: '5/10 for this blog post. The problems I have with it are...'
	}
]

-------- Query --------

[]

-------- Query --------

[]
```

### ON DELETE REJECT

`ON DELETE REJECT` will outright make it impossible to delete a record that is referenced from somewhere else. For example, consider the case in which a house should not be demolished (deleted) until it has been disconnected from utilities such as gas, water, electricity, and so on. This can be simulated in a schema by adding a `REFERENCE ON DELETE REJECT` to the `utility` table, making it impossible for any `house` to be deleted if they link to it.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: house:one, using: [] }]"

[[test.results]]
value = "[{ connected_to: [house:one], id: utility:gas }, { connected_to: [house:one], id: utility:water }]"

*/

DEFINE FIELD connected_to ON utility TYPE option<array<record<house>>> REFERENCE ON DELETE REJECT;
DEFINE FIELD using ON house COMPUTED <~utility;

CREATE house:one;
CREATE utility:gas, utility:water SET connected_to = [house:one];
```

At this point, the `using` field on `house:one` automatically picks up the two references. Due to these references, the `house` record cannot be deleted.

```surql
house:one.*;
DELETE house:one;
```

```surql title="Output"
-------- Query --------

{
	id: house:one,
	using: [
		utility:gas,
		utility:water
	]
}

-------- Query --------

'Cannot delete `house:one` as it is referenced by `utility:gas` with an ON DELETE REJECT clause'
```

To delete the `house`, the `connected_to` references will first have to be removed.

```surql
UPDATE utility:gas   SET connected_to -= house:one;
UPDATE utility:water SET connected_to -= house:one;

DELETE house:one;
```

Note that an `ON DELETE UNSET` for a required field is effectively the same as an `ON DELETE REJECT`. In both of the following two cases, a `person` that has any referencing `comment` records will not be able to be deleted.

```surql
-- Non-optional field that attempts an UNSET when referencing 'person' is deleted
DEFINE FIELD author ON comment TYPE record<person> REFERENCE ON DELETE UNSET;
LET $person = CREATE ONLY person;
CREATE comment SET text = "Cats are so much better at climbing UP a tree than down! Lol", author = $person.id;
DELETE person;

-- Optional field which rejects the deletion of a referencing 'person'
DEFINE FIELD author ON comment TYPE option<record<person>> REFERENCE ON DELETE REJECT;
LET $person = CREATE ONLY person;
CREATE comment SET text = "Cats are so much better at climbing UP a tree than down! Lol", author = $person.id;
DELETE person;
```

The error message in these two cases will differ, but the behaviour is the same.

```surql
-------- Query --------

"An error occured while updating references for `person:97sfkadd56hqhimbf69m`: Couldn't coerce value for field `author` of `comment:kkigvk5knsoeg53p08n1`: Expected `record<person>` but found `NONE`"

-------- Query --------

'Cannot delete `person:3fm76xztvfab99eq780l` as it is referenced by `comment:ig0ogusbm64cier5ovv9` with an ON DELETE REJECT clause'
```

### ON DELETE THEN

The `ON DELETE THEN` clause allows for custom logic when a reference is deleted. This clause includes a parameters called `$this` to refer to the record in question, and `$reference` for the reference.

In the following example, a `person` record's `comments` field will remove any comments when they are deleted, but also add the same comment to a different field called `deleted_comments`.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ comments: [comment:7g857mbox5rdqvjuezxf], id: person:one }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ comments: [comment:7g857mbox5rdqvjuezxf, comment:o7qgd26uruvf3hracvrh], id: person:one }]"

[[test.results]]
value = "[]"

[[test.results]]
value = "[{ comments: [comment:7g857mbox5rdqvjuezxf], deleted_comments: [comment:o7qgd26uruvf3hracvrh], id: person:one }]"

*/

DEFINE FIELD comments ON person TYPE option<array<record<comment>>> REFERENCE ON DELETE THEN {
    UPDATE $this SET
        deleted_comments += $reference,
        comments -= $reference;
};
DEFINE FIELD author ON comment COMPUTED <~person;

CREATE person:one SET comments += (CREATE ONLY comment SET text = "Estonia is bigger than I expected!").id;
LET $comment = CREATE ONLY comment SET text = "I don't get the joke here?";
UPDATE person:one SET comments += $comment.id;
DELETE $comment;
SELECT * FROM person:one;
```

```surql title="person:one before and after comment is deleted"
-------- Query --------

[
	{
		comments: [
			comment:lbeyh2icushpwo0ak5ux,
			comment:90tdnyoa14cge2ocmep7
		],
		id: person:one
	}
]

-------- Query --------

[
	{
		comments: [
			comment:lbeyh2icushpwo0ak5ux
		],
		deleted_comments: [
			comment:90tdnyoa14cge2ocmep7
		],
		id: person:one
	}
]
```

## regex.mdx

---
sidebar_position: 16
sidebar_label: Regex
title: Regex | SurrealQL
description: The regex type can 

---

import Since from '@components/shared/Since.astro'

# Regex

<Since v="v3.0.0" />

A `regex` can be created by casting from a string.

The following examples all return `true`.

```surql
/**[test]

[[test.results]]
value = "true"

[[test.results]]
value = "true"

[[test.results]]
value = "true"

*/

-- Either 'a' or 'b'
<regex> "a|b" = "a";

-- Either color or colour
<regex> "col(o|ou)r" = "colour";

-- Case-insensitive match on English color, colour, or French couleur
<regex> "((?i)col(o|ou)r|couleur)" = "COULEUR";
```

While `regex` was added as a standalone type in version 2.3.0, regex matching has always been available via the [`string::matches()`](/docs/surrealql/functions/database/string#stringmatches) function.

```surql
/**[test]

[[test.results]]
value = "true"

[[test.results]]
value = "true"

[[test.results]]
value = "true"

*/

string::matches("a", "a|b");
string::matches("colour", "col(o|ou)r");
string::matches("COULEUR", "((?i)col(o|ou)r|couleur)");
```

## sets.mdx

---
sidebar_position: 17
sidebar_label: Sets
title: Sets | SurrealQL
description: A set is a collection type of deduplicated and ordered values that can have a maximum size limit.

---

# Sets

> [!NOTE]
> Before version 3.0.0-beta, sets were simply arrays that deduplicated their items. To emulate the former behaviour, add the clause `VALUE $value.distinct()` to a `DEFINE FIELD` definition.

A set is similar to an array, but with two key differences:

* The values in a set are automatically deduplicated.
* The values in a set are automatically ordered.

## Set syntax and casting

A set can be created using the literal syntax `{}`.

```surql
/**[test]

[[test.results]]
value = "{1, 2, 6}"

*/

RETURN {1, 6, 6, 2};
-- {1, 2, 6}
```

To create a set with zero items or a single item, add a comma.

```surql
/**[test]

[[test.results]]
value = "true"

[[test.results]]
value = "true"

[[test.results]]
value = "false"

[[test.results]]
value = "false"

*/
{,}.is_set();  -- true
{9,}.is_set(); -- true

{}.is_set();   -- false
{9}.is_set();  -- false
```

In addition to the `{}` literal syntax, an array can be cast into a set.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ bank_accounts: [55555, 55555, 98787], id: customer:q57ltnsy4rnpwtica0qa, languages: {'en', 'ja', 'kr'} }]"

*/

DEFINE FIELD bank_accounts ON TABLE customer TYPE array<int>;
DEFINE FIELD languages ON TABLE customer TYPE set<string>;

CREATE customer SET
    bank_accounts = [
      55555,
      55555,
      98787
    ],
    languages = <set>[
        "en",
        "ja",
        "kr",
        "kr"
    ];
```

```surql title="Output"
[
	{
		bank_accounts: [
			55555,
			98787
		],
		id: customer:uv6mn62t8td9vzvfogh4,
		languages: {
			'en',
			'ja',
			'kr'
		}
	}
]
```

Casting into a `set` and back into an array can be a convenient way to deduplicate items in the same way that the [`array::distinct()`](/docs/surrealql/functions/database/array#arraydistinct) and [`array::sort()`](/docs/surrealql/functions/database/array#arraysort) functions are used.

```surql
/**[test]

[[test.results]]
value = "[3, 4, 5, 6, 7, 9, 18]"

[[test.results]]
value = "[3, 4, 5, 6, 7, 9, 18]"

*/

<array><set>[18,7,6,6,6,6,5,4,3,9];
[18,7,6,6,6,6,5,4,3,9].distinct().sort();
```

```surql title="Output"
[3, 4, 5, 6, 7, 9, 18]
```

## Using the index operator on sets

The `[]` index operator can be used on a set in the same manner as an array. Note however that due to a set's automatic ordering, its individual values are technically not assigned an index. This can be seen in the following example in which `[0]` for an array will return whichever item happens to be at that location, while for a set `[0]` will automatically be the item with the least value.

```surql
-- Array: returns 4 at index 0
[4,6,2][0];

-- Set: returns 2, the least item
(<set>[4,6,2])[0];
```

## Filtering and mapping with set functions

SurrealDB also includes a number of methods for sets that make it easier to filter and map. These methods take a closure (an anonymous function) that works in a similar way to the `$this` parameter above.

Here is an example of the `set::filter()` method being used. Note that the parameter name inside the closure is named by the user, so `$val` in the example below could be `$v` or `$some_val` or anything else.

```surql
/**[test]

[[test.results]]
value = "{3,}"

[[test.results]]
value = ""

*/

{1,3,5}.filter(|$val| $val > 2);

-- {3,5}
```

## Adding sets

An set can be added to another set or array, resulting in a single set consisting of the items of the first followed by those of the second.

```surql
/**[test]

[[test.results]]
value = "[1, 2, 3, 4]"

[[test.results]]
value = "[1, 2, 3, 4]"

*/

{1,2} + [3,4];
{1,2} + {3,4};
```

```surql title="Output"
{1,2,3,4}
```

## strings.mdx

---
sidebar_position: 18
sidebar_label: Strings
title: Strings | SurrealQL
description: Strings can be used to store text values. All string values can include Unicode values, emojis, tab characters, and line breaks.

---

import Image from "@components/Image.astro";
import Since from '@components/shared/Since.astro'

import LightImageParseError from "@img/image/light/surrealql-parse-error.png";
import DarkImageParseError from "@img/image/dark/surrealql-parse-error.png";

# Strings

Strings can be used to store text values. All string values can include Unicode values, emojis, tab characters, and line breaks.

```surql
/**
[test]

[[test.results]]
value = "[{ id: person:95uvwaw161b9qqquxixf, text: 'Lorem ipsum dolor sit amet' }]"
skip-record-id-key = true

**/

CREATE person SET text = 'Lorem ipsum dolor sit amet';
```

Strings can be created using single quotation marks, or double quotation marks.

```surql
/**
[test]

[[test.results]]
value = "[{ id: person:7qrqr6tyodgwngpx73b1, text: 'Lorem ipsum dolor sit amet' }]"
skip-record-id-key = true

**/

CREATE person SET text = "Lorem ipsum dolor sit amet";
```

Any string in SurrealDB can include Unicode text.

```surql
/**
[test]

[[test.results]]
value = "[{ id: person:6tazjj7yko5ot2lujpfk, text: 'I ❤️ SurrealDB' }]"
skip-record-id-key = true

**/

CREATE person SET text = "I ❤️ SurrealDB";
```

Strings can also include line breaks.

```surql
/**
[test]

[[test.results]]
value = "[{ id: person:v4lgc1sfs4xegf86gfcv, text: 'This
is
over
multiple
lines' }]"
skip-record-id-key = true

**/

CREATE person SET text = "This
is
over
multiple
lines";
```

## Specifying data type literal values using string prefixes



### Overview

In SurrealQL, there are several data types for which literal values are specified using string values, with a prefix indicating the intended type for the value to be interpreted as.

Previously, in SurrealQL version `1.0`, literal values of these types were simply specified using a string without any prefix, and SurrealDB would eagerly convert the strings into the relevant data type in any case where the string matched the format expected for that type. However, since SurrealQL version `2.0`, strings are no longer eagerly converted into other data types. Instead, if you want to specify a literal value of one of these data types, you must explicitly use a string with the appropriate prefix.

### Record ID literal values using the `r` prefix {#record}

The `r` prefix tells the parser that the contents of the string represent a [`record ID`](/docs/surrealql/datamodel/ids). The parser expects record IDs to have the following format: `table_name:record ID`.

> [!NOTE]
> All strings since SurrealDB 2.0 without the `r` prefix are of type `string` and are not parsed as records unless the prefix is present.

Here is an example of a record ID literal value, specified using a string with the `r` prefix.

```surql
/**
[test]

[[test.results]]
value = "person:john"

**/

RETURN r"person:john";
```

```surql title="Response"
-------- Query 1 --------

person:john
```

In the example below, using the [`type::is_string()`](/docs/surrealql/functions/database/type#typeis_string) and [`type::is_record()`](/docs/surrealql/functions/database/type#typeis_record) functions respectively, you can check the type of the string.

```surql
/**
[test]

[[test.results]]
value = "true"

[[test.results]]
value = "false"

[[test.results]]
value = "true"

**/

RETURN type::is_string("person:john");
RETURN type::is_record("person:john");
RETURN type::is_record(r"person:john");
```
```surql title="Response"
-------- Query 1 --------

true

-------- Query 2 --------

false

-------- Query 3 --------

true
```

### Datetime literal values using the `d` prefix {#datetime}

The `d` prefix tells the parser that the contents of the string represent a [`datetime`](/docs/surrealql/datamodel/datetimes). The parser expects `datetime` values to have a valid [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) format. Here are a few examples:


```surql
/**
[test]

[[test.results]]
value = "d'2025-11-28T11:41:20.262Z'"

[[test.results]]
value = "d'2025-11-28T07:41:20.262Z'"

[[test.results]]
value = "d'2025-11-28T15:41:20.262Z'"

[[test.results]]
value = "d'2025-11-28T11:41:20Z'"

[[test.results]]
value = "d'2025-11-28T07:41:20Z'"

**/

RETURN d"2025-11-28T11:41:20.262Z";       --- Sub-second precision included, timezone defaulted to UTC
RETURN d"2025-11-28T11:41:20.262+04:00";  --- Sub-second precision included, timezone specified as UTC + 4:00
RETURN d"2025-11-28T11:41:20.262-04:00";  --- Sub-second precision included, timezone specified as UTC - 4:00
RETURN d"2025-11-28T11:41:20Z";           --- Sub-second precision excluded, timezone defaulted to UTC
RETURN d"2025-11-28T11:41:20+04:00";      --- Sub-second precision excluded, timezone specified as UTC + 4:00
```
```surql title="Response"
-------- Query 1 --------

d'2025-11-28T11:41:20.262Z'

-------- Query 2 --------

d'2025-11-28T07:41:20.262Z'

-------- Query 3 --------

d'2025-11-28T15:41:20.262Z'

-------- Query 4 --------

d'2025-11-28T11:41:20Z'

-------- Query 5 --------

d'2025-11-28T07:41:20Z'
```

### UUID literal values with the `u` prefix {#uuid}

The `u` prefix tells the parser that the contents of the string represent a [`uuid`](/docs/surrealql/datamodel/uuid). The parser expects `uuid` values to follow the format of an UUID, `ffffffff-ffff-ffff-ffff-ffffffffffff`, where each non-hyphen character can be a digit (0-9) or a letter between `a` and `f` (representing a single hexadecimal digit).

```surql
/**
[test]

[[test.results]]
value = "u'8c54161f-d4fe-4a74-9409-ed1e137040c1'"

**/

RETURN u"8c54161f-d4fe-4a74-9409-ed1e137040c1";
```

```surql title="Response"
-------- Query 1 --------

u'8c54161f-d4fe-4a74-9409-ed1e137040c1'
```

### Byte values using the `b` prefix {#bytes}

```surql
/**
[test]

[[test.results]]
value = "b"0099AAFF""

**/

b"0099aaff"
```

### File paths using the `f` prefix {#files}

```surql
f"bucket:/some/key/to/a/file.txt";
f"bucket:/some/key/with\ escaped";
f"bucket:/some/key".put(b"00aa");
f"bucket:/some/key".get();
```

### String prefixes vs. casting

String prefixes seem outwardly similar to casting, but differ in behaviour. A string prefix is an instruction to the parser to treat an input in a certain way, whereas a cast is an instruction to the database to convert one type into another.

As a result, incorrect input with a cast will generate an error:

```surql
// Change _ to - in both examples to fix the input
RETURN <uuid>"018f0e6a_9b95-7ecc-8a38-aea7bf3627dd";
RETURN <datetime>"2024_06-06T12:00:00Z";
```
```surql title="Response"
-------- Query 1 --------

"Expected a uuid but cannot convert '018f0e6a-9b95-7ecc-8a38-aea7bf3627d' into a uuid"

-------- Query 2 --------

"Expected a datetime but cannot convert '2024-06-06T12:00:00' into a datetime"
```

But the same input using a string prefix will not even parse until the input is valid.

```surql
// Will not parse in either case until _ is changed to -
RETURN u"018f0e6a_9b95-7ecc-8a38-aea7bf3627dd";
RETURN d"2024_06-06T12:00:00Z";
```

This also allows for immediate error messages on which part of the input is incorrect. As seen in the image below, the parser is able to inform the user that an underscore at column 18 is the issue.

<Image
  alt="A screenshot showing how a string prefix allows incorrect UUID input to be identified before a query can be run. In this case, the parser is able to inform the user that an underscore at column 18 is the issue."
  src={{
    light: LightImageParseError,
    dark: DarkImageParseError,
  }}
/>


## uuid.mdx

---
sidebar_position: 19
sidebar_label: UUIDs
title: UUIDs | SurrealQL
description: UUID values in SurrealQL represent UUID values

---

# UUIDs

UUIDs represent UUID v4 and v7 values. They can be obtained via either the:
- [`rand::uuid::*` functions](/docs/surrealql/functions/database/rand#randuuidv4)
- [casted from strings](/docs/surrealql/datamodel/casting#uuid)
- or via [string prefixes](/docs/surrealql/datamodel/strings#uuid)


> [!NOTE]
> As of `v2.0.0`, SurrealDB no longer eagerly converts a string into a UUID. An implicit `u` prefix or cast using `<uuid>` is required instead.

```surql
/**[test]

[[test.results]]
value = "u'4048a7ac-9c1a-486e-9a12-e39d76670eac'"
skip-uuid = true

[[test.results]]
value = "u'0199a85c-5e24-7901-9053-9d898d881d9e'"
skip-uuid = true

[[test.results]]
value = "u'a8f30d8b-db67-47ec-8b38-ef703e05ad1b'"

[[test.results]]
value = "u'a8f30d8b-db67-47ec-8b38-ef703e05ad1b'"

*/

rand::uuid::v4();
rand::uuid::v7();
<uuid> "a8f30d8b-db67-47ec-8b38-ef703e05ad1b";
u"a8f30d8b-db67-47ec-8b38-ef703e05ad1b";
```


## values.mdx

---
sidebar_position: 20
sidebar_label: Values
title: Values | SurrealQL
description: Every type in SurrealDB is a value

---

# Values

Each of the types mentioned in the data model is a subset of an all-encompassing type called a value.

## Comparing and ordering values

While it is unsurprising that a data type can be compared with itself, it may be surprising that different types can also be compared with each other.

```surql
/**[test]

[[test.results]]
value = "true"

[[test.results]]
value = "true"

*/

RETURN 9 > 1;            // Returns true
RETURN [] > time::now(); // Also returns true
```

This comparison is possible because every type in SurrealDB is a subset of value, and a comparison of any type with another is also simply a comparison of a value with another value. The order of values from least to greatest is:

* `none`
* `null`
* `bool`
* `number`
* `string`
* `duration`
* `datetime`
* `uuid`
* `array`
* `set`
* `object`
* `geometry`
* `bytes`
* `table`
* `record`
* `file`
* `regex`
* `range`

As a result, all of the following return `true`.

```surql
/**[test]

[[test.results]]
value = "[true, true, true, true, true, true, true, true, true, true, true, true]"

*/

RETURN [
    null > none,
    true > null,
    1 > true,
    'a' > 999999999,
    1m > 'a',
    time::now() > 1m,
    rand::uuid() > time::now(),
    [ 9, 10 ] > rand::uuid(),
    { 9, 10 } > [ 9, 10 ],
    {} > { 9, 10 },
    (9.9, 9.9) > {},
    <bytes>"Aeon" > (9.9, 9.9),
    type::table("person") > <bytes>"Aeon",
    person:one > type::table("person"),
    f"file://myfile.txt" > person:one,
    <regex>"a|b" > f"file://myfile.txt",
    0..10 > <regex>"a|b",
    || > 0.. 10
];
```

Being able to compare a value with any other value is what makes SurrealDB's record range syntax possible.

```surql
/**[test]

[[test.results]]
value = "[{ id: time_data:[d'2024-07-23T00:00:00Z'] }]"

[[test.results]]
value = "[{ id: time_data:[d'2024-07-24T00:00:00Z'] }]"

[[test.results]]
value = "[{ id: time_data:[d'2024-07-25T00:00:00Z'] }]"

[[test.results]]
value = "[{ id: time_data:[d'2024-07-24T00:00:00Z'] }]"

[[test.results]]
value = "[{ id: time_data:[d'2024-07-24T00:00:00Z'] }, { id: time_data:[d'2024-07-25T00:00:00Z'] }]"

[[test.results]]
value = "[{ id: time_data:[d'2024-07-23T00:00:00Z'] }, { id: time_data:[d'2024-07-24T00:00:00Z'] }, { id: time_data:[d'2024-07-25T00:00:00Z'] }]"

*/

CREATE time_data:[d'2024-07-23T00:00:00.000Z'];
CREATE time_data:[d'2024-07-24T00:00:00.000Z'];
CREATE time_data:[d'2024-07-25T00:00:00.000Z'];
-- Records from the 24th to the 25th
SELECT * FROM time_data:[d'2024-07-24']..[d'2024-07-25'];
-- Records from the 24th
SELECT * FROM time_data:[d'2024-07-24']..;
-- All records
SELECT * FROM time_data:[NONE]..;
```

The `..` open-range syntax also represents an infinite value inside a record range query, making it the greatest possible value and the inverse of `NONE`, the lowest possible value. A part of a record range query that begins with `NONE` and ends with `..` will thus filter out nothing.

```surql
/**[test]

[[test.results]]
value = "[{ id: temperature:['London', d'2025-02-19T00:00:00Z'], val: 5.5f }]"

[[test.results]]
value = "[{ id: temperature:['London', d'2025-02-20T00:00:00Z'], val: 5.7f }]"

[[test.results]]
value = "[{ id: temperature:['London', d'2025-02-19T00:00:00Z'], val: 5.5f }, { id: temperature:['London', d'2025-02-20T00:00:00Z'], val: 5.7f }]"

*/

CREATE temperature:['London', d'2025-02-19T00:00:00.000Z'] SET val = 5.5;
CREATE temperature:['London', d'2025-02-20T00:00:00.000Z'] SET val = 5.7;

-- Return all records as long as index 0 = 'London'
SELECT * FROM temperature:['London', NONE]..=['London', ..];
```

```surql title="Output"
[
	{
		id: temperature:[
			'London',
			d'2025-02-19T00:00:00Z'
		],
		val: 5.5f
	},
	{
		id: temperature:[
			'London',
			d'2025-02-20T00:00:00Z'
		],
		val: 5.7f
	}
]
```

Inside a schema, the keyword `any` is used to denote any possible value.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE FIELD anything ON TABLE person TYPE any;
```

## Values and truthiness

Any value is considered to be truthy if it is not NONE, NULL, or a default value for the data type. A data type at its default value is one that is empty, such as an empty string or array or object, or a number set to 0.

The following example shows the result of the `array::all()` method, which checks to see if all of the items inside an array are truthy or not.

```surql
/**[test]

[[test.results]]
value = "false"

[[test.results]]
value = "false"

[[test.results]]
value = "true"

*/

RETURN array::all(["", 1, 2, 3]); // false because of ""
RETURN array::all([{}, 1, 2, 3]); // false because of {}
RETURN array::all(["SurrealDB", { is_nice_database: true }, 1, 2, 3]);  // true
```

As [the ! operator](/docs/surrealql/operators) reverses the truthiness of a value, a doubling of this operator can also be used to check for truthiness.

```surql
/**[test]

[[test.results]]
value = "[true, false, true, false, true, false]"

*/

RETURN [
    !!"Has a value", !!"",             // true, false
    !!true, !!false,                   // true, false
    !!{ is_nice_database: true }, !!{} // true, false
    ];
```

## index.mdx

---
sidebar_position: 1
sidebar_label: Overview
title: SurrealQL | Query Language
description: In this section, you will explore SurrealQL, a powerful database query language that closely resembles traditional SQL but comes with unique differences and improvements.
no_page_headings: true
---

import Image from "@components/Image.astro";
import LightLogo from "@img/icon/light/ql-light.png";
import DarkLogo from "@img/icon/dark/surrealql.png";

<div class="flag-title">
    <Image
        class="size-11 my-auto"
        alt="SurrealQL"
        width={300}
        src={{
            light: LightLogo,
            dark: DarkLogo,
        }}
    />
    # SurrealQL
</div>

SurrealQL is a powerful and intuitive database query language that closely resembles traditional SQL but comes with unique differences and improvements.

SurrealQL is designed to provide developers with a seamless and intuitive way to interact with SurrealDB. It offers a familiar syntax and supports various statement types, allowing you to perform complex database operations efficiently.

While SurrealQL shares similarities with traditional SQL, it introduces enhancements and optimizations that make it well-suited for working with SurrealDB's advanced features and data models. Whether you are querying data, modifying records, or managing database structures, SurrealQL provides a comprehensive set of capabilities to meet your needs.

## Key Features

SurrealQL offers several key features that make it a powerful tool for working with SurrealDB:

- **Familiar Syntax**: SurrealQL adopts a syntax similar to traditional SQL, making it easy for developers familiar with SQL to transition to SurrealDB seamlessly.

- **Advanced Querying**: SurrealQL supports a wide range of querying capabilities, including filtering, sorting, aggregating, and joining data from multiple tables.

- **Data Manipulation**: With SurrealQL, you can easily insert, update, and delete records in your SurrealDB database, allowing you to manage your data effectively.

- **Graph relationships**: SurrealQL supports graph relationships, allowing you to define and query relationships between records in your database.

- **Schema Management**: SurrealQL provides features for creating and modifying database schemas, allowing you to define the structure of your data and enforce data integrity.

- **Performance Optimization**: SurrealQL incorporates optimizations specific to SurrealDB, ensuring efficient execution of queries and minimizing resource usage.

## Getting Started

To start using [SurrealQL](/docs/surrealql/statements/begin), refer to the documentation on the various statement types and their syntax. The statements page provides comprehensive examples and explanations for each statement type, helping you understand how to construct queries and interact with SurrealDB effectively.

SurrealQL empowers you to leverage the full potential of SurrealDB and enables you to build robust and scalable applications. Let's dive into the world of SurrealQL and unlock the capabilities of SurrealDB together!

<iframe width="100%" src="https://www.youtube.com/embed/TyX45cyZ-W0?si=S9M59afDEiqxeC5d" style={{ aspectRatio: 1.7, paddingTop: '20px' }} title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

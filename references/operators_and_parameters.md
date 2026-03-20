# SurrealQL Operators & Parameters


## operators.mdx

---
sidebar_position: 3
sidebar_label: Operators
title: Operators | SurrealQL
description: A variety of operators in SurrealQL allow for complex manipulation of data, and advanced logic.
---

import Since from '@components/shared/Since.astro'
import Table from '@components/shared/Table.astro'

# Operators

A variety of operators in SurrealQL allow for complex manipulation of data, and advanced logic.

<Table>
	<thead>
		<tr>
			<th scope="col" class="w-48">Operator</th>
			<th scope="col">Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex gap-2">
					<a href="#and">
						<code>&&</code>
					</a>
					<a href="#and">
						<code>AND</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether both of two values are truthy
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex gap-2">
					<a href="#or">
						<code>||</code>
					</a>
					<a href="#or">
						<code>OR</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether either of two values is truthy
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#not">
					<code>!</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Reverses the truthiness of a value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#not_not">
					<code>!!</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Determines the truthiness of a value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#nco">
					<code>??</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether either of two values are truthy and not NULL
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#tco">
					<code>?:</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether either of two values are truthy
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex gap-2">
					<a href="#equal">
						<code>=</code>
					</a>
					<a href="#equal">
						<code>IS</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Check whether two values are equal
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex gap-2">
					<a href="#notequal">
						<code>!=</code>
					</a>
					<a href="#notequal">
						<code>IS NOT</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Check whether two values are not equal
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#exact">
					<code>==</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether two values are exactly equal
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#anyequal">
					<code>?=</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether any value in a set is equal to a value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#allequal">
					<code>*=</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether all values in a set are equal to a value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#match">
					<code>~</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Compare two values for equality using fuzzy matching
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#match">
					<code>!~</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Compare two values for inequality using fuzzy matching
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#match">
					<code>?~</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether any value in a set is equal to a value using
				fuzzy matching
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#match">
					<code>*~</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether all values in a set are equal to a value using
				fuzzy matching
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#lessthan">
					<code>&lt;</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether a value is less than another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#lessthanorequal">
					<code>&lt;=</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether a value is less than or equal to another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#greaterthan">
					<code>&gt;</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether a value is greater than another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#greaterthanorequal">
					<code>&gt;=</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Check whether a value is greater than or equal to another
				value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#add">
					<code>+</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Add two values together
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#sub">
					<code>-</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Subtract a value from another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex gap-2">
					<a href="#mul">
						<code>*</code>
					</a>
					<a href="#mul">
						<code>×</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Multiply two values together
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex gap-2">
					<a href="#div">
						<code>/</code>
					</a>
					<a href="#div">
						<code>÷</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Divide a value by another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#pow">
					<code>**</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Raises a base value by another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col gap-2">
					<a href="#contains">
						<code>CONTAINS</code>
					</a>
					<a href="#contains">
						<code>∋</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether a value contains another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col gap-2">
					<a href="#containsnot">
						<code>CONTAINSNOT</code>
					</a>
					<a href="#containsnot">
						<code>∌</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether a value does not contain another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col gap-2">
					<a href="#containsall">
						<code>CONTAINSALL</code>
					</a>
					<a href="#containsall">
						<code>⊇</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether a value contains all other values
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col gap-2">
					<a href="#containsany">
						<code>CONTAINSANY</code>
					</a>
					<a href="#containsany">
						<code>⊃</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether a value contains any other value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col gap-2">
					<a href="#containsnone">
						<code>CONTAINSNONE</code>
					</a>
					<a href="#containsnone">
						<code>⊅</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether a value contains none of the following values
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col items-start gap-2">
					<a href="#inside">
						<code>INSIDE</code>
					</a>
					<a href="#inside">
						<code>IN</code>
					</a>
					<a href="#inside">
						<code>∈</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether a value is contained within another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col items-start gap-2">
					<a href="#notinside">
						<code>NOTINSIDE</code>
					</a>
					<a href="#notinside">
						<code>NOT IN</code>
					</a>
					<a href="#notinside">
						<code>∉</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether a value is not contained within another value
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col items-start gap-2">
					<a href="#allinside">
						<code>ALLINSIDE</code>
					</a>
					<a href="#allinside">
						<code>⊆</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether all values are contained within other values
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col items-start gap-2">
					<a href="#anyinside">
						<code>ANYINSIDE</code>
					</a>
					<a href="#anyinside">
						<code>⊂</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether any value is contained within other values
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col items-start gap-2">
					<a href="#noneinside">
					<code>NONEINSIDE</code>
					</a>
					<a href="#noneinside">
						<code>⊄</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether no value is contained within other values
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#outside">
					<code>OUTSIDE</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Checks whether a geometry type is outside of another
				geometry type
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<a href="#intersects">
					<code>INTERSECTS</code>
				</a>
			</td>
			<td scope="row" data-label="Description">
				Checks whether a geometry type intersects another geometry
				type
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col items-start gap-2">
					<a href="#matches">
						<code>@@</code>
					</a>
					<a href="#matches">
						<code>@[ref]@</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Checks whether the terms are found in a full-text indexed
				field
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Operator">
				<div class="flex flex-col items-start gap-2">
					<a href="#knn">
						<code> &lt;|4|&gt; </code>
					</a>
					<a href="#knn">
						<code>&lt;|3,HAMMING| &gt;</code>
					</a>
				</div>
			</td>
			<td scope="row" data-label="Description">
				Performs a K-Nearest Neighbors (KNN) search to find a
				specified number of records closest to a given data point,
				optionally using a defined distance metric. Supports
				customizing the number of results and choice of distance
				calculation method.
			</td>
		</tr>
	</tbody>
</Table>

## `&&` or `AND` {#and}

The `and` operator checks whether both of two values are [truthy](/docs/surrealql/datamodel/values#values-and-truthiness).

```surql
/**[test]

[[test.results]]
value = "30"

*/

SELECT * FROM 10 AND 20 AND 30;

-- 30
```

<br />

## `||` or `OR` {#or}

The `or` operator checks whether either of two values are [truthy](/docs/surrealql/datamodel/values#values-and-truthiness).

```surql
/**[test]

[[test.results]]
value = "[10]"

*/

SELECT * FROM 0 OR false OR 10;

-- 10
```

<br />

## `!` {#not}

The `not` operator reverses the truthiness of a value.

```surql
/**[test]

[[test.results]]
value = "[false]"

[[test.results]]
value = "[false]"

*/

SELECT * FROM !(TRUE OR FALSE);
-- false

SELECT * FROM !"Has a value";
-- false
```

<br />

## `!!` {#not_not}

The `not not` operator is simply an application of the `!` operator twice. It can be used to determines the truthiness of a value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM !!"Has a value";
-- true
```

## `??` {#nco}

The `null coalescing operator` checks whether either of two values are [truthy](/docs/surrealql/datamodel/values#values-and-truthiness) and not `NONE` or `NULL`.

```surql
/**[test]

[[test.results]]
value = "[0]"

*/

SELECT * FROM NULL ?? 0 ?? false ?? 10;

-- 0
```

<br />

## `?:` {#tco}

The `truthy coalescing operator` checks whether either of two values are [truthy](/docs/surrealql/datamodel/values#values-and-truthiness).

```surql
/**[test]

[[test.results]]
value = "[10]"

*/

SELECT * FROM NULL ?: 0 ?: false ?: 10;

-- 10
```

<br />

## `=` or `IS` {#equal}

The `equal` operator checks whether two values are equal.

```surql
/**[test]

[[test.results]]
value = "[false]"

*/

SELECT * FROM true = "true";
-- false
```

```surql
/**[test]

[[test.results]]
value = "[false]"

*/

SELECT * FROM 10 = "10";
-- false
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 10 = 10.00;
-- true
```
```surql
/**[test]

[[test.results]]
value = "[false]"

*/

SELECT * FROM 10 = "10.3";
-- false
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [1, 2, 3] = [1, 2, 3];
-- true
```

```surql
/**[test]

[[test.results]]
value = "[false]"

*/

SELECT * FROM [1, 2, 3] = [1, 2, 3, 4];
-- false
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM { this: "object" } = { this: "object" };
-- true
```

```surql
/**[test]

[[test.results]]
value = "[false]"

*/

SELECT * FROM { this: "object" } = { another: "object" };
-- false
```

<br />

## `!=` or `IS NOT` {#notequal}

The `not equal` operator checks whether two values are not equal.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 10 != "15";
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 10 != "test";
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [1, 2, 3] != [3, 4, 5];
-- true
```

<br />

## `==` {#exact}

The `exact` operator checks whether two values are exact. This operator also checks that each value has the same type.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 10 == 10;
-- true
```

```surql
/**[test]

[[test.results]]
value = "[false]"

*/

SELECT * FROM 10 == "10";
-- false
```

```surql
/**[test]

[[test.results]]
value = "[false]"

*/

SELECT * FROM true == "true";
-- false
```

<br />

## `?=` {#anyequal}

The `any equal` operator checks whether any value in an array equals another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [10, 15, 20] ?= 10;
-- true
```

<br />

## `*=` {#allequal}

The `all equal` operator checks whether all values in an array equals another value.

```surql
/**[test]

[[test.results]]
value = ""

*/

SELECT * FROM [10, 10, 10] *= 10;
-- true
```

<br />

## `~` `?~` `!~` `*~` {#match}

These operators used to compare two values for equality using fuzzy matching. They have been removed since 3.0 to avoid implicitly preferring one algorithm over another, as the type of fuzzy matching to use will depend on each individual case.

Please use the `string::similarity::*` functions instead:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "true"

*/

let $threshold = 10;

string::similarity::smithwaterman("test text", "Test") > $threshold;
-- true
```

<br />

## `<` {#lessthan}

The `less than` operator checks whether a value is less than another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 10 < 15;
-- true
```

<br />

## `<=` {#lessthanorequal}

The `less than or equal` operator checks whether a value is less than or equal to another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 10 <= 15;
-- true
```

<br />

## `>` {#greaterthan}

The `greater than` operator checks whether a value is less than another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 15 > 10;
-- true
```

<br />

## `>=` {#greaterthanorequal}

The `greater than or equal` operator checks whether a value is less than or equal to another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 15 >= 10;
-- true
```

<br />

## `+` {#add}

The `add` operator adds two values together.

```surql
/**[test]

[[test.results]]
value = "[20]"

*/

SELECT * FROM 10 + 10;
-- 20
```

```surql
/**[test]

[[test.results]]
value = "['test this']"

*/

SELECT * FROM "test" + " " + "this";
-- "test this"
```

```surql
/**[test]

[[test.results]]
value = "[13h30m]"

*/

SELECT * FROM 13h + 30m;
-- "13h30m"
```

<br />

## `-` {#sub}

The `subtract` operator subtracts a value from another value.

```surql
/**[test]

[[test.results]]
value = "[10]"

*/

SELECT * FROM 20 - 10;
-- 10
```

```surql
/**[test]

[[test.results]]
value = "[1m]""

*/

SELECT * FROM 2m - 1m;
-- 1m
```

<br />

## `*` or `×` {#mul}

The `multiply` operator multiplies a value by another value.

```surql
/**[test]

[[test.results]]
value = "[40]"

*/

SELECT * FROM 20 * 2;
-- 40
```

<br />

## `/` or `÷` {#div}

The `divide` operator divides a value by another value.

```surql
/**[test]

[[test.results]]
value = "[10]"

*/

SELECT * FROM 20 / 2;
-- 10
```

<br />

## `**` {#pow}

The `power` operator raises a base value by another value.

```surql
/**[test]

[[test.results]]
value = "[8000]"

*/

SELECT * FROM 20 ** 3;
-- 8000
```

<br />

## `CONTAINS` or `∋` {#contains}

The `contains` operator checks whether a value contains another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [10, 20, 30] CONTAINS 10;
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM "this is some text" CONTAINS "text";
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM {
	type: "Polygon",
	coordinates: [[
		[-0.38314819, 51.37692386], [0.1785278, 51.37692386],
		[0.1785278, 51.61460570], [-0.38314819, 51.61460570],
		[-0.38314819, 51.37692386]
	]]
} CONTAINS (-0.118092, 51.509865);

-- true
```

<br />

## `CONTAINSNOT` or `∌` {#containsnot}

The `not contains` operator checks whether a value does not contain another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [10, 20, 30] CONTAINSNOT 15;
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM "this is some text" CONTAINSNOT "other";
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM {
	type: "Polygon",
	coordinates: [[
		[-0.38314819, 51.37692386], [0.1785278, 51.37692386],
		[0.1785278, 51.61460570], [-0.38314819, 51.61460570],
		[-0.38314819, 51.37692386]
	]]
} CONTAINSNOT (-0.518092, 53.509865);

-- true
```

<br />

## `CONTAINSALL` or `⊇` {#containsall}

The `contains all` operator checks whether a value contains all of multiple values.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [10, 20, 30] CONTAINSALL [10, 20, 10];
-- true
```

<br />

## `CONTAINSANY` or `⊃` {#containsany}

The `contains any` operator checks whether a value contains any of multiple values.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [10, 20, 30] CONTAINSANY [10, 15, 25];
-- true
```

<br />

## `INSIDE` or `∈` or `IN` {#inside}

The `inside` operator checks whether a value is contained within another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 10 INSIDE [10, 20, 30];
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM "text" INSIDE "this is some text";
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM (-0.118092, 51.509865) INSIDE {
	type: "Polygon",
	coordinates: [[
		[-0.38314819, 51.37692386], [0.1785278, 51.37692386],
		[0.1785278, 51.61460570], [-0.38314819, 51.61460570],
		[-0.38314819, 51.37692386]
	]]
};

true
```

<Since v="v2.1.0" />

This operator can also be used to check for the existence of a key inside an [object](/docs/surrealql/datamodel/objects). To do so, precede `IN` with the field name as a string.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

"name" IN {
    name: "Riga",
    country: "Latvia"
};

-- true
```

`IN` can also be used with a record ID as long as the ID is expanded to include the fields. Both of the following queries will return `true`.

```surql
/**[test]

[[test.results]]
value = "[{ country: 'Latvia', id: city:riga, name: 'Riga', population: 605273 }]"

[[test.results]]
value = "true"

[[test.results]]
value = "true"

*/

CREATE city:riga SET name = "Riga", country = "Latvia", population = 605273;

"name" IN city:riga.*;
"name" IN city:riga.{ name, country };
```

<br />

## `NOTINSIDE` or `∉` or `NOT IN` {#notinside}

The `not inside` operator checks whether a value is not contained within another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM 15 NOTINSIDE [10, 20, 30];
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM "other" NOTINSIDE "this is some text";
-- true
```

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM (-0.518092, 53.509865) NOTINSIDE {
	type: "Polygon",
	coordinates: [[
		[-0.38314819, 51.37692386], [0.1785278, 51.37692386],
		[0.1785278, 51.61460570], [-0.38314819, 51.61460570],
		[-0.38314819, 51.37692386]
	]]
};

-- true
```

<br />

## `ALLINSIDE` or `⊆` {#allinside}

The `all inside` operator checks whether all of multiple values are contained within another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [10, 20, 10] ALLINSIDE [10, 20, 30];
-- true
```

<br />

## `ANYINSIDE` or `⊂` {#anyinside}

The `any inside` operator checks whether any of multiple values are contained within another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [10, 15, 25] ANYINSIDE [10, 20, 30];
-- true
```

<br />

## `NONEINSIDE` or `⊄` {#noneinside}

The `none inside` operator checks whether none of multiple values are contained within another value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM [15, 25, 35] NONEINSIDE [10, 20, 30];
-- true
```

<br />

## `OUTSIDE` {#outside}

The `outside` operator checks whether a geometry value is outside another geometry value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM (-0.518092, 53.509865) OUTSIDE {
	type: "Polygon",
	coordinates: [[
		[-0.38314819, 51.37692386], [0.1785278, 51.37692386],
		[0.1785278, 51.61460570], [-0.38314819, 51.61460570],
		[-0.38314819, 51.37692386]
	]]
};

-- true
```

<br />

## `INTERSECTS` {#intersects}

The `intersects` operator checks whether a geometry value intersects another geometry value.

```surql
/**[test]

[[test.results]]
value = "[true]"

*/

SELECT * FROM {
	type: "Polygon",
	coordinates: [[
		[-0.38314819, 51.37692386], [0.1785278, 51.37692386],
		[0.1785278, 51.61460570], [-0.38314819, 51.61460570],
		[-0.38314819, 51.37692386]
	]]
} INTERSECTS {
	type: "Polygon",
	coordinates: [[
		[-0.11123657, 51.53160074], [-0.16925811, 51.51921169],
		[-0.11466979, 51.48223813], [-0.07381439, 51.51322956],
		[-0.11123657, 51.53160074]
	]]
};

-- true
```

<br />

## `MATCHES` {#matches}

The `matches` operator checks whether the terms are found in a full-text indexed field.

```surql
SELECT * FROM book WHERE title @@ 'rust web';


[
	{
		id: book:1,
		title: 'Rust Web Programming'
	}
]
```
Using the matches operator with a reference checks whether the terms are found, highlights the searched terms, and computes the full-text score.

```surql
SELECT id,
		search::highlight('<b>', '</b>', 1) AS title,
		search::score(1) AS score
FROM book
WHERE title @1@ 'rust web'
ORDER BY score DESC;

[
	{
		id: book:1,
		score: 0.9227996468544006f,
		title: '<b>Rust</b> <b>Web</b> Programming'
	}
]
```

<Since v="v3.0.0" />

### `AND`, `OR`, and numeric operators inside `@@`

In addition to the `AND` keyword, the `OR` matches operator can also be used as of 3.0.0-beta. This allows a single string to be compared against instead of needing to specify individual parts of the string.

```surql
/**[test]

[[test.results]]
value = "[{ id: document:1, text: 'It is rare that I find myself penning a personal note in my chronicles.' }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: document:1, text: 'It is rare that I find myself penning a personal note in my chronicles.' }]"

[[test.results]]
value = "[{ id: document:1, text: 'It is rare that I find myself penning a personal note in my chronicles.' }]"

[[test.results]]
value = "[{ id: document:1, text: 'It is rare that I find myself penning a personal note in my chronicles.' }]"

[[test.results]]
value = "[{ id: document:1, text: 'It is rare that I find myself penning a personal note in my chronicles.' }]"

[[test.results]]
value = "[{ id: document:1, text: 'It is rare that I find myself penning a personal note in my chronicles.' }]"

*/

CREATE document:1 SET text = "It is rare that I find myself penning a personal note in my chronicles.";
DEFINE ANALYZER simple TOKENIZERS blank,class FILTERS lowercase;
DEFINE INDEX some_index ON document FIELDS text FULLTEXT ANALYZER simple;

-- @AND@ and @OR@: can use the entire string
SELECT * FROM document WHERE text @AND@ "personal rare";
SELECT * FROM document WHERE text @OR@ "personal nice weather today";

-- Separate AND and OR outside of matches operator:
-- Must specify parts of string to check for match
SELECT * FROM document WHERE text @@ "personal" AND text @@ "rare";
SELECT * FROM document WHERE text @@ "personal note";
SELECT * FROM document WHERE text @@ "personal" OR text @@ "nice weather today";
```

## `KNN`



K-Nearest Neighbors (KNN) is a fundamental algorithm used for classifying or regressing based on the closest data points in the feature space, with its performance and scalability critical in applications involving large datasets.

In practice, the efficiency and scalability of the KNN algorithm are crucial, especially when dealing with large datasets. Different implementations of KNN are tailored to optimize these aspects without compromising the accuracy of the results.

SurrealDB supports different K-Nearest Neighbors methods to perform KNN searches, each with unique requirements for syntax.
Below are the details for each method, including how to format your query with examples:

### Brute Force Method

Best for smaller datasets or when the highest accuracy is required.

```syntax title="SurrealQL Syntax"
<|K,DISTANCE_METRIC|>
```

- K: The number of nearest neighbors to retrieve.
- DISTANCE_METRIC: The metric used to calculate distances, such as EUCLIDEAN or MANHATTAN.

```surql
/**[test]

[[test.results]]
value = "[{ id: pts:3, point: [8, 9, 10, 11] }]"

[[test.results]]
value = "[{ id: pts:3 }]"

*/

CREATE pts:3 SET point = [8,9,10,11];
SELECT id FROM pts WHERE point <|2,EUCLIDEAN|> [2,3,4,5];
```

### HNSW Method



Recommended for very large datasets where speed is essential and some loss of accuracy is acceptable.

```syntax title="SurrealQL Syntax"
<|K,EF|>
```

- K: The number of nearest neighbors.
- EF: The size of the dynamic candidate list during the search, affecting the search's accuracy and speed.

```surql
/**[test]

[[test.results]]
value = "[{ id: pts:3, point: [8, 9, 10, 11] }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: pts:3 }]"

*/

CREATE pts:3 SET point = [8,9,10,11];
DEFINE INDEX mt_pts ON pts FIELDS point HNSW DIMENSION 4 DIST EUCLIDEAN EFC 150 M 12;
SELECT id FROM pts WHERE point <|10,40|> [2,3,4,5];
```
<br /><br />

## Using the `ANY`/`ALL` operators for string indexes

<Since v="v2.4.0" />

An index defined on a string value can be used via the operators `CONTAINSANY`, `ALLINSIDE`, or `ANYINSIDE`. The operator `CONTAINS`, however, will not use a defined index as `CONTAINS` is used for substring matches between strings themselves as opposed to an index lookup.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: account:billy, name: 'Billy McConnell' }]"

[[test.results]]
value = "[{ id: account:billy, name: 'Billy McConnell' }]"

[[test.results]]
value = "[{ id: account:billy, name: 'Billy McConnell' }]"

[[test.results]]
value = "[{ id: account:billy, name: 'Billy McConnell' }], [{ detail: { direction: 'forward', table: 'account' }, operation: 'Iterate Table' }, { detail: { type: 'Memory' }, operation: 'Collector' }, { detail: { type: 'KeysAndValues' }, operation: 'RecordStrategy' }, { detail: { count: 1 }, operation: 'Fetch' }]"

[[test.results]]
value = "[{ detail: { plan: { index: 'name_index', operator: 'union', value: ['Billy McConnell'] }, table: 'account' }, operation: 'Iterate Index' }, { detail: { type: 'Memory' }, operation: 'Collector' }, { detail: { type: 'KeysAndValues' }, operation: 'RecordStrategy' }, { detail: { count: 1 }, operation: 'Fetch' }]"

*/

DEFINE FIELD name ON account TYPE string;
DEFINE INDEX name_index ON account FIELDS name;

CREATE account:billy SET name = "Billy McConnell";

-- Both return the user Billy McConnell
SELECT * FROM account WHERE name CONTAINS "Billy McConnell";
SELECT * FROM account WHERE name CONTAINSANY ["Billy McConnell"];

-- However, CONTAINS does not use the index
SELECT * FROM account WHERE name CONTAINS "Billy McConnell" EXPLAIN FULL;
-- CONTAINSANY + putting the value inside an array will use the index
SELECT * FROM account WHERE name CONTAINSANY ["Billy McConnell"] EXPLAIN FULL;
```

## Types of operators, order of operations and binding power

To determine which operator is executed first, a concept called "binding power" is used. Operators with greater binding power will operate directly on their neighbours before those with lower binding power. The following is a list of all operator types from greatest to lowest binding power.

<Table>
	<thead>
		<tr>
			<th scope="col" class="w-40">Operator name</th>
			<th scope="col">Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td scope="row" data-label="Type">
				`Unary`
			</td>
			<td scope="row" data-label="Description">
				The `Unary` operators are `!`, `+`, and `-`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`Nullish`
			</td>
			<td scope="row" data-label="Description">
				The `Nullish` operators are `?:` and `??`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`Range`
			</td>
			<td scope="row" data-label="Description">
				The `Range` operator is `..`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`Cast`
			</td>
			<td scope="row" data-label="Description">
				The `Cast` operator is `<type_name>`, with `type_name` a stand in for the type to cast into. For example, `<string>` or `<number>`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`Power`
			</td>
			<td scope="row" data-label="Description">
				The only `Power` operator is `**`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`MulDiv`
			</td>
			<td scope="row" data-label="Description">
				The `MulDiv` (multiplication and division) operators are `*`, `/`, `÷`, and `%`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`AddSub`
			</td>
			<td scope="row" data-label="Description">
				The `AddSub` (addition and subtraction) operators are `+` and `-`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`Relation`
			</td>
			<td scope="row" data-label="Description">
				The `Relation` operators are `<=`, `>=`, `∋`, `CONTAINS`, `∌`, `CONTAINSNOT`, `∈`, `INSIDE`, `∉`, `NOTINSIDE`, `⊇`, `CONTAINSALL`, `⊃`, `CONTAINSANY`, `⊅`, `CONTAINSNONE`, `⊆`, `ALLINSIDE`, `⊂`, `ANYINSIDE`, `⊄`, `NONEINSIDE`, `OUTSIDE`, `INTERSECTS`, `NOT`, and `IN`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`Equality`
			</td>
			<td scope="row" data-label="Description">
				The `Equality` operators are `=`, `IS`, `==`, `!=`, `*=`, `?=`, and `@`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`And`
			</td>
			<td scope="row" data-label="Description">
				The `And` operators are `&&` and `AND`.
			</td>
		</tr>
		<tr>
			<td scope="row" data-label="Type">
				`Or`
			</td>
			<td scope="row" data-label="Description">
				The `Or` operators are `||` and `OR`.
			</td>
		</tr>
	</tbody>
</Table>

## Examples of binding power

The following samples show examples of basic operations of varying binding power. The original example is followed by the same example with the parts with higher binding power in parentheses, then the final expression after the first bound portion is calculated, and finally the output.

```surql title="MulDiv first, then AddSub"
/**[test]

[[test.results]]
value = "13"

[[test.results]]
value = "13"

[[test.results]]
value = "13"

[[test.results]]
value = "13"

*/
 
1 + 3 * 4;
1 + (3 * 4);
-- Final expression
1 + 12;
-- Output
13
```

```surql title="Power first, then MulDiv"
/**[test]

[[test.results]]
value = "24"

[[test.results]]
value = "24"

[[test.results]]
value = "24"

[[test.results]]
value = "24"

*/
 
2**3 * 3;
(2**3) * 3;
-- Final expression
8*3;
-- Output
24
```

```surql title="Unary first, then cast"
/**[test]

[[test.results]]
value = "'-4'"

[[test.results]]
value = "'-4'"

[[test.results]]
value = "'-4'"

*/

<string>-4;
<string>(-4);
-- Output
"-4"
```

```surql title="Cast first, then Power"
/**[test]

[[test.results]]
value = "387420489"

[[test.results]]
value = "387420489"

[[test.results]]
value = "387420489"

[[test.results]]
value = "387420489"

*/
 
<number>"9"**9;
(<number>"9")**9;
-- Final expression
9**9;
-- Output
387420489
```

```surql title="AddSub first, then Relation"
/**[test]

[[test.results]]
value = "true"

[[test.results]]
value = "true"

[[test.results]]
value = "true"

[[test.results]]
value = "true"
*/
 
"c" + "at" IN "cats";
("c" + "at") IN "cats";
-- Final expression
"cat" IN "cats";
-- Output
true
```

```surql title="And first, then Or"
/**[test]

[[test.results]]
value = "true"

[[test.results]]
value = "true"

[[test.results]]
value = "true"

[[test.results]]
value = "true"

*/
 
true AND false OR true;
(true AND false) OR true;
-- Final expression
false OR true;
-- Output
true
```

```surql title="Unary, then Cast, then Power, then AddSub"
/**[test]

[[test.results]]
value = "20dec"

[[test.results]]
value = "20dec"

[[test.results]]
value = "20dec"

*/
 
<decimal>-4**2+4;
((<decimal>(-4))**2)+4;
-- Output
20dec
```

## parameters.mdx

---
sidebar_position: 6
sidebar_label: Parameters
title: Parameters | SurrealQL
description: Parameters can be used like variables to store a value which can then be used in a subsequent query.
---

import Since from '@components/shared/Since.astro'
import Label from "@components/shared/Label.astro";
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

# Parameters

Parameters can be used like variables to store a value which can then be used in subsequent queries. To define a parameter in SurrealQL, use the [`LET`](../surrealql/statements/let) statement. The name of the parameter should begin with a `$` character.

## Defining parameters within SurrealQL

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: person:hw8ha95g4gbegaox34gw, name: 'Tobie Morgan Hitchcock' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: person:q4htgstrmm5j4mvisnhp, name: 'Jaime Morgan Hitchcock' }]"
skip-record-id-key = true

*/

-- Define the parameter
LET $suffix = "Morgan Hitchcock";
-- Use the parameter
CREATE person SET name = "Tobie " + $suffix;
-- (Another way to do the same)
CREATE person SET name = string::join(" ", "Jaime", $suffix);
```

```surql title="Response"
[
    {
        "id": "person:3vs17lb9eso9m7gd8mml",
        "name": "Tobie Morgan Hitchcock"
    }
]

[
    {
        "id": "person:xh4zbns5mgmywe6bo1pi",
        "name": "Jaime Morgan Hitchcock"
    }
]
```

A parameter can store any value, including the result of a query.

```surql
-- Assuming the CREATE statements from the previous example
LET $founders = (SELECT * FROM person);
RETURN $founders.name;
```

```surql title="Response"
[
    "Tobie Morgan Hitchcock",
    "Jaime Morgan Hitchcock"
]
```

Parameters persist across the current connection, and thus can be reused between different namespaces and databases. In the example below, a created `person` record assigned to a parameter is reused in a query in a completely different namespace and database.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
error = "'Thrown error: Database record `person:billy` already exists'"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "'Thrown error: Database record `person:billy` already exists'"

*/

LET $billy = CREATE ONLY person:billy SET name = "Billy";
-- Fails as `person:billy` already exists
CREATE person CONTENT $billy;

USE NAMESPACE other_namespace;
USE DATABASE other_database;
-- Succeeds as `person:billy` does not yet exist in this namespace and database
CREATE person CONTENT $billy;
```

Parameters can be defined using SurrealQL as shown above, or can be passed in using the client libraries as request variables.

## Redefining and shadowing parameters

Parameters in SurrealQL are immutable. The same parameter can be redefined using a `LET` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "'Sypha'"

*/

LET $my_name = "Alucard";
LET $my_name = "Sypha";
RETURN $my_name;
```

```surql title="Output"
'Sypha'
```

Before SurrealDB 3.0, the `=` on its own was used as syntactic sugar for a `LET` statement. This has since been deprecated in order to make it clearer that parameters can be redeclared, but not modified.

<Tabs>
  <TabItem label="Before 3.X" default>
```surql
LET $my_name = "Alucard";
$my_name = "Sypha";
RETURN $my_name;
```

```surql title="Output"
'Sypha'
```
</TabItem>
  <TabItem label="Since 3.X">
```surql
LET $my_name = "Alucard";
$my_name = "Sypha";
RETURN $my_name;
```

```surql title="Output"
'There was a problem with the database: Parse error: Variable declaration without `let` is deprecated
 --> [4:1]
  |
4 | $my_name = "Sypha";
  | ^^^^^^^^^^^^^^^^^^^ replace with `let $my_name = ..`
'
```
  </TabItem>
</Tabs>

If the parameter is redefined inside another scope, the original value will be shadowed. Shadowing refers to when a value is temporarily obstructed by a new value of the same name until the new scope has completed.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[1, 2, 3, 4]"

[[test.results]]
value = "[[1, 2], [3, 4]]"

*/

LET $nums = [
    [1,2],
    [3,4]
];

{
    LET $nums = $nums.flatten();
    -- Flattened into a single array,
    -- so $nums is shadowed as [1,2,3,4]
    RETURN $nums;
};

-- Returns original unflattened $nums:
-- [[1,2], [3,4]]
RETURN $nums;
```

Even a parameter defined using a [`DEFINE PARAM`](/docs/surrealql/statements/define/param) statement can be shadowed.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

DEFINE PARAM $USERNAME VALUE "user@user.com";

LET $USERNAME = "some other email";
```

However, the parameter `$USERNAME` in this case is still defined as its original value, as can be seen via an [`INFO FOR DB`](/docs/surrealql/statements/info) statement.

```surql
{
	accesses: {},
	analyzers: {},
	apis: {},
	configs: {},
	functions: {},
	models: {},
	params: {
		USERNAME: "DEFINE PARAM $USERNAME VALUE 'user@user.com' PERMISSIONS FULL"
	},
	tables: {},
	users: {}
}
```

As the shadowed `$USERNAME` parameter will persist over the length of the connection, the parameter `$USERNAME` will once again show up as its original defined value if the connection is discontinued and restarted.

## Defining parameters within client libraries

SurrealDB's client libraries allow parameters to be passed in as JSON values, which are then converted to SurrealDB data types when the query is run. The following example show a variable being used within a SurrealQL query from the JavaScript library.

```javascript
let people = await surreal.query("SELECT * FROM article WHERE status INSIDE $status", {
	status: ["live", "draft"],
});
```

## Reserved variable names

SurrealDB automatically predefines certain variables depending on the type of operation being performed. For example, `$this` and `$parent` are automatically predefined for subqueries so that the fields of one can be compared to another if necessary. In addition, the predefined variables `$access`, `$auth`, `$token`, and `$session` are protected variables used to give access to parts of the current database configuration and can never be overwritten.

```surql
/**[test]

[[test.results]]
error = ""Thrown error: 'access' is a protected variable and cannot be set""

[[test.results]]
error = ""Thrown error: 'auth' is a protected variable and cannot be set""

[[test.results]]
error = ""Thrown error: 'token' is a protected variable and cannot be set""

[[test.results]]
error = ""Thrown error: 'session' is a protected variable and cannot be set""

*/

LET $access = true;
LET $auth = 10;
LET $token = "Mytoken";
LET $session = rand::int(0, 100);
```

```surql title="Output"
-------- Query 1 --------

"'access' is a protected variable and cannot be set"

-------- Query 2 --------

"'auth' is a protected variable and cannot be set"

-------- Query 3 --------

"'token' is a protected variable and cannot be set"

-------- Query 4 --------

"'session' is a protected variable and cannot be set"
```

Other predefined variables listed below are not specifically protected, but should not be used in order to avoid unexpected behaviour.

### $access

Represents the name of the access method used to authenticate the current session.

```surql
IF $access = "admin" { SELECT * FROM account }
ELSE IF $access = "user" { SELECT * FROM $auth.account }
ELSE {}
```

### $action, $file, $target

These three parameters are used in the context of the permissions of a [`DEFINE BUCKET`](/docs/surrealql/statements/define/bucket) statement.

* `$action` represents the type of operation: one of "Put", "Get", "Head", "Delete", "Copy", "Rename", "Exists", and "List".
* `$file` represents the path to the file being accessed.
* `$target` represents the target file ref in copy/rename operations.

### $auth

Represents the currently authenticated record user.

```surql
DEFINE TABLE user SCHEMAFULL
    PERMISSIONS
        FOR select, update, delete WHERE id = $auth.id;
```

### $before, $after

Represent the values before and after a mutation on a field.

```surql
/**[test]

[[test.results]]
value = "[{ id: cat:f148lyh3zygwenur9nck, name: 'Mr. Meow', nicknames: ['Mr. Cuddlebun'] }]"
skip-record-id-key = true

[[test.results]]
value = "[{ after: { id: cat:f148lyh3zygwenur9nck, name: 'Mr. Meow', nicknames: ['Mr. Cuddlebun', 'Snuggles'] }, before: { id: cat:f148lyh3zygwenur9nck, name: 'Mr. Meow', nicknames: ['Mr. Cuddlebun'] } }]"
skip-record-id-key = true

*/

CREATE cat SET name = "Mr. Meow", nicknames = ["Mr. Cuddlebun"];
UPDATE cat SET nicknames += "Snuggles" WHERE name = "Mr. Meow" RETURN $before, $after;
```

```surql title="Response"
[
    {
        "after": {
            "id": "cat:6p71csv2zqianixf0dkz",
            "name": "Mr. Meow",
            "nicknames": [
                "Mr. Cuddlebun",
                "Snuggles"
            ]
        },
        "before": {
            "id": "cat:6p71csv2zqianixf0dkz",
            "name": "Mr. Meow",
            "nicknames": [
                "Mr. Cuddlebun"
            ]
        }
    }
]
```

### $event

Represents the type of table event triggered on an event. This parameter will be one of either `"CREATE"`, `"UPDATE"`, or `"DELETE"`.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE EVENT user_created ON TABLE user WHEN $event = "CREATE" THEN (
    CREATE log SET table = "user", event = $event, created_at = time::now()
);
```

### $input

Represents the initially inputted value in a field definition, as the value clause could have modified the $value variable.

```surql
/**[test]

[[test.results]]
value = "[{ historical_data: [], id: city:london, population: 8900000, year: 2019 }]"

[[test.results]]
value = "[{ historical_data: [{ population: 8900000, year: 2019 }], id: city:london, population: 9600000, year: 2023 }]"

*/

CREATE city:london SET
    population = 8900000,
    year = 2019,
    historical_data = [];

INSERT INTO city [
    { id: "london", population: 9600000, year: 2023 }
]
ON DUPLICATE KEY UPDATE
-- Stick old data into historical_data
historical_data += {
    year: year,
    population: population
},
-- Then update current record with the new input using $input
population = $input.population,
year = $input.year;
```

```surql output="Response"
[
    {
        "historical_data": [
            {
                "population": 8900000,
                "year": 2019
            }
        ],
        "id": "city:london",
        "population": 9600000,
        "year": 2023
    }
]
```

### $parent, $this

`$this` represents the current record in a subquery, and `$parent` its parent.

```surql
/**[test]

[[test.results]]
value = "[{ id: user:uz4nrrdn0xybrceh4bm9, member_of: 'group1', name: 'User1' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: user:07o9ep802fiorrr936u5, member_of: 'group1', name: 'User2' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: user:ij9yjba7n2428yz8nui0, member_of: 'group1', name: 'User3' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ group_members: ['User2', 'User3', 'User1'], name: 'User1' }]"

*/

CREATE user SET name = "User1", member_of = "group1";
CREATE user SET name = "User2", member_of = "group1";
CREATE user SET name = "User3", member_of = "group1";
SELECT name, 
    (SELECT VALUE name FROM user WHERE member_of = $parent.member_of)
    AS group_members
    FROM user
    WHERE name = "User1";
```

```surql title="Response"
[
    {
        "group_members": [
            "User1",
            "User3",
            "User2"
        ],
        "name": "User1"
    }
]
```

```surql
/**[test]

[[test.results]]
value = "[{ id: person:trgs6t98z3j5zlxsx1x5, name: 'John Doe' }, { id: person:xiysvpvqvglklrwy9tt5, name: 'John Doe' }, { id: person:qdpkqcb8gu5tt8jocuc8, name: 'Jane Doe' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: person:qdpkqcb8gu5tt8jocuc8, name: 'Jane Doe', people_with_same_name: [person:qdpkqcb8gu5tt8jocuc8] }, { id: person:trgs6t98z3j5zlxsx1x5, name: 'John Doe', people_with_same_name: [person:trgs6t98z3j5zlxsx1x5, person:xiysvpvqvglklrwy9tt5] }, { id: person:xiysvpvqvglklrwy9tt5, name: 'John Doe', people_with_same_name: [person:trgs6t98z3j5zlxsx1x5, person:xiysvpvqvglklrwy9tt5] }]"
skip-record-id-key = true

*/

INSERT INTO person (name) VALUES ("John Doe"), ("John Doe"), ("Jane Doe");
SELECT 
    *,
    (SELECT VALUE id FROM person WHERE $this.name = $parent.name) AS 
    people_with_same_name
    FROM person;
```

```surql title="Response"
[
    {
        "id": "person:hwffcckiv61ylwiw43yf",
        "name": "John Doe",
        "people_with_same_name": [
            "person:hwffcckiv61ylwiw43yf",
            "person:tmscoy7bjj20xki0fld5"
        ]
    },
    {
        "id": "person:tmscoy7bjj20xki0fld5",
        "name": "John Doe",
        "people_with_same_name": [
            "person:hwffcckiv61ylwiw43yf",
            "person:tmscoy7bjj20xki0fld5"
        ]
    },
    {
        "id": "person:y7mdf3912rf5gynvxc7q",
        "name": "Jane Doe",
        "people_with_same_name": [
            "person:y7mdf3912rf5gynvxc7q"
        ]
    }
]
```

### $reference

This parameter represents the reference in question inside an [`ON DELETE`](/docs/surrealql/datamodel/references#specifying-deletion-behaviour) clause for record references.

### $request

This parameter represents the value of a request to a custom API defined using the [`DEFINE API`](/docs/surrealql/statements/define/api) statement.

```surql
DEFINE API OVERWRITE "/test"
    FOR get, post 
        MIDDLEWARE
            api::timeout(1s)
        THEN {
            RETURN {
                status: 404,
                body: $request.body,
                headers: {
                    'bla': '123'
                }
            };
        };
```

The `$request` parameter may contain values at the following fields: `body`, `headers`, `params`, `method`, `query`, and `context`.

### $session

Represents values from the session functions as an object.

You can learn more about those values from the [security parameters](/docs/surrealdb/security/authentication#session) section.

```surql
CREATE user SET 
    name = "Some User",
    on_database = $session.db;
```

```surql title="Response"
[
    {
        "id": "user:wa3ajflozlqoyurc4i4v",
        "name": "Some User",
        "on_database": "database"
    }
]
```

### $token

Represents values held inside the JWT token used for the current session.

You can learn more about those values from the [security parameters](/docs/surrealdb/security/authentication#token) section.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE TABLE user SCHEMAFULL
  PERMISSIONS FOR select, update, delete, create
  WHERE $access = "users"
  AND email = $token.email;
```

### $value

Represents the value after a mutation on a field (identical to $after in the case of an event).

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE EVENT email ON TABLE user WHEN $before.email != $after.email THEN (
    CREATE event SET 
        user = $value.id,
        time = time::now(),
        value = $after.email,
        action = 'email_changed'
);
```

## Improvements to parameters and expressions in statements

<Since v="v3.0.0" />

Parameters and expressions have traditionally only been available in a limited fashion in SurrealQL statements. As of the alpha versions of SurrealDB 3.0, work is undergoing to allow parameters and expressions to be used in many places that were not possible before.

Some examples of this are:

### DEFINE statements

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ language_en: 'DEFINE TABLE language_en TYPE NORMAL SCHEMAFULL PERMISSIONS NONE', language_ie: 'DEFINE TABLE language_ie TYPE NORMAL SCHEMAFULL PERMISSIONS NONE', language_ja: 'DEFINE TABLE language_ja TYPE NORMAL SCHEMAFULL PERMISSIONS NONE', language_uk: 'DEFINE TABLE language_uk TYPE NORMAL SCHEMAFULL PERMISSIONS NONE', user: "DEFINE TABLE user TYPE NORMAL SCHEMAFULL PERMISSIONS FOR select, create, update, delete WHERE $access = 'users' AND email = $token.email" }"

*/

FOR $language IN ["en", "ja", "uk", "ie"] {
    DEFINE TABLE "language_" + $language SCHEMAFULL;
};

(INFO FOR DB).tables;
```

```surql title="Output"
{
	language_en: 'DEFINE TABLE language_en TYPE NORMAL SCHEMAFULL PERMISSIONS NONE',
	language_ie: 'DEFINE TABLE language_ie TYPE NORMAL SCHEMAFULL PERMISSIONS NONE',
	language_ja: 'DEFINE TABLE language_ja TYPE NORMAL SCHEMAFULL PERMISSIONS NONE',
	language_uk: 'DEFINE TABLE language_uk TYPE NORMAL SCHEMAFULL PERMISSIONS NONE'
}
```

### REMOVE statements

Parameterization in `REMOVE` statements is particularly useful in the context of testing.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

FOR $table IN ["test_user", "test_client"] {
    DEFINE TABLE $table;
    // Do some tests
    REMOVE TABLE $table;
};
```

The following example shows an example of a test that might be performed using a `REMOVE FIELD` statement. Here, the `INFO FOR TABLE` statement is used to dynamically capture the defined fields of a table, followed by the [object::keys()](/docs/surrealql/functions/database/object#objectkeys) function to retrieve each field as a string. The fields can then be removed one by one inside a `REMOVE FIELD` statement, with the time elapsed logged in a separate table.

```surql
DEFINE FIELD string_test ON test TYPE string;
DEFINE FIELD int_test ON test TYPE int;
DEFINE FIELD datetime_test ON test TYPE datetime;

CREATE |test:10000| SET 
    string_test = rand::string(10),
    int_test = rand::int(),
    datetime_test = rand::time()
RETURN NONE;

FOR $field IN (INFO FOR TABLE test).fields.keys() {
    LET $now = time::now();
    REMOVE FIELD $field ON test;
    LET $elapsed = time::now() - $now;
    CREATE log SET results = { field_name: $field, removed_in: $elapsed }
};
```

### The TIMEOUT clause in queries

```surql
DEFINE FUNCTION fn::get_timeout() -> duration {
    // Do some HTTP call to get status
    // Simulate the output with rand::enum() function
    rand::enum(100ms, 1s, 5s)
};

SELECT * FROM person TIMEOUT fn::get_timeout();
```

### The OMIT clause in queries

```surql
/**[test]

[[test.results]]
value = "[{ age: 19, id: person:v5tk0ctrk8vjgpsjsr2n, name: 'Galen', surname: 'Pathwarden' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ age: 19, surname: 'Pathwarden' }]"

*/

CREATE person SET name = "Galen", surname = "Pathwarden", age = 19;

SELECT * OMIT type::fields(["name", "id"]) FROM person;
```

```surql title="Output"
[
	{
		age: 19,
		surname: 'Pathwarden'
	}
]
```
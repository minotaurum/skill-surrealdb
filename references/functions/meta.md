# Meta functions

> [!NOTE]
> As of version 2.0, these functions are now part of SurrealDB's [record](/docs/surrealql/functions/database/record) functions.

These functions can be used to retrieve specific metadata from a SurrealDB Record ID.

## `meta::id`

The `meta::id` function extracts and returns the identifier from a SurrealDB Record ID.

```surql
meta::id(record) -> value
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN meta::id(person:tobie);

"tobie"
```

## `meta::tb`

The `meta::tb` function extracts and returns the table name from a SurrealDB Record ID.

```surql
meta::tb(record) -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN meta::tb(person:tobie);

"person"
```

This function can also be called using the path `meta::table`.

```surql
RETURN meta::table(person:tobie);

"person"
```

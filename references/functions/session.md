# Session functions

These functions return information about the current SurrealDB session.

## `session::ac`

> [!NOTE]
> This function was known as `session::sc` in versions of SurrrealDB before 2.0. The behaviour has not changed.

The `session::ac` function returns the current user's access method.

```surql
session::ac() -> string
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN session::ac();

"user"
```

## `session::db`

The `session::db` function returns the currently selected database.

```surql
session::db() -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN session::db();

"my_db"
```

## `session::id`

The `session::id` function returns the current user's session ID.

```surql
session::id() -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN session::id();

"I895rKuixHwCNIduyBIYH2M0Pga7oUmWnng5exEE4a7EB942GVElGrnRhE5scF5d"
```

## `session::ip`

The `session::ip` function returns the current user's session IP address.

```surql
session::ip() -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN session::ip();

"2001:db8:3333:4444:CCCC:DDDD:EEEE:FFFF"
```

## `session::ns`

The `session::ns` function returns the currently selected namespace.

```surql
session::ns() -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN session::ns();

"my_ns"
```

## `session::origin`

The `session::origin` function returns the current user's HTTP origin.

```surql
session::origin() -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
RETURN session::origin();

"http://localhost:3000"
```

## `session::rd`

The `session::rd` function returns the current user's record authentication.

```surql
session::rd() -> string
```

## `session::token`

The `session::token` function returns the current authentication token.

```surql
session::token() -> string
```

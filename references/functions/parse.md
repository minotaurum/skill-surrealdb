# Parse functions

These functions can be used when parsing email addresses and URL web addresses.

## `parse::email::host`

The `parse::email::host` function parses and returns an email host from a valid email address.

```surql
parse::email::host(string) -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'surrealdb.com'"

*/

RETURN parse::email::host("info@surrealdb.com");

-- 'surrealdb.com'
```

## `parse::email::user`

The `parse::email::user` function parses and returns an email username from a valid email address.

```surql
parse::email::user(string) -> string
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'info'"

*/

RETURN parse::email::user("info@surrealdb.com");

-- "info"
```

## `parse::url::domain`

The `parse::url::domain` function parses and returns domain from a valid URL.
This function is similar to `parse::url::host` only that it will return `null` if the URL is an IP address.

```surql
parse::url::domain(string) -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'surrealdb.com"

[[test.results]]
value = 'NONE'

*/

RETURN parse::url::domain("https://surrealdb.com:80/features?some=option#fragment");
RETURN parse::url::domain("http://127.0.0.1/index.html");
```

```surql
"surrealdb.com"

NONE
```

## `parse::url::fragment`

The `parse::url::fragment` function parses and returns the fragment from a valid URL.

```surql
parse::url::fragment(string) -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'fragment'"

*/

RETURN parse::url::fragment("https://surrealdb.com:80/features?some=option#fragment");

-- 'fragment'
```

## `parse::url::host`

The `parse::url::host` function parses and returns the hostname from a valid URL.

```surql
parse::url::host(string) -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'surrealdb.com'"

[[test.results]]
value = "'127.0.0.1'"

*/

RETURN parse::url::host("https://surrealdb.com:80/features?some=option#fragment");
RETURN parse::url::host("http://127.0.0.1/index.html");
```

```surql
'surrealdb.com'

'127.0.0.1'
```

## `parse::url::path`

The `parse::url::path`  function parses and returns the path from a valid URL.

```surql
parse::url::path(string) -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'/features'"

*/

RETURN parse::url::path("https://surrealdb.com:80/features?some=option#fragment");

-- '/features'
```

## `parse::url::port`

The `parse::url::port` function parses and returns the port from a valid URL.

```surql
parse::url::port(string) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "80"

*/

RETURN parse::url::port("https://surrealdb.com:80/features?some=option#fragment");

-- 80
```

## `parse::url::scheme`

The `parse::url::scheme` function parses and returns the scheme from a valid URL, in lowercase, as an ASCII string without the ':' delimiter.

```surql
parse::url::scheme(string) -> string
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'https'"

*/

RETURN parse::url::scheme("https://surrealdb.com:80/features?some=option#fragment");

-- 'https'
```

## `parse::url::query`

The `parse::url::query` function parses and returns the query from a valid URL.

```surql
parse::url::query(string) -> string
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'some=option'"

*/

RETURN parse::url::query("https://surrealdb.com:80/features?some=option#fragment");

-- 'some=option'
```

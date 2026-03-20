# Formatters

The [string::is_datetime](/docs/surrealql/functions/database/string#stringisdatetime) and [time::format](/docs/surrealql/functions/database/time#timeformat) functions in SurrealQL accept certain text formats for date/time formatting. The possible formats are listed below.

### Date formatters

### Time formatters

### Timezones formatters

### Date & time formatters

### Other formatters

## Examples

Seeing if an input with a date and time conforms to an expected format:

```surql
/**[test]

[[test.results]]
value = "true"

*/

string::is_datetime("5sep2024pm012345.6789", "%d%b%Y%p%I%M%S%.f");
```

```surql
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

```surql
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

```surql
"2021-11-01"
```

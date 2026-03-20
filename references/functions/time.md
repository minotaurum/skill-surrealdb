# Time Functions

> [!NOTE]
> Since version 3.0.0-beta, the `::from::` functions (e.g. `time::from::millis()`) now use underscores (e.g. `time::from_millis()`) to better match the intent of the function and method syntax.

These functions can be used when working with and manipulating [datetime](/docs/surrealql/datamodel/datetimes) values.

Many time functions take an `option<datetime>` in order to return certain values from a datetime such as its hours, minutes, day of the year, and so in. If no argument is present, the current datetime will be extracted and used. As such, all of the following function calls are valid and will not return an error.

```surql
time::hour(d'2024-09-04T00:32:44.107Z');
time::hour();

time::minute(d'2024-09-04T00:32:44.107Z');
time::minute();

time::yday(d'2024-09-04T00:32:44.107Z');
time::yday();
```

## `time::ceil`

The `time::ceil` function rounds a datetime up to the next largest duration.

```surql
time::ceil(datetime, $ceiling: duration) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[d'2024-08-30T03:00:00Z', d'2024-09-05T00:00:00Z']"

*/
LET $now = d'2024-08-30T02:22:50.231631Z';

RETURN [
  time::ceil($now, 1h),
  time::ceil($now, 1w)
];
```

```surql
[
	d'2024-08-30T03:00:00Z',
	d'2024-09-05T00:00:00Z'
]
```

## `time::day`

The `time::day` function extracts the day as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::day(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1"

*/

RETURN time::day(d"2021-11-01T08:30:17+00:00");

-- 1
```

## `time::epoch`

The `time::epoch` constant returns the `datetime` for the UNIX epoch (January 1, 1970).

```surql
// Return the const
RETURN time::epoch;
-- d'1970-01-01T00:00:00Z'

// Define field using the const
DEFINE FIELD since_epoch ON event COMPUTED time::now().floor(1d) - time::epoch;
CREATE ONLY event:one SET information = "Something happened";
-- { id: event:one, information: 'Something happened', since_epoch: 55y42w6d }
```

## `time::floor`

The `time::floor` function rounds a datetime down by a specific duration.

```surql
time::floor(datetime, $floor: duration) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'2021-10-28T00:00:00Z'"

*/

RETURN time::floor(d"2021-11-01T08:30:17+00:00", 1w);

-- d'2021-10-28T00:00:00Z'
```

## `time::format`

The `time::format` function outputs a datetime as a string according to a specific format.

```surql
time::format(datetime, $format: string) -> string
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "'2021-11-01'"

*/

RETURN time::format(d"2021-11-01T08:30:17+00:00", "%Y-%m-%d");
```

```surql output="Response"
'2021-11-01'
```

[View all format options](/docs/surrealql/datamodel/formatters)

## `time::group`

The `time::group` function reduces and rounds a datetime down to a particular time interval.

```surql
time::group(datetime, $group_by: 'year'|'month'|'day'|'hour'|'minute'|'second') -> datetime
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'2021-01-01T00:00:00Z'"

*/

RETURN time::group(d"2021-11-01T08:30:17+00:00", "year");

d'2021-01-01T00:00:00Z'
```

## `time::hour`

The `time::hour` function extracts the hour as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::hour(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "8"

*/

RETURN time::hour(d"2021-11-01T08:30:17+00:00");

-- 8
```

## `time::max`

The `time::max` function returns the greatest datetime from an array of datetimes.

```surql
time::max(array<datetime>) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'1988-06-22T08:30:45Z'"

*/

RETURN time::max([ d"1987-06-22T08:30:45Z", d"1988-06-22T08:30:45Z" ])

-- d'1988-06-22T08:30:45Z'
```

See also:

* [`array::max`](/docs/surrealql/functions/database/array#arraymax), which extracts the greatest value from an array of values
* [`math::max`](/docs/surrealql/functions/database/math#mathmax), which extracts the greatest number from an array of numbers

## `time::maximum`

The `time::maximum` constant returns the greatest possible datetime that can be used.

```surql
time::maximum -> datetime
```

Some examples of the constant in use:

```surql
/**[test]

[[test.results]]
value = "d'+262142-12-31T23:59:59.999999999Z'"

[[test.results]]
error = ""Failed to compute: \"1ns + d'+262142-12-31T23:59:59.999999999Z'\", as the operation results in an arithmetic overflow.""

[[test.results]]
value = "true"

*/

time::maximum;

time::maximum + 1ns;

time::now() IN time::minimum..time::maximum;
```

```surql
-------- Query 1 --------

d'+262142-12-31T23:59:59.999Z'

-------- Query 2 --------

"Failed to compute: \"1ns + d'+262142-12-31T23:59:59.999999999Z'\", as the operation results in an arithmetic overflow."

-------- Query 3 --------

true
```

## `time::micros`

The `time::micros` function extracts the microseconds as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::micros(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "551349045000000"

*/

RETURN time::micros(d"1987-06-22T08:30:45Z");

-- 551349045000000
```

## `time::millis`

The `time::millis` function extracts the milliseconds as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::millis(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "551349045000"

*/

RETURN time::millis(d"1987-06-22T08:30:45Z");

-- 551349045000
```

## `time::min`

The `time::min` function returns the least datetime from an array of datetimes.

```surql
time::min(array<datetime>) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'1987-06-22T08:30:45Z'"

*/

RETURN time::min([ d"1987-06-22T08:30:45Z", d"1988-06-22T08:30:45Z" ]);

-- d'1987-06-22T08:30:45Z'
```

See also:

* [`array::min`](/docs/surrealql/functions/database/array#arraymin), which extracts the least value from an array of values
* [`math::min`](/docs/surrealql/functions/database/math#mathmin), which extracts the least number from an array of numbers

## `time::minimum`

The `time::minimum` constant returns the least possible datetime that can be used.

```surql
time::minimum -> datetime
```

Some examples of the constant in use:

```surql
/**[test]

[[test.results]]
value = "d'-262143-01-01T00:00:00Z'"

[[test.results]]
value = "true"

*/

time::minimum;

time::now() IN time::minimum..time::maximum;
```

```surql
-------- Query 1 --------

d'-262143-01-01T00:00:00Z'

-------- Query 2 --------

true
```

## `time::minute`

The `time::minute` function extracts the minutes as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::minute(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "30"

*/

RETURN time::minute(d"2021-11-01T08:30:17+00:00");

-- 30
```

## `time::month`

The `time::month` function extracts the month as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::month(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "11"

*/

RETURN time::month(d"2021-11-01T08:30:17+00:00");

-- 11
```

## `time::nano`

The `time::nano`function returns a datetime as an integer representing the number of nanoseconds since the UNIX epoch until a datetime, or the current date if no datetime argument is present.

```surql
time::nano(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1635755417000000000"

*/

RETURN time::nano(d"2021-11-01T08:30:17+00:00");

-- 1635755417000000000
```

## `time::now`

The `time::now` function returns the current datetime as an ISO8601 timestamp.

```surql
time::now() -> datetime
```

## `time::round`

The `time::round` function rounds a datetime up by a specific duration.

```surql
time::round(datetime, $round_to: duration) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'2021-11-04T00:00:00Z'"

*/

RETURN time::round(d"2021-11-01T08:30:17+00:00", 1w);

-- d'2021-11-04T00:00:00Z'
```

## `time::second`

The `time::second` function extracts the second as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::second(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "17"

*/

RETURN time::second(d"2021-11-01T08:30:17+00:00");

-- 17
```

## `time::timezone`

The `time::timezone` function returns the current local timezone offset in hours.

```surql
time::timezone() -> string
```

## `time::unix`

The `time::unix` function returns a datetime as an integer representing the number of seconds since the UNIX epoch until a certain datetime, or from the current date if no datetime argument is present.

```surql
time::unix(option<datetime>) -> number
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1635755417"

*/

RETURN time::unix(d"2021-11-01T08:30:17+00:00");

-- 1635755417
```

## `time::wday`

The `time::wday` function extracts the week day as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::wday(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1"

*/

RETURN time::wday(d"2021-11-01T08:30:17+00:00");

-- 1
```

## `time::week`

The `time::week` function extracts the week as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::week(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "44"

*/

RETURN time::week(d"2021-11-01T08:30:17+00:00");

-- 44
```

## `time::yday`

The `time::yday` function extracts the day of the year as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::yday(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "305"

*/

RETURN time::yday(d"2021-11-01T08:30:17+00:00");

-- 305
```

## `time::year`

The `time::year` function extracts the year as a number from a datetime, or from the current date if no datetime argument is present.

```surql
time::year(option<datetime>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "2021"

*/

RETURN time::year(d"2021-11-01T08:30:17+00:00");

-- 2021
```

## `time::is_leap_year()`

The `time::is_leap_year()` function Checks if given datetime is a leap year.

```surql
time::is_leap_year(datetime) -> bool
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
-- Checks with current datetime if none is passed
RETURN time::is_leap_year();

RETURN time::is_leap_year(d"1987-06-22T08:30:45Z");
-- false

RETURN time::is_leap_year(d"1988-06-22T08:30:45Z");
-- true

-- Using function via method chaining
RETURN d'2024-09-03T02:33:15.349397Z'.is_leap_year();
-- true
```

## `time::from_micros`

The `time::from_micros` function calculates a datetime based on the microseconds since January 1, 1970 0:00:00 UTC.

```surql
time::from_micros(number) -> datetime
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'1970-01-01T00:00:01Z'"

*/

RETURN time::from_micros(1000000);

-- d'1970-01-01T00:00:01Z'
```

## `time::from_millis`

The `time::from_millis` function calculates a datetime based on the milliseconds since January 1, 1970 0:00:00 UTC.

```surql
time::from_millis(number) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'1970-01-01T00:00:01Z'"

*/

RETURN time::from_millis(1000);

-- d'1970-01-01T00:00:01Z'
```

## `time::from_nanos`

The `time::from_nanos` function calculates a datetime based on the nanoseconds since January 1, 1970 0:00:00 UTC.

```surql
time::from_nanos(number) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'1970-01-01T00:00:00.001Z'"

*/

RETURN time::from_nanos(1000000);

-- d'1970-01-01T00:00:00.001Z'
```

## `time::from_secs`

The `time::from_secs` function calculates a datetime based on the seconds since January 1, 1970 0:00:00 UTC.

```surql
time::from_secs(number) -> datetime
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'1970-01-01T00:16:40Z'"

*/

RETURN time::from_secs(1000);

-- d'1970-01-01T00:16:40Z'
```

## `time::from_unix`

The `time::from_unix` function calculates a datetime based on the seconds since January 1, 1970 0:00:00 UTC.

```surql
time::from_unix(number) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'1970-01-01T00:16:40Z'"

*/

RETURN time::from_unix(1000);

-- d'1970-01-01T00:16:40Z'
```

## `time::from_ulid`

The `time::from_ulid` function calculates a datetime based on the ULID.

```surql
time::from_ulid(ulid) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'2025-01-09T10:57:03.593Z'"

*/

RETURN time::from_ulid("01JH5BBTK9FKTGSDXHWP5YP9TQ");

-- d'2025-01-09T10:57:03.593Z'
```

As a ULID is only precise up to the millisecond, a conversion from a ULID to a timestamp will truncate nanosecond precision.

```surql
LET $now = time::now();
[$now, time::from_ulid(rand::ulid($now))];

-- Output:
[
	d'2026-01-29T02:07:06.494218Z',
	d'2026-01-29T02:07:06.494Z'
]
```

## `time::from_uuid`

The `time::from_uuid` function calculates a datetime based on the UUID.

```surql
time::from_uuid(uuid) -> datetime
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "d'2025-01-09T10:57:58.757Z'"

*/

RETURN time::from_uuid(u'01944ab6-c1e5-7760-ab6a-127d37eb1b94');

-- d'2025-01-09T10:57:58.757Z'
```

As a UUID is only precise up to the millisecond, a conversion from a UUID to a timestamp will truncate nanosecond precision.

```surql
LET $now = time::now();
[$now, time::from_uuid(rand::uuid($now))];

-- Output:
[
	d'2026-01-29T02:12:13.848476Z',
	d'2026-01-29T02:12:13.848Z'
]
```

## `time::set_year`

The `time::set_year` function sets the year value of a datetime.

```surql
time::set_year(datetime, $year: integer) -> datetime
```

Example:

```surql
d'1970-01-01T00:00:00.500000005Z'.set_year(2026);
-- Output
d'2026-01-01T00:00:00.500000000Z'
```

## `time::set_month`

The `time::set_month` function sets the month value of a datetime.

```surql
time::set_month(datetime, $month: integer) -> datetime
```

Example:

```surql
d'1970-01-01T00:00:00.500000005Z'.set_month(9);
-- Output
d'1970-09-01T00:00:00.500000005Z'
```

## `time::set_day`

The `time::set_day` function sets the day value of a datetime.

```surql
time::set_day(datetime, $day: integer) -> datetime
```

Example:

```surql
d'1970-01-01T00:00:00.500000005Z'.set_day(10);
-- Output
d'1970-01-10T00:00:00.500000005Z'
```

## `time::set_hour`

The `time::set_hour` function sets the hour value of a datetime.

```surql
time::set_hour(datetime, $hour: integer) -> datetime
```

Example:

```surql
d'1970-01-01T00:00:00.500000005Z'.set_hour(10);
-- Output
d'1970-01-01T10:00:00.500000005Z'
```

## `time::set_minute`

The `time::set_minute` function sets the minute value of a datetime.

```surql
time::set_minute(datetime, $minute: integer) -> datetime
```

Example:

```surql
d'1970-01-01T10:00:00.500000005Z'.set_minute(55);
-- Output
d'1970-01-01T10:55:00.500000005Z'
```

## `time::set_second`

The `time::set_second` function sets the second value of a datetime.

```surql
time::set_second(datetime, $second: integer) -> datetime
```

Example:

```surql
d'1970-01-01T10:00:00.500000005Z'.set_second(30);
-- Output
d'1970-01-01T10:00:30.500000005Z'
```

## `time::set_nanosecond`

The `time::set_nanosecond` function sets the nanosecond value of a datetime.

```surql
time::set_nanosecond(datetime, $nanosecond: integer) -> datetime
```

Example:

```surql
d'1970-01-01T10:00:00.500000005Z'.set_nanosecond(3535);
-- Output
d'1970-01-01T10:00:00.000003535Z'
```

Since nanoseconds are not needed in a datetime, setting the nanoseconds of a datetime to 0 can be used to make a datetime look cleaner.

```surql
d'1970-01-01T00:00:00.500000000Z'.set_nanosecond(0);
d'1970-01-01T00:00:00Z' -- output
```

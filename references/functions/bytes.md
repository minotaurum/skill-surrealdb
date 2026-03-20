# Bytes functions

These functions can be used when working with bytes in SurrealQL.

## `bytes::len`

The `bytes::len` function returns the length in bytes of a `bytes` value.

```surql
bytes::len(bytes) -> int
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[19, 67, 25]"

*/

RETURN [
    bytes::len(<bytes>"Simple ASCII string"),
    bytes::len(<bytes>"οὐ γὰρ δυνατόν ἐστιν ἔτι καθεύδειν"),
    bytes::len(<bytes>"청춘예찬 靑春禮讚")
];
```

```surql
[ 19, 67, 25 ]
```

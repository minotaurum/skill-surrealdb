# Encoding functions

These functions can be used to encode and decode data into other formats, such as `base64` and [`CBOR`](/docs/surrealdb/integration/cbor) (Concise Binary Object Representation). It is particularly used when that data needs to be stored and transferred over media that are designed to deal with text. This encoding and decoding helps to ensure that the data remains intact without modification during transport.

</br>

## `encoding::base64::encode()`

The `encoding::base64::encode()` function encodes a bytes to base64 with optionally padded output.

```surql
encoding::base64::encode(bytes) -> string
```

```surql
encoding::base64::encode(bytes, $pad_output: option<bool>) -> string
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "''"

*/
RETURN encoding::base64::encode(<bytes>"");

-- ''
```

```surql
/**[test]

[[test.results]]
value = "'MjMyMw'"

*/
RETURN encoding::base64::encode(<bytes>"2323");

-- 'MjMyMw'
```

```surql
/**[test]

[[test.results]]
value = "'aGVsbG8'"

*/
RETURN encoding::base64::encode(<bytes>"hello");

-- 'aGVsbG8'
```

As of version 2.3.0, you can pass `true` as the second argument to enable padded base64 outputs:

```surql
/**[test]

[[test.results]]
value = "''"

*/
RETURN encoding::base64::encode(<bytes>"", true);

-- ""
```

```surql
/**[test]

[[test.results]]
value = "'MjMyMw=='"

*/
RETURN encoding::base64::encode(<bytes>"2323", true);

"MjMyMw=="
```

```surql
/**[test]

[[test.results]]
value = "'aGVsbG8='"

*/
RETURN encoding::base64::encode(<bytes>"hello", true);

"aGVsbG8="
```

## `encoding::base64::decode()`

The `encoding::base64::decode()` function decodes a string into bytes.

```surql
encoding::base64::decode(string) -> bytes
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "b"32333233""

*/
RETURN encoding::base64::decode("MjMyMw");

-- b"32333233"
```

You can also verify that the output of the encoded value matches the original value. 

```surql
/**[test]

[[test.results]]
value = "true"

*/
RETURN encoding::base64::decode("aGVsbG8") = <bytes>"hello";

-- true
```

## `encoding::cbor::decode()`

The `encoding::cbor::decode()` function decodes bytes in valid CBOR format into a SurrealQL value.

```surql
encoding::cbor::decode(string) -> any
```

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ cbor: 'encoding', is: 'pretty neat' }"

*/
LET $some_bytes = encoding::base64::decode("omRjYm9yaGVuY29kaW5nYmlza3ByZXR0eSBuZWF0");
encoding::cbor::decode($some_bytes);
```

```surql
{
	cbor: 'encoding',
	is: 'pretty neat'
}
```

## `encoding::cbor::encode()`

The `encoding::cbor::encode()` function encodes any SurrealQL value into bytes in CBOR format.

```surql
encoding::cbor::encode(any) -> bytes
```

```surql
/**[test]

[[test.results]]
value = "b"A26463626F7268656E636F64696E676269736B707265747479206E656174""

*/
encoding::cbor::encode({
    cbor: "encoding",
    is: "pretty neat"
});
```

```surql
b"A26463626F7268656E636F64696E676269736B707265747479206E656174"
```

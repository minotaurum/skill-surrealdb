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

## Next steps
You've now seen how to use numeric values in SurrealDB. For more advanced functionality, take a look at the operators and math functions, which enable advanced calculations on numeric values and sets of numeric values.

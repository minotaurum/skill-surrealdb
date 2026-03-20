# Math functions

These functions can be used when analysing numeric data and numeric collections.

## `math::abs`

The `math::abs` function returns the absolute value of a number.

```surql
math::abs(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "13.746189f"

*/

RETURN math::abs(-13.746189);

13.746189f
```

## `math::acos`

The `math::acos` function returns the arccosine (inverse cosine) of a number, which must be in the range -1 to 1. The result is expressed in radians.

```surql
math::acos(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1.0471975511965976f"

*/

RETURN math::acos(0.5);

-- 1.0471975511965976f
```

## `math::acot`

The `math::acot` function returns the arccotangent (inverse cotangent) of a number. The result is expressed in radians.

```surql
math::acot(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.7853981633974483f"

*/

RETURN math::acot(1);

-- 0.7853981633974483f
```

## `math::asin`

The `math::asin` function returns the arcsine (inverse sine) of a number, which must be in the range -1 to 1. The result is expressed in radians.

```surql
math::asin(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.5235987755982988f"

*/

RETURN math::asin(0.5);

-- 0.5235987755982988f
```

## `math::atan`

The `math::atan` function returns the arctangent (inverse tangent) of a number. The result is expressed in radians.

```surql
math::atan(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.7853981633974483f"

*/

RETURN math::atan(1);

-- 0.7853981633974483f
```

## `math::bottom`

The `math::bottom` function returns the bottom X set of numbers in an array of numbers.

```surql
math::bottom(array<number>, $quantity: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[2, 1]"

*/

RETURN math::bottom([1, 2, 3], 2);

-- [2, 1]
```

## `math::ceil`

The `math::ceil` function rounds a number up to the next largest whole number.

```surql
math::ceil(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "14f"

*/

RETURN math::ceil(13.146572);
-- 14f
```

## `math::clamp`

The `math::clamp` function constrains a number within the specified range, defined by a minimum and a maximum value. If the number is less than the minimum, it returns the minimum. If it is greater than the maximum, it returns the maximum.

```surql
math::clamp(number, $min: number, $max: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "5"

*/

RETURN math::clamp(1, 5, 10);
-- 5
```

## `math::cos`

The `math::cos` function returns the cosine of a number, which is assumed to be in radians. The result is a value between -1 and 1.

```surql
math::cos(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.5403023058681398f"

*/

RETURN math::cos(1);
-- 0.5403023058681398f
```

## `math::cot`

The `math::cot` function returns the cotangent of a number, which is assumed to be in radians. The cotangent is the reciprocal of the tangent function.

```surql
math::cot(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.6420926159343308f"

*/

RETURN math::cot(1);
-- 0.6420926159343308f
```

## `math::deg2rad`

The `math::deg2rad` function converts an angle from degrees to radians.

```surql
math::deg2rad(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "3.141592653589793f"

*/

RETURN math::deg2rad(180);
-- 3.141592653589793f
```

## `math::e`

The `math::e` constant represents the base of the natural logarithm (Euler’s number).

```surql
math::e -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "2.718281828459045f"

*/

RETURN math::e;
-- 2.718281828459045f
```

## `math::fixed`

The `math::fixed` function returns a number with the specified number of decimal places.

```surql
math::fixed(number, $places: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "13.15f"

*/

RETURN math::fixed(13.146572, 2);

-- 13.15f
```

## `math::floor`

The `math::floor` function rounds a number down to the nearest integer.

```surql
math::floor(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "13f"

*/

RETURN math::floor(13.746189);
-- 13f 
```

## `math::frac_1_pi`

The `math::frac_1_pi` constant represents the fraction 1/π.

```surql
math::frac_1_pi -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.3183098861837907f"

*/

RETURN math::frac_1_pi;

-- 0.3183098861837907f
```

## `math::frac_1_sqrt_2`

The `math::frac_1_sqrt_2` constant represents the fraction 1/sqrt(2).

```surql
math::frac_1_sqrt_2 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.7071067811865476f"

*/

RETURN math::frac_1_sqrt_2;
-- 0.7071067811865476f
```

## `math::frac_2_pi`

The `math::frac_2_pi` constant represents the fraction 2/π.

```surql
math::frac_2_pi -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.6366197723675814f"

*/

RETURN math::frac_2_pi;
-- 0.6366197723675814f
```

## `math::frac_2_sqrt_pi`

The `math::frac_2_sqrt_pi` constant represents the fraction 2/sqrt(π).

```surql
math::frac_2_sqrt_pi -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1.1283791670955126f"

*/

RETURN math::frac_2_sqrt_pi;
-- 1.1283791670955126f
```

## `math::frac_pi_2`

The `math::frac_pi_2` constant represents the fraction π/2.

```surql
math::frac_pi_2 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1.5707963267948966f"

*/

RETURN math::frac_pi_2;
-- 1.5707963267948966f
```

## `math::frac_pi_3`

The `math::frac_pi_3` constant represents the fraction π/3.

```surql
math::frac_pi_3 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1.0471975511965979f"

*/

RETURN math::frac_pi_3;
-- 1.0471975511965979f
```

## `math::frac_pi_4`

The `math::frac_pi_4` constant represents the fraction π/4.

```surql
math::frac_pi_4 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.7853981633974483f"

*/

RETURN math::frac_pi_4;
-- 0.7853981633974483f
```

## `math::frac_pi_6`

The `math::frac_pi_6` constant represents the fraction π/6.

```surql
math::frac_pi_6 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.5235987755982989f"

*/

RETURN math::frac_pi_6;
-- 0.5235987755982989f
```

## `math::frac_pi_8`

The `math::frac_pi_8` constant represents the fraction π/8.

```surql
math::frac_pi_8 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.39269908169872414f"

*/

RETURN math::frac_pi_8;
-- 0.39269908169872414f
```

## `math::inf`

The `math::inf` constant represents positive infinity.

```surql
math::inf -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "Infinity"

*/

RETURN math::inf;

-- Infinity
```

## `math::interquartile`

The `math::interquartile` function returns the interquartile of an array of numbers.

```surql
math::interquartile(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "51f"

*/

RETURN math::interquartile([ 1, 40, 60, 10, 2, 901 ]);
-- 51f
```

## `math::lerp`

The `math::lerp` function performs a linear interpolation between two numbers based on a given fraction. The fraction will usually be between 0 and 1, where 0 returns `$num_1` and 1 returns `$num_2`.

```surql
math::lerp($num_1: number, $num_2: number, $fraction: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "5f"

*/

RETURN math::lerp(0, 10, 0.5);
-- 5f
```

The function will not return an error if the third argument is not in the range of 0 to 1. Instead, it will extrapolate linearly beyond the first two numbers.

```surql
/**[test]

[[test.results]]
value = "5f"

*/

RETURN math::lerp(0, 10, 2);
-- 20
```

## `math::lerpangle`

The `math::lerpangle` function interpolates between two angles (`$num_1` and `$num_2`) by the given fraction. This is useful for smoothly transitioning between angles.

```surql
math::lerpangle($num_1: number, $num_2: number, $fraction: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "90f"

*/

RETURN math::lerpangle(0, 180, 0.5);
-- 90f
```

## `math::ln`

The `math::ln` function returns the natural logarithm (base e) of a number.

```surql
math::ln(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "2.302585092994046f"

*/

RETURN math::ln(10);
-- 2.302585092994046f
```

## `math::ln_10`

The `math::ln_10` constant represents the natural logarithm (base e) of 10.

```surql
math::ln_10 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "2.302585092994046f"

*/

RETURN math::ln_10;
-- 2.302585092994046f
```

## `math::ln_2`

The `math::ln_2` constant represents the natural logarithm (base e) of 2.

```surql
math::ln_2 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.6931471805599453f"

*/

RETURN math::ln_2;
-- 0.6931471805599453f
```

## `math::log`

The `math::log` function returns the logarithm of a number with a specified base.

```surql
math::log(number, $base: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "2f"

*/

RETURN math::log(100, 10);
-- 2f
```

## `math::log10`

The `math::log10` function returns the base-10 logarithm of a number.

```surql
math::log10(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "3f"

*/

RETURN math::log10(1000);
-- 3f
```

## `math::log10_2`

The `math::log10_2` constant represents the base-10 logarithm of 2.

```surql
math::log10_2 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.3010299956639812f"

*/

RETURN math::log10_2;
-- 0.3010299956639812f
```

## `math::log10_e`

The `math::log10_e` constant represents the base-10 logarithm of e, the base of the natural logarithm (Euler’s number).

```surql
math::log10_e -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.4342944819032518f"

*/

RETURN math::log10_e;

-- 0.4342944819032518f
```

## `math::log2`

The `math::log2` function returns the base-2 logarithm of a number.

```surql
math::log2(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "3f"

*/

RETURN math::log2(8);
-- 3f
```

## `math::log2_10`

The `math::log2_10` constant represents the base-2 logarithm of 10.

```surql
math::log2_10 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "3.321928094887362f"

*/

RETURN math::log2_10;
-- 3.321928094887362f
```

## `math::log2_e`

The `math::log2_e` constant represents the base-2 logarithm of e, the base of the natural logarithm (Euler’s number).

```surql
math::log2_e -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1.4426950408889634f"

*/

RETURN math::log2_e;
-- 1.4426950408889634f
```

## `math::max`

The `math::max` function returns the greatest number from an array of numbers.

```surql
math::max(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "41.42f"

*/

RETURN math::max([ 26.164, 13.746189, 23, 16.4, 41.42 ]);
-- 41.42f
```

See also:

* [`array::max`](/docs/surrealql/functions/database/array#arraymax), which extracts the greatest value from an array of values
* [`time::max`](/docs/surrealql/functions/database/time#timemax), which extracts the greatest datetime from an array of datetimes

## `math::mean`

The `math::mean` function returns the mean of a set of numbers.

```surql
math::mean(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "24.146037800000002f"

*/

RETURN math::mean([ 26.164, 13.746189, 23, 16.4, 41.42 ]);

-- 24.146037800000002f
```

## `math::median`

The `math::median` function returns the median of a set of numbers.

```surql
math::median(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "23f"

*/

RETURN math::median([ 26.164, 13.746189, 23, 16.4, 41.42 ]);
-- 23f
```

## `math::midhinge`

The `math::midhinge` function returns the midhinge of an array of numbers.

```surql
math::midhinge(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "29.5f"

*/

RETURN math::midhinge([ 1, 40, 60, 10, 2, 901 ]);
-- 29.5f
```

## `math::min`

The `math::min` function returns the least number from an array of numbers.

```surql
math::min(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "13.746189f"

*/

RETURN math::min([ 26.164, 13.746189, 23, 16.4, 41.42 ]);
-- 13.746189f
```

See also:

* [`array::min`](/docs/surrealql/functions/database/array#arraymin), which extracts the least value from an array of values
* [`time::min`](/docs/surrealql/functions/database/time#timemin), which extracts the least datetime from an array of datetimes

## `math::mode`

The `math::mode` function returns the value that occurs most often in a set of numbers. In case of a tie, the highest one is returned.

```surql
math::mode(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "901"

[[test.results]]
value = "2"

*/

RETURN math::mode([ 1, 40, 60, 10, 2, 901 ]);
-- 901

RETURN math::mode([ 1, 40, 60, 10, 2, 901, 2 ]);
-- 2
```

## `math::nearestrank`

The `math::nearestrank` function returns the nearest rank of an array of numbers by pullinng the closest extant record from the dataset at the %-th percentile.

```surql
math::nearestrank(array<number>, $percentile: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "40"

*/

RETURN math::nearestrank([1, 40, 60, 10, 2, 901], 50);
-- 40
```

A number for the percentile outside of the range 0 to 100 will return the output `NaN`.

```surql
-- Nan
math::nearestrank([1, 40, 60, 10, 2, 901], 101);

-- Also Nan
math::nearestrank([1, 40, 60, 10, 2, 901], -1);
```

## `math::neg_inf`

The `math::neg_inf` constant represents negative infinity.

```surql
math::neg_inf -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "-Infinity"

*/

RETURN math::neg_inf;

-Infinity
```

## `math::percentile`

The `math::percentile` function returns the value below which a percentage of data falls by getting the N percentile, averaging neighboring records if non-exact.

```surql
math::percentile(array<number>, $percentile: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "25f"

*/

RETURN math::percentile([1, 40, 60, 10, 2, 901], 50);
-- 25f
```

A number for the percentile outside of the range 0 to 100 will return the output `NaN`.

```surql
-- Nan
math::percentile([1, 40, 60, 10, 2, 901], 101);

-- Also Nan
math::percentile([1, 40, 60, 10, 2, 901], -1);
```

## `math::pi`

The `math::pi` constant represents the mathematical constant π.

```surql
math::pi -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "3.141592653589793f"

*/

RETURN math::pi;
-- 3.141592653589793f
```

## `math::pow`

The `math::pow` function returns a number raised to the power of a second number.

```surql
math::pow(number, $raise_to: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1.9671513572895665f"

*/

RETURN math::pow(1.07, 10);
-- 1.9671513572895665f
```

## `math::product`

The `math::product` function returns the product of a set of numbers.

```surql
math::product(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "5619119.004884841f"

*/

RETURN math::product([ 26.164, 13.746189, 23, 16.4, 41.42 ]);
-- 5619119.004884841f
```

## `math::rad2deg`

The `math::rad2deg` function converts an angle from radians to degrees.

```surql
math::rad2deg(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "180f"

*/

RETURN math::rad2deg(3.141592653589793);
-- 180f
```

## `math::round`

The `math::round` function rounds a number up or down to the nearest integer.

```surql
math::round(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "14f"

*/

RETURN math::round(13.53124);
-- 14f
```

## `math::sign`

The `math::sign` function returns the sign of a number, indicating whether the number is positive, negative, or zero.
It returns 1 for positive numbers, -1 for negative numbers, and 0 for zero.

```surql
math::sign(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "-1"

*/

RETURN math::sign(-42);
-- -1
```

## `math::sin`

The `math::sin` function returns the sine of a number, which is assumed to be in radians.

```surql
math::sin(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.8414709848078965f"

*/

RETURN math::sin(1);
-- 0.8414709848078965f
```

## `math::spread`

The `math::spread` function returns the spread of an array of numbers.

```surql
math::spread(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "900"

*/

RETURN math::spread([ 1, 40, 60, 10, 2, 901 ]);
-- 900
```

## `math::sqrt`

The `math::sqrt` function returns the square root of a number.

```surql
math::sqrt(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "3.872983346207417f"

*/

RETURN math::sqrt(15);
-- 3.872983346207417f
```

## `math::sqrt_2`

The `math::sqrt_2` constant represents the square root of 2.

```surql
math::sqrt_2 -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1.4142135623730951f"

*/

RETURN math::sqrt_2;
-- 1.4142135623730951f
```

## `math::stddev`

The `math::stddev` function calculates how far a set of numbers are away from the mean.

```surql
math::stddev(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "359.37167389765153f"

*/

RETURN math::stddev([ 1, 40, 60, 10, 2, 901 ]);
-- 359.37167389765153f
```

As of SurrealDB 3.0.0-beta, this function can be used [inside a table view](/docs/surrealql/statements/select#mathstddev-and-mathvariance-in-table-views).

```surql
DEFINE TABLE person SCHEMALESS;
DEFINE TABLE person_stats AS
	SELECT
		count(),
		age,
		math::stddev(score) AS score_stddev
	FROM person
	GROUP BY age;
```

## `math::sum`

The `math::sum` function returns the total sum of a set of numbers.

```surql
math::sum(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "120.73018900000001f"

*/

RETURN math::sum([ 26.164, 13.746189, 23, 16.4, 41.42 ]);

-- 120.730189
```

This function on its own expects a numeric value at each point in an array, meaning that on its own it will not be able to be used on an array that contains `NONE` or `NULL` values.

```surql
/**[test]

[[test.results]]
error = "Incorrect arguments for function math::sum(). Argument 1 was the wrong type. Expected `number` but found `NONE` when coercing an element of `array<number>`"

*/

math::sum([0, NONE, 10dec, 10.7, NULL]);

-- Error: Incorrect arguments for function math::sum().
-- Argument 1 was the wrong type.
-- Expected `number` but found `NONE` when coercing an element of `array<number>`
```

However, `NONE` and `NULL` can be coalesced into a default value by using the `??` operator (the "null coalescing operator").

```surql
/**[test]

[[test.results]]
value = "0"

[[test.results]]
value = "1000"

*/

NONE ?? 0; -- Finds NONE so returns latter value: 0
1000 ?? 0; -- Finds 1000 so returns 1000 instead of 0
```

Inside an array the [`array::map()`](/docs/surrealql/functions/database/array#arraymap) function can be used to ensure that each value is the number 0 if a `NONE` or `NULL` is encountered.

Classic [array filtering](/docs/surrealql/datamodel/arrays#mapping-and-filtering-on-arrays) can also be used to simply remove any `NONE` or `NULL` values before `math::sum()` is called.

```surql
/**[test]

[[test.results]]
value = "[10dec, 10.7f]"

[[test.results]]
value = "[0, 0, 10dec, 10.7f, 0]"

*/

// Classic array filtering, removes NONE / NULL
[0,NONE,10dec,10.7,NULL][? $this];
// array::map() function, turns NONE / NULL to 0
[0, NONE, 10dec, 10.7, NULL].map(|$num| $num ?? 0);
```

With this mapping in place, `math::sum()` will be guaranteed to work.

```surql
/**[test]

[[test.results]]
value = "20.7dec"

[[test.results]]
value = "20.7dec"

*/

// Classic array filtering
math::sum([0,NONE,10dec,10.7,NULL][? $this]);
// array::map() function
math::sum([0, NONE, 10dec, 10.7, NULL].map(|$num| $num ?? 0));

-- 20.7dec
```

## `math::tan`

The `math::tan` function returns the tangent of a number, which is assumed to be in radians.

```surql
math::tan(number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1.557407724654902f"

*/

RETURN math::tan(1);
-- 1.557407724654902f
```

## `math::tau`

The `math::tau` constant represents the mathematical constant τ, which is equal to 2π.

```surql
math::tau -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "6.283185307179586f"

*/

RETURN math::tau;
-- 6.283185307179586f
```

## `math::top`

The `math::top` function returns the top of an array of numbers.

```surql
math::top(array<number>, $quantity: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[40, 901, 60]"

*/

RETURN math::top([1, 40, 60, 10, 2, 901], 3);
-- [40, 901, 60]
```

## `math::trimean`

The `math::trimean` function returns the trimean of an array of numbers.

```surql
math::trimean(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "27.25f"

*/

RETURN math::trimean([ 1, 40, 60, 10, 2, 901 ]);
-- 27.25f
```

## `math::variance`

The `math::variance` function returns the variance of an array of numbers.

```surql
math::variance(array<number>) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "129148f"

*/

RETURN math::variance([ 1, 40, 60, 10, 2, 901 ]);
-- 129148
```

As of SurrealDB 3.0.0-beta, this function can be used [inside a table view](/docs/surrealql/statements/select#mathstddev-and-mathvariance-in-table-views).

```surql
DEFINE TABLE person SCHEMALESS;
DEFINE TABLE person_stats AS
	SELECT
		count(),
		age,
		math::variance(score) AS score_variance
	FROM person
	GROUP BY age;
```

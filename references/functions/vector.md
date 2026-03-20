# Vector functions

A collection of essential vector operations that provide foundational functionality for numerical computation, machine learning, and data analysis. These operations include distance measurements, similarity coefficients, and other basic and complex operations related to vectors. Through understanding and implementing these functions, we can perform a wide variety of tasks ranging from data processing to advanced statistical analyses.

## `vector::add`

The `vector::add` function performs element-wise addition of two vectors, where each element in the first vector is added to the corresponding element in the second vector.

```surql
vector::add(array, $other: array) -> array
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[2, 4, 6]"

*/

RETURN vector::add([1, 2, 3], [1, 2, 3]);

-- [2, 4, 6]
```

## `vector::angle`

The `vector::angle` function computes the angle between two vectors, providing a measure of the orientation difference between them.

```surql
vector::angle(array, $other: array) -> number
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.36774908225917935f"

*/

RETURN vector::angle([5, 10, 15], [10, 5, 20]);

-- 0.36774908225917935f
```

## `vector::cross`

The `vector::cross` function computes the cross product of two vectors, which results in a vector that is orthogonal (perpendicular) to the plane containing the original vectors.

```surql
vector::cross(array, $other: array) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[-3, 6, -3]"

*/

RETURN vector::cross([1, 2, 3], [4, 5, 6]);

[-3, 6, -3]
```

## `vector::divide`

The `vector::divide` function performs element-wise division between two vectors, where each element in the first vector is divided by the corresponding element in the second vector.

```surql
vector::divide(array, $other: array) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[NaN, 20, 15, 0]"

*/

RETURN vector::divide([10, -20, 30, 0], [0, -1, 2, -3]);

[NaN, 20, 15, 0]
```

## `vector::dot`

The `vector::dot` function computes the dot product of two vectors, which is the sum of the products of the corresponding entries of the two sequences of numbers.

```surql
vector::dot(array, $other: array) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "14"

*/

RETURN vector::dot([1, 2, 3], [1, 2, 3]);

-- 14
```

## `vector::magnitude`

The `vector::magnitude` function computes the magnitude (or length) of a vector, providing a measure of the size of the vector in multi-dimensional space.

```surql
vector::magnitude(array) -> number
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "8.54400374531753f"

*/

RETURN vector::magnitude([ 1, 2, 3, 3, 3, 4, 5 ]);

-- 8.54400374531753f
```

## `vector::multiply`

The `vector::multiply` function performs element-wise multiplication of two vectors, where each element in the first vector is multiplied by the corresponding element in the second vector.

```surql
vector::multiply(array, $other: array) -> number
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[1, 4, 9]"

*/

RETURN vector::multiply([1, 2, 3], [1, 2, 3]);

-- [1, 4, 9]
```

## `vector::normalize`

The `vector::normalize` function computes the normalization of a vector, transforming it to a unit vector (a vector of length 1) that maintains the original direction.

```surql
vector::normalize(array) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[0.8f, 0.6f]"

*/

RETURN vector::normalize([ 4, 3 ]);

-- [0.8f, 0.6f]
```

## `vector::project`

The `vector::project` function computes the projection of one vector onto another, providing a measure of the shadow of one vector on the other. The projection is obtained by multiplying the magnitude of the given vectors with the cosecant of the angle between the two vectors.

```surql
vector::project(array, $other: array) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[1.6623376623376624f, 2.077922077922078f, 2.4935064935064934f]"

*/

RETURN vector::project([1, 2, 3], [4, 5, 6]);

-- [1.6623376623376624f, 2.077922077922078f, 2.4935064935064934f]
```

## `vector::scale`

The `vector::scale` function multiplies each item in a vector by a number.

```surql
vector::scale(array, $other: number) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[15, 5, 25, -15, 35, 10]"

*/

RETURN vector::scale([3, 1, 5, -3, 7, 2], 5);

-- [15,	5, 25, -15, 35, 10]
```

## `vector::subtract`

The `vector::subtract` function performs element-wise subtraction between two vectors, where each element in the second vector is subtracted from the corresponding element in the first vector.

```surql
vector::subtract(array, $other: array) -> array
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "[1, 3, 5]"

*/

RETURN vector::subtract([4, 5, 6], [3, 2, 1]);

-- [1, 3, 5]
```

## `vector::distance::chebyshev`

The `vector::distance::chebyshev` function computes the Chebyshev distance (also known as maximum value distance) between two vectors, which is the greatest of their differences along any coordinate dimension.

```surql
vector::distance::chebyshev(array, $other: array) -> number
```
The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "6f"

*/

RETURN vector::distance::chebyshev([2, 4, 5, 3, 8, 2], [3, 1, 5, -3, 7, 2]);

-- 6f
```

## `vector::distance::euclidean`

The `vector::distance::euclidean` function computes the Euclidean distance between two vectors, providing a measure of the straight-line distance between two points in a multi-dimensional space.

```surql
vector::distance::euclidean(array, $other: array) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "432.43496620879307f"

*/

RETURN vector::distance::euclidean([10, 50, 200], [400, 100, 20]);

-- 432.43496620879307f
```

## `vector::distance::hamming`

The `vector::distance::hamming` function computes the Hamming distance between two vectors, measuring the minimum number of substitutions required to change one vector into the other, useful for comparing strings or codes.

```surql
vector::distance::hamming(array, $other: array) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "1"

*/

RETURN vector::distance::hamming([1, 2, 2], [1, 2, 3]);

-- 1
```

## `vector::distance::knn`

The `vector::distance::knn` function returns the distance computed during the query by the Knn operator (avoiding recomputation).

```surql
vector::distance::knn() -> number
```

The following example shows this function, and its output, when used in a [`SELECT`](/docs/surrealql/statements/select) statement:

```surql
/**[test]

[[test.results]]
value = "[{ id: pts:1, point: [1, 2, 3, 4] }]"

[[test.results]]
value = "[{ id: pts:2, point: [4, 5, 6, 7] }]"

[[test.results]]
value = "[{ id: pts:3, point: [8, 9, 10, 11] }]"

[[test.results]]
value = "[{ dist: 2f, id: pts:1 }, { dist: 4f, id: pts:2 }]"

*/
CREATE pts:1 SET point = [1,2,3,4];
CREATE pts:2 SET point = [4,5,6,7];
CREATE pts:3 SET point = [8,9,10,11];
SELECT id, vector::distance::knn() AS dist FROM pts WHERE point <|2,EUCLIDEAN|> [2,3,4,5];
```

```surql
[
			{
				id: pts:1,
				dist: 2f
			},
			{
				id: pts:2,
				dist: 4f
			}
]
```

## `vector::distance::manhattan`

The `vector::distance::manhattan`  function computes the Manhattan distance (also known as the L1 norm or Taxicab geometry) between two vectors, which is the sum of the absolute differences of their corresponding elements.

```surql
vector::distance::manhattan(array, $other: array) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "13"

*/

RETURN vector::distance::manhattan([10, 20, 15, 10, 5], [12, 24, 18, 8, 7]);

-- 13
```

## `vector::distance::minkowski`

The `vector::distance::minkowski` function computes the Minkowski distance between two vectors, a generalization of other distance metrics such as Euclidean and Manhattan when parameterized with different values of p.

```surql
vector::distance::minkowski(array, $other: array, $p_value: number) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "4.862944131094279f"

*/

RETURN vector::distance::minkowski([10, 20, 15, 10, 5], [12, 24, 18, 8, 7], 3);

-- 4.862944131094279f
```

## `vector::similarity::cosine`

The `vector::similarity::cosine` function computes the Cosine similarity between two vectors, indicating the cosine of the angle between them, which is a measure of how closely two vectors are oriented to each other.

```surql
vector::similarity::cosine(array, $other: array) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.15258215962441316f"

*/

RETURN vector::similarity::cosine([10, 50, 200], [400, 100, 20]);

-- 0.15258215962441316f
```

## `vector::similarity::jaccard`

The `vector::similarity::jaccard` function computes the Jaccard similarity between two vectors, measuring the intersection divided by the union of the datasets represented by the vectors.

```surql
vector::similarity::jaccard(array, $other: array) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.3333333333333333f"

*/

RETURN vector::similarity::jaccard([0,1,2,5,6], [0,2,3,4,5,7,9]);

-- 0.3333333333333333f
```

## `vector::similarity::pearson`

The `vector::similarity::pearson` function Computes the Pearson correlation coefficient between two vectors, reflecting the degree of linear relationship between them.

```surql
vector::similarity::pearson(array, array) -> number
```

The following example shows this function, and its output, when used in a [`RETURN`](/docs/surrealql/statements/return) statement:

```surql
/**[test]

[[test.results]]
value = "0.9819805060619659f"

*/

RETURN vector::similarity::pearson([1,2,3], [1,5,7]);

-- 0.9819805060619659f
```

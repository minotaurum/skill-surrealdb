# Sequence functions

These functions can be used to work with [sequences](/docs/surrealql/statements/define/sequence).

## `sequence::next`

The `sequence::next` function returns the next value in a sequence.

```surql
sequence::next($seq_name: string) -> int
```

```surql 
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "100"

*/

DEFINE SEQUENCE mySeq2 BATCH 1000 START 100 TIMEOUT 5s;
sequence::nextval('mySeq2');

-- 100
```

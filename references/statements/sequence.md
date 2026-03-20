# `ALTER SEQUENCE` statement

The `ALTER SEQUENCE` statement is used to modify a defined sequence.

  
```surrealql
ALTER SEQUENCE [ IF EXISTS ] @name [ TIMEOUT @duration ]
```

  

  

## Examples

The timeout of a sequence can be modified via an `ALTER SEQUENCE` statement. For example, a sequence can be included in the schema but effectively disabled if given a timeout of 0ns, after which `ALTER SEQUENCE` can be used to modify the timeout to make it available.

```surql
DEFINE SEQUENCE mySeq3 BATCH 1000 START 100 TIMEOUT 0ns;
INFO FOR DB.sequences;
sequence::nextval('mySeq3');

ALTER SEQUENCE mySeq3 TIMEOUT 100ms;
sequence::nextval('mySeq3');
```

```surql
-------- Query --------
{ mySeq3: 'DEFINE SEQUENCE mySeq3 BATCH 1000 START 100 TIMEOUT 0ns' },

-------- Query --------
'Thrown error: The query was not executed because it exceeded the timeout: 0ns'

-------- Query --------
100
```

## See also

* [`DEFINE SEQUENCE`](/docs/surrealql/statements/define/sequence)

# `ALTER NAMESPACE` statement

The `ALTER NAMESPACE` statement can be used to modify the namespace. `ALTER NAMESPACE` is used on the current namespace, which is why a `IF EXISTS` clause does not exist.

## Statement syntax

  
```surrealql
ALTER NAMESPACE COMPACT
```

  

  

## COMPACT

Performs storage compaction onPerforms storage compaction on the current namespace keyspace. To compact other resources, use [ALTER SYSTEM](/docs/surrealql/statements/alter/system) to compact the entire datastore, [ALTER DATABASE](/docs/surrealql/statements/alter/database) to compact the current database keyspace, or [ALTER TABLE](/docs/surrealql/statements/alter/table) to compact a specific table keyspace.

The actual compaction used will depend on the datastore, such as RocksDB or SurrealKV.

This clause will not work with in-memory storage which has nothing persistent to compact, producing the following error:

```surql
'The storage layer does not support compaction requests.'
```

A successful compaction will return `NONE`.

```surql
ALTER NAMESPACE COMPACT;
-- NONE
```

## See also

* [`DEFINE NAMESPACE`](/docs/surrealql/statements/define/namespace)

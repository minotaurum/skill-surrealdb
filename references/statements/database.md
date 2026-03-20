# `ALTER DATABASE` statement

The `ALTER DATABASE` statement can be used to modify the database. `ALTER DATABASE` is used on the current database, which is why a `IF EXISTS` clause does not exist.

## Statement syntax

  
```surrealql
ALTER DATABASE COMPACT
```

  

  

## COMPACT

Performs storage compaction on the current database keyspace. To compact other resources, use [ALTER SYSTEM](/docs/surrealql/statements/alter/database) to compact the entire datastore, [ALTER NAMESPACE](/docs/surrealql/statements/alter/namespace) to compact the current namespace keyspace, or [ALTER TABLE](/docs/surrealql/statements/alter/table) to compact a specific table keyspace.

The actual compaction used will depend on the datastore, such as RocksDB or SurrealKV.

This clause will not work with in-memory storage which has nothing persistent to compact, producing the following error:

```surql
'The storage layer does not support compaction requests.'
```

A successful compaction will return `NONE`.

```surql
ALTER DATABASE COMPACT;
-- NONE
```

## See also

* [`DEFINE DATABASE`](/docs/surrealql/statements/define/database)

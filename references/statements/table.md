---
sidebar_position: 8
sidebar_label: TABLE
title: ALTER TABLE statement | SurrealQL
description: The ALTER statement can be used to change authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---


# `ALTER TABLE` statement

The `ALTER TABLE` statement is used to alter a defined table.

  
```surrealql
ALTER TABLE [
	[ IF EXISTS ] @name
		[ DROP COMMENT ]
        [ DROP CHANGEFEED ]
        [ COMPACT ]
		[ SCHEMAFULL | SCHEMALESS ]
		[ PERMISSIONS [ NONE | FULL
			| FOR select @expression
			| FOR create @expression
			| FOR update @expression
			| FOR delete @expression
		] ]
    [ CHANGEFEED @duration ]
    [ COMMENT @string ] 
    [ CHANGEFEED ]
]
```

  

  

## COMPACT

Performs storage compaction on a specific table keyspace. To compact other resources, use [ALTER SYSTEM](/docs/surrealql/statements/alter/system) to compact the entire datastore, [ALTER NAMESPACE](/docs/surrealql/statements/alter/namespace) to compact the current namespace keyspace, or [ALTER DATABASE](/docs/surrealql/statements/alter/database) to compact the current database keyspace.

The actual compaction used will depend on the datastore, such as RocksDB or SurrealKV.

This clause will not work with in-memory storage which has nothing persistent to compact, producing the following error:

```surql
'The storage layer does not support compaction requests.'
```

A successful compaction will return `NONE`.

```surql
ALTER TABLE user COMPACT;
-- NONE
```

## See also

* [`DEFINE TABLE`](/docs/surrealql/statements/define/table)
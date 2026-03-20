---
name: surrealdb-expert
description: Expert AI assistant for SurrealDB, SurrealQL, datamodels, and query writing. Make sure to use this skill whenever the user mentions SurrealDB, SurrealQL, databases, graph queries, document structures, record IDs, or wants to write queries/functions related to a database, even if they don't explicitly mention "SurrealDB" by name. 
---

# SurrealDB Expert

You are an expert on [SurrealDB](https://surrealdb.com/), a multi-model database supporting document, graph, and relational paradigms. Your role is to write accurate and robust SurrealQL queries, data models, and database functions. 

## When to use this skill
Whenever the user wants you to write queries, configure a database schema, write SurrealDB functions or transactions, or optimize data structures, rely on this skill to ensure syntax and structure match SurrealDB's features.

## General Best Practices
- **Record IDs**: In SurrealDB, records have IDs like `table:id`. They can be complex objects or arrays as well. Always leverage Record IDs for rapid retrieval instead of standard `id = 'x'` where possible.
- **Graph vs Relational**: Differentiate between relations (using `RELATE`) and direct links (record links). Use graphs when traversal is needed (`->edge->node`). Use record links for simple cross-document mappings.
- **Variables**: SurrealQL uses `$variable_name` syntax for query parameters via JSON encapsulation.
- **Futures**: Computed fields and dynamic fields should take advantage of `<future>` syntax or `DEFINE FIELD` computed properties.
- **Strict Data Typing**: Always `DEFINE TABLE` with schemafull settings unless there's a strong reason for schemaless. `DEFINE FIELD` with strict types and granular assertions `ASSERT`.

## Progressive Disclosure of Documentation
SurrealQL is vast. To save context space while retrieving accurate syntax, you have access to extensive references in the `references/` directory. **Read the corresponding reference file when you are unsure about syntax or need real examples.**

1. **Datamodel & Structure (`references/datamodel.md`)**
   - Read this when defining tables, fields, indexes, events, or working with complex JSON properties, arrays, geometry, or cast types.
   - It covers the core `datamodel` and data typing.

2. **Statements & Clauses (`references/statements_and_clauses.md`)**
   - Read this when writing complex queries using `SELECT`, `CREATE`, `INSERT`, `UPDATE`, `UPSERT`, `RELATE`, `DELETE`, etc.
   - It also details clauses like `WHERE`, `SPLIT`, `GROUP`, `FETCH`, `LIMIT`, `START`, and graph syntax traversing.
   - Transaction syntax (`BEGIN`, `COMMIT`, `CANCEL`) is also covered here.

3. **Operators & Parameters (`references/operators_and_parameters.md`)**
   - Read this when you need info about arithmetic operators, conditional operators (e.g., `?=`, `CONTAINS`, `INSIDE`), graph navigation arrows (`->`, `<-`, `<->`).
   - Also covers `$session`, `$auth`, `$token` parameters used in authentication contexts.

4. **Functions (`references/functions.md`)**
   - Read this when you need to use built-in SurrealDB functions.
   - Covers `array::`, `crypto::`, `duration::`, `geo::`, `http::`, `math::`, `parse::`, `rand::`, `search::`, `session::`, `string::`, `time::`, `type::`, `vector::`.

5. **Demo & Comments (`references/misc.md`)**
   - Read this if you want to understand general comments (`--`, `/* */`), basic interactive console demos, or overall syntax flow.

## Response Guidelines
- **Always confirm syntactical correctness** by referring to the docs if handling a clause or function you aren't 100% sure about.
- Explain the query step-by-step so the user can learn the rationale.
- If defining a schema or complex graph relation, provide clear insert/query examples demonstrating its usage. 

### Examples

**Example 1: Defining a User Table with constraints**
```surrealql
DEFINE TABLE user SCHEMAFULL;
DEFINE FIELD name ON user TYPE string;
DEFINE FIELD email ON user TYPE string ASSERT is::email($value);
DEFINE FIELD created_at ON user TYPE datetime DEFAULT time::now() READONLY;

DEFINE INDEX emailIdx ON user COLUMNS email UNIQUE;
```

**Example 2: Relating two records in a Graph**
```surrealql
-- Assuming person:paolo and person:alex exist
RELATE person:paolo->knows->person:alex SET source = 'work', met_on = time::now();

-- Querying the relations
SELECT ->knows->person AS friends FROM person:paolo;
```

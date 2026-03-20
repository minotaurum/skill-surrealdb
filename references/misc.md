# SurrealQL Demo & Comments

# Demo data

To quickly test out SurrealDB and SurrealQL functionality, we've included two demo datasets here in `.surql` files which you can download and [`import`](/docs/surrealdb/cli/import) into SurrealDB using the [CLI](/docs/surrealdb/cli).

## Surreal Deal Store - there is a lot in store for you!

Surreal Deal Store is our new and improved demo dataset based on our [SurrealDB Store](https://surrealdb.store/).
The dataset is made up of 12 tables using both [graph relations](/docs/surrealql/statements/relate) and [record links](/docs/surrealql/datamodel/records).

In the diagram below, the nodes in pink are the [standard tables](/docs/surrealql/statements/define/table), the ones in purple represent the [edge tables](/docs/surrealql/statements/relate) which shows relationships between records and SurrealDB as a graph database. The nodes in grey are the [pre-computed table views](/docs/surrealql/statements/define/table).

### Download

  

  

### Import

Once one of the datasets has been downloaded, it's now time to [start the server](/docs/surrealdb/cli/start).

```bash
# Create a new in-memory server
surreal start --user root --pass secret --allow-all
```

Lastly, use the [import command](/docs/surrealdb/cli/import) to add the dataset.

Use the command below to import the [surreal deal store dataset](https://datasets.surrealdb.com/surreal-deal-store.surql):

```bash
surreal import --endpoint http://localhost:8000 --user root --pass secret --ns main --db main surreal-deal-store.surql
```

To import the surreal downloaded the [Surreal Deal store (mini)](https://datasets.surrealdb.com/surreal-deal-store-mini.surql) use the command below:

```bash
surreal import --endpoint http://localhost:8000 --user root --pass secret --ns main --db main surreal-deal-store-mini.surql
```

Please be aware that the import process might take a few seconds.

### Using Curl

First, start the SurrealDB server:

```bash
# Create a new in-memory server
surreal start --user root --pass secret --allow-all
```

Then download the file and load it into the database:

```bash
# Download the file
curl -L "https://datasets.surrealdb.com/surreal-deal-store.surql" -o surreal-deal-store.surql

# Load the file into the database using the rest endpoint
curl -v -X POST -u "root:secret" -H "Surreal-NS: main" -H "Surreal-DB: main" -H "Accept: application/json" --data-binary @surreal-deal-store.surql http://localhost:8000/import
```

If you want to use the mini version:

```bash
# Download the file
curl -L "https://datasets.surrealdb.com/surreal-deal-store-mini.surql" -o surreal-deal-store-mini.surql

# Load the file into the database using the rest endpoint
curl -v -X POST -u "root:secret" -H "Surreal-NS: main" -H "Surreal-DB: main" -H "Accept: application/json" --data-binary @surreal-deal-store-mini.surql http://localhost:8000/import
```

### Sample queries

Here are some sample queries you can run on the Surreal Deal Store dataset. We've also included a [Surrealist Mini](https://app.surrealdb.com/mini) below to help you run these queries.

> [!NOTE]
> The query results below have been limited to 4 rows for brevity. If you remove the `LIMIT 4` clause from the queries, you'll see the full results.

# Comments

In SurrealQL, comments can be written in a number of different ways.

```surql
/*
In SurrealQL, comments can be written as single-line
or multi-line comments, and comments can be used and
interspersed within statements.
*/

SELECT * FROM /* get all users */ user;

# There are a number of ways to use single-line comments
SELECT * FROM user;

// Alternatively using two forward-slash characters
SELECT * FROM user;

-- Another way is to use two dash characters
SELECT * FROM user;
```

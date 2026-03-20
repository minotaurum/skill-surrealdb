# SurrealQL Statements & Clauses


## access.mdx

---
sidebar_position: 0
sidebar_label: ACCESS 
title: ACCESS statement | SurrealQL
description: The ACCESS statement can be used to manage access grants.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `ACCESS` statement

<Since v="v2.2.0" />

> [!CAUTION]
> Currently, the `ACCESS` statement is an experimental feature intended to be used for validating its suitability and security. As such, it may be subject to breaking changes and may present unidentified security issues. Do not rely on this feature in production applications.

The `ACCESS` statement can be used to manage access grants. It provides the ability to generate access grants using certain access methods, such as bearer keys defined with the [`DEFINE ACCESS ... TYPE BEARER`](/docs/surrealql/statements/define/access/bearer) statement, as well as the ability to show, revoke and purge such grants.

By default, the `ACCESS` statement will default to referencing access methods defined at the current level specified with the [`USE`](/docs/surrealql/statements/use) statement. As with other statements, access methods defined at any level can be referenced by using the `ON` clause.

Operations that either create, revoke or purge access grants using the `ACCESS` statement will be logged in the server as long as it is running with the `INFO` level (the default) or any higher verbosity level. These logs are identified by the `surrealdb_core::sql::statements::access` prefix.

### Statement syntax

<Tabs syncKey="access-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
ACCESS @name [ ON [ ROOT | NAMESPACE | DATABASE ] ]
	GRANT [ FOR USER @name | FOR RECORD @record ]
	| SHOW [ GRANT @id | ALL | WHERE @expression ] 
	| REVOKE [ GRANT @id | ALL | WHERE @expression ] 
	| PURGE [ EXPIRED | REVOKED [ , EXPIRED | REVOKED ] ] [ FOR @duration ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">


export const accessAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "ACCESS" },
        { type: "NonTerminal", text: "@name" },
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "ON" },
              { type: "Choice", index: 1, children: [
                { type: "Terminal", text: "ROOT" },
                { type: "Terminal", text: "NAMESPACE" },
                { type: "Terminal", text: "DATABASE" }
              ]}
            ]
          }
        },
        {
          type: "Choice",
          index: 1,
          children: [
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "GRANT" },
                  {
                    type: "Choice",
                    index: 1,
                    children: [
                      {
                        type: "Sequence",
                        children: [
                          { type: "Terminal", text: "FOR" },
                          { type: "Terminal", text: "USER" },
                          { type: "NonTerminal", text: "@name" }
                        ]
                      },
                      {
                        type: "Sequence",
                        children: [
                          { type: "Terminal", text: "FOR" },
                          { type: "Terminal", text: "RECORD" },
                          { type: "NonTerminal", text: "@record" }
                        ]
                      }
                    ]
                  }
                ]
              },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "SHOW" },
                  {
                    type: "Choice",
                    index: 1,
                    children: [
                      {
                        type: "Sequence",
                        children: [
                          { type: "Terminal", text: "GRANT" },
                          { type: "NonTerminal", text: "@id" }
                        ]
                      },
                      { type: "Terminal", text: "ALL" },
                      {
                        type: "Sequence",
                        children: [
                          { type: "Terminal", text: "WHERE" },
                          { type: "NonTerminal", text: "@expression" }
                        ]
                      }
                    ]
                  }
                ]
              },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "REVOKE" },
                  {
                    type: "Choice",
                    index: 1,
                    children: [
                      {
                        type: "Sequence",
                        children: [
                          { type: "Terminal", text: "GRANT" },
                          { type: "NonTerminal", text: "@id" }
                        ]
                      },
                      { type: "Terminal", text: "ALL" },
                      {
                        type: "Sequence",
                        children: [
                          { type: "Terminal", text: "WHERE" },
                          { type: "NonTerminal", text: "@expression" }
                        ]
                      }
                    ]
                  }
                ]
              },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "PURGE" },
                  {
                    type: "Choice",
                    index: 1,
                    children: [
                      { type: "Terminal", text: "EXPIRED" },
                      {
                        type: "Sequence",
                        children: [
                          { type: "Terminal", text: "REVOKED" },
                          {
                            type: "Optional",
                            child: {
                              type: "Sequence",
                              children: [
                                { type: "Terminal", text: "," },
                                {
                                  type: "Choice",
                                  index: 0,
                                  children: [
                                    { type: "Terminal", text: "EXPIRED" },
                                    { type: "Terminal", text: "REVOKED" }
                                  ]
                                }
                              ]
                            }
                          }
                        ]
                      }
                    ]
                  },
                  {
                    type: "Optional",
                    child: {
                      type: "Sequence",
                      children: [
                        { type: "Terminal", text: "FOR" },
                        { type: "NonTerminal", text: "@duration" }
                      ]
                    }
                  }
                ]
              }
            ]
        }
      ]
    }
  ]
};

<RailroadDiagram ast={accessAst} className="my-6" />

  </TabItem>
</Tabs>

## `GRANT`

The `GRANT` clause creates and returns a grant for a certain subject using the specified access method. This subject can be a [system user](/docs/surrealdb/security/authentication#system-users) or [record user](/docs/surrealdb/security/authentication#record-users). Access grants can be used to access SurrealDB as that subject until they become expired or revoked.

When creating a grant, a secret (e.g. a key) corresponding with the grant will be returned. This secret should be stored securely, as it will no longer be displayed by SurrealDB, instead being printed as `[REDACTED]` whenever the grant details are shown.

```syntax title="SurrealQL Syntax"
ACCESS @name [ ON [ ROOT | NAMESPACE | DATABASE ] ] 
	GRANT [ FOR USER @name | FOR RECORD @record ]
```

#### Example: Grant for Automation using System User

```surql
-- Define system user for automation
DEFINE USER automation ON DATABASE PASSWORD 'secret' ROLES VIEWER;
-- Define bearer access method to generate API keys
DEFINE ACCESS api ON DATABASE TYPE BEARER FOR USER DURATION FOR GRANT 10d;
-- Generate bearer grant to be used by the automation
ACCESS api GRANT FOR USER automation;
```

```surql title="Response"
-- Query 1
NONE
-- Query 2
NONE
-- Query 3
{
        ac: 'api',
        creation: d'2024-12-16T16:15:51.517384293Z',
        expiration: d'2024-12-26T16:15:51.517386053Z',
        grant: {
                id: 'BNb2pS0GmaJz',
                key: 'surreal-bearer-BNb2pS0GmaJz-5eTfQ5uEu8jbRb3oblqVMAt8'
        },
        id: 'BNb2pS0GmaJz',
        revocation: NONE,
        subject: {
                user: 'automation'
        },
        type: 'bearer'
}
```

#### Example: Grant for End-User using Record User

```surql
-- Create record representing a user
CREATE user:1 CONTENT { name: "tobie" };
-- Define bearer access method to generate API keys
DEFINE ACCESS api ON DATABASE TYPE BEARER FOR RECORD DURATION FOR GRANT 10d;
-- Generate bearer grant to be used by the user
ACCESS api GRANT FOR RECORD user:1;
```

```surql title="Response"
-- Query 1
[
        {
                id: user:1,
                name: 'tobie'
        }
]
-- Query 2
NONE
-- Query 3
{
        ac: 'api',
        creation: d'2024-12-16T16:16:41.996932810Z',
        expiration: d'2024-12-26T16:16:41.996934501Z',
        grant: {
                id: 'sRLEKGxObJuM',
                key: 'surreal-bearer-sRLEKGxObJuM-iUUFe1vijFDaFDW7jceZJDkX'
        },
        id: 'sRLEKGxObJuM',
        revocation: NONE,
        subject: {
                record: user:1
        },
        type: 'bearer'
}
```

## `SHOW`

The `SHOW` clause displays the details of grants created with a specific access method. The statement allows showing the details of individual grants, all grants or only grants matching a particular SurrealQL expression. Beware that, in situations where grants are automatically created, showing all grants at once may be impractical and filtering is advised.

Note that any secrets (e.g. keys) associated with the grant will not be displayed and instead will be shown as `[REDACTED]`.

```syntax title="SurrealQL Syntax"
ACCESS @name [ ON [ ROOT | NAMESPACE | DATABASE ] ]
	SHOW [ GRANT @id | ALL | WHERE @expression ] 
```

#### Example: Showing the details of a specific grant 

```surql
-- Create record representing a user
CREATE user:1 CONTENT { name: "tobie" };
-- Define bearer access method to generate API keys
DEFINE ACCESS api ON DATABASE TYPE BEARER FOR RECORD DURATION FOR GRANT 10d;
-- Generate bearer grant to be used by the user
ACCESS api GRANT FOR RECORD user:1;
```

```surql title="Response"
-- Query 1
[
        {
                id: user:1,
                name: 'tobie'
        }
]
-- Query 2
NONE
-- Query 3
{
        ac: 'api',
        creation: d'2024-12-16T16:17:24.903832476Z',
        expiration: d'2024-12-26T16:17:24.903834523Z',
        grant: {
                id: 'JdvDFKMCVYoM',
                key: 'surreal-bearer-JdvDFKMCVYoM-0ahEAVY6egVdg33Vs5gc6J4h'
        },
        id: 'JdvDFKMCVYoM',
        revocation: NONE,
        subject: {
                record: user:1
        },
        type: 'bearer'
}
```


```surql
ACCESS api SHOW GRANT JdvDFKMCVYoM;
```

```surql title="Response"
{
        ac: 'api',
        creation: d'2024-12-16T16:17:24.903832476Z',
        expiration: d'2024-12-26T16:17:24.903834523Z',
        grant: {
                id: 'JdvDFKMCVYoM',
                key: '[REDACTED]'
        },
        id: 'JdvDFKMCVYoM',
        revocation: NONE,
        subject: {
                record: user:1
        },
        type: 'bearer'
}
```

#### Example: Showing the details of all grants for users of a certain name

Since the `subject` attribute of grants associated with a record is a record identifier, it can be used as a [record link](/docs/surrealql/datamodel/records) in order to access any record fields. This can be used to filter grants associated with record users matching certain conditions based on arbitrary data.

```surql
-- Create records representing users
CREATE user:1 CONTENT { name: "tobie" };
CREATE user:2 CONTENT { name: "jaime" };
-- Define bearer access method to generate API keys
DEFINE ACCESS api ON DATABASE TYPE BEARER FOR RECORD DURATION FOR GRANT 10d;
-- Generate bearer grants to be used by the users
ACCESS api GRANT FOR RECORD user:1;
ACCESS api GRANT FOR RECORD user:2;
```

```surql title="Response"
-- Query 1
[
        {
                id: user:1,
                name: 'tobie'
        }
]
-- Query 2
[
        {
                id: user:2,
                name: 'jaime'
        }
]
-- Query 3
NONE
-- Query 4
{
        ac: 'api',
        creation: d'2024-12-16T16:18:57.061692071Z',
        expiration: d'2024-12-26T16:18:57.061694228Z',
        grant: {
                id: 'HaJ19zCnP6RI',
                key: 'surreal-bearer-HaJ19zCnP6RI-R545vHcTbSCYdHnxIsVnjSFu'
        },
        id: 'HaJ19zCnP6RI',
        revocation: NONE,
        subject: {
                record: user:1
        },
        type: 'bearer'
}
-- Query 5
{
        ac: 'api',
        creation: d'2024-12-16T16:18:57.063673293Z',
        expiration: d'2024-12-26T16:18:57.063674755Z',
        grant: {
                id: 'ND2ZegEHfUGl',
                key: 'surreal-bearer-ND2ZegEHfUGl-JGPSr162qJ2bN8kURV8mYaLv'
        },
        id: 'ND2ZegEHfUGl',
        revocation: NONE,
        subject: {
                record: user:2
        },
        type: 'bearer'
}
```


```surql
ACCESS api SHOW WHERE subject.record.name = "tobie";
```

```surql title="Response"
[
        {
                ac: 'api',
                creation: d'2024-12-16T16:18:57.061692071Z',
                expiration: d'2024-12-26T16:18:57.061694228Z',
                grant: {
                        id: 'HaJ19zCnP6RI',
                        key: '[REDACTED]'
                },
                id: 'HaJ19zCnP6RI',
                revocation: NONE,
                subject: {
                        record: user:1
                },
                type: 'bearer'
        }
]
```

## `REVOKE`

The `REVOKE` clause revokes grants created with a specific access method. Revoking a grant ensures that the grant can no longer be used to authenticate. The grant will continue existing in revoked form and the time of the revocation is recorded in the details of the grant. Grants that have already been revoked cannot be revoked again. The statement allows revoking individual grants, all grants or only grants matching a particular SurrealQL expression.

```syntax title="SurrealQL Syntax"
ACCESS @name [ ON [ ROOT | NAMESPACE | DATABASE ] ]
	REVOKE [ GRANT @id | ALL | WHERE @expression ] 
]
```

#### Example: Revoking a specific grant

```surql
-- Create record representing a user
CREATE user:1 CONTENT { name: "tobie" };
-- Define bearer access method to generate API keys
DEFINE ACCESS api ON DATABASE TYPE BEARER FOR RECORD DURATION FOR GRANT 10d;
-- Generate bearer grant to be used by the user
ACCESS api GRANT FOR RECORD user:1;
```

```surql title="Response"
-- Query 1
[
        {
                id: user:1,
                name: 'tobie'
        }
]
-- Query 2
NONE
-- Query 3
{
        ac: 'api',
        creation: d'2024-12-17T10:36:09.215762475Z',
        expiration: d'2024-12-27T10:36:09.216227523Z',
        grant: {
                id: 'NJ2I2d7OXxN9',
                key: 'surreal-bearer-NJ2I2d7OXxN9-Oa5LqF36IzfURpo6Bhxy9WMF'
        },
        id: 'NJ2I2d7OXxN9',
        revocation: NONE,
        subject: {
                record: user:1
        },
        type: 'bearer'
}
```


```surql
ACCESS api REVOKE GRANT NJ2I2d7OXxN9;
```

```surql title="Response"
[
        [
                {
                        ac: 'api',
                        creation: d'2024-12-17T10:36:09.215762475Z',
                        expiration: d'2024-12-27T10:36:09.216227523Z',
                        grant: {
                                id: 'NJ2I2d7OXxN9',
                                key: '[REDACTED]'
                        },
                        id: 'NJ2I2d7OXxN9',
                        revocation: d'2024-12-17T10:36:52.740438379Z',
                        subject: {
                                record: user:1
                        },
                        type: 'bearer'
                }
        ]
]
```

#### Example: Revoking all grants for users of a certain name

Since the `subject` attribute of grants associated with a record is a record identifier, it can be used as a [record link](/docs/surrealql/datamodel/records) in order to access any record fields. This can be used to filter grants associated with record users matching certain conditions based on arbitrary data.

```surql
-- Create records representing users
CREATE user:1 CONTENT { name: "tobie" };
CREATE user:2 CONTENT { name: "jaime" };
-- Define bearer access method to generate API keys
DEFINE ACCESS api ON DATABASE TYPE BEARER FOR RECORD DURATION FOR GRANT 10d;
-- Generate bearer grants to be used by the users
ACCESS api GRANT FOR RECORD user:1;
ACCESS api GRANT FOR RECORD user:2;
```

```surql title="Response"
-- Query 1
[
        {
                id: user:1,
                name: 'tobie'
        }
]
-- Query 2
[
        {
                id: user:2,
                name: 'jaime'
        }
]
-- Query 3
NONE
-- Query 4
{
        ac: 'api',
        creation: d'2024-12-17T10:42:35.040901759Z',
        expiration: d'2024-12-27T10:42:35.040903414Z',
        grant: {
                id: 'mjSACes6sej4',
                key: 'surreal-bearer-mjSACes6sej4-WbEPMgmLTO3Jfg3po4we9m0V'
        },
        id: 'mjSACes6sej4',
        revocation: NONE,
        subject: {
                record: user:1
        },
        type: 'bearer'
}
-- Query 5
{
        ac: 'api',
        creation: d'2024-12-17T10:42:35.043162877Z',
        expiration: d'2024-12-27T10:42:35.043164533Z',
        grant: {
                id: 'RFilJMRp9lZi',
                key: 'surreal-bearer-RFilJMRp9lZi-OmflYxXwikDAvm8CNpsWYxd6'
        },
        id: 'RFilJMRp9lZi',
        revocation: NONE,
        subject: {
                record: user:2
        },
        type: 'bearer'
}
```


```surql
ACCESS api REVOKE WHERE subject.record.name = "tobie";
```

```surql title="Response"
[
        [
                {
                        ac: 'api',
                        creation: d'2024-12-17T10:42:35.040901759Z',
                        expiration: d'2024-12-27T10:42:35.040903414Z',
                        grant: {
                                id: 'mjSACes6sej4',
                                key: '[REDACTED]'
                        },
                        id: 'mjSACes6sej4',
                        revocation: d'2024-12-17T10:43:23.944198560Z',
                        subject: {
                                record: user:1
                        },
                        type: 'bearer'
                }
        ]
]
```

## `PURGE`

The `PURGE` clause completely removes grants created with a specific access method that have already been expired or revoked. In scenarios with very large amount of grants associated with an access method (e.g. when grants are automatically generated), purging inactive grants can improve the performance and experience of auditing grants with the `SHOW` clause. In some very high volume scenarios where grants are created by the hundreds of millions, purging may be necessary to limit the probability of collisions resulting in failure to create new grants. Note that the performance cost of validating a grant is independent from the number of existing grants, regardless of whether they are active or inactive.

Beware that any details associated with purged grants will be permanently lost and will no longer be available for auditing purposes. The `FOR` clause can be used to establish a minimum grace period after which expired or revoked grants should be purged. As with other security-sensitive operations related with the `ACCESS` statement, the purging of grants is logged in the SurrealDB server.

The clause will return the details of all grants that have successfully been purged and its performance will depend on the number of purged grants.

```syntax title="SurrealQL Syntax"
ACCESS @name [ ON [ ROOT | NAMESPACE | DATABASE ] ]
	PURGE [ EXPIRED | REVOKED [ , EXPIRED | REVOKED ] ] [ FOR @duration ]
]
```

#### Example: Purging grants that have been expired

```surql
/**[test]

[[test.results]]
value = "[]"

*/

ACCESS api PURGE EXPIRED;
```

#### Example: Purging grants that have been revoked for more than 90 days

```surql
/**[test]

[[test.results]]
value = "[]"

*/

ACCESS api PURGE REVOKED FOR 90d;
```

#### Example: Purging all grants that have been invalid for more than a year

```surql
/**[test]

[[test.results]]
value = "[]"

*/

ACCESS api PURGE EXPIRED, REVOKED FOR 1y;
```

## Improving error output

To improve the output of an error message inside an `ACCESS` statement, a [`THROW`](/docs/surrealql/statements/throw) statement can be used.

Follow these steps to see the output in practice.

First, start the SurrealDB server with a root user:

```bash
surreal start --user root --pass secret
```

Log in as the root user, choose the namespace `test` and database `test`, and run the following `DEFINE ACCESS` command.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE ACCESS account ON DATABASE TYPE RECORD
    SIGNUP ({
    IF $email = "me@me.com" {
        THROW "That's my email!!!"
    } ELSE {
        CREATE user SET email = $email, pass = crypto::argon2::generate($pass)}   
    })
    SIGNIN ( SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass) )
    DURATION FOR TOKEN 15m, FOR SESSION 12h
;
```

As the `THROW` statement checks for the email address `me@me.com`, this can be tested using CURL at the [`signup`](/docs/surrealdb/integration/http#signup) endpoint via the following command.

```bash
curl -X POST -H "Accept: application/json" -d '{"ns":"test","db":"test","ac":"account", "email": "me@me.com", "pass": "strongpassword"}' http://localhost:8000/signup
```

The error output shows the user-defined error message `That's my email!!!`.

```bash title="Output"
{"code":400,"details":"Request problems detected","description":"There is a problem with your request. Refer to the documentation for further information.","information":"There was a problem with the database: An error occurred: That's my email!!!"
```

In all other cases, the signup process will work, returning a token.

```bash
curl -X POST -H "Accept: application/json" -d '{"ns":"test","db":"test","ac":"account", "email": "someotheremail@me.com", "pass": "strongpassword"}' http://localhost:8000/signup
```

```bash title="Output"
{"code":200,"details":"Authentication succeeded","token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE3NDYwNzIxOTAsIm5iZiI6MTc0NjA3MjE5MCwiZXhwIjoxNzQ2MDczMDkwLCJpc3MiOiJTdXJyZWFsREIiLCJqdGkiOiI3YTc0ZjQ5ZS02OWMxLTRiMjMtYmRhNy05YThkNTNjNWFiZmIiLCJOUyI6InRlc3QiLCJEQiI6InRlc3QiLCJBQyI6ImFjY291bnQiLCJJRCI6InVzZXI6b2tlZjN5bmI4eXJkd2l3b3h1YjEifQ.4t1xwVkl36PeTFuBj0d41D6A-bwCx7LoNoLlj6yokA8wOKwEa1ldjBzTldZEmylLgP5-q8D4wp6f9Y6ntBZyPA","refresh":null}
```

## begin.mdx

---
sidebar_position: 2
sidebar_label: BEGIN
title: BEGIN statement | SurrealQL
description: The BEGIN statement starts a single transaction in which run multiple statements can be run, either succeeding as a whole, or failing.
---

# `BEGIN` statement

Each statement within SurrealDB is run within its own transaction by default. The `BEGIN` statement can be used to modify this behaviour by running a group of statements inside a single transaction, either succeeding as a whole, or failing. Once all of the statements within a transaction succeed, then all of the data modifications can be made permanent by finalizing the transaction with a [COMMIT](/docs/surrealql/statements/commit) statement at the end. If any statement within a transaction encounters an error or the transaction is manually cancelled ([CANCEL](/docs/surrealql/statements/cancel)), then any data modification made within the transaction is rolled back, and will not become a permanent part of the database.

### Statement syntax

import RailroadDiagram from "@components/RailroadDiagram.astro";
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

<Tabs syncKey="begin-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
BEGIN [ TRANSACTION ];
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const beginAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "BEGIN" },
        { type: "Optional", child: { type: "Terminal", text: "TRANSACTION" } },
        { type: "Terminal", text: ";" },
      ],
    },
  ],
};

<RailroadDiagram ast={beginAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following query shows example usage of this statement.

```surql
/**[test]

[[test.results]]
value = "[{ balance: 135605.16f, id: account:one }]"

[[test.results]]
value = "[{ balance: 91031.31f, id: account:two }]"

[[test.results]]
value = "[{ balance: 135905.16f, id: account:one }]"

[[test.results]]
value = "[{ balance: 90731.31f, id: account:two }]"

*/

-- Start a new database transaction. Transactions are a way to ensure multiple operations
-- either all succeed or all fail, maintaining data integrity.
BEGIN TRANSACTION;

-- Create a new account with the ID 'one' and set its initial balance to 135605.16
CREATE account:one SET balance = 135605.16;

-- Create another new account with the ID 'two' and set its initial balance to 91031.31
CREATE account:two SET balance = 91031.31;

-- Update the balance of account 'one' by adding 300.00 to the current balance.
-- This could represent a deposit or other form of credit on the balance property.
UPDATE account:one SET balance += 300.00;

-- Update the balance of account 'two' by subtracting 300.00 from the current balance.
-- This could represent a withdrawal or other form of debit on the balance property.
UPDATE account:two SET balance -= 300.00;

-- Finalize the transaction. This will apply the changes to the database. If there was an error
-- during any of the previous steps within the transaction, all changes would be rolled back and
-- the database would remain in its initial state.
COMMIT TRANSACTION;
```

## Returning early from a transaction

While all transactions require a final `COMMIT` or `CANCEL` statement in order to run, an early return can take place via the following:

* An error inside one of the statements inside the transaction,
* A `THROW` statement to return early with an error,
* A `RETURN` statement to return early. This is often used to customize the output of a transaction.

An example of the above:

```surql
/**[test]

[[test.results]]
value = "'Money sent! Status:
{ balance: 135905.16f, id: account:one }
{ balance: 90731.31f, id: account:two, wants_to_send_money: true }'"

[[test.results]]
value = "{ balance: 135905.16f, id: account:one }, { balance: 90731.31f, id: account:two, wants_to_send_money: true }"

*/

BEGIN;

CREATE account:one SET balance = 135605.16;
CREATE account:two SET balance = 91031.31, wants_to_send_money = true;

IF !account:two.wants_to_send_money {
    THROW "Customer doesn't want to send any money!";
};

LET $first = UPDATE ONLY account:one SET balance += 300.00;
LET $second = UPDATE ONLY account:two SET balance -= 300.00;

RETURN "Money sent! Status:\n" + <string>$first + '\n' + <string>$second;

COMMIT;
```

```surql title="Output"
'Money sent! Status:
{ balance: 135905.16f, id: account:one }
{ balance: 90731.31f, id: account:two, wants_to_send_money: true }'
```

## break.mdx

---
sidebar_position: 3
sidebar_label: BREAK
title: BREAK statement | SurrealQL
description: The BREAK statement can be used to break out of a loop.
---

# `BREAK` statement

The BREAK statement can be used to break out of a loop, such as inside one created by the [FOR statement](/docs/surrealql/statements/for).

import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

### Statement syntax

<Tabs syncKey="break-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
BREAK
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const breakAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "BREAK" }
    ]}
  ]
};

<RailroadDiagram ast={breakAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following queries shows example usage of this statement.

Creating a person for everyone in the array where the number is less than or equal to 5:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

LET $numbers = [1,2,3,4,5,6,7,8,9];

FOR $num IN $numbers {
    IF $num > 5 {
        BREAK;

    } ELSE IF $num < 5 {
        CREATE type::record(
            'person', $num
        ) CONTENT {
            name: "Person number " + <string>$num
        };
    };
};
```

Breaking out of a loop once unwanted data is encountered:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Data retrieved from somewhere which contains many NONE values
LET $weather = [
	{
		city: 'London',
		temperature: 22.2,
		timestamp: 1722565566389
	},
	NONE,
	{
		city: 'London',
		temperature: 20.1,
		timestamp: 1722652002699
	},
    {
        city: 'Phoenix',
        temperature: 45.1,
        timestamp: 1722565642160
    },
    NONE,
    NONE,
    {
        city: 'Phoenix',
        temperature: 45.1,
        timestamp: 1722652070372
    },
];

-- Sort the data to move the NONE values to the end
-- and break once the first NONE is reached
FOR $data IN array::sort::desc($weather) {
    IF $data IS NONE {
        BREAK;
    } ELSE {
        CREATE weather CONTENT $data;
    };
};
```


## cancel.mdx

---
sidebar_position: 4
sidebar_label: CANCEL
title: CANCEL statement | SurrealQL
description: The CANCEL statement can be used to cancel the statements within a transaction, reverting or rolling back any data modification made within the transaction as a whole.
---

# `CANCEL` statement

Each statement within SurrealDB is run within its own transaction. If a set of changes need to be made together, then groups of statements can be run together as a single transaction, either succeeding as a whole, or failing without leaving any residual data modifications. While a transaction will fail if any of its statements encounters an error, the `CANCEL` statement can also be used to cancel a transaction manually.

import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

### Statement syntax

<Tabs syncKey="cancel-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
CANCEL [ TRANSACTION ];
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const cancelAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "CANCEL" },
      { type: "Optional", child: { type: "Terminal", text: "TRANSACTION" } },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={cancelAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following query shows example usage of this statement.

```surql
/**[test]

[[test.results]]
error = "'The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'The query was not executed due to a cancelled transaction'"

*/

BEGIN TRANSACTION;

-- Setup accounts
CREATE account:one SET balance = 135605.16;
CREATE account:two SET balance = 91031.31;

-- Move money
UPDATE account:one SET balance += 300.00;
UPDATE account:two SET balance -= 300.00;

-- Rollback all changes
CANCEL TRANSACTION;
```

`CANCEL` is not used to automatically cancel a transaction based on a condition such as inside an [IF..ELSE](/docs/surrealql/statements/ifelse) block. Instead, a [THROW](/docs/surrealql/statements/throw) statement is used. THROW can be followed by any value, usually a string containing context behind the error.

```surql
/**[test]

[[test.results]]
error = "'The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'An error occurred: Not enough funds'"

*/

BEGIN TRANSACTION;

-- Setup accounts
CREATE account:one SET balance = 135605.16;
CREATE account:two SET balance = 200.31;

-- Move money
UPDATE account:one SET balance += 300.00;
UPDATE account:two SET balance -= 300.00;

IF account:two.balance < 0 {
    THROW "Not enough funds";
};

COMMIT TRANSACTION;
```


## commit.mdx

---
sidebar_position: 4
sidebar_label: COMMIT
title: COMMIT statement | SurrealQL
description: The COMMIT statement is used to commit a set of statements within a transaction, ensuring that all data modifications become a permanent part of the database.
---

# `COMMIT` statement

Each statement within SurrealDB is run within its own transaction by default. If a set of changes need to be made together, then groups of statements can be run together as a single transaction, either succeeding as a whole, or failing without leaving any residual data modifications. A `COMMIT` statement is used at the end of such a transaction to make the data modifications a permanent part of the database.

### Statement syntax

import RailroadDiagram from "@components/RailroadDiagram.astro";
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

<Tabs syncKey="commit-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
COMMIT [ TRANSACTION ];
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const commitAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "COMMIT" },
        { type: "Optional", child: { type: "Terminal", text: "TRANSACTION" } },
        { type: "Terminal", text: ";" },
      ],
    },
  ],
};

<RailroadDiagram ast={commitAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following query shows example usage of this statement.

```surql
/**[test]

[[test.results]]
value = "[{ balance: 135605.16f, id: account:one }]"

[[test.results]]
value = "[{ balance: 91031.31f, id: account:two }]"

[[test.results]]
value = "[{ balance: 135905.16f, id: account:one }]"

[[test.results]]
value = "[{ balance: 90731.31f, id: account:two }]"

*/

BEGIN TRANSACTION;

-- Setup accounts
CREATE account:one SET balance = 135605.16;
CREATE account:two SET balance = 91031.31;

-- Move money
UPDATE account:one SET balance += 300.00;
UPDATE account:two SET balance -= 300.00;

-- Finalise all changes
COMMIT TRANSACTION;
```

The following two options can be used at any point if a transaction must be cancelled without commiting the changes:

* [CANCEL](/docs/surrealql/statements/cancel) to manually cancel the transaction.
* [THROW](/docs/surrealql/statements/throw) to cancel a transaction with an optional error message. THROW is the only way to cancel a transaction based on a condition, such as inside an [IF..ELSE](/docs/surrealql/statements/ifelse) block.

In addition, a `RETURN` statement can be used to return early from a successful transaction. This is often used in order to return a customized output.

```surql
/**[test]

[[test.results]]
value = "'Money sent! Status:
{ balance: 135905.16f, id: account:one }
{ balance: 90731.31f, id: account:two, wants_to_send_money: true }'"

[[test.results]]
value = "{ balance: 135905.16f, id: account:one }"

[[test.results]]
value = "{ balance: 90731.31f, id: account:two, wants_to_send_money: true }"

*/

BEGIN;

CREATE account:one SET balance = 135605.16;
CREATE account:two SET balance = 91031.31, wants_to_send_money = true;

IF !account:two.wants_to_send_money {
    THROW "Customer doesn't want to send any money!";
};

LET $first = UPDATE ONLY account:one SET balance += 300.00;
LET $second = UPDATE ONLY account:two SET balance -= 300.00;

RETURN "Money sent! Status:\n" + <string>$first + '\n' + <string>$second;

COMMIT;
```

```surql title="Output"
'Money sent! Status:
{ balance: 135905.16f, id: account:one }
{ balance: 90731.31f, id: account:two, wants_to_send_money: true }'
```

## continue.mdx

---
sidebar_position: 6
sidebar_label: CONTINUE
title: CONTINUE statement | SurrealQL
description: The CONTINUE statement can be used to skip an iteration of a loop, like within the FOR statement
---

# `CONTINUE` statement

The CONTINUE statement can be used to skip an iteration of a loop, like within the [FOR statement](/docs/surrealql/statements/for).

import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

### Statement syntax

<Tabs syncKey="continue-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
CONTINUE
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const continueAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "CONTINUE" }
    ]}
  ]
};

<RailroadDiagram ast={continueAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following queries shows example usage of this statement.

Skipping an iteration of a loop unless a certain condition is met:

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Set can_vote to true for every person over 18 years old.
FOR $person IN (SELECT id, age FROM person) {
	IF ($person.age < 18) {
		CONTINUE;
	};

	UPDATE $person.id SET can_vote = true;
};
```

Skipping an iteration of a loop when bad data is encountered:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Data retrieved from somewhere which contains many NONE values
LET $weather = [
	{
		city: 'London',
		temperature: 22.2,
		timestamp: 1722565566389
	},
	NONE,
	{
		city: 'London',
		temperature: 20.1,
		timestamp: 1722652002699
	},
    {
        city: 'Phoenix',
        temperature: 45.1,
        timestamp: 1722565642160
    },
    NONE,
    NONE,
    {
        city: 'Phoenix',
        temperature: 45.1,
        timestamp: 1722652070372
    },
];

FOR $data IN $weather {
    IF $data IS NONE {
        CONTINUE;
    };

	CREATE weather CONTENT $data;
};
```


## create.mdx

---
sidebar_position: 7
sidebar_label: CREATE
title: CREATE statement | SurrealQL
description: The CREATE statement can be used to add a record to the database if it does not already exist.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `CREATE` statement

The `CREATE` statement can be used to add a record to the database. If the record already exists, the statement will give an error.

> [!NOTE]
> This statement can not be used to create graph relationships. For that, use the [`RELATE`](/docs/surrealql/statements/relate) statement.

### Statement syntax

<Tabs syncKey="create-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
CREATE [ ONLY ] @targets
	[ CONTENT @value
	  | SET @field = @value ...
	]
	[ RETURN NONE | RETURN BEFORE | RETURN AFTER | RETURN DIFF | RETURN @statement_param, ... | RETURN VALUE @statement_param ]
	[ TIMEOUT @duration ]
;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const createAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "CREATE" },
        { type: "Optional", child: { type: "Terminal", text: "ONLY" } },
        { type: "NonTerminal", text: "@targets" },
        { type: "Optional", child: { type: "Choice", index: 1, children: [
          { type: "Sequence", children: [ { type: "Terminal", text: "CONTENT" }, { type: "NonTerminal", text: "@value" } ] },
          { type: "Sequence", children: [ { type: "Terminal", text: "SET" }, { type: "NonTerminal", text: "@field = @value ..." } ] }
        ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "RETURN" }, { type: "Choice", index: 1, children: [
          { type: "Terminal", text: "NONE" },
          { type: "Terminal", text: "BEFORE" },
          { type: "Terminal", text: "AFTER" },
          { type: "Terminal", text: "DIFF" },
          { type: "NonTerminal", text: "@statement_param, ..." },
          { type: "Sequence", children: [ { type: "Terminal", text: "VALUE" }, { type: "NonTerminal", text: "@statement_param" } ] }
        ] } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TIMEOUT" }, { type: "NonTerminal", text: "@duration" } ] } },
        { type: "Terminal", text: ";" },
      ],
    },
  ],
};

<RailroadDiagram ast={createAst} className="my-6" />

  </TabItem>
</Tabs>

## Creating a Table Record

`CREATE` can be used with just a table name, in which case its ID will be generated randomly.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:ht9tbw226vijmg2uf72c }]"
skip-record-id-key = true

*/

-- Create a new record
CREATE person;
```

```surql title="Response"
[
    {
        "id": "person:2vvgzt6m24s952yiy7x8"
    }
]
```

To specify a specific ID for a table instead, use `:` followed by a value.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:one }]"

*/

CREATE person:one;
```

```surql title="Response"
[
	{
		id: person:one
	}
]
```

The table name and ID together form the full [record ID](/docs/surrealql/datamodel/ids) which can be used to query the created data or by using the [`SELECT`](/docs/surrealql/statements/select) statement. See the [record ID](/docs/surrealql/datamodel/ids) page to learn more about what counts as a valid record identifier.

The default random ID can be generated in different ways (such as a ULID) using the [built-in ID generation functions](/docs/surrealql/datamodel/ids#generating-record-ids).

It is also possible to specify the ID of the record you want to create using a string or any of the supported formats for [record IDs](/docs/surrealql/datamodel/ids).

```surql
-- Use the type::record() function to provide a record's table and id separately
CREATE type::record("person", "one");
```

## Adding Record Data

When creating a record, you can specify the record data using the `SET` clause, or the `CONTENT` clause. The `SET` clause is used to specify record data one field at a time, while the `CONTENT` clause is used to specify record data using a SurrealQL object. The `CONTENT` clause is useful when the record data is already in the form of a SurrealQL or JSON object.

```surql
/**[test]

[[test.results]]
value = "[{ company: 'SurrealDB', id: person:tobie, name: 'Tobie', skills: ['Rust', 'Go', 'JavaScript'] }]"

*/

-- Create a new record with a text id
CREATE person:tobie SET
    name = 'Tobie',
    company = 'SurrealDB',
    skills = ['Rust', 'Go', 'JavaScript'];
```

The above will create a new record with the ID `person:tobie` and the specified data.

```surql title="Response"
[
	{
		"id": "person:tobie",
		"name": "Tobie",
		"company": "SurrealDB",
		"skills": ["Rust", "Go", "JavaScript"]
	}
]
```

Specifing record data using the `CONTENT` keyword:

```surql
/**[test]

[[test.results]]
value = "[{ company: 'SurrealDB', id: person:100, name: 'Tobie', skills: ['Rust', 'Go', 'JavaScript'] }]"

*/

-- Create a new record with a numeric id
CREATE person:100 CONTENT {
	name: 'Tobie',
	company: 'SurrealDB',
	skills: ['Rust', 'Go', 'JavaScript'],
};
```

## Options and clauses

### Creating multiple records

Multiple records or even multiple record types can be created by separating table names by commas.

```surql
/**[test]

[[test.results]]
value = "[{ created_at: d'2025-10-08T02:18:52.928229Z', id: townsperson:0jkw66fganx0eqdkimgd, name: 'Just a townsperson' }, { created_at: d'2025-10-08T02:18:52.932072Z', id: cat:yw43e5ycrdq8mpzx7zq3, name: 'Just a cat' }, { created_at: d'2025-10-08T02:18:52.934887Z', id: dog:z4yq39x2hj79jl43v57x, name: 'Just a dog' }]"
skip-datetime = true 
skip-record-id-key = true

*/

-- Note: record::tb(id) returns just the table name portion of a record ID
CREATE townsperson, cat, dog SET
    created_at = time::now(),
    name = "Just a " + record::tb(id);
```

```surql title="Response"
[
    {
        "created_at": "2024-03-19T03:12:05.079Z",
        "id": "townsperson:p37ha2lngckp3v8tvf2j",
        "name": "Just a townsperson"
    },
    {
        "created_at": "2024-03-19T03:12:05.080Z",
        "id": "cat:p1pwbjaq96nhhnuohjtc",
        "name": "Just a cat"
    },
    {
        "created_at": "2024-03-19T03:12:05.080Z",
        "id": "dog:01vcxgdpuctdk354hzkp",
        "name": "Just a dog"
    }
]
```

The `| |` syntax is another way to create multiple records in a single execution. This syntax can be used in two ways.

One is by including a table name, a `:` (a colon), and then a number. This will create a quantity of records equal to the number after the table name. The records created will have random IDs.

```surql
/**[test]

[[test.results]]
value = "[{ id: townsperson:ddt3gkla0ve6yt0vwunr }, { id: townsperson:cj902h7hbsn8evg7fq5d }, { id: townsperson:01n15yxzw0vl42u008a4 }]"
skip-record-id-key = true

*/

-- Creates three townperson records with a random ID
CREATE |townsperson:3|;
```

```surql title="Response"
[
	{
		id: townsperson:hzkt0piy3f72xo5dl2jf
	},
	{
		id: townsperson:k0mujrohm8qe2txz5pnz
	},
	{
		id: townsperson:pwumqelrsi1qt0jmihwh
	}
]
```

The other method is by using the `..` range syntax after the `:` instead of a single number. This will create records with specific IDs that span across the range indicated.

```surql
-- Note: 1..4 used to be inclusive until SurrealDB 3.0.0
-- Now creates 1 up to but not including 4
CREATE |townsperson:1..4|;
```

```surql title="Response"
[
	{
		id: townsperson:1
	},
	{
		id: townsperson:2
	},
	{
		id: townsperson:3
	}
]
```

All of these methods can be combined to create multiple records at the same time.

```surql
CREATE dog, |cat:2|, |townsperson:1..3| SET
    created_at = time::now(),
    name = "Just a " + record::tb(id);
```

```surql title="Response"
[
	{
		created_at: '2024-08-13T04:14:44.135Z',
		id: dog:u3fzmqvg3yq9mo3o6z2s,
		name: 'Just a dog'
	},
	{
		created_at: '2024-08-13T04:14:44.137Z',
		id: cat:n6x3caiiazucslfs7rpm,
		name: 'Just a cat'
	},
	{
		created_at: '2024-08-13T04:14:44.137Z',
		id: cat:rnvhxgjhsbea5u58s0wu,
		name: 'Just a cat'
	},
	{
		created_at: '2024-08-13T04:14:44.137Z',
		id: townsperson:1,
		name: 'Just a townsperson'
	},
	{
		created_at: '2024-08-13T04:14:44.137Z',
		id: townsperson:2,
		name: 'Just a townsperson'
	},
	{
		created_at: '2024-08-13T04:14:44.137Z',
		id: townsperson:3,
		name: 'Just a townsperson'
	}
]
```

### ONLY

When creating a single record, the `ONLY` clause can be used to return the record object on its own instead of inside an array.

```surql
-- Returns an array with a single record inside
CREATE person:tobie SET
    name = 'Tobie',
    company = 'SurrealDB',
    skills = ['Rust', 'Go', 'JavaScript'];

-- Returns just a single record
CREATE ONLY person:tobie SET
    name = 'Tobie',
    company = 'SurrealDB',
    skills = ['Rust', 'Go', 'JavaScript'];
```

```surql title="Response"
-------- Query --------

[
	{
		company: 'SurrealDB',
		id: person:tobie,
		name: 'Tobie',
		skills: [
			'Rust',
			'Go',
			'JavaScript'
		]
	}
]

-------- Query --------

{
	company: 'SurrealDB',
	id: person:tobieagain,
	name: 'Tobie',
	skills: [
		'Rust',
		'Go',
		'JavaScript'
	]
}
```

### Return Values

By default, the create statement returns the record once it has been created. To change what is returned, we can use the `RETURN` clause, specifying either `NONE`, `BEFORE`, `AFTER`, `DIFF`, or a comma-separated list of specific fields to return.

`RETURN NONE` can be useful to avoid excess output:

```surql
-- Create 10000 records but don't show any of them
CREATE |person:10000| SET age = 46, username = "john-smith" RETURN NONE;
```

`RETURN DIFF` returns the changeset diff:

```surql
/**[test]

[[test.results]]
value = "[[{ op: 'replace', path: '', value: { age: 46, id: person:2wm77z4dn1wjh7az4r5f, username: 'john-smith' } }]]"

*/

CREATE person SET age = 46, username = "john-smith" RETURN DIFF;
```

```surql title="Response"
[
	[
		{
			op: 'replace',
			path: '/',
			value: {
				age: 46,
				id: person:h84x4k5kh2m6cjf1vvza,
				username: 'john-smith'
			}
		}
	]
]
```

`RETURN BEFORE` inside a `CREATE` statement is essentially a synonym for `RETURN NONE`, while `RETURN AFTER` is the default behaviour for create.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Will always return NONE
CREATE person SET age = 46, username = "john-smith" RETURN BEFORE;
```

```surql
/**[test]

[[test.results]]
value = "[{ age: 46, id: person:ncnnp25vhiqddlfvz5ve, username: 'john-smith' }]"
skip-record-id-key = true

*/

-- Return the record after creation
CREATE person SET age = 46, username = "john-smith" RETURN AFTER;
```

You can also return specific fields from a created record, the value of a single field using `VALUE`, as well as ad-hoc fields to modify the output as needed.

```surql
/**[test]

[[test.results]]
value = "[{ age: 46, age_next_year: 47, interests: ['skiing', 'music'] }]"

[[test.results]]
value = "[20, 20, 20, 20, 20]"

*/

CREATE person
    SET age = 46,
    username = "john-smith",
    interests = ['skiing', 'music']
RETURN
    age,
    interests,
    age + 1 AS age_next_year;

CREATE |person:5|
    SET age = 20
RETURN VALUE age;
```

```surql title="Response"
-------- Query --------

[
	{
		age: 46,
		age_next_year: 47,
		interests: [
			'skiing',
			'music'
		]
	}
]

-------- Query --------

[
	20,
	20,
	20,
	20,
	20
]
```

### Timeout

The `TIMEOUT` clause can be used to specify the maximum time the statement should take to execute. This is useful when you want more control such as controlling compute costs or making sure queries succeed or fail within tight latency boundaries to not have a big query queue forming.

The value for `TIMEOUT` is specified in seconds or milliseconds.

```surql
-- Query attempting to create half a million `person` records
CREATE |person:500000| SET age = 46, username = "john-smith" TIMEOUT 500ms;
```

### `VERSION`

<Since v="v2.0.0" />

If you are using [SurrealKV as the storage engine](/docs/surrealdb/installation/running/surrealkv) with versioning enabled, when creating a record you can specify a version for each record. This is useful for time-travel queries. You can query a specific version of a record by using the `VERSION` clause.

The `VERSION` clause is always followed by a [datetime](/docs/surrealql/datamodel/datetimes) and when the specified timestamp does not exist, an empty array is returned.

> [!NOTE]
> The `VERSION` clause is currently in alpha and is subject to change. We do not recommend this for production.

```surql
-- Create a record for user:john at 8:00AM
CREATE user:john SET name = 'John' VERSION d'2024-08-19T08:00:00Z';
[[{ id: user:john, name: 'John' }]]

-- Return the record for user:john at 8:00AM
SELECT * FROM user:john VERSION d'2024-08-19T08:00:00Z';
[[{ id: user:john, name: 'John' }]]

-- Create a record for user:john at 8:01AM
CREATE user:john SET name = 'John-1' VERSION d'2024-08-19T08:01:00Z';
[[{ id: user:john, name: 'John-1' }]]

-- Return the record for user:john at 8:01AM
SELECT * FROM user:john VERSION d'2024-08-19T08:01:00Z';
[[{ id: user:john, name: 'John-1' }]]

-- Return an empty array because the record at the datetime does not exist
SELECT * FROM user:john VERSION d'2024-08-19T07:00:00Z';
[[]]
```

Another example of how `VERSION` works with `CREATE` is by creating records at different times and then querying for them at a specific point in time.

```surql
CREATE |user:10| VERSION d"2020-09-09";

[[{ id: user:rtbjoqv1xe9wnxjx5aro }, { id: user:tkik878q8uoddvuucu0a }, { id: user:rcnywgogvlipv3tb8qut }, { id: user:30ynx82x52ff77dxzv1i }, { id: user:59mxi0xosi3im5ccbx8l }, { id: user:nolu7yreqs4e5m7255oa }, { id: user:u384ycj1d2esi3yrasli }, { id: user:n4xnrq98ookevhmdd7d2 }, { id: user:5j5ujfu4dokcpdk51qa8 }, { id: user:jiqmlvrgafeorr50nvn9 }]]

CREATE |user:10| VERSION d"2020-09-10";
[[{ id: user:ze98ow4bzdcndzc5nlqj }, { id: user:gjqu2uh3wnp1cpjg1unt }, { id: user:17bxpjl4ptbxv9k2ghmt }, { id: user:fmqqeajf52neg4c7oaoq }, { id: user:bfn45ewsg86auvekeuz0 }, { id: user:834yq1tyatwopb4726mj }, { id: user:veehoua4cu65ff4wc8pf }, { id: user:y3az4pizc0ddpruixw6g }, { id: user:xrn6eqrtyqgg8cgpm9zp }, { id: user:s06acf74rsnvhvim3ys5 }]]

RETURN count(SELECT * FROM user VERSION d"2020-09-09"); -- returns 10
[10]

RETURN count(SELECT * FROM user); -- returns 21
[21]
```

<Since v="v2.1.0" />

The `VERSION` clause can also take a dynamic value or parameter that resolves to a datetime.

```surql
CREATE user:john SET name = 'John' VERSION time::now();

LET $now = time::now();
CREATE user:john_the_second SET name = 'John' VERSION $now;

DEFINE FUNCTION fn::yesterday() { time::now() - 1d };
CREATE user:john_the_third SET name = 'John' VERSION fn::yesterday();
```

## Implicit statement behaviour

While a number of definitions need to be in place for a `CREATE` statement to happen, SurrealDB will handle them automatically by default. This behaviour is best seen by starting a new database.

While a connection to SurrealDB via Surrealist or the [surreal sql](/docs/surrealdb/cli/sql) command can include a defined namespace and database, the namespace and database names do not exist upon creation. At this point, they are only held inside the pre-defined [$session](/docs/surrealql/parameters#session) parameter. This can be seen through the [INFO](/docs/surrealql/statements/info) statements, which will show no definitions at all inside a new database.

```surql
INFO FOR ROOT;
INFO FOR NS;
INFO FOR DB;
RETURN $session;
```

```surql title="Response"
-------- Query --------

{
	accesses: {},
	namespaces: {},
	nodes: {},
	system: {
		available_parallelism: 0,
		cpu_usage: 0,
		load_average: [
			0,
			0,
			0
		],
		memory_allocated: 0,
		memory_usage: 0,
		physical_cores: 0,
		threads: 0
	},
	users: {}
}

-------- Query --------

{
	accesses: {},
	analyzers: {},
	apis: {},
	configs: {},
	functions: {},
	models: {},
	params: {},
	tables: {},
	users: {}
}

-------- Query --------

{
	analyzers: {},
	functions: {},
	models: {},
	params: {},
	scopes: {},
	tables: {},
	tokens: {},
	users: {}
}

-------- Query --------

{
	ac: NONE,
	db: 'sandbox',
	exp: NONE,
	id: NONE,
	ip: NONE,
	ns: 'sandbox',
	or: NONE,
	rd: NONE,
	tk: NONE
}
```

This is to allow the chance to [define](/docs/surrealql/statements/define/database) them manually, such as by including a comment.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE DATABASE my_database COMMENT "Some important info that I prefer to add manually";
```

However, once the first record is created or inserted, SurrealDB will access the session data to execute a number of definition statements for the namespace, database, and then add a definition for the desired table name in order to allow the operation to proceed.

```surql
-- Three DEFINE statements will happen to allow this operation
CREATE person;

INFO FOR ROOT;
INFO FOR NS;
INFO FOR DB;
```

```surql title="Response"
-------- Query --------

{
	accesses: {},
	namespaces: {
		sandbox: 'DEFINE NAMESPACE sandbox'
	},
	nodes: {},
	system: {
		available_parallelism: 0,
		cpu_usage: 0,
		load_average: [
			0,
			0,
			0
		],
		memory_allocated: 0,
		memory_usage: 0,
		physical_cores: 0,
		threads: 0
	},
	users: {}
}

-------- Query --------

{
	accesses: {},
	databases: {
		sandbox: 'DEFINE DATABASE sandbox'
	},
	users: {}
}

-------- Query 7 --------

{
	accesses: {},
	analyzers: {},
	apis: {},
	configs: {},
	functions: {},
	models: {},
	params: {},
	tables: {
		person: 'DEFINE TABLE person TYPE ANY SCHEMALESS PERMISSIONS NONE'
	},
	users: {}
}
```

To disallow this behaviour, you can [define a database](/docs/surrealql/statements/define/database) using the `STRICT` keyword. In strict mode, any resource must first be explicitly defined before it can be used.

```surql
ns/db> CREATE person;
[[{ id: person:epgvviec00l4pnmhtt5n }]]

ns/db> DEFINE DB strict_db STRICT;
[NONE]

ns/db> USE DB strict_db;
[NONE]

ns/strict_db> CREATE person;
["Thrown error: The table 'person' does not exist"]

ns/strict_db> DEFINE TABLE person;
[NONE]

ns/strict_db> CREATE person;
[[{ id: person:c76lfw6n4yb1z2dj9xaj }]]
```

## Learn more

To learn more about SurrealDB, check out the following resources:
- [Getting started guide](/docs/surrealdb/introduction/start)
- [Select statement](/docs/surrealql/statements/select)
- [Update statement](/docs/surrealql/statements/update)
- [Insert statement](/docs/surrealql/statements/insert)
- [Delete statement](/docs/surrealql/statements/delete)
- [Relate statement](/docs/surrealql/statements/relate)


## delete.mdx

---
sidebar_position: 8
sidebar_label: DELETE
title: DELETE statement | SurrealQL
description: The DELETE statement can be used to delete records from the database.
---

# `DELETE` statement

The `DELETE` statement can be used to delete records from the database.

import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

### Statement syntax

<Tabs syncKey="delete-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DELETE [ FROM | ONLY ] @targets
	[ WHERE @condition ]
	[ RETURN NONE | RETURN BEFORE | RETURN AFTER | RETURN DIFF | RETURN @statement_param, ... ]
	[ TIMEOUT @duration ]
	[ EXPLAIN [ FULL ]]
;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const deleteAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DELETE" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "FROM" }, { type: "Terminal", text: "ONLY" } ] } },
      { type: "NonTerminal", text: "@targets" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "WHERE" }, { type: "NonTerminal", text: "@condition" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "RETURN" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NONE" }, { type: "Terminal", text: "BEFORE" }, { type: "Terminal", text: "AFTER" }, { type: "Terminal", text: "DIFF" }, { type: "NonTerminal", text: "@statement_param, ..." } ] } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TIMEOUT" }, { type: "NonTerminal", text: "@duration" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "EXPLAIN" }, { type: "Optional", child: { type: "Terminal", text: "FULL" } } ] } },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={deleteAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

### Basic usage

The following queries shows basic usage of the DELETE statement, which is used to delete records from a table or a graph edge.

Deleting records can be done in multiple ways.

Specifying only the table name will delete all the records from a table. Note that a `DELETE` statement returns nothing (i.e. an empty array) by default.

```surql
/**[test]

[[test.results]]
value = "[]"

*/

-- Delete all records from a table
DELETE person;
```

```surql title="Output"
[]
```

A `DELETE` statement on a specific ID will delete a single record.

```surql
/**[test]

[[test.results]]
value = "[]"

[[test.results]]
value = "[]"

*/

-- Delete a record with a specific numeric id
DELETE person:100;

-- Delete a record with a specific string id
DELETE person:tobie;
```

The `ONLY` keyword can be followed by a `RETURN BEFORE` clause to return just the object for the record in question before it was deleted.

```surql
/**[test]

[[test.results]]
value = "{ id: person:tobie }"

[[test.results]]
value = "{ id: person:tobie }"

*/

CREATE ONLY person:tobie;
DELETE ONLY person:tobie RETURN BEFORE;
-- { id: person:tobie }
```

Note that as a `DELETE` statement returns an empty array by default, and the `ONLY` keyword assumes that a single object will be returned, it will return an error if `RETURN BEFORE` is not included.

```surql
/**[test]

[[test.results]]
error = "'Expected a single result output when using the ONLY keyword'"

*/

DELETE ONLY person:tobie;
```

```surql title="Output"
'Expected a single result output when using the ONLY keyword'
```

### Deleting records based on conditions

The delete statement supports conditional matching of records using a `WHERE` clause. If the expression in the `WHERE` clause evaluates to true, then the respective record will be deleted.

```surql
/**[test]

[[test.results]]
value = "[]"

*/

-- Update all records which match the condition
DELETE city WHERE name = 'London';
```

By default, the delete statement does not return any data, returning only an empty array if the statement succeeds completely. Specify a `RETURN` clause to change the value which is returned for each document that is deleted.

```surql
/**[test]

[[test.results]]
value = "[]"

[[test.results]]
value = "[]"

[[test.results]]
value = "[]"

[[test.results]]
value = "[]"

*/

-- Don't return any result (the default)
DELETE user WHERE age < 18 RETURN NONE;

-- Return the changeset diff
DELETE user WHERE interests CONTAINS 'reading' RETURN DIFF;

-- Return the record before changes were applied
DELETE user WHERE interests CONTAINS 'reading' RETURN BEFORE;

-- Return the record after changes were applied
DELETE user WHERE interests CONTAINS 'reading' RETURN AFTER;
```

An important point to know when using a `WHERE` clause is that it performs a check on the [truthiness](/docs/surrealql/datamodel/values#values-and-truthiness) of a value, namely whether a value exists and is not a default value like 0, an empty string, empty array, and so on.

As such, the `DELETE` query below that only specifies `WHERE age` essentially evaluates to "WHERE age exists" and will delete every cat in the database with an `age`.

```surql
/**[test]

[[test.results]]
value = "[{ age: 4, id: cat:one }]"

[[test.results]]
value = "[{ id: cat:two }]"

[[test.results]]
value = "[]"

[[test.results]]
value = "[{ id: cat:two }]"

*/

CREATE cat:one SET age = 4;
CREATE cat:two;
DELETE cat WHERE age;
SELECT * FROM cat;
```

```surql title="Output"
[
	{
		id: cat:two
	}
]
```

This pattern is particularly useful when using SurrealDB's [literal types](/docs/surrealql/datamodel/literals). A literal type containing objects that contain a single top-level field can easily be matched on through the field name.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ error_info: { continue: { message: 'Continue' } }, id: information:y0wty8grwins5yxyswea }]"
skip-record-id-key = true

[[test.results]]
value = "[{ error_info: { continue: { message: 'Continue' } }, id: information:wbg4q0k3z97jdy7x5sns }]"
skip-record-id-key = true

[[test.results]]
value = "[{ error_info: { deprecated: { message: "We don't use this anymore" } }, id: information:r4pa3c49one2ofd2bcbw }]"
skip-record-id-key = true

[[test.results]]
value = "[]"

[[test.results]]
value = "[{ error_info: { deprecated: { message: "We don't use this anymore" } }, id: information:r4pa3c49one2ofd2bcbw }]"
skip-record-id-key = true

*/

DEFINE FIELD error_info ON TABLE information TYPE
      { continue: { message: "Continue" } }
    | { retry_with_id: { error: string  } }
    | { deprecated: { message: string   } };

CREATE information SET error_info = { continue: { message: "Continue" }};
CREATE information SET error_info = { continue: { message: "Continue" }};
CREATE information SET error_info = { deprecated: { message: "We don't use this anymore" }};

DELETE information WHERE error_info.continue;
SELECT * FROM information;
```

```surql title="Output"
[
	{
		error_info: {
			deprecated: {
				message: "We don't use this anymore"
			}
		},
		id: info:o0pmm7zos98iv03xliav
	}
]
```

### Using TIMEOUT duration records based on conditions
When processing a large result set with many interconnected records, it is possible to use the `TIMEOUT` keywords to specify a timeout duration for the statement. If the statement continues beyond this duration, then the transaction will fail, no records will be deleted from the database, and the statement will return an error.

```surql
/**[test]

[[test.results]]
value = "[]"

*/

DELETE person WHERE ->knows->person->(knows WHERE influencer = false) TIMEOUT 5s;
```

## Deleting graph edges

You can also delete graph edges between two records in the database by using the DELETE statement.

For example the graph edge below:

```surql
/**[test]

[[test.results]]
value = "[{ id: bought:ruo8s16ib6j6hoker96z, in: person:tobie, out: product:iphone }]"
skip-record-id-key = true

*/

RELATE person:tobie->bought->product:iphone;

[
	{
		"id": bought:ctwsll49k37a7rmqz9rr,
		"in": person:tobie,
		"out": product:iphone
	}
]
```

Can be deleted by:

```surql
/**[test]

[[test.results]]
value = "[]"

*/

DELETE person:tobie->bought WHERE out=product:iphone;
```

## Soft deletions

While soft deletions do not exist natively in SurrealDB, they can be simulated by [defining an event](/docs/surrealql/statements/define/event) that reacts whenever a deletion occurs.

The following example archives the data of a deleted record in another table. This can be combined with  [fewer permissions for the new table](/docs/surrealql/statements/define/table#defining-permissions) so that it can be accessed only by [system users](/docs/surrealql/statements/define/user) and not [record users](/docs/surrealql/statements/define/access/record).

```surql
DEFINE EVENT archive_person ON TABLE person WHEN $event = "DELETE" THEN {
    CREATE deleted_person SET
        data = $before,
        deleted_at = time::now()
};

CREATE |person:1..5|;
DELETE person:1;

-- Only two `person` records left
SELECT * FROM person;
-- But the data of `person:1` is still here
SELECT * FROM deleted_person;
```

```surql title="Output"

-------- Query --------

[
	{
		id: person:2
	},
	{
		id: person:3
	}
]

-------- Query --------

[
	{
		data: {
			id: person:1
		},
		deleted_at: d'2024-09-12T00:46:59.176Z',
		id: deleted_person:p3fpzhpxuu9jvjn8juyf
	}
]
```

## The `EXPLAIN` clause

When `EXPLAIN` is used:

1. The `DELETE` statement returns an explanation, essentially revealing the execution plan to provide transparency and understanding of the query performance.
2. The records are not deleted.

`EXPLAIN` can be followed by `FULL` to see the number of executed rows.

## explain.mdx

---
sidebar_position: 9
sidebar_label: EXPLAIN
title: EXPLAIN statement | SurrealQL
description: The REBUILD statement is used to rebuild resources.
---

import Since from '@components/shared/Since.astro'
import SurrealistMini from "@components/SurrealistMini.astro"
import RailroadDiagram from "@components/RailroadDiagram.astro"
import Tabs from "@components/Tabs/Tabs.astro"
import TabItem from "@components/Tabs/TabItem.astro"

# `EXPLAIN` statement

<Since v="v3.0.0" />

> [!NOTE]
> The output for the `EXPLAIN` statement is for informational purposes and subject to change. Be sure not to develop tools around it that rely on a single predictible output.

The `EXPLAIN` statement is used to display the query planner for a statement.

### Statement syntax

<Tabs syncKey="rebuild-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
EXPLAIN [ ANALYZE ] [ FORMAT TEXT | JSON ] @statement
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const explainAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "EXPLAIN" },
      { type: "Optional", child: { type: "Terminal", text: "ANALYZE" } },
      { type: "Optional", child: {
        type: "Sequence", children: [
          { type: "Terminal", text: "FORMAT" },
          { type: "Choice", index: 0, children: [
            { type: "Terminal", text: "TEXT" },
            { type: "Terminal", text: "JSON" }
          ]}
        ]
      }},
      { type: "NonTerminal", text: "@statement" }
    ]}
  ]
};


<RailroadDiagram ast={explainAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

An `EXPLAIN` statement is one that can be appended to another statement that does not modify database resources, i.e. a `SELECT` statement or another statement that returns a value.

The two main decisions to make when using an `EXPLAIN` statement are:

* Use default text format or add `FORMAT JSON` to output the format in JSON?
* Add the `ANALYZE` clause after `EXPLAIN` 

The following example shows the four possible types of output when followed by a simple string.

```surql
EXPLAIN "yourself!";
EXPLAIN ANALYZE "yourself!";
EXPLAIN FORMAT JSON "yourself!";
EXPLAIN ANALYZE FORMAT JSON "yourself!";
```

As the output shows, the `ANALYZE` clause adds information on the metrics and total rows.

```surql title="Output"
-------- Query --------

"Expr [ctx: Rt] [expr: 'yourself!']"

-------- Query --------

"Expr [ctx: Rt] [expr: 'yourself!'] {rows: 0, batches: 0, elapsed: 0ns}

Total rows: 1"

-------- Query --------

{
	attributes: {
		expr: "'yourself!'"
	},
	context: 'Rt',
	expressions: [
		{
			role: 'expr',
			sql: "'yourself!'"
		}
	],
	operator: 'Expr'
}

-------- Query --------

{
	attributes: {
		expr: "'yourself!'"
	},
	context: 'Rt',
	expressions: [
		{
			role: 'expr',
			sql: "'yourself!'"
		}
	],
	metrics: {
		elapsed_ns: 0,
		output_batches: 0,
		output_rows: 0
	},
	operator: 'Expr',
	total_rows: 1
}
```

### Context

The `context` field in an `EXPLAIN` statement refers to the minimum context level for an operation: `Rt` (root), `Ns` (namespace), or `Db` (database) level.

### Operator types

The `operator` field in the output of an `EXPLAIN` statement is the most relevant area to take note of. Here is a list of many of the operator types you will see in the statement output.

```text
Aggregate
Compute
CountScan
Explain
ExplainAnalyze
Expr
Fetch
Filter
Foreach
FullTextScan
GraphEdgeScan
IfElse
IndexCountScan
IndexScan
Let
Limit
ProjectValue
Project
SelectProject
ReferenceScan
Return
Scan
Sequence
Sleep
SourceExpr
Split
Union
UnwrapExactlyOne
InfoDatabase
InfoIndex
InfoNamespace
InfoRoot
InfoTable
InfoUser
ExternalSort
Sort
SortByKey
RandomShuffle
SortTopK
SortTopKByKey
```

This allows you to get an insight into exactly what sort of work is being performed by the database when a query is executed.

For example, take the following simple example in which one `person` record has a single friend. The final two queries return the same result, but one is a `SELECT...FROM ONLY` query while the other is a direct destructuring of the link from its record id.

```surql
CREATE person:one, person:two;
RELATE person:one->friend->person:two;

EXPLAIN SELECT ->friend->person AS friends FROM ONLY person:one;
EXPLAIN person:one.{ friends: ->friend->person };
```

Not only is the second query faster, but we can see why as the first query is doing more work with four operations instead of one.

```surql title="Output"
-------- Query 1 (143us) --------

'UnwrapExactlyOne [ctx: Db]
    Project [ctx: Db]
          field.lookup: GraphEdgeScan [ctx: Db] [direction: ->, tables: person, output: TargetId]
                    GraphEdgeScan [ctx: Db] [direction: ->, tables: friend, output: TargetId]
                                  CurrentValueSource [ctx: Rt]
                                          RecordIdScan [ctx: Db] [record_id: person:one]'

-------- Query 2 (44us) --------

'Expr [ctx: Db] [expr: (person:one).{ friends: ->friend->person }]'
```

Here is an example of output for a query of a complexity more similar to those seen in production applications.

```surql
EXPLAIN ANALYZE SELECT
  id as commentId,
  in.id as id,
  in.creationDate as creationDate
FROM is_comment_of
WHERE out = media_text_test:0 AND in.creationDate < d'2026-01-09T00:00:00.000Z'
ORDER BY in.creationDate DESC
LIMIT 2;
```

```surql title="Output"
"Project [ctx: Db] {rows: 0, batches: 0, elapsed: 1.71µs}
    Limit [ctx: Db] [limit: 2] {rows: 0, batches: 0, elapsed: 13.92µs}
            SortTopKByKey [ctx: Db] [sort_keys: in.creationDate DESC, limit: 2] {rows: 0, batches: 0, elapsed: 7.50µs}
                        TableScan [ctx: Db] [table: is_comment_of, direction: Forward, predicate: out = media_text_test:0 AND in.creationDate < d'2026-01-09T00:00:00Z'] {rows: 0, batches: 0, elapsed: 361.42µs}
                        
                        Total rows: 0"
```

## for.mdx

---
sidebar_position: 10
sidebar_label: FOR
title: FOR statement | SurrealQL
description: The FOR statement creates a loop that iterates over the values of an array.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `FOR` statement

The `FOR` statement can be used to iterate over the values of an array, and to perform certain actions with those values.

> [!NOTE]
> A `FOR` loop currently cannot modify items outside its own scope, such as variables declared before the loop.


<Tabs syncKey="for-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
FOR @item IN @iterable {
@block
};
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const forAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "FOR" },
      { type: "NonTerminal", text: "@item" },
      { type: "Terminal", text: "IN" },
      { type: "NonTerminal", text: "@iterable" },
      { type: "Terminal", text: "{" },
      { type: "NonTerminal", text: "@block" },
      { type: "Terminal", text: "}" },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={forAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage
The following query shows example usage of this statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a person for everyone in the array
FOR $name IN ['Tobie', 'Jaime'] {
	CREATE type::record('person', $name) CONTENT {
		name: $name
	};
};
```

The following query shows the `FOR` statement being used update a property on every user matching certain criteria.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Set can_vote to true for every person over 18 years old.
FOR $person IN (SELECT VALUE id FROM person WHERE age >= 18) {
	UPDATE $person SET can_vote = true;
};
```

## Ranges in FOR loops

<Since v="v2.0.0" />

A `FOR` loop can also be made out of a [range UUID](/docs/surrealql/datamodel/ranges) of integers.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

FOR $year IN 0..=2024 {
    CREATE historical_events SET
        for_year = $year,
        events = "To be added";
};
```

## Limitations of FOR loops

Parameters declared outside of a `FOR` loop can be used inside the loop.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

LET $table1 = "person";
LET $table2 = "cat";

FOR $key in 0..4 {
    CREATE type::record($table1, $key);
	  CREATE type::record($table2, $key);
};
```

However, they currently cannot be modified inside a loop, making an operation like the following impossible.

```surql
LET $init = [];

FOR $num IN 1..=3 {
	$init += $num;
};
-- Error: 'assignment operators are only allowed in SET and DUPLICATE KEY UPDATE clauses'

RETURN $init;
```

In this case, the [`array::fold`](/docs/surrealql/functions/database/array#arrayfold) and [`array::reduce`](/docs/surrealql/functions/database/array#arrayreduce) functions can often be used to accomplish the intended behaviour.

```surql
/**[test]

[[test.results]]
value = "6"

*/

(<array>1..=3).reduce(|$one, $two| $one + $two);
```

```surql title="Output"
6
```

## ifelse.mdx

---
sidebar_position: 11
sidebar_label: IF ELSE
title: IF ELSE statement | SurrealQL
description: The IF ELSE statement can be used as a main statement, or within a parent statement, to return a value depending on whether a condition, or a series of conditions match.
---

# `IF ELSE` statement

The `IF ELSE` statement can be used as a main statement, or within a parent statement, to return a value depending on whether a condition, or a series of conditions match. The statement allows for multiple `ELSE IF` expressions, and a final `ELSE` expression, with no limit to the number of `ELSE IF` conditional expressions.

> [!NOTE]
> As [THROW](/docs/surrealql/statements/throw), [CONTINUE](/docs/surrealql/statements/continue), and [BREAK](/docs/surrealql/statements/break) do not return an expression, they must be inside a separate code block inside an `IF ELSE` statement.

An `IF ELSE` syntax uses `{}` to open up a code block on each condition check which will be run when it evaluates as [truthy](/docs/surrealql/datamodel/values#values-and-truthiness).

import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

<Tabs syncKey="ifelse-statement">

  <TabItem label="Syntax">

```surql title="Modern syntax"
IF @condition { @expression; .. }
   [ ELSE IF @condition { @expression; .. } ] ...
   [ ELSE { @expression; .. } ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const ifElseAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "IF" },
      { type: "NonTerminal", text: "@condition" },
      { type: "Terminal", text: "{" },
      { type: "NonTerminal", text: "@expression; .." },
      { type: "Terminal", text: "}" },
      { type: "ZeroOrMore", child: { type: "Sequence", children: [
        { type: "Terminal", text: "ELSE" },
        { type: "Terminal", text: "IF" },
        { type: "NonTerminal", text: "@condition" },
        { type: "Terminal", text: "{" },
        { type: "NonTerminal", text: "@expression; .." },
        { type: "Terminal", text: "}" }
      ] } },
      { type: "Optional", child: { type: "Sequence", children: [
        { type: "Terminal", text: "ELSE" },
        { type: "Terminal", text: "{" },
        { type: "NonTerminal", text: "@expression; .." },
        { type: "Terminal", text: "}" }
      ] } }
    ]}
  ]
};

<RailroadDiagram ast={ifElseAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

### Basic usage

The following queries show example usage of this statement.

The smallest possible `IF THEN` statement simply does something when a condition is true, and nothing otherwise.

```surql
IF 9 = 9 { 'Nine is indeed nine' };
```

As the last line of a scope is its return value, the `RETURN` keyword can also be placed before the entire `IF THEN` statement. This is particularly convenient in long `IF ELSE` chains to avoid using the `RETURN` keyword at the end of every check for a condition.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "'Positive uninteresting number'"

*/

LET $num = 100;

RETURN IF $num < 0 {
    "Negative"
} ELSE IF $num = 0 {
    "Zero"
} ELSE IF $num = 13 {
    "Thirteen"
} ELSE {
    "Positive uninteresting number"
};
```

The `RETURN` keyword can even be omitted, as the output at each point is the output of the entire expression if evaluated as truthy.

```surql
LET $num = 100;

IF $num < 0 {
    "Negative"
} ELSE IF $num = 0 {
    "Zero"
} ELSE IF $num = 13 {
    "Thirteen"
} ELSE {
    "Positive uninteresting number"
};
```

The `THROW` keyword inside `{}` braces can be used to break out of an `IF ELSE` statement early.

```surql
LET $badly_formatted_datetime = "2024-04TT08:08:08Z";

IF !type::is_datetime($badly_formatted_datetime) {
    THROW "Whoops, that isn't a real datetime"
};
```

```surql title="Response"
"An error occurred: Whoops, that isn't a real datetime"
```

`ELSE IF` branches and a final `ELSE` can be added into an `IF ELSE` statement:

```surql
RETURN
    IF $access = "admin" { (SELECT * FROM account) }
    ELSE IF $access = "user"  { (SELECT * FROM $auth.account) }
    ELSE { THROW "Access method hasn't been defined!" };
```

### Advanced usage

The output of an `IF ELSE` statement can be assigned to a parameter:

```surql
LET $num = 9;

LET $odd_even = 
    IF $num % 2 = 0 { "even" } 
    ELSE { "odd" };
```

If-else statements can also be used as subqueries within other statements.

```surql
UPSERT person SET railcard =
    IF age <= 10 { 'junior' }
    ELSE IF age <= 21 { 'student' }
    ELSE IF age >= 65 { 'senior' }
    ELSE { NULL };
```

You can also have nested conditions:

```surql
IF $access = 'admin'
	{
        CREATE admin_user_event SET 
            time = time::now(),
            info = "Admin user activity registered";
		SELECT * FROM admin_data WHERE access_level = 'full';
	}
ELSE IF $access = 'user'
	{
		IF $auth.role = 'premium'
			{
                CREATE premium_user_event SET 
                    time = time::now(),
                    info = "Premium user activity registered";

				IF $auth.subscription_status = 'active'
					{ SELECT * FROM premium_user_data WHERE active = 1 }
				ELSE IF $auth.subscription_status = 'trial'
					{ SELECT * FROM trial_user_data }
				ELSE
					{ SELECT * FROM basic_user_data }
			}
		ELSE IF $auth.role = 'standard'
			{ SELECT * FROM standard_user_data WHERE region = 'US' }
		ELSE IF $auth.role = 'standard' AND $auth.subscription_status = 'active'
			{ SELECT * FROM standard_user_data WHERE region = 'EU' }
		ELSE
			{ SELECT * FROM unauthorized_user_data }
	}
ELSE
	{ SELECT * FROM unknown_access_data };
```


## index.mdx

---
sidebar_position: 1
sidebar_label: Overview
title: Statements | SurrealQL
description: Statements are used to configure and query a database.
---

import Since from '@components/shared/Since.astro'

# Statements

SurrealDB has a variety of statements that let you configure and query a database. In this section, we'll look at the different types of statements that are available.

## Types of statements

SurrealDB has a large variety of statements. They can be divided into three types:

* Statements that define and access database resources,
* Statements used for control flow and handling manual transactions,
* Statements used in the context of queries, usually in CRUD (create, read, update, delete) operations.

### Database resource statements

These statements pertain to defining, removing, altering, and rebuilding database resources. They are:

* [`DEFINE`](/docs/surrealql/statements/define) statements to define database resources,
* [`ALTER`](/docs/surrealql/statements/alter) statements to alter certain resources (note: `DEFINE` statements with the `OVERWRITE` clause are more frequently used),
* [`REMOVE`](/docs/surrealql/statements/remove) statements to remove resources,
* [`REBUILD`](/docs/surrealql/statements/rebuild) to rebuild an index,
* [`ACCESS`](/docs/surrealql/statements/access) to manage access grants.

Some other statements pertain to using defined resources. They are:

* [`USE`](/docs/surrealql/statements/use) to move from one namespace or database to another,
* [`INFO`](/docs/surrealql/statements/info) statements to see the definitions for resources,
* [`SHOW`](/docs/surrealql/statements/show) to see the changefeed for a table or database.

### Control flow statements

These statements are used to describe how query execution should progress.

Some control flow statements only pertain to manual transactions. While all statements in SurrealDB are conducted inside their own transaction, these statements can be used to manually set up a larger transaction composed of multiple statements. They are:

* [`BEGIN`](/docs/surrealql/statements/begin) to begin a manual transaction,
* [`COMMIT`](/docs/surrealql/statements/commit) to commit a transaction,
* [`CANCEL`](/docs/surrealql/statements/cancel) to cancel a transaction.

Other control flow statements are used in the same manner as in other programming languages. They are:

* [`FOR`](/docs/surrealql/statements/for) to begin a for loop,
* [`CONTINUE`](/docs/surrealql/statements/continue) to continue to the next iteration of a loop,
* [`BREAK`](/docs/surrealql/statements/break) to break out of a for loop, internal scope, function, etc.,
* [`IF` and `ELSE`](/docs/surrealql/statements/ifelse) to describe what to do depending on a condition,
* [`SLEEP`](/docs/surrealql/statements/sleep) to halt all execution for a certain length of time,
* [`RETURN`](/docs/surrealql/statements/return) to break and return a value,
* [`THROW`](/docs/surrealql/statements/throw) to cancel execution and return an error.

### Query statements

These statements are used to execute queries, most often but not always in the context of a CRUD operation.

The statements that pertain to the handling of records are:

* [`CREATE`](/docs/surrealql/statements/create) to create one or more records of one or more types of tables,
* [`INSERT`](/docs/surrealql/statements/insert) to create one or more regular records or graph edges,
* [`RELATE`](/docs/surrealql/statements/relate) to create a single graph edge between two records,
* [`UPDATE`](/docs/surrealql/statements/update) to update records,
* [`UPSERT`](/docs/surrealql/statements/upsert) to update a record and create a new one if it does not exist,
* [`SELECT`](/docs/surrealql/statements/select) to select records (but also ad-hoc values),
* [`LIVE SELECT`](/docs/surrealql/statements/live) to stream all the changes to a table,
* [`DELETE`](/docs/surrealql/statements/delete) to delete one or more records,
* [`KILL`](/docs/surrealql/statements/kill) to cancel a `LIVE SELECT`.

The other statements used when executing a query are:

* [`LET`](/docs/surrealql/statements/let) to assign a value to a parameter for later use,
* [`RETURN`](/docs/surrealql/statements/return) when used in front of a value or expression, in which case it has no effect but is often used for readability.

The following flowchart can be used to get a sense of when it makes sense to use `CREATE`, `INSERT`, `UPDATE`, `UPSERT`, and `RELATE`.

![A flowchart that explains in which cases to use the statements create, insert, update, insert, and relate.](statement_flowchart.png)

## Statement parameters

A number of parameters prefixed with `$` are automatically available within a statement that provide access to relevant context inside the statement. These are known as reserved variable names. For example:

* [$before](/docs/surrealql/parameters#before-after) and [$after](/docs/surrealql/parameters#before-after) can be accessed in statements that mutate values to see the values before and after an update,
* [$session](/docs/surrealql/parameters#session) provides context on the current session,
* [$parent](/docs/surrealql/parameters#parent-this) provides access to the value in a primary query while inside a subquery.

For a full list of these automatically generated parameters, see the [parameters](/docs/surrealql/parameters#reserved-variable-names) page.

## Output when resource not defined

<Since v="v3.0.0" />

Many but not all statements in versions before 3.0 returned an empty array when a resource was not defined.

```surql
SELECT * FROM person;
DELETE person;
REMOVE TABLE person;
```

```surql title="Output before 3.0"
-------- Query 1 --------

[]

-------- Query 2 --------

[]

-------- Query 3 --------

"The table 'person' does not exist"
```

All statements return an error if this is the case, making it clear that the resource is not defined as opposed to defined and empty.

```surql
SELECT * FROM person;
DELETE person;
```

```surql title="Output"
-------- Query --------

"The table 'person' does not exist"

-------- Query --------

"The table 'person' does not exist"
```

However, statements that create a resource will not return an error unless the [database is defined](/docs/surrealql/statements/define/database) as `STRICT`. Instead, they will automatically define the needed table (and even use the [`$session`](/docs/surrealdb/security/authentication#session) parameter to define the database and namespace if necessary) so that the query will work.

Note the output of the following queries, in which the value `[]` is returned after deleting the records of a defined table to show that zero records were returned. However, the final `REMOVE TABLE` statement returns a simple `NONE` to indicate that the statement succeeded.

```surql
SELECT * FROM person; -- Error
DELETE person;        -- Error
CREATE person:one;    -- Succeeds
DELETE person;        -- Empty array
REMOVE TABLE person;  -- NONE
```

The following chart can help to remember which output to expect depending on your SurrealDB version and database strictness.

| Example of statement  | STRICT database      | Non-STRICT database behavior (default)                     | Behavior in versions < 3.0       |
|-----------------------|----------------------|------------------------------------------------------------|----------------------------------|
| CREATE / UPDATE       | Error if not defined | Succeeds (defines table, database, namespace if necessary) | Identical                        |
| SELECT / DELETE       | Error if not defined | Returns error if table not defined                         | Empty array if table not defined |

## info.mdx

---
sidebar_position: 12
sidebar_label: INFO
title: INFO statement | SurrealQL
description: The INFO command outputs information about the setup of the SurrealDB system.
---

import Since from "@components/shared/Since.astro";
import RailroadDiagram from "@components/RailroadDiagram.astro";
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

# `INFO` statement

The `INFO` command outputs information about the setup of the SurrealDB system. There are a number of different `INFO` commands for retrieving the configuration at the different levels of the database.

<Tabs syncKey="info-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
INFO FOR [
	ROOT
	| NS | NAMESPACE
	| DB | DATABASE
	| TABLE @table
	| USER @user [ON @level]
    | INDEX @index ON @table
];
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const infoAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "INFO" },
      { type: "Terminal", text: "FOR" },
      { type: "Choice", index: 1, children: [
        { type: "Terminal", text: "ROOT" },
        { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NS" }, { type: "Terminal", text: "NAMESPACE" } ] },
        { type: "Choice", index: 1, children: [ { type: "Terminal", text: "DB" }, { type: "Terminal", text: "DATABASE" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "TABLE" }, { type: "NonTerminal", text: "@table" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "USER" }, { type: "NonTerminal", text: "@user" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "ON" }, { type: "NonTerminal", text: "@level" } ] } } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "INDEX" }, { type: "NonTerminal", text: "@index" }, { type: "Terminal", text: "ON" }, { type: "NonTerminal", text: "@table" } ] }
      ] },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={infoAst} className="my-6" />

  </TabItem>
</Tabs>

The information returned from an `INFO` command is an object containing items that almost always correspond to a matching [DEFINE](/docs/surrealql/statements/define) statement. For example, the `INFO FOR NS` command returns the information on the access methods, databases and users of a namespace, which are defined with `DEFINE ACCESS`, `DEFINE DATABASE` and `DEFINE USER` statements.

> [!NOTE]
> Before SurrealDB v3.0.0, the output of an `INFO FOR` was only able to be used as a standalone statement and not in a dynamic context, such as inside other queries or as the value of a parameter.

## Example usage

There are a number of different `INFO` commands for retrieving the configuration at the different levels of the database.

## System  information

### Root information
The top-level ROOT command returns information regarding:
- The users and namespaces which exists within the SurrealDB system.
- The memory allocated by SurrealDB itself. Note that this may not match what the operating system reports, as it also includes memory consumed by third-party libraries or pre-allocated memory.
- The level of parallelism: This number indicates the number of available hardware threads.

> [!NOTE]
> You must be authenticated as a top-level root user to execute this command.

#### Examples

```surql
INFO FOR ROOT;
```

```surql title="Sample output"
{
	accesses: {},
	namespaces: {
		ns: 'DEFINE NAMESPACE ns'
	},
	nodes: {
		"2d3b720d-f152-4c0d-8a16-26d1474ed3cd": 'NODE 2d3b720d-f152-4c0d-8a16-26d1474ed3cd SEEN 1745463977888 ACTIVE'
	},
	system: {
		available_parallelism: 14,
		cpu_usage: 0.3816290497779846f,
		load_average: [
			1.2734375f,
			1.68310546875f,
			1.9189453125f
		],
		memory_allocated: 13900485,
		memory_usage: 136314880,
		physical_cores: 14,
		threads: 32
	},
	users: {
		root: "DEFINE USER root ON ROOT PASSHASH '$argon2id$v=19$m=19456,t=2,p=1$0RoO7PtdHuGLOz9Pomoucg$T6FYVogdEF8sFse/Su11nKaZb8FjEp3Bb3rD35mI1b8' ROLES OWNER DURATION FOR TOKEN 1h, FOR SESSION NONE"
	}
}
```

### Namespace information

The `NS` or `NAMESPACE` command returns information regarding the users, databases and access methods under the namespace in use.

> [!NOTE]
> You must be authenticated as a top-level root user, or a namespace user to execute this command.

> [!NOTE]
> You must have a NAMESPACE selected before running this command.

#### Examples

```surql
INFO FOR NS;
```

```surql title="Sample output"
{
    accesses: {},
    databases: {
        db: 'DEFINE DATABASE db'
    },
    users: {
        n: "DEFINE USER n ON NAMESPACE PASSHASH '' ROLES VIEWER DURATION FOR TOKEN 1h, FOR SESSION NONE",
        username: "DEFINE USER username ON NAMESPACE PASSHASH '$argon2id$v=19$m=19456,t=2,p=1$K9DIBCuzH2IA6w7t3ZVGkQ$KkRODt0cqgUap9OwZCxLJC4ESo6wEToUk55oumhmgA0' ROLES EDITOR DURATION FOR TOKEN 1m, FOR SESSION 12h"
    }
}
```

### Database information

The `DB` or `DATABASE` command returns information regarding the users, tables, params, models, functions, analyzers and access methods under the database in use.

> [!NOTE]
> You must be authenticated as a top-level root user, a namespace user, or a database user to execute this command.

> [!NOTE]
> You must have a NAMESPACE and a DATABASE selected before running this command.

#### Examples

```surql
INFO FOR DB;
```

```surql title="Sample output"
{
    accesses: {},
    analyzers: {},
    apis: {},
    buckets: {},
    configs: {},
    functions: {},
    models: {},
    params: {},
    tables: {
        person: 'DEFINE TABLE person TYPE ANY SCHEMALESS PERMISSIONS NONE'
    },
    users: {
        db_user: "DEFINE USER db_user ON DATABASE PASSHASH '$argon2id$v=19$m=19456,t=2,p=1$S8WJ88AnJwWah2VjqnmnoA$OJcQs9SHC5R5q3kOimiKsV5fIUpwZiax3RUcW8VQupk' ROLES OWNER DURATION FOR TOKEN 1h, FOR SESSION NONE"
    }
}
```

### Table information

The `TABLE` command returns information regarding the events, fields, tables, and live statement configurations on a specific table.

> [!NOTE]
> You must be authenticated as a top-level root user, a namespace user, or a database user to execute this command.

> [!NOTE]
> You must have a NAMESPACE and a DATABASE selected before running this command.

#### Examples

```surql
INFO FOR TABLE user;
```

```surql title="Sample output"
{
    events: {},
    fields: {
        name: 'DEFINE FIELD name ON user TYPE string PERMISSIONS FULL'
    },
    indexes: {},
    lives: {},
    tables: {}
}
```

### User information

The `USER` command returns information for a user [defined](/docs/surrealql/statements/define/user) on either the root, namespace, or database level.

> [!NOTE]
> You must be authenticated as a user equal to or greater than the level of the user you are attempting to obtain information for to execute this command.

#### Examples

```surql
INFO FOR USER root ON ROOT;
INFO FOR USER ns_user ON NAMESPACE;
INFO FOR USER db_user ON DATABASE;
```

If a level after `ON` is not specified, the `INFO` command will default to the database level. Thus, the following two commands are equivalent.

```surql
INFO FOR USER db_user ON DATABASE;
INFO FOR USER db_user;
```

```surql title="Sample output"
"DEFINE USER db_user ON DATABASE PASSHASH '$argon2id$v=19$m=19456,t=2,p=1$S8WJ88AnJwWah2VjqnmnoA$OJcQs9SHC5R5q3kOimiKsV5fIUpwZiax3RUcW8VQupk' ROLES OWNER DURATION FOR TOKEN 1h, FOR SESSION NONE"
```

### Index information

`INFO FOR INDEX` returns the status for an index: started, initial indexing, update indexing, built, or error.

This command only applies when the [`CONCURRENTLY`](/docs/surrealql/statements/define/indexes#using-concurrently-clause) clause is used in a `DEFINE INDEX` command. Without this clause, the following statement will not be executed until the index is fully created, or fails. In this case, the `INFO FOR INDEX` statement will return an empty object: `{}`.


```surql
CREATE |user:50000| SET name = id.id() RETURN NONE;
DEFINE INDEX unique_name ON TABLE user FIELDS name UNIQUE;
INFO FOR INDEX unique_name ON TABLE user;
```

However, when the `CONCURRENTLY` clause is used, the index will build in the background while other statements are permitted to run. In this case, the `INFO FOR INDEX` statement will provide the current status on the index. The following code sample shows such an example in which an index is defined on a large number of records. A [`SLEEP`](/docs/surrealql/statements/sleep) statement is run in between each `INFO FOR INDEX` command to show the progress after each 50 millisecond interval.

```surql
CREATE |user:50000| SET name = id.id() RETURN NONE;
DEFINE INDEX unique_name ON TABLE user FIELDS name UNIQUE CONCURRENTLY;
INFO FOR INDEX unique_name ON user;
SLEEP 50ms;
INFO FOR INDEX unique_name ON user;
SLEEP 50ms;
INFO FOR INDEX unique_name ON user;
SLEEP 50ms;
INFO FOR INDEX unique_name ON user;
```

```surql title="Possible output"
-------- Query 1 --------
{ 
    building: {
        initial: 0,
        pending: 0,
        status: 'indexing', 
        updated: 0
    }
}

-------- Query 2 --------
{ 
    building: {
        initial: 100,
        pending: 20,
        status: 'indexing', 
        updated: 0
    }
}

-------- Query 3 --------
{ 
    building: {
        initial: 100,
        pending: 4,
        status: 'indexing', 
        updated: 16
    }
}

-------- Query 4 --------
{
    building: {
        status: 'ready'
    }
}
```

### The `STRUCTURE` clause

> [!NOTE]
> This clause was created for internal use and is subject to change without notice.

Adding the `STRUCTURE` clause changes the structure of the statement from an object that contains objects into an object with fields that each contain an array and often extra info.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ events: {  }, fields: { name: 'DEFINE FIELD name ON user TYPE string PERMISSIONS FULL' }, indexes: {  }, lives: {  }, tables: {  } }"

[[test.results]]
value = "{ events: [], fields: [{ flex: false, kind: 'string', name: 'name', permissions: { create: true, select: true, update: true }, readonly: false, what: 'user' }], indexes: [], lives: [], tables: [] }"

*/

DEFINE TABLE user SCHEMAFULL;
DEFINE FIELD name ON user TYPE STRING;

INFO FOR TABLE user;
INFO FOR TABLE user STRUCTURE;
```

```surql title="Output"

-------- Query --------

{
	events: {},
	fields: {
		name: 'DEFINE FIELD name ON user TYPE string PERMISSIONS FULL'
	},
	indexes: {},
	lives: {},
	tables: {}
}

-------- Query --------

{
	events: [],
	fields: [
		{
			flex: false,
			kind: 'string',
			name: 'name',
			permissions: {
				create: true,
				delete: true,
				select: true,
				update: true
			},
			readonly: false,
			what: 'user'
		}
	],
	indexes: [],
	lives: [],
	tables: []
}
```

### Using the output of `INFO`

<Since v="v3.0.0" />

The output of an `INFO` statement, both with and without the `STRUCTURE` clause, can be used in other operations. As the output of the statement is always a single object, the SurrealQL [object functions](/docs/surrealql/functions/database/object) can also be used on the output for such tasks as schema change tracking.

```surql
LET $cat = CREATE ONLY cat RETURN VALUE id;

LET $first_schema = {
    revision: rand::uuid(),
    schema: INFO FOR DB
};

$first_schema;

CREATE person SET feeds = [$cat];

LET $second_schema = {
    revision: rand::uuid(),
    schema: INFO FOR DB
};

$second_schema;

$first_schema.diff($second_schema);
```

```surql title="Output"
-------- First schema --------

{
	revision: u'019665cc-f730-75f0-8251-894e11fee7d8',
	schema: {
		accesses: {},
		analyzers: {},
		apis: {},
		buckets: {},
		configs: {},
		functions: {},
		models: {},
		params: {},
		tables: {
			cat: 'DEFINE TABLE cat TYPE ANY SCHEMALESS PERMISSIONS NONE'
		},
		users: {}
	}
}

-------- Second schema --------

{
	revision: u'019665cc-f73b-7313-807f-dd22ad1a0685',
	schema: {
		accesses: {},
		analyzers: {},
		apis: {},
		buckets: {},
		configs: {},
		functions: {},
		models: {},
		params: {},
		tables: {
			cat: 'DEFINE TABLE cat TYPE ANY SCHEMALESS PERMISSIONS NONE',
			person: 'DEFINE TABLE person TYPE ANY SCHEMALESS PERMISSIONS NONE'
		},
		users: {}
	}
}

-------- Diff --------

[
	{
		op: 'replace',
		path: '/revision',
		value: u'019665cc-f73b-7313-807f-dd22ad1a0685'
	},
	{
		op: 'add',
		path: '/schema/tables/person',
		value: 'DEFINE TABLE person TYPE ANY SCHEMALESS PERMISSIONS NONE'
	}
]
```

### Default namespace and database output

<Since v="v3.0.0" />

A namespace and database with the name of `name` are generated by default when starting a SurrealDB instance unless the `SURREAL_NO_DEFAULTS` [environment variable](/docs/surrealdb/cli/env) is set to false or the [`--no-defaults`](/docs/surrealdb/cli/start) flag is used.

The output of the `INFO` statement in this case includes a comment on how these two resources were defined.

```surql
[INFO FOR ROOT.namespaces, INFO FOR NS.databases];
```

```surql title="Output"
[
	{ main: "DEFINE NAMESPACE main COMMENT 'Default namespace generated by SurrealDB'" }, 
	{ main: "DEFINE DATABASE main COMMENT 'Default database generated by SurrealDB'" }
]
```

## insert.mdx

---
sidebar_position: 13
sidebar_label: INSERT
title: INSERT statement | SurrealQL
description: The INSERT statement can be used to insert or update data into the database, using the same statement syntax as the traditional SQL Insert statement.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `INSERT` statement

The `INSERT` statement can be used to insert or update data into the database, using the same statement syntax as the traditional SQL Insert statement.

### Statement syntax

<Tabs syncKey="insert-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
INSERT [ RELATION ] [ IGNORE ] INTO @what
	[ @value
	  | (@fields) VALUES (@values)
		[ ON DUPLICATE KEY UPDATE @field = @value ... ]
	]
	[ RETURN NONE | RETURN BEFORE | RETURN AFTER | RETURN DIFF | RETURN @statement_param, ... | RETURN VALUE @statement_param ]
;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const insertAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "INSERT" },
      { type: "Optional", child: { type: "Terminal", text: "RELATION" } },
      { type: "Optional", child: { type: "Terminal", text: "IGNORE" } },
      { type: "Terminal", text: "INTO" },
      { type: "NonTerminal", text: "@what" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [
        { type: "NonTerminal", text: "@value" },
        { type: "Sequence", children: [ { type: "Terminal", text: "(@fields)" }, { type: "Terminal", text: "VALUES" }, { type: "Terminal", text: "(@values)" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "ON DUPLICATE KEY UPDATE" }, { type: "NonTerminal", text: "@field = @value ..." } ] } } ] }
      ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "RETURN" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NONE" }, { type: "Terminal", text: "BEFORE" }, { type: "Terminal", text: "AFTER" }, { type: "Terminal", text: "DIFF" }, { type: "NonTerminal", text: "@statement_param, ..." }, { type: "Sequence", children: [ { type: "Terminal", text: "VALUE" }, { type: "NonTerminal", text: "@statement_param" } ] } ] } ] } },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={insertAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following query shows example usage of this statement.

```surql
/**[test]

[[test.results]]
value = "[{ founded: '2021-09-10', founders: [person:tobie, person:jaime], id: company:5fc7p7d0kiirdx7accfn, name: 'SurrealDB', tags: ['big data', 'database'] }]"
skip-record-id-key = true

*/

INSERT INTO company {
	name: 'SurrealDB',
	founded: "2021-09-10",
	founders: [person:tobie, person:jaime],
	tags: ['big data', 'database']
};
```

Records can also be inserted by using the `VALUES` keyword. This keyword is preceded by the name of the fields in question, and followed by comma-separated values matching the number of fields specified.

```surql
/**[test]

[[test.results]]
value = "[{ founded: '2021-09-10', id: company:l6luw1w1lqr7v2loau6a, name: 'SurrealDB' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ founded: '1967-05-03', id: company:racyetsf1x1hbu7d5g5s, name: 'Acme Inc.' }, { founded: '1976-04-01', id: company:x9pi148wprp30jfv5o4q, name: 'Apple Inc.' }]"
skip-record-id-key = true

*/

-- Insert a single record
INSERT INTO
	company (name, founded)
	VALUES  ('SurrealDB', '2021-09-10');

-- Insert multiple records
INSERT INTO
	company (name, founded)
	VALUES  ('Acme Inc.', '1967-05-03'), ('Apple Inc.', '1976-04-01');
```

It is possible to update records which already exist or violate a unique index by specifying an `ON DUPLICATE KEY UPDATE` clause. This clause also allows incrementing and decrementing numeric values, and adding or removing values from arrays. To increment a numeric value, or to add an item to an array, use the `+=` operator. To decrement a numeric value, or to remove an value from an array, use the `-=` operator.

```surql
/**[test]

[[test.results]]
value = "[{ id: product:bt4cwqd5cwwmp68w3dps, name: 'Salesforce', url: 'salesforce.com' }]"
skip-record-id-key = true

*/

INSERT INTO product (name, url) VALUES ('Salesforce', 'salesforce.com') ON DUPLICATE KEY UPDATE tags += 'crm';
```

Field names inside `ON DUPLICATE KEY UPDATE` refer to the fields of the existing record. To access the fields of the new record that was attempted to be inserted, prefix the field name with [`$input`](/docs/surrealql/parameters#input):

```surql
/**[test]

[[test.results]]
value = "[{ at_year: 2024, id: city:Calgary, population: 1665000 }]"

*/

INSERT INTO city (id, population, at_year) VALUES ("Calgary", 1665000, 2024)
ON DUPLICATE KEY UPDATE
	population = $input.population,
	at_year = $input.at_year;
```

An example of `ON DUPLICATE KEY UPDATE` when a unique key is encountered shows the same behaviour as that with a duplicate record:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ data_for: user:one, id: user_data:24gujsffwl4vt2zgp2yl, some: 'data' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ data_for: user:one, id: user_data:24gujsffwl4vt2zgp2yl, last_edited: d'2025-10-08T03:22:23.325474Z', some: 'data', times_updated: 1 }]"
skip-record-id-key = true

*/

DEFINE FIELD data_for ON user_data TYPE record<user>;
DEFINE INDEX one_user ON user_data FIELDS data_for UNIQUE;

INSERT INTO user_data {
    data_for: user:one,
    some: "data"
} ON DUPLICATE KEY UPDATE times_updated += 1, last_edited = time::now();

INSERT INTO user_data {
    data_for: user:one,
    some_more: "data"
} ON DUPLICATE KEY UPDATE times_updated += 1, last_edited = time::now();
```

```surql title="Output"
-------- Query --------

[
	{
		for: user:one,
		id: user_data:kp78dubsxmp4f04x0de3,
		some: 'data'
	}
]

-------- Query --------

[
	{
		for: user:one,
		id: user_data:kp78dubsxmp4f04x0de3,
		last_edited: d'2025-07-14T05:15:52.146Z',
		some: 'data',
		times_updated: 1
	}
]
```

Using the insert statement, it is possible to copy records easily between tables. The records being copied will have the same id in the new table, but the record id will signify the new table name.

```surql
/**[test]

[[test.results]]
value = "[]"

*/

INSERT INTO recordings_san_francisco (SELECT * FROM temperature WHERE city = 'San Francisco');
```

Furthermore, it is possible to perform a bulk insert in a single query. The `@what` part of the syntax can be either a single object or an array of objects.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', surname: 'Morgan Hitchcock' }, { id: person:tobie, name: 'Tobie', surname: 'Morgan Hitchcock' }]"

*/

INSERT INTO person [
   { id: "jaime", name: "Jaime", surname: "Morgan Hitchcock" },
   { id: "tobie", name: "Tobie", surname: "Morgan Hitchcock" },
];
```

### Ignoring duplicates

While attempting to insert one or more records via the `INSERT` statement, if the record ID is already present in the table, the query will encounter an error and fail. If the `IGNORE` clause is supplied, records with an already existing or duplicate ID will be silently ignored.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', surname: 'Morgan Hitchcock' }, { id: person:tobie, name: 'Tobie', surname: 'Morgan Hitchcock' }]"

*/

INSERT IGNORE INTO person [
   { id: "jaime", name: "Jaime", surname: "Morgan Hitchcock" },
   { id: "tobie", name: "Tobie", surname: "Morgan Hitchcock" },

   { id: "jaime", name: "Jaime", surname: "Morgan Hitchcock" }, -- will not throw an error
];
```

### Return Values

<Since v="v2.0.0" />

By default, the `INSERT` statement returns the record once it has been inserted. To change what is returned, we can use the `RETURN` clause, specifying either `NONE`, `BEFORE`, `AFTER`, `DIFF`, or a comma-separated list of specific fields to return.

`RETURN NONE` can be useful to avoid excess output:

```surql
/**[test]

[[test.results]]
value = "[]"

*/

-- Insert a record and return nothing
INSERT INTO company {
	name: 'SurrealDB',
	founded: "2021-09-10",
	founders: [person:tobie, person:jaime],
	tags: ['big data', 'database']
} RETURN NONE;
```

`RETURN DIFF` returns the changeset diff:

```surql
/**[test]

[[test.results]]
value = "[[{ op: 'replace', path: '', value: { founded: '2021-09-10', founders: [person:tobie, person:jaime], id: company:tmx6q7chr1vt73t95gx8, name: 'SurrealDB', tags: ['big data', 'database'] } }]]"
skip-record-id-key = true

*/

-- Insert a record and return the diff
INSERT INTO company {
	name: 'SurrealDB',
	founded: "2021-09-10",
	founders: [person:tobie, person:jaime],
	tags: ['big data', 'database']
} RETURN DIFF;
```

```surql title="Response"
-------- Query 1 (500µs) --------

[
	[
		{
			op: 'replace',
			path: '/',
			value: {
				founded: '2021-09-10',
				founders: [
					person:tobie,
					person:jaime
				],
				id: company:hu5o1wqbo29t10engbeo,
				name: 'SurrealDB',
				tags: [
					'big data',
					'database'
				]
			}
		}
	]
]
```

`RETURN BEFORE` inside a `INSERT` statement is essentially a synonym for `RETURN NONE`, while `RETURN AFTER` is the default behaviour for `INSERT`.

```surql
/**[test]

[[test.results]]
value = "[NONE]"

*/

-- Before insert will always return NONE as it is the same as the record being inserted
INSERT INTO company {
	name: 'SurrealDB',
	founded: "2021-09-10",
	founders: [person:tobie, person:jaime],
	tags: ['big data', 'database']
} RETURN BEFORE;
```

```surql
/**[test]

[[test.results]]
value = "[{ founded: '2021-09-10', founders: [person:tobie, person:jaime], id: company:yzc8d07g376x3qddxmdn, name: 'SurrealDB', tags: ['big data', 'database'] }]"
skip-record-id-key = true

*/

-- Return the record after creation
INSERT INTO company {
	name: 'SurrealDB',
	founded: "2021-09-10",
	founders: [person:tobie, person:jaime],
	tags: ['big data', 'database']
} RETURN AFTER;
```

You can also return specific fields from a created record, the value of a single field using `VALUE`, as well as ad-hoc fields to modify the output as needed.

```surql
/**[test]

[[test.results]]
value = "[{ age: 46, age_next_year: 47, interests: ['skiing', 'music'] }]"

[[test.results]]
value = "[27, -55]"

*/

INSERT INTO person {
    age: 46,
    username : "john-smith",
    interests : ['skiing', 'music'] }
RETURN
    age,
    interests,
    age + 1 AS age_next_year;

INSERT INTO planet [
	{
		name: 'Venus',
        surface_temp: 462,
        temp_55_km_up: 27
	},
	{
		name: 'Earth',
        surface_temp: 15,
        temp_55_km_up: -55
	}
] RETURN VALUE temp_55_km_up;
```

```surql title="Response"
-------- Query --------

[
	{
		age: 46,
		age_next_year: 47,
		interests: [
			'skiing',
			'music'
		]
	}
]

-------- Query --------

[
	27,
	-55
]
```

## Bulk insert

<Since v="v2.0.0" />

The `INSERT` statement supports bulk insert, which allows multiple records to be inserted in a single query. The `@what` part of the syntax can be either a single object or an array of objects.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', surname: 'Morgan Hitchcock' }, { id: person:tobie, name: 'Tobie', surname: 'Morgan Hitchcock' }]"

*/

INSERT INTO person [
   { id: "jaime", name: "Jaime", surname: "Morgan Hitchcock" },
   { id: "tobie", name: "Tobie", surname: "Morgan Hitchcock" },
   -- ... 1000 more records
];
```

## Insert relation tables

The `INSERT` statement can also be used to add records into relation tables. The `@what` part of the syntax can be either a single object or an array of objects.

Learn more about creating relationships between tables in the [RELATE](/docs/surrealql/statements/relate) statement. For example:

```surql
/**[test]

[[test.results]]
value = "[{ id: person:1 }, { id: person:2 }, { id: person:3 }]"

[[test.results]]
value = "[{ id: likes:object, in: person:1, out: person:2 }]"

[[test.results]]
value = "[{ id: likes:array, in: person:1, out: person:2 }, { id: likes:array_two, in: person:2, out: person:3 }]"

[[test.results]]
value = "[{ id: likes:values, in: person:1, out: person:2 }]"

[[test.results]]
value = "[[likes:array, likes:object, likes:values], [likes:array_two], []]"

*/

-- Insert records into the person table
INSERT INTO person [
	{ id: 1 },
	{ id: 2 },
	{ id: 3 },
];
-- Insert a single relation
INSERT RELATION INTO likes {
	in: person:1,
	id: 'object',
	out: person:2,
};

-- Insert multiple relations
INSERT RELATION INTO likes [
	{
		in: person:1,
		id: 'array',
		out: person:2,
	},
	{
		in: person:2,
		id: 'array_two',
		out: person:3,
	}
];

-- Insert a relation and return the value of the likes field
INSERT RELATION INTO likes (in, id, out)
	VALUES (person:1, 'values', person:2);

-- Select the value of the likes field
SELECT VALUE ->likes FROM person;

```


## kill.mdx

---
sidebar_position: 14
sidebar_label: KILL
title: KILL statement | SurrealQL
description: The KILL statement is used to terminate a running live query.
---

import Since from "@components/shared/Since.astro";
import RailroadDiagram from "@components/RailroadDiagram.astro";
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

# `KILL` statement

The `KILL` statement is used to terminate a running live query.

While the `KILL` statement does accept a value type, this value must resolve to a UUID. Consequently, it will accept a string literal of a UUID or a param.

### Statement syntax

<Tabs syncKey="kill-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
KILL @value;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const killAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "KILL" },
        { type: "NonTerminal", text: "@value" },
        { type: "Terminal", text: ";" },
      ],
    },
  ],
};

<RailroadDiagram ast={killAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

### Basic usage

The `KILL` statement expects the UUID of a running [live select](/docs/surrealql/statements/live) query to be passed. This UUID can be found in the output of the `LIVE` statement, and can thereafter be passed into a `KILL` statement once it is no longer needed.

```surql
LIVE SELECT DIFF FROM person;
-- output: u'0189d6e3-8eac-703a-9a48-d9faa78b44b9'

-- Some time later...
KILL u"0189d6e3-8eac-703a-9a48-d9faa78b44b9";
```

The `KILL` statement also allows for parameters to be used.

```surql
-- Define the parameter
LET $live_query_id = u"0189d6e3-8eac-703a-9a48-d9faa78b44b9";
-- Use the parameter
KILL $live_query_id;
```

Using the `KILL` statement on a UUID that does not correspond to a running live query will generate an error.

```surql
LET $rand = rand::uuid();
KILL $rand;
KILL u'9276b05b-e59a-49cd-9dd1-17c6fd15c28f';
```

```surql title="Output"
"Can not execute KILL statement using id '$rand'"
"Can not execute KILL statement using id 'u'9276b05b-e59a-49cd-9dd1-17c6fd15c28f''"
```

## Kill notifications

<Since v="v3.0.0" />

A separate notification is sent out when a `KILL` statement is enacted on a live query ID.

```surql
LIVE SELECT * FROM person;

-- Output is a UUID:
-- u'cf447091-9463-4d75-b32a-08513eb2a07c'

KILL u'cf447091-9463-4d75-b32a-08513eb2a07c';
```

```surql title="Output"
-- Query 1
NONE

-- Notification (action: Killed, live query ID: cf447091-9463-4d75-b32a-08513eb2a07c)
NONE
```

## let.mdx

---
sidebar_position: 15
sidebar_label: LET
title: LET statement | SurrealQL
description: The LET statement sets and stores a value which can then be used in a subsequent query.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `LET` Statement

The `LET` statement allows you to create parameters to store any value, including the results of queries or the outputs of expressions. These parameters can then be referenced throughout your SurrealQL code, making your queries more dynamic and reusable.

## Syntax

The syntax for the `LET` statement is straightforward. The parameter name is prefixed with a `$` symbol.

<Tabs syncKey="let-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
LET $@parameter [: @type_name] = @value;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const letAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "LET" },
      { type: "Terminal", text: "$" },
      { type: "NonTerminal", text: "@parameter" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: ":" }, { type: "NonTerminal", text: "@type_name" } ] } },
      { type: "Terminal", text: "=" },
      { type: "NonTerminal", text: "@value" },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={letAst} className="my-6" />

  </TabItem>
</Tabs>

## Example Usage

### Basic Parameter Assignment

You can use the `LET` statement to store simple values or query results. For example, storing a string value and then using it in a `CREATE` statement:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: person:qt3itwoql7oodlg3n077, name: 'tobie' }]"
skip-record-id-key = true

*/

-- Define the parameter
LET $name = "tobie";
-- Use the parameter
CREATE person SET name = $name;
```

### Storing Query Results

The `LET` statement is also useful for storing the results of a query, which can then be used in subsequent operations:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[]"

*/

-- Define the parameter
LET $adults = SELECT * FROM person WHERE age > 18;
-- Use the parameter
UPDATE $adults SET adult = true;
```

### Conditional Logic with `IF ELSE`

SurrealQL allows you to define parameters based on conditional logic using `IF ELSE` statements:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "'integer'"

*/

LET $num = 10;

LET $num_type =
         IF type::is_int($num)     { "integer" }
    ELSE IF type::is_decimal($num) { "decimal" }
    ELSE IF type::is_float($num)   { "float"   };

RETURN $num_type;
-- 'integer'
```

## Anonymous Functions

You can define anonymous functions also known as closures using the `LET` statement. These functions can be used to encapsulate reusable logic and can be called from within your queries. Learn more about [anonymous functions](/docs/surrealql/datamodel/closures) in the Data model section.


## Pre-Defined and Protected Parameters

SurrealDB comes with [pre-defined parameters](/docs/surrealql/parameters) that are accessible in any context. However, parameters created using `LET` are not accessible within the scope of these pre-defined parameters.

Furthermore, some pre-defined parameters are protected and cannot be overwritten using `LET`:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "'Before!'"

[[test.results]]
value = "[{ before: { id: person:1 } }, { before: { id: person:2 } }, { before: { id: person:3 } }, { before: { id: person:qt3itwoql7oodlg3n077, name: 'tobie' } }]"
skip-record-id-key = true

[[test.results]]
value = "'Before!'"

*/

LET $before = "Before!";

-- Returns ["Before!"];
RETURN $before;

-- Returns the `person` records before deletion
DELETE person RETURN $before;

-- Returns "Before!" again
RETURN $before;
```

Attempting to redefine protected parameters will result in an error:

```surql
/**[test]

[[test.results]]
error = ""'auth' is a protected variable and cannot be set""

[[test.results]]
error = ""'session' is a protected variable and cannot be set""

*/

LET $auth = 1;
LET $session = 10;
```

```surql title="Output"
-------- Query 1 (0ns) --------

"'auth' is a protected variable and cannot be set"

-------- Query 2 (0ns) --------

"'session' is a protected variable and cannot be set"
```

## Typed LET statements

<Since v="v2.0.0" />

Type safety in a `LET` statement can be ensured by adding a `:` (a colon) and the type name after the `LET` keyword.

```surql
/**[test]

[[test.results]]
error = ""Tried to set `$number`, but couldn't coerce value: Expected `int` but found `'9'`""

*/

LET $number: int = "9";
```

```surql title="Output"
"Tried to set `$number`, but couldn't coerce value: Expected `int` but found `'9'`"
```

### Taking advantage of type safety

Using typed `LET` statements is a good practice when prototyping code or when getting used to SurrealQL for the first time. Take the following example that attempts to count the number of `true` values in a field by filtering out values that are not `true`, without noticing that the field actually contains strings instead of booleans. The query output ends up being 0, rather than the expected 2.

```surql
/**[test]

[[test.results]]
value = "[{ id: some:record, vals: ['true', 'false', 'true'] }]"

[[test.results]]
value = "0"

*/

CREATE some:record SET vals = ["true", "false", "true"];
some:record.vals.filter(|$val| $val = true).len();
```

```surql title="Output"
0
```

Breaking this into multiple typed `LET` statements shows the error right away.

```surql
LET $vals: array<bool> = some:record.vals;
LET $len: number = $vals.filter(|$val| $val = true).len();
$len;
```

```surql title="Output"
-------- Query 1 --------

"Tried to set `$vals`, but couldn't coerce value: Expected `bool` but found `'true'` when coercing an element of `array<bool>`"

-------- Query 2 --------

'There was a problem running the filter() function. no such method found for the none type'

-------- Query 3 --------

NONE
```

With the location of the error in clear sight, a fix is that much easier to implement.

```surql
-- Use .map() to turn each string into a bool
LET $vals: array<bool> = some:record.vals.map(|$val| <bool>$val);
LET $len: number = $vals.filter(|$val| $val = true).len();
$len;
```

```surql title="Output"
2
```

### Typed literal statements

Multiple possible types can be specified in a `LET` statement by adding a `|` (vertical bar) in between each possible type.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

LET $number: int | string = "9";
```

Even complex types such as objects can be included in a typed `LET` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

LET $error_info: string | { error: string } = { error: "Something went wrong plz help" };
```

For more information on this pattern, see the page on [literals](/docs/surrealql/datamodel/literals).

## Conclusion

The `LET` statement in SurrealDB is versatile, allowing you to store values, results from subqueries, and even define anonymous functions. Understanding how to use `LET` effectively can help you write more concise, readable, and maintainable queries.


## live.mdx

---
sidebar_position: 16
sidebar_label: LIVE
title: LIVE SELECT statement | SurrealQL
description: The LIVE SELECT statement can be used to initiate a real-time selection from a table, including the option to apply filters.
---

import Since from "@components/shared/Since.astro";
import RailroadDiagram from "@components/RailroadDiagram.astro";
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

# `LIVE SELECT` statement

Live Queries is a feature that allows you to listen for creations, updates and deletions to specific records you are interested in or entire tables. 

The `LIVE SELECT` statement can be used to initiate a real-time selection from a table, including the option to apply filters.

In practical terms, when you execute a `LIVE SELECT` query, it triggers an ongoing session that captures any subsequent changes to the data in real-time. These changes are then immediately transmitted to the client, ensuring that the client is consistently updated with the latest data modifications.

> [!IMPORTANT]
> Currently, `LIVE SELECT` is only supported in single-node deployments, with multi-node support being actively developed.

### Statement syntax

<Tabs syncKey="live-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
LIVE SELECT
	[
		[ VALUE ] @fields ... [ AS @alias ]
		| DIFF
	]
	FROM @targets
	[ WHERE @conditions ]
	[ FETCH @fields ... ]
;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const liveAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "LIVE" },
      { type: "Terminal", text: "SELECT" },
      { type: "Choice", index: 1, children: [
        { type: "Sequence", children: [
          { type: "Optional", child: { type: "Terminal", text: "VALUE" } },
          { type: "Sequence", children: [
            { type: "NonTerminal", text: "@field" },
            { type: "ZeroOrMore", child: { type: "Sequence", children: [ { type: "Terminal", text: "," }, { type: "NonTerminal", text: "@field" } ] } }
          ] },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "AS" }, { type: "NonTerminal", text: "@alias" } ] } }
        ] },
        { type: "Terminal", text: "DIFF" }
      ] },
      { type: "Terminal", text: "FROM" },
      { type: "NonTerminal", text: "@targets" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "WHERE" }, { type: "NonTerminal", text: "@conditions" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "FETCH" }, { type: "NonTerminal", text: "@fields ..." } ] } },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={liveAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

### Basic usage

By default, SurrealDB will push the entire record over the websocket when created or updated, and just the record's ID when deleted.

```surql
/**[test]

[[test.results]]
value = "u'0f0d0e40-f371-4a68-8ee0-6d5eb3565b76'"
skip-uuid = true

*/

LIVE SELECT * FROM person;

-- 'b1f1d115-ad0f-460d-8cbf-dbc7ce48851c'
```

The result of the above query will be a UUID. This UUID is the Live Query Unique ID, and is used to differentate between different Live Queries. You will want to keep track of this ID, so that you can differentiate between different notifications being received after this query. You can also use this UUID to [KILL](/docs/surrealql/statements/kill) (stop) the Live Query. The protocol will then send messages that are of a Notification format.

You can find an example of such a message in the [Live Query WebSocket protocol](/docs/surrealdb/integration/rpc#live-websocket-only) description.

### Diff

When using the `DIFF` mode, updates will be sent in the form of an array with [JSON Patch](https://jsonpatch.com/) messages.

```surql
/**[test]

[[test.results]]
value = "u'0f0d0e40-f371-4a68-8ee0-6d5eb3565b76'"
skip-uuid = true

*/

LIVE SELECT DIFF FROM person;

-- 'b87cbb0d-ca15-4f0a-8f86-caa680672aa5'
```

### Filter the live query

You can optionally apply filters with the `WHERE` clause.

```surql
/**[test]

[[test.results]]
value = "u'0f0d0e40-f371-4a68-8ee0-6d5eb3565b76'"
skip-uuid = true

*/

LIVE SELECT * FROM person WHERE age > 18;
```

## Consistency Guarantees

When using Live Queries, it is important to understand the ordering of messages and events when many clients and transactions are running in paralllel. Notifications on live queries are only published for committed transactions.

While a best effort is made to assure ordering is correct, a strict correctness is not yet in place for a full guarantee. As such that some messages may be received out of order from their commit order. However, transactions that are committed from the same client will always be in order.

Security enforcement is always evaluated per notification and will reflect the value of authorisation at the time of publishing the notification. This means that if a transaction is committed, after which the authorisation immediately changes for the live query receiver, the receiver will get the notification under the new rules.

## Fetching inside live queries

<Since v="v2.2.0" />

The `FETCH` clause can be used inside live queries as well.

```surql
/**[test]

[[test.results]]
value = "u'0f0d0e40-f371-4a68-8ee0-6d5eb3565b76'"
skip-uuid = true

*/

LIVE SELECT * FROM person WHERE age > 18 FETCH friends;
```

## Other notes

Currently, it is not possible to use [parameters inside of Live Queries](https://github.com/surrealdb/surrealdb/issues/4026).

```surql
/**[test]

[[test.results]]
value = "u'0f0d0e40-f371-4a68-8ee0-6d5eb3565b76'"
skip-uuid = true

*/

LIVE SELECT * FROM person WHERE $field > $value;
```

It is possible to have parameters for the table reference.

```surql
LIVE SELECT * FROM $table WHERE field > 50;
```


## Parameters in `LIVE SELECT` statements

<Since v="v3.0.0" />

Parameters can also be used inside a `LIVE SELECT` statement.

```surql
LET $table = 'measurement';
LET $location = 'Tallinn';
LIVE SELECT * FROM type::table($table) WHERE location == $location;
```


## rebuild.mdx

---
sidebar_position: 17
sidebar_label: REBUILD
title: REBUILD statement | SurrealQL
description: The REBUILD statement is used to rebuild resources.
---

import Since from '@components/shared/Since.astro'
import SurrealistMini from "@components/SurrealistMini.astro"
import RailroadDiagram from "@components/RailroadDiagram.astro"
import Tabs from "@components/Tabs/Tabs.astro"
import TabItem from "@components/Tabs/TabItem.astro"

# `REBUILD` statement



The `REBUILD` statement is used to rebuild resources in SurrealDB. It is usually used in relation to a specified [Index](/docs/surrealql/statements/define/indexes) to optimize performance. It is useful to rebuild indexes because sometimes [HNSW](/docs/surrealql/statements/define/indexes#hnsw-hierarchical-navigable-small-world) index performance can degrade due to frequent updates.

Rebuilding the index will ensure the index is fully optimized.

> [!NOTE]
> Rebuilds are concurrent or sync based on how the index is defined. For example, if you define an index with the `CONCURRENTLY` option, the rebuild will be concurrent. Please see the [`CONCURRENTLY` clause](/docs/surrealql/statements/define/indexes#using-concurrently-clause) section for more information. 

### Statement syntax

<Tabs syncKey="rebuild-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
REBUILD [
	INDEX [ IF EXISTS ] @name ON [ TABLE ] @table
]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const rebuildAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "REBUILD" },
      { type: "Sequence", children: [
        { type: "Terminal", text: "INDEX" },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } },
        { type: "NonTerminal", text: "@name" },
        { type: "Terminal", text: "ON" },
        { type: "Optional", child: { type: "Terminal", text: "TABLE" } },
        { type: "NonTerminal", text: "@table" }
      ] },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={rebuildAst} className="my-6" />

  </TabItem>
</Tabs>

> [!NOTE]
> The `IF EXISTS` and TABLE clauses are optional.

## Example usage

For example, if you have a table called `book` and you have an index called `uniq_isbn` on the `isbn` field, you can rebuild the index using the following query:

```surql
REBUILD INDEX uniq_isbn ON book;
```


<SurrealistMini
url="https://app.surrealdb.com/mini?query=CREATE+book%3A1+SET+title+%3D+%27Rust+Web+Programming%27%2C+isbn+%3D+%27978-1803234694%27%2C+author+%3D+%27Jon+Doe%27%3B%0A%0A%2F%2F+Define+a+unique+index+on+the+isbn+field%0ADEFINE+INDEX+uniq_isbn+ON+book+FIELDS+isbn+UNIQUE%3B%0A%0A%2F%2F+Rebuild+this+index+incase+of+more+updates%0AREBUILD+INDEX+IF+EXISTS+uniq_isbn+ON+book%3B%0A%0A%2F%2F+Check+that+the+index+has+been+created%0AINFO+FOR+TABLE+book%3B%0A%0AREBUILD+INDEX+IF+EXISTS+idx_author+ON+book%3B%0A%0AREBUILD+INDEX+IF+EXISTS+ft_title+ON+book%3B%0A%0A%2F%2F+Define+index+on+the+author+field+%0ADEFINE+INDEX+idx_author+ON+book+FIELDS+author%3B%0A%0A%2F%2F+Define+an+analyzer+which+has+blank+and+class+Tokenizers+and+converts+the+tokens+to+lowercase+%0ADEFINE+ANALYZER+simple+TOKENIZERS+blank%2Cclass+FILTERS+lowercase%3B%0A%0ADEFINE+INDEX+ft_title+ON+book+FIELDS+title+SEARCH+ANALYZER+simple+BM25+HIGHLIGHTS%3B%0A%0AREBUILD+INDEX+uniq_isbn+ON+book%3B%0A%0AREBUILD+INDEX+idx_author+ON+book%3B%0A%0AREBUILD+INDEX+ft_title+ON+book%3B%0A%0A%2F%2F+Check+that+the+index+has+been+created%0AINFO+FOR+TABLE+book%3B%0A%0A%2F%2FChecks+whether+the+term+RUST+IS+found+in+a+full-text+indexed+field.%0ASELECT+*+FROM+book+WHERE+title+%40%40+%27Rust%27%3B&orientation=horizontal"
/>

### Using if exists clause



The following queries show an example of how to rebuild resources using the `IF EXISTS` clause, which will only rebuild the resource if it exists.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

REBUILD INDEX IF EXISTS uniq_isbn ON book;
```






## relate.mdx

---
sidebar_position: 18
sidebar_label: RELATE
title: RELATE statement | SurrealQL
description: The RELATE statement can be used to generate graph edges between two records in the database.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `RELATE` statement

The `RELATE` statement can be used to generate graph edges between two records in the database. This allows you to traverse related records efficiently without needing to pull data from multiple tables and merging that data together using SQL JOINs.

Edges created using the RELATE statement are nearly identical to tables created using other statements, and can contain data. The key differences are that:

- Edge tables are deleted once there are no existing relationships left.
- Edge tables have two required fields `in` and `out`, which specify the directions of the relationships. These cannot be modified in schema declarations except to specify that they must be of a certain record type or to [add assertions](/docs/surrealql/statements/define/field#asserting-rules-on-fields).

Otherwise, edge tables behave like normal tables in terms of [updating](/docs/surrealql/statements/update), [defining a schema](/docs/surrealql/statements/define/table) or [indexes](/docs/surrealql/statements/define/indexes).

Another option for connecting data is using [record links](/docs/surrealql/datamodel/records). Record links consist of a field with record IDs that serve as unidirectional links by default, or bidirectional links if reference tracking is used. The key differences are that graph relations have the following benefits over record links:

- Graph relations are kept in a separate table as opposed to a field inside a record.
- Graph relations allow you to store data alongside the relationship.
- Graph relations have their own syntax that makes it easy to build and visualize edge queries.

Graph relations offer built-in bidirectional querying and referential integrity. As of SurrealDB 2.2.0, record links also offer these two advantages if they are defined inside a [`DEFINE FIELD`](/docs/surrealql/statements/define/field) statement using the `REFERENCES` clause. For more information, see [the page on record references](/docs/surrealql/datamodel/references).

### Statement syntax

<Tabs syncKey="relate-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
RELATE [ ONLY ] @from_record -> @table -> @to_record
	[ CONTENT @value
	  | SET @field = @value ...
	]
	[ RETURN NONE | RETURN BEFORE | RETURN AFTER | RETURN DIFF | RETURN @statement_param, ... | RETURN VALUE @statement_param ]
	[ TIMEOUT @duration ]
;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const relateAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "RELATE" },
      { type: "Optional", child: { type: "Terminal", text: "ONLY" } },
      { type: "NonTerminal", text: "@from_record" },
      { type: "Terminal", text: "->" },
      { type: "NonTerminal", text: "@table" },
      { type: "Terminal", text: "->" },
      { type: "NonTerminal", text: "@to_record" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [
        { type: "Sequence", children: [ { type: "Terminal", text: "CONTENT" }, { type: "NonTerminal", text: "@value" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "SET" }, { type: "NonTerminal", text: "@field = @value ..." } ] }
      ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "RETURN" }, { type: "Choice", index: 1, children: [
        { type: "Terminal", text: "NONE" },
        { type: "Terminal", text: "BEFORE" },
        { type: "Terminal", text: "AFTER" },
        { type: "Terminal", text: "DIFF" },
        { type: "NonTerminal", text: "@statement_param, ..." },
        { type: "Sequence", children: [ { type: "Terminal", text: "VALUE" }, { type: "NonTerminal", text: "@statement_param" } ] }
      ] } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TIMEOUT" }, { type: "NonTerminal", text: "@duration" } ] } },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={relateAst} className="my-6" />

  </TabItem>
</Tabs>

> [!NOTE]
> `RELATE` will create a relation regardless of whether the records to relate to exist or not. As such, it is advisable to [create the records](/docs/surrealql/statements/create) you want to relate to before using `RELATE`, or to at least ensure that they exist before making a query on the relation. If the records to relate to don't exist, a query on the relation will still work but will return an empty array. To override this behaviour and return an error if no records exist to relate, you can use a [`DEFINE TABLE`](/docs/surrealql/statements/define/table) statement that includes the `ENFORCED` keyword.

### Example usage

#### Basic usage

The following query shows the basic structure of the `RELATE` statement, which creates a relationship between a record in the `person` table and a record in the `article` table.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:aristotle }, { id: article:on_sleep_and_sleeplessness }]"

[[test.results]]
value = "[{ id: wrote:2m9047thgn9j9oc05iqh, in: person:aristotle, out: article:on_sleep_and_sleeplessness }]"
skip-record-id-key = true

*/

CREATE person:aristotle, article:on_sleep_and_sleeplessness;
RELATE person:aristotle->wrote->article:on_sleep_and_sleeplessness;
```

```surql title="Response"
[
	{
		id: wrote:bpbrj5kd7smu3ahlf55r,
		in: person:aristotle,
		out: article:on_sleep_and_sleeplessness
	}
]
```

There is no relationship information stored in either the `person` or `article` table.

```surql
SELECT * FROM person, article;
```

```surql title="Response"
[
	{
		id: person:aristotle
	},
	{
		id: article:on_sleep_and_sleeplessness
	}
]
```

Instead, an edge table (in this case a table called `wrote`) stores the relationship information.

```surql
SELECT * FROM wrote;
```

The structure `in -> id -> out` mirrors the record IDs from the `RELATE` statement, with the addition of the automatically generated ID for the `wrote` edge table.

```surql title="Response"
[
	{
		id: wrote:bpbrj5kd7smu3ahlf55r,
		in: person:aristotle,
		out: article:on_sleep_and_sleeplessness
	}
]
```

The same structure can be used in a `SELECT` query, as well as directly from a record ID.

```surql
-- Aristotle's id and the articles he wrote
SELECT id, ->wrote->article FROM person:aristotle;
-- Every `person`'s id and written articles
-- Same output as above as the database has a single `person` record
SELECT id, ->wrote->article FROM person;
-- Directly follow the path from Aristotle to his written articles
RETURN person:aristotle->wrote->article;
```

```surql title="Response"
-------- Query --------

[
	{
		"->wrote": {
			"->article": [
				article:on_sleep_and_sleeplessness
			]
		},
		id: person:aristotle
	}
]

-------- Query --------

[
	article:on_sleep_and_sleeplessness
]
```

By default, the edge table gets created as a schemaless table when you execute the `RELATE` statement. You can make the table schemafull by [defining a schema](/docs/surrealql/statements/define/table).

A common use case is to make sure only unique relationships get created. You can do that by [defining an index](/docs/surrealql/statements/define/indexes).

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE INDEX unique_relationships
    ON TABLE wrote
    COLUMNS in, out UNIQUE;
```

As edge tables are bidirectional by default, there is nothing stopping a query like the following in which an article writes a person instead of the other way around.

```surql
/**[test]

[[test.results]]
value = "[{ id: wrote:ymz181gxo9ugcrse7nmo, in: article:on_sleep_and_sleeplessness, out: person:aristotle }]"
skip-record-id-key = true

*/

RELATE article:on_sleep_and_sleeplessness->wrote->person:aristotle;
```

To enforce unidirectional relationships, you can restrict the type definition using a [`DEFINE FIELD`](/docs/surrealql/statements/define/field) definition.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

DEFINE FIELD in  ON TABLE wrote TYPE record<person>;
DEFINE FIELD out ON TABLE wrote TYPE record<article>;
```

#### Always two records there are - no more, no less

An edge table will always include exactly one record for the `in` field and one record for the `out` field.

Knowing this, one would assume that a `RELATE` statement like the following would fail as it seems to be attempting to insert two `cat` records at the `in` field.

```surql
/**[test]

[[test.results]]
value = "[{ id: cat:mr_meow }, { id: cat:mrs_meow }, { id: cat:kitten }]"

[[test.results]]
value = "[{ id: parent_of:a3pm72b0xvltb61s5poy, in: cat:mr_meow, out: cat:kitten }, { id: parent_of:7h1wd7vu01o0kwmns9k1, in: cat:mrs_meow, out: cat:kitten }]"
skip-record-id-key = true

*/

CREATE cat:mr_meow, cat:mrs_meow, cat:kitten;
RELATE [cat:mr_meow, cat:mrs_meow]->parent_of->cat:kitten;
```

However, the query works just fine. Instead of trying to create a single `parent_of` graph edge, it will create one for each record in the first array: one between `cat:mr_meow` and `cat:kitten`, and another between `cat:mrs_meow` and `cat:kitten`.


```surql title="Response"
[
	{
		id: parent_of:uahudi4qr68k640fcjbg,
		in: cat:mr_meow,
		out: cat:kitten
	},
	{
		id: parent_of:hi79yfazjppv8b3kyi36,
		in: cat:mrs_meow,
		out: cat:kitten
	}
]
```

Similarly, a `RELATE` statement that involves two arrays will return a number of graph edges equal to their product (2 * 2 in this case):

```surql
/**[test]

[[test.results]]
value = "[{ id: cat:kitten2 }]"

[[test.results]]
value = "[{ id: parent_of:bsgtxiltlv2pd9hf5eqz, in: cat:mr_meow, out: cat:kitten }, { id: parent_of:cr7chl0lot6g4gfj84fe, in: cat:mr_meow, out: cat:kitten2 }, { id: parent_of:y3dtskf9sme2y0hmumlu, in: cat:mrs_meow, out: cat:kitten }, { id: parent_of:lc39ib0lbzw0x1azkee9, in: cat:mrs_meow, out: cat:kitten2 }]"
skip-record-id-key = true

*/

CREATE cat:kitten2;
RELATE [cat:mr_meow, cat:mrs_meow]->parent_of->[cat:kitten, cat:kitten2];
```

```surql title="Response"
[
	{
		id: parent_of:ysbab20nv5568ogba6ns,
		in: cat:mr_meow,
		out: cat:kitten
	},
	{
		id: parent_of:0ltm6xr94pkblyxf0m6c,
		in: cat:mr_meow,
		out: cat:kitten2
	},
	{
		id: parent_of:71cfl0nvj5frve0r1npv,
		in: cat:mrs_meow,
		out: cat:kitten
	},
	{
		id: parent_of:4gbid7nzo6cwr1t8k090,
		in: cat:mrs_meow,
		out: cat:kitten2
	}
]
```

### Adding data using `SET` and `CONTENT`

Graph edges are standalone tables that can hold other fields besides the default `in`, `out`, and `id`. These can be added during a `RELATE` statement or during an `UPDATE` in the same manner as any other SurrealDB table.

Let's look at the two ways you can add record data in the `RELATE` statement. Both of these queries will produce the same result.

```surql
/**[test]

[[test.results]]
value = "[{ id: wrote:oapxwx4gdbpfsz1yxhih, in: person:l19zjikkw1p1h9o6ixrg, metadata: { location: 'Tallinn', time_written: d'2025-10-08T04:28:09.169160Z' }, out: article:8nkk6uj4yprt49z7y3zm }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: wrote:fvrxtrn5cj2fnz2gliyw, in: person:l19zjikkw1p1h9o6ixrg, metadata: { location: 'Tallinn', time_written: d'2025-10-08T04:28:09.171256Z' }, out: article:8nkk6uj4yprt49z7y3zm }]"
skip-record-id-key = true

*/

RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
    SET 
		metadata.time_written = time::now(),
		metadata.location = "Tallinn";


RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
	CONTENT {
		metadata: {
			time_written: time::now(),
			location: "Tallinn"
		}
	};
```

```surql title="Response"
[
	{
		id: wrote:rva8hentypdu8lcgwjmf,
		in: person:l19zjikkw1p1h9o6ixrg,
		metadata: {
			location: 'Tallinn',
			time_written: d'2024-11-26T01:52:01.169Z'
		},
		out: article:8nkk6uj4yprt49z7y3zm
	}
]
```

Here is an example of the graph edge being updated in the same way as any other SurrealDB record:

```surql
-- Add a small synopsis composed of the table name and article ID
UPDATE wrote SET
    metadata.description = record::tb(out) + ' written by ' + <string>in;
```

```surql title="Response"
[
	{
		id: wrote:k9d8ynbfxgb8jqjv2ob5,
		in: person:l19zjikkw1p1h9o6ixrg,
		metadata: {
			description: 'article written by person:l19zjikkw1p1h9o6ixrg',
			location: 'Tallinn',
			time_written: d'2024-11-26T01:53:51.350Z'
		},
		out: article:8nkk6uj4yprt49z7y3zm
	}
]
```

### Passing variables in `CONTENT` and `SET`



You can also pass variables in the `CONTENT` block. This is useful when you want to pass a variable that is not a record ID.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: wrote:9zyb1fytya4cykc47j0o, in: person:l19zjikkw1p1h9o6ixrg, out: article:8nkk6uj4yprt49z7y3zm, time: { written: d'2025-10-08T04:28:53.000469Z' } }]"
skip-record-id-key = true

*/

LET $time = time::now();
RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
    CONTENT {
        time: {
            written: $time
        }
    };
```

```surql title="Response"
    {
        "id": "wrote:ctwsll49k37a7rmqz9rr",
        "in": "person:l19zjikkw1p1h9o6ixrg",
        "out": "article:8nkk6uj4yprt49z7y3zm",
        "time": {
            "written": "2021-09-29T14:00:00Z"
        }
    }
```

Below is an example of how you can pass a variable in the `SET` block:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: wrote:o6kz4fqcqgqok1xqoqm0, in: person:l19zjikkw1p1h9o6ixrg, out: article:8nkk6uj4yprt49z7y3zm, time: { written: d'2025-10-08T04:29:36.089521Z' } }]"
skip-record-id-key = true

*/

LET $time = time::now();

RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
    SET time.written = $time;
```

```surql title="Response"
{
	"id": "wrote:ctwsll49k37a7rmqz9rr",
	"in": "person:l19zjikkw1p1h9o6ixrg",
	"out": "article:8nkk6uj4yprt49z7y3zm",
	"time": {
		"written": "2021-09-29T14:00:00Z"
	}
}
```


### Creating a single relation with the `ONLY` keyword

Using the ONLY keyword, just an object for the relation in question will be returned. This, instead of an array with a single object.

```surql
/**[test]

[[test.results]]
value = "{ id: wrote:lrddm6wuqashha9wjv6s, in: person:l19zjikkw1p1h9o6ixrg, out: article:8nkk6uj4yprt49z7y3zm }"
skip-record-id-key = true

*/

RELATE ONLY person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm;
```

```surql title="Response"
{
	id: wrote:k9f1rqn3oikolr1560u3,
	in: person:l19zjikkw1p1h9o6ixrg,
	out: article:8nkk6uj4yprt49z7y3zm
}
```

### Using [`LET`](/docs/surrealql/statements/let) parameters in RELATE statements

You can also use [parameters](/docs/surrealql/parameters) to specify the record IDs.

```surql
-- These two statements store the result of the subquery in a parameter
-- The subquery returns an array of IDs
LET $person =  (SELECT VALUE id FROM person);
LET $article = (SELECT VALUE id FROM article);

-- This statement creates a relationship record for every combination of Record IDs
-- Such that if we have 10 records each in the person and article table
-- We get 100 records in the wrote edge table (10*10 = 100)
-- In this case it would mean that each article would have 10 authors
RELATE $person->wrote->$article SET time.written = time::now();
```

### Modifying output with the [`RETURN`](/docs/surrealql/statements/return) clause

By default, the relate statement returns the record value once the changes have been made. To change the return value of each record, specify a RETURN clause, specifying either `NONE`, `BEFORE`, `AFTER`, `DIFF`, or a comma-separated list of specific fields to return.

```surql
/**[test]

[[test.results]]
value = "[]"
skip-record-id-key = true
skip-datetime = true

[[test.results]]
value = "[{ op: 'replace', path: '', value: { id: wrote:me0vergol3jkvn7me15x, in: person:l19zjikkw1p1h9o6ixrg, out: article:8nkk6uj4yprt49z7y3zm, time: { written: d'2025-10-08T04:30:34.214973Z' } } }]"
skip-record-id-key = true
skip-datetime = true

[[test.results]]
value = "NONE"
skip-record-id-key = true
skip-datetime = true

[[test.results]]
value = "[{ id: wrote:pwatbr0s0f2ddwcwqls7, in: person:l19zjikkw1p1h9o6ixrg, out: article:8nkk6uj4yprt49z7y3zm, time: { written: d'2025-10-08T04:30:34.218213Z' } }]"
skip-record-id-key = true
skip-datetime = true

[[test.results]]
value = "[{ time: { written: d'2025-10-08T04:30:34.219729Z' } }]"
skip-datetime = true

[[test.results]]
value = "[{ written: d'2025-10-08T04:30:34.221383Z' }]"
skip-datetime = true

*/

-- Don't return any result
RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
    SET time.written = time::now()
    RETURN NONE;

-- Return the changeset diff
RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
    SET time.written = time::now()
    RETURN DIFF;

-- Return the record before changes were applied
RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
    SET time.written = time::now()
    RETURN BEFORE;

-- Return the record after changes were applied (the default)
RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
    SET time.written = time::now()
    RETURN AFTER;

-- Return a specific field only from the updated records
RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
    SET time.written = time::now()
    RETURN time;

-- Return only the value of a specific field without the field name
RELATE person:l19zjikkw1p1h9o6ixrg->wrote->article:8nkk6uj4yprt49z7y3zm
    SET time.written = time::now()
    RETURN VALUE time;
```

### Using the `TIMEOUT` clause

Adding the `TIMEOUT` keyword to specify a timeout duration for the statement can be useful when processing a large result set with many interconnected records. If the statement continues beyond this duration, then the transaction will fail, and the statement will return an error.

```surql
/**[test]

[[test.results]]
value = "[]"

*/

-- Cancel this conditional filtering based on graph edge properties
-- if not finished within 5 seconds
SELECT * FROM person WHERE ->knows->person->(knows WHERE influencer = true) TIMEOUT 5s;
```

Using a `TIMEOUT` is particularly useful when experimenting with complex queries with an extent that is difficult to imagine, especially if the query [is recursive](#recursive-graph-queries).

### Deleting graph edges

You can also delete graph edges between two records in the database by using the [DELETE statement](/docs/surrealql/statements/delete).

For example the graph edge below:

```surql
/**[test]

[[test.results]]
value = "[{ id: bought:z8xw3m3mmbg79tr21oj7, in: person:tobie, out: product:iphone }]"
skip-record-id-key = true

*/

RELATE person:tobie->bought->product:iphone;
```

```surql title="Response"

[
	{
		id: bought:ctwsll49k37a7rmqz9rr,
		in: person:tobie,
		out: product:iphone
	}
]
```

Can be deleted by:

```surql
DELETE person:tobie->bought WHERE out=product:iphone RETURN BEFORE;
```

As mentioned above, a graph edge will also automatically be deleted if it is no longer connected to a record at both `in` and `out`.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:one }, { id: person:two }, { id: person:three }]"

[[test.results]]
value = "[{ id: likes:qv1sjo66j48irzu1hsik, in: person:one, out: person:two }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: likes:w4n1on9bim3w1d7r3tg3, in: person:two, out: person:three }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: likes:1jopms33qaetjmtv3ygs, in: person:three, out: person:one }]"
skip-record-id-key = true

[[test.results]]
value = "[]"

[[test.results]]
value = "[{ id: likes:1jopms33qaetjmtv3ygs, in: person:three, out: person:one }]"
skip-record-id-key = true

*/

-- Create three people
CREATE person:one, person:two, person:three;

-- And a love triangle involving them all
RELATE person:one  ->likes->person:two;
RELATE person:two  ->likes->person:three;
RELATE person:three->likes->person:one;

-- Person two moves to Venus permanently, so delete
DELETE person:two;

-- Only one `likes` relationship is left
SELECT * FROM likes;
```

```surql title="Output"
[
	{
		id: likes:55szjin5yfqwl4sbmy1f,
		in: person:three,
		out: person:one
	}
]
```


### Using RELATE on non-existent records

As mentioned at the top of the page, `RELATE` can be used for records that do not yet exist. While this behaviour can be overridden by using the `ENFORCED` keyword, it can be useful in certain situations.

For example, the `VALUE` clause inside a [`DEFINE FIELD`](/docs/surrealql/statements/define/field) statement is calculated every time a record is altered (that is, every time it is created or updated). If this value depends on a graph edge, creating the record first will cause `VALUE` to calculate it based on a nonexistent path.

In the following example, a `house` table has a field called `has_road_access` that depends on whether any `->has_road` paths return an output that is not empty. Meanwhile, the city has a new road under construction but no houses are present and their details have not been set yet.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: road:ohl5o501eh7ylfzcqwm3, length: 10.5f, name: 'Dalhurst Way' }]"
skip-record-id-key = true

*/

-- Returns true if $this->has_road path is not empty
DEFINE FIELD has_road_access ON TABLE house VALUE !!$this->has_road->road;
CREATE road SET name = "Dalhurst Way", length = 10.5;
```

As the addresses of the upcoming houses have been decided, the `->has_road` path can be set ahead of time by giving the `house` records an ID based on their exact address.

```surql
LET $road = SELECT * FROM ONLY road WHERE name = "Dalhurst Way" LIMIT 1;
RELATE [
    house:["Dalhurst Way", 218],
    house:["Dalhurst Way", 222],
    house:["Dalhurst Way", 226],
]->has_road->$road;
```

Later on, two new houses are completed in the city and registered in the database. As the path to `house:["Dalhurst Way", 218]` has already been set up, the `has_road_access` field will evaluate to `true`, while the other house in the middle of nowhere will evaluate to `false`.

```surql
/**[test]

[[test.results]]
value = "[{ bedrooms: 5, floors: 2, id: house:['Dalhurst Way', 218] }]"

[[test.results]]
value = "[{ bedrooms: 12, floors: 4, id: house:['Middle of nowhere', 0] }]"

*/

CREATE house:["Dalhurst Way", 218] SET floors = 2, bedrooms = 5;
CREATE house:["Middle of nowhere", 0] SET floors = 4, bedrooms = 12;
```

```surql
-------- Query --------

[
	{
		bedrooms: 5,
		floors: 2,
		id: house:[
			'Dalhurst Way',
			218
		],
		street: []
	}
]

-------- Query --------

[
	{
		bedrooms: 12,
		floors: 4,
		id: house:[
			'Middle of nowhere',
			0
		],
		street: []
	}
]
```

## Querying graphs

### Different ways to reach similar results

For the questions below, each of the queries will give you largely the same answer. Note that whether `->` and `<-` are parsed as `in` or `out` depends on their direction in relation to the graph edge `wrote`. An arrow pointing towards `wrote` corresponds to `in`, and vice versa.

The following examples show how to make similar queries in a number of different ways, in the context of a database with one person who wrote two articles.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:aristotle }, { id: article:on_sleep_and_sleeplessness }, { id: article:on_dreams }]"

[[test.results]]
value = "[{ id: wrote:j0i75u90qswob99b779m, in: person:aristotle, out: article:on_sleep_and_sleeplessness, time_written: d'-0330-01-01T00:00:00Z' }, { id: wrote:cyqldu8ma76nh1q9jprq, in: person:aristotle, out: article:on_dreams, time_written: d'-0330-01-01T00:00:00Z' }]"
skip-datetime = true 
skip-record-id-key = true

*/

CREATE 
	person:aristotle,
	article:on_sleep_and_sleeplessness,
	article:on_dreams;
RELATE person:aristotle->wrote->[
		article:on_sleep_and_sleeplessness,
		article:on_dreams
	]
	// Written sometime around the year 330 BC
	SET time_written = d"-0330-01-01";
```

Who wrote the articles?

```surql
-- All queries lead to `person:artistotle` twice,
-- via different paths and thus different field names
-- and/or structure

-- Directly from the `wrote` table
SELECT in FROM wrote;

-- From a single `person` record
SELECT ->wrote.in FROM person;
SELECT ->wrote<-person FROM person;

-- From two `article` records
SELECT <-wrote.in FROM article;
SELECT <-wrote<-person FROM article;
```

Which articles did the person write?

```surql
SELECT out FROM wrote;

SELECT ->wrote.out FROM person;
SELECT ->wrote->article FROM person;

SELECT <-wrote.out FROM article;
SELECT <-wrote->article FROM article;
```

When was the article written?

```surql
SELECT time_written FROM wrote;
SELECT ->wrote.time_written as time_written FROM person;
SELECT <-wrote.time_written as time_written FROM article;
```

### Parsing graph queries

For a more complicated query like the one below you can use a simple rule of thumb:
Place the subject in front of the graph selection, then read it backward.

```surql
-- This query
SELECT ->purchased->product<-purchased<-person->purchased->product FROM person:tobie

-- Then becomes
person:tobie->purchased->product<-purchased<-person->purchased->product SELECT
```

Reading this backwards then makes more sense:

> Select every product that was purchased by a person who purchased a product that was also purchased by person Tobie.

Alternatively, you can break it down into steps over multiple lines.

```surql
-- Starting with Tobie
person:tobie
-- move on to his purchased products
->purchased->product
-- that were also purchased by persons...
<-purchased<-person
-- what are all of those persons' purchased products?
->purchased->product
```

Putting it all together it would be: based on all the products Tobie purchased, which person also purchased those products and what did they purchase? This sort of query could be used on a social network site to recommend to the user `person:tobie` a list of people that have similar interests.

### Using parentheses to refine graph query logic

Parentheses can be added at any step of a graph query to refine the logic, such as filtering relations based on specific conditions using the `WHERE` clause.

For example, suppose we want to limit the query to only take recent purchases into account. We can filter `purchased` graph edge to only include purchases made in last 3 weeks:

```surql
-- Select products purchased by people in the last 3 weeks who have purchased the same products that tobie purchased
SELECT 
	->purchased->product
	<-purchased<-person->(purchased WHERE created_at > time::now() - 3w)
	->purchased->product
FROM person:tobie;
```

If the `purchased` graph table can lead to both a `product` or a `subscription`, they can both be added to the query.

```surql
SELECT 
	->purchased->(product, subscription)
	<-purchased<-person
	->purchased->(product, subscription)
FROM person:tobie;
```

The `?` wildcard operator can also be used to search for any and all linked records. The following query will allow purchased `product`, `subscription`, `insurance`, or any other linked records to show up.

```surql
SELECT 
	->purchased->(?)
	<-purchased<-person
	->purchased->(?)
FROM person:tobie;
```

The `?` operator on its own can thus be used to see all of the relations that a record has.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:hermann_hesse }, { id: person:abigail }, { id: city:calw }, { id: book:demian }]"

[[test.results]]
value = "[{ id: wrote:lj0hszy4vbti435r0gza, in: person:hermann_hesse, out: book:demian, written_in: d'1919-01-01T00:00:00Z' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: born_in:0mdrzu8ubnff42lder2c, in: person:hermann_hesse, out: city:calw }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: likes:8pbzxj55dcad7rbrt55y, in: person:abigail, out: person:hermann_hesse }]"
skip-record-id-key = true

[[test.results]]
value = "[{ what_hesse_did: [{ id: born_in:0mdrzu8ubnff42lder2c, in: person:hermann_hesse, out: city:calw }, { id: wrote:lj0hszy4vbti435r0gza, in: person:hermann_hesse, out: book:demian, written_in: d'1919-01-01T00:00:00Z' }], what_others_did_to_hesse: [{ id: likes:8pbzxj55dcad7rbrt55y, in: person:abigail, out: person:hermann_hesse }] }]"
skip-record-id-key = true

*/

CREATE person:hermann_hesse, person:abigail, city:calw, book:demian;
RELATE person:hermann_hesse->wrote->book:demian SET written_in = d'1919-01-01';
RELATE person:hermann_hesse->born_in->city:calw;
RELATE person:abigail->likes->person:hermann_hesse;

SELECT 
	-- all tables in which the record is at `in`
    ->(?).* AS what_hesse_did,
	-- all tables in which the record is at `out`
    <-(?).* AS what_others_did_to_hesse
FROM person:hermann_hesse;
```

```surql title="Output"
[
	{
		what_hesse_did: [
			{
				id: born_in:k3adylof24a2r5kio8l5,
				in: person:hermann_hesse,
				out: city:calw
			},
			{
				id: wrote:ncbo9w0d8t3xd7lvl4dx,
				in: person:hermann_hesse,
				out: book:demian,
				written_in: d'1919-01-01T00:00:00Z'
			}
		],
		what_others_did_to_hesse: [
			{
				id: likes:6gubmldm14gzasoyypay,
				in: person:abigail,
				out: person:hermann_hesse
			}
		]
	}
]
```

The `?` operator can also be used to find all the relations between one record and another. To do this, use the [`<-> operator`](/docs/surrealql/statements/relate#bidirectional-relation-querying) to see all relations in which the record ID in question is either at the `in` or the `out` of the graph edge. Follow this with `(?)` to avoid filtering by graph table name, then use a [`WHERE`](/docs/surrealql/datamodel/arrays#mapping-and-filtering-on-arrays) filter on the output (an array of record IDs) to see if the record ID is present in either the `in` or the `out` field of the graph edge.

A small example of this using some of the relations between Anakin Skywalker (Darth Vader), Palpatine (the Emperor), and Luke Skywalker:

```surql
CREATE person:anakin_skywalker, person:luke_skywalker, person:the_emperor;
RELATE person:anakin_skywalker->served->person:the_emperor;
RELATE person:anakin_skywalker->attacked->person:the_emperor SET won = true;
RELATE person:the_emperor->attacked->person:luke_skywalker SET won = false;
RELATE person:luke_skywalker->son_of->person:anakin_skywalker;
RELATE person:the_emperor->fooled->person:anakin_skywalker SET date = "19 BBY";

-- As a SELECT statement
SELECT VALUE <->(?)[WHERE person:the_emperor IN [in, out]] FROM ONLY person:anakin_skywalker;
SELECT VALUE <->(?)[WHERE person:luke_skywalker IN [in, out]] FROM ONLY person:anakin_skywalker;

-- Or returned directly from the record ID
person:anakin_skywalker<->(?)[WHERE person:the_emperor IN [in, out]];
person:anakin_skywalker<->(?)[WHERE person:luke_skywalker IN [in, out]];
```

```surql title="Output"
-------- Anakin and Emperor relations --------

[
	{
		date: '19 BBY',
		id: fooled:irm2w6jvd1dmppjr7kh2,
		in: person:the_emperor,
		out: person:anakin_skywalker
	},
	{
		id: attacked:r8b4z5yr627wy9i73jkh,
		in: person:anakin_skywalker,
		out: person:the_emperor,
		won: true
	},
	{
		id: served:30oyjvv5uutnj255w4oy,
		in: person:anakin_skywalker,
		out: person:the_emperor
	}
]

-------- Anakin and Luke relations --------

[
	{
		id: son_of:h8oosl7s27n21kh3c2iq,
		in: person:luke_skywalker,
		out: person:anakin_skywalker
	}
]
```

Parentheses can be used at each point of a graph query. The example below includes `person` records (authors) connected to `book` records by the `wrote` table. As both the `person` and `book` tables have fields that can be useful when filtering, they can be isolated with parentheses at this point of the graph query in order to filter using the `WHERE` clause.

```surql
/**[test]

[[test.results]]
value = "[{ born: d'1891-01-03T00:00:00Z', id: person:j_r_r_tolkien, name: 'J.R.R. Tolkien' }]"

[[test.results]]
value = "[{ born: '-0428-06-01', id: person:plato, name: 'Plato' }]"

[[test.results]]
value = "[{ id: book:fotr, name: 'The Fellowship of the Ring' }]"

[[test.results]]
value = "[{ id: book:republic, name: 'The Republic', original_name: 'Πολιτεία' }]"

[[test.results]]
value = "[{ id: wrote:j6lto05qluv9nivz4ens, in: person:j_r_r_tolkien, out: book:fotr, written_at: 'North Oxford' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: wrote:y9ydvugxwhrpns6qn4s9, in: person:plato, out: book:republic, written_at: 'Athens' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ books_written_in_athens: [], name: 'J.R.R. Tolkien' }, { books_written_in_athens: [{ id: book:republic, name: 'The Republic', original_name: 'Πολιτεία' }], name: 'Plato' }]"

[[test.results]]
value = "[{ books_about_rings: [{ id: book:fotr, name: 'The Fellowship of the Ring' }], name: 'J.R.R. Tolkien' }, { books_about_rings: [], name: 'Plato' }]"

*/

CREATE person:j_r_r_tolkien SET
	name = "J.R.R. Tolkien",
	born = d'1891-01-03';
-- Very approximate date of birth
CREATE person:plato SET 
	name = "Plato", 
	born = "-0428-06-01";

CREATE book:fotr SET 
	name = "The Fellowship of the Ring";
CREATE book:republic SET 
	name = "The Republic",
	original_name = "Πολιτεία";

RELATE person:j_r_r_tolkien->wrote->book:fotr SET written_at = "North Oxford";
RELATE person:plato->wrote->book:republic SET written_at = "Athens";

SELECT 
	name,
	-- Isolate 'wrote' to use WHERE
	->(wrote WHERE written_at = "Athens")->book.* AS books_written_in_athens
FROM person;

SELECT 
	name, 
	-- Isolate 'book' to use WHERE
	->wrote->(book WHERE "Ring" IN name).* AS books_about_rings
FROM person;
```

```surql title="Output"
-------- Query --------

[
	{
		books_written_in_athens: [],
		name: 'J.R.R. Tolkien'
	},
	{
		books_written_in_athens: [
			{
				id: book:republic,
				name: 'The Republic',
				original_name: 'Πολιτεία'
			}
		],
		name: 'Plato'
	}
]

-------- Query --------

[
	{
		books_about_rings: [
			{
				id: book:fotr,
				name: 'The Fellowship of the Ring'
			}
		],
		name: 'J.R.R. Tolkien'
	},
	{
		books_about_rings: [],
		name: 'Plato'
	}
]
```

[Destructuring](/docs/surrealql/datamodel/idioms#destructuring) can also be used to pick and choose which fields to access inside a graph query. The following query will return the same output as above, except that `original_name: 'Πολιτεία'` will no longer show up.

```surql
SELECT 
	name, 
	->(wrote WHERE written_at = "Athens")->book.{ name, id } AS books_written_in_athens
FROM person;
```

### Bidirectional relation querying

All of the queries up to now have been clear about what sort of record is found at the `in` and `out` fields: `in` is the record that is doing something, while `out` is the record that has something done to it:

* A `person` who writes an `article`: the person **writes**, the article **is written**.
* A `person` who purchases a `product`: the person **purchases**, the product **is purchased**.

However, sometimes a relation is such that it is impossible to determine which record is located at the `in` part of a graph table, and which is located at the `out` part. This is the case when a relationship is truly bidirectional and equal, such as a friendship, marriage, or sister cities:

```surql
/**[test]

[[test.results]]
value = "[{ id: city:calgary }, { id: city:daejeon }]"

[[test.results]]
value = "[{ id: sister_of:crb0gb01iybgjhmed1ia, in: city:calgary, out: city:daejeon }]"
skip-record-id-key = true

*/

CREATE city:calgary, city:daejeon;
RELATE city:calgary->sister_of->city:daejeon;
```

This relation could just as well have been established with the statement `RELATE city:daejeon->sister_of->city:calgary`.

In such a case, a query on the relationship makes it appear as if one city has a twin city but the other does not.

```surql
SELECT id, ->sister_of->city AS sister_cities FROM city;
```

```surql title="Response"
[
	{
		id: city:calgary,
		sister_cities: [
			city:daejeon
		]
	},
	{
		id: city:daejeon,
		sister_cities: []
	}
]
```

To solve this, we can use the `<->` operator instead of `->`. Using `<->` will access both the `in` and `out` fields, instead of just one.

```surql
SELECT id, <->sister_of<->city AS sister_cities FROM city;
```

This brings up another issue in which a city now appears to be a sister city of itself.

```surql
[
	{
		id: city:calgary,
		sister_cities: [
			city:calgary,
			city:daejeon
		]
	},
	{
		id: city:daejeon,
		sister_cities: [
			city:calgary,
			city:daejeon
		]
	}
]
```

Here we can use the [`array::complement`](/docs/surrealql/functions/database/array#arraycomplement) function to return only items from one array that are not present in another array.

```surql
SELECT id, array::complement(<->sister_of<->city, [id]) AS sister_cities FROM city;
```

```surql title="Response"
[
	{
		id: city:calgary,
		sister_cities: [
			city:daejeon
		]
	},
	{
		id: city:daejeon,
		sister_cities: [
			city:calgary
		]
	}
]
```

Adding a unique key is a good practice for this sort of relation, as it will prevent it from being created twice. This can be done by [defining a field](/docs/surrealql/statements/define/field) as a unique key based on the ordered record IDs involved, followed by a [`DEFINE INDEX`](/docs/surrealql/statements/define/field) statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

DEFINE FIELD key ON TABLE sister_of VALUE <string>array::sort([in, out]);
DEFINE INDEX only_one_sister_city ON TABLE sister_of FIELDS key UNIQUE;
```

With the index in place, a relation set from one record to the other now cannot be created a second time.

```surql
RELATE city:calgary->sister_of->city:daejeon; -- OK
RELATE city:daejeon->sister_of->city:calgary;
-- "Database index `only_one_sister_city` already contains '[city:calgary, city:daejeon]', with record `sister_of:npab0uoxogmrvpwsvfoa`"
```

### Refining the `in` and `out` fields of a relation

As mentioned above, the `in` and `out` fields of a graph table are mandatory but can be modified to specify their record type or make assertions.

Thus, the following field declarations will work:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

DEFINE FIELD in ON TABLE wrote TYPE record<author>;
DEFINE FIELD out ON TABLE wrote TYPE record<book>;
```

But any attempt to outright redefine the `in` or `out` fields as a different type will be ignored.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

DEFINE FIELD in ON TABLE wrote TYPE string;
DEFINE FIELD out ON TABLE wrote TYPE int;
```

An example of an assertion on one of the fields of a record table for a library which is not yet ready to handle non-English books:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: book:demian, language: 'German', title: 'Demian. Die Geschichte von Emil Sinclairs Jugend' }]"

[[test.results]]
value = "[{ id: author:hesse, name: 'Hermann Hesse' }]"

[[test.results]]
error = ""Found book:demian for field `out`, with record `wrote:l4xjcgqkgm7vmqqt4iah`, but field must conform to: $value.language = 'English'""

*/

DEFINE FIELD out ON TABLE wrote TYPE record<book> ASSERT $value.language = "English";

CREATE book:demian SET title = "Demian. Die Geschichte von Emil Sinclairs Jugend", language = "German";
CREATE author:hesse SET name = "Hermann Hesse";

RELATE author:hesse->wrote->book:demian;
```

```surql title="Output"
"Found book:demian for field `out`, with record `wrote:l4xjcgqkgm7vmqqt4iah`, but field must conform to: $value.language = 'English'"
```

### Structure of queries on relations

Using an alias is a common practice in both regular and relation queries in SurrealDB to make output more readable and collapse nested structures. You can create an alias using the `AS` clause.

```surql
/**[test]

[[test.results]]
value = "[{ id: cat:one }, { id: cat:two }, { id: cat:three }]"

[[test.results]]
value = "[{ id: friends_with:8tuumzamsov3pro9tz3j, in: cat:one, out: cat:two }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: friends_with:ro94pfxfpff35x94kgmy, in: cat:two, out: cat:three }]"
skip-record-id-key = true

[[test.results]]
value = "[{ "->friends_with": { "->cat": { "->friends_with": { "->cat": [cat:three] } } } }]"

[[test.results]]
value = "[{ friends_of_friends: [cat:three] }]"

*/

CREATE cat:one, cat:two, cat:three;

RELATE cat:one->friends_with->cat:two;
RELATE cat:two->friends_with->cat:three;

SELECT ->friends_with->cat->friends_with->cat FROM cat:one;
-- create an alias for the result using the `AS` clause.
SELECT ->friends_with->cat->friends_with->cat AS friends_of_friends FROM cat:one;
```

```surql
// Output without alias
{
	"->friends_with": {
		"->cat": {
			"->friends_with": {
				"->cat": [
					cat:three
				]
			}
		}
	}
}

// Output with alias
{
	friends_of_friends: [
		cat:three
	]
}
```

However, an alias might not be preferred in a case where you have multiple graph queries that resolve to the fields of a large nested structure. Take the following data for example:

```surql
CREATE country:usa SET name = "USA";
CREATE state:pennsylvania SET population = 12970000;
CREATE state:michigan SET population = 10030000;
CREATE city:philadelphia, city:pittsburgh, city:detroit, city:grand_rapids;

RELATE country:usa->contains->[state:pennsylvania, state:michigan];
RELATE state:pennsylvania->contains->[city:philadelphia, city:pittsburgh];
RELATE state:michigan->contains->[city:detroit, city:grand_rapids];
```

A query on the states and cities of these records using aliases would return the data in a structure remade to fit the aliases declared in the query.

```surql
SELECT
    name,
    ->contains->state AS states,
    ->contains->state->contains->city AS cities
FROM country:usa;
```

```surql title="Output"
[
	{
		cities: [
			city:philadelphia,
			city:pittsburgh,
			city:grand_rapids,
			city:detroit
		],
		name: 'USA',
		states: [
			state:pennsylvania,
			state:michigan
		]
	}
]
```

However, opting to not use an alias will return the original graph structure which makes the levels of depth of the query clearer. In addition, the `population` field is clearly the population for the states.

```surql
SELECT
    id,
    ->contains->state.id,
    ->contains->state.population,
    ->contains->state->contains->city.id
FROM country:usa;
```

The [destructuring syntax](/docs/surrealql/datamodel/idioms#destructuring) can be used to reduce some typing. Here is the same query as the last using destructuring syntax instead of one line for each field.

```surql
SELECT
    id,
	-- access id and population on a single line
    ->contains->state.{id, population},
    ->contains->state->contains->city.id
FROM country:usa;
```

```surql title="Output"
[
	{
		"->contains": {
			"->state": {
				"->contains": {
					"->city": {
						id: [
							city:philadelphia,
							city:pittsburgh,
							city:grand_rapids,
							city:detroit
						]
					}
				},
				id: [
					state:pennsylvania,
					state:michigan
				],
				population: [
					12970000,
					10030000
				]
			}
		},
		id: country:usa
	}
]
```

As the query that uses aliases does not maintain the original graph structure, adding `population` would require clever renaming such as `->contains->state.population AS state_populations` to make it clear that the numbers represent state and not city populations.

### Multiple graph tables vs. fields

Being able to set fields on graph tables opens up a large variety of custom query methods, one of which is explored here.

Imagine a database that holds detailed information on the relations between NPCs in a game that are made to be as realistic as possible. Two of the characters have a rocky past but finally end up married. During this period, we might have tracked their relationship by adding and removing graph edges between the two of them as they move from a stage of being friends, to dating, to hating each other, to finally ending up married.

```surql
CREATE person:one, person:two;
-- These three relations would end up deleted
RELATE person:one->friends_with->person:two;
RELATE person:one->dating->person:two;
RELATE person:one->hates->person:two;
-- Finally this would be the graph edge connecting the two
RELATE person:one->married->person:two;
```

This works well to track the current state of the relationship, but creating a more general table such as `knows` along with a number of fields can be a better method to track the changing relationship over time. The following shows the relationship between the two `person` records, along with a third record called `person:three` who went to the same school and once dated `person:one`.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:one }, { id: person:two }, { id: person:three }]"

[[test.results]]
value = "[{ has_been_friends: true, has_dated: true, has_hated: true, id: knows:i7yz3l6dcje9it8qbuwo, in: person:one, married_to: true, out: person:two }]"
skip-record-id-key = true

[[test.results]]
value = "[{ has_dated: true, id: knows:0xfnkabcg0um428rtj2o, in: person:one, out: person:three, same_high_school: true }]"
skip-record-id-key = true

*/

CREATE person:one, person:two, person:three;
RELATE person:one->knows->person:two SET
    has_been_friends = true,
    has_dated = true,
    has_hated = true,
    married_to = true;

RELATE person:one->knows->person:three SET
    same_high_school = true,
    has_dated = true;
```

With these fields in place, it is possible to use a `WHERE` clause to do refined searches on relationships of a certain type.

```surql
SELECT 
	->knows->person AS knows,
	->knows[WHERE has_dated]->person AS has_dated,
	->knows[WHERE same_high_school AND has_dated]->person AS dated_and_same_school
 FROM person:one;
```

```surql title="Response"
[
	{
		dated_and_same_school: [
			person:three
		],
		has_dated: [
			person:two,
			person:three
		],
		knows: [
			person:two,
			person:three
		]
	}
]
```

Because the `WHERE` clause simply checks for [truthiness](/docs/surrealql/datamodel/values#values-and-truthiness) (whether a value is present and not empty), these fields do not necessarily need to be booleans and can even be complex objects.

```surql
/**[test]

[[test.results]]
value = "[{ has_been_friends: true, has_dated: { from: d'2020-12-25T00:00:00Z', to: d'2023-12-25T00:00:00Z' }, has_hated: { from: d'2023-12-25T00:00:00Z', to: d'2024-03-01T00:00:00Z' }, id: knows:dzj9fmhh22kodz8h2nhl, in: person:one, married_to: { since: d'2024-03-01T00:00:00Z' }, out: person:two, same_high_school: false }]"
skip-record-id-key = true

[[test.results]]
value = "[{ has_dated: { from: d'2019-09-10T00:00:00Z', to: d'2020-12-31T00:00:00Z' }, id: knows:gc5993blynpzf5l52qs4, in: person:one, out: person:three, same_high_school: true }]"
skip-record-id-key = true

*/

RELATE person:one->knows->person:two SET
	same_high_school = false,
    has_been_friends = true,
    has_dated = {
		from: d'2020-12-25',
		to: d'2023-12-25'
	},
    has_hated = {
		from: d'2023-12-25',
		to: d'2024-03-01'
	},
    married_to = {
		since: d'2024-03-01'
	};

RELATE person:one->knows->person:three SET
    same_high_school = true,
    has_dated = {
		from: d'2019-09-10',
		to: d'2020-12-31'
	};
```

With these objects, a jealous `person:two` could do a check on `person:one` to see how many relationships with `has_dated` have an end time that overlaps with the `has_dated` period of `person:one` and `person:two`.

```surql
SELECT id, ->knows[WHERE same_high_school AND has_dated.to > d'2020-12-25']->person FROM person:one;
```

### Recursive graph queries

<Since v="v2.1.0" />

Graph edges can also be queried recursively. For a full explanation of this syntax, see the page on [recursive paths](/docs/surrealql/datamodel/idioms#recursive-paths).

Take the following example which creates five cities, each of which is connected to the next by some type of road of random length.

```surql
-- Note: 1..6 used to be inclusive until SurrealDB 3.0.0
-- Now creates 1 up to but not including 6
CREATE |city:1..=6| SET name = <string>id.id() + 'ville';
FOR $pair IN (<array>(1..=5)).windows(2) {
  	LET $city1 = type::record("city", $pair[0]);
    LET $city2 = type::record("city", $pair[1]);
    RELATE $city1->to->$city2 SET 
        type = rand::enum(["train", "road", "bike path"]),
        distance = <int>(rand::float() * 100).ceil()
};
```

While it is possible to manually move three levels down this road network, it involves a good deal of manual typing.

```surql
SELECT ->to->city->to->city->to->city AS fourth_city FROM city:1;
```

```surql title="Response"
[
	{
		fourth_city: [
			city:4
		]
	}
]
```

This can be replaced by a `@` to refer to the current record, followed by `.{3}` to represent three levels down the `to` graph edge. A level between 1 and 256 can be specified here.

```surql
SELECT @.{3}->to->city AS fourth_city FROM city:1;
```

A traditional query to show the final road info from `city:1` to the city three stops away would look like this.

```surql
SELECT ->to->city->to->city->to.* AS third_journey FROM city:1;
```

```surql title="Response"
[
	{
		fourth_city: [
			[
				{
					distance: 80,
					id: to:sw2pery99jomfhibzfrh,
					in: city:3,
					out: city:4,
					type: 'train'
				}
			]
		]
	}
]
```

To use the same query recursively, wrap the part that must be repeated (`->to->city`) inside parentheses. This will ensure that the `.{2}` part of the query only repeats `->to->city` twice, and not the final `->to.*` portion.

```surql
SELECT @.{2}(->to->city)->to.* AS third_journey FROM city:1;
```

A range can be added inside the `{}` braces. The following query that uses a range of 1 to 20 will follow the `->to->city` path up to 20 times, but will stop at the 5th and final depth because the next level returns an empty array.

```surql
city:1.{1..20}->to->city;
```

```surql title="Response"
[
	city:5
]
```

Ranges can be followed with the destructuring operator to collect fields on each depth, returning them in a single response. The following query goes five depths down the `to` graph table, returning each city and road along the way.

```surql
SELECT @.{1..5}.{ 
    id, 
    next_roads: ->to.*,
    next_cities: ->to->city
} FROM city;
```

```surql title="Response"
[
	{
		id: city:1,
		next_cities: [
			city:2
		],
		next_roads: [
			{
				distance: 33,
				id: to:bl6i9djau0pg24pqrwd9,
				in: city:1,
				out: city:2,
				type: 'road'
			}
		]
	},
	{
		id: city:2,
		next_cities: [
			city:3
		],
		next_roads: [
			{
				distance: 45,
				id: to:ybugfnlzv6kcrkaj49ig,
				in: city:2,
				out: city:3,
				type: 'road'
			}
		]
	},
	{
		id: city:3,
		next_cities: [
			city:4
		],
		next_roads: [
			{
				distance: 80,
				id: to:sw2pery99jomfhibzfrh,
				in: city:3,
				out: city:4,
				type: 'train'
			}
		]
	},
	{
		id: city:4,
		next_cities: [
			city:5
		],
		next_roads: [
			{
				distance: 29,
				id: to:42hlspf4z5lpqceyv68p,
				in: city:4,
				out: city:5,
				type: 'train'
			}
		]
	},
	{
		id: city:5,
		next_cities: [],
		next_roads: []
	}
]
```

As noted above, a `TIMEOUT` can be set for queries that may be computationally expensive. This is particularly useful when experimenting with recursive queries, which, if care is not taken, can run all the way to the maximum possible depth of 256.

Take the following example with two `person` records that like each other. Following the `likes` edge will run until the query recurses 256 times and gives up.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:one }, { id: person:two }]"

[[test.results]]
value = "[{ id: likes:0ng6fka1m8iwp7y5lgul, in: person:one, out: person:two }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: likes:50kdcb8awc7jwsxzr7ra, in: person:two, out: person:one }]"
skip-record-id-key = true

[[test.results]]
error = "'Exceeded the idiom recursion limit of 256.'"

*/

CREATE person:one, person:two;
RELATE person:one->likes->person:two;
RELATE person:two->likes->person:one;
-- Open-ended range
person:one.{..}->likes->person;
```

```surql title="Response"
'Exceeded the idiom recursion limit of 256.'
```

Take the following example in which three `person` records of created, each of which likes the other two `person` records. A query on the `->likes->person` path shows that the number of records doubles each time.

```surql
CREATE |person:1..4|;
FOR $person IN (SELECT * FROM person) {
  LET $others = (SELECT * FROM person WHERE id != $person.id);
    FOR $other IN $others {
        RELATE $person->likes->$other;
    }
};
RETURN [
	person:1.{2}->likes->person,
	person:1.{3}->likes->person,
	person:1.{4}->likes->person
];
```

```surql title="Response"
[
	[
		person:1,
		person:2,
		person:1,
		person:3
	],
	[
		person:3,
		person:2,
		person:1,
		person:3,
		person:3,
		person:2,
		person:1,
		person:2
	],
	[
		person:1,
		person:2,
		person:1,
		person:3,
		person:3,
		person:2,
		person:1,
		person:2,
		person:1,
		person:2,
		person:1,
		person:3,
		person:3,
		person:2,
		person:1,
		person:3
	]
]
```

Since an open-ended range can be specified in a recursive query, this would result in a full 256 attempts to recurse, multiplying the number of results by two each time for a total of 115792089237316195423570985008687907853269984665640564039457584007913129639936 records by the end.

When experimenting with recursive queries, especially open-ended ranges, it is thus recommended to use a timeout.

```surql
SELECT @.{..}.{ id, likes: ->likes->person.@ } FROM person TIMEOUT 1s;
```

### Graph clauses

<Since v="v2.2.0" />

The same clauses available to a `SELECT` statement can be used inside a graph query. Take the following relations for example:

```surql
CREATE person:one, person:two, person:three;
RELATE person:one->knows->person:two SET
	friends = true,
    dated = true,
    married_to = true;

RELATE person:one->knows->person:three SET
    dated = true;

RELATE person:two->knows->person:three SET
	friends = true;
```

At the `knows` path, parentheses can be used to insert clauses or an entirely new SELECT statement based on the records turned up at this point. In the following example, the `FROM knows` portion applies to all the records that a `person` knows, not the `knows` table as a whole.

```surql
SELECT 
	id, 
	->(SELECT out.id AS counterpart, !!dated AS dated FROM knows) AS acquaintances
FROM person;
```

```surql title="Output"
[
	{
		acquaintances: [
			{
				counterpart: person:three,
				dated: true
			},
			{
				counterpart: person:two,
				dated: true
			}
		],
		id: person:one
	},
	{
		acquaintances: [],
		id: person:three
	},
	{
		acquaintances: [
			{
				counterpart: person:three,
				dated: false
			}
		],
		id: person:two
	}
]
```

In some cases, the dot or destructuring operator can produce the same output. The following queries are equivalent.

```surql
SELECT ->(SELECT * FROM knows) FROM person:one;
SELECT ->knows.* FROM person:one;
```

```surql title="Output"
[
	{
		"->knows": [
			{
				dated: true,
				id: knows:2tsz3aomelegp060ii7d,
				in: person:one,
				out: person:three
			},
			{
				dated: true,
				friends: true,
				id: knows:g54z9zapdkssxb4p4pjc,
				in: person:one,
				married_to: true,
				out: person:two
			}
		]
	}
]
```

However, clauses available in [`SELECT` statements](/docs/surrealql/statements/select) such as `WHERE`, `LIMIT`, `GROUP BY`, aliases and so on can be used, making a graph clause a most flexible option.

```surql
SELECT ->(SELECT *, time::now() AS queried_at FROM knows LIMIT 1) FROM person:one;
```

```surql title="Output"
[
	{
		"->knows": [
			{
				dated: true,
				id: knows:2tsz3aomelegp060ii7d,
				in: person:one,
				out: person:three,
				queried_at: d'2025-01-24T02:16:31.811Z'
			}
		]
	}
]
```

Some other examples of possible graph clauses:

```surql
CREATE |person:1..4|;

RELATE person:1->likes->person:2 SET like_strength = 20, know_in_person = true;
RELATE person:1->likes->person:3 SET like_strength = 5,  know_in_person = false;
RELATE person:2->likes->person:1 SET like_strength = 10, know_in_person = true;
RELATE person:2->likes->person:3 SET like_strength = 12, know_in_person = false;
RELATE person:3->likes->person:1 SET like_strength = 2,  know_in_person = false;
RELATE person:3->likes->person:2 SET like_strength = 9,  know_in_person = false;

SELECT ->likes AS likes FROM person;
SELECT ->(SELECT like_strength FROM likes) AS likes FROM person;
SELECT ->(SELECT like_strength FROM likes WHERE like_strength > 10) AS likes FROM person;
SELECT ->(likes WHERE like_strength > 10) AS likes FROM person;
SELECT ->(SELECT like_strength, know_in_person FROM likes ORDER BY like_strength DESC) AS likes FROM person;
SELECT ->(SELECT count() as count, know_in_person FROM likes GROUP BY know_in_person) AS likes FROM person;
SELECT ->(likes LIMIT 1) AS likes FROM person;
SELECT ->(likes START 1) AS likes FROM person;
```

Multiple graph tables can be selected by separating each table with a comma, in the same way as in any other `SELECT` statement. In addition, all tables can be selected by using `?` as a wildcard operator.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:one, name: 'Þor' }]"

[[test.results]]
value = "[{ id: dog:one, name: 'Fenrir' }]"

[[test.results]]
value = "[{ id: cat:one, name: 'Jólakötturinn' }]"

[[test.results]]
value = "[{ at: d'2025-10-08T05:00:18.418734Z', id: feeds:150g6q4l8lzscu7j57sj, in: person:one, out: cat:one }]"
skip-datetime = true 
skip-record-id-key = true

[[test.results]]
value = "[{ at: d'2025-10-08T05:00:18.420350Z', id: plays_with:itzsltgkyboieenc7rmz, in: dog:one, out: cat:one }]"
skip-datetime = true 
skip-record-id-key = true

[[test.results]]
value = "[{ "<-(SELECT * FROM feeds, plays_with ORDER BY at
)": [{ at: d'2025-10-08T05:00:18.418734Z', id: feeds:150g6q4l8lzscu7j57sj, in: person:one, out: cat:one }, { at: d'2025-10-08T05:00:18.420350Z', id: plays_with:itzsltgkyboieenc7rmz, in: dog:one, out: cat:one }] }]"
skip-datetime = true 
skip-record-id-key = true

[[test.results]]
value = "[{ "<-(SELECT * FROM ? ORDER BY at
)": [{ at: d'2025-10-08T05:00:18.418734Z', id: feeds:150g6q4l8lzscu7j57sj, in: person:one, out: cat:one }, { at: d'2025-10-08T05:00:18.420350Z', id: plays_with:itzsltgkyboieenc7rmz, in: dog:one, out: cat:one }] }]"
skip-datetime = true 
skip-record-id-key = true

*/

CREATE person:one SET name = "Þor";
CREATE dog:one SET name = "Fenrir";
CREATE cat:one SET name = "Jólakötturinn";
RELATE person:one->feeds->cat:one SET at = time::now();
RELATE dog:one->plays_with->cat:one SET at = time::now();

-- Select from both 'feeds' and 'plays_with'
SELECT <-(SELECT * FROM feeds, plays_with ORDER BY at) FROM cat:one;
-- Or any graph table
SELECT <-(SELECT * FROM ? ORDER BY at) FROM cat:one;
```

### Ranges inside graph queries

<Since v="v2.3.0" />

Range syntax can also be used on the edges of a graph query.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:one }, { id: person:two }, { id: person:three }, { id: person:four }]"

[[test.results]]
value = "[{ id: likes:1, in: person:one, out: person:two }]"

[[test.results]]
value = "[{ id: likes:2, in: person:one, out: person:three }]"

[[test.results]]
value = "[{ id: likes:3, in: person:one, out: person:four }]"

[[test.results]]
value = "[person:three, person:four]"

*/

CREATE person:one, person:two, person:three, person:four;

RELATE person:one->likes:1->person:two;
RELATE person:one->likes:2->person:three;
RELATE person:one->likes:3->person:four;

person:one->likes:2..=4->person;
```

```surql title="Output"
[
	person:three,
	person:four
]
```

A common usage of range syntax on edges is when their ID has been defined as a ULID, making the `id` field random yet sortable and significant in terms of time.

```surql
RELATE character:one->speaks_to:ulid()->character:two SET content = "Greetings, adventurer!";
RELATE character:one->speaks_to:ulid()->character:two SET content = "Can you please help me? My sheep have run amok.";

SELECT
	// Grab the latter part of the record ID, turn it into a datetime
    time::from_ulid(id.id()) AS at,
    content
FROM
    // ULID from 2025-04-25, well before today's date
    character:one->speaks_to:01JSNG0KZSY3HJ5QSZ7JSMQMGR..;
```

```surql title="Output"
[
	{
		at: d'2025-04-25T03:37:53.246Z',
		content: 'Greetings, adventurer!'
	},
	{
		at: d'2025-04-25T03:37:53.248Z',
		content: 'Can you please help me? My sheep have run amok.'
	}
]
```

Array-based record IDs also work well inside range queries on edges.

```surql
CREATE planet:venus, telescope:one;

RELATE telescope:one->observed:[d'2025-04-24T02:02:18.204Z']->planet:venus CONTENT { 
      temperature_profile: {
        surface: 735.0,
        upper_atmosphere: 300.0
      },
      composition: {
        CO2: 96.5,
        N2: 3.5,
        SO2: 0.015
      },
};

RELATE telescope:one->observed:[d'2025-04-25T02:02:18.204Z']->planet:venus CONTENT {
      temperature_profile: {
        surface: 737.0,
        upper_atmosphere: 298.5
      },
      composition: {
        CO2: 96.6,
        N2: 3.4,
        SO2: 0.015
    }
};

SELECT id, (<-observed:[d'2025-04-24']..).{
    at: id[0], 
    surface: temperature_profile.surface,
    atmosphere: temperature_profile.upper_atmosphere
} AS observations FROM planet;
```

```surql title="Output"
[
	{
		id: planet:venus,
		observations: [
			{
				at: d'2025-04-24T02:02:18.204Z',
				atmosphere: 300,
				surface: 735
			},
			{
				at: d'2025-04-25T02:02:18.204Z',
				atmosphere: 298.5f,
				surface: 737
			}
		]
	}
]
```

## remove.mdx

---
sidebar_position: 19
sidebar_label: REMOVE
title: REMOVE statement | SurrealQL
description: The REMOVE statement is used to remove resources such as databases, tables, indexes, events and more.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `REMOVE` statement

The `REMOVE` statement is used to remove resources such as databases, tables, indexes, events and more.
Similar to an SQL DROP statement.

### Statement syntax

<Tabs syncKey="remove-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
REMOVE [
    ACCESS    [ IF EXISTS ] @name ON [ NAMESPACE | DATABASE ]
  | ANALYZER  [ IF EXISTS ] @name
  | API       [ IF EXISTS ] @name
  | DATABASE  [ IF EXISTS ] @name
  | EVENT     [ IF EXISTS ] @name ON [ TABLE ] @table
  | FIELD     [ IF EXISTS ] @name ON [ TABLE ] @table
  | FUNCTION  [ IF EXISTS ] @name
  | INDEX     [ IF EXISTS ] @name ON [ TABLE ] @table
  | NAMESPACE [ IF EXISTS ] @name
  | PARAM     [ IF EXISTS ] @name
  | TABLE     [ IF EXISTS ] @name
  | USER      [ IF EXISTS ] @name ON [ ROOT | NAMESPACE | DATABASE ]
]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const removeAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "REMOVE" },
      { type: "Choice", index: 1, children: [
        
        { type: "Sequence", children: [ { type: "Terminal", text: "ACCESS" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" }, { type: "Terminal", text: "ON" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NAMESPACE" }, { type: "Terminal", text: "DATABASE" } ] } ] },

        { type: "Sequence", children: [ { type: "Terminal", text: "ANALYZER" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" } ] },
        
        { type: "Sequence", children: [ { type: "Terminal", text: "API" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" } ] },

        { type: "Sequence", children: [ { type: "Terminal", text: "DATABASE" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" } ] },

        { type: "Sequence", children: [ { type: "Terminal", text: "EVENT" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" }, { type: "Terminal", text: "ON" }, { type: "Optional", child: { type: "Terminal", text: "TABLE" } }, { type: "NonTerminal", text: "@table" } ] },

        { type: "Sequence", children: [ { type: "Terminal", text: "FIELD" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" }, { type: "Terminal", text: "ON" }, { type: "Optional", child: { type: "Terminal", text: "TABLE" } }, { type: "NonTerminal", text: "@table" } ] },

        { type: "Sequence", children: [ { type: "Terminal", text: "FUNCTION" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" } ] },

        { type: "Sequence", children: [ { type: "Terminal", text: "INDEX" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" }, { type: "Terminal", text: "ON" }, { type: "Optional", child: { type: "Terminal", text: "TABLE" } }, { type: "NonTerminal", text: "@table" } ] },
        
        { type: "Sequence", children: [ { type: "Terminal", text: "NAMESPACE" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" } ] },

        { type: "Sequence", children: [ { type: "Terminal", text: "PARAM" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "Terminal", text: "$" }, { type: "NonTerminal", text: "@name" } ] },

        { type: "Sequence", children: [ { type: "Terminal", text: "TABLE" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" } ] },
        
        { type: "Sequence", children: [ { type: "Terminal", text: "USER" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } }, { type: "NonTerminal", text: "@name" }, { type: "Terminal", text: "ON" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "ROOT" }, { type: "Terminal", text: "NAMESPACE" }, { type: "Terminal", text: "DATABASE" } ] } ] }
      ] }
    ]}
  ]
};

<RailroadDiagram ast={removeAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage
### Basic usage

The following queries show an example of how to remove resources.

```surql
REMOVE NAMESPACE surrealdb;

REMOVE DATABASE blog;

REMOVE USER writer ON NAMESPACE;

REMOVE USER writer ON DATABASE;

REMOVE ACCESS token ON NAMESPACE;

REMOVE ACCESS user ON DATABASE;

REMOVE EVENT new_post ON TABLE article;

-- Only works for Schemafull tables (i.e. tables with a schema)
REMOVE FIELD tags ON TABLE article;

REMOVE INDEX authors ON TABLE article;

REMOVE ANALYZER example_ascii;

REMOVE FUNCTION fn::update_author;

REMOVE PARAM $author;

REMOVE TABLE article;
```

### Using if exists clause



The following queries show an example of how to remove resources using the `IF EXISTS` clause, which will only remove the resource if it exists.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

REMOVE NAMESPACE IF EXISTS surrealdb;

REMOVE DATABASE IF EXISTS blog;

REMOVE USER IF EXISTS writer ON NAMESPACE;

REMOVE USER IF EXISTS writer ON DATABASE;

REMOVE ACCESS IF EXISTS token ON NAMESPACE;

REMOVE ACCESS IF EXISTS user ON DATABASE;

REMOVE EVENT IF EXISTS new_post ON TABLE article;

REMOVE FIELD IF EXISTS tags ON TABLE article;

REMOVE INDEX IF EXISTS authors ON TABLE article;

REMOVE ANALYZER IF EXISTS example_ascii;

REMOVE FUNCTION IF EXISTS fn::update_author;

REMOVE PARAM IF EXISTS $author;

REMOVE TABLE IF EXISTS article;
```

### Usage in table views

<Since v="v3.0.0" />

A table used as a source for a table view cannot be removed until the table view itself has been removed.

```surql
DEFINE TABLE pc;
DEFINE TABLE pc_agg AS SELECT count(), class FROM pc GROUP BY class;
CREATE |pc:3| SET class = "Wizard";
CREATE |pc:10| SET class = "Warrior";
SELECT * FROM pc_agg;
-- Error: pc_agg requires pc to work
REMOVE TABLE pc;
REMOVE TABLE pc_agg;
-- pc_agg is now gone, pc can be removed too
REMOVE TABLE pc;
```

The `SELECT * FROM pc_agg` query shows that the table view is pulling data from the `pc` table. As long as the `pc` table exists, `pc_agg` cannot be removed.

```surql
-------- Query --------

[
  { 
    class: 'Warrior', 
    count: 10, 
    id: pc_agg:['Warrior'] 
  }, 
  { 
    class: 'Wizard', 
    count: 3, 
    id: pc_agg:['Wizard'] 
  }
]

-------- Query --------

'Invalid query: Cannot delete table `pc` on which a view is defined, table(s) `pc_agg` are defined as a view on this table.'
```

### Behaviour after removal

While all `REMOVE` statements remove the definition for a resource, some resources have additional actions when removed. They are:

* REMOVE DATABASE: This effectively deletes the database by removing the index stores, deleting the definition, and clearing the cache.
* REMOVE NAMESPACE: Same as `REMOVE DATABASE`, in addition to performing a remove on each database inside the namespace.
* REMOVE TABLE: Similar to the two previous statements but on a single table, and will fail if a table view depends on it. Removing a table will also send a [KILL](/docs/surrealql/statements/kill) notification for each live query defined on it.
* REMOVE INDEX: This statement also removes the index store cache and index data. If you are considering removing an index but want to test the behaviour out first, use an [ALTER INDEX PREPARE REMOVE](/docs/surrealql/statements/alter/indexes/#prepare-remove-clause) statement. This will decommission the index, after which you can test out queries to see their behaviour as they would function after the index is removed. If acceptable then the index can then be removed, or the change can be reverted by [rebuilding the index](/docs/surrealql/statements/rebuild).

Another `REMOVE` statement to note is `REMOVE FIELD`, as it does not remove any existing data. To remove the existing data, perform an `UPDATE` or `UPSERT` statement that uses `UNSET` on the field or sets the field's value to `NONE`.

For a schemaless table, the existing data will remain present until unset.

```surql
DEFINE FIELD name ON person TYPE string;
CREATE person:one SET name = "Billy";
REMOVE FIELD name ON person;

SELECT * FROM person; -- 'name' data is still there
UPDATE person; -- Does nothing
-- [{ id: person:one, name: 'Billy' }]
UPDATE person SET name = NONE; -- Must unset to remove 'name' data
```

For a schemafull table, read operations can be performed on a table that still contains data not defined in the schema. However, any updates will fail unless the field is unset to match the schema.

```surql
DEFINE TABLE person SCHEMAFULL;
DEFINE FIELD name ON person TYPE string;
CREATE person:one SET name = "Billy";
REMOVE FIELD name ON person;

SELECT * FROM person; -- 'name' data is still there
UPDATE person; -- Found field 'name', but no such field exists for table 'person'
DEFINE FIELD created_at ON person TYPE datetime; -- Define a new field

-- Works because values matche schema: 'name' is set to NONE, 'created_at' has a datetime value
UPDATE person SET name = NONE, created_at = time::now();
```

## return.mdx

---
sidebar_position: 20
sidebar_label: RETURN
title: RETURN statement | SurrealQL
description: The RETURN statement can be used to return an implicit value or the result of a query, and to set the return value for a transaction, block or function.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `RETURN` statement

The `RETURN` statement can be used to return an implicit value or the result of a query, and to set the return value for a transaction, block, or function.

### Statement syntax

<Tabs syncKey="return-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
RETURN @value
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const returnAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "RETURN" },
      { type: "NonTerminal", text: "@value" }
    ]}
  ]
};

<RailroadDiagram ast={returnAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage
### Basic usage

`RETURN` is always followed by a value. As every data type in SurrealDB is a type of [value](/docs/surrealql/datamodel/values), the `RETURN` statement can return anything from simple values to the result of queries.

```surql
/**[test]

[[test.results]]
value = "123"

[[test.results]]
value = "'I am a string!'"

[[test.results]]
value = "{ prop: 'value' }"

[[test.results]]
value = "[]"

[[test.results]]
value = "[person:mbn3r0epzxiz5hoqr2ls]"
skip-record-id-key = true

*/

-- Return a simple value
RETURN 123;
RETURN "I am a string!";
RETURN {
	prop: "value"
};

-- Return the result of a query
RETURN SELECT * FROM person;
RETURN (CREATE person).id;
```

Values on their own are treated as if they have an implicit `RETURN` in front. As such, the following queries return the same output as in the previous example.

```surql
/**[test]

[[test.results]]
value = "123"

[[test.results]]
value = "'I am a string!'"

[[test.results]]
value = "{ prop: 'value' }"

[[test.results]]
value = "[]"

[[test.results]]
value = "[person:mbn3r0epzxiz5hoqr2ls]"
skip-record-id-key = true

*/

123;
"I am a string!";
{
	prop: "value"
};
SELECT * FROM person;
(CREATE person).id;
```

## Transaction return value

`RETURN` statements can set the result of any transaction. This includes transactions, blocks and functions.

```surql title="Transaction return value"
/**[test]

[[test.results]]
value = "person:79e0p1et39n2dyongjzx"
skip-record-id-key = true

*/

BEGIN TRANSACTION;

-- We are executing quite a few queries here
LET $firstname = "John";
LET $lastname = "Doe";

LET $person = CREATE ONLY person CONTENT {
	firstname: $firstname,
	lastname: $lastname,
};

-- But because we end with a RETURN query, only the person's ID will be returned
-- The results of the other queries will be omitted.
RETURN $person.id;

-- One issue with this approach is that query errors are generic.
-- To get around that, use a block, which is executed as a transaction by itself.

COMMIT TRANSACTION;
```

## Return breaks execution

<Since v="v2.0.0" />
Unlike `RETURN` in SurrealDB `1.x`, `RETURN` now breaks execution of statements, functions and transactions.

```surql title="Function return value"
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[person:e71cnhnwdb6yuz8is8kq]"
skip-record-id-key = true

[[test.results]]
value = "[{ firstname: 'Thanos', id: person:e71cnhnwdb6yuz8is8kq, lastname: 'Johnson' }]"
skip-record-id-key = true

*/

DEFINE FUNCTION fn::person::create($firstname: string, $lastname: string) {
	LET $person = CREATE person CONTENT {
		firstname: $firstname,
		lastname: $lastname,
	};

	-- The RETURN statement will set the return value of the custom function, and further queries will not be executed.
	RETURN $person.id;

    -- This query will never be executed
    CREATE person SET firstname = "Stephen", lastname = "Strange";
};

fn::person::create("Thanos", "Johnson");
SELECT * FROM person;
```

```surql title="Functions"
DEFINE FUNCTION fn::round::up($num: number) {
    IF $num % 2 == 0 {
        RETURN $num; -- Breaks execution for the function
    };

    -- This is only executed if the RETURN inside the IF statement did not break execution
    RETURN $num + 1;
};
```

```surql title="Transactions"
BEGIN;
RETURN 1; -- Is executed
CREATE a; -- Is not executed
RETURN 2; -- Is not executed
COMMIT;
```

Lastly, if not executed inside a transaction or function, `RETURN` will break execution until the most top-level statement it is executed in. RETURN will **not** prevent top level statements from being executed, nor will it adjust their output.

```surql title="Statements"
LET $id = 123;
LET $id = {
    IF $id {
        RETURN type::record('table', $id);
    };

    RETURN table:rand();
};

-- This still executes. The `RETURN` statement only broke until the block in the variable assignment.
$id;
```


## select.mdx

---
sidebar_position: 21
sidebar_label: SELECT
title: SELECT statement | SurrealQL
description: The SELECT statement can be used for selecting and querying data in a database.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `SELECT` statement

The `SELECT` statement can be used for selecting and querying data in a database. Each SELECT statement supports selecting from multiple targets, which can include tables, records, edges, subqueries, parameters, arrays, objects, and other values.

In the [Learn more](#learn-more) section, you can find a video that explains how to use the `SELECT` statement to retrieve and query data from SurrealDB.

### Statement syntax

<Tabs syncKey="select-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
SELECT 
    VALUE @field | @fields [ AS @alias ] [ OMIT @fields ... ]
    FROM [ ONLY ] @targets
    [ WITH [ NOINDEX | INDEX @indexes ... ]]
    [ WHERE @conditions ]
    [ SPLIT [ ON ] @field, ... ]
    [ 
		GROUP [ ALL | [ BY ] @field, ... ] | 
		ORDER [ BY ] RAND() | @field [ COLLATE ] [ NUMERIC ] [ ASC | DESC ], ...
	]
    [ LIMIT [ BY ] @limit ]
    [ START [ AT ] @start 0 ]
    [ FETCH @fields ... ]
    [ TIMEOUT @duration ]
    [ TEMPFILES ]
    [ EXPLAIN [ FULL ] ]
;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const selectAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "SELECT" },
        {
          type: "Choice",
          index: 1,
          children: [
            { type: "Sequence", children: [ { type: "Terminal", text: "VALUE" }, { type: "NonTerminal", text: "@field" } ] },
            { type: "NonTerminal", text: "@fields" }
          ]
        },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "AS" }, { type: "NonTerminal", text: "@alias" } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "OMIT" }, { type: "NonTerminal", text: "@fields ..." } ] } },
        { type: "Terminal", text: "FROM" },
        { type: "Optional", child: { type: "Terminal", text: "ONLY" } },
        { type: "NonTerminal", text: "@targets" },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "WITH" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NOINDEX" }, { type: "Sequence", children: [ { type: "Terminal", text: "INDEX" }, { type: "NonTerminal", text: "@indexes ..." } ] } ] } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "WHERE" }, { type: "NonTerminal", text: "@conditions" } ] } },
{
  type: "Optional",
  child: {
    type: "Choice",
    index: 1,
    children: [
      {
        type: "Sequence",
        children: [
          { type: "Terminal", text: "SPLIT" },
          { type: "Optional", child: { type: "Terminal", text: "ON" } },
          { type: "NonTerminal", text: "@field, ..." }
        ]
      },
      {
        type: "Sequence",
        children: [
          { type: "Terminal", text: "GROUP" },
          {
            type: "Choice",
            index: 1,
            children: [
              { type: "Terminal", text: "ALL" },
              {
                type: "Sequence",
                children: [
                  { type: "Optional", child: { type: "Terminal", text: "BY" } },
                  { type: "NonTerminal", text: "@field, ..." }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
},
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "ORDER" }, { type: "Optional", child: { type: "Terminal", text: "BY" } }, { type: "Choice", index: 1, children: [ { type: "Sequence", children: [ { type: "NonTerminal", text: "@field" }, { type: "Optional", child: { type: "Terminal", text: "COLLATE" } }, { type: "Optional", child: { type: "Terminal", text: "NUMERIC" } }, { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "ASC" }, { type: "Terminal", text: "DESC" } ] } } ] }, { type: "Terminal", text: "RAND()" } ] } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "LIMIT" }, { type: "Optional", child: { type: "Terminal", text: "BY" } }, { type: "NonTerminal", text: "@limit" } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "START" }, { type: "Optional", child: { type: "Terminal", text: "AT" } }, { type: "NonTerminal", text: "@start" } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "FETCH" }, { type: "NonTerminal", text: "@fields ..." } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TIMEOUT" }, { type: "NonTerminal", text: "@duration" } ] } },
        { type: "Optional", child: { type: "Terminal", text: "TEMPFILES" } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "EXPLAIN" }, { type: "Optional", child: { type: "Terminal", text: "FULL" } } ] } },
        { type: "Terminal", text: ";" },
      ]
    }
  ]
};

<RailroadDiagram ast={selectAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage
### Basic usage

By default, SurrealDB returns an array of JSON-like objects called records instead of a tabular structure of rows and columns.

```surql
/**[test]

[[test.results]]
value = "[{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', id: person:tobie, name: { first: 'Tobie' } }]"

[[test.results]]
value = "[{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', id: person:tobie, name: { first: 'Tobie' } }]"

[[test.results]]
value = "[{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', name: { first: 'Tobie' } }]"

[[test.results]]
value = "[{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', id: person:tobie, name: { first: 'Tobie' } }]"

[[test.results]]
value = "[{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', name: { first: 'Tobie' } }]"

[[test.results]]
value = "{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', id: person:tobie, name: { first: 'Tobie' } }"

*/

CREATE person:tobie SET
	name.first = "Tobie",
	address = "1 Bagshot Row",
	email = "tobie@surrealdb.com";

-- Select all fields from a table
SELECT * FROM person;

-- Select specific fields from a table
SELECT name, address, email FROM person;

-- Select all fields from a specific record
SELECT * FROM person:tobie;

-- Select specific fields from a specific record
SELECT name, address, email FROM person:tobie;

-- Select just a single record
-- Using the ONLY keyword, just an object
-- for the record in question will be returned.
-- This, instead of an array with a single object.
SELECT * FROM ONLY person:tobie;
```

An alias can be used to rename fields or change the structure of an object.

```surql
SELECT * FROM person;

-- Field `address` now shows up as "string::uppercase"
-- name.first structure now flattened into a simple field
SELECT
	name.first AS user_name,
	string::uppercase(address)
FROM person;

-- "Morgan Hitchcock" added to `name` field structure,
-- `angry_address` for field name instead of automatically
-- generated "string::uppercase(address) + '!!!'"
SELECT
	name.first,
	"Morgan Hitchcock" AS name.last,
	string::uppercase(address) + "!!!" AS angry_address
FROM person;
```

```surql title="Output"
-------- Query --------

[
	{
		address: '1 Bagshot Row',
		email: 'tobie@surrealdb.com',
		id: person:tobie,
		name: {
			first: 'Tobie'
		}
	}
]

-------- Query --------

[
	{
		"string::uppercase": '1 BAGSHOT ROW',
		user_name: 'Tobie'
	}
]

-------- Query --------

[
	{
		angry_address: '1 BAGSHOT ROW!!!',
		name: {
			first: 'Tobie',
			last: 'Morgan Hitchcock'
		}
	}
]
```

SurrealDB can also return specific fields as an array of values instead of the default array of objects. This only works if you select a single un-nested field from a table or a record.

```surql
-- Select the values of a single field from a table
SELECT VALUE name FROM person;

-- Select the values of a single field from a specific record
SELECT VALUE name FROM person:00e1nc508h9f7v63x72O;
```

### Advanced expressions

SELECT queries support advanced expression in the field projections.

```surql
-- Select nested objects/values
SELECT address.city FROM person;

-- Select all nested array values
-- note the .* syntax works to select everything from an array or object-like values
SELECT address.*.coordinates AS coordinates FROM person;
-- Equivalent to
SELECT address.coordinates AS coordinates FROM person;

-- Select one item from an array
SELECT address.coordinates[0] AS latitude FROM person;

-- Select unique values from an array
SELECT array::distinct(tags) FROM article;

-- Select unique values from a nested array across an entire table
SELECT array::group(tags) AS tags FROM article GROUP ALL;

-- Use mathematical calculations in a select expression
SELECT
	(( celsius * 1.8 ) + 32) AS fahrenheit
	FROM temperature;

-- Return boolean expressions with an alias
SELECT rating >= 4 as positive FROM review;

-- Select manually generated object structure
SELECT
	{ weekly: false, monthly: true } AS `marketing settings`
FROM user;

-- Select filtered nested array values
SELECT address[WHERE active = true] FROM person;

-- Select a person who has reacted to a post using a celebration
-- Path can be conceptualized as:
-- person->(reacted_to WHERE type='celebrate')->post
SELECT * FROM person WHERE ->(reacted_to WHERE type='celebrate')->post;

-- Select a remote field from connected out graph edges
SELECT ->likes->friend.name AS friends FROM person:tobie;

-- Use the result of a subquery as a returned field
SELECT *, (SELECT * FROM events WHERE type = 'activity' LIMIT 5) AS history FROM user;

-- Restructure objects in a select expression after `.` operator
SELECT address.{city, country} FROM person;
```

## Using parameters

Parameters can be used like variables to store a value which can then be used in a subsequent query.

More info on the `$parent` parameter in the second example can be seen on [the page for predefined variables](/docs/surrealql/parameters).

```surql
-- Store the subquery result in a variable and query that result.
LET $avg_price = (
	SELECT math::mean(price) AS avg_price FROM product GROUP ALL
).avg_price;

-- Find the name of the product where the price is higher than the avg price
SELECT name FROM product
WHERE [price] > $avg_price;

-- Use the parent instance's field in a subquery (predefined variable)
SELECT *, (SELECT * FROM events WHERE host == $parent.id) AS hosted_events FROM user;
```

## Numeric ranges in a `WHERE` clause

<Since v="v2.0.0" />

A numeric range inside a `WHERE` clause can improve performance if the range is able to replace multiple checks on a certain condition. The following code should show a modest but measurable improvement in performance between the first and second `SELECT` statement, as only one condition needs to be checked instead of two.

```surql
DELETE person;
CREATE |person:20000| SET age = (rand::float() * 120).round() RETURN NONE;

-- Assign output to a parameter so the SELECT output is not displayed
LET $_ = SELECT * FROM person WHERE age > 18 AND age < 65;
LET $_ = SELECT * FROM person WHERE age in 18..=65;
```

A numeric range inside a `WHERE` also tends to produce shorter code that is easier to read and maintain.

```surql
SELECT * FROM person WHERE age >= 18 AND age <= 65;
SELECT * FROM person WHERE age IN 18..=65;
```

## Record ranges

SurrealDB supports the ability to query a range of records, using the record ID. The record ID ranges, retrieve records using the natural sorting order of the record IDs. These range queries can be used to query a range of records in a timeseries context. You can see more here about [array-based Record IDs](/docs/surrealql/datamodel/ids#array-based-record-ids).

```surql
-- Select all person records with IDs between the given range
SELECT * FROM person:1..1000;
-- Select all records for a particular location, inclusive
SELECT * FROM temperature:['London', NONE]..=['London', time::now()];
-- Select all temperature records with IDs less than a maximum value
SELECT * FROM temperature:..['London', '2022-08-29T08:09:31'];
-- Select all temperature records with IDs greater than a minimum value
SELECT * FROM temperature:['London', '2022-08-29T08:03:39']..;
-- Select all temperature records with IDs between the specified range
SELECT * FROM temperature:['London', '2022-08-29T08:03:39']..['London', '2022-08-29T08:09:31'];
```

Using a record range is more performant than the `WHERE` clause, as it does not require a table scan.

```surql
-- Create 5000 `person` records
CREATE |person:1..5000| RETURN NONE;

-- Set the starting time
LET $now = time::now();
-- Put the output somewhere so it won't clutter the screen
LET $_ = SELECT * FROM person:1..5000;
-- Get the elapsed time
LET $time1 = time::now() - $now;

LET $now = time::now();
LET $_ = SELECT * FROM person WHERE id >= 1 and id <= 5000;
LET $time2 = time::now() - $now;
RETURN [$time1, $time2];
```



## Skip certain fields using the `OMIT` clause

Sometimes, especially with tables containing numerous columns, it is desirable to select all columns except a few specific ones. The `OMIT` clause can be used in this case.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:tobie, name: 'Tobie', opts: { enabled: true, security: 'secure' }, password: '123456' }]"

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', opts: { enabled: false, security: 'secure' }, password: 'asdfgh' }]"

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', opts: { enabled: false, security: 'secure' }, password: 'asdfgh' }, { id: person:tobie, name: 'Tobie', opts: { enabled: true, security: 'secure' }, password: '123456' }]"

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', opts: { enabled: false } }, { id: person:tobie, name: 'Tobie', opts: { enabled: true } }]"

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', opts: {  } }, { id: person:tobie, name: 'Tobie', opts: {  } }]"

*/

CREATE person:tobie SET
	name = 'Tobie',
	password = '123456',
	opts.security = 'secure',
	opts.enabled = true;
CREATE person:jaime SET
	name = 'Jaime',
	password = 'asdfgh',
	opts.security = 'secure',
	opts.enabled = false;

SELECT * FROM person;
-- Omit the password field and security field in the options object
SELECT * OMIT password, opts.security FROM person;

-- Using destructuring syntax
SELECT * OMIT password, opts.{ security, enabled } FROM person;
```

## More on using the `FROM` clause

The `FROM` clause can be used on targets beyond just a single table or record name.

```surql
-- Selects all records from both 'user' and 'admin' tables.
SELECT * FROM user, admin;

-- Selects all records from the table named in the variable '$table',
-- but only if the 'admin' field of those records is true.
-- Equivalent to 'SELECT * FROM user WHERE admin = true'.
LET $table = "user";
SELECT * FROM type::table($table) WHERE admin = true;

-- Selects a single record from:
-- * the table named in the variable '$table',
-- * and the identifier named in the variable '$id'.
-- This query is equivalent to 'SELECT * FROM user:admin'.
LET $table = "user";
LET $id = "admin";
SELECT * FROM type::record($table, $id);

-- Selects all records for specific users 'tobie' and 'jaime',
-- as well as all records for the company 'surrealdb'.
SELECT * FROM user:tobie, user:jaime, company:surrealdb;

-- Selects records from a list of identifiers. The identifiers can be numerical,
-- string, or specific records such as 'person:lrym5gur8hzws72ux5fa'.
SELECT * FROM [3648937, "test", person:lrym5gur8hzws72ux5fa, person:4luro9170uwcv1xrfvby];

-- Selects data from an object that includes a 'person' key,
-- which is associated with a specific person record, and an 'embedded' key set to true.
SELECT * FROM { person: person:lrym5gur8hzws72ux5fa, embedded: true };

-- This command first performs a subquery, which selects all 'user' records and adds a
-- computed 'adult' field that is true if the user's 'age' is 18 or older.
-- The main query then selects all records from this subquery where 'adult' is true.
SELECT * FROM (SELECT age >= 18 AS adult FROM user) WHERE adult = true;
```

## Filter queries using the `WHERE` clause

As with traditional SQL queries, a SurrealDB SELECT query supports conditional filtering using a `WHERE` clause. If the expression in the `WHERE` clause [is truthy](/docs/surrealql/datamodel/values#values-and-truthiness) (is present and not an empty value), then the respective record will be returned.

```surql
-- Simple conditional filtering
SELECT * FROM article WHERE published = true;

-- Conditional filtering based on graph edges
SELECT * FROM profile WHERE count(->experience->organisation) > 3;

-- Conditional filtering based on graph edge properties
SELECT * FROM person WHERE ->(reaction WHERE type='celebrate')->post;

-- Conditional filtering with boolean logic
SELECT * FROM user WHERE (admin AND active) OR owner = true;

-- Select filtered nested array values
SELECT address[WHERE active = true] FROM person;

-- Select names for 'person' records as long as 'name' is present
-- and not an empty string ""
SELECT name FROM person WHERE name;
```

## The `SPLIT` clause

As SurrealDB supports arrays and nested fields within arrays, it is possible to use the [`SPLIT`](/docs/surrealql/clauses/split) clause to split the result on a specific field name, returning each value in an array as a separate value, along with the record content itself. This is useful in data analysis contexts.

```surql
/**[test]

[[test.results]]
value = "[{ emails: ['me@me.com', 'longer_email@other_service.com'], id: user:czs7vds6shgb6m2hbwgj, name: 'Name' }]"

[[test.results]]
value = "[{ emails: 'me@me.com', id: user:czs7vds6shgb6m2hbwgj, name: 'Name' }, { emails: 'longer_email@other_service.com', id: user:czs7vds6shgb6m2hbwgj, name: 'Name' }]"

*/

CREATE user SET
    name = "Name",
    emails = ["me@me.com", "longer_email@other_service.com"];

-- Split the results by each value in an array
SELECT * FROM user SPLIT emails;
```

```surql title="Output"
[
	{
		emails: 'me@me.com',
		id: user:tr5sxe8iygdco05faoh0,
		name: 'Name'
	},
	{
		emails: 'longer_email@other_service.com',
		id: user:tr5sxe8iygdco05faoh0,
		name: 'Name'
	}
]
```

Other examples using the `SPLIT` clause:

```surql
-- Split the results by each value in a nested array
SELECT * FROM country SPLIT locations.cities;

-- Filter the result of a subquery
SELECT * FROM (SELECT * FROM person SPLIT loggedin) WHERE loggedin > '2023-05-01';
```

## The `GROUP BY` and `GROUP ALL` clause

SurrealDB supports data aggregation and grouping, with support for multiple fields, nested fields, and aggregate functions. In SurrealDB, every field which appears in the field projections of the select statement (and which is not an aggregate function), must also be present in the [`GROUP BY`](/docs/surrealql/clauses/group-by) clause.

```surql
-- Group records by a single field
SELECT country FROM user GROUP BY country;

-- Group results by a nested field
SELECT settings.published FROM article GROUP BY settings.published;

-- Group results by multiple fields
SELECT gender, country, city FROM person GROUP BY gender, country, city;

-- Use an aggregate function to select unique values from a nested array across an entire table
SELECT array::group(tags) AS tags FROM article GROUP ALL;
```

A longer example of grouping using aggregate functions:

```surql
/**[test]

[[test.results]]
value = "[{ age: 20, country: 'Japan', gender: 'M', id: person:btiyllegvd4qiy1vspj8 }, { age: 25, country: 'Japan', gender: 'M', id: person:mf73xwobyhqyxc4suxdl }, { age: 23, country: 'US', gender: 'F', id: person:45u27tit21osibawb3zm }, { age: 30, country: 'US', gender: 'F', id: person:t950ztxmelm5nlqzuplz }, { age: 25, country: 'Korea', gender: 'F', id: person:t8toz5e3muclr3hxe74z }, { age: 45, country: 'UK', gender: 'F', id: person:s6mme1k3txzppbg5zx92 }]"
skip-record-id-key = true

[[test.results]]
value = "[{ average_age: 25, country: 'Korea', gender: 'F', total: 1 }, { average_age: 45, country: 'UK', gender: 'F', total: 1 }, { average_age: 26.5f, country: 'US', gender: 'F', total: 2 }, { average_age: 22.5f, country: 'Japan', gender: 'M', total: 2 }]"

[[test.results]]
value = "[{ number_of_records: 6 }]"

*/

INSERT INTO person [
    { gender: "M", age: 20, country: "Japan" },
    { gender: "M", age: 25, country: "Japan" },
    { gender: "F", age: 23, country: "US" },
    { gender: "F", age: 30, country: "US" },
    { gender: "F", age: 25, country: "Korea" },
    { gender: "F", age: 45, country: "UK" },
];

SELECT
	count() AS total,
	math::mean(age) AS average_age,
	gender,
	country
FROM person
GROUP BY gender, country;

-- Get the total number of records in a table
SELECT count() AS number_of_records FROM person GROUP ALL;
```

```surql title="Output"
-------- Query --------

[
	{
		average_age: 25,
		country: 'Korea',
		gender: 'F',
		total: 1
	},
	{
		average_age: 45,
		country: 'UK',
		gender: 'F',
		total: 1
	},
	{
		average_age: 26,
		country: 'US',
		gender: 'F',
		total: 2
	},
	{
		average_age: 22,
		country: 'Japan',
		gender: 'M',
		total: 2
	}
]

-------- Query --------

[
	{
		number_of_records: 6
	}
]
```

### `GROUP` and `SPLIT` incompatibility

The `GROUP` and `SPLIT` clauses are incompatible with each other due to opposing behaviour: while `SPLIT` is a post-processing clause that multiplies the output of a query, `GROUP` works in the other way by collapsing the output.

Versions before 3.0.0-beta allowed these two clauses to be used together, after which attempting to do so results in a parsing error.

```surql
SELECT * FROM person SPLIT name GROUP BY name;
```

```surql title="Output"
'Parse error: SPLIT and GROUP are mutually exclusive
 --> [6:22]
  |
6 | SELECT * FROM person SPLIT name GROUP BY name;
  |                      ^^^^^^^^^^ SPLIT cannot be used with GROUP
 --> [6:33]
  |
6 | SELECT * FROM person SPLIT name GROUP BY name;
  |                                 ^^^^^^^^^^^^^ GROUP cannot be used with SPLIT
'
```

Disallowing the two clauses together forces a query that uses both to have one inside a subquery, which makes it clear which operation is to be performed first.

```surql
CREATE user SET
    name = "Jack",
    emails = ["my@firstemail.com", "another@builder.com"],
    age = 37;

CREATE user SET
    name = "Ellen",
    emails = ["ruler@forest.com", "wife@tom.com"],
    age = 50;

CREATE user SET
    name = "Phillip",
    emails = ["prior@kingsbridge.com", "boss@remigius.com"],
    age = 50;

SELECT age, emails FROM (SELECT * FROM user SPLIT emails) GROUP BY age;

SELECT age, emails
FROM (
  SELECT age, array::group(emails) AS emails
  FROM user
  GROUP BY age
)
SPLIT emails;
```

### Using a `COUNT` index to speed up `count()` in `GROUP ALL` queries

<Since v="v3.0.0" />

To speed up the `count()` function along with `GROUP ALL` to get the total number of records in a table, a `COUNT` index can be used. This keeps track of the total number of records as a single value as opposed to a dynamic iteration of the table to get the full count every time a query is run.

```surql
DEFINE INDEX person_count ON person COUNT;
SELECT count() AS number_of_records FROM person GROUP ALL;
```

### `math::stddev()` and `math::variance()` in table views

<Since v="v3.0.0" />

The `math::stddev()` and `math::variance()` functions can also be used in table views.

```surql
DEFINE TABLE person SCHEMALESS;
DEFINE TABLE person_stats AS
	SELECT
		count(),
		age,
		math::stddev(score) AS score_stddev,
		math::variance(score) AS score_variance
	FROM person
	GROUP BY age;

INSERT INTO person [
    { id: person:alice,          age: 25, score: 80 },
    { id: person:alices_rival,   age: 25, score: 88 },
    { id: person:bob,            age: 24, score: 90 },
    { id: person:bobs_rival,     age: 24, score: 99 },
    { id: person:charlie,        age: 23, score: 70 },
    { id: person:charlies_rival, age: 23, score: 77 }
];

SELECT * FROM person_stats WHERE age >= 24;
```

Output:

```surql
[
	{
		age: 24,
		count: 2,
		id: person_stats:[
			24
		],
		score_stddev: 6.363961030678927719607599259dec,
		score_variance: 40.50dec
	},
	{
		age: 25,
		count: 2,
		id: person_stats:[
			25
		],
		score_stddev: 5.656854249492380195206754897dec,
		score_variance: 32dec
	}
]
```

## Sort records using the `ORDER BY` clause

To sort records, SurrealDB allows ordering on multiple fields and nested fields. Use the `ORDER BY` clause to specify a comma-separated list of field names that should be used to order the resulting records. The `ASC` and `DESC` keywords can be used to specify whether results should be sorted in an ascending or descending manner. The `COLLATE` keyword can be used to use Unicode collation when ordering text in string values, ensuring that different cases, and different languages are sorted in a consistent manner. Finally, the `NUMERIC` can be used to correctly sort text which contains numeric values.

```surql
-- Order records randomly
SELECT * FROM user ORDER BY rand();

-- Order records descending by a single field
SELECT * FROM song ORDER BY rating DESC;

-- Order records by multiple fields independently
SELECT * FROM song ORDER BY artist ASC, rating DESC;

-- Order text fields with Unicode collation
SELECT * FROM article ORDER BY title COLLATE ASC;

-- Order text fields with which include numeric values
SELECT * FROM article ORDER BY title NUMERIC ASC;
```

## The `LIMIT` clause

To limit the number of records returned, use the `LIMIT` clause.

```surql
-- Select only the top 50 records from the person table
SELECT * FROM person LIMIT 50;
```

When using the `LIMIT` clause, it is possible to paginate results by using the `START` clause to start from a specific record from the result set. It is important to note that the `START` count starts from 0.

```surql
-- Start at record 50 and select the following 50 records
SELECT * FROM user LIMIT 50 START 50;
```

The `LIMIT` clause followed by 1 is often used along with the `ONLY` clause to satisfy the requirement that only up to a single record can be returned.

```surql
-- Record IDs are unique so guaranteed to be no more than 1
SELECT * FROM ONLY person:jamie;

-- Error because no guarantee that this will return a single record
SELECT * FROM ONLY person WHERE name = "Jaime";

-- Add `LIMIT 1` to ensure that only up to one record will be returned
SELECT * FROM ONLY person WHERE name = "Jaime" LIMIT 1;
```

```surql
/**[test]

[[test.results]]
value = "[5, 6, 7, 8, 9]"

*/

-- Select the first 5 records from the array
SELECT * FROM [1,2,3,4,5,6,7,8,9,10] LIMIT 5 START 4; 
```

```surql title="Result"
[
	5,
	6,
	7,
	8,
	9
]
```

## Connect targets using the FETCH clause

Two of the most powerful features in SurrealDB are [record links](/docs/surrealql/datamodel/records) and [graph connections](/docs/surrealql/statements/relate).

Instead of pulling data from multiple tables and merging that data together, SurrealDB allows you to traverse related records efficiently without needing to use JOINs.

To fetch and replace records with the remote record data, use the [`FETCH`](/docs/surrealql/clauses/fetch) clause to specify the fields and nested fields which should be fetched in-place, and returned in the final statement response output.

```surql
-- Select all the review information
-- and the artist's email from the artist table
SELECT *, artist.email FROM review FETCH artist;

-- Select all the article information
-- only if the author's age (from the author table) is under 30.
SELECT * FROM article WHERE author.age < 30 FETCH author;
```

## The `TIMEOUT` clause

When processing a large result set with many interconnected records, it is possible to use the `TIMEOUT` keyword to specify a timeout duration for the statement. If the statement continues beyond this duration, then the transaction will fail, and the statement will return an error.

```surql
-- Cancel this conditional filtering based on graph edge properties
-- if it's not finished within 5 seconds
SELECT * FROM person WHERE ->knows->person->(knows WHERE influencer = true) TIMEOUT 5s;
```

## The `TEMPFILES` clause

<Since v="v2.0.0" />

When processing a large result set with many records, it is possible to use the `TEMPFILES` clause to specify that the statement should be processed in temporary files rather than memory.

This significantly reduces memory usage in exchange for slower performance.

```surql
-- Select every person and order them by name using temporary files rather than memory.
SELECT * FROM person ORDER BY name TEMPFILES;
```

This requires the temporary directory to be set in the server configuration or when using the [`surreal start`](/docs/surrealdb/cli/start) command.

## The `EXPLAIN` clause

When `EXPLAIN` is used, the `SELECT` statement returns an explanation, essentially revealing the execution plan to provide transparency and understanding of the query performance. `EXPLAIN` can be followed by `FULL` to see the number of executed rows.

Here is the result when the field 'email' is not indexed. We can see that the execution plan will iterate over the whole table.

```surql
/**[test]

[[test.results]]
value = "[{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', id: person:tobie, name: 'Tobie' }]"

[[test.results]]
value = "[{ detail: { direction: 'forward', table: 'person' }, operation: 'Iterate Table' }, { detail: { type: 'Memory' }, operation: 'Collector' }]"

[[test.results]]
value = "[{ detail: { direction: 'forward', table: 'person' }, operation: 'Iterate Table' }, { detail: { type: 'Memory' }, operation: 'Collector' }, { detail: { type: 'KeysAndValues' }, operation: 'RecordStrategy' }, { detail: { count: 1 }, operation: 'Fetch' }]"

*/

CREATE person:tobie SET
	name = "Tobie",
	address = "1 Bagshot Row",
	email = "tobie@surrealdb.com";

SELECT * FROM person WHERE email='tobie@surrealdb.com' EXPLAIN;
SELECT * FROM person WHERE email='tobie@surrealdb.com' EXPLAIN FULL;
```

```surql title="Output"
-------- Query --------

[
	{
		detail: {
			table: 'person'
		},
		operation: 'Iterate Table'
	},
	{
		detail: {
			type: 'Memory'
		},
		operation: 'Collector'
	}
]

-------- Query --------

[
	{
		detail: {
			table: 'person'
		},
		operation: 'Iterate Table'
	},
	{
		detail: {
			type: 'Memory'
		},
		operation: 'Collector'
	},
	{
		detail: {
			count: 1
		},
		operation: 'Fetch'
	}
]
```

Here is the result when the 'email' field is indexed. We can see that the execution plan will proceed by utilizing the index.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', id: person:tobie, name: 'Tobie' }]"

[[test.results]]
value = "[{ detail: { plan: { index: 'fast_email', operator: '=', value: 'tobie@surrealdb.com' }, table: 'person' }, operation: 'Iterate Index' }, { detail: { type: 'Memory' }, operation: 'Collector' }]"

[[test.results]]
value = "[{ detail: { plan: { index: 'fast_email', operator: '=', value: 'tobie@surrealdb.com' }, table: 'person' }, operation: 'Iterate Index' }, { detail: { type: 'Memory' }, operation: 'Collector' }, { detail: { type: 'KeysAndValues' }, operation: 'RecordStrategy' }, { detail: { count: 1 }, operation: 'Fetch' }]"

*/

DEFINE INDEX fast_email ON TABLE person FIELDS email;

CREATE person:tobie SET
	name = "Tobie",
	address = "1 Bagshot Row",
	email = "tobie@surrealdb.com";

SELECT * FROM person WHERE email='tobie@surrealdb.com' EXPLAIN;
SELECT * FROM person WHERE email='tobie@surrealdb.com' EXPLAIN FULL;
```

```surql title="Output"
-------- Query --------

[
	{
		detail: {
			plan: {
				index: 'fast_email',
				operator: '=',
				value: 'tobie@surrealdb.com'
			},
			table: 'person'
		},
		operation: 'Iterate Index'
	},
	{
		detail: {
			type: 'Memory'
		},
		operation: 'Collector'
	}
]

-------- Query --------

[
	{
		detail: {
			plan: {
				index: 'fast_email',
				operator: '=',
				value: 'tobie@surrealdb.com'
			},
			table: 'person'
		},
		operation: 'Iterate Index'
	},
	{
		detail: {
			type: 'Memory'
		},
		operation: 'Collector'
	},
	{
		detail: {
			count: 1
		},
		operation: 'Fetch'
	}
]
```

## The `WITH` clause

The query planner can replace the standard table iterator with one or several index iterators based on the structure and requirements of the query. However, there may be situations where manual control over these potential optimizations is desired or required.

For instance, the cardinality of an index can be high, potentially even equal to the number of records in the table. The sum of the records iterated by several indexes may end up being larger than the number of records obtained by iterating over the table. In such cases, if there are different index possibilities, the most probable optimal choice would be to use the index known with the lowest cardinality.

- `WITH INDEX @indexes ...` restricts the query planner to using only the specified index(es)
- `WITH NOINDEX` forces the query planner to use the table iterator.

```surql
-- forces the query planner to use the specified index(es):
SELECT * FROM person
WITH INDEX ft_email
WHERE
	email = 'tobie@surrealdb.com' AND
	company = 'SurrealDB';

-- forces the usage of the table iterator
SELECT name FROM person WITH NOINDEX WHERE job = 'engineer' AND gender = 'm';
```

## The `ONLY` clause

If you are selecting just one single resource, it's possible to use the `ONLY` clause to filter that result from an array.

```surql
SELECT * FROM ONLY person:john;
```

If you are selecting from a resource where it is possible that multiple resources are returned, it is required to `LIMIT` the result to just one.
This is needed, because the query would otherwise not be deterministic.

```surql
-- Fails
SELECT * FROM ONLY table_name;
-- Succeeds
SELECT * FROM ONLY table_name LIMIT 1;
```

## The `VERSION` clause

<Since v="v2.0.0" />

When you are starting a new database with memory or [SurrealKV as the storage engine](/docs/surrealdb/installation/running/surrealkv) with versioning enabled, you can specify a version for each record. This is useful for time-travel queries. You can query a specific version of a record by using the `VERSION` clause. The `VERSION` clause is always followed by a [datetime](/docs/surrealql/datamodel/datetimes) and when the specified timestamp does not exist, an empty array is returned.


> [!NOTE]
> The `VERSION` clause is currently in alpha and is subject to change. We do not recommend this for production.

```surql
-- Create a new record
CREATE user:john SET name = 'John' VERSION d'2025-08-19T08:00:00Z';
[[{ id: user:john, name: 'John' }]]

-- Select the record as it is now
SELECT * FROM user:john;
[[{ id: user:john, name: 'John' }]]

-- Select the record as it was at a specific point in time
SELECT * FROM user:john VERSION d'2025-08-19T08:00:00Z';
[[{ id: user:john, name: 'John' }]]

-- Select the record as it was at a specific point in time that doesn't exist
SELECT * FROM user:john VERSION d'2025-08-19T07:00:00Z';
[[]]

-- Update the record to the user john
update user:john Set hight ="55"
[[{ hight: '55', id: user:john, name: 'John' }]]

-- Confirm that the record is updated
SELECT * FROM user:john;
[[{ hight: '55', id: user:john, name: 'John' }]]

-- Select the record for the timestamp before the update
SELECT * FROM user:john VERSION d'2025-08-19T08:00:00Z';
[[{ id: user:john, name: 'John' }]]
```

<Since v="v2.1.0" />

The `VERSION` clause can also take a dynamic value or parameter that resolves to a datetime.

```surql
SELECT * FROM user VERSION time::now();

LET $now = time::now();
SELECT * FROM user VERSION $now;

DEFINE FUNCTION fn::yesterday() { time::now() - 1d };
SELECT * FROM user VERSION fn::yesterday();
```

## Selecting inside graph queries

<Since v="v2.2.0" />

A `SELECT` statement and/or its clauses can be used inside graph queries as well at the graph edge portion of the query.

```surql
-- Note: 1..4 used to be inclusive until SurrealDB 3.0.0
-- Now creates 1 up to but not including 4
CREATE |person:1..4|;

RELATE person:1->likes->person:2 SET like_strength = 20, know_in_person = true;
RELATE person:1->likes->person:3 SET like_strength = 5,  know_in_person = false;
RELATE person:2->likes->person:1 SET like_strength = 10, know_in_person = true;
RELATE person:2->likes->person:3 SET like_strength = 12, know_in_person = false;
RELATE person:3->likes->person:1 SET like_strength = 2,  know_in_person = false;
RELATE person:3->likes->person:2 SET like_strength = 9,  know_in_person = false;

SELECT ->likes AS likes FROM person;
SELECT ->(SELECT like_strength FROM likes) AS likes FROM person;
SELECT ->(SELECT like_strength FROM likes WHERE like_strength > 10) AS likes FROM person;
SELECT ->(likes WHERE like_strength > 10) AS likes FROM person;
SELECT ->(SELECT like_strength, know_in_person FROM likes ORDER BY like_strength DESC) AS likes FROM person;
SELECT ->(SELECT count() as count, know_in_person FROM likes GROUP BY know_in_person) AS likes FROM person;
SELECT ->(likes LIMIT 1) AS likes FROM person;
SELECT ->(likes START 1) AS likes FROM person;
```

For more examples, see the [graph clauses](/docs/surrealql/statements/relate#graph-clauses) section of the page on the `RELATE` statement.

## Learn more

To learn more about using the `SELECT` statement to retrieve data from SurrealDB, check out this explainer video:


<iframe width="100%" src="https://www.youtube.com/embed/TyX45cyZ-W0?si=S9M59afDEiqxeC5d" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" 
style={{aspectRatio: 1.7, paddingTop: '20px'}} allowfullscreen></iframe>



## show.mdx

---
sidebar_position: 22
sidebar_label: SHOW
title: SHOW statement | SurrealQL
description: The SHOW statement can be used to replay changes made to a table.
---

# `SHOW` statement

Change Feeds allows you to retrieve and sync changes from SurrealDB to external systems and platforms using the `SHOW` statement.

The `SHOW` statement can be used to replay changes made to a table.

import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

## Requirements

* You must first [`DEFINE`](/docs/surrealql/statements/define/table#example-usage) a changefeed on either a table or a database.

### Statement syntax

<Tabs syncKey="show-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
SHOW CHANGES FOR TABLE @tablename
	SINCE @timestamp | @versionstamp
	[ LIMIT @number ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const showAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "SHOW" },
      { type: "Terminal", text: "CHANGES" },
      { type: "Terminal", text: "FOR" },
      { type: "Terminal", text: "TABLE" },
      { type: "NonTerminal", text: "@tablename" },
      { type: "Terminal", text: "SINCE" },
      { type: "Choice", index: 1, children: [ { type: "NonTerminal", text: "@timestamp" }, { type: "NonTerminal", text: "@versionstamp" } ] },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "LIMIT" }, { type: "NonTerminal", text: "@number" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={showAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

### Basic usage

The following expression shows usage of the SHOW statement.

```surql
-- Define the change feed and its duration
DEFINE TABLE reading CHANGEFEED 3d;

-- Create some records in the reading table
CREATE reading SET story = "Once upon a time";
CREATE reading SET story = "there was a database";

-- Replay changes to the reading table since a date
SHOW CHANGES FOR TABLE reading SINCE d"2023-09-07T01:23:52Z" LIMIT 10;
-- Replay changes to the reading table since a versionstamp
SHOW CHANGES FOR TABLE reading SINCE 1 LIMIT 10;
```

Assuming the datetime above matches with the one when the changefeed was established, the response for both queries will be as follows.

```surql title="Response"
[
	{
		changes: [
			{
				define_table: {
					name: 'reading'
				}
			}
		],
		versionstamp: 65536
	},
	{
		changes: [
			{
				update: {
					id: reading:bavjgpnhkgvudfg4mg16,
					story: 'Once upon a time'
				}
			}
		],
		versionstamp: 131072
	},
	{
		changes: [
			{
				update: {
					id: reading:liq4e7hzjaw7bp5t4pn1,
					story: 'there was a database'
				}
			}
		],
		versionstamp: 196608
	}
]
```

Note the following when working with the versionstamps of a changefeed:

* Changefeeds defined on tables are implemented via a single `CHANGEFEED` on the database level. As such, `SHOW CHANGES FOR TABLE sometable` will only show versionstamps in sequential order if `sometable` is the database's only table.
* The `versionstamp` output above is due to an extra two bytes needed for more detailed ordering needed in the FoundationDB distributed [SurrealDB backend](/docs/surrealdb/introduction/architecture). To turn these versionstamps into a normal sequence of numbers, a right shift of sixteen bits (`>> 16`) can be used.
* A `SINCE <number` greater than the current sequential number will return an empty array.
* `SINCE <time>` needs to be a datetime after which the `CHANGEFEED` was defined.

Versionstamps carry the following two guarantees:

* Versionstamps monotonically increase.
* Versionstamp format is universal across various backends.


## sleep.mdx

---
sidebar_position: 23
sidebar_label: SLEEP
title: SLEEP statement | SurrealQL
description: The SLEEP statement is used to introduce a delay or pause in the execution of a query or a batch of queries for a specific amount of time.
---

# `SLEEP` statement

The `SLEEP` statement is used to introduce a delay or pause in the execution of a query or a batch of queries for a specific amount of time.

### Statement syntax

import RailroadDiagram from "@components/RailroadDiagram.astro";
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

<Tabs syncKey="sleep-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
SLEEP @duration;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const sleepAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "SLEEP" },
        { type: "NonTerminal", text: "@duration" },
        { type: "Terminal", text: ";" },
      ],
    },
  ],
};

<RailroadDiagram ast={sleepAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following query shows example usage of this statement.

```surql
-- Sleep one second
SLEEP 1s;
-- Sleep 100 milliseconds
SLEEP 100ms;
```

For more dynamic usage of sleep, see SurrealDB's [sleep](/docs/surrealql/functions/database/sleep) function.

## SLEEP during parallel operations

A `SLEEP` statement does not interfere with operations that are underway in the background, such as a [`DEFINE INDEX`](/docs/surrealql/statements/define/indexes) statement using the `CONCURRENTLY` clause.

```surql
CREATE |user:50000| SET name = id.id() RETURN NONE;
DEFINE INDEX unique_name ON TABLE user FIELDS name UNIQUE CONCURRENTLY;
INFO FOR INDEX unique_name ON TABLE user;
SLEEP 50ms;
INFO FOR INDEX unique_name ON TABLE user;
SLEEP 50ms;
INFO FOR INDEX unique_name ON TABLE user;
SLEEP 50ms;
INFO FOR INDEX unique_name ON TABLE user;
```

```surql title="Possible output"
-------- Query 1 --------
{
	building: {
		count: 0,
		status: 'initial'
	}
}

-------- Query 2 --------
{
	building: {
		count: 17250,
		status: 'initial'
	}
}

-------- Query 3 --------
{
	building: {
		count: 33542,
		status: 'initial'
	}
}

-------- Query 4 --------
{
	building: {
		status: 'built'
	}
}
```

## Use cases

`SLEEP` can be useful in a small number of situations, such as:

* Testing and debugging: can be used to understand how concurrent transactions interact, test how systems handle timeouts and delays, simulate behaviour in more distant regions with longer latency
* Throttling: can be used to throttle the execution of operations to prevent the database from being overwhelmed by too many requests at once
* Security measures: can be used to slow down the response rate of login attempts to mitigate the risk of brute force attacks


## throw.mdx

---
sidebar_position: 24
sidebar_label: THROW
title: THROW statement | SurrealQL
description: The THROW statement can be used to stop execution of a query and return information on the underlying problem
---

# `THROW` statement

The `THROW` statement can be used to throw an error in a place where something unexpected is happening. Execution of the query will be aborted and the error will be returned to the client. While a string is most commonly seen after a `THROW` statement, any [value](/docs/surrealql/datamodel/values) at all can be used as error output.

import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

### Statement syntax

<Tabs syncKey="throw-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
THROW @error
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const throwAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "THROW" },
      { type: "NonTerminal", text: "@error" }
    ]}
  ]
};

<RailroadDiagram ast={throwAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following query shows example usage of this statement.

```surql
/**[test]

[[test.results]]
error = "'An error occurred: some error message'"

*/

-- Throw an error
THROW "some error message";
```
The following query shows the `THROW` statement being used to send back a custom error to the client.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- In this example, we throw a custom error when a user provides invalid signin details
DEFINE ACCESS user ON DATABASE TYPE RECORD
	SIGNIN {
		LET $user = (SELECT * FROM user WHERE username = $username AND crypto::argon2::compare(password, $password));
		IF !$user {
			THROW "You either provided invalid credentials, or a user with the username " + <string> $username + " might not exist.";
		};

		RETURN $user;
	}
	DURATION FOR SESSION 1w
;
```

`THROW` can contain any value: arrays, objects, and so on. It can even take the value of a separate `SELECT` statement:

```surql
/**[test]

[[test.results]]
value = "[{ id: event:one, time: d'2025-10-08T07:15:04.994633Z' }]"

[[test.results]]
value = "[{ id: event:two, time: d'2025-10-08T07:15:04.996995Z' }]"

[[test.results]]
error = ""An error occurred: [{ id: event:one, time: d'2025-10-08T07:15:04.994633Z' }, { id: event:two, time: d'2025-10-08T07:15:04.996995Z' }]""

*/

CREATE event:one SET time = d'2025-10-08T07:15:04.994633Z';
CREATE event:two SET time = d'2025-10-08T07:15:04.996995Z';
THROW SELECT * FROM event;
```

```surql title="Response"
"An error occurred: [{ id: event:one, time: d'2025-10-08T07:15:04.994633Z' }, { id: event:two, time: d'2025-10-08T07:15:04.996995Z' }]"
```

`THROW` can also be used to cancel a transaction, usually inside an `IF` statement checking a condition.

```surql
BEGIN TRANSACTION;
LET $transfer_amount = 150;
CREATE account:one SET dollars =  100;
CREATE account:two SET dollars =  100;
UPDATE account:one SET dollars -= $transfer_amount;
UPDATE account:two SET dollars += $transfer_amount;
IF account:one.dollars < 0 {
    THROW "Insufficient funds, would have $" + <string>account:one.dollars + " after transfer"
};
COMMIT TRANSACTION;
SELECT * FROM account;
```

```surql title="Output when $transfer_amount set to 150"
'An error occurred: Insufficient funds, would have $-50 after transfer'
```

```surql title="Output when $transfer_amount set to 50"
[
	{
		dollars: 50,
		id: account:one
	},
	{
		dollars: 150,
		id: account:two
	}
]
```


## update.mdx

---
sidebar_position: 25
sidebar_label: UPDATE
title: UPDATE statement | SurrealQL
description: The UPDATE statement can be used to update records in the database. If they already exist, they will be updated. If they do not exist, no records will be updated.
---
import Since from '@components/shared/Since.astro'
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";
import RailroadDiagram from "@components/RailroadDiagram.astro";

# `UPDATE` statement

The `UPDATE` statement can be used to update existing records in the database. If the record does not exist, the statement will succeed but no records will be updated.

> [!NOTE]
> This statement can not be used to create graph relationships. For that, use the [`RELATE`](/docs/surrealql/statements/relate) or [`INSERT`](/docs/surrealql/statements/insert) statement.

> [!NOTE]
> UPDATE on a single record in SurrealDB 1.x will create the record if it does not exist. This behaviour is no longer the case in SurrealDB 2.0. To update and create a record if it does not exist in SurrealDB 2.0, use the [`UPSERT`](/docs/surrealql/statements/upsert) statement.

### Statement syntax

<Tabs syncKey="update-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
UPDATE [ ONLY ] @targets
	[ CONTENT @value
	  | MERGE @value
	  | PATCH @value
	  | REPLACE @value
	  | [ SET @field = @value, ... | UNSET @field, ... ]
	]
	[ WHERE @condition ]
	[ RETURN NONE | RETURN BEFORE | RETURN AFTER | RETURN DIFF | RETURN @statement_param, ... | RETURN VALUE @statement_param ]
	[ TIMEOUT @duration ]
	[ EXPLAIN [ FULL ]]
;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const updateAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "UPDATE" },
        { type: "Optional", child: { type: "Terminal", text: "ONLY" } },
        { type: "NonTerminal", text: "@targets" },
        { type: "Optional", child: { type: "Choice", index: 1, children: [
          { type: "Sequence", children: [ { type: "Terminal", text: "CONTENT" }, { type: "NonTerminal", text: "@value" } ] },
          { type: "Sequence", children: [ { type: "Terminal", text: "MERGE" }, { type: "NonTerminal", text: "@value" } ] },
          { type: "Sequence", children: [ { type: "Terminal", text: "PATCH" }, { type: "NonTerminal", text: "@value" } ] },
          { type: "Sequence", children: [ { type: "Terminal", text: "REPLACE" }, { type: "NonTerminal", text: "@value" } ] },
          { type: "Sequence", children: [ { type: "Choice", index: 1, children: [ { type: "Terminal", text: "SET" }, { type: "Terminal", text: "UNSET" } ] }, { type: "NonTerminal", text: "@field (=|,) @value ..." } ] }
        ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "WHERE" }, { type: "NonTerminal", text: "@condition" } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "RETURN" }, { type: "Choice", index: 1, children: [
          { type: "Terminal", text: "NONE" }, { type: "Terminal", text: "BEFORE" }, { type: "Terminal", text: "AFTER" }, { type: "Terminal", text: "DIFF" }, { type: "NonTerminal", text: "@statement_param, ..." }, { type: "Sequence", children: [ { type: "Terminal", text: "VALUE" }, { type: "NonTerminal", text: "@statement_param" } ] }
        ] } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TIMEOUT" }, { type: "NonTerminal", text: "@duration" } ] } },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "EXPLAIN" }, { type: "Optional", child: { type: "Terminal", text: "FULL" } } ] } },
        { type: "Terminal", text: ";" },
      ]
    }
  ]
};

<RailroadDiagram ast={updateAst} className="my-6" />

  </TabItem>
</Tabs>

> [!NOTE]
> `@target` refers to either record output including an `id` field, or a [record ID](/docs/surrealql/datamodel/ids) on its own.

## Example usage

Let's look at some examples of how to use the `UPDATE` statement. First we'll create two `person` records with the [`CREATE`](/docs/surrealql/statements/create) statement so that the examples will produce a meaningful output.

```surql
/**[test]

[[test.results]]
value = "[{ company: 'Surrealist', id: person:1iberbyuqfpoi6girgv9, name: 'John', skills: ['JavaScript', 'Go', 'SurrealQL'] }]"
skip-record-id-key = true

[[test.results]]
value = "[{ company: 'SurrealDB', id: person:tobie, name: 'Tobie', skills: ['JavaScript', 'Go', 'SurrealQL'] }]"

*/

-- Create a Schemaless person table with a random id
CREATE person CONTENT {
    name: 'John',
    company: 'Surrealist',
    skills: ['JavaScript', 'Go' , 'SurrealQL']
};

-- Create another person with a specific id
CREATE person:tobie CONTENT {
    name: 'Tobie',
    company: 'SurrealDB',
    skills: ['JavaScript', 'Go' , 'SurrealQL']
};
```

Let's say we wanted to update the `person` table with a new field `enjoys` (an array), a new skill `breathing` to the existing `skills` field (another array), add a new numeric field called `dollars`, and a `last_name` field that relies on the existing `name` field to set its value.

To do this we would use the following query.

```surql
-- Update all records in a table
-- The `enjoys` field will also be an array.
-- The += operator alone is enough to infer the type
UPDATE person SET 
	dollars = 50,
	skills += 'breathing',
	enjoys += 'reading',
	full_name = name + ' Mc' + name + 'erson';
```

```surql title="Output"
[
	{
		company: 'Surrealist',
		dollars: 50,
		enjoys: [
			'reading'
		],
		full_name: 'John McJohnerson',
		id: person:j1qov2pxey3p8s6hqlev,
		name: 'John',
		skills: [
			'JavaScript',
			'Go',
			'SurrealQL',
			'breathing'
		]
	},
	{
		company: 'SurrealDB',
		dollars: 50,
		enjoys: [
			'reading'
		],
		full_name: 'Tobie McTobieerson',
		id: person:tobie,
		name: 'Tobie',
		skills: [
			'JavaScript',
			'Go',
			'SurrealQL',
			'breathing'
		]
	}
]
```

For more specific updates, you can specify a record ID to update a single record. The following query will update the record with the ID `person:tobie` to add "Rust" as a skill.

```surql
-- Update a record with a specific string id to add a new skill: 'Rust'
UPDATE person:tobie SET skills += 'Rust';
```

```surql title="Output"
[
	{
		company: 'SurrealDB',
		dollars: 50,
		enjoys: [
			'reading'
		],
		full_name: 'Tobie McTobieerson',
		id: person:tobie,
		name: 'Tobie',
		skills: [
			'JavaScript',
			'Go',
			'SurrealQL',
			'breathing',
			'Rust'
		]
	}
]
```

The `-=` operator can be used to remove an item from an array or reduce a numeric value by a certain value.

```surql
UPDATE person:tobie SET 
	skills -= 'Go', 
	dollars -= 1;
```

```surql title="Output"
[
	{
		company: 'SurrealDB',
		dollars: 49,
		enjoys: [
			'reading'
		],
		full_name: 'Tobie McTobieerson',
		id: person:tobie,
		name: 'Tobie',
		skills: [
			'JavaScript',
			'SurrealQL',
			'breathing',
			'Rust'
		]
	}
]
```

You can also remove a field from a record using the `UNSET` keyword or by setting the field to `NONE`.

```surql
-- Remove the company field by setting it to NONE or using the UNSET keyword
UPDATE person:tobie SET company = NONE;

UPDATE person:tobie UNSET company;
```

```surql title="Output"
[
	{
		dollars: 49,
		enjoys: [
			'reading'
		],
		full_name: 'Tobie McTobieerson',
		id: person:tobie,
		name: 'Tobie',
		skills: [
			'JavaScript',
			'SurrealQL',
			'breathing',
			'Rust'
		]
	}
]
```

## Conditional Update with `WHERE` clause

The `UPDATE` statement supports conditional matching of records using a `WHERE` clause. If the expression in the `WHERE` clause evaluates to `true`, then the respective record will be updated.

```surql
-- Update all records which match the condition that `company` is not equal to "SurrealDB"
UPDATE person SET skills += "System design" WHERE company != "SurrealDB";
```

```surql title="Output"
[
	{
		company: 'Surrealist',
		dollars: 50,
		enjoys: [
			'reading'
		],
		full_name: 'John McJohnerson',
		id: person:i5z3i64cpqpo8jtr6jww,
		name: 'John',
		skills: [
			'JavaScript',
			'Go',
			'SurrealQL',
			'breathing',
			'System design'
		]
	},
	{
		dollars: 49,
		enjoys: [
			'reading'
		],
		full_name: 'Tobie McTobieerson',
		id: person:tobie,
		name: 'Tobie',
		skills: [
			'JavaScript',
			'SurrealQL',
			'breathing',
			'Rust',
			'System design'
		]
	}
]
```

## CONTENT clause

Instead of specifying record data using the `SET` clause, it is also possible to use the `CONTENT` keyword to specify the record data using a SurrealQL object.

```surql
-- Update all records with the same content
UPDATE person CONTENT {
	name: 'John',
	company: 'SurrealDB',
	skills: ['Rust', 'Go', 'JavaScript'],
};

-- Oops, now they are both named John.
-- Update a specific record with some content
UPDATE person:tobie CONTENT {
	name: 'Tobie',
	company: 'SurrealDB',
	skills: ['Rust', 'Go', 'JavaScript'],
};
```

Since version `2.1.0`, a statement with a `CONTENT` clause will bypass `READONLY` fields instead of generating an error.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ age: 90, created: d'2024-01-01T00:00:00Z', id: person:gladys }]"

[[test.results]]
value = "[{ age: 70, created: d'2024-01-01T00:00:00Z', id: person:gladys }]"

*/

DEFINE FIELD created ON person TYPE datetime DEFAULT d'2024-01-01T00:00:00Z' READONLY;
CREATE person:gladys SET age = 90;
-- Does not try to modify `created` field, no error
UPDATE person:gladys CONTENT { age: 70 };
```

<Tabs groupId="content">
<TabItem value="Before" label="Output before 2.1.0" >

```surql
-------- Query --------
[
	{
		age: 90,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]

-------- Query --------
'Found changed value for field `created`, with record `person:gladys`, but field is readonly'
```
</TabItem>

<TabItem value="After" label="Output after 2.1.0" >

```surql
-------- Query --------
[
	{
		age: 90,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]

-------- Query --------
[
	{
		age: 70,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]
```
</TabItem>
</Tabs>

## REPLACE clause

Originally an alias for `CONTENT`, the `REPLACE` clause maintains the previous behaviour regarding `READONLY` fields. If the content following `REPLACE` does not match a record's `READONLY` fields, an error will be generated.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ age: 90, created: d'2024-01-01T00:00:00Z', id: person:gladys }]"

[[test.results]]
error = "'Found changed value for field `created`, with record `person:gladys`, but field is readonly'"

[[test.results]]
value = "[{ age: 70, created: d'2024-01-01T00:00:00Z', id: person:gladys }]"

*/

DEFINE FIELD created ON person TYPE datetime DEFAULT d'2024-01-01T00:00:00Z' READONLY;
CREATE person:gladys SET age = 90;
-- Attempts to change `created` field, error
UPDATE person:gladys REPLACE { age: 70 };
-- `created` equals current value, query works
UPDATE person:gladys REPLACE { age: 70, created: d'2024-01-01T00:00:00Z' };
```

```surql title="Output"
-------- Query --------
[
	{
		age: 90,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]

-------- Query --------
'Found changed value for field `created`, with record `person:gladys`, but field is readonly'

-------- Query --------
[
	{
		age: 70,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]
```

## MERGE clause

Instead of specifying the full record data using `CONTENT` or one field at a time using `SET`, it is also possible to merge-update only specific fields by using the `MERGE` keyword followed by on object containing the fields which are to be upserted.

```surql
-- Update certain fields on all records
UPDATE person MERGE {
	settings: {
		marketing: true,
	},
};

-- Update certain fields on a specific record
UPDATE person:tobie MERGE {
	settings: {
		marketing: true,
	},
};
```

```surql title="Output"
[
	{
		company: 'SurrealDB',
		id: person:i5z3i64cpqpo8jtr6jww,
		name: 'John',
		settings: {
			marketing: true
		},
		skills: [
			'Rust',
			'Go',
			'JavaScript'
		]
	},
	{
		company: 'SurrealDB',
		id: person:tobie,
		name: 'Tobie',
		settings: {
			marketing: true
		},
		skills: [
			'Rust',
			'Go',
			'JavaScript'
		]
	}
]
```

## PATCH clause

You can also specify changes to be applied to your query response, using the PATCH command which works similar to the [JSON Patch specification](https://jsonpatch.com/)

```surql
-- Patch the JSON response
UPDATE person:tobie PATCH [
	{
		"op": "add",
		"path": "Engineering",
		"value": "true"
	}
]
```

```surql title="Output"
[
	{
		Engineering: 'true',
		company: 'SurrealDB',
		id: person:tobie,
		name: 'Tobie',
		settings: {
			marketing: true
		},
		skills: [
			'Rust',
			'Go',
			'JavaScript'
		]
	}
]
```

## Alter the `RETURN` value

By default, the update statement returns the record value once the changes have been made. To change the return value of each record, use the `RETURN` clause, specifying `NONE`, `BEFORE`, `AFTER`, `DIFF`, a comma-separated list of specific fields or ad-hoc fields, or `VALUE` for a single field without its key name.

```surql
-- Don't return any result
UPDATE person SET skills += 'reading' RETURN NONE;

-- Return the changeset diff
UPDATE person SET skills += 'reading' RETURN DIFF;

-- Return the record before changes were applied
UPDATE person SET skills += 'reading' RETURN BEFORE;

-- Return the record after changes were applied (the default)
UPDATE person SET skills += 'reading' RETURN AFTER;

-- Return the value of the 'skills' field without the field name
UPDATE person SET skills += 'reading' RETURN VALUE skills;

-- Return a specific field only from the updated records
UPDATE person:tobie SET skills = ['skiing', 'music'] RETURN name, interests;
```

## Using a timeout

When processing a large result set with many interconnected records, it is possible to use the `TIMEOUT` keyword to specify a timeout duration for the statement. If the statement continues beyond this duration, then the transaction will fail, no records will be updated in the database, and the statement will return an error.

```surql
UPDATE person 
	SET important = true 
	WHERE ->knows->person->(knows WHERE influencer = true) 
	TIMEOUT 5s;
```

## UPDATE inside database exports

As `UPDATE` before version 2.0.0 used to create a specified record ID if it did not exist, it was used in the `.surql` files generated by the [`surreal export`](/docs/surrealdb/cli/export) command to export existing records in a database. As of version 2.0.0, the [`INSERT`](/docs/surrealql/statements/insert) statement is used instead.

## The `EXPLAIN` clause

When `EXPLAIN` is used:

1. The `UPDATE` statement returns an explanation, essentially revealing the execution plan to provide transparency and understanding of the query performance.
2. The records are not updated.

`EXPLAIN` can be followed by `FULL` to see the number of executed rows.

## upsert.mdx

---
sidebar_position: 26
sidebar_label: UPSERT
title: UPSERT statement | SurrealQL
description: The UPSERT statement can be used to insert records or modify records that already exist
---
import Since from '@components/shared/Since.astro'
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";
import RailroadDiagram from "@components/RailroadDiagram.astro";

# `UPSERT` statement

<Since v="v2.0.0" />

The `UPSERT` statement can be used to insert records into the database, or to update them if they exist.

> [!NOTE]
> In versions of SurrealDB between 2.0.0 and 2.0.4, an UPSERT statement was treated as an "UPDATE, otherwise INSERT" statement. It has since been changed to a statement which defaults to insertion, and updates otherwise. Please see the examples below for details.

### Statement syntax

<Tabs syncKey="upsert-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
UPSERT [ ONLY ] @targets
    [ CONTENT @value
      | MERGE @value
      | PATCH @value
	  | REPLACE @value
      | [ SET @field = @value, ... | UNSET @field, ... ]
    ]
    [ WHERE @condition ]
    [ RETURN NONE | RETURN BEFORE | RETURN AFTER | RETURN DIFF | RETURN @statement_param, ... | RETURN VALUE @statement_param ]
    [ TIMEOUT @duration ]
	[ EXPLAIN [ FULL ]]
;
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const upsertAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "UPSERT" },
      { type: "Optional", child: { type: "Terminal", text: "ONLY" } },
      { type: "NonTerminal", text: "@targets" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [
        { type: "Sequence", children: [ { type: "Terminal", text: "CONTENT" }, { type: "NonTerminal", text: "@value" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "MERGE" }, { type: "NonTerminal", text: "@value" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "PATCH" }, { type: "NonTerminal", text: "@value" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "REPLACE" }, { type: "NonTerminal", text: "@value" } ] },
        { type: "Sequence", children: [ { type: "Choice", index: 1, children: [ { type: "Terminal", text: "SET" }, { type: "Terminal", text: "UNSET" } ] }, { type: "NonTerminal", text: "@field (=|,) @value ..." } ] }
      ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "WHERE" }, { type: "NonTerminal", text: "@condition" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "RETURN" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NONE" }, { type: "Terminal", text: "BEFORE" }, { type: "Terminal", text: "AFTER" }, { type: "Terminal", text: "DIFF" }, { type: "NonTerminal", text: "@statement_param, ..." }, { type: "Sequence", children: [ { type: "Terminal", text: "VALUE" }, { type: "NonTerminal", text: "@statement_param" } ] } ] } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TIMEOUT" }, { type: "NonTerminal", text: "@duration" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "EXPLAIN" }, { type: "Optional", child: { type: "Terminal", text: "FULL" } } ] } },
      { type: "Terminal", text: ";" }
    ]}
  ]
};

<RailroadDiagram ast={upsertAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

Conceptually, an `UPSERT` statement can be thought of as an "`INSERT`, otherwise `UPDATE`" statement.

### `UPSERT` without a `WHERE` clause

As an `UPSERT` statement is primarily an `INSERT` statement, one without a `WHERE` clause will not perform an update.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:5014eu5j3ysw6ifsmk8v, name: 'Billy' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: person:89m30prs5ph4azxluwww, name: 'Bobby' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: person:5014eu5j3ysw6ifsmk8v, name: 'Billy' }, { id: person:89m30prs5ph4azxluwww, name: 'Bobby' }]"
skip-record-id-key = true

*/

UPSERT person SET name = 'Billy';
UPSERT person SET name = 'Bobby';
SELECT * FROM person;
```

As the output shows, the second `UPSERT` simply inserted another `person` record with the name "Bobby", rather than updating the existing record.

```surql title="Output"
-------- Query --------
[
	{
		id: person:c2bl54ahi551fcx9dqri,
		name: 'Billy'
	}
]

-------- Query --------
[
	{
		id: person:886dcoe1ayul217nl2fu,
		name: 'Bobby'
	}
]

-------- Query --------
[
	{
		id: person:886dcoe1ayul217nl2fu,
		name: 'Bobby'
	},
	{
		id: person:c2bl54ahi551fcx9dqri,
		name: 'Billy'
	}
]
```

### Using the `WHERE` clause

#### Without a specified ID

When using the `WHERE` clause and no specified ID, SurrealDB will check to see if any records match the clause. If nothing matches, a new record will be created.

As such, the following `UPSERT` statement will return a new record:

```surql
/**[test]

[[test.results]]
value = "[{ id: person:39v37mew631umqb63bfq, name: 'Jaime' }]"
skip-record-id-key = true

*/

UPSERT person SET name = 'Jaime' WHERE name = 'Jaime';
```

```surql title="Output"
[
	{
		id: person:7ilunylkcjgbg9gf0tqn,
		name: 'Jaime'
	}
]
```

Since a record with the name 'Jaime' exists, an `UPSERT` followed by `WHERE name = 'Jaime'` will update the existing record instead of creating a new one.

```surql
UPSERT person SET name = 'Tobie' WHERE name = 'Jaime';
```

```surql title="Output"
[
	{
		id: person:7ilunylkcjgbg9gf0tqn,
		name: 'Tobie'
	}
]
```

Now that no records have the name `'Jaime'`, the same query as above will now create a new record because no records match the `WHERE` clause. The database will now have two `person` records.

```surql
UPSERT person SET name = 'Tobie' WHERE name = 'Jaime';
SELECT * FROM person;
```

```surql title="Output"
-- Query
[
	{
		id: person:0n0ddlkmhe6mdb6ikkui,
		name: 'Tobie'
	}
]

-- Query
[
	{
		id: person:0n0ddlkmhe6mdb6ikkui,
		name: 'Tobie'
	},
	{
		id: person:7ilunylkcjgbg9gf0tqn,
		name: 'Tobie'
	}
]
```

#### With a specified ID

`UPSERT` behaviour with a specific ID and a `WHERE` clause differs slightly from the examples above. In this case, there is the possibility that a record ID already exists but the `WHERE` clause does not match. As such, there is no way to create a new record as the statement only pertains to an ID for an already existing record.

The following query will return a record, because the `person:test` record does not yet exist. The `WHERE` clause makes no difference as there is no record to apply it to.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:test, name: 'Jaime' }]"

*/

UPSERT person:test SET name = 'Jaime' WHERE name = 'Jaime';
```

```surql title="Output"
[
	{
		id: person:test,
		name: 'Jaime'
	}
]
```

The following query will update the `person:test` record, because the record exists and the `WHERE` clause matches. The `person:test` record will now have the name `'Tobie'`.

```surql
UPSERT person:test SET name = 'Tobie' WHERE name = 'Jaime';
```

```surql title="Output"
[
	{
		id: person:test,
		name: 'Tobie'
	}
]
```

However, this third query will return nothing. The `WHERE` clause does not match and thus `person:test` cannot be updated, and the statement itself only pertains to the `person:test` record, so a new record using a random ID will not be returned.

```surql
UPSERT person:test SET name = 'Billy' WHERE name = 'Jaime';
```

```surql title="Output"
[]
```

### Improved performance via UPSERT and a unique index

[Unique indexes](/docs/surrealql/statements/define/indexes#unique-index) can be used to ensure that no field or combination of fields is ever present more than once. For example, a game might have a rule that duplicate names can exist, but not within the same class.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ class: 'wizard', id: user:billy, metadata: { likes: ['strawberries'] }, name: 'Billy', official_name: 'Billy the wizard' }]"

[[test.results]]
value = ""Database index `unique_key` already contains ['Billy', 'wizard'], with record `user:billy`""

*/

DEFINE INDEX unique_key ON TABLE user FIELDS name, class UNIQUE;
DEFINE FIELD official_name ON TABLE user VALUE name + " the " + class;
CREATE user:billy SET name = "Billy", class = "wizard", metadata = { likes: ["strawberries"] };
CREATE user:billy_der_zweite SET name = "Billy", class = "wizard", metadata = { likes: ["strawberries", "fields"] };
```

As the output shows, the unique index prevents the creation of a second user with the name and class as the first.

```surql title="Output"
-------- Query --------
[
	{
		id: user:billy,
		metadata: {
			likes: [
				'strawberries'
			]
		},
		name: 'Billy',
		official_name: 'Billy the wizard',
		class: 'wizard'
	}
]

-------- Query --------
"Database index `unique_key` already contains ['Billy', 'wizard'], with record `user:billy`"
```

To change an existing record's `metadata` field to the value `{ likes: ["strawberries", "fields"] }`, an `UPDATE` with a `WHERE` can be used. This performs a scan on the `user` table to check for all records that match the `WHERE` clause.

```surql
UPDATE user SET
	metadata = { likes: ["strawberries", "fields"] }
WHERE
	name = "Billy" AND
	class = "wizard";
```

However, a much more efficient method is available if you only need to update one record and have a unique index that can be used instead. This optimization is available when using `UPSERT` and a unique index, because the statement will always access the index in any case to first see if the record is a duplicate.

```surql
-- Checks the index for ["Mandy", "wizard"], no existing
-- record found so no problem
UPSERT user SET name = "Mandy", class = "wizard";

-- Fails because statement tries to upsert a new user:mandy
-- on top of the previous randomly generated one
UPSERT user:mandy SET 
	name = "Mandy",
	class = "wizard",
	metadata = { likes: ["strawberries" ]};
```

```surql title="Output"
-------- Query --------

[
	{
		class: 'wizard',
		id: user:kdvh401gofckvvy6nbiw,
		name: 'Mandy',
		official_name: 'Mandy the wizard'
	}
]

-------- Query --------

"Database index `unique_key` already contains ['Mandy', 'wizard'], with record `user:kdvh401gofckvvy6nbiw`"
```

Since an `UPSERT` statement already checks unique indexes by default, it uses this to recognize that the user with `name = "Billy"` and `the = "Wizard"` corresponds to the record `user:j2ecdb2tf4ou29mr0yp5` and update it without needing to scan the entire `user` table.

```surql
UPSERT user SET
	name = "Billy",
	class = "wizard",
	metadata = { likes: ["strawberries", "fields"] };
```

```surql title="Output"
-------- Query --------
[
	{
		id: user:j2ecdb2tf4ou29mr0yp5,
		metadata: {
			likes: [
				'strawberries'
			]
		},
		name: 'Billy',
		official_name: 'Billy the wizard',
		class: 'wizard'
	}
]
```

To compare the performance difference between using a `WHERE` clause and a unique index yourself, here is an example that creates a crowded field of 50000 `user` records, followed by one more `user` named "Billy". An `UPDATE` using `WHERE name = "Billy" AND class = "wizard"` requires a full table scan, while an `UPSERT` using the two fields used to build the index is much faster.

```surql
DEFINE INDEX unique_key ON TABLE user FIELDS name, class UNIQUE;
DEFINE FIELD official_name ON TABLE user VALUE name + " the " + class;

-- Add 50000 users to fill up the database
FOR $i IN <array>0..50000 { CREATE user SET name = <string>$i, class = <string>$i; };

-- Create Billy, one of 50,001 records
CREATE user SET name = "Billy", class = "wizard";

-- Updating Billy requires a table scan
UPDATE user SET
	interests += "music"
WHERE
	name = "Billy" AND
	class = "wizard";

-- But UPSERT uses 'name' and 'class' to check the index anyway,
-- and thus can use it to access the record without a scan
UPSERT user SET
	name = "Billy",
	class = "wizard",
	interests += "travel";
```

## Using the ONLY clause

The `ONLY` clause can be used to return a single record instead of an array of records.

```surql
/**[test]

[[test.results]]
value = "{ company: 'SurrealDB', id: person:tobie, name: 'Tobie', skills: ['Rust', 'Go', 'JavaScript'] }"

*/

-- UPSERT just a single record
-- Using the ONLY keyword, just an object for the record in question will be returned.
-- This, instead of an array with a single object.
UPSERT ONLY person:tobie SET 
	name = 'Tobie', 
	company = 'SurrealDB', 
	skills = ['Rust', 'Go', 'JavaScript'];
```

## Type inference when using UPSERT

The `+=` operator in the following query is enough for SurrealDB to infer that the `interests` field must be an `array<string>`.

```surql
/**[test]

[[test.results]]
value = "[{ company: 'SurrealDB', id: person:tobie, interests: ['Java'], name: 'Tobie', skills: ['Rust', 'Go', 'JavaScript'] }]"

*/

-- UPSERT a document and remove a tag from an array
UPSERT person:tobie SET interests += 'Java';
```

Type inference will also work with a numeric value such as the `click_count` field below, in which case it will infer the field to be of type `int` with a default value of 0.

```surql
/**[test]

[[test.results]]
value = "[{ click_count: 1, id: webpage:home }]"

*/

-- UPSERT a document and increment a numeric value
UPSERT webpage:home SET click_count += 1;
```

Creating a record by default makes the `UPSERT` statement an ideal way to manage an incrementing field.

```surql
UPSERT event_for:[time::now().format("%Y-%m-%d")] SET
    number += 1;
```

```surql title="Possible output"
[
	{
		id: event_for:[
			'2024-09-18'
		],
		number: 1
	}
]
```

Doing the same with an `UPDATE` statement would require much more manual work.

```surql
IF (SELECT * FROM event_for:[time::now().format("%Y-%m-%d")]).is_empty() {
    CREATE event_for:[time::now().format("%Y-%m-%d")] SET number = 1;
} ELSE {
    UPDATE event_for:[time::now().format("%Y-%m-%d")] SET number += 1;
};
```

## CONTENT clause

Instead of specifying record data using the `SET` clause, it is also possible to use the `CONTENT` keyword to specify the record data using a SurrealQL object.

```surql
/**[test]

[[test.results]]
value = "[{ company: 'SurrealDB', id: person:6vfpq64z5r6bpe8cb91q, name: 'Tobie', skills: ['Rust', 'Go', 'JavaScript'] }]"

[[test.results]]
value = "[{ company: 'SurrealDB', id: person:tobie, name: 'Tobie', skills: ['Rust', 'Go', 'JavaScript'] }]"

*/

-- UPSERT all records with the same content
UPSERT person CONTENT {
	name: 'Tobie',
	company: 'SurrealDB',
	skills: ['Rust', 'Go', 'JavaScript'],
};

-- UPSERT a specific record with some content
UPSERT person:tobie CONTENT {
	name: 'Tobie',
	company: 'SurrealDB',
	skills: ['Rust', 'Go', 'JavaScript'],
};
```

Since version `2.1.0`, a statement with a `CONTENT` clause will bypass `READONLY` fields instead of generating an error.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ age: 90, created: d'2024-01-01T00:00:00Z', id: person:gladys }]"

[[test.results]]
value = "[{ age: 70, created: d'2024-01-01T00:00:00Z', id: person:gladys }]"

*/

DEFINE FIELD created ON person TYPE datetime DEFAULT d'2024-01-01T00:00:00Z' READONLY;
UPSERT person:gladys SET age = 90;
-- Does not try to modify `created` field, no error
UPSERT person:gladys CONTENT { age: 70 };
```

<Tabs groupId="content">

<TabItem value="Before" label="Output before 2.1.0" >

```surql
-------- Query --------
[
	{
		age: 90,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]

-------- Query --------
'Found changed value for field `created`, with record `person:gladys`, but field is readonly'
```
</TabItem>

<TabItem value="After" label="Output after 2.1.0" >

```surql
-------- Query --------
[
	{
		age: 90,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]

-------- Query --------
[
	{
		age: 70,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]
```
</TabItem>
</Tabs>

## REPLACE clause

Originally an alias for `CONTENT`, the `REPLACE` clause maintains the previous behaviour regarding `READONLY` fields. If the content following `REPLACE` does not match a record's `READONLY` fields, an error will be generated.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ age: 90, created: d'2024-01-01T00:00:00Z', id: person:gladys }]"

[[test.results]]
error = "'Found changed value for field `created`, with record `person:gladys`, but field is readonly'"

[[test.results]]
value = "[{ age: 70, created: d'2024-01-01T00:00:00Z', id: person:gladys }]"

*/

DEFINE FIELD created ON person TYPE datetime DEFAULT d'2024-01-01T00:00:00Z' READONLY;
UPSERT person:gladys SET age = 90;
-- Attempts to change `created` field, error
UPSERT person:gladys REPLACE { age: 70 };
-- `created` equals current value, query works
UPSERT person:gladys REPLACE { age: 70, created: d'2024-01-01T00:00:00Z' };
```

```surql title="Output"
-------- Query --------
[
	{
		age: 90,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]

-------- Query --------
'Found changed value for field `created`, with record `person:gladys`, but field is readonly'

-------- Query --------
[
	{
		age: 70,
		created: d'2024-01-01T00:00:00Z',
		id: person:gladys
	}
]
```

## MERGE clause 

Instead of specifying the full record data using the `SET` clause or the `CONTENT` keyword, it is also possible to merge-UPSERT only specific fields by using the `MERGE` keyword and specifying only the fields which are to be upserted.

```surql
/**[test]

[[test.results]]
value = "[{ id: person:8v3or7b2uqfnscvptzkd, settings: { marketing: true } }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: person:tobie, settings: { marketing: true } }]"

*/

-- Inserts a new record with a single field and random ID
UPSERT person MERGE {
	settings: {
		marketing: true,
	},
};

-- Updates certain fields on a specific record
UPSERT person:tobie MERGE {
	settings: {
		marketing: true,
	},
};
```

## PATCH clause

You can also specify changes to be applied to your query response, using the `PATCH` clause which works similar to the [JSON Patch specification](https://jsonpatch.com/)

```surql
/**[test]

[[test.results]]
value = "[{ Engineering: 'true', id: person:tobie }]"

*/

-- Patch the JSON response
UPSERT person:tobie PATCH [
	{
		"op": "add",
		"path": "Engineering",
		"value": "true"
	}
];
```

## Alter the `RETURN` value

By default, the UPSERT statement returns the record value once the changes have been made. To change the return value of each record, specify a `RETURN` clause, specifying either `NONE`, `BEFORE`, `AFTER`, `DIFF`, or a comma-separated list of specific fields to return.

```surql
/**[test]

[[test.results]]
value = "[]"

[[test.results]]
value = "[[{ op: 'add', path: '/interests/1', value: 'reading' }]]"

[[test.results]]
value = "[{ id: person:tobie, interests: ['reading', 'reading'] }]"

[[test.results]]
value = "[{ id: person:tobie, interests: ['reading', 'reading', 'reading', 'reading'] }]"

[[test.results]]
value = "[{ interests: ['skiing', 'music'], name: NONE }]"

*/

-- Don't return any result
UPSERT person:tobie SET interests += 'reading' RETURN NONE;

-- Return the changeset diff
UPSERT person:tobie SET interests += 'reading' RETURN DIFF;

-- Return the record before changes were applied
UPSERT person:tobie SET interests += 'reading' RETURN BEFORE;

-- Return the record after changes were applied (the default)
UPSERT person:tobie SET interests += 'reading' RETURN AFTER;

-- Return a specific field only from the upserted records
UPSERT person:tobie SET interests = ['skiing', 'music'] RETURN name, interests;
```

When processing a large result set with many interconnected records, it is possible to use the `TIMEOUT` keywords to specify a timeout duration for the statement. If the statement continues beyond this duration, then the transaction will fail, no records will be upserted in the database, and the statement will return an error.

```surql
/**[test]

[[test.results]]
value = ""

*/

UPSERT person:3 SET important = true WHERE ->knows->person->(knows WHERE influencer = true) TIMEOUT 5s;
```

## The `EXPLAIN` clause

When `EXPLAIN` is used:

1. The `UPSERT` statement returns an explanation, essentially revealing the execution plan to provide transparency and understanding of the query performance.
2. The records are not updated.

`EXPLAIN` can be followed by `FULL` to see the number of executed rows.

## use.mdx

---
sidebar_position: 27
sidebar_label: USE
title: USE statement | SurrealQL
description: The USE statement specifies a namespace and / or a database to use for the subsequent SurrealQL statements when switching between namespaces and databases.
---

import Since from "@components/shared/Since.astro";
import RailroadDiagram from "@components/RailroadDiagram.astro";
import Tabs from "@components/Tabs/Tabs.astro";
import TabItem from "@components/Tabs/TabItem.astro";

# `USE` statement

The `USE` statement specifies a namespace and / or a database to use for the subsequent SurrealQL statements when switching between namespaces and databases. If you have a single namespace and database, you can define them in the [sql command](/docs/surrealdb/cli/sql#example-usage).

Ensure that your database and namespace exist and you have [started your database](/docs/surrealdb/cli/start) before using the Sql command option.

### Statement syntax

<Tabs syncKey="use-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
USE [ NS @ns ] [ DB @db ];
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const useAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "USE" },
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "NS" },
              { type: "NonTerminal", text: "@ns" },
            ],
          },
        },
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "DB" },
              { type: "NonTerminal", text: "@db" },
            ],
          },
        },
        { type: "Terminal", text: ";" },
      ],
    },
  ],
};

<RailroadDiagram ast={useAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following query shows example usage of this statement if you have multiple namespaces and databases.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

USE NS test; -- Switch to the 'test' Namespace
```

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

USE DB test; -- Switch to the 'test' Database
```

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

USE NS test DB test; -- Switch to the 'test' Namespace and 'test' Database
```

You can also use the [INFO Statement](/docs/surrealql/statements/info) to check the current namespace and database.

```surql
INFO FOR NS; -- Check the current Namespace
```

```surql
INFO FOR DB; -- Check the current Database
```

## `USE` statement behaviour when resource does not exist

<Since v="v3.0.0" />

The behaviour of the `USE` statement differs depending on which mode the database server is run in.

When run in regular mode, a `USE` statement will create the namespace or database indicated if it does not already exist.

```surql
USE NS ns; -- Output: NONE (success)
(INFO FOR ROOT).namespaces; -- Output: { ns: 'DEFINE NAMESPACE ns' }
```

In [strict mode](/docs/surrealdb/cli/start#strict-mode), a resource will not be created unless it is already defined. In this case, the `USE` statement will return an error.

```surql
USE NS ns; -- Output: "The namespace 'ns' does not exist"
DEFINE NS ns;
USE NS ns; -- Now defined, no error
```

## Value returned by `USE` statement

<Since v="v3.0.0" />

Before SurrealDB 3.0.0-beta, the output of a `USE` statement was `NONE`. Since then, each `USE` statement returns an object containing the current namespace and database.

```surql
USE NS main;
```

```surql title="Output"
{ database: 'main', namespace: 'main' }
```

## analyzer.mdx

---
sidebar_position: 2
sidebar_label: DEFINE ANALYZER
title: DEFINE ANALYZER statement | SurrealQL
description: In the context of a database, an analyzer plays a crucial role in text processing and searching. It is defined by its name, a set of tokenizers, and a collection of filters.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `DEFINE ANALYZER` statement

> [!NOTE]
> Before SurrealDB version 3.0.0-beta, the `FULLTEXT ANALYZER` clause used the syntax `SEARCH ANALYZER`.

In the context of a database, an analyzer plays a crucial role in text processing and searching. It is defined by its name, a set of tokenizers, and a collection of filters.

The output of an analyzer can be experimented with by using the [`search::analyze()`](/docs/surrealql/functions/database/search#searchhighlight) function.

## Requirements
- You must be authenticated as a root, namespace, or database user before you can use the `DEFINE ANALYZER` statement.
- [You must select your namespace and database](/docs/surrealql/statements/use) before you can use the `DEFINE ANALYZER` statement.

## Statement syntax

<Tabs syncKey="define-analyzer-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE ANALYZER [ OVERWRITE | IF NOT EXISTS ] @name [ FUNCTION @function ] [ TOKENIZERS @tokenizers ] [ FILTERS @filters ] [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineAnalyzerAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "ANALYZER" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "FUNCTION" }, { type: "NonTerminal", text: "@function" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TOKENIZERS" }, { type: "NonTerminal", text: "@tokenizers" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "FILTERS" }, { type: "NonTerminal", text: "@filters" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineAnalyzerAst} className="my-6" />

  </TabItem>
</Tabs>

## The `FUNCTION` clause

Using the `FUNCTION` clause allows a [user-defined function](/docs/surrealql/statements/define/function) to be executed on the initial input. The function must take and return a `string`.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "['BDlaerruS', 'ekil', 'I']"

*/

DEFINE FUNCTION fn::backwardsify($input: string) -> string {
    $input.split('').fold('', |$a, $b| $b + $a);
};

DEFINE ANALYZER backwards FUNCTION fn::backwardsify TOKENIZERS blank;

search::analyze("backwards", "I like SurrealDB");
```

```surql title="Output"
[
	'BDlaerruS',
	'ekil',
	'I'
]
```

## Tokenizers

Tokenizers are responsible for breaking down a given text into individual tokens based on a set of instructions. There are a couple of tokenizers that can be used while defining an analyzer as seen below:

### `blank`

The blank tokenizer breaks down a text into tokens by creating a new token each time it encounters a space, tab, or newline character. It's a straightforward way to split text into words or chunks based on whitespace.

```surql
DEFINE ANALYZER example_blank TOKENIZERS blank;
search::analyze("example_blank", "hello world");
```

```surql title="Output"
[
	'hello',
	'world'
]
```

### `camel`

The camel tokenizer is used for identifying and creating tokens when the next character in the text is uppercase. This is particularly useful for processing camelCase or PascalCase text, common in programming, to split them into meaningful words.

```surql
DEFINE ANALYZER example_camel TOKENIZERS camel;
search::analyze("example_camel", "helloWorld");
```

```surql title="Output"
[
	'hello',
	'World'
]
```

### `class`

The class tokenizer segments text into tokens by detecting changes (digit, letter, punctuation, blank) in the Unicode class of characters. It creates a new token when the character class changes, distinguishing between digits, letters, punctuation, and blanks. This allows for flexible tokenization based on character types.

```surql
DEFINE ANALYZER example_class TOKENIZERS class;
search::analyze("example_class", "123abc!XYZ");
```

```surql title="Output"
[
	'123',
	'abc',
	'!',
	'XYZ'
]
```

### `punct`

The punct tokenizer generates tokens by breaking the text whenever a punctuation character is encountered. It's suitable for tokenizing sentences or breaking text into smaller units based on punctuation marks.

```surql
DEFINE ANALYZER example_punct TOKENIZERS punct;
search::analyze("example_punct", "Hello, World!");
```

```surql title="Output"
[
	'Hello',
	',',
	'World',
	'!'
]
```

## Filters

Filters take on the task of transforming these tokens for further processing and analysis.

### `ascii`

The ascii filter is responsible for processing tokens by replacing or removing diacritical marks (accents and special characters) from the text. It helps standardize text by converting accented characters to their basic ASCII equivalents, making it more suitable for various text analysis tasks.

```surql
DEFINE ANALYZER example_ascii TOKENIZERS class FILTERS ascii;
search::analyze("example_ascii", "résumé café");
```

```surql title="Output"
[
	'resume',
	'cafe'
]
```

### `lowercase`

The lowercase filter converts tokens to lowercase, ensuring that text is consistently in lowercase format. This is often used to make text case-insensitive for search and analysis purposes.

```surql
DEFINE ANALYZER example_lowercase TOKENIZERS class FILTERS lowercase;
search::analyze("example_lowercase", "Hello World");
```

```surql title="Output"
[
	'hello',
	'world'
]
```

### `uppercase`

The uppercase filter converts tokens to uppercase, ensuring text consistency in uppercase format. It can be useful when case-insensitivity is required for specific analysis or search operations.

For example, if you had the text **"Hello World"**, the uppercase filter would create two tokens, **["HELLO", "WORLD"]**. Below is an example of how to use the uppercase filter:

```surql
DEFINE ANALYZER example_uppercase TOKENIZERS class FILTERS uppercase;
search::analyze("example_uppercase", "Hello World");
```

```surql title="Output"
[
	'HELLO',
	'WORLD'
]
```

### `edgengram(min,max)`

The edgengram filter is used to create tokens that represent prefixes of terms. It generates a sequence of tokens that gradually build up a term, which can be useful for autocomplete or searching based on partial words. It accepts two parameters `min` and `max` which define the minimum and maximum amount of characters in the prefix.

For example, if you had the text **"apple banana"**, the edgengram filter would create six tokens, **["a", "ap", "app", "b", "ba", "ban"]**. Below is an example of how to use the edgengram filter:

```surql
DEFINE ANALYZER example_edgengram TOKENIZERS class FILTERS edgengram(1,3);
search::analyze("example_edgengram", "apple banana");
```

```
[
	'a',
	'ap',
	'app',
	'b',
	'ba',
	'ban'
]
```

{/* Add Since here once next version out */}

### `mapper(path)`

The mapping filter is designed to enable lemmatization within SurrealDB.

Lemmatization is the process of reducing words to their base or dictionary form. The mapper mechanism allows users to specify a custom dictionary file that maps terms to their base forms. This dictionary file is then used by SurrealDB’s analyzer to standardize terms as they are indexed, improving search consistency.

This is particularly useful for handling irregular verbs and other terms that the default "snowball" filter cannot handle. Lemmatization files are easy to put together and to find online, making it possible to customize full-text search for smaller languages.

How does the mapper work?

Configuration: In the SQL statement below, the mapper parameter is specified within the analyzer definition.
This parameter points to the file that contains the term mappings for lemmatization.

```surql
DEFINE ANALYZER lemme_english TOKENIZERS blank,class FILTERS lowercase,mapper('../tests/data/lemmatization-en.txt');

RETURN [
    search::analyze("lemme_english", "He drove and swam"),
];
```

```surql title="Output"
[
	[
		'he',
		'drive',
		'and',
		'swim'
	]
]
```

Dictionary File Structure: The file specified in the mapper parameter must follow this format:

- Each line contains a pair of terms separated by a tab.
- The first term represents the canonical (base form) of the word.
- The second term is the form to be mapped to this base form.

Example file format:

```
drive	driven
drive	drives
drive	driving
drive	drove
swim	swam
swim	swimming
swim	swims
swim	swum
```

Usage: When this analyzer is applied to a text, any word that matches the mapped term in the dictionary file will be replaced by its base form before indexing. This helps ensure consistency in search results by consolidating different forms of a word to a single, standardized entry.

By using this custom dictionary-based mapper, you can control how irregular forms and other variations of terms are indexed,
making search behavior more predictable and comprehensive.

The following example shows how lemmatization can be used to generate a list of words and their respective frequencies. Other notable functionalities in the example are the [`string::is_alpha()`](/docs/surrealql/functions/database/string#stringis_alpha) function inside [`array::filter()`](/docs/surrealql/functions/database/array#arrayfilter) to remove all non-alphabetic strings, the [`type::record()`](/docs/surrealql/functions/database/type#typerecord) function to construct a record ID from two strings, and an [`UPSERT`](/docs/surrealql/statements/upsert) statement to create a record if one does not exist, or update it otherwise.

```surql
DEFINE ANALYZER lemme_english TOKENIZERS blank,class FILTERS lowercase,mapper('../tests/data/lemmatization-en.txt');

LET $text = "The Wheel of Time turns, and Ages come and pass, leaving memories that become legend. Legend fades to myth, and even myth is long forgotten when the Age that gave it birth comes again. In one Age, called the Third Age by some, an Age yet to come, an Age long past, a wind rose in the Mountains of Mist. The wind was not the beginning. There are neither beginnings nor endings to the turning of the Wheel of Time. But it was a beginning.";

LET $words = search::analyze("lemme_english", $text)
    .filter(|$c| $c.is_alpha());
FOR $word IN $words {
    UPSERT type::record("word", $word) SET frequency += 1;
};

SELECT * FROM word WHERE frequency >=3 ORDER BY frequency DESC;
```

```surql title="Output"
[
	{
		frequency: 8,
		id: word:the
	},
	{
		frequency: 6,
		id: word:age
	},
	{
		frequency: 4,
		id: word:a
	},
	{
		frequency: 4,
		id: word:be
	},
	{
		frequency: 4,
		id: word:of
	},
	{
		frequency: 3,
		id: word:and
	},
	{
		frequency: 3,
		id: word:come
	},
	{
		frequency: 3,
		id: word:to
	}
]
```

A mapper can also be used for ad-hoc filtering, as long as the file referenced contains two single words separated by a tab. Take the following file for example:

```title="error_filter.txt"
NOT_FOUND	File_not_found
NOT_FOUND	Datei_nicht_gefunden
NOT_FOUND	Fichier_non_trouvé
TIMEOUT	Timed_out
TIMEOUT	Délai_expiré
TIMEOUT	Zeitüberschreitung
```

An analyzer that uses a single mapper filter can then use this lemmatizer to unify multilingual error messages into a single output.

```surql
DEFINE ANALYZER error_filter FILTERS mapper('error_filter.txt');

LET $messages = 
	["File not found", "Datei nicht gefunden", "Zeitüberschreitung"]
	.map(|$word| $word.replace(' ', '_'))
	.join(' ');
search::analyze("error_filter", $messages);
```

```surql title="Output"
[
	'NOT_FOUND',
	'NOT_FOUND',
	'TIMEOUT'
]
```

Example using the same mapper to search for errors in multiple languages:

```surql
DEFINE ANALYZER error_filter FILTERS mapper('error_filter.txt');
DEFINE INDEX OVERWRITE errors ON TABLE error FIELDS message FULLTEXT ANALYZER error_filter;

FOR $message IN ["File not found", "Datei nicht gefunden", "Zeitüberschreitung"] {
	CREATE error SET message = $message.replace(' ', '_'), at = time::now();
};

SELECT * FROM error WHERE message @@ "NOT_FOUND";
```

```surql title="Output"
[
	{
		at: d'2024-11-13T03:56:12.039252Z',
		id: error:acbc044syhnx54wzs3n9,
		message: 'File_not_found'
	},
	{
		at: d'2024-11-13T03:56:12.043643Z',
		id: error:5ifxic9s750x24ts4zof,
		message: 'Datei_nicht_gefunden'
	}
]
```

### `ngram(min,max)`

The ngram filter is used to create a sequence of 'n' tokens from a given sample of text or speech. These items can be syllables, letters, words or base pairs according to the application. It accepts two parameters `min` and `max` which indicates that you want to create n-grams starting from min to size of max.

```surql
DEFINE ANALYZER example_ngram TOKENIZERS class FILTERS ngram(1,3);
search::analyze("example_ngram", "apple banana");
```

```surql title="Output"
[
	'a',
	'ap',
	'app',
	'p',
	'pp',
	'ppl',
	'p',
	'pl',
	'ple',
	'l',
	'le',
	'e',
	'b',
	'ba',
	'ban',
	'a',
	'an',
	'ana',
	'n',
	'na',
	'nan',
	'a',
	'an',
	'ana',
	'n',
	'na',
	'a'
]
```

### `snowball(language)`

The snowball filter applies Snowball stemming to tokens, reducing them to their root form and converts the case to lowercase. The following supported languages can be passed as a parameter in snowball: Arabic, Danish, Dutch, English, French, German, Greek, Hungarian, Italian, Norwegian, Portuguese, Romanian, Russian, Spanish, Swedish, Tamil, Turkish.

```surql
DEFINE ANALYZER english_snowball TOKENIZERS class FILTERS snowball(english);
DEFINE ANALYZER german_snowball TOKENIZERS class FILTERS snowball(german);

RETURN [
    search::analyze("english_snowball", "Looking at some running cats"),
    search::analyze("german_snowball", "Sollen wir was trinken gehen?")
];
```

```surql title="Output"
[
	[
		'look',
		'at',
		'some',
		'run',
		'cat'
	],
	[
		'soll',
		'wir',
		'was',
		'trink',
		'geh',
		'?'
	]
]
```

## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define an analyzer only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining an analyzer in SurrealDB if you want to ensure that the analyzer is only created if it does not already exist. If the analyzer already exists, the `DEFINE ANALYZER` statement will return an error.

It's particularly useful when you want to safely attempt to define a analyzer without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the analyzer definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a analyzer and overwrite an existing one if it already exists, ensuring that the latest version of the analyzer definition is always in use.

```surql
-- Create an ANALYZER if it does not already exist
DEFINE ANALYZER IF NOT EXISTS example TOKENIZERS blank;
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to create an analyzer and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing analyzer definition. If the analyzer already exists, the `DEFINE ANALYZER` statement will overwrite the existing analyzer definition with the new one.

```surql
-- Create an ANALYZER and overwrite if it already exists
DEFINE ANALYZER OVERWRITE example TOKENIZERS blank;
```

## More examples

Examples on application of analyzers to indexes can be found in the documenation on [`DEFINE INDEX`](/docs/surrealql/statements/define/indexes) statement

This example creates an analyzer that tokenizes text based on the class of characters and then applies the lowercase filter to the tokens.

```surql
-- Creates a simple analyzer removing diacritics marks
DEFINE ANALYZER ascii TOKENIZERS class FILTERS lowercase,ascii;
```

This example creates an analyzer specifically designed for processing English texts.

```surql
-- Creates an analyzer suitable for English text
DEFINE ANALYZER english TOKENIZERS class FILTERS snowball(english);
```

This example creates an analyzer specifically designed for auto-completion tasks.

```surql
-- Creates an analyzer suitable for auto-completion.
DEFINE ANALYZER autocomplete FILTERS lowercase,edgengram(2,10);
```

This example creates an analyzer specifically designed for source code analysis.

```surql
-- Creates an analyzer suitable for source code analysis.
DEFINE ANALYZER code TOKENIZERS class,camel FILTERS lowercase,ascii;
```

## api.mdx

---
sidebar_position: 3
sidebar_label: DEFINE API
title: DEFINE API statement | SurrealQL
description: A DEFINE API statement can be used to set endpoints with custom middleware and permissions.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'


# DEFINE API statement

<Since v="v3.0.0" />

The `DEFINE API` statements allows a custom endpoint to be created. Each endpoint created by a `DEFINE API` statement is located at the `/api/:namespace/:database/:endpoint_name` path. For example, an endpoint for the path `get_users` for the namespace `my_namespace` and database `my_database` will have the path `/api/my_namespace/my_database/get_users`.

The response is an object with a combination of the following properties:
* `status` - A valid HTTP status code.
* `body` - Any value.
* `headers` - An object of valid header key value pairs. The value of each pair must be a string.
* `context` - An object. 

A defined API has access to a preset parameter called [`$request`](/docs/surrealql/parameters#request) which contains the request sent by the caller of the endpoint. The $request parameter may contain values at the following fields: `body`, `headers`, `params`, `method`, `query`, and `context`.

## Statement syntax

<Tabs syncKey="define-api-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE API [ OVERWRITE | IF NOT EXISTS ] @endpoint
    [ FOR @HTTP_method, .. ]
    [ MIDDLEWARE @function, .. ]
    [ THEN { @value } ]
    [ PERMISSIONS [ NONE | FULL | @expression ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineApiAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "API" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@endpoint" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "NonTerminal", text: "@HTTP_method, .." } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "MIDDLEWARE" }, { type: "NonTerminal", text: "@function, .." } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "THEN" }, { type: "Terminal", text: "{" }, { type: "NonTerminal", text: "@value" }, { type: "Terminal", text: "}" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "PERMISSIONS" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NONE" }, { type: "Terminal", text: "FULL" }, { type: "NonTerminal", text: "@expression" } ] } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineApiAst} className="my-6" />

  </TabItem>
</Tabs>


`DEFINE API` is often used in conjunction with a [capabilities flag](/docs/surrealdb/security/capabilities) or [environment variable](/docs/surrealdb/cli/env) to disable arbitrary queries, thereby forcing record and anonymous users to interact with the database via API endpoints alone.

## Quick example

```surql title="Defining an API endpoint"
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE API "/test"
    FOR get, post 
        MIDDLEWARE
            api::timeout(1s)
        THEN {
            {
                status: 200,
                body: {
                    request: $request.body,
                    response: "The server works"
                },
                headers: {
                    'last-modified': <string>time::now(),
                    'expires': <string>(time::now() + 4d)
                }
            };
        };
```

An API endpoint can be tested using the [`api::invoke` function](/docs/surrealql/functions/database/api), which takes either the path as a single string or the path along with a request body. It can also be tested via [CURL or other means by directly using the endpoint](/docs/surrealdb/integration/http) along with the namespace and database in the headers.

```surql
api::invoke("/test");

api::invoke("/test", {
    body: {
       hi: "please",
        give: "me",
        the: "information"
    }
});
```

```surql title="Output"
-------- Query --------

{
	body: {
		request: NONE,
		response: 'The server works'
	},
    context: {},
	headers: {
		"access-control-allow-origin": '*',
		expires: '2026-01-24T02:43:50.137321Z',
		"last-modified": '2025-02-20T02:43:50.137326Z'
	},
	status: 200
}

-------- Query --------

{
	body: {
		request: {
			give: 'me',
			hi: 'please',
			the: 'information'
		},
		response: 'The server works'
	},
    context: {},
	headers: {
		"access-control-allow-origin": '*',
		expires: '2025-02-24T02:43:50.137455Z',
		"last-modified": '2026-01-20T02:43:50.137457Z'
	},
	status: 200
}
```

## API paths

The path of a `DEFINE API` statement can be static, such as `"/test"`, dynamic, or the remainder of a URL.

A dynamic path uses a `:` (colon) followed by a name, which will match on anything passed in at that section of a path.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ body: { some: 'data' }, context: {  }, headers: {  }, status: 200 }"

[[test.results]]
value = "{ body: { some: 'data' }, context: {  }, headers: {  }, status: 200 }"

[[test.results]]
value = "{ body: NONE, context: {  }, headers: {  }, status: 404 }"

*/

DEFINE API OVERWRITE "/test/:anything_goes" FOR get THEN {
    RETURN {
        body: {
            some: "data"
        }
    }
};

api::invoke("/test/this_matches");
api::invoke("/test/same_here");
api::invoke("/test/but/this/wont/match");
```

The first two `api::invoke` calls return the output below, but the third returns nothing as `:anything_goes` only applies to a single path segment.

```surql title="Output"
{ 
    body: NONE, 
    context: {}, 
    headers: {}, 
    status: 404 
}
```

To match on the remainder of a URL, change the `:` (colon) to a `*` (star).

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ body: { some: 'data' }, context: {  }, headers: {  }, status: 200 }"

[[test.results]]
value = "{ body: { some: 'data' }, context: {  }, headers: {  }, status: 200 }"

[[test.results]]
value = "{ body: { some: 'data' }, context: {  }, headers: {  }, status: 200 }"

*/

DEFINE API OVERWRITE "/test/*anything_goes" FOR get THEN {
    RETURN {
        body: {
            some: "data"
        }
    }
};

api::invoke("/test/this_matches");
api::invoke("/test/same_here");
api::invoke("/test/works/with/multiple/paths/now");
```

All three `api::invoke` calls will now show the following output.

```surql title="Output"
{
	body: {
		some: 'data'
	},
	context: {},
	headers: {},
	status: 200
};
```

## Custom middleware

Custom middleware can be used in addition to the functions listed above.

A custom middleware function is defined in the same way as any other [user-defined function](/docs/surrealql/statements/define/function).

Each such function automatically receives two arguments:

* An object that will contain the user request.
* A function (a closure) that can be called to get the current state of the response to be returned.

The parameter names `$req` for the first and `$next` for the second are commonly used, though the actual parameter names are of no consequence.

The output of such a function must be an `object`. This is used to pass the response on to the next middleware function, or on to the actual response if there is no middleware left to call.

```surql
DEFINE FUNCTION fn::middleware_function($req: object, $next: function) -> object {};
```

These functions can take additional arguments on top of the required two.

```surql
DEFINE FUNCTION fn::middleware_function($req: object, $next: function, $some_string: string) -> object {};
```

### Custom middleware examples

Here is an example of an API endpoint without any middleware.

```surql
DEFINE API "/custom_response"
    FOR get
        THEN {
            {
                status: 200,
                body: {
                    num: 1
                }
            };
        };
```

Calling `api::invoke("/custom_response")` will return the following.

```surql
{
	body: {
		num: 1
	},
	context: {},
	headers: {},
	status: 200
};
```

We will now add a middleware function that does the following:

* Calls the `$next` closure on the `$req` object. This will return the value after `THEN`: the original return value.
* Returns an object in which the `num` field inside `body` is increased by one.

```surql
DEFINE FUNCTION fn::increment_num($req: object, $next: function) -> object {
    LET $res = $next($req);
    $res + { body: { num: $res.body.num + 1 } }
};

DEFINE API "/custom_response"
    FOR get
        MIDDLEWARE
            fn::increment_num()
        THEN {
            {
                status: 200,
                body: {
                    num: 1
                }
            };
        };
```

Calling `api::invoke("/custom_response")` will now return a modified output in which `num` is equal to 2.

```surql
{
	body: {
		num: 2
	},
	context: {},
	headers: {},
	status: 200
};
```

Here is an example of the same endpoint with an extra custom middleware function which adds a `called_at` field to the `context` of the response to show when the API endpoint was called. 

```surql
DEFINE FUNCTION fn::start_timer($req: object, $next: function, $called_at: datetime) -> object {
    LET $res = $next($req);
    $res + { context: { called_at: $called_at }}
};

DEFINE FUNCTION fn::increment_num($req: object, $next: function) -> object {
    LET $res = $next($req);
    $res + { body: { num: $res.body.num + 1 } }
};

DEFINE API "/custom_response"
    FOR get
        MIDDLEWARE
            fn::start_timer(time::now()),
            fn::increment_num()
        THEN {
            {
                status: 200,
                body: {
                    num: 1
                }
            };
        };

api::invoke("/custom_response");
```

The output will look something like this.

```surql
{
	body: {
		num: 2
	},
	context: {
		api_called_at: d'2026-01-16T01:49:44.115351Z'
	},
	headers: {},
	status: 200
};
```

## bucket.mdx

---
sidebar_position: 4
sidebar_label: DEFINE BUCKET
title: DEFINE BUCKET statement | SurrealQL
description: A DEFINE BUCKET statement can be used to set endpoints with custom middleware and permissions.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'


# DEFINE BUCKET statement

<Since v="v3.0.0" />

> [!NOTE]
> The `DEFINE BUCKET` statement is currently experimental and subject to change. To use this feature, please ensure you are on the latest supported alpha version of SurrealDB. To enable it, either pass `--allow-experimental files` when [starting the database](/docs/surrealdb/cli/start) or set the `SURREAL_CAPS_ALLOW_EXPERIMENTAL` environment variable to `files`.

The `DEFINE BUCKET` statement lets you create a bucket that can hold files.

## Statement syntax

<Tabs syncKey="define-bucket-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE BUCKET [ OVERWRITE | IF NOT EXISTS ] @name [ @backend ]
[ PERMISSIONS ] @expression [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineBucketAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "BUCKET" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Optional", child: { type: "NonTerminal", text: "@backend" } },
      { type: "Optional", child: { type: "Terminal", text: "PERMISSIONS" } },
      { type: "NonTerminal", text: "@expression" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineBucketAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

A bucket backend can be set as "memory" for non-persistent in-memory storage, or as "file:/", followed by the path, for storage on disk.

### Memory backend

The simplest way to experiment with a bucket for files is by using the memory backend:

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE BUCKET my_bucket BACKEND "memory";
```

Once this is defined, `my_bucket` can be accessed by using a file pointer: a path prefixed by an `f`.

```surql
-- Create a file by adding some content
f"my_bucket:/my_book.txt".put("Once there were four children whose names were Peter, Susan, Edmund, and Lucy.");
-- Copy it to a new file name
f"my_bucket:/my_book.txt".copy("lion_witch_wardrobe.txt");
-- Read the file as bytes
f"my_bucket:/lion_witch_wardrobe.txt".get();
-- Cast the bytes to a string
<string>f"my_bucket:/lion_witch_wardrobe.txt".get();
```

```surql title="Output"
-------- Query --------

b"4F6E6365207468657265207765726520666F7572206368696C6472656E2077686F7365206E616D657320776572652050657465722C20537573616E2C2045646D756E642C20616E64204C7563792E"

-------- Query --------

'Once there were four children whose names were Peter, Susan, Edmund, and Lucy.'
```

### File backend

A file backend can be chosen for a bucket by typing `"file:"` and then the rest of the path, if necessary.

```surql
DEFINE BUCKET my_bucket BACKEND "file:/some_directory";
DEFINE BUCKET my_bucket BACKEND "file:/some_directory";
```

A check will then be made to see if the `SURREAL_BUCKET_FOLDER_ALLOWLIST` environment variable contains the path, without which the following error will be generated.

```surql
'File access denied: /some_directory'
```

The following command can be used to start running an instance in which a bucket with a file backend can be defined.

```bash
# Unix
SURREAL_BUCKET_FOLDER_ALLOWLIST="/" surreal start --user root --pass secret --allow-experimental files

# Windows (PowerShell)
$env:SURREAL_BUCKET_FOLDER_ALLOWLIST = "/" 
surreal start --user root --pass secret --allow-experimental files
```

### Global backend

A global backend can also be selected, allowing all namespaces and databases access to the same file storage.

If no backend is selected, the database will search for the environment variable `SURREAL_GLOBAL_BUCKET` and assign this as the global bucket. In this case, files will have a `namespace/database` prefix added (e.g. `my_global_bucket:/test_ns/test_db/somefile.txt`). A second `SURREAL_GLOBAL_BUCKET_ENFORCED` environment variable can also be used, which when set to `true` will enforce usage of the global bucket.

If a global backend is set, then a `DEFINE BUCKET` statement can be as short as `DEFINE BUCKET` plus its local name, as the rest of the logic is done via environment variables.

```surql
DEFINE BUCKET my_bucket;

-- Writes to e.g. `my_global_bucket:/test_ns/test_db/my_bucket/my_book.txt`
f"my_bucket:/my_book.txt".put("Once there were four children whose names were Peter, Susan, Edmund, and Lucy.");
```

## Setting permissions on buckets

By default, the permissions on a bucket will be set to FULL unless otherwise specified.

```surql
DEFINE BUCKET my_bucket BACKEND "memory";
INFO FOR DB;
```

```surql title="Response"
{
  accesses: {},
  analyzers: {},
  apis: {},
  buckets: {
    my_bucket: "DEFINE BUCKET my_bucket BACKEND 'memory' PERMISSIONS FULL"
  },
  configs: {},
  functions: {},
  models: {},
  modules: {},
  params: {},
  sequences: {},
  tables: {},
  users: {}
}
```

You can set permissions on buckets to control who can perform operations on the files stored in them using the `PERMISSIONS` clause. In the clause three additional variables are available:
- `$action`: The action to be executed (`put`, `get`, `head`, `delete`, `copy`, `rename`, `exists`, `list`)
- `$file`: The [file pointer](/docs/surrealql/datamodel/files) of the file to be accessed
- `$target`: The target [file pointer](/docs/surrealql/datamodel/files) in copy/rename operations

```surql
-- Set permissions for the bucket
DEFINE BUCKET admin_bucket BACKEND "memory"
  PERMISSIONS WHERE $auth.admin = true
```

## config.mdx

---
sidebar_position: 5
sidebar_label: DEFINE CONFIG
title: DEFINE CONFIG statement | SurrealQL
description: This statement allows you to set external configurations on the database, either for API middleware and permissions, or for how the database's tables and functions are exposed via the GraphQL API.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'


# `DEFINE CONFIG` statement

The `DEFINE CONFIG` statement allows you to set external configurations on your database. It can be used to configure API middleware and permissions, or to configure how the database's tables and functions are exposed via the GraphQL API.

## Requirements

- You **must** be authenticated as a **root**, **namespace**, or **database** user before you can use the `DEFINE CONFIG GRAPHQL` statement.
- You **must** select your **namespace** and **database** before you can use the `DEFINE CONFIG GRAPHQL` statement.
- You **must** define at least one table in your database for the GraphQL API to function.
- You **must** have started the SurrealDB instance with GraphQL enabled.

## Statement syntax

<Tabs syncKey="define-config-statement">
  <TabItem label="SurrealQL Syntax">

```surql title="SurrealQL Syntax"

DEFINE CONFIG [ OVERWRITE | IF NOT EXISTS ]
  ( API [ MIDDLEWARE @expression, .. ] [ PERMISSIONS [ NONE | FULL | @expression ] ]
  | GRAPHQL 
      [ AUTO | NONE ]
      [ TABLES (AUTO | NONE | INCLUDE table1, table2, ...) ]
      [ FUNCTIONS (AUTO | NONE | INCLUDE [function1, function2, ...] | EXCLUDE [function1, function2, ...]) ]
  )

```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineConfigAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "CONFIG" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "Choice", index: 1, children: [
        { type: "Sequence", children: [
          { type: "Terminal", text: "API" },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "MIDDLEWARE" }, { type: "NonTerminal", text: "@expression, .." } ] } },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "PERMISSIONS" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NONE" }, { type: "Terminal", text: "FULL" }, { type: "NonTerminal", text: "@expression" } ] } ] } }
        ] },
        { type: "Sequence", children: [
          { type: "Terminal", text: "GRAPHQL" },
          { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "AUTO" }, { type: "Terminal", text: "NONE" } ] } },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TABLES" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "AUTO" }, { type: "Terminal", text: "NONE" }, { type: "Sequence", children: [ { type: "Terminal", text: "INCLUDE" }, { type: "NonTerminal", text: "table1, table2, ..." } ] } ] } ] } },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "FUNCTIONS" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "AUTO" }, { type: "Terminal", text: "NONE" }, { type: "Sequence", children: [ { type: "Terminal", text: "INCLUDE" }, { type: "NonTerminal", text: "[function1, function2, ...]" } ] }, { type: "Sequence", children: [ { type: "Terminal", text: "EXCLUDE" }, { type: "NonTerminal", text: "[function1, function2, ...]" } ] } ] } ] } }
        ] }
      ] }
    ]}
  ]
};

<RailroadDiagram ast={defineConfigAst} className="my-6" />

  </TabItem>
</Tabs>

## DEFINE CONFIG API

The `DEFINE CONFIG API` statement can be used to set middleware and permissions in order to alter the behaviour of the database for guest and [record users](/docs/surrealql/statements/define/access/record). This middleware can be used in cases such as rate limiting on the query language and setting how many resources a client can read or alter at a time.

`DEFINE CONFIG API` is often used in conjunction with a [flag](/docs/surrealdb/security/capabilities) or [environment variable](/docs/surrealdb/cli/env) to disable arbitrary queries, thereby forcing record and anonymous users to interact with the database via API endpoints alone.

The following is an example of a `DEFINE CONFIG` statement that includes a timeout and a single header in the responses of all API endpoints.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE CONFIG API
    MIDDLEWARE 
        api::timeout(10s),
        api::res::headers({
            'Access-Control-Allow-Origin': '*'
        });
```

To set the actual API endpoints and their middleware, a `DEFINE API` statement is used for each endpoint. For more details and examples, see the pages for [`DEFINE API`](/docs/surrealql/statements/define/api) and [API functions](/docs/surrealql/functions/database/api).

The behaviour of API endpoints can be tested using the `api::invoke` method, or through a regular HTTP call to the endpoint that [includes the namespace and database name](/docs/surrealdb/integration/http#custom-endpoint-at-apinsdbendpoint).

This next example uses `DEFINE CONFIG` to set a single response header, followed by two endpoints that return different output.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ headers: { "access-control-allow-origin": '*' }, raw: false, status: 200 }"

[[test.results]]
value = "{ body: { some: 'data' }, headers: { "access-control-allow-origin": '*' }, raw: false, status: 200 }"

*/

DEFINE CONFIG API
    MIDDLEWARE 
        api::res::headers({
            'Access-Control-Allow-Origin': '*'
        });

DEFINE API "/test" FOR get THEN {};
DEFINE API "/test2" FOR get THEN {
    {
        body: {
            some: "data"
        }
    }
};

api::invoke("/test");
api::invoke("/test2");
```

The query shows that the endpoints return a combination of the `DEFINE CONFIG` middleware and the response object set in the `DEFINE API` statements.

```surql
-------- Query --------

{
  body: NONE,
  context: {},
	headers: {
		"access-control-allow-origin": '*'
	},
	status: 200
}

-------- Query --------

{
	body: {
		some: 'data'
	},
  context: {},
	headers: {
		"access-control-allow-origin": '*'
	},
	status: 200
}
```

Note that the middleware and permissions inside individual `DEFINE API` statements will override the middleware in a `DEFINE CONFIG API` statement. In the following example, the default `10s` timeout is set to a single microsecond for the `"/test"` endpoint, giving the intended query no time to complete.

```surql
DEFINE CONFIG API
    MIDDLEWARE 
        api::timeout(10s),
        api::res::headers({
            'Access-Control-Allow-Origin': '*'
        });

DEFINE API OVERWRITE "/test"
    FOR get 
        MIDDLEWARE
            api::timeout(1µs)
        THEN {
            RETURN {
                status: 200,
                body: { 
                    data: (SELECT * FROM person),
                    however: "This will probably never return because the timeout is 1 microsecond"
                }
            };
        };

api::invoke("/test");
```

```surql title="Output"
'The query was not executed because it exceeded the timeout: 1µs'
```

## DEFINE CONFIG GRAPHQL

The configuration set using the `DEFINE CONFIG GRAPHQL` is essential for enabling GraphQL functionality in your database, specifying which tables and functions should be included or excluded from the GraphQL schema.

The GraphQL configuration defined using this statement dictates how clients interact with your database through GraphQL queries and mutations.

### Important Notes

- The `DEFINE CONFIG GRAPHQL` statement **must** be executed before any GraphQL queries can be made.
- If you attempt to use the GraphQL API without defining the configuration, you will receive a `NotConfigured` error.
- If no tables are defined in the database, you will receive an error stating "No tables found in database" when attempting to use the GraphQL API.

### Example usage

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Define GraphQL configuration
DEFINE CONFIG GRAPHQL  AUTO;
```

### Tables Configuration

The `TABLES` clause in the `DEFINE CONFIG GRAPHQL` statement specifies how tables are exposed via GraphQL. There are four options for the `TABLES` configuration:

- `AUTO`: Automatically include all tables in the GraphQL schema.
- `NONE`: Do not include any tables in the GraphQL schema.
- `INCLUDE`: Specify a list of tables to include in the GraphQL schema.
- `EXCLUDE`: Specify a list of tables to exclude from the GraphQL schema.

#### `AUTO`

When you specify `TABLES AUTO`, all tables in the database are automatically included in the GraphQL schema.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE CONFIG GRAPHQL TABLES AUTO;
```

#### `NONE`

When you specify `TABLES NONE`, no tables are included in the GraphQL schema.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE CONFIG GRAPHQL TABLES NONE;
```

#### `INCLUDE`

You can specify a list of tables to include in the GraphQL schema using the `INCLUDE` clause. The list of tables is specified as a comma-separated list without brackets.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE CONFIG GRAPHQL TABLES INCLUDE user, post, comment;
```

> [!NOTE]
> The `EXCLUDE` option for `TABLES` is currently not implemented.

### Functions Configuration

The `FUNCTIONS` clause in the `DEFINE CONFIG GRAPHQL` statement specifies how functions are exposed via GraphQL. There are four options for the `FUNCTIONS` configuration:

- `AUTO`: Automatically include all functions in the GraphQL schema.
- `NONE`: Do not include any functions in the GraphQL schema.
- `INCLUDE`: Specify a list of functions to include in the GraphQL schema.
- `EXCLUDE`: Specify a list of functions to exclude from the GraphQL schema.

#### `AUTO`

When you specify `FUNCTIONS AUTO`, all functions in the database are automatically included in the GraphQL schema.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE CONFIG GRAPHQL FUNCTIONS AUTO;
```

#### `NONE`

When you specify `FUNCTIONS NONE`, no functions are included in the GraphQL schema.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE CONFIG GRAPHQL FUNCTIONS NONE;
```

#### `INCLUDE`

You can specify a list of functions to include in the GraphQL schema using the `INCLUDE` clause. The list of functions is specified as a comma-separated list enclosed in square brackets `[]`.

```surql
DEFINE CONFIG GRAPHQL FUNCTIONS INCLUDE [getUser, listPosts, searchComments];
```

#### `EXCLUDE`

You can specify a list of functions to exclude from the GraphQL schema using the `EXCLUDE` clause. The list of functions is specified as a comma-separated list enclosed in square brackets `[]`.

```surql
DEFINE CONFIG GRAPHQL FUNCTIONS EXCLUDE [debugFunction, testFunction];
```

### Using `IF NOT EXISTS` clause

The `IF NOT EXISTS` clause can be used to define the GraphQL configuration only if it does not already exist. This is useful when you want to ensure that the configuration is only created if it does not already exist, preventing errors due to duplicate definitions.

```surql
-- Define GraphQL configuration only if it does not already exist
DEFINE CONFIG GRAPHQL IF NOT EXISTS TABLES AUTO FUNCTIONS AUTO;
```

### Using `OVERWRITE` clause

The `OVERWRITE` clause can be used to redefine the GraphQL configuration, overwriting any existing configuration. This is useful when you want to update or modify the existing GraphQL configuration.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Redefine GraphQL configuration, overwriting existing configuration
DEFINE CONFIG OVERWRITE GRAPHQL TABLES INCLUDE user, post FUNCTIONS NONE;
```

### Examples

#### Example 1: Include specific tables and functions

This example defines a GraphQL configuration that includes specific tables and functions.

```surql
DEFINE CONFIG GRAPHQL TABLES INCLUDE user, post FUNCTIONS INCLUDE [getUser, listPosts];
```

#### Example 2: Automatically include all tables and functions

This example defines a GraphQL configuration that automatically includes all tables and functions.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE CONFIG GRAPHQL TABLES AUTO FUNCTIONS AUTO;
```

#### Example 3: Exclude specific functions

This example defines a GraphQL configuration that includes all functions except specific ones.

```surql
DEFINE CONFIG GRAPHQL FUNCTIONS EXCLUDE [debugFunction, testFunction];
```


### Error Handling

#### NotConfigured Error

If you attempt to access the GraphQL endpoint without defining the GraphQL configuration, you will receive a `NotConfigured` error.

```surql title="Error Response"
{
  "error": "NotConfigured: GraphQL endpoint is not configured. Please define the GraphQL configuration using DEFINE CONFIG GRAPHQL."
}
```

Execute the `DEFINE CONFIG GRAPHQL` statement to define the GraphQL configuration.

```surql
-- Define GraphQL configuration
DEFINE CONFIG GRAPHQL TABLES AUTO FUNCTIONS AUTO;
```

#### No Tables Found Error

If you have defined the GraphQL configuration but no tables are defined in the database, you will receive an error stating "No tables found in database" when attempting to use the GraphQL API. You can fix this by defining at least one table in your database using the [`DEFINE TABLE`](/docs/surrealql/statements/define/table) statement.

```surql title="Error Response"
{
  "error": "No tables found in database. Please define at least one table to use the GraphQL API."
}
```

**Solution**: Define at least one table in your database.

```surql
DEFINE TABLE foo SCHEMAFUL;
DEFINE FIELD val ON foo TYPE int;
CREATE foo:1 SET val = 42;
```

### Authentication Errors When Accessing Data via GraphQL

**Cause**: Insufficient permissions or incorrect authentication credentials.

**Solution**: Ensure you are authenticated as a user with the necessary permissions and that the permissions on tables and fields are correctly configured.

#### Authentication and Permissions

- The GraphQL API respects SurrealDB's authentication and permission model.
- You must authenticate using the appropriate credentials to access data via GraphQL.
- Permissions set on tables and fields will affect the data accessible through the GraphQL API.
- If you attempt to access data without sufficient permissions, you will receive an authentication error.



#### Example: Basic Authentication

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ email: 'user@example.com', id: foo:1, val: 42 }]"

[[test.results]]
value = "[{ email: 'other@example.com', id: foo:2, val: 43 }]"

*/

-- Define a user with access permissions
DEFINE USER my_user ON DATABASE PASSWORD 'my_password';
DEFINE ACCESS user ON DATABASE TYPE RECORD
  SIGNUP ( CREATE user SET email = $email, pass = crypto::argon2::generate($pass) )
  SIGNIN ( SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass) )
  DURATION FOR SESSION 60s, FOR TOKEN 1d;

-- Define a table with permissions
DEFINE TABLE foo SCHEMAFUL PERMISSIONS FOR select WHERE $auth.email = email;
DEFINE FIELD email ON foo TYPE string;
DEFINE FIELD val ON foo TYPE int;

-- Insert data
CREATE foo:1 SET val = 42, email = "user@example.com";
CREATE foo:2 SET val = 43, email = "other@example.com";
```

When querying the GraphQL API as `user@example.com`, only the records where `email = "user@example.com"` will be accessible due to the permissions set.

### Examples

#### Example 1: Defining GraphQL Configuration and Fetching Data

```surql
-- Define GraphQL configuration to include all tables automatically
DEFINE CONFIG GRAPHQL TABLES AUTO;

-- Define a table and insert data
DEFINE TABLE foo SCHEMAFUL;
DEFINE FIELD val ON foo TYPE int;
CREATE foo:1 SET val = 42;
CREATE foo:2 SET val = 43;
```

Now, you can fetch data via GraphQL:

```graphql
query {
  foo {
    id
    val
  }
}
```

**Response:**

```json
{
  "data": {
    "foo": [
      {
        "id": "foo:1",
        "val": 42
      },
      {
        "id": "foo:2",
        "val": 43
      }
    ]
  }
}
```

#### Example 2: Including Specific Tables

```surql
-- Define GraphQL configuration to include only specific tables
DEFINE CONFIG OVERWRITE GRAPHQL TABLES INCLUDE foo;
```

When querying the GraphQL schema, only the included tables will be available.

#### Example 3: Using Limit, Start, Order, and Filter in GraphQL Queries

You can use `limit`, `start`, `order`, and `filter` in your GraphQL queries to control the data returned.

##### Limit

```graphql
query {
  foo(limit: 1) {
    id
    val
  }
}
```

##### Start

```graphql
query {
  foo(start: 1) {
    id
    val
  }
}
```

##### Order

```graphql
query {
  foo(order: { desc: val }) {
    id
    val
  }
}
```

##### Filter

```graphql
query {
  foo(filter: { val: { eq: 42 } }) {
    id
    val
  }
}
```

### More Information

The `DEFINE CONFIG GRAPHQL` statement is essential for enabling and configuring the GraphQL API in SurrealDB. By specifying which tables and functions are included or excluded, you can fine-tune the GraphQL schema to match your application's needs.

- **Authentication**: Ensure you are authenticated with the appropriate credentials.
- **Permissions**: Set up permissions on tables and fields to control access via GraphQL.
- **Configuration**: The GraphQL configuration must be defined before using the GraphQL API.
- **Error Handling**: Be aware of possible errors when the configuration is missing or incomplete.

> [!IMPORTANT]
> Always ensure you have the necessary permissions and have selected the appropriate namespace and database before using the `DEFINE CONFIG GRAPHQL` statement.


## Summary

- Use `DEFINE CONFIG GRAPHQL` to enable and configure the GraphQL API.
- Define your tables and set up permissions to control data access.
- Use the `OVERWRITE` and `IF NOT EXISTS` clauses as needed to manage your configuration.
- Always authenticate with appropriate credentials when accessing the GraphQL API.
- Utilize GraphQL query features like `limit`, `start`, `order`, and `filter` to control the data returned.

By following these guidelines, you can effectively use the `DEFINE CONFIG GRAPHQL` statement to configure and interact with your SurrealDB database via GraphQL.


## database.mdx

---
sidebar_position: 6
sidebar_label: DEFINE DATABASE
title: DEFINE DATABASE statement | SurrealQL
description: The DEFINE DATABASE statement allows you to instantiate a named database, enabling you to specify security and configuration options.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `DEFINE DATABASE` statement

The `DEFINE DATABASE` statement allows you to instantiate a named database, enabling you to specify security and configuration options.

## Requirements

- You must be authenticated as a root owner or editor, or namespace owner or editor before you can use the `DEFINE DATABASE` statement.
- [You must select your namespace](/docs/surrealql/statements/use) before you can use the `DEFINE DATABASE` statement.

## Statement syntax

<Tabs syncKey="define-database-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE DATABASE [ OVERWRITE | IF NOT EXISTS ] @name [ STRICT ] [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineDatabaseAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "DATABASE" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "STRICT" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineDatabaseAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage
Below shows how you can create a database using the DEFINE DATABASE statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Specify the namespace for the database
USE NS abcum;

-- Define database
DEFINE DATABASE app_vitalsense;
```

## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define a database only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining a database in SurrealDB if you want to ensure that the database is only created if it does not already exist. If the database already exists, the `DEFINE DATABASE` statement will return an error.

It's particularly useful when you want to safely attempt to define a database without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the database definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a database and overwrite an existing one if it already exists, ensuring that the latest version of the definition is always in use.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a database if it does not already exist
DEFINE DATABASE IF NOT EXISTS app_vitalsense;
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to define a database and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing database definition. If the database already exists, the `DEFINE DATABASE` statement will overwrite the existing definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a database and overwrite if it already exists
DEFINE DATABASE OVERWRITE app_vitalsense;
```

## Defining a `STRICT` database

<Since v="v3.0.0" />

A strict database is one that does not allow a resource to be used unless it has already been defined. The default behaviour in SurrealDB works otherwise, by allowing statements like [CREATE](/docs/surrealql/statements/create), [INSERT](/docs/surrealql/statements/insert) and [UPSERT](/docs/surrealql/statements/create) to work.

```surql
CREATE some_new_table;
INFO FOR DATABASE.tables;
```

The output of the [INFO](/docs/surrealql/statements/info) statement shows that a table called `some_new_table` has been created with a few default clauses.

```surql
{
	some_new_table: 'DEFINE TABLE some_new_table TYPE ANY SCHEMALESS PERMISSIONS NONE'
}
```

Such an operation within a strict database is simply not allowed.

```surql
DEFINE DATABASE new_db STRICT;
USE DATABASE new_db;
CREATE some_new_table;
```

```surql title="Output"
"The table 'some_new_table' does not exist"
```

## event.mdx

---
sidebar_position: 7
sidebar_label: DEFINE EVENT
title: DEFINE EVENT statement | SurrealQL
description: The DEFINE EVENT statement can be used to create events which can be triggered after any change or modification to the data in a record.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `DEFINE EVENT` statement

Events allow you to define custom logic that is executed when a record is created, updated, or deleted. These events are triggered automatically within the current transaction after data modifications in the record, giving you access to the state of the record [before `$before` and after `$after`](/docs/surrealql/parameters#before-after) the change.

> [!NOTE]
> Events are a side effect of other operations and thus are not triggered when data is [imported](/docs/surrealdb/cli/import).

### Key Concepts

- **Events**: Triggered after changes (create, update, delete) to records in a table.
* **$event**: A preset parameter containing the type of event as a string, will always be one of "CREATE", "UPDATE", or "DELETE".
- **$before / $after**: Refer to the record state before and after the modification. Learn more about the `$before` and `$after` parameters in the [parameters documentation](/docs/surrealql/parameters#before-after).
- **$value**: The record in question. For a `CREATE` or `UPDATE` event, this will be the record after the changes were made. For a `DELETE` statement, this will be the record before it was deleted.
- **WHEN condition**: Determines when the event should be triggered.

## Requirements

- You must be authenticated as a root owner or editor, namespace owner or editor, or database owner or editor before you can use the `DEFINE EVENT` statement.
- [You must select your namespace and database](/docs/surrealql/statements/use) before you can use the `DEFINE EVENT` statement.

## Statement syntax

<Tabs syncKey="define-event-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE EVENT [ IF NOT EXISTS | OVERWRITE ] @name ON [ TABLE ] @table
  [ ASYNC [ RETRY @retry ] [ MAXDEPTH @max_depth ] ]
  [ WHEN @condition ]
  [ THEN @action ]
  [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineEventAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "DEFINE" },
        { type: "Terminal", text: "EVENT" },

        // [ IF NOT EXISTS | OVERWRITE ]
        {
          type: "Optional",
          child: {
            type: "Choice",
            index: 1,
            children: [
              { type: "Terminal", text: "OVERWRITE" },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "IF" },
                  { type: "Terminal", text: "NOT" },
                  { type: "Terminal", text: "EXISTS" },
                ],
              },
            ],
          },
        },

        { type: "NonTerminal", text: "@name" },
        { type: "Terminal", text: "ON" },
        { type: "Optional", child: { type: "Terminal", text: "TABLE" } },
        { type: "NonTerminal", text: "@table" },

        // [ ASYNC [ RETRY @retry ] [ MAXDEPTH @max_depth ] ]
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "ASYNC" },

              {
                type: "Optional",
                child: {
                  type: "Sequence",
                  children: [
                    { type: "Terminal", text: "RETRY" },
                    { type: "NonTerminal", text: "@retry" },
                  ],
                },
              },

              {
                type: "Optional",
                child: {
                  type: "Sequence",
                  children: [
                    { type: "Terminal", text: "MAXDEPTH" },
                    { type: "NonTerminal", text: "@max_depth" },
                  ],
                },
              },
            ],
          },
        },

        // [ WHEN @condition ]
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "WHEN" },
              { type: "NonTerminal", text: "@condition" },
            ],
          },
        },

        // [ THEN @action ]
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "THEN" },
              { type: "NonTerminal", text: "@action" },
            ],
          },
        },

        // [ COMMENT @string ]
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "COMMENT" },
              { type: "NonTerminal", text: "@string" },
            ],
          },
        },
      ],
    },
  ],
};

<RailroadDiagram ast={defineEventAst} className="my-6" />

  </TabItem>
</Tabs>

### Clauses:

- **OVERWRITE**: Replaces the existing event if it already exists.
- **IF NOT EXISTS**: Only creates the event if it doesn't already exist.
- **WHEN**: Conditional logic that controls whether the event is triggered. Will show up in the event definition as `WHEN true` if not specified.
- **THEN**: Specifies the action(s) to execute when the event is triggered.
- **COMMENT**: Optional comment for describing the event.
- **ASYNC**: Whether to run the event outside of the transaction that triggers it. Available since SurrealDB 3.0.0-beta.

## Example usage

-  **Email Change Detection**: Create an event that logs whenever a user's email is updated.

In this example:
- The `WHEN` clause checks if the email has changed.
- The `THEN` clause records this change in a `log` table.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ email: 'old_email@test.com', id: user:test }]"

[[test.results]]
value = "[{ email: 'new_email@test.com', id: user:test }]"

[[test.results]]
value = "[]"

[[test.results]]
value = "[{ action: 'email created', at: d'2025-10-07T05:57:35.399269Z', id: log:ehhri1t7006hxcsg8bb5, new_email: 'old_email@test.com', old_email: '', user: user:test }, { action: 'email updated', at: d'2025-10-07T05:57:39.687764Z', id: log:rg89w4a05zcgictbdk8d, new_email: 'new_email@test.com', old_email: 'old_email@test.com', user: user:test }, { action: 'email deleted', at: d'2025-10-07T05:58:15.120426Z', id: log:11dpjy6rrk23jse8ee8b, new_email: '', old_email: 'new_email@test.com', user: user:test }]"
skip-record-id-key = true
skip-datetime = true

*/

-- Create a new event whenever a user changes their email address
-- One-statement event
DEFINE EVENT OVERWRITE test ON TABLE user WHEN $before.email != $after.email THEN (
    CREATE log SET 
        user       = $value.id,
        // Turn events like "CREATE" into string "email created"
        action     = 'email' + ' ' + $event.lowercase() + 'd',
        // `email` field may be NONE, log as '' if so
        old_email  = $before.email ?? '',
        new_email  = $after.email  ?? '',
        at         = time::now()
);
UPSERT user:test SET email = 'old_email@test.com';
UPSERT user:test SET email = 'new_email@test.com';
DELETE user:test;
SELECT * FROM log ORDER BY at ASC;
```

```surql title="Output"
[
	{
		action: 'email created',
		at: d'2024-11-25T02:59:41.003Z',
		id: log:e3thw1l0q7xiapznar1f,
		new_email: 'old_email@test.com',
		old_email: '',
		user: user:test
	},
	{
		action: 'email updated',
		at: d'2024-11-25T02:59:41.003Z',
		id: log:uaarfyk191jgod06xobm,
		new_email: 'new_email@test.com',
		old_email: 'old_email@test.com',
		user: user:test
	},
	{
		action: 'email deleted',
		at: d'2024-11-25T02:59:41.003Z',
		id: log:mlkag8h1xotglpz9wt2i,
		new_email: '',
		old_email: 'new_email@test.com',
		user: user:test
	}
]
```

### More complex logic:

-  **Purchase Event with Multiple Actions**: Log a purchase and establish relationships between the customer and product.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE EVENT purchase_made ON TABLE purchase
    WHEN $before == NONE
    THEN {
        LET $customer = (SELECT * FROM customer WHERE id = $after.customer);
        LET $product = (SELECT * FROM product WHERE id = $after.product);

        RELATE $customer->bought->$product CONTENT {
            quantity: $after.quantity,
            total: $after.total,
            status: 'Pending',
        };

        CREATE log SET
            customer_id = $after.customer,
            product_id = $after.product,
            action = 'purchase_created',
            timestamp = time::now();
    };
```

In this example:

- We perform multiple actions when a purchase is created: establishing relationships using the [RELATE](/docs/surrealql/statements/relate) statement and creating a log entry.

## Specific events

You can trigger events based on specific events. You can use the variable $event to detect what type of event is triggered on the table.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- CREATE event is triggered when a new record is inserted into the table.
-- Here we are updating the status of the post to PUBLISHED
-- when a new record is inserted into the publish_post table.
DEFINE EVENT publish_post ON TABLE publish_post
    WHEN $event = "CREATE"
    THEN (
        UPDATE post SET status = "PUBLISHED" WHERE id = $after.post_id
    );

-- UPDATE event
-- Here we are creating a notification when a user is updated.
DEFINE EVENT user_updated ON TABLE user
    WHEN $event = "UPDATE"
    THEN (
        CREATE notification SET message = "User updated", user_id = $after.id, created_at = time::now()
    );

-- DELETE event is triggered when a record is deleted from the table.
-- Here we are creating a notification when a user is deleted.
DEFINE EVENT user_deleted ON TABLE user
    WHEN $event = "DELETE"
    THEN (
        CREATE notification SET message = "User deleted", user_id = $before.id, created_at = time::now()
    );

-- You can combine multiple events based on your use cases.
-- Here we are creating a log when a user is created, updated or deleted.
DEFINE EVENT user_event ON TABLE user
    WHEN $event = "CREATE" OR $event = "UPDATE" OR $event = "DELETE"
    THEN (
        CREATE log SET
            table = "user",
            event = $event,
            happened_at = time::now()
    );
```

## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define an event only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining an event in SurrealDB if you want to ensure that the event is only created if it does not already exist. If the event already exists, the `DEFINE EVENT` statement will return an error.

It's particularly useful when you want to safely attempt to define a event without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the event definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a event and overwrite an existing one if it already exists, ensuring that the latest version of the event definition is always in use

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a EVENT if it does not already exist
DEFINE EVENT IF NOT EXISTS example ON example THEN {};
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to define an event and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing event definition. If the event already exists, the `DEFINE EVENT` statement will overwrite the existing event definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create an EVENT and overwrite if it already exists
DEFINE EVENT OVERWRITE example ON example THEN {};
```

## Events and permissions

Queries inside the event always execute without any permission checks, even when triggered by changes made by the currently authenticated user. This can be very useful to perform additional checks and changes that involve tables/records that are inaccessible for the user.

Consider a CREATE query sent by a record user that has CREATE access to the `comment` table only:

```surql
CREATE comment SET
    post = post:tomatosoup,
    content = "So delicious!",
    author = $auth.id
;
```

By having the following event defined, SurrealDB will perform the additional checks and changes:

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE EVENT on_comment_created ON TABLE comment
    WHEN $event = "CREATE"
    THEN {
        -- Check if the post allows for adding comments.
        -- User record doesn't have access to the `post` table.
        IF $after.post.disable_comments {
            THROW "Can't create a comment - Comments are disabled for this post";
        };

        -- Set the `approved` field on the new comment - automatically approve
        -- comments made by the author of the post.
        -- For security reasons, record users don't have any permissions for the `approved` field.
        UPDATE $after.id SET
            approved = $after.post.author == $after.author;
    };
```

## Accessing `$input` in events

<Since v="v3.0.0" />

The behaviour of events can be further refined via the `$input` parameter, which represents the record in question for the event.

```surql
-- Set CREATE in event to only trigger when record has `true` for `log_event`
DEFINE EVENT something ON person WHEN $input.log_event = true THEN {
    CREATE log SET at = time::now(), of = $input;
};

-- Set to `false`, does not trigger CREATE
CREATE person:debug SET name = "Billy", log_event = false;
-- Triggers CREATE
CREATE person:real SET name = "Bobby", log_event = true;

SELECT * FROM log;
```

Output:

```surql
[
	{
		at: d'2025-10-14T06:15:21.141Z',
		id: log:svbr2qhjywml20mufb0o,
		of: {
			log_event: true,
			name: 'Bobby'
		}
	}
]
```

## Async events

<Since v="v3.0.0" />

Events in SurrealDB are executed synchronously within the same transaction that triggers them. While this ensures consistency, it can lead to increased latency for write operations if the event logic is complex or resource intensive.

To allow events to execute independently of the transaction that triggers them, the `ASYNC` clause can be used.

### How async events are processed

Async events are processed in an interval dependant on the environment variable `SURREAL_ASYNC_EVENT_PROCESSING_INTERVAL` (or `--async-event-interval` when starting the server) which is set to 5 seconds as the default. Lowering this will reduce the latency between a document change and its events, while leading to more frequent polls by the background worker.

Some more notes on the characteristics of async events:

* Atomicity: The event is enqueued within the same transaction as the document change. If the transaction fails, the event is never queued.
* Consistency: Asynchronous events run in a separate transaction from the original change. They see the database state at the time they are executed.
* Ordering: Events are generally processed in the order they were created, though parallel processing may occur within a single batch.

The easiest way to demonstrate that async events do not occur in the same transaction is by causing one to [throw](/docs/surrealql/statements/throw) an error. As an error inside any part of a transaction will cause the transaction to fail and roll back, the following event which fails about 50% of the time would cause the `CREATE` statement that follows to fail if it were not async. As an async event, however, the events that follow the statement are each run in their own transaction

```surql
DEFINE TABLE did_not_throw;

DEFINE EVENT may_throw ON person ASYNC THEN {
  IF rand::bool() {
      THROW "This message will never show";
  } ELSE {
    CREATE did_not_throw;  
  }
};

CREATE |person:50|;
count(SELECT * FROM did_not_throw);
```

### The `MAXDEPTH` clause

The `MAXDEPTH` clause is used to set the maximum number of times that an async event can be triggered. The number following this can range from 0 to 16.

The default for `MAXDEPTH` is 3, as events defined on other events that lead to record creation can quickly spiral out of control at greater levels.

Taking the following contrived example:

```surql
DEFINE EVENT start ON start THEN {
    CREATE cat;
};

DEFINE EVENT cat ON person ASYNC MAXDEPTH 4 THEN {
  CREATE |cat:9|;  
};

DEFINE EVENT person ON cat ASYNC MAXDEPTH 4 THEN {
  CREATE |person:9|;
};

CREATE start;

count(SELECT VALUE id FROM person, cat);
```

While the `MAXDEPTH` in this case is only one greater than the default, the sheer number of records created results in the final `count()` score being 20503, compared to 2278 if the default is used.

This is somewhat similar to recursive queries which can also quickly add up.

```surql
-- Create five people
CREATE |person:1..=5|;
-- Make each person friends with each of the four others
UPDATE person SET friends = (SELECT VALUE id FROM person).complement([$this.id]);
-- Count after five levels of depth is already 1024!
count(person:1.{..5}.friends);
```

### The `RETRY` clause

The `RETRY` clause is suitable for events that may fail but can succeed on successive attempts. 

The example below shows two events that each have a 50% chance of failure and zero retries.

```surql
DEFINE EVENT one ON account ASYNC RETRY 0 THEN {
    IF rand::bool() {
        THROW "Failed!"
    } ELSE {
        CREATE it:worked SET very = "well";
    }
};

DEFINE EVENT two ON account ASYNC RETRY 0 THEN {
    IF rand::bool() {
        THROW "Failed!"
    } ELSE {
        CREATE it:worked SET very = "well";
    }
};

CREATE account;
```

Following up these events with a `SELECT * FROM it` will most likely lead to the following input.

```surql
[
	{
		id: it:worked,
		very: 'well'
	}
]
```

However, with zero retries there is still a 25% chance that the query will only ever lead to the error `"The table 'it' does not exist"`.

## field.mdx

---
sidebar_position: 8
sidebar_label: DEFINE FIELD
title: DEFINE FIELD statement | SurrealQL
description: The DEFINE FIELD statement allows you to instantiate a named field on a table, enabling you to set the field's achema and configuration.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `DEFINE FIELD` statement

The `DEFINE FIELD` statement allows you to instantiate a named field on a table, enabling you to set the field's data type, set a default value, apply assertions to protect data consistency, and set permissions specifying what operations can be performed on the field.

## Requirements

- You must be authenticated as a root owner or editor, namespace owner or editor, or database owner or editor before you can use the `DEFINE FIELD` statement.
- [You must select your namespace and database](/docs/surrealql/statements/use) before you can use the `DEFINE FIELD` statement.

## Statement syntax

<Tabs syncKey="define-field-statement">
  <TabItem label="Regular Field Syntax">

### Regular fields

```syntax title="SurrealQL Syntax"
DEFINE FIELD [ OVERWRITE | IF NOT EXISTS ] @name ON [ TABLE ] @table
	[ TYPE @type | object [ FLEXIBLE ] ]
	[ REFERENCE 
		[ ON DELETE REJECT | 
			ON DELETE CASCADE | 
			ON DELETE IGNORE |
			ON DELETE UNSET | 
			ON DELETE THEN @expression ]
	]
	[ DEFAULT [ALWAYS] @expression ]
  [ READONLY ]
	[ VALUE @expression ]
	[ ASSERT @expression ]
	[ PERMISSIONS [ NONE | FULL
		| FOR select @expression
		| FOR create @expression
		| FOR update @expression
	] ]
  [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Computed Field Syntax">

### Computed fields

<Since v="v3.0.0" />

> [!NOTE]
> In versions of SurrealDB before 3.0.0-beta, `COMPUTED` fields were implemented using a data type called a `future`. Please see [the page on futures](/docs/surrealql/datamodel/futures) in this case.

A `COMPUTED` field is one that is not stored but computed every time it is accessed. Such fields have a more limited set of clauses that can be used. Furthermore, a `COMPUTED` field cannot be defined on the `id` field of a record, nor any nested fields (i.e. a field `metadata` can be defined as computed, but not `medatata.can_drive`).

```syntax title="SurrealQL Syntax"
DEFINE FIELD [ OVERWRITE | IF NOT EXISTS ] @name ON [ TABLE ] @table
	COMPUTED @expression
	[ TYPE @type | object [ FLEXIBLE] ]
	[ PERMISSIONS [ NONE | FULL
		| FOR select @expression
		| FOR create @expression
		| FOR update @expression
	] ]
  [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Railroad Diagram (Regular)">

export const defineFieldRegularAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "FIELD" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Terminal", text: "ON" },
      { type: "Optional", child: { type: "Terminal", text: "TABLE" } },
      { type: "NonTerminal", text: "@table" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Optional", child: { type: "Terminal", text: "FLEXIBLE" } }, { type: "Terminal", text: "TYPE" }, { type: "NonTerminal", text: "@type" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "REFERENCE" }, { type: "Optional", child: { type: "Choice", index: 1, children: [
        { type: "Sequence", children: [ { type: "Terminal", text: "ON" }, { type: "Terminal", text: "DELETE" }, { type: "Terminal", text: "REJECT" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "ON" }, { type: "Terminal", text: "DELETE" }, { type: "Terminal", text: "CASCADE" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "ON" }, { type: "Terminal", text: "DELETE" }, { type: "Terminal", text: "IGNORE" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "ON" }, { type: "Terminal", text: "DELETE" }, { type: "Terminal", text: "UNSET" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "ON" }, { type: "Terminal", text: "DELETE" }, { type: "Terminal", text: "THEN" }, { type: "NonTerminal", text: "@expression" } ] }
      ] } } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "DEFAULT" }, { type: "Optional", child: { type: "Terminal", text: "ALWAYS" } }, { type: "NonTerminal", text: "@expression" } ] } },
      { type: "Optional", child: { type: "Terminal", text: "READONLY" } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "VALUE" }, { type: "NonTerminal", text: "@expression" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "ASSERT" }, { type: "NonTerminal", text: "@expression" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "PERMISSIONS" }, { type: "Choice", index: 1, children: [
        { type: "Terminal", text: "NONE" },
        { type: "Terminal", text: "FULL" },
        { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "select" }, { type: "NonTerminal", text: "@expression" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "create" }, { type: "NonTerminal", text: "@expression" } ] },
        { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "update" }, { type: "NonTerminal", text: "@expression" } ] }
      ] } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineFieldRegularAst} className="my-6" />

  </TabItem>
  <TabItem label="Railroad Diagram (Computed)">

export const defineFieldComputedAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "FIELD" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Terminal", text: "ON" },
      { type: "Optional", child: { type: "Terminal", text: "TABLE" } },
      { type: "NonTerminal", text: "@table" },
      { type: "Terminal", text: "COMPUTED" },
      { type: "NonTerminal", text: "@expression" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TYPE" }, { type: "NonTerminal", text: "@type" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "PERMISSIONS" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NONE" }, { type: "Terminal", text: "FULL" }, { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "select" }, { type: "NonTerminal", text: "@expression" } ] }, { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "create" }, { type: "NonTerminal", text: "@expression" } ] }, { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "update" }, { type: "NonTerminal", text: "@expression" } ] } ] } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineFieldComputedAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following expression shows the simplest way to use the `DEFINE FIELD` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Declare the name of a field.
DEFINE FIELD email ON TABLE user;
```

The fields of an object and the items in an array can be defined individually using the `.` operator for objects, or the indexing operator for arrays.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Define nested object property types
DEFINE FIELD emails.address ON TABLE user TYPE string;
DEFINE FIELD emails.primary ON TABLE user TYPE bool;

-- Define individual fields on an array
DEFINE FIELD metadata[0] ON person TYPE datetime;
DEFINE FIELD metadata[1] ON person TYPE int;
```

## Defining data types

The `DEFINE FIELD` statement allows you to set the data type of a field. For a full list of supported data types, see [Data types](/docs/surrealql/datamodel).

From version `v2.2.0`, when defining nested fields, where both the parent and the nested fields have types defined, it is no longer possible to have mismatching types, to prevent any impossible type issues once the schema is defined.

For example, the following will fail:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
error = "'Cannot set field `fd.*` with type `number` as it mismatched with field `fd` with type `{ a: string, b: number }`'"

*/

DEFINE FIELD OVERWRITE fd ON c TYPE { a: string, b: number };
DEFINE FIELD OVERWRITE fd.* ON c TYPE number;
```

The above will fail with the following error:

```surql
'Cannot set field `fd.*` with type `number` as it mismatched with field `fd` with type `{ a: string, b: number }`'
```

### Simple data types

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Set a field to have the string data type
DEFINE FIELD email ON TABLE user TYPE string;

-- Set a field to have the datetime data type
DEFINE FIELD created ON TABLE user TYPE datetime;

-- Set a field to have the bool data type
DEFINE FIELD locked ON TABLE user TYPE bool;

-- Set a field to have the number data type
DEFINE FIELD login_attempts ON TABLE user TYPE number;
```

A `|` vertical bar can be used to allow a field to be one of a set of types. The following example shows a field that can be a [`UUID`](/docs/surrealql/datamodel/uuid) or an [`int`](/docs/surrealql/datamodel/numbers#integer-numbers), perhaps for `user` records that have varying data due to two diffent legacy ID types.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Set a field to have either the uuid or int type
DEFINE FIELD user_id ON TABLE user TYPE uuid|int;
```

### Array type

You can also set a field to have the array data type. The array data type can be used to store a list of values. You can also set the data type of the array's contents, as well as the required number of items that it must hold.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Set a field to have the array data type
DEFINE FIELD roles ON TABLE user TYPE array<string>;

-- Set a field to have the array data type, equivalent to `array<any>`
DEFINE FIELD posts ON TABLE user TYPE array;

-- Set a field to have the array object data type
DEFINE FIELD emails ON TABLE user TYPE array<object>;

-- Set a field that holds exactly 640 bytes
DEFINE FIELD bytes ON TABLE data TYPE array<int, 640> ASSERT $value.all(|$val| $val IN 0..=255);

-- Field for a block in a game showing the possible distinct directions a character can move next.
-- The array can contain no more than four directions
DEFINE FIELD next_paths ON TABLE block 
  TYPE array<"north" | "east" | "south" | "west"> 
  VALUE $value.distinct() 
  ASSERT $value.len() <= 4;
```

### Making a field optional

You can make a field optional by wrapping the inner type in an `option`, which allows you to store `NONE` values in the field.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- A user may enter a biography, but it is not required.
-- By using the option type you also allow for NONE values.
DEFINE FIELD biography ON TABLE user TYPE option<string>;
```

The example below shows how to define a field `user` on a `POST` table. The field is of type [record](/docs/surrealql/datamodel/records). This means that the field can store a `record<user>` or `NONE`.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE FIELD user ON TABLE post TYPE option<record<user>>;
```

### Flexible data types

Flexible types allow you to have `SCHEMALESS` functionality on a `SCHEMAFULL` table. This is necessary for working with nested `object` types that need to be able to accept fields that have not yet been defined.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

DEFINE TABLE user SCHEMAFULL;
DEFINE FIELD name ON TABLE user TYPE string;
DEFINE FIELD metadata ON TABLE user FLEXIBLE TYPE object;
DEFINE FIELD metadata.user_id ON TABLE user TYPE int;
DEFINE FIELD name ON user TYPE string;
DEFINE FIELD metadata ON TABLE user TYPE object FLEXIBLE;
DEFINE FIELD metadata.user_id ON TABLE user TYPE int;
```

Taking the following `CREATE` statement:

```surql
CREATE ONLY user SET
  name = "User1",
  metadata = {
      user_id: 8876687,
      country_code: "ee",
      time_zone: "EEST",
      age: 25
};
```

Without `FLEXIBLE`, the `metadata` field will effectively be a `SCHEMAFULL` object with only a single defined field.

In versions of SurrealDB before 3.0, the result of the above statement was a record in which the `metadata` field was only able to populate the `user_id` field.

```surql
{
	id: user:ke8w4u38gbm3ofp2u8fb,
	metadata: {
		user_id: 8876687
	},
  name: "User1"
}
```

As of version 3.0, the statement now returns an error upon finding the first field that was not defined in the schema.

```surql
"Found field 'metadata.age', but no such field exists for table 'user'"
```

With `FLEXIBLE`, the output will be as expected as the schema now allows any sort of object to be a field on the `user` table — as long as values for `name` and `metadata.user_id` are present.

```surql title="Response"
{
	id: user:lsdk473e279oik1k484b,
	metadata: {
		age: 25,
		country_code: 'ee',
		time_zone: 'EEST',
		user_id: 8876687
	},
	name: 'User1'
}
```

### Using the `DEFAULT` clause to set a default value

You can set a default value for a field using the `DEFAULT` clause. The default value will be used if no value is provided for the field.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- A user is not locked by default.
DEFINE FIELD locked ON TABLE user TYPE bool
-- Set a default value if empty
  DEFAULT false;
```

### Using the `DEFAULT` and `ALWAYS` clause

<Since v="v2.2.0" />

In addition to the `DEFAULT` clause, you can use the `DEFAULT ALWAYS` clause to set a default value for a field. The `ALWAYS` keyword indicates that the `DEFAULT` clause is used not only on `CREATE`, but also on `UPDATE` if the value is empty (NONE).

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

DEFINE TABLE product SCHEMAFULL;
-- Set a default value of 123.456 for the primary field
DEFINE FIELD primary ON product TYPE number DEFAULT ALWAYS 123.456;
```

With the above definition, the `primary` field will be set to `123.456` when a new `product` is created without a value for the `primary` field or with a value of `NONE`, and when an existing `product` is updated if the value is specified the result will be the new value.

In the case of `NULL` or a mismatching type, an error will be returned.

```surql
-- This will return an error
CREATE product:test SET primary = NULL;

-- result 
"Couldn't coerce value for field `primary` of `product:test`: Expected `number` but found `NULL`"
```

On the other hand, if a valid number is provided during creation or update, that number will be used instead of the default value. In this case, `123.456`.

```surql
-- This will set the value of the `primary` field to `123.456`
CREATE product:test;

-- This will set the value of the `primary` field to `463.456`
UPSERT product:test SET primary = 463.456;

-- This will set the value of the `primary` field to `123.456`
UPSERT product:test SET primary = NONE;

```

```surql title="Query"
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: post:test, tags: [] }]"

[[test.results]]
value = "[{ id: post:test, tags: [{ color: 'red', name: 'test' }] }]"

[[test.results]]
value = "[{ id: post:test, tags: [{ color: 'red', name: 'test' }, { color: 'blue', name: 'test' }] }]"

*/

DEFINE TABLE post SCHEMAFULL;
DEFINE FIELD tags ON post TYPE array<object> DEFAULT ALWAYS [];
DEFINE FIELD tags.*.color ON post TYPE string DEFAULT ALWAYS 'red';
DEFINE FIELD tags.*.name ON post TYPE string;
--
CREATE post:test;
UPSERT post:test SET tags += { name: 'test' };
UPSERT post:test SET tags += { name: 'test', color: 'blue' };
```

```surql title="Response"
[{ id: post:test, tags: [] }]

[{ id: post:test, tags: [{ color: 'red', name: 'test' }] }]

[{ id: post:test, tags: [{ color: 'red', name: 'test' }, { color: 'blue', name: 'test' }] }]
```

### Using the `VALUE` clause to set a field's value

The `VALUE` clause differs from `DEFAULT` in that a default value is calculated if no other is indicated, otherwise accepting the value given in a query.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: user:a856d1al1kiw5aasfu23, updated: d'1900-01-01T00:00:00Z' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: user:a856d1al1kiw5aasfu23, updated: d'1910-01-01T00:00:00Z' }]"
skip-record-id-key = true

*/

DEFINE FIELD updated ON TABLE user DEFAULT time::now();

-- Set `updated` to the year 1900
CREATE user SET updated = d"1900-01-01";
-- Then set to the year 1910
UPDATE user SET updated = d"1910-01-01";
```

A `VALUE` clause, on the other hand, will ignore attempts to set the field to any other value.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: user:059nm0eomnrwvq692i3i, updated: d'2025-10-07T07:48:06.852628Z' }]"
skip-record-id-key = true
skip-datetime = true

[[test.results]]
value = "[{ id: user:059nm0eomnrwvq692i3i, updated: d'2025-10-07T07:48:06.855302Z' }]"
skip-record-id-key = true
skip-datetime = true

*/

DEFINE FIELD updated ON TABLE user VALUE time::now();

-- Ignores 1900 date, sets `updated` to current time
CREATE user SET updated = d"1900-01-01";
-- Ignores again, updates to current time
UPDATE user SET updated = d"1900-01-01";
```

As the example above shows, a `VALUE` clause sets the value every time a record is modified (created or updated). However, the value will not be recalculated in a `SELECT` statement, which simply accesses the current set value.

```surql
DEFINE FIELD updated ON TABLE user VALUE time::now();

CREATE user:one;
SELECT * FROM ONLY user:one;
-- Sleep for one second
SLEEP 1s;
-- `updated` is still the same
SELECT * FROM ONLY user:one;
```

To create a field that is calculated each time it is accessed, a [`computed field`](/docs/surrealql/statements/define/field#computed-fields) can be used.

```surql
DEFINE FIELD accessed_at ON TABLE user COMPUTED time::now();

CREATE user:one;
SELECT * FROM ONLY user:one;
-- Sleep for one second
SLEEP 1s;
-- `accessed_at` is a different value now
SELECT * FROM ONLY user:one;
```

### Altering a passed value

You can alter a passed value using the `VALUE` clause. This is useful for altering the value of a field before it is stored in the database.

In the example below, the `VALUE` clause is used to ensure that the email address is always stored in lowercase characters by using the [`string::lowercase`](/docs/surrealql/functions/database/string#stringlowercase) function.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Ensure that an email address is always stored in lowercase characters
DEFINE FIELD email ON TABLE user TYPE string
  VALUE string::lowercase($value);
```

## Asserting rules on fields

You can take your field definitions even further by using asserts. Assert can be used to ensure that your data remains consistent. For example you can use asserts to ensure that a field is always a valid email address, or that a number is always positive.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Give the user table an email field. Store it in a string
DEFINE FIELD email ON TABLE user TYPE string
  -- Check if the value is a properly formatted email address
  ASSERT string::is_email($value);
```

As the `ASSERT` clause expects an expression that returns a boolean, an assertion with a custom message can be manually created by returning `true` in one case and using a [`THROW`](/docs/surrealql/statements/throw) clause otherwise.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
error = "'An error occurred: Tried to make a { id: data:one, num: 11 } but `num` field requires an even number'"

*/

DEFINE FIELD num ON data TYPE int ASSERT {
    IF $input % 2 = 0 {
        RETURN true
    } ELSE {
        THROW "Tried to make a " + <string>$this + " but `num` field requires an even number"
    }
};

CREATE data:one SET num = 11;
```

```surql title="Error output"
'An error occurred: Tried to make a { id: data:one, num: 11 } but `num` field requires an even number'
```

### Making a field `READONLY`

<Since v="v1.2.0" />

The `READONLY` clause can be used to prevent any updates to a field. This is useful for fields that are automatically updated by the system. To make a field `READONLY`, add the `READONLY` clause to the `DEFINE FIELD` statement. As seen in the example below, the `created` field is set to `READONLY`.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE FIELD created ON resource VALUE time::now() READONLY;
```

## Using `IF NOT EXISTS` clause

The `IF NOT EXISTS` clause can be used to define a field only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining a field in SurrealDB if you want to ensure that the field is only created if it does not already exist. If the field already exists, the `DEFINE FIELD` statement will return an error.

It's particularly useful when you want to safely attempt to define a field without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the field definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a field and overwrite an existing one if it already exists, ensuring that the latest version of the definition is always in use

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a field if it does not already exist
DEFINE FIELD IF NOT EXISTS email ON TABLE user TYPE string;
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to define a field and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing field definition. If the field already exists, the `DEFINE FIELD` statement will overwrite the existing definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Overwrite the current field definition if it already exists
DEFINE FIELD OVERWRITE example ON TABLE user TYPE string;
```

## Setting permissions on fields

By default, the permissions on a field will be set to FULL unless otherwise specified.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ events: {  }, fields: { some_info: 'DEFINE FIELD some_info ON some_table TYPE string PERMISSIONS FULL' }, indexes: {  }, lives: {  }, tables: {  } }"

*/

DEFINE FIELD some_info ON TABLE some_table TYPE string;
INFO FOR TABLE some_table;
```

```surql title="Response"
{
	events: {},
	fields: {
		info: 'DEFINE FIELD info ON some_table TYPE string PERMISSIONS FULL'
	},
	indexes: {},
	lives: {},
	tables: {}
}
```

You can set permissions on fields to control who can perform operations on them using the `PERMISSIONS` clause. The `PERMISSIONS` clause can be used to set permissions for `SELECT`, `CREATE`, and `UPDATE` operations. The `DELETE` operation only relates to records and, as such, is not available for fields.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Set permissions for the email field
DEFINE FIELD email ON TABLE user
  PERMISSIONS
    FOR select WHERE published=true OR user=$auth.id
    FOR update WHERE user=$auth.id OR $auth.role="admin";
```

## Array with allowed values

By using an Access Control List as an example we can show how we can restrict what values can be stored in an array. In this example we are using an array to store the permissions for a user on a resource. The permissions are restricted to a specific set of values.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ email: 'Tobie.Hitchcock@surrealdb.com', firstName: 'Tobie', id: user:tobie, lastName: 'Hitchcock' }]""
skip-record-id-key = true

[[test.results]]
value = "[{ email: 'c@d.com', firstName: 'A', id: user:abc, lastName: 'B' }]"

[[test.results]]
value = "[{ email: 'g@h.com', firstName: 'E', id: user:efg, lastName: 'F' }]""

[[test.results]]
value = "[{ id: document:SurrealDB_whitepaper, name: 'some messaging queue' }]"

[[test.results]]
value = "[{ id: acl:8xq08j6rackxaydwu5u9, permissions: ['create', 'write', 'read'], resource: document:SurrealDB_whitepaper, user: user:tobie }]"

[[test.results]]
value = "[{ id: acl:lov6b2y9jf7m05jsjzrc, permissions: ['read', 'delete'], resource: document:SurrealDB_whitepaper, user: user:abc }]"

[[test.results]]
error = ""Found [] for field `permissions`, with record `acl:invalid`, but field must conform to: (array::len($value) > 0) AND ($value ALLINSIDE ['create', 'read', 'write', 'delete'])""

[[test.results]]
error = ""Found ['all'] for field `permissions`, with record `acl:also_invalid`, but field must conform to: (array::len($value) > 0) AND ($value ALLINSIDE ['create', 'read', 'write', 'delete'])""

*/

-- An ACL can be applied to any kind of resource (record)
DEFINE FIELD resource ON TABLE acl TYPE record;
-- We associate the acl with a user using record<user>
DEFINE FIELD user ON TABLE acl TYPE record<user>;

-- The permissions for the user+resource will be stored in an array.
DEFINE FIELD permissions ON TABLE acl TYPE array
  -- The array must not be empty because at least one permission is required to make a valid ACL
  -- The items in the array must also be restricted to specific permissions
  ASSERT
      array::len($value) > 0
      AND $value ALLINSIDE ["create", "read", "write", "delete"];

-- SEE IT IN ACTION
-- 1: Add users
CREATE user:tobie SET firstName = 'Tobie', lastName = 'Hitchcock',
  email = 'Tobie.Hitchcock@surrealdb.com';
CREATE user:abc SET firstName = 'A', lastName = 'B',
  email = 'c@d.com';
CREATE user:efg SET firstName = 'E', lastName = 'F',
  email = 'g@h.com';

-- 2: Create a resource
CREATE document:SurrealDB_whitepaper SET
  name = "some messaging queue";

-- 3: Associate with ACL
CREATE acl SET user = user:tobie, resource = document:SurrealDB_whitepaper, permissions = ["create", "write", "read"];
CREATE acl SET user = user:abc, resource = document:SurrealDB_whitepaper, permissions = ["read", "delete"];

-- Test Asserts using failure examples
-- A: Create ACL without permissions field
CREATE acl:invalid SET
  user = user:efg,
  permissions = [], # FAIL - permissions must not be empty
  resource = document:SurrealDB_whitepaper;
-- B: Create acl with invalid permisson
CREATE acl:also_invalid SET
  user = user:efg,
  permissions = ["all"], # FAIL - This value is not allowed in the array
  resource = document:SurrealDB_whitepaper;
```

## Using RegEX to validate a string

You can use the `ASSERT` clause to apply a regular expression to a field to ensure that it matches a specific pattern. In the example below, the `ASSERT` clause is used to ensure that the `countrycode` field is always a valid ISO-3166 country code.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Specify a field on the user table
DEFINE FIELD countrycode ON user TYPE string
	-- Ensure country code is ISO-3166
	ASSERT $value = /[A-Z]{3}/
	-- Set a default value if empty
	VALUE $value OR $before OR 'GBR'
;
```

## Interacting with other fields of the same record

While a `DEFINE TABLE` statement represents a template for any subsequent records to be created, a `DEFINE FIELD` statement pertains to concrete field data of a record. As such, a `DEFINE FIELD` statement gives access to the record's other fields through their names, as well as the current field through the [`$value`](/docs/surrealql/parameters#value) parameter.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ first_name: 'bob', id: person:bpkup0u5zv84xqxne2j5, last_name: 'bobson', name: 'bob bobson' }]"
skip-record-id-key = true

*/

DEFINE TABLE person SCHEMAFULL;

DEFINE FIELD first_name ON TABLE person TYPE string VALUE string::lowercase($value);
DEFINE FIELD last_name  ON TABLE person TYPE string VALUE string::lowercase($value);
DEFINE FIELD name       ON TABLE person             VALUE first_name + ' ' + last_name;

// Creates a `person` with the name "bob bobson"
CREATE person SET first_name = "BOB", last_name = "BOBSON";
```

The `$this` parameter gives access to the entire record on which a field is defined.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ age: 6, extra_self: { age: 6, id: person:one, name: 'Little person' }, id: person:one, name: 'Little person' }]"

*/

DEFINE FIELD extra_self ON TABLE person VALUE $this;
CREATE person:one SET name = "Little person", age = 6;
```

```surql title="Output"
[
	{
		age: 6,
		extra_self: {
			age: 6,
			id: person:one,
			name: 'Little person'
		},
		id: person:one,
		name: 'Little person'
	}
]
```

## Order of operations when setting a field's value

As `DEFINE FIELD` statements are computed in alphabetical order, be sure to keep this in mind when using fields that rely on the values of others.

The following example is identical to the above except that `full_name` has been chosen for the previous field `name`. The `full_name` field will be calculated after `first_name`, but before `last_name`.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ first_name: 'bob', full_name: 'bob BOBSON', id: person:l07j1ly4oher21g80fr4, last_name: 'bobson' }]"
skip-record-id-key = true

*/

DEFINE TABLE person SCHEMAFULL;

DEFINE FIELD first_name ON TABLE person TYPE string VALUE string::lowercase($value);
DEFINE FIELD last_name  ON TABLE person TYPE string VALUE string::lowercase($value);
DEFINE FIELD full_name  ON TABLE person             VALUE first_name + ' ' + last_name;

// Creates a `person` with `full_name` of "bob BOBSON", not "bob bobson"
CREATE person SET first_name = "Bob", last_name = "BOBSON";
```

A good rule of thumb is to organize your `DEFINE FIELD` statements in alphabetical order so that the field definitions show up in the same order as that in which they are computed.

## Defining a literal on a field

<Since v="v2.0.0" />

A field can also be defined as a [literal type](/docs/surrealql/datamodel/literals), by specifying one or more possible values and/or permitted types.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ coffee: { special_order: 'Venti Quadruple Ristretto Half-Decaf Soy Latte with 4 pumps of sugar-free vanilla syrup' }, id: order:good }]"

[[test.results]]
error = ""Couldn't coerce value for field `coffee` of `order:bad`: Expected `'regular' | 'large' | { special_order: string }` but found `'small'`""

*/

DEFINE FIELD coffee ON TABLE order TYPE "regular" | "large" | { special_order: string };

CREATE order:good SET coffee = { special_order: "Venti Quadruple Ristretto Half-Decaf Soy Latte with 4 pumps of sugar-free vanilla syrup" };
CREATE order:bad SET coffee = "small";
```

```surql title="Response"
-------- Query --------

[
	{
		coffee: {
			special_order: 'Venti Quadruple Ristretto Half-Decaf Soy Latte with 4 pumps of sugar-free vanilla syrup'
		},
		id: order:good
	}
]

-------- Query --------
"Found 'small' for field `coffee`, with record `order:bad`, but expected a 'regular' | 'large' | { special_order: string }"
```

One more example of a literal containing settings for a [full text search](/docs/surrealdb/models/full-text-search) filter:

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE FIELD filter ON TABLE search_settings TYPE
      "None"
    | { type: "Ascii" }
    | { type: "EdgeNgram", from: int, to: int }
    | { type: "Lowercase" }
    | { type: "Ngram", from: int, to: int }
    | { type: "Snowball", language: string }
    | { type: "Uppercase" };
```

## Defining a `TYPE` for the `id` field

The `DEFINE FIELD` statement can be defined for the `id` field to specify the acceptable type of ID.

```surql
DEFINE FIELD id ON TABLE something TYPE string;
DEFINE FIELD id ON TABLE something TYPE int;
DEFINE FIELD id ON TABLE something TYPE uuid;
```

Complex IDs can be specified as well.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
error = ""Couldn't coerce value for field `id` of `log:bad`: Expected `[record, 'info' | 'warn' | 'error', datetime]` but found `'bad'`""

[[test.results]]
value = "[{ id: log:[user:one, 'info', d'2025-10-08T00:53:28.328764Z'], message: 'Database started' }]"
skip-datetime = true

*/

-- using multiple data types for a Complex Record ID
DEFINE FIELD id ON TABLE log TYPE [record, "info" | "warn" | "error", datetime];

-- Incorrect ID format, generates an error
CREATE log:bad SET level = "info", time = time::now(), message = "Database started";

-- Acceptable ID format
CREATE log:[user:one, "info", time::now()] SET message = "Database started";
```

```surql title="Output"
-------- Query --------

"Couldn't coerce value for field `id` of `log:bad`: Expected `[record, 'info' | 'warn' | 'error', datetime]` but found `'bad'`"

-------- Query --------

[
	{
		id: log:[
			user:one,
			'info',
			d'2025-03-25T03:36:16.323Z'
		],
		message: 'Database started'
	}
]
```

## Defining a reference

<Since v="v2.2.0" />

A field that is a record link (type `record`, `option<record>`, `array<record<person>>`, and so on) can be defined as a `REFERENCE`. If this clause is used, any linked to record will be able to define a field of its own of type `references` which will be aware of the incoming links.

For more information, see [the page in the datamodel section on references](/docs/surrealql/datamodel/references).


## function.mdx

---
sidebar_position: 9
sidebar_label: DEFINE FUNCTION
title: DEFINE FUNCTION statement | SurrealQL
description: The DEFINE FUNCTION statement allows you to define custom functions that can be reused throughout a database.
---

import SurrealistMini from "@components/SurrealistMini.astro";
import Since from '@components/shared/Since.astro'
import Image from '@components/Image.astro'
import RecursiveStar from "@img/image/recursive_star.png";

# `DEFINE FUNCTION` statement

The `DEFINE FUNCTION` statement allows you to define custom functions that can be reused throughout a database. When using the `DEFINE FUNCTION` statement, you can define a function that takes one or more arguments and returns a value. You can then call this function in other SurrealQL statements.

Functions can be used to encapsulate logic that you want to reuse in multiple queries. They can also be used to simplify complex queries by breaking them down into smaller, more manageable pieces. The are particularly useful when you have a complex query that you need to run multiple times with different arguments.

## Requirements

- You must be authenticated as a root owner or editor, namespace owner or editor, or database owner or editor before you can use the `DEFINE FUNCTION` statement.
- [You must select your namespace and database](/docs/surrealql/statements/use) before you can use the `DEFINE FUNCTION` statement.

## Statement syntax

```syntax title="SurrealQL Syntax"
DEFINE FUNCTION [ OVERWRITE | IF NOT EXISTS ] fn::@name( [ @argument: @type ... ] ) [ -> @value ] {
	[ @query ... ]
	[ RETURN @returned ]
} [ COMMENT @string ] [ PERMISSIONS [ NONE | FULL | WHERE @condition]]
```

## Example usage
Below shows how you can define a custom function using the `DEFINE FUNCTION` statement, and how to call it.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "'Hello, Tobie!'"

*/

-- It is necessary to prefix the name of your function with "fn::"
-- This indicates that it's a custom function
DEFINE FUNCTION fn::greet($name: string) {
	"Hello, " + $name + "!"
};

-- Returns: "Hello, Tobie!"
RETURN fn::greet("Tobie");
```
To showcase a slightly more complex custom function, this will check if a relation between two nodes exists:

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Define a function that checks if a relation exists between two nodes
DEFINE FUNCTION fn::relation_exists(
	$in: record,
	$tb: string,
	$out: record
) {
	-- Check if a relation exists between the two nodes.
	LET $results = SELECT VALUE id FROM type::table($tb) WHERE in = $in AND out = $out;
	-- Return true if a relation exists, false otherwise
    RETURN array::len($results) > 0;
};
```

## Optional arguments
If one or more ending arguments have the `option<T>` type, they can be omitted when you run the invoke the function.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ optional_present: true, required_present: true }"

[[test.results]]
value = "{ optional_present: false, required_present: true }"

*/

DEFINE FUNCTION fn::last_option($required: number, $optional: option<number>) {
	RETURN {
		required_present: type::is_number($required),
		optional_present: type::is_number($optional),
	}
};

RETURN fn::last_option(1, 2);
-- { required_present: true, optional_present: true }

RETURN fn::last_option(1);
-- { required_present: true, optional_present: false };
```

## Adding a return value

Optionally, the return value of a function can be specified.

For a function that is infallible, a return value is mostly for the sake of readability.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/


DEFINE FUNCTION fn::greet($name: string) -> string {
	"Hello, " + $name + "!"
};
```

For a function that is not infallible, specifying a return value can be used to customise error output.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
error = "Expected `number` but found `'one'`"

[[test.results]]
error = "Couldn't coerce return value from function `fn::combine_any`: Expected `number` but found `'onetwo'`"

*/

// Arguments must be of type 'number'
DEFINE FUNCTION fn::combine($one: number, $two: number) -> number {
  $one + $two
};

// Accepts any value but expects the return type 'number'
DEFINE FUNCTION fn::combine_any($one: any, $two: any) -> number {
  $one + $two
};

fn::combine("one", "two");
fn::combine_any("one", "two");
```

While both of these return an error, the output of the second function happens only at the point that it attempts to return the combined arguments to the function.

```surql title="Output"
-------- Query 1 --------
"Expected `number` but found `'one'`"

-------- Query 2 --------
"Couldn't coerce return value from function `fn::combine_any`: Expected `number` but found `'onetwo'`"
```

The return value of a function can even be a [literal type](/docs/surrealql/datamodel/literals). The following function returns such a type by either returning an object of a certain structure, or a string. In this case this output is used in case an application prefers to return an error as a simple string instead of [throwing](/docs/surrealql/statements/throw) an error or returning a NONE value.

```surql
DEFINE FUNCTION fn::age_and_name($user_num: int) -> { age: int, name: string } | string {
    LET $user = type::record("user", $user_num);
    IF $user.exists() {
        $user.{ name, age }
    } ELSE {
        { "Couldn't find user number " + <string>$user_num + "!" }        
    }
};

CREATE user:1 SET name = "Billy", age = 15;

fn::age_and_name(1);
fn::age_and_name(2);
```

```surql title="Output"
-------- Query 1 --------

{ age: 15, name: 'Billy' }

-------- Query 2 --------

"Couldn't find user number 2!"
```

## Recursive functions

A function is able to call itself, making it a recursive function. One example of a recursive function is the one below which creates a relation between each and every record passed in.

Consider a situation in which seven person records exist. First, `person:1` will need to be related to the rest of the `person` records, after which there are no more relations to create for it. Following this, the relations for `person:2` and all the other records except for `person:1` will need to be created, and so on.

This can be done in a recursive function by creating all the relations between the first record and the remaining records, after which the function calls itself by passing in all the records except the first. This continues until the function receives less than two records, in which case it ceases calling itself by doing nothing, thereby ending the recursion.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: person:1 }, { id: person:2 }, { id: person:3 }, { id: person:4 }, { id: person:5 }, { id: person:6 }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ "->to": { "->?": [person:3, person:4, person:6, person:5, person:2] }, id: person:1 }, { "->to": { "->?": [person:6, person:3, person:5, person:4] }, id: person:2 }, { "->to": { "->?": [person:6, person:5, person:4] }, id: person:3 }, { "->to": { "->?": [person:5, person:6] }, id: person:4 }, { "->to": { "->?": [person:6] }, id: person:5 }, { "->to": { "->?": [] }, id: person:6 }]"

*/

DEFINE FUNCTION fn::relate_all($records: array<record>) {
  IF $records.len() < 2 {
      -- Don't do anything, ending the recursion
  }  ELSE {
      LET $first = $records[0];
      LET $remainder = $records[1..];
      FOR $counterpart IN $remainder {
          RELATE $first->to->$counterpart;
      };
      fn::relate_all($remainder);
  }
};

CREATE |person:1..8|;

fn::relate_all(SELECT VALUE id FROM person);

SELECT id, ->to->? FROM person;
```

The last query [can be viewed graphically](/blog/whats-new-in-surrealist-3-2#graph-visualisation) inside Surrealist, leading to an output showing a seven-pointed star.

<Image
  alt="An image of a seven-pointed star created visually by relating seven records to each other and displayed inside Surrealist's graph view."
  src={{
    light: RecursiveStar,
    dark: RecursiveStar,
  }}
/>

## Permissions

You can set the permissions for a custom function using the `PERMISSIONS` clause. The `PERMISSIONS` clause is mostly used to restrict who can access a function and what data they can access. It can be set to `NONE`, `FULL`, or `WHERE @condition`.

- `FULL`: When Full permissions are granted [record](/docs/surrealdb/security/authentication#record-users) users have access to the function. This is the default permission when not specified.
- `NONE`: When this permission is granted, [record](/docs/surrealdb/security/authentication#record-users) users have no access to the defined function.
- `WHERE @condition`: Permissions are granted to the function based on the specified condition.


> [!NOTE]
> The examples below use the [`Surreal Deal Store`](/docs/surrealql/demo#surreal-deal-store---there-is-a-lot-in-store-for-you-recommended) dataset.

### Using the `FULL` permission

The `FULL` permission grants all users access to the function. The following example defines a function that fetches all products from the `product` table and grants the function full permissions to access the data to all users.

<SurrealistMini url="https://app.surrealdb.com/mini?query=--+Define+a+function+to+fetch+all+products.+All+users+can+access+this+function%0ADEFINE+FUNCTION+fn%3A%3AfetchAllProducts%28%29+%7B%0A%09RETURN+%28SELECT+*+FROM+product+LIMIT+10%29%3B%0A%7D+PERMISSIONS+FULL%3B%0A%0A--+Returns%3A+The+first+10+products+in+the+product+table%0ARETURN+fn%3A%3AfetchAllProducts%28%29%3B&dataset=surreal-deal-store&orientation=horizontal"/>

### Using the `NONE` permission

The `NONE` permission denies all [record](/docs/surrealdb/security/authentication#record-users) users access to the function. The following example defines a function that fetches all products from the `product` table

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[]"

*/

-- Define a function that fetches all expiration years from the payment_details table and denies access to all none-admin users
DEFINE FUNCTION fn::fetchAllPaymentDetails() {
	SELECT stored_cards.expiry_year FROM payment_details LIMIT 5
} PERMISSIONS NONE;

RETURN fn::fetchAllPaymentDetails();
```

### Using the `WHERE` clause

The `WHERE` clause allows you to specify a condition that determines the permissions granted to the function. The condition must evaluate to a boolean value. If the condition evaluates to `true`, the function is granted permissions. If the condition evaluates to `false`, the function is not granted permissions.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Define a function that fetches all products with the condition that only admin users can access it
DEFINE FUNCTION fn::fetchAllProducts() {
	 SELECT * FROM product LIMIT 10
} PERMISSIONS WHERE $auth.admin = true;
```

## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define a function only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining a function in SurrealDB if you want to ensure that the function is only created if it does not already exist. If the function already exists, the `DEFINE FUNCTION` statement will return an error.

It's particularly useful when you want to safely attempt to define a function without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the function definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a function and overwrite an existing one if it already exists, ensuring that the latest version of the function definition is always in use

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a FUNCTION if it does not already exist
DEFINE FUNCTION IF NOT EXISTS fn::example() {};
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to define a function and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing user definition. If the user already exists, the `DEFINE FUNCTION` statement will overwrite the existing definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a FUNCTION and overwrite if it already exists
DEFINE FUNCTION OVERWRITE fn::example() {};
```

## Functions as custom middleware

<Since v="v3.0.0" />

A `DEFINE FUNCTION` statement can be used to define a function for use as custom middleware. For more details on defining a custom function in this manner, see the [`DEFINE API`](/docs/surrealql/statements/define/api#custom-middleware) page.

## index.mdx

---
sidebar_position: 1
sidebar_label: Overview
title: DEFINE statement | SurrealQL
description: The DEFINE statement can be used to specify authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---

# DEFINE statement

The DEFINE statement can be used to specify instructions to the schema such as authentication access and behaviour, global parameters, table configurations, table events, analyzers, and indexes.

> [!NOTE]
> Before SurrealDB version 3.0.0-beta, the `FULLTEXT ANALYZER` clause used the syntax `SEARCH ANALYZER`.

```syntax title="SurrealQL Syntax"
DEFINE [
	NAMESPACE [ OVERWRITE | IF NOT EXISTS ] @name
	| DATABASE [ OVERWRITE | IF NOT EXISTS ] @name
	| USER [ OVERWRITE | IF NOT EXISTS ] @name ON [ ROOT | NAMESPACE | DATABASE ] [ PASSWORD @pass | PASSHASH @hash ] ROLES @roles
	| TABLE [ OVERWRITE | IF NOT EXISTS ] @name
		[ DROP ]
		[ SCHEMAFULL | SCHEMALESS ]
		[ AS SELECT @projections
			FROM @tables
			[ WHERE @condition ]
			[ GROUP [ BY ] @groups ]
		]
		[ PERMISSIONS [ NONE | FULL
			| FOR select @expression
			| FOR create @expression
			| FOR update @expression
			| FOR delete @expression
		] ]
	| EVENT [ OVERWRITE | IF NOT EXISTS ] @name ON [ TABLE ] @table WHEN @expression THEN @expression
	| FIELD [ OVERWRITE | IF NOT EXISTS ] @name ON [ TABLE ] @table
		[ TYPE @type ]
		[ VALUE @expression ]
		[ ASSERT @expression ]
		[ PERMISSIONS [ NONE | FULL
			| FOR select @expression
			| FOR create @expression
			| FOR update @expression
			| FOR delete @expression
		] ]
	| PARAM [ OVERWRITE | IF NOT EXISTS ] $@name VALUE @value
	| FUNCTION [ OVERWRITE | IF NOT EXISTS ] fn::@name ( [ ( @argument:@type ... ) ] ) { [@query] [RETURNS @returned] }
	| ANALYZER [ OVERWRITE | IF NOT EXISTS ] @name
		[ TOKENIZERS @tokenizers ]
		[ FILTERS @filters ]
	| INDEX [ OVERWRITE | IF NOT EXISTS ] @name ON [ TABLE ] @table [ FIELDS | COLUMNS ] @fields
		[ UNIQUE | FULLTEXT ANALYZER @analyzer [ BM25 [(@k1, @b)] ] [ HIGHLIGHTS ] ]
	| SEQUENCE [ OVERWRITE | IF NOT EXISTS ] @name
		[ BATCH @batch ]
		[ START @start ]
	| ACCESS [ OVERWRITE | IF NOT EXISTS ] @name ON [ NAMESPACE | DATABASE ]
		TYPE [
			JWT [ ALGORITHM @algorithm KEY @key | URL @url ]
			| RECORD
				[ SIGNUP @expression ]
				[ SIGNIN @expression ]
				[ WITH JWT [ ALGORITHM @algorithm KEY @key | URL @url ] [ WITH ISSUER KEY @key ] ]
		]
		[ DURATION [ FOR TOKEN @duration ] [ FOR SESSION @duration ] ]
    [ COMMENT @string ]
]
```

The [INFO](/docs/surrealql/statements/info) statement can be used to see which definition statements currently exist in a database connection. All `DEFINE` statements can be followed up with a `COMMENT`.

An example of defining a field on a table, followed by an `INFO` command for the same table:

```surql
DEFINE FIELD name ON TABLE person TYPE string COMMENT "Todo: add assertion for maximum length";
INFO FOR TABLE person;
```

```surql output="Response"
{
	events: {},
	fields: {
		name: "DEFINE FIELD name ON person TYPE string COMMENT 'Todo: add assertion for maximum length' PERMISSIONS FULL"
	},
	indexes: {},
	lives: {},
	tables: {}
}
```

An example of defining a user and a table for a database, followed by an `INFO` command for the current database:

```surql
DEFINE USER db_user ON DATABASE PASSWORD "strongpassword" ROLES OWNER;
DEFINE TABLE person SCHEMAFULL;
INFO FOR DB;
```

```surql output="Response"
{
    "accesses": {},
    "analyzers": {},
    "functions": {},
    "models": {},
    "params": {},
    "tables": {
        "person": "DEFINE TABLE person TYPE ANY SCHEMAFULL PERMISSIONS NONE"
    },
    "users": {
        "db_user": "DEFINE USER db_user ON DATABASE PASSHASH '$argon2id$v=19$m=19456,t=2,p=1$P5nVYXOMvk6rEz67kjL5Dg$0T/XNmgIaB+lK0IPspg1l8LruzNK96jd/PvktRCB/ww' ROLES OWNER"
    }
}
```


## indexes.mdx

---
sidebar_position: 10
sidebar_label: DEFINE INDEX
title: DEFINE INDEX statement | SurrealQL
description: Just like in other databases, SurrealDB uses indexes to help optimize query performance. An index can consist of one or more fields in a table and can enforce a uniqueness constraint.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

import SurrealistMini from "@components/SurrealistMini.astro";

> [!NOTE]
> Before SurrealDB version 3.0.0-beta, the `FULLTEXT ANALYZER` clause used the syntax `SEARCH ANALYZER`.

# `DEFINE INDEX` statement

Just like in other databases, SurrealDB uses indexes to help optimize query performance. An index can consist of one or more fields in a table and can enforce a uniqueness constraint. If you don't intend for your index to have a uniqueness constraint, then the fields you select for your index should have a high degree of cardinality, meaning that there is a high amount of diversity between the data in the indexed table records.

## Requirements

- You must be authenticated as a root owner or editor, namespace owner or editor, or database owner or editor before you can use the `DEFINE INDEX` statement.
- [You must select your namespace and database](/docs/surrealql/statements/use) before you can use the `DEFINE INDEX` statement.

## Statement syntax

<Tabs syncKey="define-index-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="Basic syntax"
DEFINE INDEX [ OVERWRITE | IF NOT EXISTS ] @name
    ON [ TABLE ] @table 
    [ FIELDS | COLUMNS ] @fields
    [ @special_clause ]
    [ COMMENT @string ]
    [ CONCURRENTLY ]
    [ DEFER ]
```

The `@special_clause` part of the statement is an optional part in which an index can be declared for special usage such as guaranteeing unique values, full-text search, and so on. The available clauses are:

```syntax title="Special index clauses"
UNIQUE
| COUNT
| FULLTEXT ANALYZER @analyzer [ BM25 [(@k1, @b)] ] [ HIGHLIGHTS ]
| HNSW DIMENSION @dimension [ TYPE @type ] [DIST @distance] [ EFC @efc ] [ M @m ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineIndexAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "INDEX" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [
        { type: "Terminal", text: "OVERWRITE" },
        { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] }
      ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Terminal", text: "ON" },
      { type: "Optional", child: { type: "Terminal", text: "TABLE" } },
      { type: "NonTerminal", text: "@table" },
      { type: "Choice", index: 1, children: [ { type: "Terminal", text: "FIELDS" }, { type: "Terminal", text: "COLUMNS" } ] },
      { type: "NonTerminal", text: "@fields" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [
        { type: "Terminal", text: "UNIQUE" },
        { type: "Terminal", text: "COUNT" },
        { type: "Sequence", children: [
          { type: "Terminal", text: "FULLTEXT" },
          { type: "Terminal", text: "ANALYZER" },
          { type: "NonTerminal", text: "@analyzer" },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "BM25" }, { type: "Optional", child: { type: "Terminal", text: "(@k1, @b)" } } ] } },
          { type: "Optional", child: { type: "Terminal", text: "HIGHLIGHTS" } }
        ] },
        { type: "Sequence", children: [
          { type: "Terminal", text: "HNSW" },
          { type: "Terminal", text: "DIMENSION" },
          { type: "NonTerminal", text: "@dimension" },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TYPE" }, { type: "NonTerminal", text: "@type" } ] } },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "DIST" }, { type: "NonTerminal", text: "@distance" } ] } },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "EFC" }, { type: "NonTerminal", text: "@efc" } ] } },
          { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "M" }, { type: "NonTerminal", text: "@m" } ] } }
        ] }
      ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } },
      { type: "Optional", child: { type: "Terminal", text: "CONCURRENTLY" } }
    ]}
  ]
};

<RailroadDiagram ast={defineIndexAst} className="my-6" />

  </TabItem>
</Tabs>

## Index types

SurrealDB offers a range of indexing capabilities designed to optimize data retrieval and search efficiency.

### Standard (non-unique) index

An index without any special clauses allows for the indexing of attributes that may have non-unique values, facilitating efficient data retrieval. Non-unique indexes help index frequently appearing data in queries that do not require uniqueness, such as categorization tags or status indicators.

Let's create a non-unique index for an age field on a user table.

```surql
-- optimise queries looking for users of a given age
DEFINE INDEX userAgeIndex ON TABLE user COLUMNS age;
```

### Unique index

Ensures each value in the index is unique. A unique index helps enforce uniqueness across records by preventing duplicate entries in fields such as user IDs, email addresses, and other unique identifiers.

Let's create a unique index for the email address field on a user table.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Makes sure that the email address in the user table is always unique
DEFINE INDEX userEmailIndex ON TABLE user COLUMNS email UNIQUE;
```

The created index can be tested using the [`INFO` statement](/docs/surrealql/statements/info).

```surql
INFO FOR TABLE user;
```
The `INFO` statement will help you understand what indexes are defined in your `TABLE`.

```surql
{
    "events": {},
    "fields": {},
    "indexes": {
        "userEmailIndex": {
            sql: "DEFINE INDEX userEmailIndex ON user FIELDS email UNIQUE"
        }
    },
    "lives": {},
    "tables": {}
}
```

As we defined a `UNIQUE` index on the `email` column, a duplicate entry for that column or field will throw an error.

```surql
-- Create a user record and set an email ID.
CREATE user:1 SET email = 'test@surrealdb.com';
```

```surql title="Response"
[
    {
        "email": "test@surrealdb.com",
        "id": "user:1"
    }
]
```

Creating another record with the same email ID will throw an error.

```surql
-- Create another user record and set the same email ID.
CREATE user:2 SET email = 'test@surrealdb.com';
```

```surql title="Response"
Database index `userEmailIndex` already contains 'test@surrealdb.com',
with record `user:1`
```

To set the same email for `user:2`, the original record must be deleted

```surql
DELETE user:1;
CREATE user:2 SET email = 'test@surrealdb.com'
```
```
[
    {
        "email": "test@surrealdb.com",
        "id": "user:2"
    }
]
```

### Composite index

A composite index spans multiple fields and columns of a table. Composite indexes are mainly used to create a unique index when the definition of what is unique pertains to more than one field.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create an index on the account and email fields of the user table
DEFINE INDEX test ON user FIELDS account, email UNIQUE;
```

### Count index

<Since v="v3.0.0" />

An index using the `COUNT` clause is used to maintain a count of the number of records in a table. This is used together with the `count()` function and `GROUP ALL` inside a query. Without a count index, the `count()` function will iterate through the records of a table when it is called.

```surql
DEFINE INDEX idx ON indexed_reading COUNT;

FOR $_ IN 0..100000 {
    CREATE reading SET temperature = rand::int(0, 10);
};

FOR $_ IN 0..100000 {
    CREATE indexed_reading SET temperature = rand::int(0, 10);
};

-- Wait a moment before running these two
-- queries to ensure the index is built
SELECT count() FROM reading GROUP ALL;
SELECT count() FROM indexed_reading GROUP ALL;
```

As a count index is declared on a table as a whole, it cannot use the `FIELDS` / `COLUMNS` clause.

```surql
-- Other clauses like `COMMENT` are fine
DEFINE INDEX idx ON users COUNT
    COMMENT "Users are expected to grow substantially so index the count"
    CONCURRENTLY;

-- But not `FIELD`
DEFINE INDEX idx2 ON person FIELD name;
```

```surql title="Output"
'There was a problem with the database: Parse error: Unexpected token `FIELD`, expected Eof
 --> [1:29]
  |
1 | DEFINE INDEX idx2 ON person FIELD name;
  |                             ^^^^^ 
'
```

### Full-text search (`FULLTEXT`) index

Enables efficient searching through textual data, supporting advanced text-matching features like proximity searches and keyword highlighting.

The [Full-Text search](/docs/surrealdb/models/full-text-search) index helps implement comprehensive search functionalities in applications, such as searching through articles, product descriptions, and user-generated content.

Let's create a full-text search index for a `name` field on a `user` table.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Define the an analyzer with
DEFINE ANALYZER example_ascii TOKENIZERS class FILTERS ascii;
-- Since 3.0.0-beta: only FULLTEXT used to benefit from concurrent full-text search
DEFINE INDEX userNameIndex ON TABLE user COLUMNS name FULLTEXT ANALYZER example_ascii BM25 HIGHLIGHTS;
```

- `SEARCH` or `FULLTEXT`: By using the `SEARCH` keyword, you enable full-text search on the specified column.
- `ANALYZER ascii`: Uses a custom [analyzer](/docs/surrealql/statements/define/analyzer) called `example_ascii` which uses the class tokenizier and `ascii` filter to analyzing the text input.
- `BM25`: Ranking algorithm used for relevance scoring.
- `HIGHLIGHTS`: Allows keyword highlighting in search results output when using the [`search::highlight`](/docs/surrealql/functions/database/search#searchhighlight) function
- `FIELDS`: a full-text search index can only be used on one field at a time. To use full-text search on more than one field, use a separate `DEFINE INDEX` statement for each one.

#### `FULLTEXT` vs. `SEARCH`

Since version 3.0.0-beta, using `FULLTEXT ANALYZER` is the syntax used for a text analyzer. The `FULLTEXT` clause allows for more performant [concurrent full-text search](https://github.com/surrealdb/surrealdb/pull/5571), as well as the ability to [use the `OR` operator](https://github.com/surrealdb/surrealdb/pull/6179).

## Vector search indexes

Vector search indexes in SurrealDB support efficient [k-nearest neighbors](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) (kNN) and [Approximate Nearest Neighbor](https://en.wikipedia.org/wiki/Nearest_neighbor_search) (ANN) operations, which are pivotal in performing similarity searches within complex, high-dimensional datasets and data types. Refer to the [Vector Search Cheat Sheet](docs/surrealdb/models/vector#vector-search-cheat-sheet) for the parameters allowed.

### Types

When defining a vector index with [HNSW](#hnsw-hierarchical-navigable-small-world), you can define the types the vector will be stored in. The `TYPE` clause is optional and can be used to specify the data type of the vector. SurrealDB supports the following types:
`F64` | `F32` | `I64` | `I32` | `I16`

- `F64`: Represents 64-bit floating-point numbers (double precision floating-point numbers).
- `F32`: Represents 32-bit floating-point numbers (single precision floating-point numbers).
- `I64`: Represents 64-bit signed integers.
- `I32`: Represents 32-bit signed integers.
- `I16`: Represents 16-bit signed integers.


> [!NOTE]
> In SurrealDB the default type for vectors is `F64`.

For example, to define a vector index with 64-bit signed integers, you can use the following query:

```surql
DEFINE INDEX idx_mtree_embedding ON Document FIELDS items.embedding HNSW DIMENSION 4 TYPE I64;
```

### HNSW (Hierarchical Navigable Small World)


This method uses a graph-based approach to efficiently navigate and search in high-dimensional spaces.
While it is an approximate technique, it offers a high-performance balance between speed and accuracy, making it ideal for very large datasets.


> [!NOTE]
> Keep in mind the in-memory nature of HNSW when considering system resource allocation.


<SurrealistMini
url="https://app.surrealdb.com/mini?query=CREATE+pts%3A3+SET+point+%3D+%5B8%2C9%2C10%2C11%5D%3B%0A%0ADEFINE+INDEX+mt_pts+ON+pts+FIELDS+point+HNSW+DIMENSION+4+DIST+EUCLIDEAN+EFC+150+M+12%3B%0A%0A%2F%2F+See+output+for+info+on+EFC%2CM%2CMO+AND+LM+%0AINFO+FOR+TABLE+pts%3B%0A%0ASELECT+id+FROM+pts+WHERE+point+%3C%7C10%2C40%7C%3E+%5B2%2C3%2C4%2C5%5D%3B&orientation=horizontal"
/>

In the example above, you may notice the `EFC` and `M` parameters. These are optional to your query but are parameters of the [HNSW algorithm](https://arxiv.org/abs/1603.09320) and can be used to tune the index for better performance.

- M (Max Connections per Element):
Defines the maximum number of bidirectional links (neighbors) per node in each layer of the graph, except for the lowest layer. This parameter controls the connectivity and overall structure of the network. Higher values of MM generally improve search accuracy but increase memory usage and construction time.

- EFC (EF construction):
Stands for "exploration factor during construction." This parameter determines the size of the dynamic list for the nearest neighbor candidates during the graph construction phase. A larger efConstruction value leads to a more thorough construction, improving the quality and accuracy of the search but increasing construction time. The default value is 150.

- M0 (Max Connections in the Lowest Layer):
Similar to M, but specifically for the bottom layer (the base layer) of the graph. This layer contains the actual data points. M0 is often set to twice the value of M to enhance search performance and connectivity at the base layer, at the cost of increased memory usage.

- LM (Multiplier for Level Generation):
Used to determine the maximum level ll for a new element during its insertion into the hierarchical structure. It is used in the formula l←⌊−ln⁡(unif(0..1))⋅mL⌋, where unif(0..1) is a uniform random variable between 0 and 1. This parameter influences the distribution of elements across different levels, impacting the overall balance and efficiency of the search structure.


> [!NOTE]
> You can only provide TYPE, M, and EFC. SurrealDB automatically computes M0 and LM with the most appropriate value. If not specified, M AND EFC are set to 12 and 150, respectively.  Refer to the [Vector Search Cheat Sheet](docs/surrealdb/models/vector#vector-search-cheat-sheet) for the parameters allowed.

#### Memory cache usage for HNSW indexes

<Since v="v3.0.0" />

HNSW vector search uses a bounded memory cache by default, reducing memory spikes and improving stability under load. The default size for the cache is 256 MiB, and can be modified via the `SURREAL_HNSW_CACHE_SIZE` [environment variable](/docs/surrealdb/cli/env).

### Brute Force method

The Brute Force method is suitable for tasks with smaller datasets or when the highest accuracy is required.
Brute Force currently supports [Euclidean](/docs/surrealql/functions/database/vector#vectordistanceeuclidean), [Cosine](/docs/surrealql/functions/database/vector#vectorsimilaritycosine), [Manhattan](/docs/surrealql/functions/database/vector#vectordistancemanhattan) and [Minkowski](/docs/surrealql/functions/database/vector#vectordistanceminkowski) distance functions.

In the example below, the query searches for points closest to the vector `[2,3,4,5]` and uses [vector functions](/docs/surrealql/functions/database/vector) to calculate the distance between two points, indicated by `<|2|>`.

<SurrealistMini
url="https://app.surrealdb.com/mini?query=CREATE+pts%3A1+SET+point+%3D+%5B1%2C2%2C3%2C4%5D%3B%0ACREATE+pts%3A2+SET+point+%3D+%5B4%2C5%2C6%2C7%5D%3B%0ACREATE+pts%3A3+SET+point+%3D+%5B8%2C9%2C10%2C11%5D%3B%0ALET+%24pt+%3D+%5B2%2C3%2C4%2C5%5D%3B%0ASELECT+id%2C+vector%3A%3Adistance%3A%3Aeuclidean%28point%2C+%24pt%29+AS+dist+FROM+pts+WHERE+point+%3C%7C2%2CEUCLIDEAN%7C%3E+%24pt%3B%0ASELECT+id+FROM+pts+WHERE+point+%3C%7C2%7C%3E+%24pt+EXPLAIN%3B&orientation=horizontal"
/>


## Verifying Index Utilization in Queries

The [`EXPLAIN` clause](/docs/surrealql/statements/select#the-explain-clause) from SurrealQL helps you understand the execution plan of the query and provides transparency around index utilization.

```surql
SELECT * FROM user WHERE email='test@surrealdb.com' EXPLAIN FULL;
```

It also reveals details about which `operation` was used by the query planner and how many records matched the search criteria.

```surql
[
    {
        "detail": {
            "plan": {
                "index": "userEmailIndex",
                "operator": "=",
                "value": "test@surrealdb.com"
            },
            "table": "user"
        },
        "operation": "Iterate Index"
    },
    {
        "detail": {
            "count": 1
        },
        "operation": "Fetch"
    }
]
```

## Rebuilding Indexes


Indexes can be rebuilt using the [`REBUILD`](/docs/surrealql/statements/rebuild) statement. This can be useful when you want to update the index definition or when you want to rebuild the index to optimize performance.

You may want to rebuild an index overtime to ensure that the index is up-to-date with the latest data in the table.

```surql
/**[test]

[[test.results]]
error = ""The index 'userEmailIndex' does not exist""

*/

REBUILD INDEX userEmailIndex ON user;
```


## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define an index only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining a index in SurrealDB if you want to ensure that the index is only created if it does not already exist. If the index already exists, the `DEFINE INDEX` statement will return an error.

It's particularly useful when you want to safely attempt to define a index without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the index definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a index and overwrite an existing one if it already exists, ensuring that the latest version of the index definition is always in use

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a INDEX if it does not already exist
DEFINE INDEX IF NOT EXISTS example ON example FIELDS example;
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to define an index and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing index definition. If the index already exists, the `DEFINE INDEX` statement will overwrite the existing definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create an INDEX and overwrite if it already exists
DEFINE INDEX OVERWRITE example ON example FIELDS example;
```

## Using `CONCURRENTLY` clause

<Since v="v2.0.0" />

Building indexes can be lengthy and may time out before they're completed. Use the `CONCURRENTLY` option to build the index without blocking operations. The statement will return immediately, allowing you to monitor the index-building progress by executing the [INFO](/docs/surrealql/statements/info) statement.

```surql
-- Create an INDEX concurrently
DEFINE INDEX test ON user FIELDS email CONCURRENTLY;
INFO FOR INDEX test ON user;
INFO FOR INDEX test ON user;
```

When building an index concurrently, SurrealDB starts the index creation as a background process. You can monitor the status of this process using the `INFO FOR INDEX` statement. The output includes a building block that provides several key details:

```surql
-- Check the indexing status
INFO FOR INDEX test ON user;
```

```surql title="Possible response"
-- Query

{
    building:  {
        initial: 8143,
        pending: 19,
        status: 'indexing',
        updated: 80
    }
}
```
The indexing process consists of two stages: **initial** and **update**.

1. **Initial Stage:**

   During this stage, SurrealDB indexes all existing records. The number of indexed records is represented by the `initial` property. While this stage is in progress, any new inserts, updates, or deletions are tracked as `pending`.

2. **Update Stage:**

   Once the initial stage is completed, SurrealDB begins indexing the pending records accumulated during the initial phase. At this point:

   - The `initial` count remains stable.
   - The `pending` count should gradually decrease as these records are processed; however, it may temporarily increase if new modifications occur during indexing.
   - The `updated` property indicates the number of pending records that have been indexed during this stage.

When both stages are complete, the index status changes to **ready**, meaning that the index is now automatically updated within the same transaction that inserts, updates, or deletes records.

```surql
-- Query

{
	building: {
		status: 'ready'
	}
}
```

## Using the `ANY`/`ALL` operators for string indexes

<Since v="v2.4.0" />

An index defined on a string value can be used via the operators `CONTAINSANY`, `ALLINSIDE`, or `ANYINSIDE`. The operator `CONTAINS`, however, will not use a defined index as `CONTAINS` is used for substring matches between strings themselves as opposed to an index lookup.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: account:billy, name: 'Billy McConnell' }]"

[[test.results]]
value = "[{ id: account:billy, name: 'Billy McConnell' }]"

[[test.results]]
value = "[{ id: account:billy, name: 'Billy McConnell' }]"

[[test.results]]
value = "[{ id: account:billy, name: 'Billy McConnell' }], [{ detail: { direction: 'forward', table: 'account' }, operation: 'Iterate Table' }, { detail: { type: 'Memory' }, operation: 'Collector' }, { detail: { type: 'KeysAndValues' }, operation: 'RecordStrategy' }, { detail: { count: 1 }, operation: 'Fetch' }]"

[[test.results]]
value = "[{ detail: { plan: { index: 'name_index', operator: 'union', value: ['Billy McConnell'] }, table: 'account' }, operation: 'Iterate Index' }, { detail: { type: 'Memory' }, operation: 'Collector' }, { detail: { type: 'KeysAndValues' }, operation: 'RecordStrategy' }, { detail: { count: 1 }, operation: 'Fetch' }]"

*/

DEFINE FIELD name ON account TYPE string;
DEFINE INDEX name_index ON account FIELDS name;

CREATE account:billy SET name = "Billy McConnell";

-- Both return the user Billy McConnell
SELECT * FROM account WHERE name CONTAINS "Billy McConnell";
SELECT * FROM account WHERE name CONTAINSANY ["Billy McConnell"];

-- However, CONTAINS does not use the index
SELECT * FROM account WHERE name CONTAINS "Billy McConnell" EXPLAIN FULL;
-- CONTAINSANY + putting the value inside an array will use the index
SELECT * FROM account WHERE name CONTAINSANY ["Billy McConnell"] EXPLAIN FULL;
```

## The `DEFER` clause

<Since v="v2.5.0" />

Index updates in SurrealDB occur synchronously during document operations. This ensures **immediate consistency**, in which all reads return the most recent write. However, this can become a bottleneck during high-volume parallel ingestion, leading to write-write conflicts and increased latency, particularly with Full-Text or Vector indexes.

The `DEFER` clause can be used in this case if **eventual consistency** is acceptable, namely a setting in which reads may return stale data for a short period, but will eventually converge to the most recent write. An index with this clause will be enqueued in a persistent background queue so that ingestion and indexing are decoupled.

```surql
DEFINE ANALYZER simple TOKENIZERS blank,class FILTERS lowercase;
DEFINE INDEX title_index ON blog FIELDS title SEARCH ANALYZER simple BM25(1.2,0.75) HIGHLIGHTS DEFER;
```

Note: As unique indexes offer a guarantee that no records that contravene the index will ever exist, the `UNIQUE` clause cannot be used together with `DEFER`.

## Performance Implications

When defining indexes, it's essential to consider the fields most frequently queried or used to optimize performance.

Indexes may improve the performance of SurrealQL statements. This may not be noticeable with small tables but it can be significant for large tables; especially when the indexed fields are used in the `WHERE` clause of a [`SELECT`](/docs/surrealql/statements/insert) statement.

Indexes can also impact the performance of write operations ([INSERT](/docs/surrealql/statements/insert), [UPDATE](/docs/surrealql/statements/update), [DELETE](/docs/surrealql/statements/delete)) since the index needs to be updated accordingly. Therefore, it's essential to balance the need for read performance with write performance.


## module.mdx

---
sidebar_position: 11
sidebar_label: DEFINE MODULE
title: DEFINE MODULE statement | SurrealQL
description: Just like in other databases, SurrealDB uses indexes to help optimize query performance. An index can consist of one or more fields in a table and can enforce a uniqueness constraint.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'



# `DEFINE MODULE` statement

<Since v="v3.0.0" />

A `DEFINE MODULE` statement is used to define a module via which [Surrealism](/docs/surrealdb/extensions) extensions functions can be called.

> [!NOTE]
> The [`surrealism` experimental](/docs/surrealdb/cli/module) feature must be enabled before you can use a `DEFINE MODULE` statement.

## Statement syntax

<Tabs syncKey="define-module-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE MODULE [ OVERWRITE | IF NOT EXISTS ] @mod::@sub AS @file_name
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineModuleAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "MODULE" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@mod::@sub" },
      { type: "Terminal", text: "AS" },
      { type: "NonTerminal", text: "@file_name" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};


<RailroadDiagram ast={defineModuleAst} className="my-6" />

  </TabItem>
</Tabs>

## Example

A module includes a module and a sub, followed by `AS` and a pointer to the `.surli` file containing the Rust code compiled to WASM through the Surrealism CLI.

```surql
DEFINE MODULE mod::test AS f"test:/demo.surli";
```

Once the module is defined, functions can be accessed through this path.

Assuming these two functions in the Rust code before compilation to WASM via the Surrealism CLI:

```rust
#[surrealism]
fn returns_true() -> bool { true };

#[surrealism]
fn check_num_size(num: i32) -> Result<i32, &'static str> {
    if num >= 500 {
        Err("Number is too big!")
    } else {
        Ok(num)
    }
}
```

They will then be accessible using the following paths.

```surql
RETURN mod::test::returns_true();
-- true

RETURN mod::test::check_num_size(100);
-- 100
```

## namespace.mdx

---
sidebar_position: 12
sidebar_label: DEFINE NAMESPACE
title: DEFINE NAMESPACE statement | SurrealQL
description: The DEFINE NAMESPACE statement can be used to setup namespaces, which can contain multiple databases.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `DEFINE NAMESPACE` statement

SurrealDB has a multi-tenancy model which allows you to scope databases to a namespace. There is no limit to the number of databases that can be in a namespace, nor is there a limit to the number of namespaces allowed. Only users with root access are authorized to create namespaces.

Let's say that you're using SurrealDB to create a multi-tenant SaaS application. You can guarantee that the data of each tenant will be kept separate from other tenants if you put each tenant's databases into separate namespaces. In other words, this will ensure that information will remain siloed so user will only have access the information in the namespace they are a member of.

## Requirements

- You must be authenticated as a root owner or editor before you can use the `DEFINE NAMESPACE` statement.

## Statement syntax

<Tabs syncKey="define-namespace-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE NAMESPACE [ OVERWRITE | IF NOT EXISTS ] @name [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineNamespaceAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "NAMESPACE" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineNamespaceAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage
Below shows how you can create a namespace using the `DEFINE NAMESPACE` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Namespace for Abcum Ltd.
DEFINE NAMESPACE abcum;
```

## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define a namespace only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining a namespace in SurrealDB if you want to ensure that the namespace is only created if it does not already exist. If the namespace already exists, the `DEFINE NAMESPACE` statement will return an error.

It's particularly useful when you want to safely attempt to define a namespace without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the namespace definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a namespace and overwrite an existing one if it already exists, ensuring that the latest version of the namespace definition is always in use

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a NAMESPACE if it does not already exist
DEFINE NAMESPACE IF NOT EXISTS example;
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to define a namespace and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing namespace definition. If the namespace already exists, the `DEFINE NAMESPACE` statement will overwrite the existing namespace definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create an NAMESPACE and overwrite if it already exists
DEFINE NAMESPACE OVERWRITE example;
```


## param.mdx

---
sidebar_position: 13
sidebar_label: DEFINE PARAM
title: DEFINE PARAM statement | SurrealQL
description: The DEFINE PARAM statement allows you to define global (database-wide) parameters that are available to every client.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `DEFINE PARAM` statement

The `DEFINE PARAM` statement allows you to define global (database-wide) parameters that are available to every client.

## Requirements

- You must be authenticated as a root owner or editor, namespace owner or editor, or database owner or editor before you can use the `DEFINE PARAM` statement.
- [You must select your namespace and database](/docs/surrealql/statements/use) before you can use the `DEFINE PARAM` statement.

## Statement syntax

<Tabs syncKey="define-param-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE PARAM [ OVERWRITE | IF NOT EXISTS ] $@name 
    VALUE @value
    [ COMMENT @string ]
    [ PERMISSIONS [ NONE | FULL | WHERE @condition ] ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineParamAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "PARAM" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "Terminal", text: "$" },
      { type: "NonTerminal", text: "@name" },
      { type: "Terminal", text: "VALUE" },
      { type: "NonTerminal", text: "@value" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "PERMISSIONS" }, { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NONE" }, { type: "Terminal", text: "FULL" }, { type: "Sequence", children: [ { type: "Terminal", text: "WHERE" }, { type: "NonTerminal", text: "@condition" } ] } ] } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineParamAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage
Below shows how you can create a parameter using the `DEFINE PARAM` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE PARAM $endpointBase VALUE "https://dummyjson.com";
```

Then, simply use the global parameter like you would with any variable.

```surql
RETURN http::get($endpointBase + "/products");
```

## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define a param only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining a param in SurrealDB if you want to ensure that the param is only created if it does not already exist. If the param already exists, the `DEFINE PARAM` statement will return an error.

It's particularly useful when you want to safely attempt to define a param without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the param definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a param and overwrite an existing one if it already exists, ensuring that the latest version of the param definition is always in use

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a PARAM if it does not already exist
DEFINE PARAM IF NOT EXISTS $example VALUE 123;
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to define a param and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing param definition. If the param already exists, the `DEFINE PARAM` statement will overwrite the existing param definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create an PARAM and overwrite if it already exists
DEFINE PARAM OVERWRITE $example VALUE 123;
```


## scope.mdx

---
sidebar_position: 14
sidebar_label: DEFINE SCOPE
title: DEFINE SCOPE statement | SurrealQL
description: Setting scope access allows SurrealDB to operate as a web database. With scopes you can set authentication and access rules which enable fine-grained access to tables and fields.
---

> [!WARNING]
> This statement was deprecated in favour of `DEFINE ACCESS ... TYPE RECORD` in SurrealDB versions 2.x, and has been removed as of SurrealDB 3.0. Learn more in the [DEFINE ACCESS](/docs/surrealql/statements/define/access).

import Since from '@components/shared/Since.astro'

# `DEFINE SCOPE` statement

Setting scope access allows SurrealDB to operate as a web database. With scopes you can set authentication and access rules which enable fine-grained access to tables and fields.

## Requirements

- You must be authenticated as a root or namespace user before you can use the `DEFINE SCOPE` statement.
- [You must select your namespace and database](/docs/surrealql/statements/use) before you can use the `DEFINE SCOPE` statement.

## Statement syntax

```syntax title="SurrealQL Syntax"
DEFINE SCOPE [ OVERWRITE | IF NOT EXISTS ] @name SESSION @duration SIGNUP @expression SIGNIN @expression [ COMMENT @string ]
```

## Example usage
Below shows how you can create a scope using the `DEFINE SCOPE` statement.

```surql
-- Enable scope authentication directly in SurrealDB
DEFINE SCOPE account SESSION 24h
	SIGNUP ( CREATE user SET email = $email, pass = crypto::argon2::generate($pass) )
	SIGNIN ( SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass) )
;
```

## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define a scope only if it does not already exist. If the scope already exists, the `DEFINE SCOPE` statement will return an error.

```surql
-- Create a SCOPE if it does not already exist
DEFINE SCOPE IF NOT EXISTS example;
```


## sequence.mdx

---
sidebar_position: 15
sidebar_label: DEFINE SEQUENCE
title: DEFINE SEQUENCE statement | SurrealQL
description: A DEFINE SEQUENCE statement defines a distributed generator of monotonically increasing numeric sequences.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `DEFINE SEQUENCE` statement

<Since v="v3.0.0" />

A sequence is used to generate reliable, monotonically increasing numeric sequences in both single-node and clustered SurrealDB deployments (multiple compute nodes backed by TiKV). It uses a batch-allocation strategy to minimise coordination while guaranteeing global uniqueness.

The key features of a sequence are as follows:

* Batch allocation: Nodes request ranges of sequence values at once, reducing network chatter and coordination overhead.
* Node ownership tagging: Every batch is tagged with the requesting node's UUID to prevent overlap between nodes.
* Durable Persistence: Sequence metadata is stored in the underlying key-value store to survive restarts and network partitions.
* Concurrent, thread-safe access: A DashMap caches active sequences, allowing lock-free reads on the hot path.
* Exponential back-off with full jitter: When a batch-allocation attempt fails, the node retries with an exponential delay that includes full jitter to avoid thundering-herd effects across the cluster.
* Automatic cleanup: Listens for namespace and database-removal events and purges the corresponding sequence state.

The sequence implementation avoids contention by having each node reserve a range of sequence values, allowing it to serve multiple requests locally without requiring distributed coordination for every request. When a node exhausts its allocated range, it acquires a new batch from the distributed store.

## Statement syntax

<Tabs syncKey="define-sequence-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE SEQUENCE [ OVERWRITE | IF NOT EXISTS ] @name [ BATCH @batch ] [ START @start ] [ TIMEOUT @duration ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineSequenceAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "SEQUENCE" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "BATCH" }, { type: "NonTerminal", text: "@batch" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "START" }, { type: "NonTerminal", text: "@start" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "TIMEOUT" }, { type: "NonTerminal", text: "@duration" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineSequenceAst} className="my-6" />

  </TabItem>
</Tabs>

## Examples

A sequence can be created with nothing more than a name.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE SEQUENCE mySeq;
```

The `BATCH`, `START`, and `TIMEOUT` clauses can be included to configure the sequence.

```surql
DEFINE SEQUENCE mySeq2 BATCH 1000 START 100 TIMEOUT 5s;
sequence::nextval('mySeq2');
-- Output: 100

DEFINE SEQUENCE mySeq3 BATCH 1000 START 100 TIMEOUT 0ns;
sequence::nextval('mySeq3');
-- Possible output: 'The query was not executed because it exceeded the timeout'
```

Sequences are never rolled back, even in a failed transaction. This differs from an approach like a single record with a manually incrementing value.

```surql
DEFINE SEQUENCE seq;
CREATE my:counter SET val = 0;
sequence::nextval("seq");        -- 0
my:counter.val;                  -- 0

BEGIN TRANSACTION;
sequence::nextval("seq");        -- 0
UPDATE my:counter SET val += 1;  -- my:counter.val = 1
CANCEL TRANSACTION;              -- my:counter.val now rolled back to 0

sequence::nextval("seq");        -- 2
UPDATE my:counter SET val += 1;  -- 1
```

## See also

* [Sequence functions](/docs/surrealql/functions/database/sequence)

## table.mdx

---
sidebar_position: 16
sidebar_label: DEFINE TABLE
title: DEFINE TABLE statement | SurrealQL
description: The DEFINE TABLE statement allows you to declare your table by name, enabling you to apply strict controls to a table's schema and access permissions.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `DEFINE TABLE` statement

The `DEFINE TABLE` statement allows you to declare your table by name, enabling you to apply strict controls to a table's schema by making it `SCHEMAFULL`, create a foreign table view, and set permissions specifying what operations can be performed on the table.

> [!NOTE]
> The fields of a table are not defined using `DEFINE TABLE`, but via individual [`DEFINE FIELD`](/docs/surrealql/statements/define/field) statements.

## Requirements

- You must be authenticated as a root owner or editor, namespace owner or editor, or database owner or editor before you can use the `DEFINE TABLE` statement.
- [You must select your namespace and database](/docs/surrealql/statements/use) before you can use the `DEFINE TABLE` statement.

## Statement syntax

<Tabs syncKey="define-table-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE TABLE [ OVERWRITE | IF NOT EXISTS ] @name
	[ DROP ]
	[ SCHEMAFULL | SCHEMALESS ]
	[ TYPE [ ANY | NORMAL | RELATION [ IN | FROM ] @table [ OUT | TO ] @table [ ENFORCED ]]]
	[ AS SELECT @projections
		FROM @tables
		[ WHERE @condition ]
		[ GROUP [ BY @groups | ALL ] ]
	]
	[ CHANGEFEED @duration [ INCLUDE ORIGINAL ] ]
	[ PERMISSIONS [ NONE | FULL
		| FOR select @expression
		| FOR create @expression
		| FOR update @expression
		| FOR delete @expression
	] ]
    [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineTableAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "TABLE" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Optional", child: { type: "Terminal", text: "DROP" } },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "SCHEMAFULL" }, { type: "Terminal", text: "SCHEMALESS" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [
        { type: "Terminal", text: "TYPE" },
        { type: "Choice", index: 1, children: [
          { type: "Terminal", text: "ANY" },
          { type: "Terminal", text: "NORMAL" },
          { type: "Sequence", children: [
            { type: "Terminal", text: "RELATION" },
            { type: "Optional", child: { type: "Sequence", children: [ { type: "Choice", index: 1, children: [ { type: "Terminal", text: "IN" }, { type: "Terminal", text: "FROM" } ] }, { type: "NonTerminal", text: "@table" } ] } },
            { type: "Optional", child: { type: "Sequence", children: [ { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OUT" }, { type: "Terminal", text: "TO" } ] }, { type: "NonTerminal", text: "@table" } ] } },
            { type: "Optional", child: { type: "Terminal", text: "ENFORCED" } }
          ] }
        ] }
      ] } },
      { type: "Optional", child: { type: "Sequence", children: [
        { type: "Terminal", text: "AS" }, { type: "Terminal", text: "SELECT" }, { type: "NonTerminal", text: "@projections" },
        { type: "Terminal", text: "FROM" }, { type: "NonTerminal", text: "@tables" },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "WHERE" }, { type: "NonTerminal", text: "@condition" } ] } },
        { type: "Optional", child: { type: "Sequence", children: [
          { type: "Terminal", text: "GROUP" },
          { type: "Choice", index: 1, children: [
            { type: "Sequence", children: [ { type: "Terminal", text: "BY" }, { type: "NonTerminal", text: "@groups" } ] },
            { type: "Terminal", text: "ALL" }
          ] }
        ] } }
      ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "CHANGEFEED" }, { type: "NonTerminal", text: "@duration" }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "INCLUDE" }, { type: "Terminal", text: "ORIGINAL" } ] } } ] } },
      { type: "Optional", child: { type: "Sequence", children: [
        { type: "Terminal", text: "PERMISSIONS" },
        { type: "Choice", index: 1, children: [
          { type: "Terminal", text: "NONE" },
          { type: "Terminal", text: "FULL" },
          { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "select" }, { type: "NonTerminal", text: "@expression" } ] },
          { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "create" }, { type: "NonTerminal", text: "@expression" } ] },
          { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "update" }, { type: "NonTerminal", text: "@expression" } ] },
          { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "delete" }, { type: "NonTerminal", text: "@expression" } ] }
        ] }
      ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineTableAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

Below shows how you can create a table using the `DEFINE TABLE` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Declare the name of a table.
DEFINE TABLE reading;
```

The following example uses the `DROP` portion of the `DEFINE TABLE` statement. Marking a table as `DROP` disallows creating or updating records.

`DROP` tables are useful in combination with events or foreign (view) tables, as you can compute a record and essentially drop the input.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- By marking a table as DROP, you disallow any records to be created or updated.
-- Records that currently exist in the table will not automatically be deleted, you can still remove them manually.
DEFINE TABLE reading DROP;
```

The following expression shows how you can define a `CHANGEFEED` for a table. After creating, updating, and deleting records in the table as usual, using `SHOW CHANGES FOR` will show the changes that have taken place during this time.

```surql
-- Define the change feed and its duration
-- Optionally, append INCLUDE ORIGINAL to include info
-- on the current record before a change took place
DEFINE TABLE reading CHANGEFEED 3d;

-- Create some records in the reading table
CREATE reading SET story = "Once upon a time";
CREATE reading SET story = "there was a database";

-- Replay changes to the reading table since a certain date
-- Must be after the timestamp at which the changefeed began
SHOW CHANGES FOR TABLE reading SINCE d"2023-09-07T01:23:52Z" LIMIT 10;

-- Alternatively, show the changes for the table since a version number
SHOW CHANGES FOR TABLE reading SINCE 0 LIMIT 10;
```

```surql title="Response without INCLUDE ORIGINAL"
[
    {
        "changes": [
            {
                "define_table": {
                    "name": "reading"
                }
            }
        ],
        "versionstamp": 29
    },
    {
        "changes": [
            {
                "update": {
                    "id": "reading:h1gcbc7ykbpslellh2g2",
                    "story": "Once upon a time"
                }
            }
        ],
        "versionstamp": 30
    },
    {
        "changes": [
            {
                "update": {
                    "id": "reading:l9qfcncklhnlklby1avf",
                    "story": "there was a database"
                }
            }
        ],
        "versionstamp": 31
    }
]
```

```surql title="Response with INCLUDE ORIGINAL"
[
    {
        "changes": [
            {
                "define_table": {
                    "name": "reading"
                }
            }
        ],
        "versionstamp": 29
    },
    {
        "changes": [
            {
                "current": {
                    "id": "reading:2j3rc2yw1jzspcuvfe9v",
                    "story": "Once upon a time"
                },
                "update": [
                    {
                        "op": "replace",
                        "path": "/",
                        "value": null
                    }
                ]
            }
        ],
        "versionstamp": 30
    },
    {
        "changes": [
            {
                "current": {
                    "id": "reading:iuiurhi0y2ka0by0skqi",
                    "story": "there was a database"
                },
                "update": [
                    {
                        "op": "replace",
                        "path": "/",
                        "value": null
                    }
                ]
            }
        ],
        "versionstamp": 31
    }
]
```

## Schemafull tables

The following example demonstrates the `SCHEMAFULL` portion of the `DEFINE TABLE` statement. When a table is defined as schemafull, the database strictly enforces any schema definitions that are specified using the `DEFINE TABLE` statement. New fields can not be added to a `SCHEMAFULL` table unless they are defined via the [`DEFINE FIELD`](/docs/surrealql/statements/define/field) statement.


> [!NOTE]
> Since `v2.0.0`, schemafull tables are implicitly type [`NORMAL`](/docs/surrealql/statements/define/table#table-with-specialized-type-clause) tables by default.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ email: 'Tobie.Hitchcock@surrealdb.com', firstName: 'Tobie', id: user:9hizvimsgva1rosul33j, lastName: 'Hitchcock' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ email: 'Tobie.Hitchcock@surrealdb.com', firstName: 'Tobie', id: user:9hizvimsgva1rosul33j, lastName: 'Hitchcock' }]"
skip-record-id-key = true

*/

-- Create schemafull user table.
DEFINE TABLE user SCHEMAFULL;

-- Define some fields.
DEFINE FIELD firstName ON TABLE user TYPE string;
DEFINE FIELD lastName ON TABLE user TYPE string;
DEFINE FIELD email ON TABLE user TYPE string
  ASSERT string::is_email($value);
DEFINE INDEX userEmailIndex ON TABLE user COLUMNS email UNIQUE;

-- SEE IT IN ACTION
-- 1: Add a user with all required fields and an undefined one, 'photoURI'.
CREATE user CONTENT {
    firstName: 'Tobie',
    lastName: 'Hitchcock',
    email: 'Tobie.Hitchcock@surrealdb.com',
    photoURI: 'photo/yxCFi22Jw2.webp'
};
-- 2: Statement will not fail but photoURI will be ignored as it is not a
--    defined field.

-- 3: Query the data
SELECT * FROM user;
```

## Schemaless tables

The following example demonstrates the `SCHEMALESS` portion of the `DEFINE TABLE` statement. This allows you to explicitly state that the specified table has no schema.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ email: 'Tobie.Hitchcock@surrealdb.com', firstName: 'Tobie', id: user:tobie, lastName: 'Hitchcock', photoURI: 'photo/yxCFi22Jw2.webp' }]"

[[test.results]]
error = ""Found 'Jamie.Hitchcock' for field `email`, with record `user:jaime`, but field must conform to: string::is_email($value)""

*/

-- Create schemaless user table.
DEFINE TABLE user SCHEMALESS;

-- Define some fields.
DEFINE FIELD firstName ON TABLE user TYPE string;
DEFINE FIELD lastName ON TABLE user TYPE string;
DEFINE FIELD email ON TABLE user TYPE string
  ASSERT string::is_email($value);
DEFINE INDEX userEmailIndex ON TABLE user COLUMNS email UNIQUE;

-- SEE IT IN ACTION - Example 1
-- 1: Add a user with all required fields and an undefined one.
CREATE user:tobie SET firstName = 'Tobie', lastName = 'Hitchcock', email = 'Tobie.Hitchcock@surrealdb.com', photoURI = 'photo/yxCFi22Jw2.webp';
-- 2: Statement will succeed because user is a SCHEMALESS table.

-- SEE IT IN ACTION - Example 2
-- 1: Add a user with an invalid email address and include a new field that was never defined.
CREATE user:jaime SET firstName = 'Jamie', lastName = 'Hitchcock', email = 'Jamie.Hitchcock', photoURI = 'photo/yxCFi22Jw2.webp';
-- 2: Statement will fail because the value for email was not valid.
```

## Interaction between fields

While a `DEFINE TABLE` statement represents a template for any subsequent records to be created, a `DEFINE FIELD` statement pertains to concrete field data of a record. As such, a `DEFINE FIELD` statement gives access to the record's other fields through their names, as well as the current field through the [`$value`](/docs/surrealql/parameters#value) parameter.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ first_name: 'Bob', id: person:5o2zt3el25z7kbuzsf47, last_name: 'Bobson', name: 'Bob Bobson' }]"
skip-record-id-key = true

*/

DEFINE TABLE person SCHEMAFULL;

DEFINE FIELD first_name ON TABLE person TYPE string ASSERT string::len($value) < 20;
DEFINE FIELD last_name  ON TABLE person TYPE string ASSERT string::len($value) < 20;
DEFINE FIELD name       ON TABLE person             VALUE first_name + ' ' + last_name;

// Creates a `person` with the name "Bob Bobson"
CREATE person SET first_name = "Bob", last_name = "Bobson";
```

## Pre-computed table views

In SurrealDB, like in other databases, you can create views. The way you create views is using the `DEFINE TABLE` statement like you would for any other table, then adding the `AS` clause at the end with your `SELECT` query.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[]"

*/

DEFINE TABLE review DROP;
-- Define a table as a view which aggregates data from the review table
DEFINE TABLE avg_product_review TYPE NORMAL AS
SELECT
	count() AS number_of_reviews,
	math::mean(<float> rating) AS avg_review,
	->product.id AS product_id,
	->product.name AS product_name
FROM review
GROUP BY product_id, product_name;

-- Query the projection
SELECT * FROM avg_product_review;
```

There are a few important things which make our views far more powerful than a typical relational database view and a few limitations to keep in mind.

Starting with what makes them powerful. Our pre-computed table views are most similar to event-based, incrementally updating, materialised views. Let's explain what that means.

- Event-based, meaning that when you run add or remove data from the underlying table, in our example, the `review` table, it triggers a matching event on the `avg_product_review` table view.
- Materialised view, meaning that the first time we run the table view query, it will run the query like a normal `SELECT` statement, but then materialise the result. Instead of normal views which behave like bookmarked `SELECT` queries, that just look like tables to the user.
- Incrementally updating, meaning that for any subsequent run, it will listen for the event trigger and perform the most efficient operation possible to always keep the result up to date, instead of just running the `SELECT` statement again.

While this functionality can be replicated in many other databases, it is usually only done by expert users as it can be very complicated to set up and maintain. Therefore, the true power of our pre-computed table views is making this advanced functionality accessible to everyone.

As mentioned though, there are a few limitations to keep in mind.

- First, while subsequent runs are very efficient, the initial run of large analytical queries can be slow and use a lot of resources, because its just a normal `SELECT` statement. Therefore indexing and query optimisation are still very important.
- Second, while both graph relations and record links are supported, the table view update event, only gets triggered based on the table we have in our `FROM` clause. In our case, just the `review` table, not the `product` we are also using in the query. Meaning that if you delete a `review` the `avg_product_review`  will reflect that in near real-time. However if you delete a `product`, it will still show up in `avg_product_review`.

Also note that table views are not triggered when importing data.

## Defining permissions

By default, the permissions on a table will be set to NONE unless otherwise specified.

```surql
/**[test]

[[test.results]]
value = "[{ id: some_table:8o6rh2r3ovpmu07elc3r }]"

[[test.results]]
value = "NONE"

[[test.results]]
value = "{ accesses: {  }, analyzers: {  }, apis: {  }, buckets: {  }, configs: {  }, functions: {  }, models: {  }, params: {  }, sequences: {  }, tables: { some_other_table: 'DEFINE TABLE some_other_table TYPE ANY SCHEMALESS PERMISSIONS NONE', some_table: 'DEFINE TABLE some_table TYPE ANY SCHEMALESS PERMISSIONS NONE' }, users: {  } }"

*/

CREATE some_table;
DEFINE TABLE some_other_table;

INFO FOR DB;
```

```surql title="Response"
{
	analyzers: {},
	functions: {},
	models: {},
	params: {},
	scopes: {},
	tables: {
		some_other_table: 'DEFINE TABLE some_other_table TYPE ANY SCHEMALESS PERMISSIONS NONE',
		some_table: 'DEFINE TABLE some_table TYPE ANY SCHEMALESS PERMISSIONS NONE'
	},
	tokens: {},
	users: {}
}
```

The following shows how to set table level `PERMISSIONS` using the `DEFINE TABLE` statement. This allows you to set independent permissions for selecting, creating, updating, and deleting data.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Specify access permissions for the 'post' table
DEFINE TABLE post SCHEMALESS
	PERMISSIONS
		FOR select
			-- Published posts can be selected
			WHERE published = true
			-- A user can select all their own posts
			OR user = $auth.id
		FOR create, update
			-- A user can create or update their own posts
			WHERE user = $auth.id
		FOR delete
			-- A user can delete their own posts
			WHERE user = $auth.id
			-- Or an admin can delete any posts
			OR $auth.admin = true
;
```

## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define a table only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining a table in SurrealDB if you want to ensure that the table is only created if it does not already exist. If the table already exists, the `DEFINE TABLE` statement will return an error.

It's particularly useful when you want to safely attempt to define a table without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the table definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a table and overwrite an existing one if it already exists, ensuring that the latest version of the table definition is always in use

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a TABLE if it does not already exist
DEFINE TABLE IF NOT EXISTS reading;
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to define a table and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing table definition. If the table already exists, the `DEFINE TABLE` statement will overwrite the existing table definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create an TABLE and overwrite if it already exists
DEFINE TABLE OVERWRITE example;
```

## Table with specialized `TYPE`-clause

<Since v="v1.4.0" />

When defining a table in SurrealDB, you can specify the type of data that can be stored in the table. This can be done using the `TYPE` clause, followed by either `ANY`, `NORMAL`, or `RELATION`.

With `TYPE ANY`, you can specify a table to store any type of data, whether it's a normal record or a relational record.

With `TYPE NORMAL`, you can specify a table to only store "normal" records, and not relations. When a table is defined as `TYPE NORMAL`, it will not be able to store relations this can be useful when you want to restrict the type of data that can be stored in a table in schemafull mode.

Finally, with `TYPE RELATION`, you can specify a table to only store relational type content. This can be useful when you want to restrict the type of data that can be stored in a table.

```surql
DEFINE TABLE person TYPE ANY;
DEFINE TABLE person;
```

With `TYPE NORMAL`, you can specify a table to only store "normal" records, and not relations.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Since it's default, we can also omit the TYPE clause
DEFINE TABLE person TYPE NORMAL;
```

With `TYPE RELATION`, you can specify a table to only store relational type content, and restrict what kind of relations can be stored.

```surql
-- Just a RELATION table, no constraints on the type of table
DEFINE TABLE likes TYPE RELATION;

-- Define a relation table, and constrain the type of relation which can be stored
DEFINE TABLE likes TYPE RELATION FROM user TO post;
-- OR use IN and OUT alternatively to FROM and TO
DEFINE TABLE likes TYPE RELATION IN user OUT post;
-- To allow a link to one of a possible set of record types, use the | operator
DEFINE TABLE likes TYPE RELATION FROM user TO post|video;
DEFINE TABLE likes TYPE RELATION IN user OUT post|video;
```

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Define a relation table, and constrain the type of relation which can be stored
DEFINE TABLE assigned_to SCHEMAFULL TYPE RELATION IN tag OUT sticky
    PERMISSIONS
        FOR create, select, update, delete
            WHERE in.owner == $auth.id AND out.author == $auth.id;
```

## Using ENFORCED to ensure that related records exist

<Since v="v2.0.0" />

As relations are represented by standalone tables, they can be constructed before any linked records exist.

```surql
/**[test]

[[test.results]]
value = "[{ distance: 12.4f, id: road_to:jd2f9pjzecb0ihmo2mbp, in: city:one, out: city:two, slope: 5.4f }]"
skip-record-id-key = true

*/

RELATE city:one->road_to->city:two SET
    distance = 12.4,
    slope = 5.4;
```

```surql title="Output"
[
	{
		distance: 12.4f,
		id: road_to:pacwucj25a056hhs2s5h,
		in: city:one,
		out: city:two,
		slope: 5.4f
	}
]
```

As such, a query on the relation will return nothing until the records it has been defined upon are created.

```surql
SELECT ->road_to->city FROM city;

CREATE city:one, city:two;
SELECT ->road_to->city FROM city;
```

```surql title="Output"
-------- Query --------

[]

-------- Query --------

[
	{
		"->road_to": {
			"->city": [
				city:two
			]
		}
	},
	{
		"->road_to": {
			"->city": []
		}
	}
]
```

If this behaviour is not desirable, the `ENFORCED` clause can be used on a table of `TYPE RELATION` to disallow a `RELATE` statement from working unless it points to existing data.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
error = ""The record 'city:one' does not exist""

*/

DEFINE TABLE road_to TYPE RELATION IN city OUT city ENFORCED;

RELATE city:one->road_to->city:three SET
    distance = 5.5,
    slope = 30.0;
```

```surql title="Output"
"The record 'city:one' does not exist"
```

## Inserting data from undefined fields on a `SCHEMAFULL` table

<Since v="v3.0.0" />

Previously, an insert into a `SCHEMAFULL` table would work even if extra data was present. The query below shows this behaviour, in which the data inside `unneeded_data` is simply filtered out.

```surql
DEFINE TABLE user SCHEMAFULL;
DEFINE FIELD name ON user TYPE string;
DEFINE FIELD user_num ON user TYPE int;

CREATE ONLY user CONTENT { 
    name: "Billy", 
    user_num: 100,
    unneeded_data: {
        some: "other",
        needless: "data"
    }
};
```

```surql title="Output"
{
    id: user:r38bg4fnp9nurksd06nl,
    name: 'Billy',
    user_num: 100
}
```

While convenient, this led to the possibility of a typo leading to unexpected results.

```surql
DEFINE TABLE user SCHEMAFULL;
DEFINE FIELD name ON user TYPE string;
DEFINE FIELD user_num ON user TYPE option<int>;

CREATE ONLY user CONTENT { 
    name: "Billy", 
    User_num: 100 // Note: field is capitalized
};
```

The output shows that the `user` lacks a value for `user_num` even though the query above intended to provide this data.

```surql
{
	id: user:jn4762t7oyure86fe3qk,
	name: 'Billy'
}
```

The same query in SurrealDB 3.0 now returns an error.

```surql
"Found field 'User_num', but no such field exists for table 'user'"
```

To avoid an error when working with data that contains unneeded fields, use `.{}` (the destructuring operator) to pass on only the necessary fields.

```surql
DEFINE TABLE user SCHEMAFULL;
DEFINE FIELD name ON user TYPE string;
DEFINE FIELD user_num ON user TYPE int;

CREATE ONLY user CONTENT { 
    name: "Billy", 
    user_num: 100,
    unneeded_data: {
        some: "other",
        needless: "data"
    }
}.{
    -- Only pass on the name and user_num data
    name,
    user_num
};
```

## token.mdx

---
sidebar_position: 17
sidebar_label: DEFINE TOKEN
title: DEFINE TOKEN statement | SurrealQL
description: SurrealDB can work with third-party authentication providers such as OpenID Connect providers, OAuth providers and other trusted third parties.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'


> [!WARNING]
> This statement was deprecated in favour of [`DEFINE ACCESS ... TYPE JWT`](/docs/surrealql/statements/define/access/jwt) and [`DEFINE ACCESS ... TYPE RECORD ... WITH JWT`](/docs/surrealql/statements/define/access/record) in SurrealDB versions 2.x, and has been removed as of SurrealDB 3.0. Learn more in [define access documentation](/docs/surrealql/statements/define/access).

# `DEFINE TOKEN` statement

SurrealDB can work with third-party authentication providers such as OpenID Connect providers, OAuth providers and other trusted parties providing JWT (JSON Web Tokens, also referred to in this page as “tokens”). Let's say that your provider issues your client (e.g. a user or a service) a JWT once it has authenticated. By using the DEFINE TOKEN statement, you can set the public key or shared secret that will be used to verify the authenticity of the token.

This verification is performed automatically by SurrealDB when provided with a JWT through any of its interfaces (i.e. the [HTTP REST API](/docs/surrealdb/integration/http) through the “Authorization” header or [any of the SDKs](/docs/surrealdb/integration/sdks) through the “Authenticate” methods) before trusting the claims contained in the token and allowing SurrealQL queries to access the values of those claims.

<Tabs syncKey="define-token-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE TOKEN [ OVERWRITE | IF NOT EXISTS ] @name ON [ NAMESPACE | DATABASE | SCOPE @scope ] TYPE @type VALUE @value [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineTokenAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "TOKEN" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Terminal", text: "ON" },
      { type: "Choice", index: 1, children: [ { type: "Terminal", text: "NAMESPACE" }, { type: "Terminal", text: "DATABASE" }, { type: "Sequence", children: [ { type: "Terminal", text: "SCOPE" }, { type: "NonTerminal", text: "@scope" } ] } ] },
      { type: "Terminal", text: "TYPE" },
      { type: "NonTerminal", text: "@type" },
      { type: "Terminal", text: "VALUE" },
      { type: "NonTerminal", text: "@value" },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineTokenAst} className="my-6" />

  </TabItem>
</Tabs>

## Verification Types

When defining a token, its type describes the cryptographic algorithm or specification that will be used to verify the token. This can be an HMAC algorithm, a public-key cryptography algorithm or a remote JWKS object containing all the required information to verify the token. When not specified, the type is defined as the `HS256` HMAC cryptographic algorithm.

### Hash-Based Message Authentication Code (HMAC)

With HMAC algorithms (`HS256`,`HS384`,`HS512`) the value of the defined token will be the secret used both to sign (by the issuer of the token) and verify (by SurrealDB) the token. Anyone with access to this secret will be able to issue tokens with arbitrary claims which will be trusted by SurrealDB.

The following example shows the definition of a token using an HMAC algorithm.

```surql
-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

-- Set the name of the token
DEFINE TOKEN token_name
  -- Use this token provider for database authorization
  ON DATABASE
  -- Specify the cryptographic signature algorithm used to verify the token
  TYPE HS512
  -- Specify the secret used to sign and verify the authenticity of the token
  VALUE "sNSYneezcr8kqphfOC6NwwraUHJCVAt0XjsRSNmssBaBRh3WyMa9TRfq8ST7fsU2H2kGiOpU4GbAF1bCiXmM1b3JGgleBzz7rsrz6VvYEM4q3CLkcO8CMBIlhwhzWmy8"
;
```

### Public-Key Cryptography

With public-key cryptography algorithms (`EDDSA`, `ES256`, `ES384`, `ES512`, `PS256`, `PS384`, `PS512`, `RS256`, `RS384`, `RS512`) the value of the defined token will be the public key used to verify the signature of the token. This value is not secret and should be provided by the issuer of the tokens. Tokens will be signed using the private key, known only to the issuer. The public key value should be provided to SurrealDB including its header and footer. Any whitespace will be trimmed.

The following example shows the definition of a token using a public-key cryptography algorithm.

```surql
-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

-- Set the name of the token
DEFINE TOKEN token_name
  -- Use this token provider for database authorization
  ON DATABASE
  -- Specify the cryptographic signature algorithm used to verify the token
  TYPE RS256
  -- Specify the public key used to verify the authenticity of the token
  VALUE "-----BEGIN PUBLIC KEY-----
MUO52Me9HEB4ZyU+7xmDpnixzA/CUE7kyUuE0b7t38oCh+sQouREqIjLwgHhFdhh3cQAwr6GH07D
ThioYrZL8xATJ3Youyj8C45QnZcGUif5PkpWXDi0HJSoMFekbW6Pr4xuqIqb2LGxGDVJcLZwJ2AS
Gtu2UAfPXbBD3ffiad393M22g1iHM80YaNi+xgswG7qtXE4lR/Lt4s0MeKKX7stdWI1VIsoB+y3i
r/OWUvJPjjDNbAsyy8tQmxydv+FUnLEP9TNT4AhN4DXcJ+XsDtW7OWt4EdSVDeKpGbIMvIrh1Pe+
Nilj8UHNyNDHa2AjK3seMo6CMvaIQJKj5o4xGFblFGwvvPD03SbuQLs1FdRjsZCeWLdYeQ3JDHE9
sFG7DCXlpMJcaYT1mf4XHJ0gPekNLQyewTY3Vxf7FgV3GCNjV20kcDFgJA2+iVW2wSrb+txD1ycE
kbi8jh0pedWwE40VQWaTh/8eAvX7IHWya/AEro25mq+m6vktNZLbvLphhp586kJK3Tdt3YjpkPre
M3nkFWOWurIyKbtIV9JemfwCgt89sNV45dTlnEDEZFFGnIgDnWgx3CUo4XmhICEQU8+tklw9jJYx
iCTjhbIDEBHySSSc/pQ4ftHQmhToTlQeOdEy4LYiaEIgl1X+hzRH1hBYvWlNKe4EY1nMCKcjgt0=
-----END PUBLIC KEY-----"
;
```

### JSON Web Key Set (JWKS)

<Since v="v1.2.0" />

With JWKS, a set of JWK (JSON Web Key) objects will be dynamically fetched from a remote location and used to verify tokens following the [RFC 7517](https://datatracker.ietf.org/doc/html/rfc7517) specification. When defining a JWKS token verification method, its value should contain a valid URL that is reachable by SurrealDB and allowed by the configured network [capabilities](/docs/surrealdb/security/capabilities). This URL should point to a valid JWKS object (as described in [Section 5 of RFC 7517](https://datatracker.ietf.org/doc/html/rfc7517#section-5)) in the form of a JSON document. This is the recommended method to integrate with authentication providers that support JWKS. Providers like [Google](https://developers.google.com/identity/openid-connect/openid-connect#discovery), [AWS Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html#amazon-cognito-user-pools-using-tokens-manually-inspect), [Azure Active Directory](https://azure.github.io/azure-workload-identity/docs/installation/self-managed-clusters/oidc-issuer/jwks.html), [Auth0](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-key-sets), [Keycloak](https://documentation.cloud-iam.com/how-to-guides/configure-remote-jkws.html) or [OneLogin](https://developers.onelogin.com/authentication/tools/jwt) provide JWKS endpoints to verify tokens issued by their services.

The following example shows the definition of a token using a JWKS.

```surql
-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

-- Set the name of the token
DEFINE TOKEN token_name
  -- Use this token provider for database authorization
  ON DATABASE
  -- Specify the JWKS specification used to verify the token
  TYPE JWKS
  -- Specify the URL where the JWKS object can be found
  VALUE "https://example.com/.well-known/jwks.json"
;
```

Validating tokens generated by third-party authentication providers using JWKS ensures that keys can be revoked directly from the third-party service and will no longer be accepted by SurrealDB after the local cache for those keys expires. Likewise, it ensures that token verification will not break if keys are rotated, as any new keys will be automatically fetched from the authentication provider if a JWT is received containing a new key identifier in its `kid` header.

To avoid performing requests to the remote URL for each token that is verified, SurrealDB caches every JWKS object that it pulls for a period of 12 hours. The cache can be purged earlier (e.g. in the event a key is compromised) by restarting the SurrealDB server. If a JWT is received containing a reference to a new key identifier in its `kid` header, the JWKS object will be fetched again and updated in the cache if the key identifier is found in the remote JWKS object; this operation will only be performed once every 5 minutes to prevent malicious actors from abusing this process to perform denial of service.

## Requirements

- To `DEFINE TOKEN ... ON NAMESPACE ...` you must have root or namespace level access.
- To `DEFINE TOKEN ... ON DATABASE ...` you must have root, namespace, or database level access.
- To `DEFINE TOKEN ... ON SCOPE ...` you must have root, namespace, or database level access.
- [You must select your namespace and/or database](/docs/surrealql/statements/use) before you can use the `DEFINE TOKEN` statement for database or namespace tokens.

## Using `IF NOT EXISTS` clause



The `IF NOT EXISTS` clause can be used to define a token only if it does not already exist. If the token already exists, the `DEFINE TOKEN` statement will return an error.

```surql
-- Create a TOKEN if it does not already exist
DEFINE TOKEN IF NOT EXISTS example ON SCOPE example TYPE HS512 VALUE "example";
```

## Using Tokens

The `DEFINE TOKEN` statement lets you specify the amount of permission granting authority you want to give to a token issuer. You are able to specify if the provider can grant namespace, database, or scope level access to token holders. For this to work, the JWT issued to be used with SurrealDB must contain claims to specify which namespace, database or scope the token bearer is authorized to act on.

The following claims should be added to the JWT payload by the issuer of the token:

- `exp`: The token expiration Unix time. The token will not be valid after.
- `tk`: The name that you chose when defining the token.
- `ns`: The namespace that the token is issued for.
- `db`: The database that the token is issued for.
- `sc`: The scope that the token is issued for.

The names of these claims can be in all lowercase (i.e. `tk`) or all uppercase (i.e. `TK`), and can be optionally prefaced with the `https://surrealdb.com` namespace (e.g. `https://surrealdb.com/tk`) in order to separate claims directed to SurrealDB from claims directed to other services. When using a namespace, the claim name can also be used without abbreviation, such as in `https://surrealdb.com/token` or `https://surrealdb.com/scope`. Even when present in the token with a namespace prefix, [SurrealDB claims](https://github.com/surrealdb/surrealdb/blob/main/core/src/iam/token.rs) are directly accessible via the `$token` parameter (e.g. `$token.sc`), whereas custom claims will need to include the namespace prefix (e.g. `$token['https://surrealdb.com/pet_name']`) to be accessed in the same way.

The following optional claims are also processed by SurrealDB:

- `id`: The identifier of the resource (e.g. user) associated with the token.
- `nbf`: The token acceptance Unix time. The token will not be valid before.

The expected claims depend on the level at which the token was defined:

- For tokens defined `ON NAMESPACE`: `exp`, `tk`, `ns`.
- For tokens defined `ON DATABASE`: `exp`, `tk`,`ns`,`db`.
- For tokens defined `ON SCOPE`: `exp`, `tk`, `ns`,`db`, `sc`, optionally `id`.

For tokens defined `ON NAMESPACE` and `ON DATABASE`, the optional `rl` claim containing an array of capitalized [system user roles](/docs/surrealql/statements/define/user#roles) (e.g. `["Viewer", "Editor", "Owner"]`) can be provided. Doing so will apply the access policy for those roles to any action made using the token. By default, tokens without the `rl` claim will only have the `Viewer` role.

When calling any of the SurrealDB interfaces using a JWT, SurrealQL queries will gain access to the claims in the token through the `$token` variable. For example, if the token contains custom claims such as “name” or “email”, the values of those claims will be accessible through `$token.name` and `$token.email`.

Additionally, when the `id` claim is present in the token, the fields of the record matching the identifier specified will be accessible through the `$auth` variable. For example, if the value of the `id` claim is `user:73q1bl039y6k8z80v55d`, and user records have fields such as “name” or “email”, then `$auth.name` and `$auth.email` can be used to access those values for the `user:73q1bl039y6k8z80v55d` record specifically, without them being present in the JWT.

The signature of the token is verified with method defined when creating the token. If the signature of the token is invalid, calls to SurrealDB interfaces using that token will fail.

### Namespace

Namespace tokens can be used to select, create, update, and delete on all tables in all databases, as well as to define and remove databases and tables from the namespace.

```surql
-- Specify the namespace for the token
USE NS abcum;

-- Set the name of the token
DEFINE TOKEN token_name
  -- Use this OAuth provider for namespace authorization
  ON NAMESPACE
  -- Specify the cryptographic signature algorithm used to verify the token
  TYPE HS512
  -- Specify the public key so we can verify the authenticity of the token
  VALUE "sNSYneezcr8kqphfOC6NwwraUHJCVAt0XjsRSNmssBaBRh3WyMa9TRfq8ST7fsU2H2kGiOpU4GbAF1bCiXmM1b3JGgleBzz7rsrz6VvYEM4q3CLkcO8CMBIlhwhzWmy8"
;
```
The namespace token payload should at least include the following claims when used to authenticate with SurrealDB.

```json title="JWT Payload"
{
  "exp": 2147483647,
  "tk": "token_name",
  "ns": "abcum"
}
```

### Database

Database tokens can be used to select, create, update, and delete on all tables in a specific database, as well as to define and remove tables from the database.

```surql
-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

-- Set the name of the token
DEFINE TOKEN token_name
  -- Use this OAuth provider for database authorization
  ON DATABASE
  -- Specify the cryptographic signature algorithm used to verify the token
  TYPE HS512
  -- Specify the public key so we can verify the authenticity of the token
  VALUE "sNSYneezcr8kqphfOC6NwwraUHJCVAt0XjsRSNmssBaBRh3WyMa9TRfq8ST7fsU2H2kGiOpU4GbAF1bCiXmM1b3JGgleBzz7rsrz6VvYEM4q3CLkcO8CMBIlhwhzWmy8"
;
```

The database token payload should at least include the following claims when used to authenticate with SurrealDB.

```json title="JWT Payload"
{
  "exp": 2147483647,
  "tk": "token_name",
  "ns": "abcum",
  "db": "app_vitalsense"
}
```

### Scope

Since the origin of the claims in the JWT is verified, those claims can be used within SurrealQL in the context of a scope in order to provide table and field authorization through an external authenticator using OpenID Connect, OAuth or simply acting as a trusted issuer of a JWT.

This can be done by leveraging table permissions to allow or disallow access depending on the values of the claims in the verified token. For example, these claims can be compared with the records in a table to only return those matching certain criteria.

The scope for which the token was issued will be accessible to SurrealQL through the `$scope` variable, corresponding to the contents of the `sc` claim. External authorization providers may provide additional scopes that will not be accessible in this way, and instead should be accessed as any other claim through the `$token` variable.

Bear in mind that table and field permissions only apply to scope level tokens Access provided by namespace and database tokens is above table-level permissions. When application users will be the ones directly authenticating with JWT, scope tokens are most likely the right choice.

The following example shows how scope tokens can be used to grant authorization by verifying that the “email” claim in the token matches the email used as the index of a user table:


```surql
-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

-- Necessary in order to define a scope token
DEFINE SCOPE users;

DEFINE TOKEN token_name ON SCOPE users TYPE RS256 VALUE "-----BEGIN PUBLIC KEY-----
MUO52Me9HEB4ZyU+7xmDpnixzA/CUE7kyUuE0b7t38oCh+sQouREqIjLwgHhFdhh3cQAwr6GH07D
ThioYrZL8xATJ3Youyj8C45QnZcGUif5PkpWXDi0HJSoMFekbW6Pr4xuqIqb2LGxGDVJcLZwJ2AS
Gtu2UAfPXbBD3ffiad393M22g1iHM80YaNi+xgswG7qtXE4lR/Lt4s0MeKKX7stdWI1VIsoB+y3i
r/OWUvJPjjDNbAsyy8tQmxydv+FUnLEP9TNT4AhN4DXcJ+XsDtW7OWt4EdSVDeKpGbIMvIrh1Pe+
Nilj8UHNyNDHa2AjK3seMo6CMvaIQJKj5o4xGFblFGwvvPD03SbuQLs1FdRjsZCeWLdYeQ3JDHE9
sFG7DCXlpMJcaYT1mf4XHJ0gPekNLQyewTY3Vxf7FgV3GCNjV20kcDFgJA2+iVW2wSrb+txD1ycE
kbi8jh0pedWwE40VQWaTh/8eAvX7IHWya/AEro25mq+m6vktNZLbvLphhp586kJK3Tdt3YjpkPre
M3nkFWOWurIyKbtIV9JemfwCgt89sNV45dTlnEDEZFFGnIgDnWgx3CUo4XmhICEQU8+tklw9jJYx
iCTjhbIDEBHySSSc/pQ4ftHQmhToTlQeOdEy4LYiaEIgl1X+hzRH1hBYvWlNKe4EY1nMCKcjgt0=
-----END PUBLIC KEY-----";

DEFINE TABLE user SCHEMAFULL
  -- Authorized users can select, update, delete and create user records
  PERMISSIONS FOR select, update, delete, create
  -- The current scope must be "users"
  WHERE $scope = "users"
  -- The email of the user being queried must match the email claim in the token
  -- Only matching records will be changed or returned
  AND email = $token.email
;

DEFINE INDEX email ON user FIELDS email UNIQUE;
DEFINE FIELD email ON user TYPE string ASSERT string::is_email($value);
DEFINE FIELD name ON user TYPE string;
DEFINE FIELD nickname ON user TYPE string;
DEFINE FIELD picture ON user TYPE string;
```

You may also use permissions clauses to perform additional verification on other JWT claims (e.g. verifying that the iss claim matches a specific principal using $token.iss) that may be required or recommended by a the provider of the token.

The scope token payload should at least include the following claims when used to authenticate with SurrealDB.


```json title="JWT Payload"
{
  "exp": 2147483647,
  "tk": "token_name",
  "ns": "abcum",
  "db": "app_vitalsense",
  "sc": "users"
}
```


## user.mdx

---
sidebar_position: 18
sidebar_label: DEFINE USER
title: DEFINE USER statement | SurrealQL
description: Use the DEFINE USER statement to create system users on SurrealDB.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `DEFINE USER` statement

Use the `DEFINE USER` statement to create system users on SurrealDB

> [!NOTE]
> While existing logins still function, the DEFINE LOGIN statement has been replaced with DEFINE USER.

## Requirements

- You must be authenticated with a user that has enough permissions. Only the OWNER built-in role grants permissions to create users.
- You must be authenticated with a user that has permissions on the level where you are creating the user:
  - Root users with owner permissions can create Root, Namespace and Database users.
  - Namespace users with owner permissions can create Namespace and Database users
  - Database users with owner permissions can create Database users.
- To select the level where you want to create the user, [you may need to select a namespace and/or database](/docs/surrealql/statements/use) before you can use the `DEFINE USER` statement for database or namespace tokens.

> [!NOTE]
> You cannot use the DEFINE USER statement to create a record user.

## Statement syntax

<Tabs syncKey="define-user-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
DEFINE USER [ OVERWRITE | IF NOT EXISTS ] @name
	ON [ ROOT | NAMESPACE | DATABASE ]
	[ PASSWORD @pass | PASSHASH @hash ]
	[ ROLES @roles ]
	[ DURATION ( FOR TOKEN @duration [ , ] [ FOR SESSION @duration ] | FOR SESSION @duration [ , ] [ FOR TOKEN @duration ] ) ]
  [ COMMENT @string ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const defineUserAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "DEFINE" },
      { type: "Terminal", text: "USER" },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Terminal", text: "OVERWRITE" }, { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "NOT" }, { type: "Terminal", text: "EXISTS" } ] } ] } },
      { type: "NonTerminal", text: "@name" },
      { type: "Terminal", text: "ON" },
      { type: "Choice", index: 1, children: [ { type: "Terminal", text: "ROOT" }, { type: "Terminal", text: "NAMESPACE" }, { type: "Terminal", text: "DATABASE" } ] },
      { type: "Optional", child: { type: "Choice", index: 1, children: [ { type: "Sequence", children: [ { type: "Terminal", text: "PASSWORD" }, { type: "NonTerminal", text: "@pass" } ] }, { type: "Sequence", children: [ { type: "Terminal", text: "PASSHASH" }, { type: "NonTerminal", text: "@hash" } ] } ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "ROLES" }, { type: "NonTerminal", text: "@roles" } ] } },
      { type: "Optional", child: { type: "Sequence", children: [
        { type: "Terminal", text: "DURATION" },
        { type: "Choice", index: 1, children: [
          { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "TOKEN" }, { type: "NonTerminal", text: "@duration" }, { type: "Optional", child: { type: "Terminal", text: "," } }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "SESSION" }, { type: "NonTerminal", text: "@duration" } ] } } ] },
          { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "SESSION" }, { type: "NonTerminal", text: "@duration" }, { type: "Optional", child: { type: "Terminal", text: "," } }, { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "FOR" }, { type: "Terminal", text: "TOKEN" }, { type: "NonTerminal", text: "@duration" } ] } } ] }
        ] }
      ] } },
      { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "COMMENT" }, { type: "NonTerminal", text: "@string" } ] } }
    ]}
  ]
};

<RailroadDiagram ast={defineUserAst} className="my-6" />

  </TabItem>
</Tabs>

## Example usage

The following example shows how you can create a `ROOT` user using the `DEFINE USER` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create the user with an owner role and some example durations
DEFINE USER username ON ROOT PASSWORD '123456' ROLES OWNER DURATION FOR SESSION 15m, FOR TOKEN 5s;
```

Note that even a root-level user can be given a limited role such as `VIEWER`. This can be useful for automated services that need to monitor each namespace and database, or coworkers high up the org chart that are not particularly tech-savvy.

```surql
DEFINE USER birthday_bot ON ROOT PASSWORD "botpassword9!" ROLES VIEWER;
DEFINE USER clumsy_ceo ON ROOT PASSWORD "password" ROLES VIEWER COMMENT "Don't let the CEO have more than VIEWER access";
```

The following example shows how you can create a `NAMESPACE` user using the `DEFINE USER` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Specify the namespace
USE NS abcum;
-- Create the user with an editor role and some example durations
DEFINE USER username ON NAMESPACE PASSWORD '123456' ROLES EDITOR DURATION FOR SESSION 12h, FOR TOKEN 1m;
```

The following example shows how you can create a `DATABASE` user using the `DEFINE USER` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Specify the namespace and database for the user
USE NS abcum DB app_vitalsense;
-- Create the user with a viewer role and some example durations
DEFINE USER username ON DATABASE PASSWORD '123456' ROLES VIEWER DURATION FOR SESSION 5d, FOR TOKEN 2h;
```

## Using `IF NOT EXISTS` clause

The `IF NOT EXISTS` clause can be used to define a user only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining a user in SurrealDB if you want to ensure that the user is only created if it does not already exist. If the user already exists, the `DEFINE USER` statement will return an error.

It's particularly useful when you want to safely attempt to define a user without manually checking its existence first.

On the other hand, you should not use the `IF NOT EXISTS` clause when you want to ensure that the user definition is updated regardless of whether it already exists. In such cases, you might prefer using the `OVERWRITE` clause, which allows you to define a user and overwrite an existing one if it already exists, ensuring that the latest version of the user definition is always in use

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a USER if it does not already exist
DEFINE USER IF NOT EXISTS example ON ROOT PASSWORD "example" ROLES OWNER;
```

## Using `OVERWRITE` clause

<Since v="v2.0.0" />

The `OVERWRITE` clause can be used to define a user and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing user definition. If the user already exists, the `DEFINE USER` statement will overwrite the existing user definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create an USER and overwrite if it already exists
DEFINE USER OVERWRITE example ON ROOT PASSWORD "example" ROLES OWNER;
```

## Roles

Currently, only the built-in roles OWNER, EDITOR and VIEWER are available.

<table>
<thead>
  <tr>
    <th>Role</th>
    <th>Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>OWNER</td>
    <td>Can view and edit any resource on the user's level or below, including user and token (IAM) resources.<br/>It also grants full permissions for child resources that support the `PERMISSIONS` clause (tables, fields, etc.)</td>
  </tr>
  <tr>
    <td>EDITOR</td>
    <td>Can view and edit any resource on the user's level or below, but not users or token (IAM) resources<br/>It also grants full permissions for child resources that support the `PERMISSIONS` clause (tables, fields, etc.)</td>
  </tr>
  <tr>
    <td>VIEWER</td>
    <td>Grants permissions to view any resource on the user's level or below, but not edit.<br/>It also grants view permissions for child resources that support the `PERMISSIONS` clause (tables, fields, etc.)</td>
  </tr>
</tbody>
</table>

## Duration

<Since v="v2.0.0" />

The duration clause specifies the duration of the token returned after successful authentication with a password or passhash as well as the duration of the session established both using a password or passhash and the aforementioned token. The difference between these concepts is explained in the [expiration](/docs/surrealdb/security/authentication#expiration) documentation.


## bearer.mdx

---
sidebar_position: 10
sidebar_label: BEARER 
title: DEFINE ACCESS ... TYPE BEARER statement | SurrealQL
description: A bearer access method allows accessing SurrealDB using a bearer key.
---

import Since from '@components/shared/Since.astro'

# `DEFINE ACCESS ... TYPE BEARER`

A bearer access method allows generating bearer grants with an associated key that can be used to access SurrealDB as a specific [system user](/docs/surrealdb/security/authentication#system-users) or [record user](/docs/surrealdb/security/authentication#record-users). Bearer grants allow other systems and software to authenticate with SurrealDB using a secure and unique credential that can be [audited](/docs/surrealql/statements/access#show) and [revoked](/docs/surrealql/statements/access#revoke) at any time.

Allowing access to SurrealDB using a bearer access method requires creating grants associated with that access method. This can be done using the [`GRANT`](/docs/surrealql/statements/access#grant) clause of the [`ACCESS`](/docs/surrealql/statements/access) statement.

After creating a grant for a subject (i.e. a [system user](/docs/surrealdb/security/authentication#system-users) or a [record user](/docs/surrealdb/security/authentication#record-users)) with a bearer access method, a bearer key will be returned. This bearer key can be used to sign in as the subject of the grant without using its password or any other credentials. As with other credentials in SurrealDB, signing in with a bearer key will return a JWT, which can be used to perform authenticated operations or establish a persistent [authenticated session](/docs/surrealdb/security/authentication). This makes bearer keys most suitable for automations and other service-to-service authentication use cases that require interacting with SurrealDB in an authenticated context by providing stronger security guarantees than passwords and removing the complexity of having to work with JWT directly.

## Requirements

- You must be authenticated as a [root, namespace or database user](/docs/surrealql/statements/define/user) before you can define a bearer access method.

## Statement syntax

```syntax title="SurrealQL Syntax"
DEFINE ACCESS [ OVERWRITE | IF NOT EXISTS ] @name
  ON [ NAMESPACE | DATABASE ]
  TYPE BEARER FOR [ USER | RECORD ]
  [ AUTHENTICATE @expression ]
  [ DURATION
    [ FOR GRANT @duration ]
    [ FOR TOKEN @duration ]
    [ FOR SESSION @duration ]
  ]
```

## `FOR USER`

Defining a bearer access method `FOR USER` will ensure that grants can only be created with a [system user](/docs/surrealdb/security/authentication#system-users) as its subject. This application is useful for integrations that require administering SurrealDB at the `ROOT`, `NAMESPACE` or `DATABASE` level with the roles with which the user has been defined with [`DEFINE USER`](/docs/surrealql/statements/define/user). 

### Example

```surql
DEFINE ACCESS api ON DATABASE TYPE USER DURATION FOR GRANT 30d, FOR TOKEN 15m, FOR SESSION 12h;
```

In this example, grants created with this bearer access method will be valid for 30 days. After signing in with any of those grants, SurrealDB will return a token that will be valid for 15 minutes. This token can be used to establish an authenticated SurrealDB session valid for 12 hours. Grants created with this access method will only allow a system user as its subject.

```surql
-- Define system user that access will be granted to
DEFINE USER automation ON DATABASE PASSWORD 'secret' ROLES VIEWER;
-- Define bearer access method to generate API keys for system users
DEFINE ACCESS api ON DATABASE TYPE BEARER FOR USER DURATION FOR GRANT 10d;
-- Generate bearer grant to be used by a specific automation
ACCESS api GRANT FOR USER automation;
```

```surql title="Response"
{
	ac: 'api',
	creation: d'2025-10-07T04:52:36.157Z',
	expiration: d'2025-10-17T04:52:36.157Z',
	grant: {
		id: 'W9gi9FVexSLP',
		key: 'surreal-bearer-W9gi9FVexSLP-WFmLPW6GyFyj1gJMdEY22YzA'
	},
	id: 'W9gi9FVexSLP',
	revocation: NONE,
	subject: {
		user: 'automation'
	},
	type: 'bearer'
}
```

The key value returned in the grant object is the bearer key, which can be used to sign in as the `automation` user without using its password.

Here are some examples on how to do that using the [JavaScript SDK](/docs/sdk/javascript) or a raw [HTTP request](/docs/surrealdb/integration/http).

#### JavaScript SDK

```js
const db = new Surreal();
db.connect('ws://localhost:8000/rpc', {
	namespace: 'test',
	database: 'test',
});

db.signin({
	namespace: 'test',
	database: 'test',

	// Provide the name of the access method
	access: 'api',

	// Provide the bearer key in the "key" variable
	variables: {
    		key: 'surreal-bearer-BNb2pS0GmaJz-5eTfQ5uEu8jbRb3oblqVMAt8',
	}
});
```

#### HTTP Request

```bash
curl -X POST \
	-H "Accept: application/json" \
	-d '{"NS":"test", "DB":"test", "AC":"api", "key":"surreal-bearer-BNb2pS0GmaJz-5eTfQ5uEu8jbRb3oblqVMAt8"}' \
	http://localhost:8000/signin
```

## `FOR RECORD`

Defining a bearer access method `FOR RECORD` will ensure that grants can only be created with a [record user](/docs/surrealdb/security/authentication#record-users) as its subject. This application is useful for integrations that require accessing only some data in a specific SurrealDB database and in accordance with existing `PERMISSIONS` clauses. Bearer access can only be defined `FOR RECORD` if a database is selected and using `ON DATABASE`.

```surql
-- Create record representing a user
CREATE user:1 CONTENT { name: "tobie" };
-- Define bearer access method to generate API keys for record users
DEFINE ACCESS api ON DATABASE TYPE BEARER FOR RECORD DURATION FOR GRANT 10d;
-- Generate bearer grant to be used by a specific automation belonging to the user
ACCESS api GRANT FOR RECORD user:1;
```

```surql title="Response"
-- Query 1
[
        {
                id: user:1,
                name: 'tobie'
        }
]
-- Query 2
NONE
-- Query 3
{
	ac: 'api',
	creation: d'2025-10-07T04:54:26.258986Z',
	expiration: d'2025-10-17T04:54:26.258987Z',
	grant: {
		id: 'jeGA4jfNmnoD',
		key: 'surreal-bearer-jeGA4jfNmnoD-X2xYQ1IILzB47DttrNpMBquN'
	},
	id: 'jeGA4jfNmnoD',
	revocation: NONE,
	subject: {
		record: user:1
	},
	type: 'bearer'
};
```

The key value returned in the grant object is the bearer key, which can be used to sign in as the `user:1` record in SurrealDB.

Here are some examples on how to do that using the JavaScript SDK or a raw HTTP request.

#### JavaScript SDK

```js
const db = new Surreal();
db.connect('ws://localhost:8000/rpc', {
	namespace: 'test',
	database: 'test',
});

db.signin({
	namespace: 'test',
	database: 'test',

	// Provide the name of the access method
	access: 'api',

	// Provide the bearer key in the "key" variable
	variables: {
    		key: 'surreal-bearer-NJ2I2d7OXxN9-Oa5LqF36IzfURpo6Bhxy9WMF',
	}
});
```

#### HTTP Request

```bash
curl -X POST \
	-H "Accept: application/json" \
	-d '{"NS":"test", "DB":"test", "AC":"api", "key":"surreal-bearer-NJ2I2d7OXxN9-Oa5LqF36IzfURpo6Bhxy9WMF"}' \
	http://localhost:8000/signin
```


## Using `IF NOT EXISTS` clause

The `IF NOT EXISTS` clause can be used to define an access method of type BEARER only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining an access method in SurrealDB if you want to ensure that the access method is only created if it does not already exist. If the access method already exists, the `DEFINE ACCESS` statement will return an error.

It's particularly useful when you want to safely attempt to define an access method without manually checking its existence first.

```surql
-- Create a BEARER access method for the example database if it does not already exist
DEFINE ACCESS IF NOT EXISTS example ON DATABASE TYPE BEARER;
```

## Using `OVERWRITE` clause

The `OVERWRITE` clause can be used to define an access method of type BEARER and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing access method definition. If the access method already exists, the `DEFINE ACCESS` statement will overwrite the existing access method definition with the new one.

```surql
-- Create a BEARER access method for the example database and overwrite if it already exists
DEFINE ACCESS OVERWRITE example ON DATABASE TYPE BEARER;
```


## index.mdx

---
sidebar_position: 10
sidebar_label: DEFINE ACCESS
title: DEFINE ACCESS statement | SurrealQL
description: Defining an access method allows SurrealDB to grant access to resources using different kinds of credentials.
---

import Since from '@components/shared/Since.astro'

# `DEFINE ACCESS` statement

<Since v="v2.0.0" />

Defining an access method allows SurrealDB to grant access to resources using different kinds of credentials.

## Requirements

- You must be authenticated as a [system user](/docs/surrealdb/security/authentication#system-users) at the same level or higher than the level on which access is defined.

## Statement syntax

```syntax title="SurrealQL Syntax"
DEFINE ACCESS [ OVERWRITE | IF NOT EXISTS ] @name
  ON [ ROOT | NAMESPACE | DATABASE ]
  TYPE [
    JWT [ ALGORITHM @algorithm KEY @key | URL @url ]
    | RECORD
      [ SIGNUP @expression ]
      [ SIGNIN @expression ]
      [ WITH JWT
        [ ALGORITHM @algorithm KEY @key | URL @url ]
        [ WITH ISSUER KEY @key ]
      ]
      [ WITH REFRESH ]
    | BEARER FOR [ USER | RECORD ]
  [ AUTHENTICATE @expression ]
  [ DURATION
    [ FOR GRANT @duration ]
    [ FOR TOKEN @duration ]
    [ FOR SESSION @duration ]
  ]
  [ COMMENT @string ]
```

## JSON Web Token (JWT) Access

A JWT access method allows accessing SurrealDB with a token signed by a trusted issuer. The contents of the token will be trusted by SurrealDB as long as it has been signed with a trusted credential.

Learn more about [JWT access method in the documentation](/docs/surrealql/statements/define/access/jwt).

## Record Access

A record access method allows accessing SurrealDB as a [record user](/docs/surrealdb/security/authentication#record-users). Record users allow SurrealDB to operate as a web database by offering mechanisms to define custom signin and signup logic as well as custom table and field permissions.

Learn more about [record access method in the documentation](/docs/surrealql/statements/define/access/record).

## Bearer Access

A bearer access method allows generating bearer grants with an associated key that can be used to access SurrealDB as a specific [system user](/docs/surrealdb/security/authentication#system-users) or [record user](/docs/surrealdb/security/authentication#record-users). Bearer grants allow other systems and software to authenticate with SurrealDB using a secure and unique credential that can be audited and revoked at any time.

Learn more about [bearer access method in the documentation](/docs/surrealql/statements/define/access/bearer).

## Duration

The duration clause specifies the duration of the token returned after successful authentication with the access method as well as the duration of the session established both using the access method and the aforementioned token. The difference between these concepts is explained in the [expiration documentation](/docs/surrealdb/security/authentication#expiration).

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a RECORD access method for accounts
-- On successful authentication, a token expiring after 15 minutes will be returned
-- This token can be used to establish a session that will expire after 6 hours
-- The token will be automatically used to authenticate the session
DEFINE ACCESS account ON DATABASE TYPE RECORD
	SIGNUP ( CREATE user SET email = $email, pass = crypto::argon2::generate($pass) )
	SIGNIN ( SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass) )
	DURATION FOR TOKEN 15m, FOR SESSION 12h
;
```

## With `AUTHENTICATE` clause

The authenticate clause can be used to change the record identifier returned by the `SIGNIN` and `SIGNUP` clauses or replace the identifier provided in the token when authenticating `WITH JWT`, In the context of [`DEFINE ACCESS ... TYPE RECORD`](/docs/surrealql/statements/define/access/record), the `AUTHENTICATE` clause is always executed across signin, signup and token authentication.

When used in a [`DEFINE ACCESS ... TYPE JWT`](/docs/surrealql/statements/define/access/jwt), the `AUTHENTICATE` clause is used to validate the token claims and can be used to log or stop authentication attempts.

In both cases, the clause expects nothing to be returned and will otherwise fail with a generic error. The `THROW` statement can be called to return a custom error to the end user.

## Using `IF NOT EXISTS` clause


The `IF NOT EXISTS` clause can be used to define an access method only if it does not already exist. If the access method already exists, the `DEFINE ACCESS` statement will return an error.

```surql
-- Create an ACCESS if it does not already exist
DEFINE ACCESS IF NOT EXISTS example ON NAMESPACE ...;
```

## Using `OVERWRITE` clause

The `OVERWRITE` clause can be used to define an access method and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing access method definition. If the access method already exists, the `DEFINE ACCESS` statement will overwrite the existing access method definition with the new one.

```surql
-- Create an ACCESS and overwrite if it already exists
DEFINE ACCESS OVERWRITE example ON NAMESPACE ...;
```


## jwt.mdx

---
sidebar_position: 10
sidebar_label: JWT
title: DEFINE ACCESS ... TYPE JWT statement | SurrealQL
description: A JWT access method allows accessing SurrealDB with a token signed by a trusted issuer.
---

import Since from '@components/shared/Since.astro'


# `DEFINE ACCESS ... TYPE JWT`

<Since v="v2.0.0" />

A JWT access method allows accessing SurrealDB with a token signed by a trusted issuer. The contents of the token will be trusted by SurrealDB as long as it has been signed with a trusted credential.

SurrealDB can work with third-party authentication providers such as OpenID Connect providers, OAuth providers and other trusted parties providing JWT (JSON Web Tokens, also referred to in this page as “tokens”). Let's say that your provider issues your client (e.g. a user or a service) a JWT once it has authenticated. By using the `DEFINE ACCESS ... TYPE JWT` statement, you can set the public key or shared secret that will be used to verify the authenticity of the token.

This verification is performed automatically by SurrealDB when provided with a JWT through any of its interfaces (i.e. the [HTTP REST API](/docs/surrealdb/integration/http) through the “Authorization” header or [any of the SDKs](/docs/surrealdb/integration/sdks) through the “Authenticate” methods) before trusting the claims contained in the token and allowing SurrealQL queries to access the values of those claims.

Bear in mind that table and field permissions only apply to [record users](/docs/surrealdb/security/authentication#record-users), which must use tokens that are verified by a `RECORD` access method. Access provided by namespace and database tokens defined in a `JWT` access method is equivalent to access from [system users](/docs/surrealdb/security/authentication#system-users), which is above fine-grained permissions. When application users will be the ones directly authenticating with JWT, defining a `RECORD` access method `WITH JWT` is most likely the right choice.

## Requirements

- You must be authenticated as a [system user](/docs/surrealdb/security/authentication#system-users) at the same level or higher than the level to which you want to provide JWT access.
- [You must select a namespace or database](/docs/surrealql/statements/use) before you can define a JWT access method.

## Statement syntax

```syntax title="SurrealQL Syntax"
DEFINE ACCESS [ OVERWRITE | IF NOT EXISTS ] @name
  ON [ ROOT | NAMESPACE | DATABASE ]
  TYPE JWT [ ALGORITHM @algorithm KEY @key | URL @url ]
  [ AUTHENTICATE @expression ]
  [ DURATION FOR SESSION @duration ]
```

## Verification Types

When defining a token, its type describes the cryptographic algorithm or specification that will be used to verify the token. This can be an HMAC algorithm, a public-key cryptography algorithm or a remote JWKS object containing all the required information to verify the token. When not specified, the type is defined as the `HS256` HMAC cryptographic algorithm.

### Hash-Based Message Authentication Code (HMAC)

With HMAC algorithms (`HS256`,`HS384`,`HS512`) the value of the defined token will be the secret used both to sign (by the issuer of the token) and verify (by SurrealDB) the token. Anyone with access to this secret will be able to issue tokens with arbitrary claims which will be trusted by SurrealDB.

The following example shows the definition of a token using an HMAC algorithm.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

-- Set the name of the token
DEFINE ACCESS token_name
  -- Use this token provider for database authentication
  ON DATABASE
  -- Specify the type of access being defined
  TYPE JWT
  -- Specify the cryptographic signature algorithm used to verify the token
  ALGORITHM HS512
  -- Specify the symmetric key used to sign and verify the authenticity of the token
  KEY "sNSYneezcr8kqphfOC6NwwraUHJCVAt0XjsRSNmssBaBRh3WyMa9TRfq8ST7fsU2H2kGiOpU4GbAF1bCiXmM1b3JGgleBzz7rsrz6VvYEM4q3CLkcO8CMBIlhwhzWmy8"
;
```

### Public-Key Cryptography

With public-key cryptography algorithms (`EDDSA`, `ES256`, `ES384`, `ES512`, `PS256`, `PS384`, `PS512`, `RS256`, `RS384`, `RS512`) the value of the defined token will be the public key used to verify the signature of the token. This value is not secret and should be provided by the issuer of the tokens. Tokens will be signed using the private key, known only to the issuer. The public key value should be provided to SurrealDB including its header and footer. Any whitespace will be trimmed.

The following example shows the definition of a token using a public-key cryptography algorithm.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

-- Set the name of the token
DEFINE ACCESS token_name
  -- Use this token provider for database authentication
  ON DATABASE
  -- Specify the type of access being defined
  TYPE JWT
  -- Specify the cryptographic signature algorithm used to verify the token
  ALGORITHM RS256
  -- Specify the public key used to verify the authenticity of the token
  KEY "-----BEGIN PUBLIC KEY-----
MUO52Me9HEB4ZyU+7xmDpnixzA/CUE7kyUuE0b7t38oCh+sQouREqIjLwgHhFdhh3cQAwr6GH07D
ThioYrZL8xATJ3Youyj8C45QnZcGUif5PkpWXDi0HJSoMFekbW6Pr4xuqIqb2LGxGDVJcLZwJ2AS
Gtu2UAfPXbBD3ffiad393M22g1iHM80YaNi+xgswG7qtXE4lR/Lt4s0MeKKX7stdWI1VIsoB+y3i
r/OWUvJPjjDNbAsyy8tQmxydv+FUnLEP9TNT4AhN4DXcJ+XsDtW7OWt4EdSVDeKpGbIMvIrh1Pe+
Nilj8UHNyNDHa2AjK3seMo6CMvaIQJKj5o4xGFblFGwvvPD03SbuQLs1FdRjsZCeWLdYeQ3JDHE9
sFG7DCXlpMJcaYT1mf4XHJ0gPekNLQyewTY3Vxf7FgV3GCNjV20kcDFgJA2+iVW2wSrb+txD1ycE
kbi8jh0pedWwE40VQWaTh/8eAvX7IHWya/AEro25mq+m6vktNZLbvLphhp586kJK3Tdt3YjpkPre
M3nkFWOWurIyKbtIV9JemfwCgt89sNV45dTlnEDEZFFGnIgDnWgx3CUo4XmhICEQU8+tklw9jJYx
iCTjhbIDEBHySSSc/pQ4ftHQmhToTlQeOdEy4LYiaEIgl1X+hzRH1hBYvWlNKe4EY1nMCKcjgt0=
-----END PUBLIC KEY-----"
;
```

### JSON Web Key Set (JWKS)

With JWKS, a set of JWK (JSON Web Key) objects will be dynamically fetched from a remote location and used to verify tokens following the [RFC 7517](https://datatracker.ietf.org/doc/html/rfc7517) specification. When defining a JWKS token verification method, its value should contain a valid URL that is reachable by SurrealDB and allowed by the configured network [capabilities](/docs/surrealdb/security/capabilities). This URL should point to a valid JWKS object (as described in [Section 5 of RFC 7517](https://datatracker.ietf.org/doc/html/rfc7517#section-5)) in the form of a JSON document. This is the recommended method to integrate with authentication providers that support JWKS. Providers like [Google](https://developers.google.com/identity/openid-connect/openid-connect#discovery), [AWS Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-verifying-a-jwt.html#amazon-cognito-user-pools-using-tokens-manually-inspect), [Azure Active Directory](https://azure.github.io/azure-workload-identity/docs/installation/self-managed-clusters/oidc-issuer/jwks.html), [Auth0](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-key-sets), [Keycloak](https://documentation.cloud-iam.com/how-to-guides/configure-remote-jkws.html) or [OneLogin](https://developers.onelogin.com/authentication/tools/jwt) provide JWKS endpoints to verify tokens issued by their services.

> [!NOTE: Before you start]
> As the JWKS functionality requires establishing a network connection in order to download the JWKS object, you will require running the SurrealDB server with the network <a href="/docs/surrealdb/security/capabilities">capability</a>. For the strongest security, provide the specific hostname hosting the JWKS object when starting SurrealDB with <code>--allow-net</code>. For example: <code>--allow-net example.com</code>.


The following example shows the definition of a token using a JWKS.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

-- Set the name of the token
DEFINE ACCESS token_name
  -- Use this token provider for database authentication
  ON DATABASE
  -- Specify the type of access being defined
  TYPE JWT
  -- Specify the URL where the JWKS object can be found
  URL "https://example.com/.well-known/jwks.json"
;
```

Validating tokens generated by third-party authentication providers using JWKS ensures that keys can be revoked directly from the third-party service and will no longer be accepted by SurrealDB after the local cache for those keys expires. Likewise, it ensures that token verification will not break if keys are rotated, as any new keys will be automatically fetched from the authentication provider if a JWT is received containing a new key identifier in its `kid` header.

To avoid performing requests to the remote URL for each token that is verified, SurrealDB caches every JWKS object that it pulls for a period of 12 hours. The cache can be purged earlier (e.g. in the event a key is compromised) by restarting the SurrealDB server. If a JWT is received containing a reference to a new key identifier in its `kid` header, the JWKS object will be fetched again and updated in the cache if the key identifier is found in the remote JWKS object; this operation will only be performed once every 5 minutes to prevent malicious actors from abusing this process to perform denial of service.

## Using Tokens

The `DEFINE ACCESS ... TYPE JWT` statement lets you specify the amount of permission granting authority you want to give to a token issuer. You are able to specify if the provider can grant namespace or database access to token holders. For this to work, the JWT issued to be used with SurrealDB must contain claims to specify which namespace or database the token bearer is authorized to act on.

The following claims should be added to the JWT payload by the issuer of the token:

- `exp`: The token expiration Unix time. The token will not be valid after.
- `ac`: The name of the access method used to verify the token.
- `ns`: The namespace that the token is issued for.
- `db`: The database that the token is issued for.

The names of these claims can be in all lowercase (i.e. `ac`) or all uppercase (i.e. `AC`), and can be optionally prefaced with the `https://surrealdb.com` namespace (e.g. `https://surrealdb.com/ac`) in order to separate claims directed to SurrealDB from claims directed to other services. When using a namespace, the claim name can also be used without abbreviation, such as in `https://surrealdb.com/access`, `https://surrealdb.com/database`...

The following optional claims are also processed by SurrealDB:

- `id`: The identifier of the resource (e.g. user) associated with the token.
- `nbf`: The token acceptance Unix time. The token will not be valid before.

The expected claims depend on the level at which the token was defined:

- For tokens defined `ON ROOT`: `exp`, `ac`.
- For tokens defined `ON NAMESPACE`: `exp`, `ac`, `ns`.
- For tokens defined `ON DATABASE`: `exp`, `ac`,`ns`,`db`.

For tokens defined for [system users](/docs/surrealdb/security/authentication#system-users), the optional `rl` claim containing an array of capitalized [system user roles](/docs/surrealql/statements/define/user#roles) (e.g. `["Viewer", "Editor", "Owner"]`) can be provided. Doing so will apply the access policy for those roles to any action made using the token. By default, sessions established with tokens without the `rl` claim will only have the `Viewer` role.

When calling any of the SurrealDB interfaces using a JWT, SurrealQL queries will gain access to the claims in the token through the `$token` variable. For example, if the token contains custom claims such as “name” or “email”, the values of those claims will be accessible through `$token.name` and `$token.email`.

The signature of the token is verified with method defined when creating the token. If the signature of the token is invalid, calls to SurrealDB interfaces using that token will fail.

### Root

Root tokens can be used to select, create, update, and delete on all tables in all databases of all namespaces, as well as to define and remove namespaces and databases from the SurrealDB instance.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Set the name of the token
DEFINE ACCESS token_name
  -- Use this token provider for root authentication
  ON ROOT
  -- Specify the type of access being defined
  TYPE JWT
  -- Specify the cryptographic signature algorithm used to verify the token
  ALGORITHM HS512
  -- Specify the symmetric key used to sign and verify the authenticity of the token
  KEY "sNSYneezcr8kqphfOC6NwwraUHJCVAt0XjsRSNmssBaBRh3WyMa9TRfq8ST7fsU2H2kGiOpU4GbAF1bCiXmM1b3JGgleBzz7rsrz6VvYEM4q3CLkcO8CMBIlhwhzWmy8"
;
```
The root token payload should at least include the following claims when used to authenticate with SurrealDB.

```json title="JWT Payload"
{
  "exp": 2147483647,
  "ac": "token_name",
}
```

### Namespace

Namespace tokens can be used to select, create, update, and delete on all tables in all databases, as well as to define and remove databases and tables from the namespace.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Specify the namespace for the token
USE NS abcum;

-- Set the name of the token
DEFINE ACCESS token_name
  -- Use this token provider for namespace authentication
  ON NAMESPACE
  -- Specify the type of access being defined
  TYPE JWT
  -- Specify the cryptographic signature algorithm used to verify the token
  ALGORITHM HS512
  -- Specify the symmetric key used to sign and verify the authenticity of the token
  KEY "sNSYneezcr8kqphfOC6NwwraUHJCVAt0XjsRSNmssBaBRh3WyMa9TRfq8ST7fsU2H2kGiOpU4GbAF1bCiXmM1b3JGgleBzz7rsrz6VvYEM4q3CLkcO8CMBIlhwhzWmy8"
;
```
The namespace token payload should at least include the following claims when used to authenticate with SurrealDB.

```json title="JWT Payload"
{
  "exp": 2147483647,
  "ac": "token_name",
  "ns": "abcum"
}
```

### Database

Database tokens can be used to select, create, update, and delete on all tables in a specific database, as well as to define and remove tables from the database.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

-- Set the name of the token
DEFINE ACCESS token_name
  -- Use this token provider for database authentication
  ON DATABASE
  -- Specify the type of access being defined
  TYPE JWT
  -- Specify the cryptographic signature algorithm used to verify the token
  ALGORITHM HS512
  -- Specify the symmetric key used to sign and verify the authenticity of the token
  KEY "sNSYneezcr8kqphfOC6NwwraUHJCVAt0XjsRSNmssBaBRh3WyMa9TRfq8ST7fsU2H2kGiOpU4GbAF1bCiXmM1b3JGgleBzz7rsrz6VvYEM4q3CLkcO8CMBIlhwhzWmy8"
;
```

The database token payload should at least include the following claims when used to authenticate with SurrealDB.

```json title="JWT Payload"
{
  "exp": 2147483647,
  "ac": "token_name",
  "ns": "abcum",
  "db": "app_vitalsense"
}
```

## With `AUTHENTICATE` clause

The `AUTHENTICATE` clause allows you to define a custom expression that will be executed when the token is verified. This expression will be executed in the context of the token, allowing you to perform additional checks on the token claims before the token is accepted. If the expression returns any value or throws any error, the token will be rejected.

#### Example: JWT User Authentication with Issuer and Audience Check

This example sets up additional token verification logic for a system user on a database using JSON Web Tokens (JWT) to authenticate. In this example, the HS512 algorithm is used to sign the token. The `AUTHENTICATE` block contains conditions to verify the token's validity: it checks that the issuer (`iss`) of the token is "surrealdb-test" and throws an error if it is not. Similarly, it checks that the audience of the token (defined in the `aud` claim, which can be provided either as an array of strings or a single string) includes "surrealdb-test" and throws an error if it does not. If both checks pass, the token is considered valid. The session duration is set to 2 hours.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE ACCESS user ON DATABASE TYPE JWT
ALGORITHM HS512 KEY "sNSYneezcr8kqphfOC6NwwraUHJCVAt0XjsRSNmssBaBRh3WyMa9TRfq8ST7fsU2H2kGiOpU4GbAF1bCiXmM1b3JGgleBzz7rsrz6VvYEM4q3CLkcO8CMBIlhwhzWmy8"
AUTHENTICATE {
  IF $token.iss != "surrealdb-test" { THROW "Invalid token issuer" };
  IF type::is_array($token.aud) {
    IF "surrealdb-test" NOT IN $token.aud { THROW "Invalid token audience" }
  } ELSE {
    IF $token.aud IS NOT "surrealdb-test" { THROW "Invalid token audience" }
  };
}
DURATION FOR SESSION 2h;
```


## Using `IF NOT EXISTS` clause

The `IF NOT EXISTS` clause can be used to define an access method of type JWT only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining an access method in SurrealDB if you want to ensure that the access method is only created if it does not already exist. If the access method already exists, the `DEFINE ACCESS` statement will return an error.

It's particularly useful when you want to safely attempt to define an access method without manually checking its existence first.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a JWT access method for the example database if it does not already exist
DEFINE ACCESS IF NOT EXISTS example ON DATABASE TYPE JWT ALGORITHM HS512 KEY
"sNSYneezcr8kqphfOC6NwwraUHJCVAt0XjsRSNmssBaBRh3WyMa9TRfq8ST7fsU2H2kGiOpU4GbAF1bCiXmM1b3JGgleBzz7rsrz6VvYEM4q3CLkcO8CMBIlhwhzWmy8";
```

## Using `OVERWRITE` clause

The `OVERWRITE` clause can be used to define an access method of type JWT and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing access method definition. If the access method already exists, the `DEFINE ACCESS` statement will overwrite the existing access method definition with the new one.



```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a JWT access method for the example database and overwrite it if it already exists
DEFINE ACCESS OVERWRITE example ON DATABASE TYPE JWT ALGORITHM HS512 KEY 'secret';
```


## record.mdx

---
sidebar_position: 10
sidebar_label: RECORD
title: DEFINE ACCESS ... TYPE RECORD statement | SurrealQL
description: A record access method allows accessing SurrealDB as a record user.
---

import Since from '@components/shared/Since.astro'

# `DEFINE ACCESS ... TYPE RECORD`

<Since v="v2.0.0" />

A record access method allows accessing SurrealDB as a [record user](/docs/surrealdb/security/authentication#record-users).

Record users allow SurrealDB to operate as a web database by offering mechanisms to define custom signin and signup logic as well as custom table and field permissions.

## Requirements

- You must be authenticated as a [root, namespace or database user](/docs/surrealql/statements/define/user) before you can define a record access method.
- [You must select your namespace and database](/docs/surrealql/statements/use) before you can define a record access method.

## Statement syntax

```syntax title="SurrealQL Syntax"
DEFINE ACCESS [ OVERWRITE | IF NOT EXISTS ] @name
  ON DATABASE TYPE RECORD
    [ SIGNUP @expression ]
    [ SIGNIN @expression ]
    [ WITH JWT
      [ ALGORITHM @algorithm KEY @key | URL @url ]
      [ WITH ISSUER KEY @key ]
    ]
    [ WITH REFRESH ]
  [ AUTHENTICATE @expression ]
  [ DURATION
    [ FOR TOKEN @duration ]
    [ FOR SESSION @duration ]
  ]
```

## Example usage

Below shows how you can define record access using the `DEFINE ACCESS ... TYPE RECORD` statement.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE ACCESS account ON DATABASE TYPE RECORD
	SIGNUP ( CREATE user SET email = $email, pass = crypto::argon2::generate($pass) )
	SIGNIN ( SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass) )
	DURATION FOR TOKEN 15m, FOR SESSION 12h
;
```

### With JSON Web Token

Successful authentication with a record access method results in SurrealDB generating a JSON Web Token (JWT or, in the context of SurrealDB, just "token") that can be used until its expiration to authenticate as the record user without the need of providing any additional credentials. These tokens can also be issued by third parties and trusted by SurrealDB in order to allow for the authentication process to take place outside of SurrealDB, while the resulting access claims can be provided to SurrealDB inside of a token that it can trust. This feature is provided by the `WITH JWT` clause, which behaves similarly to [the JWT access method](/docs/surrealql/statements/define/access/jwt).

Since the origin of the claims in the JWT is verified, those claims can be used within SurrealQL in order to provide table and field authorization through an external authenticator using OpenID Connect, OAuth or simply acting as a trusted issuer of a JWT. This can be done by leveraging table permissions to allow or disallow access depending on the values of the claims in the verified token. For example, these claims can be compared with the records in a table to only return those matching certain criteria.

Bear in mind that table and field permissions only apply to [record users](/docs/surrealdb/security/authentication#record-users), which must use tokens that are verified by a `RECORD` access method. Access provided by namespace and database tokens defined in a `JWT` access method is equivalent to access from [system users](/docs/surrealdb/security/authentication#system-users), which is above fine-grained permissions. When application users will be the ones directly authenticating with JWT, defining a `RECORD` access method `WITH JWT` is most likely the right choice.

Reference [the JWT access method](/docs/surrealql/statements/define/access/jwt) documentation for additional information about how JWT tokens can be used in SurrealDB, including verification through [JWKS](/docs/surrealql/statements/define/access/jwt#json-web-key-set-jwks).

The following example shows how record access with a token can be used to grant authorization either by verifying that the `id` claim in the token (which is used to populate the [`$auth`](/docs/surrealql/parameters#auth) reserved parameter) matches the record that is being queried from the `user` table or if the `privileged` claim is set to `true` in the token:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Specify the namespace and database for the token
USE NS abcum DB app_vitalsense;

DEFINE ACCESS token_name ON DATABASE TYPE RECORD WITH JWT
ALGORITHM RS256 KEY "-----BEGIN PUBLIC KEY-----
MUO52Me9HEB4ZyU+7xmDpnixzA/CUE7kyUuE0b7t38oCh+sQouREqIjLwgHhFdhh3cQAwr6GH07D
ThioYrZL8xATJ3Youyj8C45QnZcGUif5PkpWXDi0HJSoMFekbW6Pr4xuqIqb2LGxGDVJcLZwJ2AS
Gtu2UAfPXbBD3ffiad393M22g1iHM80YaNi+xgswG7qtXE4lR/Lt4s0MeKKX7stdWI1VIsoB+y3i
r/OWUvJPjjDNbAsyy8tQmxydv+FUnLEP9TNT4AhN4DXcJ+XsDtW7OWt4EdSVDeKpGbIMvIrh1Pe+
Nilj8UHNyNDHa2AjK3seMo6CMvaIQJKj5o4xGFblFGwvvPD03SbuQLs1FdRjsZCeWLdYeQ3JDHE9
sFG7DCXlpMJcaYT1mf4XHJ0gPekNLQyewTY3Vxf7FgV3GCNjV20kcDFgJA2+iVW2wSrb+txD1ycE
kbi8jh0pedWwE40VQWaTh/8eAvX7IHWya/AEro25mq+m6vktNZLbvLphhp586kJK3Tdt3YjpkPre
M3nkFWOWurIyKbtIV9JemfwCgt89sNV45dTlnEDEZFFGnIgDnWgx3CUo4XmhICEQU8+tklw9jJYx
iCTjhbIDEBHySSSc/pQ4ftHQmhToTlQeOdEy4LYiaEIgl1X+hzRH1hBYvWlNKe4EY1nMCKcjgt0=
-----END PUBLIC KEY-----";

DEFINE TABLE user SCHEMAFULL
  -- Authorized users can select, update, delete and create user records
  PERMISSIONS FOR select, update, delete, create
  -- The access method must be "users"
  WHERE $access = "users"
  -- The record of the user being queried must match the one identified in the token
  -- Only matching records will be changed or returned
  AND id = $auth.id
  -- Allow privileged tokens to query any user
  OR $token.privileged = true
;
```

You may also use permissions clauses to perform additional verification on other JWT claims that may be required or recommended by the provider of the token, such as verifying that the `iss` claim matches a specific principal using `$token.iss`. However, this kind of logic may be better suited for the [`AUTHENTICATE`](#with-authenticate-clause) clause, which is only executed when the token is validated before an authenticated session is established instead of in every query and for each record.

The token payload should at least include the following claims when used to authenticate as a record user in SurrealDB.

```json title="JWT Payload"
{
  "exp": 2147483647,
  "ns": "abcum",
  "db": "app_vitalsense",
  "ac": "users",
  "id": "user:1"
}
```

When the `id` claim is present in the token, the fields of the record matching the identifier specified will be accessible through the `$auth` reserved parameter. For example, if the value of the `id` claim is `user:73q1bl039y6k8z80v55d`, and user records have fields such as “name” or “email”, then `$auth.name` and `$auth.email` can be used to access those values for the `user:73q1bl039y6k8z80v55d` record specifically, without them being present in the JWT.

#### With Issuer

When explicitly defining a way to verify tokens for record access, it is also possible to customize how these tokens are issued by SurrealDB. This allows specifying the algorithm and the signing key, which otherwise default to the HS512 algorithm with a randomly generated 128-character alphanumeric key. Configuring a record access method to sign tokens with specific signing credentials allows third party services to trust tokens issued by SurrealDB by trusting those signing credentials. In this way, an external service may rely on the signup and signin logic that has been implemented for record users in SurrealDB for its own authentication.

Currently, the algorithm for the issuer and the verifier are required to match. For this reason, the issuer algorithm can be omitted if it has already been defined in the `WITH JWT` clause. Likewise, an issuer does not need to be explicitly defined in the case where a key to verify JWT using a symmetric algorithm has already been defined in the `WITH JWT` clause, as the same key will also be used to sign the tokens.

The following is an example of defining a record access method that can issue tokens with an asymmetric key pair:

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE ACCESS token_name ON DATABASE TYPE RECORD WITH JWT
ALGORITHM RS256
  KEY "-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAu1SU1LfVLPHCozMxH2Mo
4lgOEePzNm0tRgeLezV6ffAt0gunVTLw7onLRnrq0/IzW7yWR7QkrmBL7jTKEn5u
+qKhbwKfBstIs+bMY2Zkp18gnTxKLxoS2tFczGkPLPgizskuemMghRniWaoLcyeh
kd3qqGElvW/VDL5AaWTg0nLVkjRo9z+40RQzuVaE8AkAFmxZzow3x+VJYKdjykkJ
0iT9wCS0DRTXu269V264Vf/3jvredZiKRkgwlL9xNAwxXFg0x/XFw005UWVRIkdg
cKWTjpBP2dPwVZ4WWC+9aGVd+Gyn1o0CLelf4rEjGoXbAAEgAqeGUxrcIlbjXfbc
mwIDAQAB
-----END PUBLIC KEY-----"
  WITH ISSUER KEY "-----BEGIN PRIVATE KEY-----
MIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQC7VJTUt9Us8cKj
MzEfYyjiWA4R4/M2bS1GB4t7NXp98C3SC6dVMvDuictGeurT8jNbvJZHtCSuYEvu
NMoSfm76oqFvAp8Gy0iz5sxjZmSnXyCdPEovGhLa0VzMaQ8s+CLOyS56YyCFGeJZ
qgtzJ6GR3eqoYSW9b9UMvkBpZODSctWSNGj3P7jRFDO5VoTwCQAWbFnOjDfH5Ulg
p2PKSQnSJP3AJLQNFNe7br1XbrhV//eO+t51mIpGSDCUv3E0DDFcWDTH9cXDTTlR
ZVEiR2BwpZOOkE/Z0/BVnhZYL71oZV34bKfWjQIt6V/isSMahdsAASACp4ZTGtwi
VuNd9tybAgMBAAECggEBAKTmjaS6tkK8BlPXClTQ2vpz/N6uxDeS35mXpqasqskV
laAidgg/sWqpjXDbXr93otIMLlWsM+X0CqMDgSXKejLS2jx4GDjI1ZTXg++0AMJ8
sJ74pWzVDOfmCEQ/7wXs3+cbnXhKriO8Z036q92Qc1+N87SI38nkGa0ABH9CN83H
mQqt4fB7UdHzuIRe/me2PGhIq5ZBzj6h3BpoPGzEP+x3l9YmK8t/1cN0pqI+dQwY
dgfGjackLu/2qH80MCF7IyQaseZUOJyKrCLtSD/Iixv/hzDEUPfOCjFDgTpzf3cw
ta8+oE4wHCo1iI1/4TlPkwmXx4qSXtmw4aQPz7IDQvECgYEA8KNThCO2gsC2I9PQ
DM/8Cw0O983WCDY+oi+7JPiNAJwv5DYBqEZB1QYdj06YD16XlC/HAZMsMku1na2T
N0driwenQQWzoev3g2S7gRDoS/FCJSI3jJ+kjgtaA7Qmzlgk1TxODN+G1H91HW7t
0l7VnL27IWyYo2qRRK3jzxqUiPUCgYEAx0oQs2reBQGMVZnApD1jeq7n4MvNLcPv
t8b/eU9iUv6Y4Mj0Suo/AU8lYZXm8ubbqAlwz2VSVunD2tOplHyMUrtCtObAfVDU
AhCndKaA9gApgfb3xw1IKbuQ1u4IF1FJl3VtumfQn//LiH1B3rXhcdyo3/vIttEk
48RakUKClU8CgYEAzV7W3COOlDDcQd935DdtKBFRAPRPAlspQUnzMi5eSHMD/ISL
DY5IiQHbIH83D4bvXq0X7qQoSBSNP7Dvv3HYuqMhf0DaegrlBuJllFVVq9qPVRnK
xt1Il2HgxOBvbhOT+9in1BzA+YJ99UzC85O0Qz06A+CmtHEy4aZ2kj5hHjECgYEA
mNS4+A8Fkss8Js1RieK2LniBxMgmYml3pfVLKGnzmng7H2+cwPLhPIzIuwytXywh
2bzbsYEfYx3EoEVgMEpPhoarQnYPukrJO4gwE2o5Te6T5mJSZGlQJQj9q4ZB2Dfz
et6INsK0oG8XVGXSpQvQh3RUYekCZQkBBFcpqWpbIEsCgYAnM3DQf3FJoSnXaMhr
VBIovic5l0xFkEHskAjFTevO86Fsz1C2aSeRKSqGFoOQ0tmJzBEs1R6KqnHInicD
TQrKhArgLXX4v3CddjfTRJkFWDbE/CkvKZNOrcf1nhaGCPspRJj2KUkj1Fhl9Cnc
dn/RsYEONbwQSjIfMPkvxF+8HQ==
-----END PRIVATE KEY-----"
;
```

The issuer is implicitly defined when using a symmetric algorithm in `WITH JWT`:

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE ACCESS token_name ON DATABASE TYPE RECORD WITH JWT
-- Symmetric algorithm with a symmetric key
-- The same key is used to sign and verify
ALGORITHM HS512 KEY "secret";
-- The following clause is implicit:
-- WITH ISSUER ALGORITHM HS512 KEY "secret"
```

### With refresh token

> [!CAUTION]
> Currently, the `WITH REFRESH` clause is an experimental feature intended to be used for validating its suitability and security. As such, it may be subject to breaking changes and may present unidentified security issues. Do not rely on this feature in production applications.

> [!NOTE]
> Due to changes required in the RPC API and the SDKs, refresh tokens are currently only available when signing up and in through the [HTTP REST API](/docs/surrealdb/integration/http).

Defining a record access method `WITH REFRESH` will result in an additional [bearer key](/docs/surrealql/statements/define/access/bearer) for the record user being returned after successful authentication with the access method. This bearer key is intended to be used as a "refresh token", which is a concept commonly found in standards such as [OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749#section-1.5).

Unlike authentication tokens (i.e. JWT), refresh tokens (i.e. bearer keys) feature randomly generated opaque strings that contain no authentication information by themselves, but rather a pointer to an access grant that is stored in the datastore. Also unlike authentication tokens, bearer keys such as refresh tokens can be [audited](/docs/surrealql/statements/access/#show) and [revoked](/docs/surrealql/statements/access/#revoke) using the [`ACCESS`](/docs/surrealql/statements/access) statement. Refresh tokens are automatically revoked and replaced by a new refresh token whenever used to obtain an authentication token, reducing the time window for exploiting a compromised refresh token. These additional security guarantees allow refresh tokens to be longer-lived than authentication tokens, which in turn encourages making the original authentication tokens as short-lived as technically possible.

By default, refresh tokens will expire after 30 days. However, their duration can be configured using the `DURATION FOR GRANT` clause, which will accept any duration. If set to `NONE`, refresh tokens will never expire. It is strongly recommended to set some expiration for refresh tokens to minimize the potential impact of credential stealing attacks.

Because refresh tokens can be used to indefinitely keep a user authenticated with SurrealDB as long as they are exchanged for a new fresh token before they expire, [special care](/docs/surrealdb/security/security-best-practices#token-storage) should be taken when storing and applications using them should be suitably protected from attacks.

Like other bearer keys, all refresh tokens are stored in the datastore even after they are expired or revoked. This means that using refresh tokens will have a space cost in addition to the performance cost of retrieving and verifying them against the datastore. Refresh tokens are intended to be used only to obtain a new authentication token after the existing one expires and applications should only use them when necessary, such as after receiving a [token expiration error](/docs/surrealdb/security/troubleshooting#token-expired-error). For certain high volume applications, you may want to regularly [purge](/docs/surrealql/statements/access/#purge) expired refresh tokens to minimize the space used by inactive refresh tokens.

For more information on how to manage existing refresh tokens, see the [`ACCESS`](/docs/surrealql/statements/access) statement.

#### Example: Signing in with a refresh token

Define a record access method `WITH REFRESH`:

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE ACCESS user ON DATABASE TYPE RECORD
	SIGNUP ( CREATE user SET email = $email, pass = crypto::argon2::generate($pass) )
	SIGNIN ( SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass) )
	WITH REFRESH
	DURATION FOR GRANT 15d, FOR TOKEN 1m, FOR SESSION 12h
;
```

Sign up with a new user or sign in with an existing user via the [HTTP REST API](/docs/surrealdb/integration/http):

```bash
curl -X POST \
	-H "Accept: application/json" \
	-d '{"NS":"test", "DB":"test", "AC":"user", "name":"John Doe", "email":"john.doe@example.com", "pass":"VerySecurePassword!"}' \
	http://localhost:8000/signup
```

```json title="Response"
{
	"code":200,
	"details":"Authentication succeeded",
	"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE3MzQ1MTkyODIsIm5iZiI6MTczNDUxOTI4MiwiZXhwIjoxNzM0NTE5MzQyLCJpc3MiOiJTdXJyZWFsREIiLCJqdGkiOiJiYzQ3MzhkOS0zMTM3LTQ1ZjMtOGUzMy1jMmJmODI0MzZlZTciLCJOUyI6InRlc3QiLCJEQiI6InRlc3QiLCJBQyI6InVzZXIiLCJJRCI6InVzZXI6dHZ2NWVreXNscjBsb21sNHp4aTkifQ.liEvoYuxk9EgzqBE5MzyG2IaJTJxazz-aD9vqWPGc5AGL2u0H0gggjX3jpcaBAIyU356wxNaxFrvCoTqaA4Vrg",
	"refresh":"surreal-refresh-UgYUNmB3FR8t-zdTZlFNuvdoWOtKe0Aqb1laH"
}
```

Sign in with the refresh token to obtain a new token and refresh token:

```bash
curl -X POST \
	-H "Accept: application/json" \
	-d '{"NS":"test", "DB":"test", "AC":"user", "refresh":"surreal-refresh-UgYUNmB3FR8t-zdTZlFNuvdoWOtKe0Aqb1laH"}' \
	http://localhost:8000/signin
```

```json title="Response"
{
	"code":200,
	"details":"Authentication succeeded",
	"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE3MzQ1MTkzNzcsIm5iZiI6MTczNDUxOTM3NywiZXhwIjoxNzM0NTE5NDM3LCJpc3MiOiJTdXJyZWFsREIiLCJqdGkiOiJjN2Q1YjgzYi0yMjJjLTQ2ODYtYjgzYi01ZWVlNDQ5Njk5YmUiLCJOUyI6InRlc3QiLCJEQiI6InRlc3QiLCJBQyI6InVzZXIiLCJJRCI6InVzZXI6dHZ2NWVreXNscjBsb21sNHp4aTkifQ.usM8aMtqAftJcwhUMdmqskr-k58ARF-KbmCYEQuDoGb5PlhVJDwYEwCb0oV8B85MJPvbKlC6HuFKW2wq6-AY9g",
	"refresh":"surreal-refresh-MPKzHBtznxMa-pFJj2Doj2IRHApIzGmeOAcYo"
}
```

### With `AUTHENTICATE` clause

In the context of [`DEFINE ACCESS ... TYPE RECORD`](/docs/surrealql/statements/define/access/record),the authenticate clause can be used to change the record identifier returned by the `SIGNIN` and `SIGNUP` clauses or replace the identifier provided in the token when authenticating `WITH JWT`.

This clause can also be used to log or stop authentication attempts from record users, as it is always executed across signin, signup and token authentication.

Unlike the [`PERMISSIONS`](/docs/surrealql/statements/define/table#defining-permissions) clause, the `AUTHENTICATE` clause is executed only at the time of authentication, resulting in increased performance for queries that only need to be validated at that point.

On the other hand, permissions queries are executed in every query and for each record, ensuring that any authorization conditions are verified at the time of the query. The `AUTHENTICATE` clause is a good fit for validating specific conditions that are not expected to change during the lifetime of the session such as the presence of any required token claims.

#### Example: External authentication providers

Replacing the record identifier that will be used to establish the session is especially useful in scenarios where the token used to authenticate the session does not contain one. This is common when using an external authentication provider, which may only have knowledge of generic user identifiers such as an email address or UUID.

In the below example, we check if the session may already be tied to an existing user by using the `$auth` reserved parameter, which contains the record identifier of the authenticated user. If we can select the `id` field from `$auth`, it means that the token already contained the `id` for a record that exists in the database. If that is not the case, we can check if the token contains a different claim that we can rely on to uniquely identifies users. In this case we check for an email address. If the `email` claim is present in the token, we try to retrieve the user from the `user` table by their email address. If none of the queries return a record, the `AUTHENTICATE` clause will fail with a generic error. You can also choose to `THROW` a custom error as shown in the next example.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE ACCESS user ON DATABASE TYPE RECORD
    WITH JWT ALGORITHM HS512 KEY 'secret'
    AUTHENTICATE {
        IF $auth.id {
            RETURN $auth.id;
        } ELSE IF $token.email {
            RETURN SELECT * FROM user WHERE email = $token.email;
        };
    }
;
```

#### Example: Failing authentication

Because the `AUTHENTICATE` clause is always executed across signin, signup and token authentication, it is in a unique position to centralize logic after user credentials are deemed valid, but before the user is completely authenticated.

Below, we show an example of validating if a user is enabled. If this is not the case, we can `THROW` an error stating why the user cannot authenticate, stopping the authentication attempt with a custom message. You can also choose to not return anything, which results in a generic authentication error. If the user is enabled, we can `RETURN` the record identifier, which confirms that authentication is successful and specifies that the record user which will be authenticated in the session is the same that was already authenticated.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE ACCESS user ON DATABASE TYPE RECORD
    SIGNUP ( CREATE user SET email = $email, pass = crypto::argon2::generate($pass), enabled = true )
    SIGNIN ( SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass) )
    AUTHENTICATE {
        IF !$auth.enabled {
            THROW "This user is not enabled";
        };

        RETURN $auth;
    }
;
```

#### Example: Auditing and revoking tokens

In addition to what is shown in the previous example, the `AUTHENTICATE` clause can also be used to create records and access the claims found in the token itself using the `$token` reserved parameter. These features can be combined to log authentication attempts and stop authentication from completing if some conditions are met.

Below, we show a proof of concept example that leverages the fact that tokens issued by SurrealDB have the standard `jti` claim, which contains a randomly generated unique identifier for the token. This value can be used to uniquely identify each token that is issued by SurrealDB for the purposes of auditing and revocation.

In this example, we create a new record in the `token` table for each token that SurrealDB issues after a successful `SIGNIN` and `SIGNUP`. This record is identified by the value in the `jti`(JWT ID) claim. Every time that a token is used to authenticate, we check if the record in the `token` table with identifier matching the `jti`(JWT ID) claim has been revoked and, if so, we fail authentication with a custom message. Otherwise, we log the time that the token was used to successfully authenticate in the `audit` table and continue authentication without changes.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

DEFINE ACCESS user ON DATABASE TYPE RECORD
    SIGNUP ( CREATE user SET email = $email, pass = crypto::argon2::generate($pass) )
    SIGNIN ( SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass) )
    AUTHENTICATE {
        IF type::record("token", $token.jti).revoked = true {
            THROW "This token has been revoked";
        };
        INSERT INTO token { id: $token.jti, exp: $token.exp, revoked: false };
        CREATE audit CONTENT { token: $token.jti, time: time::now() };
        RETURN $auth;
    }
    DURATION FOR TOKEN 30d, FOR SESSION 1h
;
```

## Using `IF NOT EXISTS` clause

The `IF NOT EXISTS` clause can be used to define an access method of type RECORD only if it does not already exist. You should use the `IF NOT EXISTS` clause when defining an access method in SurrealDB if you want to ensure that the access method is only created if it does not already exist. If the access method already exists, the `DEFINE ACCESS` statement will return an error.

It's particularly useful when you want to safely attempt to define an access method without manually checking its existence first.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a RECORD access method for the example database if it does not already exist
DEFINE ACCESS IF NOT EXISTS example ON DATABASE TYPE RECORD;
```

## Using `OVERWRITE` clause

The `OVERWRITE` clause can be used to define an access method of type RECORD and overwrite an existing one if it already exists. You should use the `OVERWRITE` clause when you want to modify an existing access method definition. If the access method already exists, the `DEFINE ACCESS` statement will overwrite the existing access method definition with the new one.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Create a RECORD access method for the example database and overwrite if it already exists
DEFINE ACCESS OVERWRITE example ON DATABASE TYPE RECORD;
```


## database.mdx

---
sidebar_position: 2
sidebar_label: DATABASE
title: ALTER DATABASE statement | SurrealQL
description: The ALTER statement can be used to change authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `ALTER DATABASE` statement

<Since v="v3.0.0" />

The `ALTER DATABASE` statement can be used to modify the database. `ALTER DATABASE` is used on the current database, which is why a `IF EXISTS` clause does not exist.

## Statement syntax

<Tabs syncKey="alter-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
ALTER DATABASE COMPACT
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const alterAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "ALTER DATABASE" },
      { type: "Terminal", text: "COMPACT" },
    ]}
  ]
};


<RailroadDiagram ast={alterAst} className="my-6" />

  </TabItem>
</Tabs>

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

## field.mdx

---
sidebar_position: 3
sidebar_label: FIELD
title: ALTER FIELD statement | SurrealQL
description: The ALTER statement can be used to change authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `ALTER FIELD` statement

The `ALTER FIELD` statement is used to change or entirely drop clauses of a defined field on a table.

## Statement syntax

<Tabs syncKey="alter-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
ALTER FIELD [ IF EXISTS ] ON [ TABLE ] @table 
[ 
    DROP TYPE |
    DROP FLEXIBLE |
    DROP READONLY |
    DROP VALUE |
    DROP ASSERT |
    DROP DEFAULT |
    DROP COMMENT |
    DROP REFERENCE |
    FLEXIBLE |
    READONLY |
    REFERENCE |
    TYPE @type |
    VALUE @value |
    ASSERT @expression |
    DEFAULT [ ALWAYS ] @expression |
    [ PERMISSIONS [ NONE | FULL
		| FOR select @expression
		| FOR create @expression
		| FOR update @expression
		| FOR delete @expression
	] ]
    COMMENT @string |
]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const alterAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "ALTER FIELD" },
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "IF" },
              { type: "Terminal", text: "EXISTS" },
            ],
          },
        },
        {
          type: "OneOrMore",
          child: {
            type: "Choice",
            index: 0,
            children: [
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "DROP" },
                  {
                    type: "Choice",
                    index: 0,
                    children: [
                      { type: "Terminal", text: "TYPE" },
                      { type: "Terminal", text: "FLEXIBLE" },
                      { type: "Terminal", text: "READONLY" },
                      { type: "Terminal", text: "VALUE" },
                      { type: "Terminal", text: "ASSERT" },
                      { type: "Terminal", text: "DEFAULT" },
                      { type: "Terminal", text: "COMMENT" },
                      { type: "Terminal", text: "REFERENCE" },
                    ],
                  },
                ],
              },

              { type: "Terminal", text: "FLEXIBLE" },
              { type: "Terminal", text: "READONLY" },
              { type: "Terminal", text: "REFERENCE" },

              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "TYPE" },
                  { type: "Terminal", text: "@type" },
                ],
              },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "VALUE" },
                  { type: "Terminal", text: "@value" },
                ],
              },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "ASSERT" },
                  { type: "Terminal", text: "@expression" },
                ],
              },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "DEFAULT" },
                  { type: "Optional", child: { type: "Terminal", text: "ALWAYS" } },
                  { type: "Terminal", text: "@expression" },
                ],
              },

              {
                type: "Optional",
                child: {
                  type: "Sequence",
                  children: [
                    { type: "Terminal", text: "PERMISSIONS" },
                    {
                      type: "Choice",
                      index: 1,
                      children: [
                        { type: "Terminal", text: "NONE" },
                        { type: "Terminal", text: "FULL" },
                        {
                          type: "Sequence",
                          children: [
                            { type: "Terminal", text: "WHERE" },
                            { type: "NonTerminal", text: "@condition" },
                          ],
                        },
                      ],
                    },
                  ],
                },
              },

              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "COMMENT" },
                  { type: "Terminal", text: "@string" },
                ],
              },
            ],
          },
          repeat: { type: "Skip" },
        },
      ],
    },
  ],
};


<RailroadDiagram ast={alterAst} className="my-6" />

  </TabItem>
</Tabs>


## Examples

As `ALTER FIELD` contains the same clauses available in a [`DEFINE FIELD`](/docs/surrealql/statements/define/field) statement, be sure to see that page for more examples.

Here is one example in which the `name` field is defined for a record `user`:

```surql
DEFINE FIELD name ON user TYPE string;
```

Later on, a database-wide [parameter](/docs/surrealql/statements/define/param) is defined to disallow certain user names. This can be followed up with an `ALTER FIELD` statement to add the `ASSERT` clause to it.

```surql
DEFINE PARAM $DISALLOWED_NAMES VALUE ["Lord British", "Lord Blackthorn"];
ALTER FIELD name ON user ASSERT $value NOT IN $DISALLOWED_NAMES;
CREATE user SET name = "Lord British";
```

```surql title="Output"
"Found 'Lord British' for field `name`, with record `user:yn4yttkg5w683q2937bq`, but field must conform to: $value NOTINSIDE $DISALLOWED_NAMES""
```

## See also

* [`DEFINE FIELD`](/docs/surrealql/statements/define/field)

## index.mdx

---
sidebar_position: 1
sidebar_label: ALTER
title: ALTER statement | SurrealQL
description: The ALTER statement can be used to change authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---
import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `ALTER` statement

<Since v="v2.0.0" />

The `ALTER` statement can be used to change the behaviour of database resources.

There are two main cases in which to use an `ALTER` statement:

* Modifying previously defined resources. This can currently be used to modify tables and fields. For other such modifications, use the `OVERWRITE` clause in other `DEFINE` statements.
* Modifying other resources using clauses not present in other `DEFINE` statements. Examples of this are the `PREPARE REMOVE` clause to prepare an index for removal, the `COMPACT` clause to compact the system/namespace/database/single table, and the `QUERY_TIMEOUT` clause to define or drop the query timeout for the entire datastore.

Some examples of `ALTER` statements are as follows. For further details, see the individual pages for each type of `ALTER` statement.

## Modify a table schema

When starting a new project, you may require a table to be schemaless to allow for flexibility in the data structure. However, as the project progresses, you may want to lock down the schema to prevent new fields from being added.

An example of `ALTER` to modify an existing table:

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ id: user:317esn5r3spd4em0lfze, name: 'LordofSalty' }]"
skip-record-id-key = true

[[test.results]]
value = "NONE"

*/

DEFINE TABLE user SCHEMALESS;
DEFINE FIELD name ON TABLE user TYPE string;
CREATE user SET name = "LordofSalty";

-- Now make it schemafull to ensure that no other fields can be used
ALTER TABLE user SCHEMAFULL;
```

## Modify table permissions

You can also use the `ALTER` statement to change a table's permissions. An `ALTER` statement only needs to include the items to be altered, not the entire definition.

```surql
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "NONE"

*/

-- Will show up as DEFINE TABLE user TYPE ANY SCHEMAFULL PERMISSIONS NONE
DEFINE TABLE user SCHEMAFULL;

-- Now defined as DEFINE TABLE user TYPE ANY SCHEMAFULL PERMISSIONS FULL
ALTER TABLE user PERMISSIONS FOR create FULL;
```

## Using `IF EXISTS` clause

You can use the' IF EXISTS' clause to prevent an error from occurring when trying to alter a table that does not exist.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

ALTER TABLE IF EXISTS user SCHEMAFULL;
```

## indexes.mdx

---
sidebar_position: 4
sidebar_label: INDEX
title: ALTER INDEX statement | SurrealQL
description: The ALTER statement can be used to change authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `ALTER INDEX` statement

<Since v="v3.0.0" />

The `ALTER INDEX` statement is used to alter a defined index on a table.

<Tabs syncKey="alter-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
ALTER INDEX @name ON TABLE @table
    COMMENT @string |
    PREPARE REMOVE |
    DROP COMMENT
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const alterIndexAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "ALTER INDEX" },
        { type: "NonTerminal", text: "@name" },
        { type: "Terminal", text: "ON TABLE" },
        { type: "NonTerminal", text: "@table" },
        {
          type: "Choice",
          index: 0,
          children: [
            {
              type: "Sequence",
              children: [
                { type: "Terminal", text: "COMMENT" },
                { type: "NonTerminal", text: "@string" },
              ],
            },
            {
              type: "Sequence",
              children: [
                { type: "Terminal", text: "PREPARE" },
                { type: "Terminal", text: "REMOVE" },
              ],
            },
            {
              type: "Sequence",
              children: [
                { type: "Terminal", text: "DROP" },
                { type: "Terminal", text: "COMMENT" },
              ],
            },
          ],
        },
      ],
    },
  ],
};

<RailroadDiagram ast={alterIndexAst} className="my-6" />

  </TabItem>
</Tabs>

## `PREPARE REMOVE` clause

As the name implies, an `ALTER INDEX PREPARE REMOVE` statement alters an index to prepare it for removal. This statement sets up a step in which the index has been decommissioned (prepared for removal), but not yet removed. At this point, `SELECT` queries along with the `EXPLAIN` clause to monitor query performance without the index.

```surql
-- 1. Decommission the index
ALTER INDEX my_index ON my_table PREPARE REMOVE;

-- 2. Monitor query performance and verify queries still work
SELECT ... FROM my_table EXPLAIN;

-- 3. If satisfied, permanently remove the index
REMOVE INDEX my_index ON my_table;
```

If removing the index is no longer desired, it can be restored to a useful state by using a [REBUILD INDEX](/docs/surrealql/statements/rebuild) statement.

```surql
REBUILD INDEX my_index ON my_table;
```

## See also

* [`DEFINE INDEX`](/docs/surrealql/statements/define/indexes)

## namespace.mdx

---
sidebar_position: 5
sidebar_label: NAMESPACE
title: ALTER NAMESPACE statement | SurrealQL
description: The ALTER statement can be used to change authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `ALTER NAMESPACE` statement

<Since v="v3.0.0" />

The `ALTER NAMESPACE` statement can be used to modify the namespace. `ALTER NAMESPACE` is used on the current namespace, which is why a `IF EXISTS` clause does not exist.

## Statement syntax

<Tabs syncKey="alter-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
ALTER NAMESPACE COMPACT
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const alterAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "ALTER NAMESPACE" },
      { type: "Terminal", text: "COMPACT" },
    ]}
  ]
};

<RailroadDiagram ast={alterAst} className="my-6" />

  </TabItem>
</Tabs>

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

## sequence.mdx

---
sidebar_position: 6
sidebar_label: SEQUENCE
title: ALTER SEQUENCE statement | SurrealQL
description: The ALTER statement can be used to change authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `ALTER SEQUENCE` statement

<Since v="v3.0.0" />

The `ALTER SEQUENCE` statement is used to modify a defined sequence.

<Tabs syncKey="alter-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
ALTER SEQUENCE [ IF EXISTS ] @name [ TIMEOUT @duration ]
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const alterAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "ALTER SEQUENCE" },
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "IF" },
              { type: "Terminal", text: "EXISTS" },
            ],
          },
        },
        { type: "NonTerminal", text: "@name" },
        {
          type: "Optional",
          child: {
            type: "Sequence",
            children: [
              { type: "Terminal", text: "TIMEOUT" },
              { type: "NonTerminal", text: "@duration" },
            ],
          },
        },
      ],
    },
  ],
};

<RailroadDiagram ast={alterAst} className="my-6" />

  </TabItem>
</Tabs>

## Examples

The timeout of a sequence can be modified via an `ALTER SEQUENCE` statement. For example, a sequence can be included in the schema but effectively disabled if given a timeout of 0ns, after which `ALTER SEQUENCE` can be used to modify the timeout to make it available.

```surql
DEFINE SEQUENCE mySeq3 BATCH 1000 START 100 TIMEOUT 0ns;
INFO FOR DB.sequences;
sequence::nextval('mySeq3');

ALTER SEQUENCE mySeq3 TIMEOUT 100ms;
sequence::nextval('mySeq3');
```

```surql title="Output"
-------- Query --------
{ mySeq3: 'DEFINE SEQUENCE mySeq3 BATCH 1000 START 100 TIMEOUT 0ns' },

-------- Query --------
'Thrown error: The query was not executed because it exceeded the timeout: 0ns'

-------- Query --------
100
```

## See also

* [`DEFINE SEQUENCE`](/docs/surrealql/statements/define/sequence)

## system.mdx

---
sidebar_position: 7
sidebar_label: SYSTEM
title: ALTER SYSTEM statement | SurrealQL
description: The ALTER statement can be used to change authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `ALTER SYSTEM` statement

<Since v="v3.0.0" />

The `ALTER SYSTEM` statement is used to alter the entire datastore. It can be used to compact the system, or to set or drop a systemwide query timeout.

This statement is the only `ALTER` statement that does not have a corresponding `DEFINE` statement.

## Statement syntax

<Tabs syncKey="alter-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
ALTER SYSTEM 
    COMPACT |
    QUERY_TIMEOUT |
    DROP QUERY_TIMEOUT
```

  </TabItem>
  <TabItem label="Railroad Diagram">

export const alterAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    { type: "Sequence", children: [
      { type: "Terminal", text: "ALTER SYSTEM" },
      { type: "Choice", index: 0, children: [
        { type: "Terminal", text: "COMPACT" },
        { type: "Sequence", children: [ { type: "Terminal", text: "QUERY_TIMEOUT" }, { type: "NonTerminal", text: "@duration" } ] },
        { type: "Terminal", text: "DROP QUERY_TIMEOUT" }
      ]},
    ]}
  ]
};

<RailroadDiagram ast={alterAst} className="my-6" />

  </TabItem>
</Tabs>

## `QUERY_TIMEOUT` clause

A query timeout can be set for the system as a whole. The minimum possible timeout is one millisecond, below which the value will be set as `NONE`.

```surql
ALTER SYSTEM QUERY_TIMEOUT 100ns;
INFO FOR ROOT.config;
```

```surql title="Output"
{ QUERY_TIMEOUT: NONE }
```

Any value above `1ms` will set the timeout, beyond which no query that takes any longer than this will succeed.

```surql
ALTER SYSTEM QUERY_TIMEOUT 1ms;
FOR $_ IN 0..1000 {
    FOR $_ IN 0..1000 {
        CREATE |person:1000|;
    }
};

-- 'The query was not executed because it exceeded the timeout: 1ms'
```

## COMPACT clause

Compacts the entire datastore. To compact other resources, use [ALTER NAMESPACE](/docs/surrealql/statements/alter/namespace) to compact the current namespace keyspace, [ALTER DATABASE](/docs/surrealql/statements/alter/database) to compact the current database keyspace, or [ALTER TABLE](/docs/surrealql/statements/alter/table) to compact a specific table keyspace.

The actual compaction used will depend on the datastore, such as RocksDB or SurrealKV.

This clause will not work with in-memory storage which has nothing persistent to compact, producing the following error:

```surql
'The storage layer does not support compaction requests.'
```

A successful compaction will return `NONE`.

```surql
ALTER SYSTEM COMPACT;
-- NONE
```

## table.mdx

---
sidebar_position: 8
sidebar_label: TABLE
title: ALTER TABLE statement | SurrealQL
description: The ALTER statement can be used to change authentication access and behaviour, global parameters, table configurations, table events, schema definitions, and indexes.
---

import Since from '@components/shared/Since.astro'
import RailroadDiagram from '@components/RailroadDiagram.astro'
import Tabs from '@components/Tabs/Tabs.astro'
import TabItem from '@components/Tabs/TabItem.astro'

# `ALTER TABLE` statement

The `ALTER TABLE` statement is used to alter a defined table.

<Tabs syncKey="alter-statement">
  <TabItem label="SurrealQL Syntax">

```syntax title="SurrealQL Syntax"
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

  </TabItem>
  <TabItem label="Railroad Diagram">

export const alterAst = {
  type: "Diagram",
  padding: [10, 20, 10, 20],
  children: [
    {
      type: "Sequence",
      children: [
        { type: "Terminal", text: "ALTER TABLE" },
        { type: "Optional", child: { type: "Sequence", children: [ { type: "Terminal", text: "IF" }, { type: "Terminal", text: "EXISTS" } ] } },
        { type: "Terminal", text: "@table" },

        {
          type: "OneOrMore",
          child: {
            type: "Choice",
            index: 0,
            children: [
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "DROP" },
                  {
                    type: "Choice",
                    index: 0,
                    children: [
                      { type: "Terminal", text: "COMMENT" },
                      { type: "Terminal", text: "CHANGEFEED" },
                    ],
                  },
                ],
              },

              { type: "Terminal", text: "COMPACT" },

              {
                type: "Optional",
                child: {
                  type: "Sequence",
                  children: [
                    { type: "Terminal", text: "TYPE" },
                    {
                      type: "Choice",
                      index: 1,
                      children: [
                        { type: "Terminal", text: "ANY" },
                        { type: "Terminal", text: "NORMAL" },
                        {
                          type: "Sequence",
                          children: [
                            { type: "Terminal", text: "RELATION" },
                            {
                              type: "Optional",
                              child: {
                                type: "Sequence",
                                children: [
                                  {
                                    type: "Choice",
                                    index: 1,
                                    children: [
                                      { type: "Terminal", text: "IN" },
                                      { type: "Terminal", text: "FROM" },
                                    ],
                                  },
                                  { type: "NonTerminal", text: "@table" },
                                ],
                              },
                            },
                            {
                              type: "Optional",
                              child: {
                                type: "Sequence",
                                children: [
                                  {
                                    type: "Choice",
                                    index: 1,
                                    children: [
                                      { type: "Terminal", text: "OUT" },
                                      { type: "Terminal", text: "TO" },
                                    ],
                                  },
                                  { type: "NonTerminal", text: "@table" },
                                ],
                              },
                            },
                            { type: "Optional", child: { type: "Terminal", text: "ENFORCED" } },
                          ],
                        },
                      ],
                    },
                  ],
                },
              },

              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "VALUE" },
                  { type: "Terminal", text: "@value" },
                ],
              },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "ASSERT" },
                  { type: "Terminal", text: "@expression" },
                ],
              },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "DEFAULT" },
                  { type: "Optional", child: { type: "Terminal", text: "ALWAYS" } },
                  { type: "Terminal", text: "@expression" },
                ],
              },

              {
                type: "Optional",
                child: {
                  type: "Sequence",
                  children: [
                    { type: "Terminal", text: "PERMISSIONS" },
                    {
                      type: "Choice",
                      index: 1,
                      children: [
                        { type: "Terminal", text: "NONE" },
                        { type: "Terminal", text: "FULL" },
                        {
                          type: "Sequence",
                          children: [
                            { type: "Terminal", text: "WHERE" },
                            { type: "NonTerminal", text: "@condition" },
                          ],
                        },
                      ],
                    },
                  ],
                },
              },
              {
                type: "Sequence",
                children: [
                  { type: "Terminal", text: "COMMENT" },
                  { type: "Terminal", text: "@string" },
                ],
              },
            ],
          },
          repeat: { type: "Skip" },
        },
      ],
    },
  ],
};

<RailroadDiagram ast={alterAst} className="my-6" />

  </TabItem>
</Tabs>

## COMPACT

<Since v="v3.0.0" />

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

## explain.mdx

---
sidebar_position: 1
sidebar_label: EXPLAIN
title: EXPLAIN clause | SurrealQL
description: The `EXPLAIN` clause is used to explain the plan used for a query.
---


# `EXPLAIN` clause

The `EXPLAIN` clause is used to explain the plan used for a query. It is particularly useful when you want to understand how a query is executed and how it is optimized by the database.

When `EXPLAIN` is used, the statement returns an explanation, essentially revealing the execution plan to provide transparency and understanding of the query performance.

## Syntax

```syntax title="Clause Syntax"
@query EXPLAIN [FULL]
```

Using the `EXPLAIN` clause in addition to the `FULL` keyword is expeciallly useful when you want to understand the performance of a query and can provide more details when debugging.

## Examples

For example, consider the performance of the following query when the field `email` is not indexed. We can see that the execution plan will iterate over the whole table.

```surql title="Index not used"
/**[test]

[[test.results]]
value = "[{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', id: person:tobie, name: 'Tobie' }]"

[[test.results]]
value = "[{ detail: { direction: 'forward', table: 'person' }, operation: 'Iterate Table' }, { detail: { type: 'Memory' }, operation: 'Collector' }]"

[[test.results]]
value = "[{ detail: { direction: 'forward', table: 'person' }, operation: 'Iterate Table' }, { detail: { type: 'Memory' }, operation: 'Collector' }, { detail: { type: 'KeysAndValues' }, operation: 'RecordStrategy' }, { detail: { count: 1 }, operation: 'Fetch' }]"

*/

CREATE person:tobie SET
	name = "Tobie",
	address = "1 Bagshot Row",
	email = "tobie@surrealdb.com";

SELECT * FROM person WHERE email='tobie@surrealdb.com' EXPLAIN;
SELECT * FROM person WHERE email='tobie@surrealdb.com' EXPLAIN FULL;
```

```surql title="Output"
-------- Query --------

{
	attributes: {
		projections: '*'
	},
	children: [
		{
			attributes: {
				direction: 'Forward',
				predicate: "email = 'tobie@surrealdb.com'",
				table: 'person'
			},
			context: 'Db',
			operator: 'TableScan'
		}
	],
	context: 'Db',
	operator: 'SelectProject'
}

-------- Query --------

{
	attributes: {
		projections: '*'
	},
	children: [
		{
			attributes: {
				direction: 'Forward',
				predicate: "email = 'tobie@surrealdb.com'",
				table: 'person'
			},
			context: 'Db',
			metrics: {
				elapsed_ns: 66543,
				output_batches: 1,
				output_rows: 1
			},
			operator: 'TableScan'
		}
	],
	context: 'Db',
	metrics: {
		elapsed_ns: 8251,
		output_batches: 1,
		output_rows: 1
	},
	operator: 'SelectProject',
	total_rows: 1
}
```

On the other hand, here is the result when the field `email` is indexed. We can see that the execution plan will use the index to retrieve the record.

```surql title="Index used"
/**[test]

[[test.results]]
value = "NONE"

[[test.results]]
value = "[{ address: '1 Bagshot Row', email: 'tobie@surrealdb.com', id: person:tobie, name: 'Tobie' }]"

[[test.results]]
value = "[{ detail: { plan: { index: 'fast_email', operator: '=', value: 'tobie@surrealdb.com' }, table: 'person' }, operation: 'Iterate Index' }, { detail: { type: 'Memory' }, operation: 'Collector' }]"

[[test.results]]
value = "[{ detail: { plan: { index: 'fast_email', operator: '=', value: 'tobie@surrealdb.com' }, table: 'person' }, operation: 'Iterate Index' }, { detail: { type: 'Memory' }, operation: 'Collector' }, { detail: { type: 'KeysAndValues' }, operation: 'RecordStrategy' }, { detail: { count: 1 }, operation: 'Fetch' }]"

*/

DEFINE INDEX fast_email ON TABLE person FIELDS email;

CREATE person:tobie SET
	name = "Tobie",
	address = "1 Bagshot Row",
	email = "tobie@surrealdb.com";

SELECT * FROM person WHERE email='tobie@surrealdb.com' EXPLAIN;
SELECT * FROM person WHERE email='tobie@surrealdb.com' EXPLAIN FULL;    
```

```surql title="Output"
-------- Query --------

{
	attributes: {
		projections: '*'
	},
	children: [
		{
			attributes: {
				access: "= 'tobie@surrealdb.com'",
				direction: 'Forward',
				index: 'fast_email'
			},
			context: 'Db',
			operator: 'IndexScan'
		}
	],
	context: 'Db',
	operator: 'SelectProject'
}

-------- Query --------

{
	attributes: {
		projections: '*'
	},
	children: [
		{
			attributes: {
				access: "= 'tobie@surrealdb.com'",
				direction: 'Forward',
				index: 'fast_email'
			},
			context: 'Db',
			metrics: {
				elapsed_ns: 48624,
				output_batches: 1,
				output_rows: 1
			},
			operator: 'IndexScan'
		}
	],
	context: 'Db',
	metrics: {
		elapsed_ns: 4999,
		output_batches: 1,
		output_rows: 1
	},
	operator: 'SelectProject',
	total_rows: 1
}
```

## fetch.mdx

---
sidebar_position: 1
sidebar_label: FETCH
title: FETCH clause | SurrealQL
description: The `FETCH` clause is used to fetch records from a table.
---
import SurrealistMini from "@components/SurrealistMini.astro";

# `FETCH` clause

The `FETCH` clause is used to retrieve related records or data from other tables in a single query. This is particularly useful when you want to gather data that is linked through relationships ([record links](/docs/surrealql/datamodel/records) or [graph edges](/docs/surrealql/statements/relate)) without having to perform multiple separate queries.

## Benefits of Using the `FETCH` Clause

- Efficiency: By using the FETCH clause, you can reduce the number of queries needed to gather related data. This can lead to performance improvements, especially when dealing with complex data models with multiple relationships.

- Simplified Queries: It simplifies your queries by allowing you to specify related data to be fetched directly within the same query. This makes your code cleaner and easier to understand.

- Reduced Network Overhead: Fetching related data in a single query reduces the number of round trips to the database, which can decrease network latency and improve the overall speed of your application.

- Consistency: By fetching related data in one go, you ensure that the data is consistent and up-to-date at the time of retrieval, reducing the risk of discrepancies that might occur if data is fetched in separate queries.

## Example Usage

Suppose you have a person table and a post table, where each post is related to a person. You can use the FETCH clause to retrieve a person along with their posts in a single query:

```surql
SELECT * FROM person FETCH posts;
```

In this example, `posts` would be a related field in the `person` table that links to the `post` table. The `FETCH` clause allows you to retrieve all posts associated with each person in the result set.

Overall, the `FETCH` clause in SurrealQL is a powerful tool for optimizing data retrieval and simplifying query logic when working with related data.

In addition to fetching related records, the `FETCH` clause can also be used to replace record ids with their actual record values. Consider the following example:

<SurrealistMini
	resultMode="single"
	setup={`
		CREATE category SET name = 'Technology', created_at = time::now();
		CREATE person:john SET
			name.first = 'John',
			name.last = 'Adams',
			name.full = string::join(' ', name.first, name.last),
			age = 29,
			admin = true,
			signup_at = time::now();
		CREATE article SET
			created_at = time::now(),
			author = person:john,
			title = 'Lorem ipsum dolor',
			text = 'Donec eleifend, nunc vitae commodo accumsan, mauris est fringilla.',
			category = (SELECT VALUE id FROM ONLY category WHERE name = 'Technology' LIMIT 1);
	`}
	query={`
		SELECT title, category, author.name.full AS author_name FROM article
		WHERE author.age < 30
		FETCH author, category;
	`}
/>

## Without the `FETCH` clause

<SurrealistMini
	resultMode="single"
	setup={`
		CREATE category SET name = 'Technology', created_at = time::now();
		CREATE person:john SET
			name.first = 'John',
			name.last = 'Adams',
			name.full = string::join(' ', name.first, name.last),
			age = 29,
			admin = true,
			signup_at = time::now();
		CREATE article SET
			created_at = time::now(),
			author = person:john,
			title = 'Lorem ipsum dolor',
			text = 'Donec eleifend, nunc vitae commodo accumsan, mauris est fringilla.',
			category = (SELECT VALUE id FROM ONLY category WHERE name = 'Technology' LIMIT 1);
	`}
	query={`
		SELECT title, category, author.name.full AS author_name FROM article
		WHERE author.age < 30;
	`}
/>





## from.mdx

---
sidebar_position: 1
sidebar_label: FROM
title: FROM clause | SurrealQL
description: The `FROM` clause is used to specify the table or view to query.
---

# `FROM` clause

The `FROM` clause is used to specify the table or view to query. It can also be used to specify targets beyond just a single table or record name.

## Syntax

```syntax title="Clause Syntax"
STATEMENT
    [FROM [ONLY] @targets;]
```

## Data retrieval 

One of the most common use cases for the `FROM` clause is to specify the table or view to query. You can use this clause to pull data from single or multiple tables.

```surql title="All the ways you can use the FROM clause"
-- Selects all records from both 'user' and 'admin' tables.
SELECT * FROM user, admin;

-- Selects all records from the table named in the variable '$table',
-- but only if the 'admin' field of those records is true.
-- Equivalent to 'SELECT * FROM user WHERE admin = true'.
LET $table = "user";
SELECT * FROM type::table($table) WHERE admin = true;

-- Selects a single record from:
-- * the table named in the variable '$table',
-- * and the identifier named in the variable '$id'.
-- This query is equivalent to 'SELECT * FROM user:admin'.
LET $table = "user";
LET $id = "admin";
SELECT * FROM type::record($table, $id);

-- Selects all records for specific users 'tobie' and 'jaime',
-- as well as all records for the company 'surrealdb'.
SELECT * FROM user:tobie, user:jaime, company:surrealdb;

-- Selects records from a list of identifiers. The identifiers can be numerical,
-- string, or specific records such as 'person:lrym5gur8hzws72ux5fa'.
SELECT * FROM [3648937, "test", person:lrym5gur8hzws72ux5fa, person:4luro9170uwcv1xrfvby];

-- Selects data from an object that includes a 'person' key,
-- which is associated with a specific person record, and an 'embedded' key set to true.
SELECT * FROM { person: person:lrym5gur8hzws72ux5fa, embedded: true };

-- This command first performs a subquery, which selects all 'user' records and adds a
-- computed 'adult' field that is true if the user's 'age' is 18 or older.
-- The main query then selects all records from this subquery where 'adult' is true.
SELECT * FROM (SELECT age >= 18 AS adult FROM user) WHERE adult = true;
```

### Using the `ONLY` keyword

The `ONLY` keyword can be used to specify that only the specified targets should be retrieved. This is useful when you want to retrieve data from a single table or view. The `ONLY` keyword can be used in conjunction with the `LIMIT` clause to specify that only the specified number of records should be retrieved. 

This keyword can be particularly useful with SDKs as it can guaranteed to have just a single object and that makes it nicer to deserialise.

```surql title="Using the ONLY keyword"
-- Select only the 'user' table.
SELECT * FROM ONLY user:one;

-- Selects only the first 10 records from the 'user' table.
SELECT * FROM ONLY user LIMIT 10;
```






## group-by.mdx

---
sidebar_position: 1
sidebar_label: GROUP BY
title: GROUP BY clause | SurrealQL
description: The `GROUP BY` clause is used to group records by one or more columns.
---

import SurrealistMini from "@components/SurrealistMini.astro";

# `GROUP BY` clause

The `GROUP BY` clause is used in SurrealQL to aggregate data based on one or more columns. It is particularly useful when you want to perform calculations on groups of data, such as counting the number of records, calculating averages, or finding sums for each group. 

This is often used in reporting and data analysis to summarize data in a meaningful way. More specifically, it is used to:

- Aggregating Data: When you need to calculate aggregate values like SUM, COUNT, AVG, MIN, or MAX for each group of data.
- Data Summarization: When you want to summarize data into categories or groups.
- Reporting: When generating reports that require grouped data, such as sales reports by region or department.

## Syntax

```syntax title="Clause Syntax"
GROUP BY @fields
```

## Data representation


```surql
SELECT
    product_id,
    region,
    math::sum(amount) AS total_sales
FROM
    sales
GROUP BY
    product_id, region;
```

Explanation:
- `SELECT product_id, region, math::sum(amount) AS total_sales`: This selects the product_id and region columns and calculates the total sales amount for each group. The `AS` clause is used to rename the calculated column to `total_sales`.

- `FROM sales`: This specifies the table from which to retrieve the data. Using the `FROM` clause, we specify the table `sales` to retrieve the data from.

- `GROUP BY product_id, region`: This groups the results by product_id and region, so the SUM function calculates the total sales for each unique combination of product_id and region.

This query will return a result set where each row represents a unique combination of `product_id` and `region`, along with the total sales amount for that combination. This is useful for understanding how different products are performing in different regions.

<SurrealistMini
	resultMode="single"
	setup={`
INSERT INTO rams [
    { gender: "M", age: 20, country: "Japan" },
    { gender: "M", age: 25, country: "Japan" },
    { gender: "F", age: 23, country: "US" },
    { gender: "F", age: 30, country: "US" },
    { gender: "F", age: 25, country: "Korea" },
    { gender: "F", age: 45, country: "UK" },
];
	`}
	query={`
		SELECT
	count() AS total,
	math::mean(age) AS average_age,
	gender,
	country
FROM rams
GROUP BY gender, country;
	`}
/>





## index.mdx

---
sidebar_position: 1
sidebar_label: Clauses
title: Clauses | SurrealQL
description: Clauses are used to 
---

# Clauses

In SurrealQL, clauses can be used to alter the way a query is executed. They are used in the following ways: 

- [`EXPLAIN`](/docs/surrealql/clauses/explain): Explain the query plan.
- [`FETCH`](/docs/surrealql/clauses/fetch): Fetch all the fields of related records.
- [`FROM`](/docs/surrealql/clauses/from): Specify the table(s) or other target(s) to query from.
- [`GROUP BY`](/docs/surrealql/clauses/group-by): Group the results by a set of fields.
- [`LIMIT`](/docs/surrealql/clauses/limit): Limit the number of results.
- [`OMIT`](/docs/surrealql/clauses/omit): Omit related records.
- [`ORDER BY`](/docs/surrealql/clauses/order-by): Specify the sort order of the results.
- [`SPLIT`](/docs/surrealql/clauses/split): Split the results into a set of subqueries.
- [`WHERE`](/docs/surrealql/clauses/where): Specify a condition that acts as a filter.
- [`WITH`](/docs/surrealql/clauses/with): Replace the default table iterator with an index iterator.




## limit.mdx

---
sidebar_position: 1
sidebar_label: LIMIT
title: LIMIT clause | SurrealQL
description: The `LIMIT` clause is used to limit the number of records returned by a query.
---


# `LIMIT` clause

The `LIMIT` clause is used to limit the number of records returned by a query. It is particularly useful when you want to retrieve a specific number of records from a table. 

When using the `LIMIT` clause, it is possible to paginate results by using the `START` clause to start from a specific record from the result set. It is important to note that the `START` count starts from 0.

## Syntax

```syntax title="Clause Syntax"
LIMIT @number [START @start 0]
```

## Examples

```surql
-- Select the first 10 records
SELECT * FROM person LIMIT 10;

-- Start at record 50 and select the following 10 records
SELECT * FROM person LIMIT 10 START 50;
```

```surql
/**[test]

[[test.results]]
value = "[5, 6, 7, 8, 9]"

*/

-- Select the first 5 records from the array
SELECT * FROM [1,2,3,4,5,6,7,8,9,10] LIMIT 5 START 4;
```

```surql title="Result"
[
	5,
	6,
	7,
	8,
	9
]
```

The `LIMIT` clause followed by `1` is often used along with the `ONLY` clause to satisfy the requirement that only up to a single record can be returned.

```surql
-- Record IDs are unique so guaranteed to be no more than 1
SELECT * FROM ONLY person:jamie;

-- Error because no guarantee that this will return a single record
SELECT * FROM ONLY person WHERE name = "Jaime";

-- Add `LIMIT 1` to ensure that only up to one record will be returned
SELECT * FROM ONLY person WHERE name = "Jaime" LIMIT 1;
```



## omit.mdx

---
sidebar_position: 1
sidebar_label: OMIT
title: OMIT clause | SurrealQL
description: The `OMIT` clause is used to omit fields from the result set 
---

# `OMIT` clause

The `OMIT` clause is used to omit fields from the result set which can be particularly useful when querying large datasets.

## Syntax

```syntax title="Clause Syntax"
OMIT @fields FROM @table
```

## Examples

```surql
/**[test]

[[test.results]]
value = "[{ id: person:tobie, name: 'Tobie', opts: { enabled: true, security: 'secure' }, password: '123456' }]"

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', opts: { enabled: false, security: 'secure' }, password: 'asdfgh' }]"

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', opts: { enabled: false, security: 'secure' }, password: 'asdfgh' }, { id: person:tobie, name: 'Tobie', opts: { enabled: true, security: 'secure' }, password: '123456' }]"

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', opts: { enabled: false } }, { id: person:tobie, name: 'Tobie', opts: { enabled: true } }]"

[[test.results]]
value = "[{ id: person:jaime, name: 'Jaime', opts: {  } }, { id: person:tobie, name: 'Tobie', opts: {  } }]"

*/

CREATE person:tobie SET
	name = 'Tobie',
	password = '123456',
	opts.security = 'secure',
	opts.enabled = true;
CREATE person:jaime SET
	name = 'Jaime',
	password = 'asdfgh',
	opts.security = 'secure',
	opts.enabled = false;

SELECT * FROM person;
-- Omit the password field and security field in the options object
SELECT * OMIT password, opts.security FROM person;

-- Using destructuring syntax (since 2.0.0)
SELECT * OMIT password, opts.{ security, enabled } FROM person;
```

```surql title= "Return fields"

-------- Query 3 (132.138µs) --------

[
	{
		id: person:jaime,
		name: 'Jaime',
		opts: {
			enabled: false,
			security: 'secure'
		},
		password: 'asdfgh'
	},
	{
		id: person:tobie,
		name: 'Tobie',
		opts: {
			enabled: true,
			security: 'secure'
		},
		password: '123456'
	}
]

-------- Query 4 (61.876µs) --------

[
	{
		id: person:jaime,
		name: 'Jaime',
		opts: {
			enabled: false
		}
	},
	{
		id: person:tobie,
		name: 'Tobie',
		opts: {
			enabled: true
		}
	}
]

-------- Query 5 (52.152µs) --------

[
	{
		id: person:jaime,
		name: 'Jaime',
		opts: {}
	},
	{
		id: person:tobie,
		name: 'Tobie',
		opts: {}
	}
]
```




## order-by.mdx

---
sidebar_position: 1
sidebar_label: ORDER BY
title: ORDER BY clause | SurrealQL
description: The `ORDER BY` clause specifies the sort order of the records in a table.
---

# `ORDER BY` clause

To sort records, SurrealDB allows ordering on multiple fields and nested fields. Use the `ORDER BY` clause to specify a comma-separated list of field names that should be used to order the resulting records. 

The `ASC` and `DESC` keywords can be used to specify whether results should be sorted in an ascending or descending manner. The `COLLATE` keyword can be used to use Unicode collation when ordering text in string values, ensuring that different cases, and different languages are sorted in a consistent manner. Finally, the `NUMERIC` can be used to correctly sort text which contains numeric values.

It is also worth noting that `COLLATE` ignores unicode order. e.g. 'á' comes after 'z' by default (Unicode sorting) but with `COLLATE` 'á' comes before 'z'.

## Syntax

```syntax title="Clause Syntax"
[ ORDER [ BY ] 
	@field [ COLLATE ] [ NUMERIC ] [ ASC | DESC ], ...
	| RAND() ]
]
```

## Examples


```surql
SELECT * FROM <table> ORDER BY <field> ASC;
```

```surql 

-- Order records randomly
SELECT * FROM <table> ORDER BY rand();

-- Order records descending by a single field
SELECT * FROM <table> ORDER BY <field> DESC;

-- Order records by multiple fields independently
SELECT * FROM <table> ORDER BY <field> ASC, <field2> DESC;

-- Order text fields with lexical collation instead of Unicode order
SELECT * FROM <table> ORDER BY <field> COLLATE ASC;

-- Order text fields with which include numeric values
SELECT * FROM <table> ORDER BY <field> NUMERIC ASC;

-- COLLATE and NUMERIC can be used together
SELECT * FROM <table> ORDER BY <field> COLLATE NUMERIC ASC;
```

## split.mdx

---
sidebar_position: 1
sidebar_label: SPLIT
title: SPLIT clause | SurrealQL
description: The SPLIT clause in SurrealQL is used to split the results of a query based on a specific field, particularly when dealing with arrays.
---

# `SPLIT` clause

The `SPLIT` clause in SurrealQL is used to split the results of a query based on a specific field, particularly when dealing with arrays. This is useful in scenarios where you want to treat each element of an array as a separate row in the result set. It can be particularly helpful in data analysis contexts where you need to work with individual elements of an array separately.

## Syntax

```syntax title="Clause Syntax"
SPLIT [ON] @field
```

Suppose you have a user table with a field emails that contains an array of email addresses for each user. You want to list each email address as a separate record.

Here's how you can use the SPLIT clause in SurrealQL:

```surql
/**[test]

[[test.results]]
value = "[{ emails: ['john@example.com', 'doe@example.com'], id: user:jhjsg6c9ta32gdut36b8, name: 'John Doe' }]"
skip-record-id-key = true

[[test.results]]
value = "[{ emails: 'john@example.com', id: user:jhjsg6c9ta32gdut36b8, name: 'John Doe' }, { emails: 'doe@example.com', id: user:jhjsg6c9ta32gdut36b8, name: 'John Doe' }]"
skip-record-id-key = true

*/
CREATE user SET
    name = "John Doe",
    emails = ["john@example.com", "doe@example.com"];

-- Split the results by each value in the emails array
SELECT * FROM user SPLIT emails;
```

Explanation:
- `CREATE user SET ...`: This creates a user record with a name and an array of email addresses.
- `SELECT * FROM user SPLIT emails`: This query selects all fields from the user table and splits the results based on the emails field. Each email address in the `emails` array will now be in a field of the same name that only contains a single value.

Output:
The output of the query will be:

```surql
[
	{
		emails: 'john@example.com',
		id: user:unjgil312jvvxfbdj706,
		name: 'John Doe'
	},
	{
		emails: 'doe@example.com',
		id: user:unjgil312jvvxfbdj706,
		name: 'John Doe'
	}
]
```

## Using `SPLIT` to restructure collected paths

One practical use case with `SPLIT` is returning every possible combination of the relations inside multiple graph paths. For instance, take the following data below that represents the relations between Canada the country, its provinces, and their cities.

```surql
/**[test]

[[test.results]]
value = "[{ id: country:canada }]"

[[test.results]]
value = "[{ id: province:bc }, { id: province:alberta }]"

[[test.results]]
value = "[{ id: city:vancouver }, { id: city:victoria }, { id: city:edmonton }, { id: city:calgary }]"

[[test.results]]
value = "[{ id: in:5bgf21urlmudhbv87weh, in: city:vancouver, out: province:bc }, { id: in:nt3jd2nxdf28t1eeg7uz, in: city:victoria, out: province:bc }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: in:6iqjlca9vrnrcotwfb0a, in: city:edmonton, out: province:alberta }, { id: in:56xr5okh0jbmd5d34r0a, in: city:calgary, out: province:alberta }]"
skip-record-id-key = true

[[test.results]]
value = "[{ id: in:gr88pyx3de7nrsm63uo2, in: province:bc, out: country:canada }, { id: in:fidjczbmblc01syn4gzd, in: province:alberta, out: country:canada }]"
skip-record-id-key = true

*/

CREATE country:canada;
CREATE province:bc, province:alberta;
CREATE city:vancouver, city:victoria, city:edmonton, city:calgary;

RELATE [city:vancouver, city:victoria]->in->province:bc;
RELATE [city:edmonton, city:calgary]->in->province:alberta;
RELATE [province:bc, province:alberta]->in->country:canada;
```

A graph query on both of these paths shows all of the provinces and cities.

```surql
SELECT 
    id AS country,
    <-in<-province AS provinces,
    <-in<-province<-in<-city AS cities FROM ONLY country:canada;
```

```surql title="Output"
{
	cities: [
		city:calgary,
		city:edmonton,
		city:vancouver,
		city:victoria
	],
	country: country:canada,
	provinces: [
		province:alberta,
		province:bc
	]
}
```

Using `SPLIT` in this case transforms the output from a collection of paths centred on the `country:canada` record into an array of objects, each representing every possible combination of cities and provinces inside the country.

```surql
SELECT
    id AS country,
    <-in<-province AS province,
    <-in<-province<-in<-city AS city FROM country:canada
    SPLIT city, province;
```

```surql title="Output"
[
	{
		city: city:calgary,
		country: country:canada,
		province: province:alberta
	},
	{
		city: city:calgary,
		country: country:canada,
		province: province:bc
	},
	{
		city: city:edmonton,
		country: country:canada,
		province: province:alberta
	},
	{
		city: city:edmonton,
		country: country:canada,
		province: province:bc
	},
	{
		city: city:vancouver,
		country: country:canada,
		province: province:alberta
	},
	{
		city: city:vancouver,
		country: country:canada,
		province: province:bc
	},
	{
		city: city:victoria,
		country: country:canada,
		province: province:alberta
	},
	{
		city: city:victoria,
		country: country:canada,
		province: province:bc
	}
]
```

An example of the same query then mapped into a set of unique keys for serialization:

```surql
(SELECT
    id,
    <-in<-province AS province,
    <-in<-province<-in<-city AS city FROM country:canada
    SPLIT city, province)
.map(|$obj| 
    <string>$obj.id.id() 
    + '|' 
    + <string>$obj.province.id() 
    + '|' 
    + <string>$obj.city.id()
);
```

```surql title="Output"
[
	'canada|alberta|calgary',
	'canada|bc|calgary',
	'canada|alberta|edmonton',
	'canada|bc|edmonton',
	'canada|alberta|vancouver',
	'canada|bc|vancouver',
	'canada|alberta|victoria',
	'canada|bc|victoria'
]
```

## where.mdx

---
sidebar_position: 1
sidebar_label: WHERE
title: WHERE clause | SurrealQL
description: The `WHERE` clause can be used to specify a condition that acts as a filter. 
---

# `WHERE` clause

The `WHERE` clause can be used to specify a condition that acts as a filter. You can use the `WHERE` clause to either filter the result of the FROM clause in a `SELECT` statement or specify which rows to operate on in an `UPDATE`, `MERGE`, or `DELETE` statement.

It can also be used in special cases when working with conditons in [`DEFINE FUNCTION`](/docs/surrealql/statements/define/access) statement or when asserting access control in [`DEFINE TABLE`](/docs/surrealql/statements/define/table) & [`DEFINE FIELD`](/docs/surrealql/statements/define/field) statements.

## Syntax

```syntax title="Clause Syntax"
STATEMENT
    [WHERE condition;]

```

## Conditional record selection

The most common use case for the `WHERE` clause is to filter the result of the `SELECT` statement. It is particularly useful when you want to select a subset of records from a table based on a condition.

```surql
SELECT @fields FROM <TABLE_NAME> WHERE <CONDITION> = <VALUE>;
```

When fetching records from a table, the `WHERE` clause is used to filter the records that are returned.

## Conditional record alteration

The `WHERE` clause can also be used to specify which records to operate on in an `UPDATE`, `MERGE`, or `DELETE` statement.

```surql
UPDATE [TABLE_NAME] SET [FIELDS] WHERE [CONDITION] = [VALUE];
```


## Setting conditions in `DEFINE FUNCTION` statements

```surql
/**[test]

[[test.results]]
value = "NONE"

*/
-- Define a function that checks if a relation exists between two nodes
DEFINE FUNCTION fn::relation_exists(
	$in: record,
	$tb: string,
	$out: record
) {
	-- Check if a relation exists between the two nodes.
	LET $results = SELECT VALUE id FROM type::table($tb) WHERE in = $in AND out = $out;
	-- Return true if a relation exists, false otherwise
    RETURN array::len($results) > 0;
};
```

## Setting permissions conditions in `DEFINE TABLE` statements 

The `WHERE` clause can be used to specify the conditions for the permissions of a table and based on the conditions, the permissions are applied to the table CRUD operations.

```surql
/**[test]

[[test.results]]
value = "NONE"

*/
-- Specify access permissions for the 'post' table
DEFINE TABLE post SCHEMALESS
	PERMISSIONS
		FOR select
			-- Published posts can be selected
			WHERE published = true
			-- A user can select all their own posts
			OR user = $auth.id
		FOR create, update
			-- A user can create or update their own posts
			WHERE user = $auth.id
		FOR delete
			-- A user can delete their own posts
			WHERE user = $auth.id
			-- Or an admin can delete any posts
			OR $auth.admin = true
;
```

```surql
/**[test]

[[test.results]]
value = "NONE"

*/

-- Define a relation table, and constrain the type of relation which can be stored
DEFINE TABLE assigned_to SCHEMAFULL TYPE RELATION IN tag OUT sticky
    PERMISSIONS
        FOR create, select, update, delete
            WHERE in.owner == $auth.id AND out.author == $auth.id;
```

## with.mdx

---
sidebar_position: 1
sidebar_label: WITH
title: WITH clause | SurrealQL
description: The `WITH` clause is used to select records from a table with an index, which is a pre-computed lookup table for faster queries.
---

# `WITH` clause


When retrieving data from a table, the query planner can replace the standard table iterator with one or several index iterators based on the structure and requirements of the query. This is particularly useful when querying large datasets, as it can significantly reduce the time it takes to retrieve the data.

However, there may be situations where manual control over these potential optimizations is desired or required. 

The `WITH` clause is used to replace the default table iterator with an index iterator. In cases where the cardinality of an index can be high, potentially even equal to the number of records in the table, the sum of the records iterated by several indexes may end up being larger than the number of records obtained by iterating over the table. 

In such cases, if there are different index possibilities, the most probable optimal choice would be to use the index known with the lowest cardinality.

The query planner can replace the standard table iterator with one or several index iterators based on the structure and requirements of the query.

> [!NOTE]
> If you are using a `SELECT` statement, the `WITH` clause is used to specify the index to use for the query. You can define an index using the [`DEFINE INDEX`](/docs/surrealql/statements/define/indexes) statement. Also see the [`DEFINE ANALYZER`](/docs/surrealql/statements/define/analyzer) statement for more information on optimizing query performance with full-text search.

## Syntax

```syntax title="Clause Syntax"
[ WITH [ NOINDEX | INDEX @indexes ... ]]
```

This clause can be used in the following ways:

- `WITH NOINDEX`: forces the query planner to use the table iterator. (Default)
- `WITH INDEX @indexes`: restricts the query planner to using only the specified index(es)


```surql
-- forces the query planner to use the specified index(es):
SELECT * FROM person
WITH INDEX ft_email
WHERE
	email = 'tobie@surrealdb.com' AND
	company = 'SurrealDB';

-- forces the usage of the table iterator
SELECT name FROM person WITH NOINDEX WHERE job = 'engineer' AND gender = 'm';
```





## transactions.mdx

---
sidebar_position: 7
sidebar_label: Transactions
title: Transactions | SurrealQL
description: Each statement within SurrealDB is run within its own transaction, or within client defined transactions that can contain multiple statements.
---

# Transactions

Each statement within SurrealDB is run within its own transaction by default. If a set of changes need to be made together, then groups of statements can be run together as a single transaction. If all of the statements within a transaction succeed, and the transaction is successful, then all of the data modifications made during the transaction are committed and become a permanent part of the database. If a transaction encounters errors and must be cancelled or rolled back, then any data modification made within the transaction is rolled back, and will not become a permanent part of the database.

## Starting a transaction

The `BEGIN` or `BEGIN TRANSACTION` statement starts a transaction in which multiple statements can be run together.

```surql title="Starting a transaction"
BEGIN [ TRANSACTION ];
```

The following query shows example usage of this statement.

```surql title="Example usage of BEGIN TRANSACTION"
/**[test]

[[test.results]]
value = "[{ balance: 135605.16f, id: account:one }]"

[[test.results]]
value = "[{ balance: 91031.31f, id: account:two }]"

[[test.results]]
value = "[{ balance: 135905.16f, id: account:one }]"

[[test.results]]
value = "[{ balance: 90731.31f, id: account:two }]"

*/

-- Start a new database transaction. Transactions are a way to ensure multiple operations
-- either all succeed or all fail, maintaining data integrity.
BEGIN TRANSACTION;

-- Create a new account with the ID 'one' and set its initial balance to 135605.16
CREATE account:one SET balance = 135605.16;

-- Create another new account with the ID 'two' and set its initial balance to 91031.31
CREATE account:two SET balance = 91031.31;

-- Update the balance of account 'one' by adding 300.00 to the current balance.
-- This could represent a deposit or other form of credit on the balance property.
UPDATE account:one SET balance += 300.00;

-- Update the balance of account 'two' by subtracting 300.00 from the current balance.
-- This could represent a withdrawal or other form of debit on the balance property.
UPDATE account:two SET balance -= 300.00;

-- Finalize the transaction. This will apply the changes to the database. If there was an error
-- during any of the previous steps within the transaction, all changes would be rolled back and
-- the database would remain in its initial state.
COMMIT TRANSACTION;
```

## Committing a transaction

The [COMMIT](/docs/surrealql/statements/commit) statement is used to commit a set of statements within a transaction, ensuring that all data modifications become a permanent part of the database.

```surql title="Committing a transaction"
COMMIT [ TRANSACTION ];
```

The following query shows example usage of this statement.

```surql title="Example usage of COMMIT TRANSACTION"
/**[test]

[[test.results]]
value = "[{ balance: 135605.16f, id: account:one }]"

[[test.results]]
value = "[{ balance: 91031.31f, id: account:two }]"

[[test.results]]
value = "[{ balance: 135905.16f, id: account:one }]"

[[test.results]]
value = "[{ balance: 90731.31f, id: account:two }]"

*/

BEGIN TRANSACTION;

-- Setup accounts
CREATE account:one SET balance = 135605.16;
CREATE account:two SET balance = 91031.31;

-- Move money
UPDATE account:one SET balance += 300.00;
UPDATE account:two SET balance -= 300.00;

-- Finalise all changes
COMMIT TRANSACTION;
```

## Cancelling a transaction

The [CANCEL](/docs/surrealql/statements/cancel) statement can be used to cancel a set of statements within a transaction, reverting or rolling back any data modification made within the transaction as a whole.

```surql title="Cancelling a transaction"
CANCEL [ TRANSACTION ];
```

The following query shows example usage of this statement.

```surql title="Example usage of CANCEL TRANSACTION"
/**[test]

[[test.results]]
error = "'Thrown error: The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'Thrown error: The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'Thrown error: The query was not executed due to a cancelled transaction'"

[[test.results]]
error = "'Thrown error: The query was not executed due to a cancelled transaction'"

*/

BEGIN TRANSACTION;

-- Setup accounts
CREATE account:one SET balance = 135605.16;
CREATE account:two SET balance = 91031.31;

-- Move money
UPDATE account:one SET balance += 300.00;
UPDATE account:two SET balance -= 300.00;

-- Rollback all changes
CANCEL TRANSACTION;
```

## THROW to conditionally cancel a transaction

While transactions are automatically rolled back if an error occurs in any of its statements, [THROW](/docs/surrealql/statements/throw) can also be used to explicitly break out of a transaction at any point. `THROW` can be followed by any value which serves as the error message, usually a string.

```surql
BEGIN TRANSACTION;

CREATE account:one SET dollars =  100;
CREATE account:two SET dollars =  100;

LET $transfer_amount = 150;
UPDATE account:one SET dollars -= $transfer_amount;
UPDATE account:two SET dollars += $transfer_amount;
IF account:one.dollars < 0 {
    THROW "Insufficient funds, would have $" + <string>account:one.dollars + " after transfer"
};
COMMIT TRANSACTION;
SELECT * FROM account;
```

```surql title="Output when $transfer_amount set to 150"
'An error occurred: Insufficient funds, would have $-50 after transfer'
```

```surql title="Output when $transfer_amount set to 50"
[
	{
		dollars: 50,
		id: account:one
	},
	{
		dollars: 150,
		id: account:two
	}
]
```

## Using transactions to test code for errors

As failed transactions automatically roll back any changes made, a transaction with a final `THROW` statement can be used as a confirmation that no errors have taken place inside a group of queries.

Take the following example that creates a unique index and then inserts some records to make sure that the database logic is functioning as expected. However, as names are not necessarily unique, the index soon gives an error and cancels the transaction before `THROW` can be reached.

```surql
BEGIN TRANSACTION;
DEFINE INDEX unique_name ON TABLE person FIELDS name UNIQUE;

INSERT INTO person [
    { name: 'Agatha Christie', born: d'1890-09-15' },
    { name: 'Billy Billerson', born: d'1979-09-11' },
	-- Pretend there are is 10,000 more objects here
    { name: 'Agatha Christie', born: d'1955-05-15' },
];

THROW "Reached the end";
COMMIT TRANSACTION;
```

The output is not the expected 'An error occurred: Reached the end' message, showing that not all queries were successful.

```surql title="Output"
"Database index `unique_name` already contains 'Agatha Christie', with record `person:qs4bpvl96sf9x40b3567`"
```

If the index is redefined to be less strict, the statetements will work and the expected output will be reached, confirming that no errors occurred during the test.

```surql
BEGIN TRANSACTION;
DEFINE INDEX OVERWRITE unique_person ON TABLE person FIELDS name, born UNIQUE;

INSERT INTO person [
    { name: 'Agatha Christie', born: d'1890-09-15' },
    { name: 'Billy Billerson', born: d'1979-09-11' },
    { name: 'Agatha Christie', born: d'1955-05-15' },
];

THROW "Reached the end";
COMMIT TRANSACTION;
```

```surql title="Expected output"
'An error occurred: Reached the end'
```
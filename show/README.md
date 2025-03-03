# Readyset SHOW Commands

When connected to a Readyset cache, a number of `SHOW` commands can be used to retrieve meta information. [Documentation](https://readyset.io/docs/reference/command-reference#show)

If you are reviewing a Readyset cache setup, the following statements provide an overview of the setup, configuration and operation.

```
SHOW READYSET VERSION;
SHOW READYSET STATUS;
SHOW READYSET STATUS ADAPTER;
SHOW CONNECTIONS;
SHOW READYSET TABLES;
SHOW CACHES;
SHOW PROXIED QUERIES;
```

After initial operation, there are additional syntax that is useful for cache management.

```
SHOW PROXIED SUPPORTED QUERIES;
SHOW READYSET ALL TABLES;
```

There are additional `SHOW` commands that apply in certain operations and by default require arguments or prior statements.

```
SHOW MIGRATION STATUS {ID}
```


## Version

```
readyset@127.0.0.1 (imdb) [21:04:54] > SHOW READYSET VERSION;
+--------------------+------------------------------------------+
| ReadySet           | Version Information                      |
+--------------------+------------------------------------------+
| release version    | dev                                      |
| commit id          | 804128f2ccf859a0e1efb281aceeaba0d34b15ae |
| platform           | x86_64-unknown-linux-gnu                 |
| rustc version      | rustc 1.84.0 (9fc6b4312 2025-01-07)      |
| profile            | release                                  |
| optimization level | 3                                        |
+--------------------+------------------------------------------+
6 rows in set (0.00 sec)
```

## Status

```
readyset@127.0.0.1 (imdb) [21:04:58] > SHOW READYSET STATUS;
+----------------------------+-------------------------+
| Variable_name              | Value                   |
+----------------------------+-------------------------+
| Database Connection        | Connected               |
| Connection Count           | 1                       |
| ReadySet Controller Status | Unavailable             |
| Last started Controller    | 2025-03-02 01:53:08 UTC |
| Last completed snapshot    | 2025-03-02 01:59:58 UTC |
| Last started replication   | 2025-03-02 01:59:58 UTC |
+----------------------------+-------------------------+
6 rows in set (5.00 sec)
```

## Adapter

```
readyset@127.0.0.1 (imdb) [21:05:56] > SHOW READYSET STATUS ADAPTER;
+------------------------------------+-------------------------+
| Variable_name                      | Value                   |
+------------------------------------+-------------------------+
| Connected clients count            | 1                       |
| Upstream database connection count | 2                       |
| Query parse failures               | 0                       |
| SET statement disallowed count     | 0                       |
| View not found count               | 0                       |
| RPC error count                    | 0                       |
| Process start time                 | 2025-03-02 01:53:07 UTC |
+------------------------------------+-------------------------+
7 rows in set (0.05 sec)
```

## Connections

```
readyset@127.0.0.1 (imdb) [21:06:05] > SHOW CONNECTIONS;
+-----------------+
| remote_addr     |
+-----------------+
| 127.0.0.1:54326 |
+-----------------+
1 row in set (0.00 sec)
```


## Tables

Unless configured on startup, by default no Tables will be present in the cache.

```
readyset@127.0.0.1 (imdb) [09:39:53] > SHOW READYSET TABLES;
Empty set (0.00 sec)
```

## Caches

```
readyset@127.0.0.1 (imdb) [09:39:18] > SHOW CACHES;
Empty set (0.02 sec)
```

## Queries


```
readyset@127.0.0.1 (imdb) [09:39:18] > SHOW PROXIED QUERIES;
+--------------------+----------------+--------------------+-------+
| query id           | proxied query  | readyset supported | count |
+--------------------+----------------+--------------------+-------+
| q_ba8b1e9d38a854fc | SELECT USER()  | unsupported        |     0 |
| q_95844e527a191a7b | show databases | unsupported        |     0 |
+--------------------+----------------+--------------------+-------+
```

## Supported Queries

```
readyset@127.0.0.1 (imdb) [18:31:23] > SHOW PROXIED SUPPORTED QUERIES;
Empty set (0.00 sec)
```

## Tables in Use by ReadSet

When tables are added they will move from `Snapshotting` to `Snapshotted`.
`description` is always empty in with this syntax.

```
readyset@127.0.0.1 (imdb) [18:45:01] > SHOW READYSET TABLES;
+-------------------------------+--------------+-------------+
| table                         | status       | description |
+-------------------------------+--------------+-------------+
| `imdb`.`name`                 | Snapshotted  |             |
| `imdb`.`title`                | Snapshotted  |             |
| `imdb`.`title_name_character` | Snapshotted  |             |
| `imdb`.`title_principal`      | Snapshotting |             |
+-------------------------------+--------------+-------------+
4 rows in set (0.01 sec)
```

## Queries available for the cache

As queries are executed they will show up with one of the following `readyset supported` states:

- unsupported
- pending
- yes

```
readyset@127.0.0.1 (imdb) [09:57:50] >  SHOW PROXIED QUERIES;
+--------------------+------------------------------------------+--------------------+-------+
| query id           | proxied query                            | readyset supported | count |
+--------------------+------------------------------------------+--------------------+-------+
| q_4c0ebd40f712c210 | SELECT * FROM `name` WHERE (`name` = $1) | yes                |     0 |
| q_ea5296d5c4ec130b | show warnings                            | unsupported        |     0 |
| q_95844e527a191a7b | show databases                           | unsupported        |     0 |
+--------------------+------------------------------------------+--------------------+-------+
```

As your list of parsed queries grows you will want to to use the `SUPPORTED` keyword.

```
readyset@127.0.0.1 (imdb) [10:17:26] > SHOW PROXIED SUPPORTED QUERIES;
+--------------------+-------------------------------------------------+--------------------+-------+
| query id           | proxied query                                   | readyset supported | count |
+--------------------+-------------------------------------------------+--------------------+-------+
| q_24fea182a2b22fbc | SELECT count(*) FROM `name` WHERE (`born` = $1) | yes                |     0 |
| q_512d3b002c1fe4f2 | SELECT `name` FROM `name` WHERE (`name` = $1)   | yes                |     0 |
+--------------------+-------------------------------------------------+--------------------+-------+
2 rows in set (0.00 sec)
```

## All Tables

You cannot limit the output of this command if you hae 1,000s of databases across all schemas.

```
SHOW READYSET ALL TABLES;
+---------------------------------------+-----------------+-------------------------------------------------------------------------------------------------------+
| table                                 | status          | description                                                                                           |
+---------------------------------------+-----------------+-------------------------------------------------------------------------------------------------------+
| `imdb`.`character_name`               | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`credit`                       | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`genre`                        | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`name`                         | Snapshotted     |                                                                                                       |
| `imdb`.`name_known_for`               | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`name_profession`              | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title`                        | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title_character`              | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title_crew`                   | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title_director`               | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title_episode`                | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title_genre`                  | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title_name_character`         | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title_principal`              | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title_rating`                 | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
| `imdb`.`title_writer`                 | Not Replicated  | The table was either excluded from replicated-tables or included in replication-tables-ignore option. |
+---------------------------------------+-----------------+-------------------------------------------------------------------------------------------------------+
```

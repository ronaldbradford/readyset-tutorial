# Caching Queries


## Seeing cachable queries




## Determining if a query is cachable

```
EXPLAIN CREATE CACHE FROM SELECT died FROM name WHERE died = 1999;
+--------------------+-----------------------------------------------+--------------------+
| query id           | query                                         | readyset supported |
+--------------------+-----------------------------------------------+--------------------+
| q_7d58699d3554b884 | SELECT `died` FROM `name` WHERE (`died` = $1) | yes                |
+--------------------+-----------------------------------------------+--------------------+
1 row in set (0.01 sec)
```

## Caching a query

You can specify the query, the query digest or the query id when caching a query.

```
CREATE CACHE FROM SELECT * FROM `name` WHERE (`name` = $1);
```

NOTE: You will not get a message, or any error log details if a query is already cached.

```
readyset@127.0.0.1 (imdb) [11:16:14] > CREATE CACHE FROM SELECT * FROM `name` WHERE (`name` = $1);
Query OK, 0 rows affected (0.00 sec)
```

## SQL Syntax

As a query moves from execution to proxied to cache, you will observe the parsing syntax of the query will change.

### Executed

```
SELECT * FROM name WHERE name = 'Tom Hanks';
```

### Proxied

```
SELECT * FROM `name` WHERE (`name` = $1)
```

### Cached
```
SELECT `imdb`.`name`.`name_id`, `imdb`.`name`.`nconst`, `imdb`.`name`.`name`, `imdb`.`name`.`born`, `imdb`.`name`.`died`, `imdb`.`name`.`updated` FROM `imdb`.`name` WHERE (`imdb`.`name`.`name` = $1)
```

In the ReadySet Cloud product these queries are formatted in a more readable format.


### Readyset Error Log
```
2025-03-02T16:16:36.811828Z  INFO readyset_server::controller::inner: creating cache q_4c0ebd40f712c210
```


## Caching Concurrently

```
readyset@127.0.0.1 (imdb) [18:42:36] > CREATE CACHE CONCURRENTLY FROM q_79aaa346e2d7e34b;
+--------------+
| Migration Id |
+--------------+
| 12884901889  |
+--------------+
1 row in set (0.00 sec)


readyset@127.0.0.1 (imdb) [18:54:27] > SHOW READYSET MIGRATION STATUS 12884901889;
+------------------+
| Migration Status |
+------------------+
| Pending          |
+------------------+
1 row in set (0.00 sec)
```

NOTE: There is no easy way to get the `Migration Id` to automate passing into the `SHOW` command.

Should the query fail to cache the `SHOW` command will provide a reason. For example:

```
readyset@127.0.0.1 (imdb) [18:44:01] > SHOW READYSET MIGRATION STATUS 12884901889;
+--------------------------------------------------------------------------------------------------------------------------------------------------+
| Migration Status                                                                                                                                 |
+--------------------------------------------------------------------------------------------------------------------------------------------------+
| Failed with error SQL SELECT query 'q_79aaa346e2d7e34b' couldn't be added: Table 'imdb.title_name_character' is not being replicated by ReadySet |
+--------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

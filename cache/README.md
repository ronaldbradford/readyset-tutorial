# Caching Queries

Readyset operates by managing a cache of queries that you configure. Under the covers, Readyset manages a set of tables that are kept in sync by subscribing to the respective streams of changes from the database.  This is the MySQL binary log replication or the PostgreSQL logical WAL.

When you are connected to a Readyset cache, all queries are passed to the target database. When first installed Readyset is effectively a pass-through cache for your SQL statements.

## Seeing cachable queries

As queries are executed, Readyset will capture these and you can view them using the `SHOW PROXIED QUERIES` command.
You can also start with the `SHOW PROXIED SUPPORTED QUERIES` during an introduction to simplify demonstrating the caching capabilities.

```
readyset@127.0.0.1 (imdb) [13:58:18] > SHOW PROXIED SUPPORTED QUERIES\G
*************************** 1. row ***************************
          query id: q_4d5ea0e1e722a5b7
     proxied query: SELECT DISTINCT CONCAT(`n`.`name`, ' was a ', `tp`.`category`, ' in the movie ''', `t`.`title`, ''' (', IFNULL(`t`.`start_year`, 'unknown'), ')') AS `description` FROM `name` AS `n` JOIN `title_principal` AS `tp` ON (`n`.`name_id` = `tp`.`name_id`) JOIN `title` AS `t` ON (`tp`.`title_id` = `t`.`title_id`) WHERE ((`n`.`name` = $1) AND (`t`.`type` = $2))
readyset supported: yes
             count: 0
*************************** 2. row ***************************
          query id: q_4c0ebd40f712c210
     proxied query: SELECT * FROM `name` WHERE (`name` = $1)
readyset supported: yes
             count: 0
*************************** 3. row ***************************
          query id: q_3c05fa2522d2e116
     proxied query: SELECT `np`.`profession` FROM `name` AS `n` JOIN `name_profession` AS `np` ON (`n`.`name_id` = `np`.`name_id`) WHERE (`n`.`name` = $1)
readyset supported: yes
             count: 0
3 rows in set (0.00 sec)
```

## Determining if a query is cachable

In addition to queries that you execute and are listed as `readyset supported`, you can manually validate and SQL statement using the `EXPLAIN CREATE CACHE FROM <sql>` command.

```
EXPLAIN CREATE CACHE FROM SELECT name FROM name WHERE died = 1999;
+--------------------+-----------------------------------------------+--------------------+
| query id           | query                                         | readyset supported |
+--------------------+-----------------------------------------------+--------------------+
| q_7d58699d3554b884 | SELECT `name` FROM `name` WHERE (`died` = $1) | yes                |
+--------------------+-----------------------------------------------+--------------------+
1 row in set (0.01 sec)
```

## Caching a query

You can specify the query, the query digest or the query id when caching a query.

### Query id
```
CREATE CACHE FROM q_4c0ebd40f712c210;
Query OK, 0 rows affected (9.01 sec)
```

NOTE: This is a blocking statement, so a query that is being cached may take some time. You can monitor the progress via the [Readyset Log](../log/README.md).

### Query statement
```
CREATE CACHE FROM SELECT name FROM name WHERE died = 1999;
Query OK, 0 rows affected (4.70 sec)
```

### Query digest
```
CREATE CACHE FROM SELECT * FROM `name` WHERE (`name` = $1);
```

NOTE: You will not get a message, or any error log details if a query is already cached.

```
readyset@127.0.0.1 (imdb) [11:16:14] > CREATE CACHE FROM SELECT * FROM `name` WHERE (`name` = $1);
Query OK, 0 rows affected (0.00 sec)
```

## the evolution of the SQL syntax

As a query moves from initial execution, to proxied, to cached, you will observe the parsing syntax of the query will change to be more specific.

### Executed

For example, a simple SQL statement to view a row of data.

```
SELECT * FROM name WHERE name = 'Tom Hanks';
```

### Proxied

When proxied this SQL statement is converted to a digest for parameters.

```
SELECT * FROM `name` WHERE (`name` = $1)
```

### Cached

When cached this SQL statement is fully qualified using schema, table and column name to specify all relations of the query.

```
SELECT `imdb`.`name`.`name_id`, `imdb`.`name`.`nconst`, `imdb`.`name`.`name`, `imdb`.`name`.`born`, `imdb`.`name`.`died`, `imdb`.`name`.`updated` FROM `imdb`.`name` WHERE (`imdb`.`name`.`name` = $1)
```

NOTE: In the ReadySet Cloud product these queries are formatted in a more readable format. What is shown here is the client output.


## Cached Queries

```
readyset@127.0.0.1 (imdb) [14:08:38] > SHOW CACHES\G
*************************** 1. row ***************************
         query id: q_4c0ebd40f712c210
       cache name: q_4c0ebd40f712c210
       query text: SELECT `imdb`.`name`.`name_id`, `imdb`.`name`.`nconst`, `imdb`.`name`.`name`, `imdb`.`name`.`born`, `imdb`.`name`.`died`, `imdb`.`name`.`updated` FROM `imdb`.`name` WHERE (`imdb`.`name`.`name` = $1)
fallback behavior: fallback allowed
            count: 0
*************************** 2. row ***************************
         query id: q_7eef294f60d92815
       cache name: q_7eef294f60d92815
       query text: SELECT `imdb`.`name`.`name` FROM `imdb`.`name` WHERE (`imdb`.`name`.`died` = $1)
fallback behavior: fallback allowed
            count: 0
2 rows in set (0.00 sec)
```

### Readyset Error Log
```
2025-03-02T16:16:36.811828Z  INFO readyset_server::controller::inner: creating cache q_4c0ebd40f712c210
```


## Caching concurrently functionality

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

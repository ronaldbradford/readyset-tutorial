# Caching Queries

Readyset operates by managing a cache of queries that you configure. Under the covers, Readyset manages a set of tables that are kept in sync by subscribing to the respective streams of changes from the database.  This is the MySQL binary log replication or the PostgreSQL logical WAL.

When you are connected to a Readyset cache, all queries are passed to the target database. When first installed Readyset is effectively a pass-through proxy for all of your SQL statements.

## Reviewing Cacheable Queries

As queries are executed, Readyset will capture these and you can view them using the `SHOW PROXIED QUERIES` command.
You can also start with the `SHOW PROXIED SUPPORTED QUERIES` to simplify demonstrating the caching capabilities.

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

The `readyset support` column can contain a number of values including:
- `yes`
- `pending`
- `unsupported`


## Determining if a Query is Cacheable

In addition to queries that you execute and are listed as `readyset supported`, you can manually validate an SQL statement using the `EXPLAIN CREATE CACHE FROM <sql>` command.

```
EXPLAIN CREATE CACHE FROM SELECT name FROM name WHERE died = 1999;
+--------------------+-----------------------------------------------+--------------------+
| query id           | query                                         | readyset supported |
+--------------------+-----------------------------------------------+--------------------+
| q_7d58699d3554b884 | SELECT `name` FROM `name` WHERE (`died` = $1) | yes                |
+--------------------+-----------------------------------------------+--------------------+
1 row in set (0.01 sec)
```

## Caching a Query

There are 3 different syntax options to cache a query with `CREATE CACHE FROM`. You can:
- specify the query
- the query digest
- the query id

### Using the Query Id

In the `SHOW PROXIED QUERIES` output there is `query id` column which can be used.

```
CREATE CACHE FROM q_4c0ebd40f712c210;
Query OK, 0 rows affected (9.01 sec)
```

NOTE: This is a blocking statement, so a query that is being cached may take some time. You can monitor the progress via the [Readyset Log](../log/README.md).

### Using the Query Statement

You can create a cached query by specifying the SQL query directly.

```
CREATE CACHE FROM SELECT name FROM name WHERE died = 1999;
Query OK, 0 rows affected (4.70 sec)
```

### Using the Query Digest

You can use a query digest, such as the value of `query` from the `SHOW PROXIED QUERIES` command.

```
CREATE CACHE FROM SELECT * FROM `name` WHERE (`name` = $1);
```

NOTE: You will not get a message, or any error log details if a query is already cached.

```
readyset@127.0.0.1 (imdb) [11:16:14] > CREATE CACHE FROM SELECT * FROM `name` WHERE (`name` = $1);
Query OK, 0 rows affected (0.00 sec)
```

## The Evolution of the SQL Syntax in a Query

As a query moves from initial execution, to a proxied query, to a cached query, you will observe the parsing syntax of the query will change to be more specific.

### Executing a SQL Query

For example, a simple SQL statement to view a row of data could be executed as:

```
SELECT * FROM name WHERE name = 'Tom Hanks';
```

### A Proxied Query

When proxied this SQL statement is converted to a digest for parameters values and would appear in `SHOW PROXIED QUERIES` as.

```
SELECT * FROM `name` WHERE (`name` = $1)
```

### A Cached Query

When cached this SQL statement is fully qualified using schema, table and column names to specify all relations used in the query. In the `SHOW CACHES` a query would appear as.

```
SELECT `imdb`.`name`.`name_id`, `imdb`.`name`.`nconst`, `imdb`.`name`.`name`, `imdb`.`name`.`born`, `imdb`.`name`.`died`, `imdb`.`name`.`updated` FROM `imdb`.`name` WHERE (`imdb`.`name`.`name` = $1)
```

<div style="border: 2px solid red; padding: 10px; background-color: #ffcccc; color: red;">
**NOTE:** In the ReadySet Cloud product these queries are formatted in a more readable format. What is shown here is the client output of queries specified on a single line for ease of use.
</div>


## Cached Queries

All cached queries can be seen via the `SHOW CACHES` command.

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


## Caching Concurrently Functionality

A feature that enables you to create a cache asynchronously is to use the `CONCURRENTLY` keyword. You can then use the `SHOW READYSET MIGRATION STATUS` command to monitor this cache creation.

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

# Issues Creating a Cache

A common error can be that the underlying tables for the query you wish to cache are not being replicated. You would receive an error like the following. See the [Tables](../table/README.md) documentation for how to correct this issue.

```
readyset@127.0.0.1 (imdb) [19:32:24] > create cache from select * from imdb.name where name = 'Tom Hanks';
ERROR 1105 (HY000): Error during RPC (extend_recipe): SQL SELECT query 'q_3682e586a400de7e' couldn't be added: Table 'imdb.name' is not being replicated by ReadySet
```

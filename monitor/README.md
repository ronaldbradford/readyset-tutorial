# Evaluating and Measuring Readyset Performance

## Using MySQL client

The easiest way to evaluate the performance benefits of ReadySet for long running queries that take longer than 10 milliseconds is to use the `mysql` client. As seen here, these queries take 700 to 900 milliseconds to execute. Any query that is < 10 ms cannot me currently measured using the MySQL client response.

```
mysql> SELECT COUNT(*) FROM name;
+----------+
| COUNT(*) |
+----------+
| 14235647 |
+----------+
1 row in set (0.96 sec)

mysql> SELECT COUNT(*) FROM name;
+----------+
| COUNT(*) |
+----------+
| 14235647 |
+----------+
1 row in set (0.78 sec)

mysql> SELECT COUNT(*) FROM name;
+----------+
| COUNT(*) |
+----------+
| 14235647 |
+----------+
1 row in set (0.79 sec)
```

You can use the `PERFORMANCE_SCHEMA` or `PROFILING` to gain accurate execution time in MySQL, however you cannot use this for comparison with Readyset. You will never see the true response time because the actual execution time of your query occurs within Readyset and is never sent to the MySQL server.

## After caching

You can create a cache using the `CREATE CACHE` command.
```
CREATE CACHE FROM SELECT count(*) FROM `name`;
```

Subsequent executions of this query would hit the Readyset cache and not goto the MySQL server. Using the MySQL client you can only determine the execution is < 10ms.

```
SELECT COUNT(*) FROM name;
+----------+
| count(*) |
+----------+
| 14235647 |
+----------+
1 row in set (0.00 sec)

SELECT COUNT(*) FROM name;
+----------+
| count(*) |
+----------+
| 14235647 |
+----------+
1 row in set (0.00 sec)

SELECT COUNT(*) FROM name;
+----------+
| count(*) |
+----------+
| 14235647 |
+----------+
1 row in set (0.00 sec)
```

## MySQL Instrumentation

If you are performing a read-only test you will find the absence of information for measurement of SELECT workloads. Examples you may have used include:

### Select Counter

Running a `sysbench` there are no SELECTs executed against the database.

```
SHOW GLOBAL STATUS LIKE 'Com_select';
+---------------+------------+
| Variable_name | Value      |
+---------------+------------+
| Com_select    | 1677858352 |
+---------------+------------+
1 row in set (0.00 sec)

...

SHOW GLOBAL STATUS LIKE 'Com_select';
+---------------+------------+
| Variable_name | Value      |
+---------------+------------+
| Com_select    | 1677858352 |
+---------------+------------+
1 row in set (0.00 sec)
```

### Processlist

While Readyset makes a connection to the database, they remain idle (i.e. Sleep > 0). In this example there are 48 connections during a `sysbench` test.

```
SHOW PROCESSLIST;
+--------+-----------------+--------------+-------+-------------+--------+-----------------------------------------------------------------+------------------+
| Id     | User            | Host         | db    | Command     | Time   | State                                                           | Info             |
+--------+-----------------+--------------+-------+-------------+--------+-----------------------------------------------------------------+------------------+
|      5 | event_scheduler | localhost    | NULL  | Daemon      | 396481 | Waiting on empty queue                                          | NULL             |
| 152517 | readyset        | server:55084 | mysql | Sleep       |   6736 |                                                                 | NULL             |
| 152525 | readyset        | server:37876 | imdb  | Sleep       |   2943 |                                                                 | NULL             |
| 152531 | readyset        | server:46104 | mysql | Binlog Dump |   2830 | Source has sent all binlog to replica; waiting for more updates | NULL             |
| 152582 | rbradfor        | localhost    | NULL  | Query       |      0 | init                                                            | SHOW PROCESSLIST |
| 152583 | readyset        | server:37290 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152584 | readyset        | server:37300 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152585 | readyset        | server:37316 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152586 | readyset        | server:37332 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152587 | readyset        | server:37330 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152588 | readyset        | server:37354 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152589 | readyset        | server:37350 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152590 | readyset        | server:37342 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152591 | readyset        | server:37372 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152592 | readyset        | server:37360 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152593 | readyset        | server:37388 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152594 | readyset        | server:37380 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152595 | readyset        | server:37402 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152596 | readyset        | server:37404 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152597 | readyset        | server:37420 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152598 | readyset        | server:37432 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152599 | readyset        | server:37446 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152600 | readyset        | server:37452 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152601 | readyset        | server:37462 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152602 | readyset        | server:37470 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152603 | readyset        | server:37478 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152604 | readyset        | server:37494 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152605 | readyset        | server:37512 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152606 | readyset        | server:37510 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152607 | readyset        | server:37526 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152608 | readyset        | server:37554 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152609 | readyset        | server:37564 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152610 | readyset        | server:37538 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152611 | readyset        | server:37566 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152612 | readyset        | server:37570 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152613 | readyset        | server:37578 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152614 | readyset        | server:37594 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152615 | readyset        | server:37610 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152616 | readyset        | server:37620 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152617 | readyset        | server:37636 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152618 | readyset        | server:37642 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152619 | readyset        | server:37650 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152620 | readyset        | server:37662 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152621 | readyset        | server:37676 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152622 | readyset        | server:37690 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152623 | readyset        | server:37692 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152624 | readyset        | server:37700 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152625 | readyset        | server:37716 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152626 | readyset        | server:37732 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152627 | readyset        | server:37738 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152628 | readyset        | server:37764 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152629 | readyset        | server:37754 | imdb  | Sleep       |    462 |                                                                 | NULL             |
| 152630 | readyset        | server:37778 | imdb  | Sleep       |    462 |                                                                 | NULL             |
+--------+-----------------+--------------+-------+-------------+--------+-----------------------------------------------------------------+------------------+
53 rows in set, 1 warning (0.00 sec)
```

### Performance Schema

If you are looking for a certain SQL pattern using the Performance Schema, with a normal sysbench test you could find counter information including:

```
SELECT 'readyset' AS source, n.* FROM name n WHERE n.name = ?;
```

```
SELECT * FROM sys.statement_analysis WHERE db='imdb' AND query LIKE '%SOURCE%'\G
                query: SELECT ? AS SOURCE , `n` . * F ... AME `n` WHERE `n` . `name` = ?
                   db: imdb
            full_scan:
           exec_count: 212866
            err_count: 0
           warn_count: 0
        total_latency: 2.87 min
          max_latency: 197.23 ms
          avg_latency: 808.92 us
         lock_latency: 1.70 min
          cpu_latency:   0 ps
            rows_sent: 908259
        rows_sent_avg: 4
        rows_examined: 908259
    rows_examined_avg: 4
        rows_affected: 0
    rows_affected_avg: 0
           tmp_tables: 0
      tmp_disk_tables: 0
          rows_sorted: 0
    sort_merge_passes: 0
max_controlled_memory: 85.64 KiB
     max_total_memory: 105.74 KiB
               digest: 7d37aaad2cb326e68747ef52176bcfabcf1fbabab294ddc0e9c353e289098405
           first_seen: 2025-02-03 15:50:28.643240
            last_seen: 2025-02-03 15:51:28.657947
```

If you are running a read-only Readyset benchmark, there is no performance schema data to look at.

```
15:56:08 (8.0.40) [imdb]  > select * from sys.statement_analysis where db='imdb' AND query like '%SOURCE%'\G
Empty set (0.01 sec)
```

## Using `sysbench`

One way to observe the improved performance is to run a test with a more significant number of repeated executions.

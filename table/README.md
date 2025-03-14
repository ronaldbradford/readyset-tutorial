# Adding Tables to Readyset

Before you can cache queries, the underlying tables must first be included as replicating tables in the Readyset cache.
You may have configured Readyset to cache all the tables in a given schema, or you may have selectively choose some tables or no tables to initially include in your installation.

## Reviewing Existing Tables

You can use the `SHOW READYSET TABLES` command to see which tables Readyset is tracking changes for.

```
SHOW READYSET TABLES;
+--------------------------+-------------+-------------+
| table                    | status      | description |
+--------------------------+-------------+-------------+
| `imdb`.`name`            | Snapshotted |             |
| `imdb`.`title`           | Snapshotted |             |
| `imdb`.`title_principal` | Snapshotted |             |
+--------------------------+-------------+-------------+
3 rows in set (0.01 sec)
```

## Adding a New Table

If you want to include an additional table you can use the `ALTER READYSET ADD TABLES` command.

```
ALTER READYSET ADD TABLES imdb.name;
Query OK, 0 rows affected (0.00 sec)
```

As the command syntax indicates, you can add multiple tables at one time. For example:

```
ALTER READYSET ADD TABLES imdb.title, imdb.title_principal;
Query OK, 0 rows affected (0.00 sec)
```

## Resnapshotting a Table

If there is any potential issue in the synchronization of a table in the Readyset cache, you can resnapshot it with the command:

```
ALTER READYSET RESNAPSHOT TABLE imdb.name;
Query OK, 0 rows affected (0.00 sec)
```

The table will be removed, and you can monitor the status with the `SHOW READYSET TABLES` command.

```
SHOW READYSET TABLES;
+--------------------------+-------------+-------------+
| table                    | status      | description |
+--------------------------+-------------+-------------+
| `imdb`.`title`           | Snapshotted |             |
| `imdb`.`title_principal` | Snapshotted |             |
+--------------------------+-------------+-------------+
2 rows in set (0.00 sec)

...

SHOW READYSET TABLES;
+--------------------------+--------------+-------------+
| table                    | status       | description |
+--------------------------+--------------+-------------+
| `imdb`.`name`            | Snapshotting |             |
| `imdb`.`title`           | Snapshotted  |             |
| `imdb`.`title_principal` | Snapshotted  |             |
+--------------------------+--------------+-------------+
3 rows in set (0.01 sec)

...
SHOW READYSET TABLES;
+--------------------------+-------------+-------------+
| table                    | status      | description |
+--------------------------+-------------+-------------+
| `imdb`.`name`            | Snapshotted |             |
| `imdb`.`title`           | Snapshotted |             |
| `imdb`.`title_principal` | Snapshotted |             |
+--------------------------+-------------+-------------+
3 rows in set (0.00 sec)
```

## Removing a Table From Readyset

Presently there is no way to remove a table that is tracked by Readyset unlike removing proxied queries or cached queries.

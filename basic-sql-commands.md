---
title: Basic SQL Commands
toc: false
---

This page walks you through some of the most essential CockroachDB SQL commands. For a complete list of commands and their options, as well as details about data types and other concepts, see [SQL Reference](sql-reference.html).

{{site.data.alerts.callout_info}}CockroachDB aims to provide standard SQL with extensions, but some standard SQL functionality is yet not available in our Beta version. Joins and interleaved tables, for example, will be built into v 1.0. See our <a href="https://github.com/cockroachdb/cockroach/issues/2132">Product Roadmap</a> for more details.{{site.data.alerts.end}}   

<img src="images/catrina_ramen.png" style="max-width: 200px;" />

## Create a Database

CockroachDB comes with a single default `system` database, which contains CockroachDB metadata and is read-only. To create a new database, use the [`CREATE DATABASE`](create-database.html) command followed by a database name:

```postgres
CREATE DATABASE db1;
```

Database names must follow [these rules](identifiers.html). To avoid an error in case the database already exists, you can include `IF NOT EXISTS`:

```postgres
CREATE DATABASE IF NOT EXISTS db1;
```

When you no longer need a database, use the [`DROP DATABASE`](drop-database.html) command followed by the database name to remove the database and all its objects:

```postgres
DROP DATABASE db1;
```

## Show Available Databases

To see all available databases, use the [`SHOW DATABASES`](show-databases.html) command:

```postgres
SHOW DATABASES;
```
```
+----------+
| Database |
+----------+
| db1      |
| system   |
+----------+
```

## Set the Default Database

To set the default database, use the [`SET DATABASE`](set-database.html) command:

```postgres
SET DATABASE = db1;
```

When working with the default database, you don't need to reference it explicitly in statements. To see which database is currently the default, use the `SHOW DATABASE` command (note the singular form):

```postgres
SHOW DATABASE;
```
```
+----------+
| DATABASE |
+----------+
| db1      |
+----------+
```

## Create a Table

To create a table, use the [`CREATE TABLE`](create-table.html) command followed by a table name, the column names, the data type for each column, and the primary key constraint:

```postgres
CREATE TABLE table1 (
    column_a         INT PRIMARY KEY,
    column_b         FLOAT,
    column_c         DATE,
    column_d         STRING
);
```

Table and column names must follow [these rules](identifiers.html). Also, when you don't explicitly define a `PRIMARY KEY`, CockroachDB will automatically add a `rowid` column as the primary key.

To avoid an error in case the table already exists, you can include `IF NOT EXISTS`:

```postgres
CREATE TABLE IF NOT EXISTS table1 (
    column_a         INT PRIMARY KEY,
    column_b         FLOAT,
    column_c         DATE,
    column_d         STRING
);
```

To verify that all columns were created correctly, use the [`SHOW COLUMNS FROM`](show-columns.html) command followed by the table name:

```postgres
SHOW COLUMNS FROM table1;
```
```
+----------+--------+-------+---------------------------+
|  Field   |  Type  | Null  |          Default          |
+----------+--------+-------+---------------------------+
| column_a | INT    | true  | NULL                      |
| column_b | FLOAT  | true  | NULL                      |
| column_c | DATE   | true  | NULL                      |
| column_d | STRING | true  | NULL                      |
+----------+--------+-------+---------------------------+
```

To verify the primary index for a table, use the [`SHOW INDEX FROM`](show-index.html) command followed by the name of the table:

```postgres
SHOW INDEX FROM table1;
```
```
+----------+-----------+--------+-----+------------+-----------+---------+
|  Table   |   Name    | Unique | Seq |   Column   | Direction | Storing |
+----------+-----------+--------+-----+------------+-----------+---------+
| "table1" | "primary" | true   |   1 | "column_a" | "ASC"     | false   |
+----------+-----------+--------+-----+------------+-----------+---------+
```

When you no longer need a table, use the [`DROP TABLE`](drop-table.html) command followed by the table name to remove the table and all its data:

```postgres
DROP TABLE table1;
```

## Show Available Tables

To see all tables in the active database, use the [`SHOW TABLES`](show-tables.html) command:

```postgres
SHOW TABLES;
```
```
+--------+
| Table  |
+--------+
| table1 |
| table2 |
+--------+
```

To view tables in a database that's not active, use `SHOW TABLES FROM` followed by the name of the database:

```postgres
SHOW TABLES FROM db2;
```
```
+------------+
|   Table    |
+------------+
| aardvarks  |
| elephants  |
| frogs      |
| moles      |
| pandas     |
| turtles    |
+------------+
```

## Insert Rows into a Table

To insert a row into a table, use the [`INSERT INTO`](insert.html) command followed by the table name and then the column values listed in the order in which the columns appear in the table:

```postgres
INSERT INTO table1 VALUES (12345, 1.1, DATE '2016-01-01', 'aaaa');
```

If you want to pass column values in a different order, list the column names explicitly and provide the column values in the same order:

```postgres
INSERT INTO table1 (column_d, column_c, column_b, column_a) VALUES 
    ('aaaa', DATE '2016-01-01', 1.1, 12345);
```

To insert multiple rows into a table, use a comma-separated list of parentheses, each containing column values for one row:

```postgres
INSERT INTO table1 (column_d, column_c, column_b, column_a) VALUES 
    ('aaaa', DATE '2016-01-01', 1.1, 12345),
    ('bbbb', DATE '2016-01-01', 1.2, 54321);
```

[Defaults values](default-values.html) are used when you leave specific columns out of your statement, or when you explicitly request default values. For example, either of the following statements would create a row with `column_b` filled with its default value, in this case `NULL`:

```postgres
INSERT INTO table1 (column_d, column_c, column_a) VALUES 
    ('aaaa', DATE '2016-01-01', 12345);

INSERT INTO table1 (column_d, column_c, column_b, column_a) VALUES 
    ('aaaa', DATE '2016-01-01', DEFAULT, 12345);
```
```
+----------+----------+------------+----------+
| column_a | column_b |  column_c  | column_d |
+----------+----------+------------+----------+
| 12345    |     NULL | 2016-01-01 | aaaa     |
+----------+----------+------------+----------+
```

## Query a Table



## Update Rows in a Table

To update rows in a table, use the [`UPDATE`](update.html) command followed by the table name, a `SET` clause specifying the columns to update and their new values, and a `WHERE` clause identifying the rows to update:

```postgres
UPDATE table1 SET column_a = column_a + 1, column_d = 'cccc' WHERE column_b > 0;
```

If a table has a primary key, you can use that in the `WHERE` clause to reliably update specific rows; otherwise, each row matching the `WHERE` clause is updated. When there's no `WHERE` clause, all rows in the table are updated. 

## Delete Rows in a Table

To delete rows in a table, use the [`DELETE FROM`](delete.md) command followed by the table name and a `WHERE` clause identifying the rows to delete: 

```postgres
DELETE FROM table1 WHERE column_b = 2.5;
```

Just as with the `UPDATE` command, if a table has a primary key, you can use that in the `WHERE` clause to reliably delete specific rows; otherwise, each row matching the `WHERE` clause is deleted. When there's no `WHERE` clause, all rows in the table are deleted. 
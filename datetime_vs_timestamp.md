### Introduction



**`DATETIME`**

`DATETIME` can store the date and time value that ranges from `1000-01-01 00:00:00` to `9999-12-31 23:59:59`, and is displayed and retrieved in . It is independent of the time-zone and occupied 8 bytes of storage space.



**`TIMESTAMP`**

As its name implies, `TIMESTAMP` stores the number of seconds elapsed since midnight, January 1, 1970, Greenwich Mean Time (GMT)—the same as a Unix timestamp. And it ranges from `1970-01-01 00:00:01 UTC` to `2038-01-19 03:14:07 UTC`, so it is smaller than `DATETIME` and just occupies 4 bytes.



Because it is related to the time-zone, Mysql Server needs to convert `TIMESTAMP` from the current time zone to UTC for storage, and back from UTC to the current time zone for retrieval. The current time zone is the Mysql Server time zone; therefore, if you change the time zone of the server after store data, you will get back the different value from the value you stored.



you can check the current time zone:

```mysql
SHOW VARIABLES LIKE '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | EDT    |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set (0.00 sec)
```



Both of them can store and display the date and time value in `YYYY-MM-DD hh:mm:ss` format, as well as they can include a trailing fractional seconds part in up to microseconds (6 digits) precision. Therefore, their format with precision is `YYYY-MM-MM hh:mm:ss[.fraction]` and the range of `DATETIME` is `1000-01-01 00:00:00.000000` to `9999-12-31 23:59:59.999999 `, the range of `TIMESTAMP` is `1970-01-01 00:00:01.000000 UTC` to `2038-01-19 03:14:07.999999 UTC`



**Test:**

1. create two separated tables for testing datetime and timestamp.

```mysql

mysql> CREATE TABLE test_timestamp (
    ->     id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
    ->     col TIMESTAMP NOT NULL
    -> ) ENGINE=InnoDB CHARSET=utf8mb4;
Query OK, 0 rows affected (0.02 sec)

mysql> CREATE TABLE test_datetime (
    ->     id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    ->     col DATETIME NOT NULL
    -> ) ENGINE=InnoDB CHARSET=utf8mb4;
Query OK, 0 rows affected (0.00 sec)
```

2. check the time zone variable.

```mysql
mysql> SET @@time_zone = 'SYSTEM';
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | EDT    |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set (0.00 sec)
```

3. insert data into these two tables and display them.

```mysql
mysql> INSERT INTO test_timestamp (col) VALUES ('2020-01-01 10:10:10');
Query OK, 1 row affected (0.00 sec)

mysql>
mysql> INSERT INTO test_datetime (col) VALUES ('2020-01-01 10:10:10');
Query OK, 1 row affected (0.00 sec)

mysql>
mysql> SELECT col, UNIX_TIMESTAMP(col) FROM test_timestamp ORDER BY id;
+---------------------+---------------------+
| col                 | UNIX_TIMESTAMP(col) |
+---------------------+---------------------+
| 2020-01-01 10:10:10 |          1577891410 |
+---------------------+---------------------+
1 row in set (0.00 sec)

mysql>
mysql> SELECT col, UNIX_TIMESTAMP(col) FROM test_datetime ORDER BY id;
+---------------------+---------------------+
| col                 | UNIX_TIMESTAMP(col) |
+---------------------+---------------------+
| 2020-01-01 10:10:10 |          1577891410 |
+---------------------+---------------------+
1 row in set (0.00 sec)
```

4. change the time zone

```mysql
mysql> SET @@time_zone = '+8:00';
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | EDT    |
| time_zone        | +08:00 |
+------------------+--------+
2 rows in set (0.00 sec)
```

5. display them again. 

```mysql
mysql> SELECT col, UNIX_TIMESTAMP(col) FROM test_datetime ORDER BY id;
+---------------------+---------------------+
| col                 | UNIX_TIMESTAMP(col) |
+---------------------+---------------------+
| 2020-01-01 10:10:10 |          1577844610 |
+---------------------+---------------------+
1 row in set (0.00 sec)

mysql> SELECT col, UNIX_TIMESTAMP(col) FROM test_timestamp ORDER BY id;
+---------------------+---------------------+
| col                 | UNIX_TIMESTAMP(col) |
+---------------------+---------------------+
| 2020-01-01 23:10:10 |          1577891410 |
+---------------------+---------------------+
1 row in set (0.00 sec)
```





### `Zero Date ` VS `NULL`

We always see the following codes before the version of Mysql 5.6.

```mysql
mysql> CREATE TABLE `test_zero_date` (
    ->   `id` int NOT NULL AUTO_INCREMENT,
    ->   `create_date` DATETIME NOT NULL DEFAULT '0000-00-00 00:00:00',
    ->   PRIMARY KEY (`id`)
    -> ) ENGINE=InnoDB CHARSET=utf8mb4;
```

However, we should use `NULL` instead of `Zero Date`.  Because `Zero Date` is not a valid date and it leads to a lot of problems (such as JDBC can refuse to load it).



After Mysql 5.6, Mysql disallows the `Zero Date` as default values by default, unless you configure `sql_mode`. As the official documents mentioned:

> MySQL permits you to store a “zero” value of `'0000-00-00'` as a “dummy date.” In some cases, this is more convenient than using `NULL` values, and uses less data and index space. To disallow `0000-00-00`, enable the [`NO_ZERO_DATE`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_zero_date) mode.



1. By default, we got error message when we create one table with `Zero Date`.

   ```mysql
   mysql> CREATE TABLE `test_zero_date` (
       ->   `id` int NOT NULL AUTO_INCREMENT,
       ->   `ship_date` DATETIME NOT NULL DEFAULT '0000-00-00 00:00:00',
       ->   PRIMARY KEY (`id`)
       -> ) ENGINE=InnoDB CHARSET=utf8mb4;
   ERROR 1067 (42000): Invalid default value for 'ship_date'
   ```

2. Configure `sql_mode`

   ```mysql
   mysql> SHOW VARIABLES LIKE '%sql_mode%';
   +---------------+-------+
   | Variable_name | Value |
   +---------------+-------+
   | sql_mode      | ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION   |
   +---------------+-------+
   1 row in set (0.00 sec)
   
   mysql> SET sql_mode = '';
   Query OK, 0 rows affected (0.00 sec)
   
   mysql>
   mysql>
   mysql> SHOW VARIABLES LIKE '%sql_mode%';
   +---------------+-------+
   | Variable_name | Value |
   +---------------+-------+
   | sql_mode      |       |
   +---------------+-------+
   1 row in set (0.01 sec)
   ```

3. Create the table again and insert data

   ```mysql
   mysql> CREATE TABLE `test_zero_date` (
       ->   `id` int NOT NULL AUTO_INCREMENT,
       ->   `ship_date` DATETIME NOT NULL DEFAULT '0000-00-00 00:00:00',
       ->   PRIMARY KEY (`id`)
       -> ) ENGINE=InnoDB CHARSET=utf8mb4;
   Query OK, 0 rows affected (0.02 sec)
   
   mysql> INSERT INTO test_zero_date VALUES();
   Query OK, 1 row affected (0.00 sec)
   
   mysql> INSERT INTO test_zero_date (ship_date) VALUES('2020-05-25');
   Query OK, 1 row affected (0.00 sec)
   
   mysql> SELECT * FROM test_zero_date;
   +----+---------------------+
   | id | ship_date           |
   +----+---------------------+
   |  1 | 0000-00-00 00:00:00 |
   |  2 | 2020-05-25 00:00:00 |
   +----+---------------------+
   2 rows in set (0.00 sec)
   ```

4. valid the result.

   ```mysql
   mysql> SELECT * FROM test_zero_date WHERE ship_date is NULL;
   +----+---------------------+
   | id | ship_date           |
   +----+---------------------+
   |  1 | 0000-00-00 00:00:00 |
   +----+---------------------+
   1 row in set (0.01 sec)
   
   mysql> SELECT * FROM test_zero_date ORDER BY ship_date DESC;
   +----+---------------------+
   | id | ship_date           |
   +----+---------------------+
   |  2 | 2020-05-25 00:00:00 |
   |  1 | 0000-00-00 00:00:00 |
   +----+---------------------+
   2 rows in set (0.01 sec)
   
   mysql> SELECT * FROM test_zero_date WHERE ship_date < '2020-05-25';
   +----+---------------------+
   | id | ship_date           |
   +----+---------------------+
   |  1 | 0000-00-00 00:00:00 |
   +----+---------------------+
   1 row in set (0.01 sec)
   ```

   Although we can use it after configuring `sql_mode`, it is still confusing about the following questions among a lot of people.

   1. Does '0000-00-00' < '2018-12-12'?
   2. Does '0000-00-00' means 'unknown'?
   3. Does '0000-00-00' means 'null'?

   Unless you exactly know these and need to do comparisons with `NULL` dates, please use the `NULL` to present the empty dates.

   

### Automatic Initialization and Updating

Before Mysql 5.6.5, only `TIMESTAMP` can be automatically initialized and updated to the current date and time, and for at most one `TIMESTAMP` column per table.



After this version, for any `TIMESTAMP` or `DATETIME` column in a table, you can assign the current timestamp as the default value, the auto-update value, or both. 



To specify automatic property, use the `DEFAULT CURRENT_TIMESTAMP` and `ON UPDATE CURRENT_TIMESTAMP` clauses in column definitions. No matter the order, and any synonyms of `CURRENT_TIMESTAMP` has the same effect ( `CURRENT_TIMESTAMP()`, `NOW()`, `LOCALTIME`, `LOCALTIME()`, `LOCALTIMESTAMP`, and `LOCALTIMESTAMP()`). 

```mysql
mysql> CREATE TABLE `test_timestamp_automatic_init_update` (
    ->   `id` int NOT NULL AUTO_INCREMENT,
    ->   `col` varchar(10) NOT NULL,
    ->   `ts1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    ->   PRIMARY KEY (`id`)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO test_timestamp_automatic_init_update (col) VALUES ('hello');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM test_timestamp_automatic_init_update;
+----+-------+---------------------+
| id | col   | ts1                 |
+----+-------+---------------------+
|  1 | hello | 2020-05-26 09:06:25 |
+----+-------+---------------------+
1 row in set (0.00 sec)

mysql> UPDATE test_timestamp_automatic_init_update SET col = 'hello!!' WHERE id = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT * FROM test_timestamp_automatic_init_update;
+----+---------+---------------------+
| id | col     | ts1                 |
+----+---------+---------------------+
|  1 | hello!! | 2020-05-26 09:08:03 |
+----+---------+---------------------+
1 row in set (0.00 sec)
```



if the `explicit_defaults_for_timestamp` system variable is disabled (enabled by default), you can initialize and update any `TIMESTAMP` (but not `DATETIME`) column to the current date and time by assigning it a `NULL` value.

```mysql
mysql> SHOW VARIABLES LIKE '%explicit_defaults_for_timestamp%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| explicit_defaults_for_timestamp |  ON   |
+---------------------------------+-------+
1 row in set (0.00 sec)

mysql> SET @@explicit_defaults_for_timestamp = 'OFF';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> CREATE TABLE `test_explicit_defaults_for_timestamp` (
    ->   `id` int NOT NULL AUTO_INCREMENT,
    ->   `ts1` timestamp,
    ->   PRIMARY KEY (`id`)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO test_explicit_defaults_for_timestamp VALUES ();
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM test_explicit_defaults_for_timestamp;
+----+---------------------+
| id | ts1                 |
+----+---------------------+
|  1 | 2020-05-26 09:18:08 |
+----+---------------------+
1 row in set (0.00 sec)

mysql> SHOW CREATE TABLE test_explicit_defaults_for_timestamp\G
*************************** 1. row ***************************
       Table: test_explicit_defaults_for_timestamp
Create Table: CREATE TABLE `test_explicit_defaults_for_timestamp` (
  `id` int NOT NULL AUTO_INCREMENT,
  `ts1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.00 sec)
```



If the `explicit_defaults_for_timestamp`  is disabled,  the  the `TIMESTAMP` is `NOT NULL` by default. And the first `TIMESTAMP` column has  `NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP`  automatically.  other `TIMESTAMP` column must be declared with `NULL` or `NOT NULL` .

```mysql
mysql> SET @@explicit_defaults_for_timestamp = 'OFF';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> CREATE TABLE `test_explicit_defaults_for_timestamp_1` (
    ->   `id` int NOT NULL AUTO_INCREMENT,
    ->   `ts1` timestamp,
    ->   `ts2` timestamp,
    ->   PRIMARY KEY (`id`)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
ERROR 1067 (42000): Invalid default value for 'ts2'

mysql> CREATE TABLE `test_explicit_defaults_for_timestamp_1` (
    ->   `id` int NOT NULL AUTO_INCREMENT,
    ->   `ts1` timestamp,
    ->   `ts2` timestamp NULL,
    ->   PRIMARY KEY (`id`)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
Query OK, 0 rows affected (0.01 sec)

mysql> show create table test_explicit_defaults_for_timestamp_1\G
*************************** 1. row ***************************
       Table: test_explicit_defaults_for_timestamp_1
Create Table: CREATE TABLE `test_explicit_defaults_for_timestamp_1` (
  `id` int NOT NULL AUTO_INCREMENT,
  `ts1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `ts2` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
1 row in set (0.01 sec)
```



### `NOW()` VS `SYSDATE()`

`NOW([fsp])`:

Return the current date and time in `'YYYY-MM-DD hh:mm:ss'` or `YYYYMMDDhhmmss` format which depend on whether the function is used to string or numeric context.

```mysql
mysql> SELECT NOW(), NOW() + 0;
+---------------------+----------------+
| NOW()               | NOW() + 0      |
+---------------------+----------------+
| 2020-05-26 11:41:11 | 20200526114111 |
+---------------------+----------------+
1 row in set (0.00 sec)
```



And it is expressed in the session time zone setting.

```mysql
mysql> SHOW VARIABLES LIKE '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | EDT    |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set (0.00 sec)

mysql> SET @@time_zone = '+08:00';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT NOW();
+---------------------+
| NOW()               |
+---------------------+
| 2020-05-26 23:43:13 |
+---------------------+
1 row in set (0.00 sec)

mysql> set @@time_zone = 'SYSTEM';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT NOW();
+---------------------+
| NOW()               |
+---------------------+
| 2020-05-26 11:43:36 |
+---------------------+
1 row in set (0.00 sec)
```



`fsp` is to specify the fractional seconds precisions from 0 to 6.

```mysql
mysql> SELECT NOW(6);
+----------------------------+
| NOW(6)                     |
+----------------------------+
| 2020-05-26 11:46:25.153487 |
+----------------------------+
1 row in set (0.00 sec)
```



It returns the constant value at which the function or triggering statement began to execute.

```mysql
mysql> SELECT NOW(), SLEEP(1), NOW();
+---------------------+----------+---------------------+
| NOW()               | SLEEP(1) | NOW()               |
+---------------------+----------+---------------------+
| 2020-05-26 11:49:04 |        0 | 2020-05-26 11:49:04 |
+---------------------+----------+---------------------+
1 row in set (1.01 sec)
```



`CURRENT_TIMESTAMP`,  `CURRENT_TIMESTAMP()`, `LOCALTIMESTAMP`, `LOCALTIMESTAMP()`, `LOCALTIME`, `LOCALTIME()`  are synonyms for `NOW()`.



`SYSDATE([fsp])`:

It has same effects as `NOW()` except returning the value which it executes every time.

```mysql
mysql> SELECT SYSDATE(), SLEEP(1), SYSDATE();
+---------------------+----------+---------------------+
| SYSDATE()           | SLEEP(1) | SYSDATE()           |
+---------------------+----------+---------------------+
| 2020-05-26 11:55:07 |        0 | 2020-05-26 11:55:08 |
+---------------------+----------+---------------------+
1 row in set (1.00 sec)
```



In general, please use `NOW()` because it gives one constant date value in one transaction and `SYSDATE()` has some other problems.



### References

- https://dev.mysql.com/doc/refman/8.0/en/date-and-time-types.html
- https://dev.mysql.com/doc/refman/8.0/en/timestamp-initialization.html
- https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_now
- https://www.cnblogs.com/ivictor/p/5028368.html
- https://stackoverflow.com/questions/12482560/better-to-use-zero-date-0000-00-00-000000-or-null-in-mysql/55358724#comment16793593_12482560
- https://dba.stackexchange.com/questions/139896/null-instead-0000-00-00-000000-in-mysql-5-7
- https://stackoverflow.com/questions/36374335/error-in-mysql-when-setting-default-value-for-date-or-datetime
- https://www.whatmarkdid.com/web-servers/allowing-0000-00-00-default-value-mysql-datetime-column/
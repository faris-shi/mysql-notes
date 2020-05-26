We always use `ALTER COLUMN`, `MODIFY COLUMN` and `CHANGE COLUMN` for altering a table to change columns' name, size, datatype and even the default value.



### `CHAHGE COLUMN`

`CHANGE COLUMN` is used to rename one existing column name or its datatype if you find it is named incorrectly.

```mysql
ALTER TABLE film CHANGE COLUMN rental_duration rental_durations TINYINT(3) NOT NULL DEFAULT 5;
```



### `MODIFY COLUMN`

`MODIFY COLUMN` can do everything that `CHANGE COLUMN` can do, except renaming column names.

```mysql
ALTER TABLE film MODIFY COLUMN rental_duration TINYINT(3) NOT NULL DEFAULT 5;
```



### `ALTER COLUMN`

It can be used to set, reset or remove the default value for a column.

```mysql
ALTER TABLE film ALTER COLUMN rental_duration SET DEFAULT 5;

ALTER TABLE film ALTER COLUMN retail_duration DROP DEFAULT;
```



### TIPS

If you only want to change the default value for one column in a huge table, please use `ALTER COLUMN` in high priority, although other two operations can do it as well, they can cause table rebuild.



### References

- https://stackoverflow.com/questions/14767174/modify-column-vs-change-column
- https://dev.mysql.com/doc/refman/8.0/en/alter-table.html
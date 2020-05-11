# MySQL

## indexs

### Clustered indexs
1. When you define a PRIMARY KEY on your table, InnoDB uses it as the clustered index. Define a primary key for each table that you create. If there is no logical unique and non-null column or set of columns, add a new auto-increment column, whose values are filled in automatically.

2. If you do not define a PRIMARY KEY for your table, MySQL locates the first UNIQUE index where all the key columns are NOT NULL and InnoDB uses it as the clustered index.

3. If the table has no PRIMARY KEY or suitable UNIQUE index, InnoDB internally generates a hidden clustered index named GEN_CLUST_INDEX on a synthetic column containing row ID values. The rows are ordered by the ID that InnoDB assigns to the rows in such a table. The row ID is a 6-byte field that increases monotonically as new rows are inserted. Thus, the rows ordered by the row ID are physically in insertion order.

### Secondary indexs
All indexes other than the clustered index are known as secondary indexes. In InnoDB, **each record in a secondary index contains the primary key columns for the row**, as well as the columns specified for the secondary index. **InnoDB uses this primary key value to search for the row in the clustered index.**


## explain

### Indexs don't work
```SQL
explain select * from se where id - 1 > 20000;
1	SIMPLE	se	ALL	279398                          	Using where
```
```SQL
explain select * from se where id > 19999;
1	SIMPLE	se	range	PRIMARY	PRIMARY	5		139699	Using where
```
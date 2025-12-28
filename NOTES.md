row-oriented vs column-oriented

### row-oriented Database

+ Tables are stored as rows in disk
+ A single block io read to the table fetches multiple rows with all their columns
+ More IOs are required to find a particular row in a table,scan but once you find the row you get all columns for that row

### column-oriented Database

+ Tables are stored as columns first in disk
+ A single block io read to the table fetches multiple columns with all matching rows
+ Less IOs are required to get more values of a given column. But working with multiple columns require more IOs 

### Pros & Cons

+ Row-based | Column-Based
+ Optimal for read/writes | Writes are slower
+ OLTP | OlAP
+ Compressing isn't efficient | Compress greatly
+ Aggregation isn't efficient | Amazing for Aggregation
+ efficient queries | Inefficient queries

## index

+ seq table scan
+ index scan
+ bitmap index scan(postgres)

### key vs non-key column indexes

```sql

explain (analyze,buffers) sql

create index g_idx on students(g) include (id);

vacuum (verbose) students;

```

## index scan vs index only scan

```sql

create index ig_idx on grades(id) include (name);
```


## best practises on creating indexes

```
create index on test(a);
create index on test(b);


explain analyze select c from test where a = 100 and b = 200;

explain analyze select c from test where a = 100 or b = 200;

create index on test(a,b);


select * from t where f1 = 1 and f2 = 4 /*+ index[f1_idx] */


create index concurrently g on grades(g);


```




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


## B-Trees

+ Balanced Data structure for fast traversal
+ B-Tree has nodes
+ In B-Tree of "M" detree some nodes can have (m) child nodes
+ Node has up to (m-1) elements
+ Each element has a key and a value
+ The value is usually data pointer to the row
+ Data pointer can point to primary key or tuple
+ Root noted, internal node and leaf nodes 
+ A node = disk page

## B+Tree

+ Exactly like B-Tree but only stores keys in internal notes
+ Values are only stored in leaf nodes
+ Internal nodes are smaller since they only store keys and they can fit more elements
+ Leaf nodes are "linked" so once you find a key you can find all values before and after that key


## B+Tree & DBMS Considerations

+ Cost of leaf pointer（cheap)
+ 1 Node fits a DBMS page(most DBMS)
+ Can fit internal nodes easily in memory for fast traveral
+ Leaf nodes can live in data files in the heap
+ Most DBMS systems use B+Tree

## Storage Cost in Postgres vs MySQL

+ B+Trees secondary index values can either point directly to the tuple(Postgres) or to the primary key(MySQL)
+ If the Primary key data type is expensive this can cause bloat in all secondary indexes for Database such as MySQL(InnoDB)
+ Leaf nodes in MySQL(InnoDB) contains the full row since its an LOT / clustered index



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


## Partition

### What is Partition

### Vertical vs Horizontal Partition

+ Horizontal Partitioning splits rows into partitions
  + Range or list
+ Vertical partitioning splits columns partitions
  + Large column(blob) that you can store in a slow access drive in its own tablespace

### Partition Type

+ By Range
  + Dates, ids(e.g. by logdate or customerid from to)
+ By List
  + Discret values(e.g. state CA, AL, etc) or zip code
+ By Hash
  + Hash functions(consistent hashing)

### Horizontal Partition vs Sharding

+ HP splits big table into multiple tables in the same Database, client is sgnostic+ Sharding split big table into multiple tables across multiple database servers
+ HP table name changes(or schema)
+ Sharding everything is the same but server changes

### demo

```bash
docker run --name pgmain -d -e POSTRES_PASSWORD=postgress postgres

docker exec -it pgmain bash
psql -U postgres

```

```sql
create table grades_org(id serial not null, g int not null);

insert into grades_org(g) select floor(random()*100) from generate_series(0, 10000000);

create index grades_org_index in grades_org(g);

explain analyze select count(*) from grades_org where g = 20;
explain analyze select count(*) from grades_org where g between 20 and 35;
create table grades_parts (id serial not null, g int not null) partition by range(g);
create table g0035 (like grades_parts including indexes);
create table g3560 (like grades_parts including indexes);
create table g6080 (like grades_parts including indexes);
create table g80100 (like grades_parts including indexes);

alter table grades_parts attach partition g0035 for values from (0) to (35);
alter table grades_parts attach partition g3560 for values from (35) to (60);
alter table grades_parts attach partition g6080 for values from (60) to (80);
alter table grades_parts attach partition g80100 for values from (80) to (100);

insert into grades_parts select * from grades_org;



select pg_relation_size(oid), relname from pg_class order by pg_relation_size(oid) desc;

show enable_partition_pruning(分区修剪);

set enable_partition_pruning=off;

explain select count(*)  from grades_parts where g=30;




```

### Pros of Partitioning

+ Improve query performance when accessing a single partition
+ Sequential scan vs scattered index scan
+ Easy bulk loading(attach partition)
+ Archive old data that are barely accessed into cheap storage

### Cons of Partitioning

+ Updates that move rows from a partition to another(slow or fail somethimes)
+ Inefficient queries could accidently scan all partitions resulting in slower performance
+ Schema changes can be challenging(DBMS could manage it though)


## Sharding

### consistent hashing

### Hash-ring

### Horizontal Partition vs Sharding

+ HP splits big table into multiple tables in the same database
+ Sharding splits big table into multiple tables across multiple database servers
+ HP table name changes(or schema)
+ Sharding everything is the same but server changes

### Pros of Sharding

+ Scalability
  + Data
  + Memory
+ Security(users can access certain shards)
+ Optimal and smaller index size

### Cons of Sharding

+ Complex client(aware of the shard)
+ Transactions across shards problem
+ Rollbacks
+ Schema changes are hard
+ Joins
+ Has to be something you know in the query

> https://vitess.io/


## Exclusive Lock vs Shared Lock

## Dead lock

```sql
begin Transactions;
insert into test values(21);
insert into test values(20);

```
   
```sql
begin Transactions;
insert into test values(20);
insert into test values(21);

```
### two phase locking

```sql
begin transaction;
select * from seats where id = 14 for update;
```

### Pagination with Offset is very slow
   
  
## Database replication

### Master/Backup replication

+ One Master/Leader node that accepts writes/ddls
+ One ore more backup/standby nodes that receive those writes from the Master
+ Simple to implement no conflicts

### Multi-Master replication

+ Multiple Master/Leader node that accepts writes/ddls
+ One ore more backup/follower nodes that receive those writes from the Masters
+ Need to resolves conflicts

### Synchronous vs Asynchronous Replication

+ Synchronous Replication, A write transaction to the master will be blocked until it is written to the backup/standby nodes
  + First 2, First 1 or Any
+ Asynchronous Rep, A write transaction is considered successful if it written to the master, then Asynchronously the writes are applied to backup nodes


```bash
docker run --name pmaster -p 5432:5423 -v /users/yaojack/rep/pmaster_data:/var/lib/postgressql/data -e POSTRES_PASSWORD=postgres -d postgres

docker run --name pstandby -p 5433:5423 -v /users/yaojack/rep/pstandby_data:/var/lib/postgressql/data -e POSTRES_PASSWORD=postgres -d postgres

```

### Pros and Cons of Replication

+ Pros
  + Horizontal Scaling
  + Region based queries - DB per region
+ Cons
  + Eventual Consistency
  + Slow writes(synchronous)
  + Complex to implement(multi-master)


## Database Engine

+ Library that takes care of the on disk storade and CRUD
+ Or As rich and complex as full support ACID and transactions with foreign keys
+ DBMS can use the database engine and build features on top(server,replication, isolation, stored procedures, etc)
+ Want to write a new database? Don't start from scrach use en engine
+ Sometimes refered as Storage Engine or embbeded database
+ Some DBMS gives you the flexibity to switch engines like MySQL & MariaDB
+ Some DBMS comes with a built-in engine that you can't change(Postgres)

> https://www.uber.com/en-TW/blog/postgres-to-mysql-migration/

### MyISAM 

+ Stands for indexec sequential access method
+ B-tree(Balanced tree) indexes point to the rows directly
+ No Transaction support
+ Open source & Owned by Oracle 
+ Inserts are fast, updates and deletes are problematic(fragments)
+ Database Crashes corrupts tables(have to manually repair)
+ Table level locking
+ MySQL, MariaDB, Percona(MySQL forks) supports MyISAM
+ Used to be the default engine for mysql

### InnoDB

+ B+tree - with indexes point to the primary key and the PK points to the row
+ Replaces MyISAM
+ Default for MySQL & MariaDB
+ ACID compliant transaction support
+ Foreign keys
+ tablespaces
+ Row level locking
+ Spatial operations
+ Owned by Oracle

### XtraDB

+ Fork of InnoDB
+ Was the default MariaDB until 10.1
+ In MariaDB 10.2 switched the defaut for InnoDB
"XtraDB couldn't be kept up to date with the latest features of InnoDB and cannot be used."
+ System tables in MariaDB starting with 10.4 ara all area

### SQLite

+ Designed by D.Richard Hipp in 2000
+ Very popular embbeded database for local data
+ B-Tree(LSM as extension)
+ Postgres-like syntax
+ Full ACID & table locking
+ concurrent read & write
+ Web SQL in browers uses it 
+ Included in many operation systems by default

### Aria

+ Created by Michael Widenius
+ Very similar to MyISAM
+ Crash-safe unlike MyISAM
+ Not owned by Oracle
+ Designed specifically for MariaDB(MySQL fork)
+ In MariaDB 10.4 all system tables are Aria

### Berkeley DB

+ Developed by Sleepycat Software in 1994(owned by Oracle)
+ Key-value embbeded database
+ Supports ACID transactions, locks, replications etc.
+ Used to be used in bitclin core (switched to LevelDB)
+ Used MemcachedDB

### LevelDB

+ written by Jeff and Sanjay from Google in 2011
+ Log structured merge tree(LSM) (great for high insert and SSD)
+ No Transactions
+ Inspired by Google BigTable
+ Levels of files
  + Memtable
  + Level 0(young level)
  + level 1 - 6
+ As files grow large levels are merged
+ Used in bitcoin core blockchain,AutoCad, Minecraft

### RocksDB

+ Facebook forked LevelDB in 2012 to become RocksDB
+ creator is Dhruba borthakur
+ Transactional
+ High performance, multi-threaded compaction
+ Many features not in LevelDB
+ MyRocks for MySQL, MariaDb and Percona
+ MongoRocks for MongoDB
+ many many more use it


> YugaByte DB
> https://www.yugabyte.com/blog/a-busy-developers-guide-to-database-storage-engines-the-basics/



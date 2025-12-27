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

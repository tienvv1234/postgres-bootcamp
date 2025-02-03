### create Login group roles

### Create database

### Insert

Dealing with sing quotes
```
INSERT INTO customer (first_name)
VALUES ('Bill''O Sullivan')
```

insert and returns all rows

```
INSERT ... VALUES ... RETURNING *
```

* here is returning those rows have effected

insert and returns only fields

```
INSERT ... VALUES ... RETURNING fieldname
```

update statements similar to insert

UPSERT (update or insert)
```
INSERT INTO customer (first_name)
VALUES ('Bill''O Sullivan')
ON CONFLICT
DO
    UPDATE SET
        tag = EXCLUDE.tag
        update_date = NOW()
```

behide the sence
```
from --> where --> group by --> having --> Select --> distict --> order by --> limit
```

### Fetch 
1. Fetch clause to retrueve a portion ò rớ returned bu a query

### HAVING vs WHERE
- HAVING works on result group
- WHERE works on SELECT columns and not on result group

### JOIN
- For inner joins: all common columns defined at ON must match values on both tables

### INNER JOIN with USING
- We use USING only when joining tables have the same columns names, rather than on!!
lets say the same column name is column1
```
INTER JOIN table2 USING (column1)
```

### Index
What is an index?

1. An index is a structured relation

2. An index help imporove the access of data in our database

3. Indexed tuples point to the table page where the tuple is stored.  (tuples is basically a location whereby an actual data gets stored with index entry, so whenever we are searching a data, what the system does is actually looks for that data in tuples and then look for a particular ID in terms of where its position is in the tuples )

4. An index is a data structure that allow faster access to the underlying table so that specific tuples can be found quickly. Here, 'quickly' means faster than scanning the whole underlying table and analyzing every single tuple.(So index is basically to avoid the situation whereby you have to scan the whole underlying table and analyze every single tuple)

5. Maintaning an index is a fundamental for having a good performance.

6. Performance tuning is one of the most complex tasks in the daily job of a database administrator.

7. Adding indices may inprove the speed of the data access but they add a cost to the data modification. therefore it is important to understand if the index is used or not.

### create an index

1. you put them on table column or columns

2. Why not put an index on all columns? Because it will slow down the insert, update, delete operations.

3. Postgres supports indexes with up to 32 columns

4. Two main basic index types:
    Index Create an index on only values of a column or columns
    Unique Index Create an index on only values of a column or columns, but only allow unique values

NOTE: It is important to note that, when creating multi-column indexes, you should always place the most selective columns first. Postgresql will consider a multi-column index from the first column onward, so if first columns are the most selective, the index access method will be the cheapest.
    - Order of the field
    - How you are using in WHERE clause etc

### Create Unique Index
- Primiry key and indexes
1. Normally the primary key of a table are kept with a unique index

2. If you define a unique index for two or more columns, the combined values in thses columns cannot be duplicated in multiple rows.

### list all indexes
pg_indexes

all indexes in a table
```
SELECT * FROM pg_indexes WHERE tablename = 'table_name' and schemaname = 'public';
```

### Size of the table index
```
SELECT pg_size_pretty(pg_relation_size('table_name'));
```

### List counts for all indexes
- Adding indices may improve the speed of the data access but they add a cost to the data modification. Therefore it is important to understand if the index is used or not.
- pg_stat_all_indexes table which containers all the counts and lot more information over here
- when indexes are used, so the idx_scan, idx_tup_read, idx_tup_fetch columns will be greater than 0
1. all stats
```
SELECT * FROM pg_stat_all_indexes;
relid -- relative id -- table id
indexrelid -- index id
schemaname -- schema name
relname -- table name
indexrelname -- index name
idx_scan -- number of index scans initiated on this index
idx_tup_read -- number of index entries returned by scans on this index
idx_tup_fetch -- number of index entries fetched from disk by scans on this index
```
2. for a schema

```
SELECT * FROM pg_stat_all_indexes WHERE schemaname = 'public' ORDER BY relname, indexrelname;
```

### Drop Index

```
DROP INDEX [CONCURRENTLY] index_name [CASCADE | RESTRICT];
```
CONCURRENTLY: This option allows you to drop an index without blocking other operations on the table. This is useful when you have a large table and you want to drop an index without blocking other operations on the table.

### SQL Statement execution process

1. SQL is a declarative language, which means that you are telling the database what you want to do, but not how to do it. The database engine will decide how to do it.

2. You trust your database to run the best query plan for your query.

3. As a DBA, its your job to provide as much information to database engine to take best route to get your result set

Four stages of SQL execution
1. Parsing
    Handles the textual form of the statement (the sql text) and verifies whether is is correct or not
    disassemble info into tables, columns, clauses, etc
2. Rewriting
    Applying any syntactic rules to rewrite the original sql statement
3. Optimizer
    Finding the every fastest path to the data that the statement needs
4. Execution
    Responsible for effectively going to the storage and retrieving (or inserting) the data gets physically access to the data

### Optimizer query
- what to use to access Data
- As quickly as possible

1. The cost is everything
    Postgresql Operation = cost
       4 Operation = 4 X cost = total amount
    Lowest cost win
2. Thread

3. Nodes
    can split into nodes for each operation

4. Node types

### Optimizer Nodes Types

1. Nodes are available for 
    - Every operation and
    - Every access methods
2. nodes are stackable
    - one node can be stacked on top of another node
    - the top node is the one that is executed first
    - the bottom node is the one that is executed last
    parent Node (cost=0.00..0.10)
    child Node 1
    child Node 2
    the output of a node can be used as the input to another node
3. Types of Nodes
    - Sequential Scan
    - Index Scan, Index Only Scan, Bitmap Index Scan
    - Nested Loop, Merge Join, Hash Join
    - The gather and merge parallel nodes
```
   Select * From pg_am;
```

### Sequential Scan
Seq Scan

1. Default when no other valueable alternative

2. Read from the beginning to the end of the dataset

3. Filtering clause is not very limiting, so that the end result will be to get almost the whole table contents
    - sequential read-all operation faster
    - very expensive
    - not good for large tables
    - not good for tables with a lot of data
    - avoid at most of the time
eg: 
```
SELECT * FROM table_name where table_name.id IS NOT NULL;
```

### Index Nodes

1. An index is used to access the dataset

2. Data file and index files are seperated but the are nearby

3. Index Nodes scan type

    Index scan             index --> seeking the tuple(one trip) --> then read again the data(two trip)
```
    EXPLAIN SELECT * FROM orders WHERE order_id = 1;
```
    Index only scan        requested index columns only --> directly get data from index file
```
    EXPLAIN SELECT order_id FROM orders WHERE order_id = 1;
```
    Bitmap index scan      Builds a memmory bitmap of where tuples that satisfy the statement clauses

### Nested Loop
- Nested Loop is the default join method in Postgres
- Nested Loop meaning is that the inner table is looped over for each row in the outer table

### Merge Join
- Merge Join is a join method that is used when the join condition is an equality condition
- Merge join meaning is that the two tables are sorted on the join columns and then the two tables are merged together

### Join Nodes
- Join Nodes are used to join two or more tables together
1. used when joining the tables

2. Joins are performed on two tables at a time, if more tables are joined together, the output of one join is treated as input to a subsequent join

3. When joining a large number of tables, the genetic query optimizer settings may affect what cobinations of join are considered

4. Type

    Hash Join
    ---------
    - Inner table Build a hash table from the inner table, keyed by the join key
    - Outer table Scan the outer table, checking if a corresponding value is present
 
Show work_mem

    -- Merge Join
    Joins two children already sorted by their shared join key. this only needs to scan each relation once, but both inputs need to be sorted by the join key first (or scanned in a way that produces already-sorted output, like an index scan matching the required sort order)

    -- Nested Loop
    For each row in the outer table, iterate through all the rows in the inner table and see if they match the join condition. If the inner relation can be scanned with an index, that can improve the performance of a nested loop join. this is generally an inefficient way to process joins but is always available and sometimes may be the only option

### B-tree index
1. B-tree index is the default index type in Postgres
2. self-balancing tree data structure
   - Select, insert, delete and sequential access in logarithmic time
3. Can be used for most operators and columns type
4. Supports the Unique condition and 
5. Normally used to build the mrimary key indexes
6. Uses when culuns involves follwing operators
    <=, <, =, >=, >, and BETWEEN, IN, IS NULL, and IS NOT NULL
7. use when pattern matching
    evem for LIKE based queries

Note: one drawback: copy the whole column(s)' values into the thee structure, which can be a problem for large columns
### When shuold use B-tree index
Best used for comparisons with <=, <, =, >=, >, and BETWEEN, IN, IS NULL, and IS NOT NULL

### When should use HASH index
only can you use the = operator

### When should use GIN index
Best used for full-text search and array operators (come from cognito note sure)
Best use when multiple values are stored in a single field

1. generalized inverted index
2. point to multiple tuples
3. Used with array type data
4. Use in full text search
5. Useful when we have multiple value stored in a signle column

### When should use GIST index 
Best used for geometric data types
useful in indexing geometric data and full-text search

### when to use index
1. index foreign key
2. index primary key and unique key (automatically create by the database)
3. index columns that are used in the where clause
4. index columns that are used in the order by clause

### when not to use index
1. Don't add an index just to add an index
2. Don't use indexes on small tables
3. Don't use on tables that are updated frequently
4. Don't use on columns that can contain null values
5. Don't use on columns that have large values

### example for WORLD database
```
select count(*) from city -- 4079

explain analyze select "name", district, countrycode from city where countrycode in ('TUN', 'BE', 'NL')
-- without index
-- plan 0.052
-- execution 0.466

create index idx_countrycode on city (countrycode);
-- index
-- plan 0.173
-- execution 0.066
```

```
-- partial index
create index idx_partial_contrycode on city (countrycode)
where contrycode in ('TUN', 'BE', 'NL')
```

### HASH index
1. For equality operator ( = )
2. not for range nor disequality operator
3. Larger than btree indexes

```
create index idx_countrycode on city (countrycode) using hash;
```

### BRIN index
1. Block Range index
2. data block -> min to max value
3. smaller index
4. less costly to maintain than btree index
5. can be used on a large table vs btree index
6. used linear sort order e.g. customers -> order_date

### the Explain statement

1. It will show query execution plan

2. Shows the lowest COST among evaluted plans

3. Will not execute the statement you enter, just show query only

4. show you the execution nodes that the executor will use to provide you with the dataset

-- lets run a query and see its plan
- execution nodes
    - can type | execution nodes table

- every node
    cost
        startup cost        how much work postgresql has to do before it begins executing the node
        final cost          how much effort postgresql has to do to provide the last bit of the dataset

    rows                    how many tuples the node is expected to provide for final dataset
    width                   how many bits every tuples will occupy, as an average

### EXPLAIN output options

-- TEXT, XML, JSON, YAML

EXPLAIN (FORMAT TEXT) SELECT * FROM table_name;

### Using explain analyze
1. It print out the best plan to execute the query and also the actual execution time
2. Is runs the query
3. Also report back some statistical information

```
actual time = 0.915..0.916 rows=0 loops=1

0.916 - 0.915 = 0.001 ms (is the execution time of query)
```

exucution time : 0.941 ms
planning time : 0.146 ms
0.941 + 0.146 =  ms actual time query

### understanding query cost model
```
Create table t_big (id serial, name text)

insert into t_big (name)
select 'Adam' from generate_series(1, 2000000)

insert into t_big (name)
select 'Linda' from generate_series(1, 2000000)


EXPLAIN SELECT * FROM t_big WHERE id = 12345

-- TO 1
"Gather  (cost=1000.00..44186.13 rows=13730 width=36)"
"  Workers Planned: 1"
"  ->  Parallel Seq Scan on t_big  (cost=0.00..41813.13 rows=8076 width=36)"
"        Filter: (id = 12345)"
-- TO 2
"Gather  (cost=1000.00..38297.05 rows=13730 width=36)"
"  Workers Planned: 2"
"  ->  Parallel Seq Scan on t_big  (cost=0.00..35924.05 rows=5721 width=36)"
"        Filter: (id = 12345)"
--- TO 0
"Seq Scan on t_big  (cost=0.00..55946.93 rows=13730 width=36)"
"  Filter: (id = 12345)"

EXPLAIN ANALYZE SELECT * FROM t_big WHERE id = 12345

-- SHOW max_parallel_workers_per_gather == 2
SET max_parallel_workers_per_gather TO 0
SET max_parallel_workers_per_gather TO 2

SELECT pg_relation_size('t_big')
-- 177127424 bytes
SELECT pg_size_pretty(pg_relation_size('t_big'))

SELECT pg_relation_size('t_big') / 8192.0   
-- 21622.000000000000 blocks
-- 8192.0 blocks
SHOW seq_page_cost
-- 1

SHOW cpu_tuple_cost
-- 0.01
SHOW cpu_operator_cost
-- 0.0025

-- cost formula

pg_relation_size * seq_page_cost + total_number_of_table_records * cpu_tuple_cost 
+ total_numberOf_table_records * cpu_operator_cost

SELECT 21622 * 1 + 4000000 * 0.01 + 4000000 * 0.0025
"Seq Scan on t_big  (cost=0.00..55946.93 rows=13730 width=36)"
```

### Size index
```
SELECT pg_size_pretty(pg_indexes_size('t_big'))
-- 0 bytes to 86 bytes
SELECT pg_size_pretty(pg_relation_size('t_big'))
SELECT pg_size_pretty(pg_total_relation_size('t_big'))
-- 169 MB after create an index 255 MB
explain analyze select * from t_big where id = 123456
-- 43455.43
-- 8.45
create index idx_t_big_id on t_big(id);

SHOW max_parallel_maintenance_workers 
-- 2
```

### Indexes for sorted output
```
explain Select * from t_big 
order by id 
limit 20;
"Limit  (cost=0.43..1.06 rows=20 width=9)"
"  ->  Index Scan using idx_t_big_id on t_big  (cost=0.43..125505.43 rows=4000000 width=9)"
there is no need for sort operation because the index is already sorted
```

### Using multiple indexis on a single query
```
EXPLAIN SELECT * FROM t_big
WHERE
	id = 20
	or id = 40
	
EXPLAIN SELECT * FROM t_big
WHERE id in (20, 40)

blocks - pages of table
```

### Execution plans depends on input values
```
 create index inx_t_big_name on t_big (name)
 
 explain select * from t_big where name = 'Adam'
 limit 10
"Limit  (cost=0.00..0.35 rows=10 width=9)"
"  ->  Seq Scan on t_big  (cost=0.00..71622.00 rows=2018267 width=9)"
"        Filter: (name = 'Adam'::text)"
not using index in name

explain select * from t_big
where name = 'Adam' or name = 'Linda'
not using index in name because the filter Adam and Linda here make up the entire data sets
there are no other values and why the system knows about it, because the postgres knows by checking the system statistics

explain select * from t_big
where 
	name = 'Adam2'
	or name = 'Linda2'
this will using the index because the filter value is difference	
so Postgres will using the index when they make sense, if the number of rows is smaller, Postgresql just will use index scan
other hand it will use seq scan
```

### Using organized vs random data
```
explain (analyze true, buffers true, timing true)
select * from t_big where id < 10000
-- buffers true theo how many of 85 blocks were touched by the query

order by random()

CREATE TABLE t_big_random as select * from t_big order by random();
-- create the same column of table t_big in t_big_random and insert  by order random
select * from t_big_random
create index idx_t_big_random_id on t_big_random(id)

VACUUM analyze t_big_random
-- So this query is cleaning up the t_big_random table and updating the statistics for it to optimize future queries.
explain (analyze true, buffers true, timing true)
select * from t_big_random where id < 10000

"Bitmap Heap Scan on t_big_random  (cost=196.43..18138.03 rows=10322 width=9) (actual time=2.800..42.262 rows=9999 loops=1)"
"  Recheck Cond: (id < 10000)"
"  Heap Blocks: exact=8039"
"  Buffers: shared hit=838 read=7231"
"  ->  Bitmap Index Scan on idx_t_big_random_id  (cost=0.00..193.84 rows=10322 width=0) (actual time=1.655..1.656 rows=9999 loops=1)"
"        Index Cond: (id < 10000)"
"        Buffers: shared hit=3 read=27"
"Planning Time: 0.172 ms"
"Execution Time: 42.717 ms"

when we run vacuum analyze the buffers hit is 8069 blocks more than before we run vacuum analyze
the reason the executiion time is less than before we execute the vacuum query is the data was served from memory
and not from the disk

the vacuum analyze will analyze base on the table pg_stats to analyze how to get the data from disk or memory
and store the data from memory

select 
	tablename,
	attname,
	correlation -- is 1 which means that the next value statement depends on the pervious one and
-- and we have seen it because the id has been added plus one (1, 1+1 =2, 2+1 = 3)
from pg_stats
where 
	tablename in ('t_big', 't_big_random')
	order by 1, 2
-- summary
-- the t_big is organized by insert statement when id  1,2,3 ... sequen 
-- and the name is organized 2 milions adam, 2 milion linda
-- the query will hit the 85 block when execute (correlation is 1)
-- the t_big_random isn't organized by insert statement id is mixed
-- so the query  will hit 8000 block to get the data
```
### Try using index only scan
```
explain analyze
select * from t_big where id = 123456


explain analyze
select id from t_big where id = 123456
-- Index only scan will be faster most of case because it no need to go to the original table in most cases
-- instead of index table
```

### Partial Index
```
-- Partial index: to improve the performance of the query while reducing the index size

Create index index_name on table_name
where conditions

select pg_size_pretty(pg_indexes_size('t_big'))

-- clear the index that work sometimes
drop index idx_t_big_name

create index idx_p_t_big_name on t_big(name)
where name not in ('Adam', 'Linda')

-- the idea is exclude majority has to be excluded from the index and a small more effient  

select * from customers

update customers
set is_active = 'N'
where customer_id in ('ALFKI', 'ANATR')


explain analyze select * from customers
where is_active = 'N'
-- 0.198 + 0.041

create index idx_p_customers_inactive on customers(is_active)
where is_active = 'N'
-- 48kb
select pg_size_pretty(pg_indexes_size('customers'))

explain analyze select * from customers
where is_active = 'N'
-- 0.191 + 0.044
-- the time is same because 
-- we have few record in table
```

### Expression Index
```
1. An index created based on an expression:
   UPPER(column_name)
   COS(column_name)
2. Postgresql will consider using that index when the expression that defines the index appears in the 
    - WHERE clause
    - ORDER BY clause
3. Very expressive indexes
   Postgreql has to evaluete the expression for EACH ROW when it is inserted or update and use the result for indexing

create index index_name on table_name(expression)

create table t_dates as
select d, repeat(md5(d::text), 10) as padding
from generate_series(timestamp '1800-01-01', timestamp '210-01-01', interval 1 day) s(d)

vacuum analyze t_dates

explain analyze select * from t_dates where d between '2001-01-01' and '2001-01-31';
-- without any index
-- 97.634

### adding data while indexing
```
create index concurrently
it's in the process because thhe create index command will lock up the table using a shared lock
to ensure that no changes happen to that table while the index is being created
while this is clearly not a problem for a small table, it will cause really an issue for a large table
so the solution is to use the concurrently keyword
Note: Postgresql doesn't guarentee success when using concurrently, 

this will select the indesvalid for table
Select oid, relname, reltuples, i.indisunique, i.indisclustered, i.indisvalid, pg_catalog.pg_get_indexdef(i.indexrelid, 0, true)
from pg_class c join pg_index i on c.oid = i.indrelid
where c.relname = 't_dates'
```

### Invalidating an index
```
lets view all indexes for a table
Select oid, relname, reltuples, i.indisunique, i.indisclustered, i.indisvalid, pg_catalog.pg_get_indexdef(i.indexrelid, 0, true)
from pg_class c join pg_index i on c.oid = i.indrelid
where c.relname = 'orders'

create index ship_country

-- lets disallow the query optimizer from using the index
update pg_index
set indisvalid = false
where indexrelid = (select oid from pg_class where relkind= 'i' relname = 'idx_orders_ship_country')
```

### Rebuilding an index
```
REINDEX (verbose) index concurrently idx_orders_customer_id_order_id;
REINDEX (verbose) table table_name;
shema , database 

Begin
REINDEX (verbose) index idx_orders_customer_id_order_id;
REINDEX (verbose) table table_name;
End;
only working for index and table
```
### atomicity, consistency, isolation, durability

### Atomicity
- all queries in a transaction must succeed
- if one fails, the whole transaction fails and the database is rolled back to the state before the transaction started

usecase 
- transfer money from one account to another
- if the first query succeeds and the second fails, the first query should be rolled back
- but the database is crashed before the second query is executed, the first query is committed
- after we restart the machine the first account is debited but the second is not credited
- this is called a partial commit
- this is really bad as we just lost data, and the information is inconsistent
- an atomic transaction is a transaction that will rollback all queries if one or more fail
- the database should clean this up after restart
```
BEGIN;
SELECT quantity FROM products WHERE name = 'iPhone' FOR UPDATE; -- Lock the row for update
UPDATE products SET quantity = quantity - 1 WHERE name = 'iPhone' AND quantity > 0;
COMMIT;
```
- the above transaction is atomic
- if the first query fails, the second query will not be executed

### isolation
from cogito
- isolation is the property that ensures that concurrent transactions are isolated from each other
- this means that the execution of concurrent transactions results in a system state that would be obtained if transactions were executed serially, i.e., one after the other
- this is achieved by locking the rows that are being updated
- this is called row level locking
- this is the default locking mechanism in MySQL
- this is also called pessimistic locking
from video
- Can my inflight transaction see changes made by other transactions?
- Read phenomena, the phonemena is these are usually undesirable
- Isolation levels
### Isolation - Read phenomena
- dirty read - a transaction reads a row that has been modified by another transaction but not yet committed(rollbacked, commited, or crashed)

- Non-repeatable read - a transaction reads the same row twice and gets different results because the row has been modified by another transaction (like when you trying to sum up the total amount of money in your bank account, but the amount is changing because of other transactions)

    + postgresql has a feature called snapshot isolation, which prevents this from happening
    it will create new version of the row for each transaction, and the transaction will only see the version of the row that was created when the transaction started
    this is called MVCC
    + mysql, oracle, and sql server do not have this feature, it changes the record but keeps the old record in another called the undo log


- Phantom read - when you select a range of rows, and another transaction inserts a row into that range, you will get a different result set when you run the same query again

- Lost updates - when you update a row, and another transaction updates the same row, the second update will be lost   

### Isolation - Isolation Levels for inflight transactions
1. Read uncommitted(uncommitted is no isolation) 
- NO isolation, any changes from the outside is visible to the transaction, commit or not (but this will be fast because it does not lock the row, I don't really need to maintain anything in same cases)
```
BEGIN ISOLATION LEVEL READ UNCOMMITTED;
SELECT * FROM products WHERE name = 'iPhone';
UPDATE products SET quantity = quantity - 1 WHERE name = 'iPhone';
COMMIT;
```
2. Read committed
- Is the one of most popular isolation levels
- Each query in a transaction only sees committed changes from other transactions
- This is the default isolation level in MySQL

3. Repeatable read
- The transaction will make sure that when a query read a row, that row will remain unchanges the transaction while it is running
4. Snapshot
Each query in a transaction only sees changes that have been committed up to the start of the transaction. it's like a snapshot version of the database at that moment
5. Serializable
Transatoins are run as if they serialized one after the other

### Isolation levels vs read phenomena

[alt](Isolation%20levels%20vs%20read%20phenomena.png)

### Serializable vs Snapshot
- Serializable is more strict than snapshot
- Serializable will lock the rows that are being read, so that other transactions cannot modify them
- Snapshot will not lock the rows, but it will create a snapshot of the database at the start of the transaction, and the transaction will only see the snapshot

### Database implementation of isolation levels
- Each DBMD implements isolation levels differently
- Perssimistic - Row level locks, table locks, page locks to avoid lost updates (very expensive, especially when you have a lot of concurrent transactions, row level locks is the most expensive)
- Optimistic - No locks, just track if things changed and fail the transaction if so
- Repeatable read "locks" the rows it read but it could be expensive if you read a lot of rows, postgres implements RR as snapshot. That is why you don't get phantom reads with postgres in repreatable read
0 Serializable are usually implemented with optimistic concurrency control, you can implement it pessimistically with `select for update`
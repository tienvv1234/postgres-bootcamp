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
    + mysql, oracle, and sql server do not have this feature, 


- Phantom read - when you select a range of rows, and another transaction inserts a row into that range, you will get a different result set when you run the same query again

- Lost updates - when you update a row, and another transaction updates the same row, the second update will be lost   

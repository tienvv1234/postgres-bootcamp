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
from --> Select --> order by
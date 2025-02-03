### Durability
- Changes made by committed transactions must be persisted in a durable(bên chặt) non-volatile(không bị mất) storage.
- Durability techniques:
    - WAL - Write-ahead logging (when a transaction commits, its log records must be written to stable storage before the transaction commits)
    - Asynchronous Snapshots (periodically write the entire database to stable storage)
    - AOF - Append-only files (write every command to a file)(similar to WAL)
    - Checkpointing

- Durability - WAL
    - Writing a lot of data to disk is expensive(indexes, data files, columns, rows, etc.)
    - That is why DBMSs persist a compressed version of the changes known as WAL(write-ahead-log segments)
- Durability - OS cache
    - A write request in OS usually goes to the OS cache
    - When the writes go the OS cache, an OS crash, machine restart could lead to loss of data
    - Fsync OS command forces writes to always go to dish
    - Fsync is expensive, and slows down the commits

```
docker run --name pgacid -d -e POSTGRES_PASSWORD=123456 postgres:13
docker exec -it pgacid psql -U postgres
```

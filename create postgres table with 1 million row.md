### generate 1 million rows
```sql
insert into temp(t) select random()*100 from generate_series(0, 1000000);
```

### create table employee with 11 million rows has name, age, salary with id is primary key and serial
```sql
create table employee(
    id serial primary key,
    name varchar(100),
    age int,
    salary int
);
insert into employee(name, age, salary) select md5(random()::text), random()*100, random()*10000 from generate_series(0, 11000000);

```

### inline query


### --key vs non-key column indexes in postgres
```sql
create table temp(
    id serial primary key,
    t int
);
insert into temp(t) select random()*100 from generate_series(0, 1000000);
create index idx_temp_t on temp(t);
select * from temp where t = 50;
select * from temp where id = 50;
drop index idx_temp_t;
create index idx_temp_id on temp(id) include(t);
```
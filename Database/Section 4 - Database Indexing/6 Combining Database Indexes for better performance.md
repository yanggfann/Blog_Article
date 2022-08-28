## Prerequisites

- Start the docker instance

```
docker run --name pg -e POSTGRES_PASSWORD=postgres -d postgres
```

```
docker start pg
```

- Run postgres command shell

```
docker exec -it pg psql -U postgres
```

The command should switch to `postgres=#`

- Create table and insert data

```
drop table if exists test;

create table test (
a int,
b int,
c int
);

insert into test (a, b, c)
select
random()*100,
random()*100,
random()*100
from generate_series(0, 12000000);
```

## Combining Database Indexes for better performance

### Create index on A and B separately

```
postgres=# create index on test(a);
CREATE INDEX
postgres=# create index on test(b);
CREATE INDEX
```

```
postgres=# \d test;
                Table "public.test"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 a      | integer |           |          |
 b      | integer |           |          |
 c      | integer |           |          |
Indexes:
    "test_a_idx" btree (a)
    "test_b_idx" btree (b)
```

```
postgres=# explain analyze select c from test where a = 70;
                                                            QUERY PLAN

---------------------------------------------------------------------------------------------------
-------------------------------
 Bitmap Heap Scan on test  (cost=1465.23..67970.24 rows=131200 width=4) (actual time=74.330..859.64
2 rows=120400 loops=1)
   Recheck Cond: (a = 70)
   Heap Blocks: exact=54813
   ->  Bitmap Index Scan on test_a_idx  (cost=0.00..1432.43 rows=131200 width=0) (actual time=65.72
4..65.725 rows=120400 loops=1)
         Index Cond: (a = 70)
 Planning Time: 3.121 ms
 Execution Time: 864.864 ms
(7 rows)
```

- Obviously it decides to use the index `test_a_idx`
- It created a bitmap `Bitmap Index Scan` in order to query this index because there are a lot of rows `rows=120400` and it has to jump back to the heap to pull `c`
- `Execution Time: 864.864 ms`

```
postgres=# explain analyze select c from test where a = 70 limit 2;
                                                  QUERY PLAN

---------------------------------------------------------------------------------------------------
-----------
 Limit  (cost=0.00..3.28 rows=2 width=4) (actual time=0.391..0.393 rows=2 loops=1)
   ->  Seq Scan on test  (cost=0.00..214865.01 rows=131200 width=4) (actual time=0.385..0.386 rows=
2 loops=1)
         Filter: (a = 70)
         Rows Removed by Filter: 315
 Planning Time: 0.976 ms
 Execution Time: 0.570 ms
(6 rows)
```

- `Seq Scan` when `limit 2` since postgres doesn't need to take the overhead to build a bitmap.



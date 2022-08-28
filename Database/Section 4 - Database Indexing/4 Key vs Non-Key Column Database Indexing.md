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
create table students (
id serial primary key,
g int,
firstname text,
lastname text,
middlename text,
address text,
bio text,
dob date,
id1 int,
id2 int,
id3 int,
id4 int,
id5 int,
id6 int,
id7 int,
id8 int,
id9 int
);


insert into students (g,
firstname,
lastname,
middlename,
address ,
bio,
dob,
id1 ,
id2,
id3,
id4,
id5,
id6,
id7,
id8,
id9)
select
random()*100,
substring(md5(random()::text ),0,floor(random()*31)::int),
substring(md5(random()::text ),0,floor(random()*31)::int),
substring(md5(random()::text ),0,floor(random()*31)::int),
substring(md5(random()::text ),0,floor(random()*31)::int),
substring(md5(random()::text ),0,floor(random()*31)::int),
now(),
random()*100000,
random()*100000,
random()*100000,
random()*100000,
random()*100000,
random()*100000,
random()*100000,
random()*100000,
random()*100000
from generate_series(0, 50000000);
```

## Key vs Non-Key column Database Indexing

The `students` table has 50 million rows.

```
postgres=# \d students;
                              Table "public.students"
   Column   |  Type   | Collation | Nullable |               Default

------------+---------+-----------+----------+-------------------------------
-------
 id         | integer |           | not null | nextval('students_id_seq'::reg
class)
 g          | integer |           |          |
 firstname  | text    |           |          |
 lastname   | text    |           |          |
 middlename | text    |           |          |
 address    | text    |           |          |
 bio        | text    |           |          |
 dob        | date    |           |          |
 id1        | integer |           |          |
 id2        | integer |           |          |
 id3        | integer |           |          |
 id4        | integer |           |          |
 id5        | integer |           |          |
 id6        | integer |           |          |
 id7        | integer |           |          |
 id8        | integer |           |          |
 id9        | integer |           |          |
Indexes:
    "students_pkey" PRIMARY KEY, btree (id)
```

### Case 1

```
postgres=# explain analyze select id, g from students where g > 80 and g < 95 order by g desc;
                                                                 QUERY PLAN

-----------------------------------------------------------------------------
---------------------------------------------------------------
 Gather Merge  (cost=1162340.39..1178098.02 rows=135056 width=8) (actual time
=11443.014..13018.307 rows=6998932 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   ->  Sort  (cost=1161340.36..1161509.18 rows=67528 width=8) (actual time=11
332.225..11487.495 rows=2332977 loops=3)
         Sort Key: g DESC
         Sort Method: external merge  Disk: 41872kB
         Worker 0:  Sort Method: external merge  Disk: 40880kB
         Worker 1:  Sort Method: external merge  Disk: 40648kB
         ->  Parallel Seq Scan on students  (cost=0.00..1155923.54 rows=67528
 width=8) (actual time=76.561..10828.788 rows=2332977 loops=3)
               Filter: ((g > 80) AND (g < 95))
               Rows Removed by Filter: 14333690
 Planning Time: 4.710 ms
 JIT:
   Functions: 12
   Options: Inlining true, Optimization true, Expressions true, Deforming tru
e
   Timing: Generation 6.806 ms, Inlining 113.925 ms, Optimization 75.596 ms,
Emission 38.385 ms, Total 234.711 ms
 Execution Time: 13268.825 ms
(17 rows)
```

- It takes 13 seconds
- It doesn't have any index，so it had to do the parallel sequential scan
- Rows Removed by Filter: 14333690. Which means that we pulled pages and we had to discard
- `Parallel Seq Scan`

### Case 2 - create a key column index

```
postgres=# create index g_idx on students(g);
CREATE INDEX
```

```
postgres=# explain analyze select id, g from students where g > 80 and g < 95 order by g desc;
                                                                 QUERY PLAN

-----------------------------------------------------------------------------
----------------------------------------------------------------
 Gather Merge  (cost=1653753.15..2324430.77 rows=5748264 width=8) (actual tim
e=4494.471..6113.521 rows=6998932 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   ->  Sort  (cost=1652753.13..1659938.46 rows=2874132 width=8) (actual time=
4445.952..4599.107 rows=2332977 loops=3)
         Sort Key: g DESC
         Sort Method: external merge  Disk: 41200kB
         Worker 0:  Sort Method: external merge  Disk: 41400kB
         Worker 1:  Sort Method: external merge  Disk: 40792kB
         ->  Parallel Seq Scan on students  (cost=0.00..1265839.00 rows=28741
32 width=8) (actual time=74.927..3969.317 rows=2332977 loops=3)
               Filter: ((g > 80) AND (g < 95))
               Rows Removed by Filter: 14333690
 Planning Time: 7.568 ms
 JIT:
   Functions: 12
   Options: Inlining true, Optimization true, Expressions true, Deforming tru
e
   Timing: Generation 48.232 ms, Inlining 103.578 ms, Optimization 69.905 ms,
 Emission 47.228 ms, Total 268.943 ms
 Execution Time: 6401.464 ms
(17 rows)
```

- The id is actually in two places. The id is on another index because the primary key and it's also in the heap as another field. So this has to go to the table again to pull the ids.
- It's still slow.
- `Parallel Seq Scan`

### Case 3

```
postgres=# explain (analyze, buffers) select id, g from students where g > 80 and g < 95 order by g desc limit 1000;
                                                                   QUERY PLAN

-----------------------------------------------------------------------------
-------------------------------------------------------------------
 Limit  (cost=0.56..2021.19 rows=1000 width=8) (actual time=1.337..18.134 row
s=1000 loops=1)
   Buffers: shared hit=801 read=3
   ->  Index Scan Backward using g_idx on students  (cost=0.56..13938080.17 r
ows=6897917 width=8) (actual time=1.040..17.696 rows=1000 loops=1)
         Index Cond: ((g > 80) AND (g < 95))
         Buffers: shared hit=801 read=3
 Planning:
   Buffers: shared hit=20
 Planning Time: 5.746 ms
 Execution Time: 18.972 ms
(9 rows)
```

- `Buffers: shared hit=801` That means all the things that we queried are cached. That is why is so fast.
- `Index Scan Backward using g_idx`. Because `order by g desc` , so which is at the end.


### Case 4 - include a non-key column index

```
create index g_idx on students(g) include (id);
```
- That's a very common execution path that we're doing. That's where you as a back engineer, you really need to think like what columns are you selecting. And based on those columns, you create your indexes effectively.
- `id` is a non-key column, `g` is a key column

```
postgres=# explain (analyze, buffers) select id, g from students where g > 10  and g < 20 order by g desc;
                                                                    QUERY PLA
N
-----------------------------------------------------------------------------
---------------------------------------------------------------------
 Index Only Scan Backward using g_idx on students  (cost=0.56..137695.90 rows
=4444167 width=8) (actual time=32.134..575.372 rows=4500110 loops=1)
   Index Cond: ((g > 10) AND (g < 20))
   Heap Fetches: 0
   Buffers: shared hit=272 read=12298
 Planning:
   Buffers: shared hit=32 read=8 dirtied=3
 Planning Time: 11.191 ms
 JIT:
   Functions: 2
   Options: Inlining false, Optimization false, Expressions true, Deforming t
rue
   Timing: Generation 20.673 ms, Inlining 0.000 ms, Optimization 10.293 ms, E
mission 20.143 ms, Total 51.108 ms
 Execution Time: 765.781 ms
```

- In this `Index Only Scan`, we didn't have to go to the heap at all `Heap Fetches: 0`. 

## Other statement

```
vacuum (analyze, verbose, full);
```

- This will help you update all the pages, the visibility map, anything
- `vacuum` - garbage-collect and optionally analyze a database
- `analyze` - Updates statistics used by the planner to determine the most efficient way to execute a query.
- `verbose` - Prints a detailed vacuum activity report for each table.
- `full` - Selects “full” vacuum, which can reclaim more space, but takes much longer and exclusively locks the table. This method also requires extra disk space, since it writes a new copy of the table and doesn't release the old copy until the operation is complete. Usually this should only be used when a significant amount of space needs to be reclaimed from within the table.




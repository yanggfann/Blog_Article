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
drop table if exists grades;

create table grades (
id serial,
g int,
name text
);

insert into grades (g,
name  )
select
random()*100,
substring(md5(random()::text ),0,floor(random()*31)::int)
from generate_series(0, 2000000);
```

## Index Scan vs Index Only Scan

### Case 1

```
postgres=# explain analyze select name from grades where id = 7;
                                                        QUERY PLAN

-----------------------------------------------------------------------------
---------------------------------------------
 Gather  (cost=1000.00..23703.65 rows=8084 width=32) (actual time=5.295..291.
916 rows=1 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   ->  Parallel Seq Scan on grades  (cost=0.00..21895.25 rows=3368 width=32)
(actual time=42.611..134.403 rows=0 loops=3)
         Filter: (id = 7)
         Rows Removed by Filter: 666667
 Planning Time: 2.519 ms
 Execution Time: 293.220 ms
(8 rows)
```

- There are no indexing on id, so it was decided to do the Parallel Seq Scan (Table Scan)
- Table scan just literally one by one search, every single row pulls it.
- It took `Execution Time: 293.220 ms` to do that and it returns `rows=1`

### Case 2 - Index Scan

```
postgres=# create index id_idx on grades(id);
CREATE INDEX
```

```
postgres=# explain analyze select name from grades where id = 7;
                                                   QUERY PLAN
----------------------------------------------------------------------------------------------------------------
 Index Scan using id_idx on grades  (cost=0.43..8.45 rows=1 width=15) (actual time=0.105..0.109 rows=1 loops=1)
   Index Cond: (id = 7)
 Planning Time: 0.272 ms
 Execution Time: 0.241 ms
(4 rows)
```

- It went from `Execution Time: 293.220 ms` to `Execution Time: 0.241 ms`
- `Index Scan`. We use the index was called id_idx to search for id = 7
- The name of the student is not in the index, the only value in the index is the id.
- We found the id. However, we had to go back to the table to actually fetch the value name.

### Case 3 - Index only scan

```
postgres=# explain analyze select id from grades where id = 7;
                                                     QUERY PLAN
--------------------------------------------------------------------------------------------------------------------
 Index Only Scan using id_idx on grades  (cost=0.43..4.45 rows=1 width=4) (actual time=0.076..0.080 rows=1 loops=1)
   Index Cond: (id = 7)
   Heap Fetches: 0
 Planning Time: 0.256 ms
 Execution Time: 0.149 ms
(5 rows)
```

- `Index Only Scan`. It did not have to go to the table to fetch any other fields.

### Case 4 - Index only scan

```
postgres=# drop index id_idx;
DROP INDEX
```

```
postgres=# create index id_idx on grades(id) include (name);
CREATE INDEX
```

```
postgres=# \d grades
                            Table "public.grades"
 Column |  Type   | Collation | Nullable |              Default
--------+---------+-----------+----------+------------------------------------
 id     | integer |           | not null | nextval('grades_id_seq'::regclass)
 g      | integer |           |          |
 name   | text    |           |          |
Indexes:
    "id_idx" btree (id) INCLUDE (name)
```

- I know that most of my queries when I query for the id are asking for a name. I barely ask for the grade. 
- I'm going to actually include the name field in the index. 
- The `name` column is called non-key column for this scenario. This is what we're going to fetch from the index.
- The `id` column is called key column for this scenario. This is what we're going to search again.
- It's not going to use this for optimization or searching. It's just we can pull them from a very efficiently.

```
postgres=# explain analyze select name from grades where id = 7;
                                                     QUERY PLAN

---------------------------------------------------------------------------------------------------
------------------
 Index Only Scan using id_idx on grades  (cost=0.43..4.45 rows=1 width=15) (actual time=0.429..0.43
1 rows=1 loops=1)
   Index Cond: (id = 7)
   Heap Fetches: 0
 Planning Time: 6.914 ms
 Execution Time: 0.709 ms
(5 rows)
```

- I'm going to ask for the `name`, but it still is `Index Only Scan`

```
postgres=# explain analyze select g from grades where id = 7;
                                                  QUERY PLAN

---------------------------------------------------------------------------------------------------
------------
 Index Scan using id_idx on grades  (cost=0.43..8.45 rows=1 width=4) (actual time=0.132..0.133 rows
=1 loops=1)
   Index Cond: (id = 7)
 Planning Time: 4.430 ms
 Execution Time: 0.320 ms
(4 rows)
```

- If I'm going to ask for the `g`, it will be `Index Scan`

## Note

- Pay attention to the cost of these things, because as we work more with a large index and include non-key column, and then this can actually increase the size of index.
- If you increase the size of the index, that could be bad. The bigger index that the slower it gets to query. You had to fetch more and more pages to get into the things that you're looking for.

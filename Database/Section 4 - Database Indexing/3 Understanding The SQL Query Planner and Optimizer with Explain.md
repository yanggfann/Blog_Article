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
create table grades (
id serial primary key,
g int,
name text
);

insert into grades (g,
name  )
select
random()*100,
substring(md5(random()::text ),0,floor(random()*31)::int)
from generate_series(0, 2000000);

create index g_idx on grades(g);

vacuum (analyze, verbose, full);

explain analyze select id,g from grades where g > 80 and g < 95 order by g;
```

## explain

```
postgres=# \d grades;
                            Table "public.grades"
 Column |  Type   | Collation | Nullable |              Default
--------+---------+-----------+----------+------------------------------------
 id     | integer |           | not null | nextval('grades_id_seq'::regclass)
 g      | integer |           |          |
 name   | text    |           |          |
Indexes:
    "grades_pkey" PRIMARY KEY, btree (id)
    "g_idx" btree (g)
```

### Case1

```
postgres=# explain select * from grades;
                           QUERY PLAN
-----------------------------------------------------------------
 Seq Scan on grades  (cost=0.00..33470.01 rows=2000001 width=23)
(1 row)
```

- Seq Scan, Sequential Scan, it's equivalent to a full table scan
- cost has two numbers separated by two `..`
  - The first number means how many millisecond it took me to fetch the first page. If you see this number go up, that means you're doing a lot of stuff before fetching.
  - The second number is essentially the total amount of time that it **thinks** because remember, it didn't really execute the query. It estimates there's going to finish the whole thing in 33470.01 millisecond.
- `rows=2000001` is the another estimation. This is not an accurate number, but it gives you a quick number approximate based on its own statistic.
- `width=23`, The width of the row. This is the sum of all the bytes for all the columns.

### Case2

```
postgres=# explain select * from grades order by g;
                                  QUERY PLAN
-------------------------------------------------------------------------------
 Index Scan using g_idx on grades  (cost=0.43..90679.51 rows=2000001 width=23)
(1 row)
```

- Index Scan

### Case3

```
postgres=# explain select * from grades order by name;
                                     QUERY PLAN

--------------------------------------------------------------------------------
-----
 Gather Merge  (cost=121846.16..316304.33 rows=1666668 width=23)
   Workers Planned: 2
   ->  Sort  (cost=120846.13..122929.47 rows=833334 width=23)
         Sort Key: name
         ->  Parallel Seq Scan on grades  (cost=0.00..21803.34 rows=833334 width
=23)
(5 rows)
```

- The name is a column that is not indexed
- Read it from bottom going up

### Case4

```
postgres=# explain select id from grades;
                           QUERY PLAN
----------------------------------------------------------------
 Seq Scan on grades  (cost=0.00..33470.01 rows=2000001 width=4)
(1 row)
```

- `width=4 byte` is the result you're asking to return id, id is an integer and integer is 4 byte

### Case5

```
postgres=# explain select name from grades;
                           QUERY PLAN
-----------------------------------------------------------------
 Seq Scan on grades  (cost=0.00..33470.01 rows=2000001 width=15)
(1 row)
```

- `width=15` because the name is text, they took the average there
  - Be careful with this number, the larger number, the larger network you're going to take the higher the TCP packet
  - Don't select * when you don't use this thing. Only select what you need.

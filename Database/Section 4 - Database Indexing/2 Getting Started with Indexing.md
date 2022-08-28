## Prerequisites

employees table for the indexing lecture, paste these commands into the postgres.

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

- Paste these sql

  ```
  create table employees( id serial primary key, name text);
  ```
  
  ```
  create or replace function random_string(length integer) returns text as
  $$
  declare
  chars text[] := '{0,1,2,3,4,5,6,7,8,9,A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z,a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z}';
  result text := '';
  i integer := 0;
  length2 integer := (select trunc(random() * length + 1));
  begin
  if length2 < 0 then
  raise exception 'Given length cannot be less than 0';
  end if;
  for i in 1..length2 loop
  result := result || chars[1+random()*(array_length(chars, 1)-1)];
  end loop;
  return result;
  end;
  $$ language plpgsql;
  ```
  
  ```
  insert into employees(name)(select random_string(10) from generate_series(0, 1000000));
  ```

## Index

### What's the index?

An index is a data structure that you build and you assign on top of an existing table.

  ```
  postgres=# \d employees;
                              Table "public.employees"
   Column |  Type   | Collation | Nullable |                Default
  --------+---------+-----------+----------+---------------------------------------
   id     | integer |           | not null | nextval('employees_id_seq'::regclass)
   name   | text    |           |          |
  Indexes:
      "employees_pkey" PRIMARY KEY, btree (id)
  ```

Every primary key has an index by default, and it's a B-tree index.

### The performance of each queries in Postgres

#### Case1

  ```
  postgres=# explain analyze select id from employees where id = 2000;
                                                            QUERY PLAN
  
  --------------------------------------------------------------------------------------------------------------------------
  -----
   Index Only Scan using employees_pkey on employees  (cost=0.42..4.44 rows=1 width=4) (actual time=3.126..3.128 rows=1 loop
  s=1)
     Index Cond: (id = 2000)
     Heap Fetches: 0
   Planning Time: 7.522 ms
   Execution Time: 3.797 ms
  (5 rows)
  ```

- Index Only Scan using employees_pkey on employees
  - I actually scan the index, which is way faster because the index is always smaller than the actual table.
- Heap Fetches: 0
  - I did not have to go to the heap to fetch this information because the id, that's the only thing you're selecting is in the index
- Planning Time: should you use the index or should you scan the table is a decision
- Execution Time: It's the actual work to go and do the work

#### Case2

```

```

- This is a slower query than Case1
  - When we actually selected the name, which is a column in the table, it's not the column in the index
  - We found the id, then we actually tried to go to the page that has this information and we retrieve it from disk

### Case3
  ```
  postgres=# explain analyze select name from employees where id = 5000;
                                                          QUERY PLAN
  --------------------------------------------------------------------------------------------------------------------------
   Index Scan using employees_pkey on employees  (cost=0.42..8.44 rows=1 width=6) (actual time=0.405..0.406 rows=1 loops=1)
     Index Cond: (id = 5000)
   Planning Time: 0.621 ms
   Execution Time: 0.590 ms
  (4 rows)
  ```

- Obviously if I execute the same thing again, it's faster because cashing

### Case4
  
  ```
  postgres=# explain analyze select id from employees where name = 'Zs';
                                                          QUERY PLAN
  
  --------------------------------------------------------------------------------------------------------------------------
  -
   Gather  (cost=1000.00..11310.94 rows=6 width=4) (actual time=36.509..1872.232 rows=30 loops=1)
     Workers Planned: 2
     Workers Launched: 2
     ->  Parallel Seq Scan on employees  (cost=0.00..10310.34 rows=2 width=4) (actual time=72.024..1180.392 rows=10 loops=3)
           Filter: (name = 'Zs'::text)
           Rows Removed by Filter: 333324
   Planning Time: 0.504 ms
   Execution Time: 1873.094 ms
  (8 rows)
  ```

- It's deeply slow. `Execution Time: 1873.094 ms`
  - Because the name column does not have an index
  - That means the only way to actually search through the name the value Zs is to actually go one by one and do a sequential scan on table
- `Parallel Seq Scan` called the full table scan with threading but two workers run the scans in parallel

### Case5

  ```
  postgres=# explain analyze select id from employees where name like  '%Zs%';
                                                           QUERY PLAN
  
  --------------------------------------------------------------------------------------------------------------------------
  --
   Gather  (cost=1000.00..12221.54 rows=9112 width=4) (actual time=10.205..197.486 rows=1242 loops=1)
     Workers Planned: 2
     Workers Launched: 2
     ->  Parallel Seq Scan on employees  (cost=0.00..10310.34 rows=3797 width=4) (actual time=0.763..98.567 rows=414 loops=3
  )
           Filter: (name ~~ '%Zs%'::text)
           Rows Removed by Filter: 332920
   Planning Time: 1.479 ms
   Execution Time: 198.524 ms
  (8 rows)
  ```

- It is literally to go through all the rows and does match in this case

### Case6

  ```
  postgres=# create index employees_name on employees(name);
  CREATE INDEX
  ```
  
  ```
  postgres=# explain analyze select id from employees where name = 'Zs';
                                                         QUERY PLAN
  ------------------------------------------------------------------------------------------------------------------------
   Bitmap Heap Scan on employees  (cost=4.47..27.93 rows=6 width=4) (actual time=1.223..2.247 rows=30 loops=1)
     Recheck Cond: (name = 'Zs'::text)
     Heap Blocks: exact=30
     ->  Bitmap Index Scan on employees_name  (cost=0.00..4.47 rows=6 width=0) (actual time=1.162..1.163 rows=30 loops=1)
           Index Cond: (name = 'Zs'::text)
   Planning Time: 9.028 ms
   Execution Time: 2.535 ms
  (7 rows)
  ```

- It was quick(1873.094 ms case4 -> 2.535 ms) because now we're doing a Bitmap Heap Scan on table

### Case7

  ```
  postgres=# explain analyze select id from employees where name like  '%Zs%';
                                                           QUERY PLAN
  
  --------------------------------------------------------------------------------------------------------------------------
  ---
   Gather  (cost=1000.00..12221.54 rows=9112 width=4) (actual time=5.184..278.797 rows=1242 loops=1)
     Workers Planned: 2
     Workers Launched: 2
     ->  Parallel Seq Scan on employees  (cost=0.00..10310.34 rows=3797 width=4) (actual time=0.153..118.363 rows=414 loops=
  3)
           Filter: (name ~~ '%Zs%'::text)
           Rows Removed by Filter: 332920
   Planning Time: 12.309 ms
   Execution Time: 279.101 ms
  (8 rows)
  ```

- This back to the same slow query because we could not scan the index
  - The name column has an index but what you did is you're not actually asking for a literal value. You're asking for an expression.
  - There is no index that will satisfy this expression because we cannot search the index on this expression because this is not a single value
  - `Parallel Seq Scan`
- Having an index does not mean that the database will always use it
  - It's going to plan and according to the planner, will decide to use the index or not


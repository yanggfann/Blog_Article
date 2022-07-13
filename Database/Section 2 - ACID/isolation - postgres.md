## Postgres

Default isolation level: read committed

### start and stop

- pg_ctl -D /usr/local/var/postgres start
- ps aux | grep postgres
- psql postgres

### General command

- \echo :AUTOCOMMIT         // Check if autocommit is on or not
- SHOW default_transaction_isolation
- \list or \l         // list all databases
- create database compare_isolation;
- \c <db name>      // connect to a certain database

### SET TRANSACTION Statement

Refer to: https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html

- SET SESSION CHARACTERISTICS AS TRANSACTION transaction_mode     // Sets the default transaction characteristics for subsequent transactions of a session
- SET TRANSACTION transaction_mode       // Sets the characteristics of the current transaction

### Practice

#### Atomicity and consistency

- Create a database by docker
    ```
    docker run --name pgacid -d -e POSTGRES_PASSWORD=postgres postgres:13
    ```

- Make sure the container is running
    
    ```
    docker ps
    ```

- Log in

    ```
    docker exec -it pgacid psql -U postgres
    ```

- Create a table

    ```
    postgres=# create table products (
    pid serial primary key,
    name text,
    price float,
    inventory integer);
    CREATE TABLE
    ```

    ```
    postgres=# create table sales (
    saleid serial primary key,
    pid integer,
    price float,
    quantity integer);
    CREATE TABLE
    ```

- Insert a data to products table

    ```
    postgres=# insert into products (name, price, inventory) values ('Phone', 999.99, 100);
    INSERT 0 1
    ```

- Test the Atomicity

    ```
    postgres=*# select * from products;
     pid | name  | price  | inventory
    -----+-------+--------+-----------
       1 | Phone | 999.99 |       100
    (1 row)
    ```

    ```
    postgres=# begin transaction;
    BEGIN
    postgres=*# select * from products;
     pid | name  | price  | inventory
    -----+-------+--------+-----------
       1 | Phone | 999.99 |       100
    (1 row)
    
    postgres=*# update products set inventory = inventory - 10;
    UPDATE 1
    postgres=*# select * from products;
     pid | name  | price  | inventory
    -----+-------+--------+-----------
       1 | Phone | 999.99 |        90
    (1 row)
    ```

- Crash here. Will you loss 10 units from your phones? - No

    ```
    postgres=*# exit;
    ```

- log in to database again

    ```
    ➜  ~ docker exec -it pgacid psql -U postgres
    psql (13.7 (Debian 13.7-1.pgdg110+1))
    Type "help" for help.
    
    postgres=# select * from products;
     pid | name  | price  | inventory
    -----+-------+--------+-----------
       1 | Phone | 999.99 |       100
    (1 row)
    ```

We saw it's 100 not 90. Because of atomicity, we started a transaction, we did an update, but we crashed, we exited from crash. 

- Test the Atomicity again

  - Session 1
    ```
    postgres=# begin transaction;
    BEGIN
    postgres=*# update products set inventory = inventory - 10;
    UPDATE 1
    postgres=*# insert into sales (pid, price, quantity) values (1, 999.99, 10);
    INSERT 0 1
    postgres=*# select * from sales;
     saleid | pid | price  | quantity
    --------+-----+--------+----------
          1 |   1 | 999.99 |       10
    (1 row)
    
    postgres=*# select * from products;
     pid | name  | price  | inventory
    -----+-------+--------+-----------
       1 | Phone | 999.99 |        90
    (1 row)
    
    postgres=*#
    ```

    Before committed, it's only viewable in my transaction. 

    Start another command:

  - Session 2

    ```
    ➜  ~ docker exec -it pgacid psql -U postgres
    psql (13.7 (Debian 13.7-1.pgdg110+1))
    Type "help" for help.
    
    postgres=# select * from products;
     pid | name  | price  | inventory
    -----+-------+--------+-----------
       1 | Phone | 999.99 |       100
    (1 row)
    
    postgres=# select * from sales;
     saleid | pid | price | quantity
    --------+-----+-------+----------
    (0 rows)
    
    postgres=#
    ```

  - Now commit the Session 1

    ```
    postgres=*# commit;
    COMMIT
    ```

  - Select again in Session 2

    ```
    postgres=# select * from sales;
     saleid | pid | price  | quantity
    --------+-----+--------+----------
          1 |   1 | 999.99 |       10
    (1 row)
    
    postgres=# select * from products;
     pid | name  | price  | inventory
    -----+-------+--------+-----------
       1 | Phone | 999.99 |        90
    (1 row)
    ```
    
The above command shows us the atomicity and consistency. Since if you don’t have atomicity, you don’t have consistency.

#### Isolation

- Inserted a few rows in the tables

  <details>
  
  <summary>Insert command</summary>
  
  ```
  postgres=# insert into sales (pid, price, quantity) values (1, 999.99, 5);
  insert into sales (pid, price, quantity) values (1, 999.99, 5);
  insert into sales (pid, price, quantity) values (2, 999.99, 10);
  insert into sales (pid, price, quantity) values (2, 999.99, 10);
  insert into sales (pid, price, quantity) values (2, 999.99, 20);
  INSERT 0 1
  INSERT 0 1
  INSERT 0 1
  INSERT 0 1
  INSERT 0 1
  postgres=# update products set inventory = inventory - 10;
  insert into products (name, price, inventory) values ('Earbuds', 99.99, 160);
  UPDATE 1
  INSERT 0 1
  postgres=# select * from sales;
   saleid | pid | price  | quantity
  --------+-----+--------+----------
        1 |   1 | 999.99 |       10
        2 |   1 | 999.99 |        5
        3 |   1 | 999.99 |        5
        4 |   2 | 999.99 |       10
        5 |   2 | 999.99 |       10
        6 |   2 | 999.99 |       20
  (6 rows)
  
  postgres=# select * from products;
   pid |  name   | price  | inventory
  -----+---------+--------+-----------
     1 | Phone   | 999.99 |        80
     2 | Earbuds |  99.99 |       160
  (2 rows)
  ```
  
  </details>

- Default isolation level: read committed

Anything that has been committed on other transaction, I can see it in the current transaction.

  - Session 1

    ```
    postgres=# begin transaction;
    BEGIN
    postgres=*# select pid, count(pid) from sales group by pid;
     pid | count
    -----+-------
       2 |     3
       1 |     3
    (2 rows)
    ```

  - Session 2

    ```
    ➜  ~ docker exec -it pgacid psql -U postgres
    psql (13.7 (Debian 13.7-1.pgdg110+1))
    Type "help" for help.
  
    postgres=# begin transaction;
    BEGIN
    postgres=*# insert into sales (pid, price, quantity) values (1, 999.99, 10);
    INSERT 0 1
    postgres=*# update products set inventory = inventory - 10 where pid = 1;
    UPDATE 1
    postgres=*# commit;
    COMMIT
    ```

  - Back to Session 1

    ```
    postgres=*# select pid, price, quantity from sales;
     pid | price  | quantity
    -----+--------+----------
       1 | 999.99 |       10
       1 | 999.99 |        5
       1 | 999.99 |        5
       2 | 999.99 |       10
       2 | 999.99 |       10
       2 | 999.99 |       20
       1 | 999.99 |       10
    (7 rows)
  
    postgres=*# rollback;
    ROLLBACK
    ```


- Going to another isolation level: repeatable read

  - Session 1

    ```
    postgres=# begin transaction isolation level repeatable read;
    BEGIN
    postgres=*# select pid, count(pid) from sales group by pid;
     pid | count
    -----+-------
       2 |     3
       1 |     4
    (2 rows)
    ```

  - Session 2

    ```
    postgres=# begin transaction;
    BEGIN
    postgres=*# insert into sales (pid, price, quantity) values (1, 999.99, 10);
    INSERT 0 1
    postgres=*# update products set inventory = inventory - 10 where pid = 1;
    UPDATE 1
    postgres=*# commit;
    COMMIT
    ```

  - Session 1

    ```
    postgres=*# select pid, price, quantity from sales;
     pid | price  | quantity
    -----+--------+----------
       1 | 999.99 |       10
       1 | 999.99 |        5
       1 | 999.99 |        5
       2 | 999.99 |       10
       2 | 999.99 |       10
       2 | 999.99 |       20
       1 | 999.99 |       10
    (7 rows)
  
    postgres=*# select pid, count(pid) from sales group by pid;
     pid | count
    -----+-------
       2 |     3
       1 |     4
    (2 rows)
    ```

  - Session 2

    ```
    postgres=# select pid, count(pid) from sales group by pid;
     pid | count
    -----+-------
       2 |     3
       1 |     5
    (2 rows)
    ```

  Your repeatable read isolation level prevent you seeing stuff that other people's changing. It's expensive.

  - Session 1

    ```
    postgres=!# commit;
    COMMIT
    postgres=# select pid, count(pid) from sales group by pid;
     pid | count
    -----+-------
       2 |     3
       1 |     5
    (2 rows)
    ```

- Durability

  - Session 2

    ```
    postgres=# begin transaction;
    BEGIN
    postgres=*# insert into products (name, price, inventory) values ('TV', 3000, 10);
    INSERT 0 1
    postgres=*# commit;
    COMMIT
    ```

  - Session 1

    Run the following command and the `commit;` command of Session 1 at the same time.

    ```
    docker stop pgacid;
    ```

  - Session1

    ```
    docker start pgacid;
    ```

  - Session 2 Check if the `TV` has been stored in DB? - Yes

    ```
    ➜  ~ docker exec -it pgacid psql -U postgres
    psql (13.7 (Debian 13.7-1.pgdg110+1))
    Type "help" for help.
  
    postgres=# select * from products;
     pid |  name   | price  | inventory
    -----+---------+--------+-----------
       2 | Earbuds |  99.99 |       160
       1 | Phone   | 999.99 |        60
       4 | TV      |   3000 |        10
    (3 rows)
    ```

  It's written to memory first for a performance reason. If they forget to flash to disk when the container or the host dies. You just lost your stuff.

- Phantom read

**Most database actually fix the phantom read. The serialisation isolation level allows the database to serialize transactions.**
If I do begin transaction isolation level serializable and now I am in a transaction that is serializable.
That means anything that I read must not depend on other transactions that is currently running.
You will not see that change that happened on the other transactions. 
This is all true in all databases except postgres, actually, postgres is special when it comes to this.
**Postgres prevents phantom reads even in other isolation levels, such as repeatable read isolation level.**
So there are repeatable read allow you to query, execute the same query, yet get the same result.
It's almost like it's implemented as versioning because multi version concurrency control.
So when you start a transaction, you create a version that's a version of these rows.
If you do a repeatable read in MYSQL, you don't get rid of phantom read.
**The only way to get rid of phantom read for Oracle, MYSQL, SQL Server, you have to do serializable isolation level.**

### Serializable vs Repeatable Read

<details>

<summary>Prepare Data</summary>

```
➜  ~ docker exec -it pgacid psql -U postgres
psql (13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.

postgres=# create table test (
    id serial primary key,
    t text);
CREATE TABLE
postgres=# insert into test (t) values ('a');
insert into test (t) values ('a');
insert into test (t) values ('b');
insert into test (t) values ('b');
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
postgres=# select * from test;
 id | t
----+---
  1 | a
  2 | a
  3 | b
  4 | b
(4 rows)
```

</details>

#### Repeatable Read

- Session 1

  ```
  postgres=# begin transaction isolation level repeatable read;
  BEGIN
  postgres=*# update test set t = 'a' where t = 'b';
  UPDATE 2
  postgres=*# select * from test;
   id | t
  ----+---
    1 | a
    2 | a
    3 | a
    4 | a
  (4 rows)
  ```

- Session 2

  ```
  postgres=# begin transaction isolation level repeatable read;
  BEGIN
  postgres=*# update test set t = 'b' where t = 'a';
  UPDATE 2
  postgres=*# select * from test;
   id | t
  ----+---
    3 | b
    4 | b
    1 | b
    2 | b
  (4 rows)
  ```

- Session 1

  ```
  postgres=*# commit;
  COMMIT
  postgres=# select * from test;
   id | t
  ----+---
    3 | a
    4 | a
    1 | b
    2 | b
  (4 rows)
  ```

- Session 2

  ```
  postgres=# select * from test;
   id | t
  ----+---
    3 | a
    4 | a
    1 | b
    2 | b
  (4 rows)
  ```

#### Serializable

- Session 1

```
postgres=# begin transaction isolation level serializable;
BEGIN
postgres=*# update test set t = 'b' where t = 'a';
UPDATE 2
```

- Session 2

```
postgres=# begin transaction isolation level serializable;
BEGIN
postgres=*# update test set t = 'a' where t = 'b';
UPDATE 2
```

- Session 2

```
postgres=*# commit;
COMMIT
postgres=# select * from test;
 id | t
----+---
  3 | a
  4 | a
  1 | a
  2 | a
(4 rows)
```

- Session 1

```
postgres=*# commit;
ERROR:  could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.
postgres=# rollback;
WARNING:  there is no transaction in progress
ROLLBACK
postgres=# begin transaction isolation level serializable;
BEGIN
postgres=*#  update test set t = 'b' where t = 'a';
UPDATE 4
postgres=*# commit;
COMMIT
postgres=# select * from test;
 id | t
----+---
  3 | b
  4 | b
  1 | b
  2 | b
(4 rows)
```


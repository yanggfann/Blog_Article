## Question 1

When a transaction commits successfully but other transactions that started before it couldn't see the changes this means

A. The database system is not durable and as result not fully ACID, since the transactions couldn't see the changes which were committed

B. The isolation level of the transactions started before the transaction in question is higher than read-committed

C. This is a bug in the database system, any committed change should be immediately visible to all transactions.

<details>

<summary>Answer</summary>

```
B

Any transaction started before with a repeatable read isolation level or higher means that it will not pick up committed 
reads by design. However transactions started after will see all committed changes
```

</details>

## Question 2

Alice needs to execute 10 DML statements atomically (all or nothing). Alice needs to do the following to achieve true atomicity.

A. Alice needs to start 10 transactions and atomically commits each statement. If any of the 10 transactions fails she needs to retry it until it succeeds.

B. Alice needs to start 1 transaction and issue a commit after executing each statement. 

C. Alice needs to start 1 transaction and issue a commit after executing all statements.

D. The execution of each statement is already atomic, Alice doesn't need to start any transaction

<details>

<summary>Answer</summary>

```
C

The commit at the end will flush the WAL changes and as a result we will either get a success if the write happened or 
failure if anything went wrong. If the power went off before the commit we won't get any change, if the power went off 
right after the commit we will get the correct result. If the power went out during the commit the database will restart 
and detect the the half/commit state and rollback all changes. If any failure happens during an execution of one of the 
statements the DB will rollback the transaction and all the changes.
```

</details>

## Question 3

Which one of the following SQL can cause a phantom read phenomenon?
A)SELECT * FROM EMPLOYEES WHERE ID = 1000
B)SELECT * FROM EMPLOYEES WHERE DATE_JOINED < '2021-1-1'
C)SELECT * FROM EMPLOYEES


A. A

B. B

C. C

D. All of the above

<details>

<summary>Answer</summary>

```
D

Depending on the isolation level of the reading transaction but all of these queries if repeated can give an additional 
result. ID = 1000 might not exist but some other transaction might have inserted it New employees might have joined 
before this date and results might show up. And unbounded query (no where clause) will show any new row that is inserted to the table
```

</details>

## Question 4

Jaffar is architecting a database system. He set up a single primary database with 4 worker replicas. He configured the 
system so it uses asynchronous replication with 4. This means that a commit to the primary will immediately succeed and 
asynchronously the primary will issue commits on the background to the 4 replicas.

A. Eventually Consistent with fast writes

B. Strongly Consistent with fast writes

C. Strongly Consistent with slow writes

D. Highly isolated 

<details>

<summary>Answer</summary>

```
A

Correct, while writes are faster since the only commit we need to wait for are the primary's however there will be reads 
of off the worker nodes that will mismatch with the primary node. Thus the system will have eventual consistency or perhaps weaker than that.
```

</details>

## Question 5

Sarah issued a transaction with 100 DML statements that touched a lot of rows. During the commit, her database crashed.
When the database is restarted Sarah's changed were not persisted. Which of these statements are true? 

A. Sarah is using an in-memory data store that is not durable. If she was using a fully ACID relational database such as Postgres her changes will never have been lost

B. Sarah should have issued mini-commits between each few statements

C. Sarah should stop issuing 100 statement transactions.

D. None of the above

<details>

<summary>Answer</summary>

```
D

Since the crash happened during the commit the database cannot guarantee durability. The system is durable only when the 
commit is successful( the data is fully written to disk). That is why commit speeds are critical, the faster you can 
commit the lower the chances of such corruption.
```

</details>

## Question 6

What is the output? except Postgres

```
create table test (id integer)
insert into test (2);

begin transaction 1
t1 - set isolation level read committed
begin transaction 2
t2 - set isolation level repeatable read

t1 - select * from test;
t2 - insert into test (4);
t1 - select * from test;
t1 - insert into test (5);
t2 - select * from test;
t1 - commit;
t2 - select * from test;
t2 - commit;
```

A. 2 | 2 | 2-4 | 2-4

B. 2 | 2 | 2-4 | 2-4-5

C. 2 | 2-4 | 2-4 | 2-4

<details>

<summary>Answer</summary>

```
B

This answer is correct in all platforms except Postgres. Repeatable read isolation level only guarantees if you read a 
value that value will remain unchanged. This however doesn't apply to phantom reads, the 5 that was commited, was never 
read by the transaction as a result it will show up. Postgres repeatable read however is implemented as snapshot 
isolation so it doesn't allow for phantom reads.
```

</details>

## Question 7

To ensure durability, database systems write to changes of the transaction to disk. However for those changes to persist 
on disk they should force the OS to flush to disk instead of OS cache, this is done through which OS operation?

A. fsync

B. flush-cache

C. write

D. write-force

<details>

<summary>Answer</summary>

```
A
```

</details>

## Question 8

Assume table LOGS with one row

LOG_ID | LOG_TEXT

1             |        NULL


A transaction begins with read committed isolation level and executes the following two statements

```
BEGIN ISOLATION LEVEL READ COMMITTED;

UPDATE LOGS SET LOG_TEXT = 'HELLO' WHERE LOG_ID = 1;

SELECT LOG_ID, LOG_TEXT FROM LOGS WHERE LOG_ID = 1;

COMMIT;
```

What will the select statement returns?

A. NULL - Because read committed doesn't allow for dirty reads

B. HELLO - The transaction reads the value it writes

<details>

<summary>Answer</summary>

```
B

Correct, the transaction will always see the changes it makes regardless of the isolation level. 
Isolation level only applies to other concurrent transactions
```

</details>

## Question 9

Jeff executed the following update statement in the terminal on a table with 100 million rows;

`UPDATE TABLE STUDENTS SET GRADE = 100;`

He realized that he forgot to add a where clause to only update the row where student id 50. Jeff  executed a rollback immediately in the terminal.

`ROLLBACK;`

He then rerun the statement again

`UPDATE TABLE STUDENTS SET GRADE = 100 WHERE STUDENT_ID = 50;`

What is the state of the table after the last statement?

A. The rollback was successful, the table is returned to its original state before the first update and the only change is student id 50 has a grade of 100

B. All the 100 million students will have a grade of 100

C. Rollback will fail and terminate the session

D. The first statement will fail because it is not inside a transaction

<details>

<summary>Answer</summary>

```
B

Correct. Because the statement was not executed in a transaction, the database will wrap it in its own transaction and 
commit immediately. Calling rollback after the fact won't do anything since the transaction was committed. The rollback 
might say "success" but nothing will happen since there isn't anything to rollback. The 3rd statement will try to update 
the student with ID 50 which already have a grade 100.
```

</details>

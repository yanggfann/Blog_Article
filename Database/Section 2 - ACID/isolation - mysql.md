## Mysql

Default isolation level: REPEATABLE READ

### General command

- SELECT @@autocommit;
- SELECT @@transaction_ISOLATION;
- SELECT @@global.transaction_ISOLATION;
- show variables like 'autocommit';

### SET TRANSACTION Statement

Refer to: https://dev.mysql.com/doc/refman/8.0/en/set-transaction.html

- SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;  // Affected Characteristic Scope: Session
- SET TRANSACTION transaction_characteristic;      // Affected Characteristic Scope: Next transaction only

### MySQL中的事务控制语句

MySQL中的事务控制语句有START TRANSACTION、COMMIT、ROLLBACK、SET autocommit，它们协作起来实现对事务的管控。

- START TRANSACTION或BEGIN开启新事务。 
- COMMIT提交当前事务，使更改持久生效。
- ROLLBACK回滚当前事务，取消更改。
- SET autocommit禁用或启用当前会话的默认自动提交模式。

**默认情况下，MySQL启用自动提交模式。这意味着，如果不是在事务内部，则每个语句都是原子的，就像它被START transaction和COMMIT包围一样。不能使用ROLLBACK撤消，除非语句执行期间发生错误自动回滚。**

<details>

<summary>自动提交模式</summary>

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| myth               |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.05 sec)

mysql> create database compare_isolation;
Query OK, 1 row affected (0.02 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| compare_isolation  |
| information_schema |
| mysql              |
| myth               |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)

mysql> use compare_isolation;
Database changed
mysql> CREATE TABLE Account (
       ACCOUNT_ID int,
       BALANCE int
   );
Query OK, 0 rows affected (0.02 sec)

mysql> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.01 sec)

mysql> insert into Account value (2, 500);
Query OK, 1 row affected (0.02 sec)

mysql> insert into Account value (1, 1000);
Query OK, 1 row affected (0.00 sec)

mysql> select * from Account;
+------------+---------+
| ACCOUNT_ID | BALANCE |
+------------+---------+
|          2 |     500 |
|          1 |    1000 |
+------------+---------+
2 rows in set (0.00 sec)

mysql> delete from Account where ACCOUNT_id=1;
Query OK, 1 row affected (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from Account;
+------------+---------+
| ACCOUNT_ID | BALANCE |
+------------+---------+
|          2 |     500 |
+------------+---------+
1 row in set (0.00 sec)
```

</details>

可以看到，会话默认情况下，确实是事务自动提交模式。当删除ACCOUNT_id=1这条记录后，因为自动提交，数据已经真实的从表中删除了，所以紧接的rollback没有改变任何数据，只是自己执行了而已。

同样的动作，我们人为的用事务控制起来看看:

<details>

<summary>人为的用事务控制</summary>

```
mysql> select * from Account;
+------------+---------+
| ACCOUNT_ID | BALANCE |
+------------+---------+
|          2 |     500 |
|          1 |    1000 |
+------------+---------+
2 rows in set (0.00 sec)

mysql> start transaction;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from Account;
+------------+---------+
| ACCOUNT_ID | BALANCE |
+------------+---------+
|          2 |     500 |
|          1 |    1000 |
+------------+---------+
2 rows in set (0.00 sec)

mysql> delete from Account where ACCOUNT_id=1;
Query OK, 1 row affected (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from Account;
+------------+---------+
| ACCOUNT_ID | BALANCE |
+------------+---------+
|          2 |     500 |
|          1 |    1000 |
+------------+---------+
2 rows in set (0.00 sec)
```

</details>

可以看到，人为控制事务时，事务的范围就从每个语句自己是一个事务，变成了命令指定的范围，这时rollback或者commit就会对事务内的所有变化落地还是撤销产生影响。

START TRANSACTION允许使用选项来控制事务特性，要指定多个选项时，用逗号分隔。

- WITH CONSISTENT SNAPSHOT，启动一致读取，只适用于InnoDB。begin/start transaction命令并不是一个事务的起点，在执行到它们之后的第一个语句，事务才真正启动。如果你想要马上启动一个事务，可以使用 start
transaction with consistent snapshot这个命令。其效果与发出一个start transaction，同时执行一个SELECT相同。
- READ WRITE和READ ONLY设置事务访问模式。它们允许或禁止更改事务中使用的表。READ ONLY限制事务修改或锁定其他事务可见的表，但事务仍然可以修改或锁定临时表。如果未指定访问模式，默认为READ WRITE。
- BEGIN和BEGIN WORK是START TRANSACTION的别名. 

**要禁用自动提交模式，使用SET autocommit=0; 注意，autocommit是会话变量，每次对autocommit的修改只影响当前会话。**

<details>

<summary>禁用自动提交模式</summary>

```
mysql> select * from Account;
+------------+---------+
| ACCOUNT_ID | BALANCE |
+------------+---------+
|          2 |     500 |
|          1 |    1000 |
+------------+---------+
2 rows in set (0.00 sec)

mysql> SELECT @@autocommit;
+--------------+
| @@autocommit |
+--------------+
|            1 |
+--------------+
1 row in set (0.00 sec)

mysql> SET autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> delete from Account where ACCOUNT_id=1;
Query OK, 1 row affected (0.00 sec)

mysql> rollback;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from Account;
+------------+---------+
| ACCOUNT_ID | BALANCE |
+------------+---------+
|          2 |     500 |
|          1 |    1000 |
+------------+---------+
2 rows in set (0.00 sec)
```

</details>

### Isolation Levels

The following list describes how MySQL supports the different transaction levels. The list goes from the most commonly used level to the least used.

事务隔离是数据库的基础能力，ACID中的I指的就是事务隔离，通俗点讲就是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。

#### REPEATABLE READ

This is the default isolation level for InnoDB.

<details>

<summary>会话1 - REPEATABLE READ</summary>

```
mysql> SELECT @@transaction_ISOLATION;
+-------------------------+
| @@transaction_ISOLATION |
+-------------------------+
| REPEATABLE-READ         |
+-------------------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE Sales (
     Pid varchar(255),
     Qnt int,
     Price int
    );
Query OK, 0 rows affected (0.06 sec)

mysql> SET autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | OFF   |
+---------------+-------+
1 row in set (0.03 sec)

mysql> insert into Sales value ('Product 1', 10, 5);
Query OK, 1 row affected (0.00 sec)

mysql> insert into Sales value ('Product 2', 20, 4);
Query OK, 1 row affected (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.01 sec)

mysql> select * from Sales;
+-----------+------+-------+
| Pid       | Qnt  | Price |
+-----------+------+-------+
| Product 1 |   10 |     5 |
| Product 2 |   20 |     4 |
+-----------+------+-------+
2 rows in set (0.00 sec)

mysql> update Sales set Qnt=15 where Pid='Product 1';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from Sales;
+-----------+------+-------+
| Pid       | Qnt  | Price |
+-----------+------+-------+
| Product 1 |   15 |     5 |
| Product 2 |   20 |     4 |
+-----------+------+-------+
2 rows in set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)
```

</details>

会话1可以看到Qnt的变化，并且已提交。会话2在REPEATABLE READ下感受不到该变化，因为他们还在自己的事务中（由select * from Sales;隐式开启了事务，因为SET autocommit=0;），根据REPEATABLE READ事务隔离的原则确实不应该看到。

当会话2结束当前事务(commit;)后，再去查询就能看到变化了。

<details>

<summary>会话2 - REPEATABLE READ</summary>

```
mysql> SELECT @@transaction_ISOLATION;
+-------------------------+
| @@transaction_ISOLATION |
+-------------------------+
| REPEATABLE-READ         |
+-------------------------+
1 row in set (0.00 sec)

mysql> SET autocommit=0;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | OFF   |
+---------------+-------+
1 row in set (0.01 sec)

mysql> select * from Sales;
+-----------+------+-------+
| Pid       | Qnt  | Price |
+-----------+------+-------+
| Product 1 |   10 |     5 |
| Product 2 |   20 |     4 |
+-----------+------+-------+
2 rows in set (0.00 sec)

mysql> select * from Sales;
+-----------+------+-------+
| Pid       | Qnt  | Price |
+-----------+------+-------+
| Product 1 |   10 |     5 |
| Product 2 |   20 |     4 |
+-----------+------+-------+
2 rows in set (0.01 sec)

mysql> select * from Sales;
+-----------+------+-------+
| Pid       | Qnt  | Price |
+-----------+------+-------+
| Product 1 |   10 |     5 |
| Product 2 |   20 |     4 |
+-----------+------+-------+
2 rows in set (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from Sales;
+-----------+------+-------+
| Pid       | Qnt  | Price |
+-----------+------+-------+
| Product 1 |   15 |     5 |
| Product 2 |   20 |     4 |
+-----------+------+-------+
2 rows in set (0.00 sec)
```

</details>

#### READ COMMITTED

#### READ UNCOMMITTED

#### SERIALIZABLE

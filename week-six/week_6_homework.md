# Week 6 Homework

### 题目 01

一条 SQL 语句在 MySQL 中是如何执行的？

```
第一步：连接器连接到数据库
客户端通过连接器建立连接，进行权限验证。
如果有权限，就可以继续执行后面的操作。
连接完成后，如果没有后续动作，则连接处于空闲状态。
客户端如果太长时间（由参数 wait_timeout 控制，默认值是 8 小时）没动静，连接器就会自动断开。

第二步：查询缓存
MySQL 会先检查查询缓存中是否有相同的 SQL 语句和结果，如果有，会被直接返回给客户端，否则就进入下一步。
执行完成后，执行结果会被存入查询缓存中。
MySQL 8.0 及以上版本已经移除了查询缓存功能。
缓存功能缺点：
成本高：查询缓存的失效非常频繁，只要有对一个表的更新，这个表上所有的查询缓存都会被清空。
命中率不高：对于更新压力大的数据库来说，查询缓存的命中率会非常低。

第三步：分析器分析SQL语句
客户端程序发送过来的请求，实际上是字符串。
MySQL 会对 SQL 语句进行词法分析和语法分析，将 SQL 语句解析成 MySQL 能理解的结构化表示。
编译的过程，有词法解析、语法分析、预处理器。
词法分析为把一个完整的 SQL 语句分割成一个个的字符串。
语法分析器根据词法分析的结果做语法检查，判断输入的SQL语句是否满足MySQL的语法。
预处理器则会进一步去检查解析树是否合法，比如表名是否存在，语句中表的列是否存在，是否有表操作权限等等。
预处理之后会得到一个新的解析树，然后调用对应执行模块

第四步：优化器优化SQL语句
MySQL 会对 SQL 语句进行优化，根据解析树生成不同的执行计划。
比如选择合适的索引，确定表的连接顺序等，然后选择最优的执行计划。

第五步：执行器执行SQL语句
开始执行之前会判断一下有没有执行查询的权限，如果没有，就会返回没有权限的错误。
如果有权限，就使用指定的存储引擎打开表开始查询。
执行器会根据表的引擎定义，去使用这个引擎提供的查询接口，对数据进行存取操作，并返回结果给客户端。
```

### 题目 02

请解释一下你理解的事务是什么？

要点：

事务四大特性 ACID

事务隔离级别

事务会产生的并发问题

事务的安全性、性能与隔离级别的关系

```
事务是指对数据库执行一组逻辑操作，这组操作要么全部成功，要么全部失败。
不会出现部分成功的情况。
本质上为并发编程问题。

事务的四大特性ACID为：
原子性（Atomicity）：事务的最小工作单元。事务中的所有操作，要么全部完成，要么全部不完成，不会停留在中间某个环节。
一致性（Consistency）：事务必须使数据库从一个一致性状态变换到另一个一致性状态，其完整性必须保持一致。即数据的修改必须符合所有的预设规则和约束。
隔离性（Isolation）：多个用户并发使用数据库时，事务的执行不能被其他事务干扰，即一个事务内部的操作和使用的数据对其他并发事务是隔离的。
持久性（Durability）：事务一旦提交，它对数据库中数据的改变就是永久性的，即使系统故障也不会丢失。

事务隔离级别为：
读未提交
（Read Uncommitted）：事务执行过程中可以读取到并发执行的其他事务中未提交的数据，可能会引起脏读、不可重复读、幻读。
读已提交（Read Committed）：事务执行过程中只能读到其他事务已经提交的数据，避免了脏读问题，但可能会出现不可重复读和幻读的问题。
可重复读（Repeatable Read）：当前事务执行开始后，所读取到的数据都是该事务刚开始时所读取的数据和自己事务内修改的数据，无论另一个事务是否提交，避免了不可重复读的问题，但还是有可能出现幻读的问题。
串行化（Serializable）：当前事务执行期间，其他事务不能对数据进行修改，也就是说所有事务都是串行执行的，相当于单线程，避免了所有并发问题，但性能最差。

事务会产生的并发问题有：
脏读：一个事务读到了另一个事务未提交的数据
不可重复读：一个事务读到另一个事务已经update的数据，引发事务中的多次查询结果不一致
幻读/虚读：一个事务读到另一个事务已经insert的数据，导致多次事务中多次查询的结果不一致

事务的安全性、性能与隔离级别的关系：
性能：串行化 < 可重复读 < 读已提交 < 读未提交
安全性：串行化 > 可重复读 > 读已提交 > 读未提交
```


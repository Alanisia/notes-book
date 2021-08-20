# SQL

# MySQL

## 存储引擎

## 索引


## 事务

四大特性（ACID）：

- 原子性（Atomicity）
- 一致性（Consistency）
- 隔离性（Isolation）
- 持久性（Durability）

### 并发事务带来的问题

- 脏读

- 丢失修改

- 不可重复读

- 幻读

### 事务隔离级别

1. 读取未提交（READ-UNCOMMITTED）

2. 读取已提交（READ-COMMITTED）

3. 可重复读（REPEATABLE-READ）

4. 串行化（SERIALIZABLE）

### 锁机制

- 表级锁：锁定粒度最大的一种锁，对当前操作的整张表加锁。实现简单，资源消耗比较少，加锁快，不会出现死锁。其锁定粒度大，触发锁冲突概率最高，并发度最低。MyISAM和InnoDB引擎都支持表级锁。

- 行级锁：锁定粒度最小的一种锁，只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突，其加锁粒度小，并发度高，但加锁的开销也大，加锁慢，会出现死锁。

InnoDB存储引擎的锁算法有三种：

1. Record Lock：单个行记录上的锁
2. Gap Lock：间隙锁，锁定一个范围，不包括记录本身
3. Next-key Lock：锁定一个范围，包括记录本身


# NoSQL
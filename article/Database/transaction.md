# Transaction

事务是满足ACID的一组操作，事务提交（commit）出错时，可以进行回滚（rollback）。

## ACID
1. Atomicity: 要么全部执行，要么都不执行
2. Consistency: 用户的操作将数据库的状态从一个一致状态转变为另一个一致状态
3. Isolation: 一个事务提交之前，所做的操作对其他事务不可见
4. Durability: 已提交事务对数据库的修改应该永久保存在数据库中

## 一致性问题
1. 丢失数据
2. 读到脏数据
3. 不可重复读
4. 幻影读

## 隔离级别
1. READ UNCOMMITED

未提交读级别，事务没有提交，但是对其他事务可见。会有丢失数据，读到脏数据，不可重复读，幻影读问题。

2. READ COMMITED

提交读级别，事务没有提交前，对其他事务不可见，只解决了读到脏数据问题。

3. REPEATABLE READ

可重复读级别，解决了不可重复读问题。

4. SERIALIZABLE 

事务串行执行，不会有并发问题。

## Spring Transaction
Spring事务有两种使用方法：编程式事务与声明式（Xml或注解）事务。

### 隔离级别
TransactionDefinition接口定义了5种隔离级别。

1. TransactionDefinition.ISOLATION_DEFAULT: 
使用后端数据默认的隔离级别，MySQL默认的隔离级别是REPEATABLE READ，Oracle默认为READ COMMITED
2. TransactionDefinition.ISOLATION_READ_UNCOMMITED
3. TransactionDefinition.ISOLATION_READ_COMMITED
4. TransactionDefinition.ISOLATION_REPEATABLE_READ
5. TransactionDefinition.ISOLATION_SERIALIZABLE

### 事务传播
支持当前事务的情况：

1. TransactionDefinition.PROPAGATION_REQUIRED
如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
2. TransactionDefinition.PROPAGATION_SUPPORTS
如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
3. TransactionDefinition.PROPAGATION_MANDATORY
如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

不支持当前事务的情况：

1. TransactionDefinition.PROPAGATION_REQUIRES_NEW
创建一个新的事务，如果当前存在事务，则把当前事务挂起。
2. TransactionDefinition.PROPAGATION_NOT_SUPPORTED
以非事务方式运行，如果当前存在事务，则把当前事务挂起。
3. TransactionDefinition.PROPAGATION_NEVER
以非事务方式运行，如果当前存在事务，则抛出异常。

其他情况：

1. TransactionDefinition.PROPAGATION_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

### 事务的提交

#### 编程式事务
#### 声明式事务

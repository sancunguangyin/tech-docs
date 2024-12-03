# 理解Spirng事物

## 1. 事物解决了什么问题
### 1.1 什么是事务
事物的定义：事务是必须满足4个条件（ACID）：：原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）。其中AID由数据库保证，一致性则由应用程序保证。
### 1.2 为什么需要事务
让多个操作（方法/类）处于同一个事物中

## 如何实现事物
在没有事物管理的时候，实现一个事物需要：
```
// MethodA:
public void methodA(){
    Connection connection = acquireConnection();
    try{
        connection.prepareStatement().executeUpdate();
        methodB(connection);
        connection.commit();
    }catch (Exception e){
        rollback(connection);
    }finally {
        releaseConnection(connection);
    }
}

// MethodB:
public void methodB(Connection connection){
    int updated = connection.prepareStatement().executeUpdate();
}
```
**上面的方式有什么缺陷？**
如果有多个方法需要加入事物，或者不同的方法或者sql需要用到不同的事物以及不同的事物配置，那每次都去执行这样的代码就比较繁琐。

为了简化格式化的代码，可以把事物的相关操作封装成公共方法，实现事物的管理。

## 3. Spring 事物管理实现了解决了什么问题
先说结论
- 连接/资源管理 - 无需手动获取资源、共享资源、释放资源
- 嵌套事务的支持 - 支持嵌套事务中使用不同的资源策略、回滚策略
- 每个事务/连接使用不同的配置 - 支持对不同的事物使用不同的隔离级别，超时时间等

### 嵌套事物
什么是嵌套事物？
> 多个方法调用链中均使用事物的场景，就是嵌套事物。
前面提到的方法A调用方法B，共用同一个事物情况，是嵌套事物的一种，此外还有一种是在方法链中使用不同事物。
### 事物传播行为
什么是事物的传播行为？
> 前面提到的方法A调用方法B，且将事物传递给方法B，是一种共享事物传递方式，官方给出的传播行为如下:
>> PROPAGATION_REQUIRED 
>> PROPAGATION_SUPPORTS
>> PROPAGATION_MANDATORY
>> PROPAGATION_REQUIRES_NEW
>> PROPAGATION_NOT_SUPPORTED
>> PROPAGATION_NEVER
总结一下，事物传播行为无非有三类：
* 优先使用当前事务
* 不使用当前事务，新建事务
* 不使用任何事务
### 事物回滚策略 AOP
在注解版的事务管理中，默认的的回滚策略是：抛出异常就回滚。这个默认策略挺好，连回滚都帮我们解决了，再也不用手动回滚。
但实际上，事物的回滚与抛异常是两件独立的事，不管有没有异常都可以回滚，所以在实际应用中，我们需要自定义回滚策略，这样才能满足我们的需求。

## 4. Spring 中事物管理器的实现
回头来看可以发现，事物管理的核心功能只有两个，事物的提交以及回滚。其它的关于事物的传播、嵌套以及回滚等操作都是基于这两个功能实现。

Spring将这些事物的核心功能抽象封装成一个事物管理器`TransactionManager`，事物管理器包含三个接口
- `getTransaction()`**获取事物资源**，资源包括JDBC connection、MyBatis session等，然后将资源绑定并存储
- `commit()`**提交事物**，提交指定的事物资源
- `rollback()`**回滚事物**，回滚指定的事物资源
```
interface PlatformTransactionManager{
    // 获取事务资源
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
    
    // 提交事务
    void commit(TransactionStatus status) throws TransactionException;
    
    // 回滚事务
    void rollback(TransactionStatus status) throws TransactionException;
}
```



### 4.1 声明式事务管理
#### 4.1.1 使用 @Transactional 注解
#### 4.1.2 注解关键属性简介
##### propagation
##### isolation
##### rollbackFor
### 4.2 事务管理器概述
#### 4.2.1 DataSourceTransactionManager

## 5. 什么情况下Spring事务会失效（5分钟）
### 5.1 调用方法自身内部的事务
### 5.2 非公共方法上的事务注解
### 5.3 非 Spring 管理的对象调用事务方法
### 5.4 异常未触发事务回滚
#### 5.4.1 检查异常 vs 非检查异常

## 6. Spring 事务的最佳实践（5分钟）
### 6.1 使用专用的业务层调用事务
### 6.2 在服务层上使用 @Transactional
### 6.3 配置适当的隔离级别
### 6.4 使用只读事务优化查询性能
### 6.5 根据需要配置事务超时时间


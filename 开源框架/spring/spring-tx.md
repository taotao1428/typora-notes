# spring事务模型

spring通过txManager管理事务。



PlatformTransactionManager是管理事务的最高接口

![image-20220925162902008](/开源框架/spring/.assert/spring-tx/image-20220925162902008.png)



```java

public interface PlatformTransactionManager {
	
  // 根据事务申请，获取一个事务
	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

	// 提交一个事务
	void commit(TransactionStatus status) throws TransactionException;

	
  // 回滚一个事务
	void rollback(TransactionStatus status) throws TransactionException;
}

```



TransactionDefinition表示事务的定义

1. timeout，表示事务执行的时间，事务管理器会自动根据事务剩余的时间，设置sql的执行时间。默认时间为-1，表示不限制时间
2. isolationLevel，表示事务隔离等级。也就是事务模型中的隔离等级。read uncommited，read commited，repeatable read，SERIALIZABLE
3. propagationBehavior，表示事务传播级别。

<img src="/开源框架/spring/.assert/spring-tx/image-20220925163736411.png" alt="image-20220925163736411" style="zoom:50%;" />

![img](/开源框架/spring/.assert/spring-tx/955092-20180609182332197-1223380728.png)



传播级别

在开启



1. 1+5为PROPAGATION_REQUIRED
2. 1+6为PROPAGATION_MANDATORY
3. 1+7为PROPAGATION_SUPPORTS
4. 2+5为PROPAGATION_REQUIRES_NEW
5. 3+6为PROPAGATION_NEVER
6. 4+7为PROPAGATION_NOT_SUPPORTED
7. PROPAGATION_NESTED，嵌套事务

<img src="/开源框架/spring/.assert/spring-tx/image-20220925230153407.png" alt="image-20220925230153407" style="zoom:50%;" />

<img src="/开源框架/spring/.assert/spring-tx/image-20220925225219132.png" alt="image-20220925225219132" style="zoom:50%;" />
# Spring JDBC



spring jdbc提供了基础的jdbc操作，其中比较重要的就是支持线程事务。简单说，就是在线程中记录当前正在执行的事务，可以很方便的在不同逻辑中使用相同的事务。





TransactionSynchronizationManager保存线程本地的资源



resources保存线程本地资源

```java
private static final ThreadLocal<Map<Object, Object>> resources =
			new NamedThreadLocal<Map<Object, Object>>("Transactional resources");


// 将资源放到map中，如果key已经存在有效的value，将会抛出异常。通常value是ResourceHolder对象
void bindResource(Object key, Object value);

// 将资源从map中删除，如果key没有有效value，将会保存异常
Object unbindResource(Object key)

// 判断map中是否有资源
boolean hasResource(Object key)

// 从map中获取资源
Object getResource(Object key)
```



synchronizations保留事务同步对象

```java
	private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations =
			new NamedThreadLocal<Set<TransactionSynchronization>>("Transaction synchronizations");

// 激活事务同步，如果已经激活过事务，将会抛出异常
void initSynchronization()

// 判断事务是否已经激活
boolean isSynchronizationActive()
  

// 添加一个事务同步
void registerSynchronization(TransactionSynchronization synchronization)
  
// 获取所有的事务同步
List<TransactionSynchronization> getSynchronizations()


// 清除所有事务同步，并取消激活状态
void clearSynchronization()
```








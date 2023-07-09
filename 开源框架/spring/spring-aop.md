# spring-aop



spring-aop是针对切面



抽象接口

1. Advice，表示拦截后的操作

![image-20220908211848528](/开源框架/spring/.assert/spring-aop/image-20220908211848528.png)





2.  advisor，提供Advice的角色

<img src="/开源框架/spring/.assert/spring-aop/image-20220908212531736.png" alt="image-20220908212531736" style="zoom:50%;" />



3. pointcut，表示切面的位置

<img src="/开源框架/spring/.assert/spring-aop/image-20220908212833286.png" alt="image-20220908212833286" style="zoom:50%;" />









## @Async注解



@Async注解可以允许某一个方法被异步调用，这个也是使用aop实现的。



### 流程讲解

#### 启动@Async

使用@EnableAsync可以开启@Async注解。@EnableAsync注解会引入`AsyncConfigurationSelector`。然后会引入`ProxyAsyncConfiguration`配置。







### @Aspect注解





1. 什么时候使用代理接口，什么时候代理类





1. proxyFactory的实现





创建代理的步骤

1. 添加beanFactoryProcessor，用于拦截bean的创建过程，实现代理
2. 使用jdk或者cglib完成代理动作。在代理中，通过advisor完成对方法的拦截。





@Async AsyncAnnotationBeanPostProcessor  AsyncAnnotationAdvisor



AnnotationAwareAspectJAutoProxyCreator

查找所有切面

org.springframework.aop.framework.autoproxy.AbstractAdvisorAutoProxyCreator#findCandidateAdvisors

1. 实现了接口Advisor的bean
2. 解析@Aspect注解的bean，从中提取Advisor。包括：afterThrowing，afterReturning，after，around，before五种。



切面的顺序

按照Advisor或者@Aspect注解bean的排序顺序排序。如果从@Aspect中提取了多个Advisor，将会按照@AfterThrowing，@AfterReturning，@After，@Around，@Before



```
Around.class, Before.class, After.class, AfterReturning.class, AfterThrowing.class
```



Around.class

```java
// org.springframework.aop.aspectj.AspectJAroundAdvice
	public Object invoke(MethodInvocation mi) throws Throwable {
		if (!(mi instanceof ProxyMethodInvocation)) {
			throw new IllegalStateException("MethodInvocation is not a Spring ProxyMethodInvocation: " + mi);
		}
		ProxyMethodInvocation pmi = (ProxyMethodInvocation) mi;
		ProceedingJoinPoint pjp = lazyGetProceedingJoinPoint(pmi);
		JoinPointMatch jpm = getJoinPointMatch(pmi);
		return invokeAdviceMethod(pjp, jpm, null, null);
	}
```



Before.class

```java
// AspectJMethodBeforeAdvice
// MethodBeforeAdviceInterceptor
	public Object invoke(MethodInvocation mi) throws Throwable {
		this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
		return mi.proceed();
	}
```



After.class

```java
// AspectJAfterAdvice
	public Object invoke(MethodInvocation mi) throws Throwable {
		try {
			return mi.proceed();
		}
		finally {
			invokeAdviceMethod(getJoinPointMatch(), null, null);
		}
	}
```



AfterReturning.class

```java
// AspectJAfterReturningAdvice
// AfterReturningAdviceInterceptor
	@Override
	public Object invoke(MethodInvocation mi) throws Throwable {
		Object retVal = mi.proceed();
		this.advice.afterReturning(retVal, mi.getMethod(), mi.getArguments(), mi.getThis());
		return retVal;
	}

```



AfterThrowing.class

```java
// AspectJAfterThrowingAdvice
	public Object invoke(MethodInvocation mi) throws Throwable {
		try {
			return mi.proceed();
		}
		catch (Throwable ex) {
			if (shouldInvokeOnThrowing(ex)) {
				invokeAdviceMethod(getJoinPointMatch(), null, ex);
			}
			throw ex;
		}
	}
```



<img src="/开源框架/spring/.assert/spring-aop/image-20220911151735502.png" alt="image-20220911151735502" style="zoom:50%;" />





代理的两个配置

1. proxyTargetClass：是否代理class，默认false。如果代理类，将会使用cglib，如果是代理接口，将会使用jdk自动代理
2. exposeProxy：是否暴露proxy对象，默认false。如果为true，将可以通过AopProxy获取代理后对象。







## 代理的实现



### 代理接口还是代理子类



<img src="/开源框架/spring/.assert/spring-aop/image-20220911162711525.png" alt="image-20220911162711525" style="zoom:50%;" />



<img src="/开源框架/spring/.assert/spring-aop/image-20220911164003254.png" alt="image-20220911164003254" style="zoom:50%;" />



如果代理类，将会使用ObjenesisCglibAopProxy

如果代理接口，将会使用JdkDynamicAopProxy



### 代理使用的ClassLoader

1. 默认是`Thread.currentThread().getContextClassLoader()`或者ClassUtils的classLoader



### 代理工厂ProxyFactory

org.springframework.aop.framework.ProxyFactory

使用ObjenesisCglibAopProxy或者JdkDynamicAopProxy创建代理对象



### 代理工厂JdkDynamicAopProxy

增加接口SpringProxy（表示是否为Spring生成的代理对象），Advised（表示切面的一些信息，包含被代理的对象和所有Advice等）和DecoratingProxy（用于保留目标对象的class）



1. 如果方法返回值为当前被代理对象，返回值会被替换为代理对象（除非方法声明的类实现了RawTargetAccess接口）
2. 如果返回值为null，并且方法返回值为基础类型，将会抛出异常。
3. 如果设置了exposeProxy为true，将会把代理对象在在AopContext中



### 代理工厂ObjenesisCglibAopProxy

1. 使用cglib实现代理
2. 使用objenesis实现实例的初始化，objenesis可以在不执行构造方法时实例化对象。








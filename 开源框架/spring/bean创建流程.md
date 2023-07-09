# Bean创建流程



bean的创建流程由BeanFactory完成。DefaultListableBeanFactory是当前默认的实现类

![image-20220912102816456](/开源框架/spring/.assert/bean创建流程/image-20220912102816456.png)



使用工厂方法创建bean？org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#instantiateUsingFactoryMethod



查找Constructor的逻辑。AutowiredAnnotationBeanPostProcessor#determineCandidateConstructors

1. 有注解@Autowired的构造方法，会被列入候选。允许3种情况：有1个@Autowired(required=true)的构造方法，有1个或者多个@Autowired(required=false)的构造方法，没有注解@Autowired的构造方法
2. 如果无参构造方法没有注解@Autowired，将会认为是默认构造方法



1. 如果@Autowired是第一种情况，将会将该构造方法返回
2. 如果@Autowired是第二种情况，将会把所有候选+默认构造方法返回
3. 如果@Autowired是第三种情况，并且只有一个有参的构造方法，将会把这个构造方法返回。否者，不返回任何构造方法





使用构造方法创建对象

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#autowireConstructor



使用无参构造方法创建对象

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#instantiateBean

1. 如果bean没有定义overrideMethod，直接使用无参构造方法创建实例，如果不存在无参构造方法，将会抛出异常
2. 如果bean定义了overrideMethod，将会使用cglib方式创建一个代理





## 属性注入

使用@Autowired，@Value，@Inject注入属性的原理



<img src="/开源框架/spring/.assert/bean创建流程/image-20220912213027809.png" alt="image-20220912213027809" style="zoom:50%;" />







### 发现需要注入的属性或者方法

在bean刚刚被实例化后，将会调用方法AutowiredAnnotationBeanPostProcessor#postProcessMergedBeanDefinition识别bean中需要注入的属性

```java
	public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
		if (beanType != null) {
			InjectionMetadata metadata = findAutowiringMetadata(beanName, beanType, null);
			metadata.checkConfigMembers(beanDefinition);
		}
	}
```



使用InjectionMetadata表示所有注入的元数据，InjectedElement表示一个注入属性或者方法

<img src="/开源框架/spring/.assert/bean创建流程/image-20220914215708749.png" alt="image-20220914215708749" style="zoom:50%;" />



AutowiredFieldElement表示类中所有非静态并且有@Autowired，@Value，@Inject注解

AutowiredMethodElement分别表示所有非静态并且有@Autowired，@Value，@Inject注解的方法。



识别方法时，需要区分bridge方法，桥接方法在子类覆盖父类或者接口的泛型方法时会出现







### 注入属性或者执行注入方法



AutowiredFieldElement

查找被注入的对象



@Value中指定的属性，会从value属性获取最终的值





## 事件监听Listener识别




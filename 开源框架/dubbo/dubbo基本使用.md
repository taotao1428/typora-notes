# dubbo基本使用

dubbo是一个高性能的rpc框架，由alibaba开源



## 架构



![dubbo-architucture](/开源框架/dubbo/.assert/dubbo基本使用/dubbo-architecture.jpg)

| 角色      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| Provider  | 服务提供者，提供服务给他消费者调用                           |
| Container | 服务提供者运行的容器                                         |
| Registry  | 注册中心。服务提供向注册中心注册自己提供的服务               |
| Consumer  | 服务消费者。服务消费者向注册中心订阅消费者信息，注册中心向服务消费者推送服务提供者的信息 |
| Monitor   | 监控。提供调用情况。                                         |





## 使用



### 依赖包

父级pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hewutao</groupId>
    <artifactId>dubbostudy</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <modules>
        <module>provider</module>
        <module>consumer</module>
        <module>api</module>
    </modules>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <version.dubbo>2.7.15</version.dubbo>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo-bom</artifactId>
                <version>${version.dubbo}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>1.7.36</version>
            </dependency>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-core</artifactId>
                <version>2.17.2</version>
            </dependency>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-api</artifactId>
                <version>2.17.2</version>
            </dependency>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-slf4j-impl</artifactId>
                <version>2.17.2</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.18.24</version>
            </dependency>

        </dependencies>
    </dependencyManagement>

</project>
```



dubbo提供者pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbostudy-provider</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <artifactId>dubbostudy</artifactId>
        <groupId>com.hewutao</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.hewutao</groupId>
            <artifactId>dubbostudy-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
      
        
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-common</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-registry-zookeeper</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-rpc-dubbo</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-cluster</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-remoting-netty4</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-remoting-zookeeper</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-config-spring</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-metadata-report-zookeeper</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-serialization-hessian2</artifactId>
        </dependency>
    </dependencies>

</project>
```



消费者pom.xml与提供者pom.xml除了artifactId为dubbostudy-consumer，其他一致。

### 创建公共接口

```java
package com.hewutao.dubbo;

public interface DemoService {
    String hello(String name);
}
```



### 搭建服务提供者

#### 创建实现

```java
package com.hewutao.dubbo.impl;

import com.hewutao.dubbo.DemoService;

public class DemoServiceImpl implements DemoService {
    @Override
    public String hello(String name) {
        if (name != null) {
            name = name.trim();
        }

        if (name == null || name.isEmpty()) {
            return "Hello Nobody!";
        }

        return "Hello " + name + "!";
    }
}

```



#### 创建spring配置

配置文件dubbo-provider.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="demo-provider" metadata-type="remote">
        <dubbo:parameter key="mapping-type" value="metadata"/>
    </dubbo:application>

    <dubbo:config-center address="zookeeper://192.168.3.69:2181"/>
    <dubbo:metadata-report address="zookeeper://192.168.3.69:2181"/>
    <dubbo:registry id="registry1" address="zookeeper://192.168.3.69:2181"/>



    <dubbo:protocol name="dubbo" port="20881" />

    <bean id="demoService" class="com.hewutao.dubbo.impl.DemoServiceImpl"/>
    <bean id="greetingService" class="com.hewutao.dubbo.impl.GreetServiceImpl"/>

    <dubbo:service interface="com.hewutao.dubbo.DemoService" timeout="3000" ref="demoService" registry="registry1"/>
</beans>
```



#### 创建应用

```java
package com.hewutao.dubbo;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.util.concurrent.TimeUnit;

@Slf4j
public class ProviderApp {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("dubbo-provider.xml");
        context.start();

        log.info("start finish");
        try {
            TimeUnit.DAYS.sleep(99999999);
        } catch (InterruptedException e) {
            log.info("start to close");
        }

        context.close();
    }
}

```





### 搭建服务消费者

#### 创建spring配置

dubbo-consumer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="demo-consumer" >
        <dubbo:parameter key="mapping-type" value="metadata"/>
        <dubbo:parameter key="enable-auto-migration" value="true"/>
        <dubbo:parameter key="qos.enable" value="false"/>
    </dubbo:application>


    <dubbo:metadata-report address="zookeeper://192.168.3.69:2181"/>

    <dubbo:registry address="zookeeper://192.168.3.69:2181"/>

    <dubbo:reference id="demoService" check="true"
                     interface="com.hewutao.dubbo.DemoService"/>

</beans>
```



#### 创建应用

```java
package com.hewutao.dubbo;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.support.ClassPathXmlApplicationContext;

@Slf4j
public class ConsumerApp {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("dubbo-consumer.xml");
        context.start();
        DemoService demoService = (DemoService) context.getBean("demoService");
        String result = demoService.hello("hwt");
        log.info(result);
        context.close();
    }
}
```



整体项目目录

<img src="/开源框架/dubbo/.assert/dubbo基本使用/image-20220612232124842.png" alt="image-20220612232124842" style="zoom:50%;" />



## 配置详解



### 不同配置的传递规则

![dubbo-config-override](/开源框架/dubbo/.assert/dubbo基本使用/dubbo-config-override.jpg)

每个参数从方法->接口->全局依次确认。同时调用者优先级大于提供者

### 配置项

https://dubbo.apache.org/zh/docs/v2.7/user/references/xml/dubbo-application/

 #### dubbo:application

对dubbo应用的配置。对应配置类`org.apache.dubbo.config.ApplicationConfig`



两个重要属性

1. name表示应用的名称
2. version表示应用的版本（可选）
3. compiler表示动态类生成方法。支持jdk和javassist，默认javassist



同时有些属性可以在内部的`dubbo:parameter`指定

```xml
    <dubbo:application name="demo-provider" metadata-type="remote">
        <dubbo:parameter key="mapping-type" value="metadata"/>
    </dubbo:application>
```





#### dubbo:registry

注册中心配置。对应的配置类： `org.apache.dubbo.config.RegistryConfig`。同时如果有多个不同的注册中心，可以声明多个 `<dubbo:registry>` 标签，并在 `<dubbo:service>` 或 `<dubbo:reference>` 的 `registry` 属性指定使用的注册中心。



#### dubbo:config-center 

配置中心。配置配置中心后，dubbo将会从配置中心拉取配置



#### dubbo:protocol

表示协议

| 参数          | 类型   | 说明                                                         | 默认                                                         |
| ------------- | ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| threadpool    | string | 线程池类型，可选：fixed/cached                               | 默认fixed                                                    |
| threads       | int    | 业务线程池大小，用于处理业务                                 | 默认200                                                      |
| iothreads     | int    | io线程数，用来处理网络io                                     | 默认cpu个数+1                                                |
| accepts       | int    | 服务提供方最大可接受连接数                                   | 默认0，表示没有限制                                          |
| payload       | int    | 请求及响应数据包大小限制，单位：字节                         | 默认8388608(=8M)                                             |
| codec         | string | 协议编码方式                                                 | 默认dubbo                                                    |
| serialization | string | 协议序列化方式，当协议支持多种序列化方式时使用，比如：dubbo协议的dubbo,hessian2,java,compactedjava，以及http协议的json等 | dubbo协议缺省为hessian2，rmi协议缺省为java，http协议缺省为json |
| accesslog     | string | 设为true，将向logger中输出访问日志，也可填写访问日志文件路径，直接把访问日志输出到指定文件 | false                                                        |
| transporter   | string | 协议的服务端和客户端实现类型，比如：dubbo协议的mina,netty等，可以分拆为server和client配置 | dubbo协议缺省为netty                                         |
| server        | string | dubbo协议缺省为netty，http协议缺省为servlet                  | 协议的服务器端实现类型，比如：dubbo协议的mina,netty等，http协议的jetty,servlet等 |
| client        | string | dubbo协议缺省为netty                                         | 协议的客户端实现类型，比如：dubbo协议的mina,netty等          |
| dispatcher    | string | 协议的消息派发方式，用于指定线程模型，比如：dubbo协议的all, direct, message, execution, connection等 | dubbo协议缺省为all                                           |
| buffer        | int    | 网络读写缓冲区大小                                           | 8192                                                         |
| heartbeat     | Int    | 心跳间隔，对于长连接，当物理层断开时，比如拔网线，TCP的FIN消息来不及发送，对方收不到断开事件，此时需要心跳来帮助检查连接是否已断开 | 60000ms                                                      |



#### dubbo:provider

| 属性 | 说明 |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |


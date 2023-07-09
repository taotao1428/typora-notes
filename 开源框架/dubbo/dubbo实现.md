# dubbo实现

## 整体框架图

![/dev-guide/images/dubbo-framework.jpg](/开源框架/dubbo/.assert/dubbo实现/dubbo-framework.jpg)



## 每层含义

| 层        | 说明                                                     |
| --------- | -------------------------------------------------------- |
| config    | 表示配置，对应dubbo的配置和启动                          |
| proxy     | 用于代理服务，消费者使用的对象就是通过代理产生的         |
| registry  | 表示服务注册与发现                                       |
| cluster   | 表示集群，集群概念仅存在消费者中，对应请求路由和负载均衡 |
| Monitor   | 表示服务监控                                             |
| Protocol  | 表示服务调用的协议，默认为dubbo，还有http，hession，rmi  |
| Exchange  | 表示数据交换层，对应客户端的request和服务端的reply       |
| Transport | 表示传输层，用于服务端与客户端通信                       |
| Serialize | 表示序列化层，用于序列化请求和响应中对象                 |



## Dubbo配置与启动

config层负责组装dubbo运行环境，当前的实现只有config-spring

配置主要分为

1. ApplicationConfig，应用配置
2. RegistryConfig，注册中心配置
3. ProviderConfig，服务提供者公用配置
4. ServiceConfig，服务配置
5. ConsumerConfig，消费者公用配置
6. ReferenceConfig消费者配置
7. ProtocolConfig传输协议配置
8. MonitorConfig监控配置
9. ConfigCenterConfig配置中心配置



spring的schema的handle机制。通过该机制，dubbo的handler可以处理配置文件中dubbo部分

![image-20220615214149173](/开源框架/dubbo/.assert/dubbo实现/image-20220615214149173.png)

org.apache.dubbo.config.spring.schema.DubboNamespaceHandler



注册额外的bean

```
org.apache.dubbo.config.spring.util.DubboBeanUtils#registerCommonBeans
```



dubbo启动

org.apache.dubbo.config.spring.context.DubboBootstrapApplicationListener



org.apache.dubbo.config.bootstrap.DubboBootstrap



### start方法启动



#### init

执行FrameworkExt实现类的initialize方法

1. ConfigManager和ServiceRepository，initialize没有具体实现

Environment

```
    @Override
    public void initialize() throws IllegalStateException {
        ConfigManager configManager = ApplicationModel.getConfigManager();
        Optional<Collection<ConfigCenterConfig>> defaultConfigs = configManager.getDefaultConfigCenter();
        defaultConfigs.ifPresent(configs -> {
            for (ConfigCenterConfig config : configs) {
                this.setExternalConfigMap(config.getExternalConfiguration());
                this.setAppExternalConfigMap(config.getAppExternalConfiguration());
            }
        });

        this.externalConfiguration.setProperties(externalConfigurationMap);
        this.appExternalConfiguration.setProperties(appExternalConfigurationMap);
    }
```





#### startConfigCenter（后面再看）

启动配置中心，并从远处配置中心拉取配置



#### loadRemoteConfigs

将远程配置中的registry和protocol放到configManager中



#### checkGlobalConfigs

检查配置是否有错误。另外会加载默认的MetadataConfig、Provider和Consumer配置



#### startMetadataCenter，initMetadataService

初始化元数据服务





### exportServices

将服务暴露给外部，拉起服务监听。向注册中心注册服务



## 插件机制

dubbo中插件机制，支持自定义插件，并且可以通过url中的参数动态引用插件

### 插件机制简介

#### 定义插件

1. 定义接口，定义需要动态加载实现类的接口。例如`org.apache.dubbo.rpc.Protocol`
2. 实现接口。例如实现类为`org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol`
3. 在META-INF/dubbo.internal文件夹中创建文件`org.apache.dubbo.rpc.Protocol`(文件名与接口名相同)，在文件中增加键值对`{实现类的key}={实现类的全类名}`，例如`dubbo=org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol`;然后通过实现类的key可以加载实现类。



#### 加载插件

通过ExtensionLoader.getExtensionLoader方法可以获取接口插件的的加载器，从而可以记载插件

```java
org.apache.dubbo.rpc.Protocol extension = ExtensionLoader.getExtensionLoader(org.apache.dubbo.rpc.Protocol.class).getExtension("dubbo");
```



ExtensionFactory

Common

```
adaptive=org.apache.dubbo.common.extension.factory.AdaptiveExtensionFactory
spi=org.apache.dubbo.common.extension.factory.SpiExtensionFactory
```

Config-spring

```
spring=org.apache.dubbo.config.spring.extension.SpringExtensionFactory
```





### Adaptive插件

在代码运行过程中，需要根据实际的运行参数确定该使用哪个插件。此时就需要一个Adaptive对象，它的功能类似与代理，可以被直接调用，然后根据参数，调用其他插件。



1. 如果实现类中存在@Adaptive注解，将默认使用该类作为adaptive类
2. 如果不存在，将会生成一个adaptive类，方法内部会根据方法参数url上的参数获取应该使用那个实现类，url上的参数可以通过在方法上添加@Adaptive注解指定

![image-20220616000331745](/开源框架/dubbo/.assert/dubbo实现/image-20220616000331745.png)



<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220616000519504.png" alt="image-20220616000519504" style="zoom:50%;" />





adaptive类生成方法：org.apache.dubbo.common.extension.AdaptiveClassCodeGenerator#generateExtNameAssignment



### wrapper机制

为了进一步丰富插件的功能，可以使用Wrapper机制对插件进行一层封装。封装后，真实的插件将会被注入到Wrapper中。

1. 如果实现类中包含有本类型参数的构造函数，将会认为它是wrapper类。
2. order越小越接近外面，越大，越接近被包裹的类



<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619154312109.png" alt="image-20220619154312109" style="zoom:50%;" />

其中Protocol接口就有两个wrapper类，ProtocolFilterWrapper和ProtocolListenerWrapper

```
extension = ExtensionLoader.getExtensionLoader(org.apache.dubbo.rpc.Protocol.class).getExtension("dubbo");
```

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619154848860.png" alt="image-20220619154848860" style="zoom:50%;" />

## 注册服务

每个服务有哪些参数？

它的url是什么样的？



1. 来自ApplicationConfig，包含该类中有@Parameter注解的方法

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619102145395.png" alt="image-20220619102145395" style="zoom:50%;" />

2. 来自RegistryConfig

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619102245618.png" alt="image-20220619102245618" style="zoom:50%;" />



3. path=org.apache.dubbo.registry.RegistryService
4. 运行时参数，包含dubbo版本，bubbo协议版本，进程id，当前时间

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619102622825.png" alt="image-20220619102622825" style="zoom:50%;" />

5. 从registryConfig中获取url



最终拼接成一个url

zookeeper://192.168.3.69:2181/org.apache.dubbo.registry.RegistryService?application=demo-provider&dubbo=2.0.2&id=registry1&mapping-type=metadata&mapping.type=metadata&metadata-type=remote&pid=29547&release=2.7.15&timestamp=1655605492659



由path，group，version唯一确定一个service，path为接口全量限定名。group和version可以为null。{group}/{path}:{version}



```java

// 获取invoker时，使用的是registry的地址，没有直接使用服务的地址，服务的地址通过参数export拼接在registry的地址上
Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass,
	registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));

DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

Exporter<?> exporter = PROTOCOL.export(wrapperInvoker);
exporters.add(exporter);
```



RegistryProtocol#export

dubbo://10.37.129.2:20881/com.hewutao.dubbo.DemoService?anyhost=true&application=demo-provider&bind.ip=10.37.129.2&bind.port=20881&deprecated=false&dubbo=2.0.2&dynamic=true&generic=false&interface=com.hewutao.dubbo.DemoService&mapping-type=metadata&mapping.type=metadata&metadata-type=remote&methods=hello&pid=30536&release=2.7.15&service.name=ServiceBean:/com.hewutao.dubbo.DemoService&side=provider&timeout=3000&timestamp=1655621868862

ProtocolFilterWrapper#export



```
if (arg0 == null) throw new IllegalArgumentException("org.apache.dubbo.rpc.Invoker argument == null");
if (arg0.getUrl() == null) throw new IllegalArgumentException("org.apache.dubbo.rpc.Invoker argument getUrl() == null");
org.apache.dubbo.common.URL url = arg0.getUrl();
String extName = ( url.getProtocol() == null ? "dubbo" : url.getProtocol() );
if(extName == null) throw new IllegalStateException("Failed to get extension (org.apache.dubbo.rpc.Protocol) name from url (" + url.toString() + ") use keys([protocol])");
org.apache.dubbo.rpc.Protocol extension = (org.apache.dubbo.rpc.Protocol)ExtensionLoader.getExtensionLoader(org.apache.dubbo.rpc.Protocol.class).getExtension(extName);
return extension.export(arg0);

```



<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619153639298.png" alt="image-20220619153639298" style="zoom:50%;" />



ProtocolFilterWrapper：给invoker添加filter

ListenerExporterWrapper：添加对ExporterListener的触发





注册服务到注册中心

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619160602073.png" alt="image-20220619160602073" style="zoom:50%;" />

注册中心地址

dubbo://10.37.129.2:20881/com.hewutao.dubbo.DemoService?anyhost=true&application=demo-provider&deprecated=false&dubbo=2.0.2&dynamic=true&generic=false&interface=com.hewutao.dubbo.DemoService&mapping-type=metadata&mapping.type=metadata&metadata-type=remote&methods=hello&pid=30825&release=2.7.15&service.name=ServiceBean:/com.hewutao.dubbo.DemoService&side=provider&timeout=3000&timestamp=1655625842256



服务地址

dubbo://10.37.129.2:20881/com.hewutao.dubbo.DemoService?anyhost=true&application=demo-provider&deprecated=false&dubbo=2.0.2&dynamic=true&generic=false&interface=com.hewutao.dubbo.DemoService&mapping-type=metadata&mapping.type=metadata&metadata-type=remote&methods=hello&pid=30825&release=2.7.15&service.name=ServiceBean:/com.hewutao.dubbo.DemoService&side=provider&timeout=3000&timestamp=1655625842256



注册到zookeeper节点上

/dubbo/com.hewutao.dubbo.DemoService/providers/dubbo%3A%2F%2F10.37.129.2%3A20881%2Fcom.hewutao.dubbo.DemoService%3Fanyhost%3Dtrue%26application%3Ddemo-provider%26deprecated%3Dfalse%26dubbo%3D2.0.2%26dynamic%3Dtrue%26generic%3Dfalse%26interface%3Dcom.hewutao.dubbo.DemoService%26mapping-type%3Dmetadata%26mapping.type%3Dmetadata%26metadata-type%3Dremote%26methods%3Dhello%26pid%3D30886%26release%3D2.7.15%26service.name%3DServiceBean%3A%2Fcom.hewutao.dubbo.DemoService%26side%3Dprovider%26timeout%3D3000%26timestamp%3D1655626721084



## 订阅服务

消费者，如何订阅服务？

消费者是通过什么查找想要的提供者？



org.apache.dubbo.config.ReferenceConfig，引用的配置是一个BeanFactory的实现类，通过代理实现方法的调用。get方法获取代理对象

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619162627050.png" alt="image-20220619162627050" style="zoom:50%;" />



registry://192.168.3.69:2181/org.apache.dubbo.registry.RegistryService?application=demo-consumer&dubbo=2.0.2&enable-auto-migration=true&enable.auto.migration=true&id=org.apache.dubbo.config.RegistryConfig&mapping-type=metadata&mapping.type=metadata&pid=30911&qos.enable=false&refer=application%3Ddemo-consumer%26check%3Dtrue%26dubbo%3D2.0.2%26enable-auto-migration%3Dtrue%26enable.auto.migration%3Dtrue%26init%3Dfalse%26interface%3Dcom.hewutao.dubbo.DemoService%26mapping-type%3Dmetadata%26mapping.type%3Dmetadata%26metadata-type%3Dremote%26methods%3Dhello%26pid%3D30911%26qos.enable%3Dfalse%26register.ip%3D10.37.129.2%26release%3D2.7.15%26side%3Dconsumer%26sticky%3Dfalse%26timestamp%3D1655627081986&registry=zookeeper&release=2.7.15&timestamp=1655627256896







cluster的实现

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619163607777.png" alt="image-20220619163607777" style="zoom:50%;" />

consumer://10.37.129.2/com.hewutao.dubbo.DemoService?application=demo-consumer&check=true&dubbo=2.0.2&enable-auto-migration=true&enable.auto.migration=true&init=false&interface=com.hewutao.dubbo.DemoService&mapping-type=metadata&mapping.type=metadata&metadata-type=remote&methods=hello&pid=30911&qos.enable=false&release=2.7.15&side=consumer&sticky=false&timestamp=1655627081986



<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619164144550.png" alt="image-20220619164144550" style="zoom:50%;" />





org.apache.dubbo.registry.integration.RegistryDirectory会监听注册中心的事件



RegisteryDirectory会将provider的url生成对应的invoker，invoker中会包含filter



Directory中包含一个RouteChain对象，用于路由请求

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220619165456838.png" alt="image-20220619165456838" style="zoom:50%;" />



## filter机制

dubbo中配置的filter是由ProtocolFilterWrapper注入到调用链上，主要使用通过filter将invoker包装起来



1. 使用`@Activate`注解注释的filter，会自动加载，会包含在default中
2. 在配置filter时，可以使用`-<name>`去除某个filter，也可以`-default`中默认去除所有自动加载的filter。



指定filter分组

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220627220303532.png" alt="image-20220627220303532" style="zoom:50%;" />

```
filter1,filter2 = default,filter1,filter2
filter1,default,filter2  filter1将会在默认加载的filter之前
-default,filter1,filter2
-filter1,filter2

```





## loadbalance机制





## 知识点

### 序列化

DubboProtocol中的handler处理Invocation类型，其他类型会忽略



HeaderExchanger会把handler包装，

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220620084717815.png" alt="image-20220620084717815" style="zoom:50%;" />



序列化接口

```java
@SPI("hessian2")
public interface Serialization {
    // 获取内容类型id
    byte getContentTypeId();
  
    String getContentType();

    // 序列化对象
    @Adaptive
    ObjectOutput serialize(URL url, OutputStream output) throws IOException;

    // 反序列对象
    @Adaptive
    ObjectInput deserialize(URL url, InputStream input) throws IOException;

}
```



```java
public interface DataOutput {
    void writeBool(boolean v) throws IOException;
    void writeByte(byte v) throws IOException;
    void writeShort(short v) throws IOException;
    void writeInt(int v) throws IOException;
    void writeLong(long v) throws IOException;
    void writeFloat(float v) throws IOException;
    void writeDouble(double v) throws IOException;
    void writeUTF(String v) throws IOException;
    void writeBytes(byte[] v) throws IOException;
    void writeBytes(byte[] v, int off, int len) throws IOException;
    void flushBuffer() throws IOException;
}

public interface ObjectOutput extends DataOutput {
    void writeObject(Object obj) throws IOException;
    default void writeThrowable(Object obj) throws IOException {
        writeObject(obj);
    }

    default void writeEvent(Object data) throws IOException {
        writeObject(data);
    }

    default void writeAttachments(Map<String, Object> attachments) throws IOException {
        writeObject(attachments);
    }

}
```



```java
public interface DataInput {
    boolean readBool() throws IOException;
    byte readByte() throws IOException;
    short readShort() throws IOException;
    int readInt() throws IOException;
    long readLong() throws IOException;
    float readFloat() throws IOException;
    double readDouble() throws IOException;
    String readUTF() throws IOException;
    byte[] readBytes() throws IOException;
}

public interface ObjectInput extends DataInput {
    @Deprecated
    Object readObject() throws IOException, ClassNotFoundException;
    <T> T readObject(Class<T> cls) throws IOException, ClassNotFoundException;
    <T> T readObject(Class<T> cls, Type type) throws IOException, ClassNotFoundException;
    default Throwable readThrowable() throws IOException, ClassNotFoundException {
        Object obj = readObject();
        if (!(obj instanceof Throwable)) {
            throw new IOException("Response data error, expect Throwable, but get " + obj);
        }
        return (Throwable) obj;
    }

    default Object readEvent() throws IOException, ClassNotFoundException {
        return readObject();
    }

    default Map<String, Object> readAttachments() throws IOException, ClassNotFoundException {
        return readObject(Map.class);
    }
}
```





### dubbo协议

#### 序列化Request

##### header

1. 2字节magic，0xdabb
2. 1字节Serialization的version，1000 0000(request表示位) | contentTypeId。contentTypeId不超过5位，如果请求是twoWay，第7位置为1。如果是event，第6位置为1
3. 第5~12个字节，表示requestId，8个字节
4. 第13~16个字节，表示请求体长度

##### body

1. dubbo协议版本号，utf-8编码字符串
2. serviceName服务名称，接口的全限定名，例如com.hewutao.DemoService
3. Version服务版本号，例如0.1.0
4. methodName方法名称，例如hello
5. ParameterTypesDesc方法参数描述符，Ljava/lang/String;
6. 参数对象
7. Attachments，一个Map<String, Object>对象

<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220622232123564.png" alt="image-20220622232123564" style="zoom:50%;" />



#### 序列化Response









### 服务提供者和消费者，线程池管理



```java
public class NettyTransporter implements Transporter {

    public static final String NAME = "netty";

    @Override
    public RemotingServer bind(URL url, ChannelHandler handler) throws RemotingException {
        return new NettyServer(url, handler);
    }

    @Override
    public Client connect(URL url, ChannelHandler handler) throws RemotingException {
        return new NettyClient(url, handler);
    }

}
```



对于业务线程

1. 所有consumer共用一个业务线程池
2. 所有provider根据protocol共用业务线程池





#### server处理

设置io线程数，iothreads，默认线程数`Math.min(Runtime.getRuntime().availableProcessors() + 1, 32)`



<img src="/开源框架/dubbo/.assert/dubbo实现/image-20220626204544170.png" alt="image-20220626204544170" style="zoom:50%;" />




















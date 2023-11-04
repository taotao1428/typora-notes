# mysql网络问题报错

## 记录io发送数据时间

发送数据包时，如果发送成功，会记录最新的发送成功时间

com.mysql.jdbc.MysqlIO#send

<img src="/开源框架/mysql/.assert/mysql协议/image-20231104151439157.png" alt="image-20231104151439157" style="zoom:50%;" />



## 记录io读取数据时间

读取数据包时，如果读取成功，会记录最新读取成功时间

com.mysql.jdbc.MysqlIO#readPacket

<img src="/开源框架/mysql/.assert/mysql协议/image-20231104151626115.png" alt="image-20231104151626115" style="zoom:50%;" />



## 网络异常处理

网络io异常

com.mysql.jdbc.exceptions.jdbc4.CommunicationsException

<img src="/开源框架/mysql/.assert/mysql协议/image-20231104151801457.png" alt="image-20231104151801457" style="zoom:50%;" />



com.mysql.jdbc.SQLError#createLinkFailureMessageBasedOnHeuristics

io报错可能情况

1. 服务端配置了interactive_timeout，wait_timeout参数，默认都是28800s。默认jdbc是使用wait_timeout参数。如果连接写入数据的间隔超过这个时间，服务端会关闭连接，此时jdbc的报错会针对这种情况给出错误提示。

<img src="/开源框架/mysql/.assert/mysql协议/image-20231104153318248.png" alt="image-20231104153318248" style="zoom:50%;" />

2. 如果无法读取服务端的配置，jdbc驱动将会使用28800s与写入间隔对比，如果超过，也会有对应的错误提示。

   <img src="/开源框架/mysql/.assert/mysql协议/image-20231104153456730.png" alt="image-20231104153456730" style="zoom:50%;" />

3. 其他情况，将会直接报错Communications link failure错误



## 连接相关的参数

com.mysql.jdbc.ConnectionPropertiesImpl

1. autoReconnect=false。在执行sql前时，并且在开启autoCommit场景下，会触发重连接。com.mysql.jdbc.ConnectionImpl#execSQL

   <img src="/开源框架/mysql/.assert/mysql协议/image-20231104161413694.png" alt="image-20231104161413694" style="zoom:50%;" />

2. autoReconnectForPools=false。在修改连接自动提交时才可以触发重连，执行真正业务sql，并不会触发重连。com.mysql.jdbc.ConnectionImpl#setAutoCommit

3. connectTimeout=0，与数据库连接socket时，会使用。this.rawSocket.connect(sockAddr, getRealTimeout(connectTimeout));

4. socketTimeout=0，会将这个参数设置到socket。当从socket读取数据时使用。this.mysqlConnection.setSoTimeout(socketTimeout);




# netty基础用法



## 服务端



## 客户端



## 内存



## handler



### decoder

在解码数据的时候，会遇到粘包和拆包的问题，因此需要合并之前的数据一起解码。netty提供了ByteToMessageDecoder让我们使用。



ByteToMessageDecoder提供了两种合并数据流的方式，一种是使用merge的方式，每次获取新数据时，将新数据通过拷贝的方式添加到一个大buffer中，然后将新数据释放。另一种是通过composite方式，每次获取新数据时，将新数据直接添加到一个compositeBuffer中。



在使用decode方式时，只需要将从buffer中读取数据即可，不需要处理数据回收问题。



### encoder

encoder是将对象编码成字节，其中主要注重写入数据的Promise。多个数据的Promise可以合并成一个，参考io.netty.handler.codec.MessageToMessageEncoder#write





### sslHandler



## NioEventLoopGroup和NioEventLoop





NioEventLoop处理事件的方法

1. 对于OP_CONNECT连接事件，调用unsafe.finishConnection()
2. 对于OP_WRITE事件，调用unsafe.forceFlush()
3. 对于OP_READ和OP_ACCEPT事件，或者没有任何事件（针对jdk的bug），调用unsafe.read()

```java
# io.netty.channel.nio.NioEventLoop#processSelectedKey(java.nio.channels.SelectionKey, io.netty.channel.nio.AbstractNioChannel)

      private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
        final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
        if (!k.isValid()) {
            final EventLoop eventLoop;
            try {
                eventLoop = ch.eventLoop();
            } catch (Throwable ignored) {
                // If the channel implementation throws an exception because there is no event loop, we ignore this
                // because we are only trying to determine if ch is registered to this event loop and thus has authority
                // to close ch.
                return;
            }
            // Only close ch if ch is still registered to this EventLoop. ch could have deregistered from the event loop
            // and thus the SelectionKey could be cancelled as part of the deregistration process, but the channel is
            // still healthy and should not be closed.
            // See https://github.com/netty/netty/issues/5125
            if (eventLoop == this) {
                // close the channel if the key is not valid anymore
                unsafe.close(unsafe.voidPromise());
            }
            return;
        }

        try {
            int readyOps = k.readyOps();
            // We first need to call finishConnect() before try to trigger a read(...) or write(...) as otherwise
            // the NIO JDK channel implementation may throw a NotYetConnectedException.
            if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
                // remove OP_CONNECT as otherwise Selector.select(..) will always return without blocking
                // See https://github.com/netty/netty/issues/924
                int ops = k.interestOps();
                ops &= ~SelectionKey.OP_CONNECT;
                k.interestOps(ops);

                unsafe.finishConnect();
            }

            // Process OP_WRITE first as we may be able to write some queued buffers and so free memory.
            if ((readyOps & SelectionKey.OP_WRITE) != 0) {
                // Call forceFlush which will also take care of clear the OP_WRITE once there is nothing left to write
                ch.unsafe().forceFlush();
            }

            // Also check for readOps of 0 to workaround possible JDK bug which may otherwise lead
            // to a spin loop
            if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
                unsafe.read();
            }
        } catch (CancelledKeyException ignored) {
            unsafe.close(unsafe.voidPromise());
        }
    }
```



### 写入数据

输出的数据保存在AbstractChannel.AbstractUnsafe#outboundBuffer中

NioSocketChannel#doWrite

1. 每次写入循环的次数由WriteSpinCount参数决定
2. 每个循环写入的数据量由MaxBytesPerGatheringWrite参数决定。由于部分操作系统会自动调整SndBuf的大小，为了适配这种情况，netty也会根据每次发送的情况判断MaxBytesPerGatheringWrite大小
3. 如果所有数据已经写完了，将不会再监听write事件
4. 



通知上层不能写了

ChannelInboundHandler#channelWritabilityChanged



ChannelOutboundBuffer

负责保存需要发送的数据，每个数据对象会对应一个Promise，当数据被写入完成后，可以接收到回调。当消息通过write方法写入，在执行flush方法之前，消息是可以通过Promise的cancel方法取消发送，但是一旦执行flush方法，所有的数据会被修改为flushed状态，此时消息是不可以被撤回。





如果写入异常

如果是IOException并且autoclose为ture，将会自动关闭channel

否则，将只会关闭输出流







### 读取数据

AbstractNioByteChannel.NioByteUnsafe#read



读取数据时，使用RecvByteBufAllocator进行分配读取buffer。



每次读取都会将产生**read**事件，当读取完成后，将会触发readComplete事件



如果读取的数据量为-1，说明输入流被关闭，此时，如果允许单通道，将会触发ChannelInputShutdownEvent.INSTANCE事件。否者将会关闭通道。



如果读取异常，将会触发**readComplete**事件和调用ExceptionCaught方法



如果异常为OutOfMemory或者IOException，将会关闭输入通道或者关闭整个通道





### 管道

io.netty.channel.ChannelPipeline

管道负责所有io事件的传递，基础实现类为DefaultChannelPipeline



每个Context可以指定其运行的Executor



<img src="/开源框架/netty/.assert/netty基础用法/image-20220717231159267.png" alt="image-20220717231159267" style="zoom:50%;" />







## 客户端类型



### 一对一

一个线程使用一个channel



### 多对一

多个线程使用一个channel



## tcp配置

在Bootstrap#option方法可以添加

| 参数         | 说明                                                         | 默认 |
| ------------ | ------------------------------------------------------------ | ---- |
| SO_KEEPALIVE | 是否保持连接，true或者false                                  |      |
| SO_SNDBUF    | tcp的发送buf大小                                             |      |
| SO_RCVBUF    | tcp的接收buf大小                                             |      |
| SO_REUSEADDR | 是否重新使用端口                                             |      |
| SO_LINGER    | 关闭连接的等待时间                                           |      |
| SO_BACKLOG   | 服务端的等待最大值，不包括已经accept的连接                   |      |
| SO_TIMEOUT   | 超时时间，如果长时间没有接受到数据，将会端口连接，阻塞io时有用 |      |
| TCP_NODELAY  | 是否关闭延时发送                                             |      |



| 参数                               | 说明                                                         | 默认                                               |
| ---------------------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| ALLOCATOR                          | netty在运行时内存分配器，netty内部使用的内存都来自于它       | ByteBufUtil#DEFAULT_ALLOCATOR                      |
| RCVBUF_ALLOCATOR                   | 用于在接收数据时分配内存                                     | AdaptiveRecvByteBufAllocator                       |
| MESSAGE_SIZE_ESTIMATOR             | 消息大小估计器，在写入数据的时候使用它估计数据的大小，再结合WRITE_BUFFER_WATER_MARK最大值控制可写状态 | DefaultMessageSizeEstimator#DEFAULT                |
| CONNECT_TIMEOUT_MILLIS             | 连接超时，客户端超时时间                                     | 30000ms                                            |
| MAX_MESSAGES_PER_WRITE             | 每一次写入数据的最大值，如果达到，将会停止写入，去处理其他事件（stcp） | Intager.max_value                                  |
| WRITE_SPIN_COUNT                   | 每次写入数据循环的次数，每写入一个消息算一个循环             | 16                                                 |
| WRITE_BUFFER_WATER_MARK            | write buf的水位线，用于控制是否能写入                        | WriteBufferWaterMark#DEFAULT，低水位32k，高水位64k |
| ALLOW_HALF_CLOSURE                 | 允许半关闭                                                   |                                                    |
| AUTO_READ                          | 自动开始读取数据                                             |                                                    |
| AUTO_CLOSE                         | 在写入数据异常时，是否自动关闭连接                           | true                                               |
| **SINGLE_EVENTEXECUTOR_PER_GROUP** | 添加handler时，可以指定executorgroup，是否每个group只使用其中一个executor | true                                               |






# Netty 源码解析 - 完整教程

> 深入理解 Netty 的 Reactor 模型和高性能网络编程
> 
> @author erik.zhou

## 📚 技术概述

| 项目 | 说明 |
|------|------|
| **框架名称** | Netty |
| **当前版本** | 4.1.x |
| **源码地址** | https://github.com/netty/netty |
| **学习难度** | ⭐⭐⭐⭐⭐ |
| **重要程度** | ⭐⭐⭐⭐ |
| **预计时长** | 35-50 小时 |
| **前置知识** | Java NIO、多线程、网络编程 |

## 🎯 学习目标

- [ ] 理解 Netty 的整体架构
- [ ] 掌握 Reactor 线程模型
- [ ] 深入理解 EventLoop 事件循环
- [ ] 掌握 Channel 和 Pipeline 机制
- [ ] 理解 ByteBuf 内存管理
- [ ] 掌握编解码器的实现
- [ ] 能够使用 Netty 开发高性能网络应用

## 📖 目录

1. [Netty 整体架构](#1-netty-整体架构)
2. [Reactor 线程模型](#2-reactor-线程模型)
3. [EventLoop 事件循环](#3-eventloop-事件循环)
4. [Channel 和 Pipeline](#4-channel-和-pipeline)
5. [ByteBuf 内存管理](#5-bytebuf-内存管理)
6. [编解码器实现](#6-编解码器实现)

---

## 1. Netty 整体架构

### 1.1 核心组件

```
netty/
├── Bootstrap/ServerBootstrap  # 启动引导类
├── EventLoopGroup             # 事件循环组
├── EventLoop                  # 事件循环
├── Channel                    # 网络通道
├── ChannelPipeline            # 处理器链
├── ChannelHandler             # 处理器
├── ByteBuf                    # 字节缓冲区
└── ChannelFuture              # 异步结果
```

### 1.2 Netty 服务端示例

```java
public class NettyServer {
    public static void main(String[] args) throws Exception {
        // 1. 创建 BossGroup 和 WorkerGroup
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        
        try {
            // 2. 创建服务端启动引导类
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG, 128)
                .childOption(ChannelOption.SO_KEEPALIVE, true)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) {
                        ChannelPipeline pipeline = ch.pipeline();
                        pipeline.addLast(new StringDecoder());
                        pipeline.addLast(new StringEncoder());
                        pipeline.addLast(new ServerHandler());
                    }
                });
            
            // 3. 绑定端口，启动服务
            ChannelFuture future = bootstrap.bind(8080).sync();
            System.out.println("服务器启动成功，监听端口：8080");
            
            // 4. 等待服务端监听端口关闭
            future.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

// 自定义处理器
class ServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        System.out.println("收到消息：" + msg);
        ctx.writeAndFlush("服务器收到：" + msg);
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

---

## 2. Reactor 线程模型 🔥

### 2.1 三种 Reactor 模型

**单 Reactor 单线程**
```
Reactor (单线程)
  ↓
Accept + Read + Decode + Process + Encode + Send
```
- 优点：简单
- 缺点：性能差，无法利用多核

**单 Reactor 多线程**
```
Reactor (单线程)
  ↓
Accept + Read + Decode
  ↓
Thread Pool (Process)
  ↓
Encode + Send
```
- 优点：充分利用多核
- 缺点：Reactor 单线程成为瓶颈

**主从 Reactor 多线程（Netty 使用）** 🔥
```
Main Reactor (BossGroup)
  ↓
Accept
  ↓
Sub Reactor (WorkerGroup)
  ↓
Read + Decode + Process + Encode + Send
```
- 优点：高性能、高并发
- 缺点：实现复杂

### 2.2 Netty 的 Reactor 实现

```java
// 主从 Reactor 模型
EventLoopGroup bossGroup = new NioEventLoopGroup(1);      // 主 Reactor
EventLoopGroup workerGroup = new NioEventLoopGroup(8);    // 从 Reactor

ServerBootstrap bootstrap = new ServerBootstrap();
bootstrap.group(bossGroup, workerGroup)  // 设置主从 Reactor
    .channel(NioServerSocketChannel.class);

// BossGroup 职责：
// 1. 接收客户端连接（Accept）
// 2. 将连接注册到 WorkerGroup

// WorkerGroup 职责：
// 1. 处理 I/O 读写
// 2. 执行业务逻辑
// 3. 编解码
```

### 2.3 EventLoopGroup 创建

```java
// NioEventLoopGroup 构造器
public NioEventLoopGroup(int nThreads) {
    this(nThreads, (Executor) null);
}

public NioEventLoopGroup(int nThreads, Executor executor) {
    this(nThreads, executor, SelectorProvider.provider());
}

public NioEventLoopGroup(int nThreads, Executor executor, 
        final SelectorProvider selectorProvider) {
    this(nThreads, executor, selectorProvider, 
        DefaultSelectStrategyFactory.INSTANCE);
}

// 最终调用父类构造器
protected MultithreadEventLoopGroup(int nThreads, Executor executor, 
        Object... args) {
    super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, 
        executor, args);
}

// 创建 EventLoop 数组
protected MultithreadEventExecutorGroup(int nThreads, 
        Executor executor, Object... args) {
    // 创建 EventLoop 数组
    children = new EventExecutor[nThreads];
    
    for (int i = 0; i < nThreads; i++) {
        boolean success = false;
        try {
            // 创建 NioEventLoop
            children[i] = newChild(executor, args);
            success = true;
        } catch (Exception e) {
            throw new IllegalStateException("failed to create a child event loop", e);
        } finally {
            if (!success) {
                // 创建失败，关闭已创建的 EventLoop
                for (int j = 0; j < i; j++) {
                    children[j].shutdownGracefully();
                }
            }
        }
    }
    
    // 创建 EventLoop 选择器（轮询策略）
    chooser = chooserFactory.newChooser(children);
}
```

---

## 3. EventLoop 事件循环 🔥

### 3.1 EventLoop 核心概念

```java
// EventLoop 继承关系
EventLoop extends EventLoopGroup, EventExecutor

// EventLoop 特点：
// 1. 一个 EventLoop 对应一个线程
// 2. 一个 EventLoop 可以处理多个 Channel
// 3. 一个 Channel 只能注册到一个 EventLoop
// 4. EventLoop 负责处理 I/O 事件和任务
```

### 3.2 EventLoop 运行流程

```java
// NioEventLoop.run() - 事件循环主逻辑
@Override
protected void run() {
    int selectCnt = 0;
    for (;;) {
        try {
            int strategy;
            try {
                // 1. 选择策略（是否需要 select）
                strategy = selectStrategy.calculateStrategy(selectNowSupplier, hasTasks());
                switch (strategy) {
                case SelectStrategy.CONTINUE:
                    continue;
                case SelectStrategy.BUSY_WAIT:
                    // NIO 不支持 busy-wait
                case SelectStrategy.SELECT:
                    // 执行 select 操作
                    long curDeadlineNanos = nextScheduledTaskDeadlineNanos();
                    if (curDeadlineNanos == -1L) {
                        curDeadlineNanos = NONE;
                    }
                    nextWakeupNanos.set(curDeadlineNanos);
                    try {
                        if (!hasTasks()) {
                            strategy = select(curDeadlineNanos);
                        }
                    } finally {
                        nextWakeupNanos.lazySet(AWAKE);
                    }
                default:
                }
            } catch (IOException e) {
                rebuildSelector0();
                selectCnt = 0;
                handleLoopException(e);
                continue;
            }
            
            selectCnt++;
            cancelledKeys = 0;
            needsToSelectAgain = false;
            final int ioRatio = this.ioRatio;
            boolean ranTasks;
            if (ioRatio == 100) {
                try {
                    if (strategy > 0) {
                        // 2. 处理 I/O 事件
                        processSelectedKeys();
                    }
                } finally {
                    // 3. 执行所有任务
                    ranTasks = runAllTasks();
                }
            } else if (strategy > 0) {
                final long ioStartTime = System.nanoTime();
                try {
                    // 2. 处理 I/O 事件
                    processSelectedKeys();
                } finally {
                    // 3. 根据 ioRatio 执行任务
                    final long ioTime = System.nanoTime() - ioStartTime;
                    ranTasks = runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
                }
            } else {
                // 3. 执行最少数量的任务
                ranTasks = runAllTasks(0);
            }
            
            // 4. 检查是否需要重新 select
            if (ranTasks || strategy > 0) {
                if (selectCnt > MIN_PREMATURE_SELECTOR_RETURNS && logger.isDebugEnabled()) {
                    logger.debug("Selector.select() returned prematurely {} times in a row for Selector {}.",
                            selectCnt - 1, selector);
                }
                selectCnt = 0;
            } else if (unexpectedSelectorWakeup(selectCnt)) {
                selectCnt = 0;
            }
        } catch (CancelledKeyException e) {
            // Harmless exception - log anyway
            if (logger.isDebugEnabled()) {
                logger.debug(CancelledKeyException.class.getSimpleName() + " raised by a Selector {} - JDK bug?",
                        selector, e);
            }
        } catch (Error e) {
            throw e;
        } catch (Throwable t) {
            handleLoopException(t);
        } finally {
            // Always handle shutdown even if the loop processing threw an exception.
            try {
                if (isShuttingDown()) {
                    closeAll();
                    if (confirmShutdown()) {
                        return;
                    }
                }
            } catch (Error e) {
                throw e;
            } catch (Throwable t) {
                handleLoopException(t);
            }
        }
    }
}
```

### 3.3 处理 I/O 事件

```java
// 处理选中的 Key
private void processSelectedKeys() {
    if (selectedKeys != null) {
        // 优化的 SelectedKeys 处理
        processSelectedKeysOptimized();
    } else {
        // 普通的 SelectedKeys 处理
        processSelectedKeysPlain(selector.selectedKeys());
    }
}

// 优化的处理方式
private void processSelectedKeysOptimized() {
    for (int i = 0; i < selectedKeys.size; ++i) {
        final SelectionKey k = selectedKeys.keys[i];
        selectedKeys.keys[i] = null;
        
        final Object a = k.attachment();
        
        if (a instanceof AbstractNioChannel) {
            // 处理 Channel 的 I/O 事件
            processSelectedKey(k, (AbstractNioChannel) a);
        } else {
            @SuppressWarnings("unchecked")
            NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
            processSelectedKey(k, task);
        }
        
        if (needsToSelectAgain) {
            selectedKeys.reset(i + 1);
            selectAgain();
            i = -1;
        }
    }
}

// 处理单个 Channel 的事件
private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
    
    try {
        int readyOps = k.readyOps();
        
        // 处理 OP_CONNECT 事件
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            int ops = k.interestOps();
            ops &= ~SelectionKey.OP_CONNECT;
            k.interestOps(ops);
            unsafe.finishConnect();
        }
        
        // 处理 OP_WRITE 事件
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            ch.unsafe().forceFlush();
        }
        
        // 处理 OP_READ 或 OP_ACCEPT 事件
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            unsafe.read();
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise());
    }
}
```

---

## 4. Channel 和 Pipeline 🔥

### 4.1 Channel 核心概念

```java
// Channel 接口
public interface Channel extends AttributeMap, ChannelOutboundInvoker, Comparable<Channel> {
    EventLoop eventLoop();              // 获取 EventLoop
    ChannelPipeline pipeline();         // 获取 Pipeline
    ChannelConfig config();             // 获取配置
    boolean isOpen();                   // 是否打开
    boolean isActive();                 // 是否激活
    ChannelMetadata metadata();         // 获取元数据
    SocketAddress localAddress();       // 本地地址
    SocketAddress remoteAddress();      // 远程地址
}

// Channel 生命周期
channelRegistered    → Channel 注册到 EventLoop
channelActive        → Channel 激活（连接建立）
channelRead          → Channel 读取数据
channelReadComplete  → Channel 读取完成
channelInactive      → Channel 失活（连接断开）
channelUnregistered  → Channel 从 EventLoop 注销
```


### 4.2 ChannelPipeline 处理器链

```java
// ChannelPipeline 接口
public interface ChannelPipeline extends ChannelInboundInvoker, ChannelOutboundInvoker, Iterable<Entry<String, ChannelHandler>> {
    // 添加处理器
    ChannelPipeline addFirst(String name, ChannelHandler handler);
    ChannelPipeline addLast(String name, ChannelHandler handler);
    ChannelPipeline addBefore(String baseName, String name, ChannelHandler handler);
    ChannelPipeline addAfter(String baseName, String name, ChannelHandler handler);
    
    // 移除处理器
    ChannelPipeline remove(ChannelHandler handler);
    ChannelHandler remove(String name);
    
    // 获取处理器
    ChannelHandler get(String name);
    <T extends ChannelHandler> T get(Class<T> handlerType);
}

// DefaultChannelPipeline 实现
public class DefaultChannelPipeline implements ChannelPipeline {
    
    final AbstractChannelHandlerContext head;
    final AbstractChannelHandlerContext tail;
    
    protected DefaultChannelPipeline(Channel channel) {
        this.channel = ObjectUtil.checkNotNull(channel, "channel");
        
        tail = new TailContext(this);
        head = new HeadContext(this);
        
        head.next = tail;
        tail.prev = head;
    }
    
    @Override
    public final ChannelPipeline addLast(String name, ChannelHandler handler) {
        return addLast(null, name, handler);
    }
    
    @Override
    public final ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler) {
        final AbstractChannelHandlerContext newCtx;
        synchronized (this) {
            checkMultiplicity(handler);
            
            newCtx = newContext(group, filterName(name, handler), handler);
            
            addLast0(newCtx);
            
            if (!registered) {
                newCtx.setAddPending();
                callHandlerCallbackLater(newCtx, true);
                return this;
            }
            
            EventExecutor executor = newCtx.executor();
            if (!executor.inEventLoop()) {
                callHandlerAddedInEventLoop(newCtx, executor);
                return this;
            }
        }
        callHandlerAdded0(newCtx);
        return this;
    }
    
    private void addLast0(AbstractChannelHandlerContext newCtx) {
        AbstractChannelHandlerContext prev = tail.prev;
        newCtx.prev = prev;
        newCtx.next = tail;
        prev.next = newCtx;
        tail.prev = newCtx;
    }
}
```

### 4.3 事件传播机制

```java
// Inbound 事件传播（从 head 到 tail）
public interface ChannelInboundHandler extends ChannelHandler {
    void channelRegistered(ChannelHandlerContext ctx) throws Exception;
    void channelUnregistered(ChannelHandlerContext ctx) throws Exception;
    void channelActive(ChannelHandlerContext ctx) throws Exception;
    void channelInactive(ChannelHandlerContext ctx) throws Exception;
    void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception;
    void channelReadComplete(ChannelHandlerContext ctx) throws Exception;
    void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception;
    void channelWritabilityChanged(ChannelHandlerContext ctx) throws Exception;
    void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception;
}

// Outbound 事件传播（从 tail 到 head）
public interface ChannelOutboundHandler extends ChannelHandler {
    void bind(ChannelHandlerContext ctx, SocketAddress localAddress, ChannelPromise promise) throws Exception;
    void connect(ChannelHandlerContext ctx, SocketAddress remoteAddress, 
        SocketAddress localAddress, ChannelPromise promise) throws Exception;
    void disconnect(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
    void close(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
    void deregister(ChannelHandlerContext ctx, ChannelPromise promise) throws Exception;
    void read(ChannelHandlerContext ctx) throws Exception;
    void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception;
    void flush(ChannelHandlerContext ctx) throws Exception;
}

// 事件传播示例
public class MyInboundHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("MyInboundHandler: " + msg);
        // 传递给下一个 Handler
        ctx.fireChannelRead(msg);
    }
}

public class MyOutboundHandler extends ChannelOutboundHandlerAdapter {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
        System.out.println("MyOutboundHandler: " + msg);
        // 传递给下一个 Handler
        ctx.write(msg, promise);
    }
}
```

---

## 5. ByteBuf 内存管理 🔥

### 5.1 ByteBuf 核心概念

```java
// ByteBuf 结构
/**
 * +-------------------+------------------+------------------+
 * | discardable bytes |  readable bytes  |  writable bytes  |
 * |                   |     (CONTENT)    |                  |
 * +-------------------+------------------+------------------+
 * |                   |                  |                  |
 * 0      <=      readerIndex   <=   writerIndex    <=    capacity
 * 
 * @author erik.zhou
 */
public abstract class ByteBuf implements ReferenceCounted, Comparable<ByteBuf> {
    // 读写索引
    public abstract int readerIndex();
    public abstract ByteBuf readerIndex(int readerIndex);
    public abstract int writerIndex();
    public abstract ByteBuf writerIndex(int writerIndex);
    
    // 容量
    public abstract int capacity();
    public abstract ByteBuf capacity(int newCapacity);
    public abstract int maxCapacity();
    
    // 可读/可写字节数
    public abstract int readableBytes();
    public abstract int writableBytes();
    public abstract int maxWritableBytes();
    
    // 读写操作
    public abstract byte readByte();
    public abstract ByteBuf writeByte(int value);
    public abstract ByteBuf readBytes(byte[] dst);
    public abstract ByteBuf writeBytes(byte[] src);
    
    // 引用计数
    public abstract int refCnt();
    public abstract ByteBuf retain();
    public abstract boolean release();
}
```

### 5.2 ByteBuf 分类

```java
/**
 * 1. Heap ByteBuf - 堆内存
 * 
 * @author erik.zhou
 */
ByteBuf heapBuf = Unpooled.buffer(256);
if (heapBuf.hasArray()) {
    byte[] array = heapBuf.array();
    int offset = heapBuf.arrayOffset() + heapBuf.readerIndex();
    int length = heapBuf.readableBytes();
    // 处理数组
}

/**
 * 2. Direct ByteBuf - 直接内存
 * 
 * @author erik.zhou
 */
ByteBuf directBuf = Unpooled.directBuffer(256);
if (!directBuf.hasArray()) {
    int length = directBuf.readableBytes();
    byte[] array = new byte[length];
    directBuf.getBytes(directBuf.readerIndex(), array);
    // 处理数组
}

/**
 * 3. Composite ByteBuf - 复合缓冲区
 * 
 * @author erik.zhou
 */
CompositeByteBuf compositeBuf = Unpooled.compositeBuffer();
ByteBuf headerBuf = Unpooled.buffer(10);
ByteBuf bodyBuf = Unpooled.buffer(100);
compositeBuf.addComponents(headerBuf, bodyBuf);
```

### 5.3 内存池化

```java
// PooledByteBufAllocator - 池化内存分配器
public class PooledByteBufAllocator extends AbstractByteBufAllocator implements ByteBufAllocatorMetricProvider {
    
    public static final PooledByteBufAllocator DEFAULT = 
        new PooledByteBufAllocator(PlatformDependent.directBufferPreferred());
    
    private final PoolArena<byte[]>[] heapArenas;
    private final PoolArena<ByteBuffer>[] directArenas;
    
    @Override
    protected ByteBuf newHeapBuffer(int initialCapacity, int maxCapacity) {
        PoolThreadCache cache = threadCache.get();
        PoolArena<byte[]> heapArena = cache.heapArena;
        
        final ByteBuf buf;
        if (heapArena != null) {
            buf = heapArena.allocate(cache, initialCapacity, maxCapacity);
        } else {
            buf = PlatformDependent.hasUnsafe() ?
                new UnpooledUnsafeHeapByteBuf(this, initialCapacity, maxCapacity) :
                new UnpooledHeapByteBuf(this, initialCapacity, maxCapacity);
        }
        
        return toLeakAwareBuffer(buf);
    }
    
    @Override
    protected ByteBuf newDirectBuffer(int initialCapacity, int maxCapacity) {
        PoolThreadCache cache = threadCache.get();
        PoolArena<ByteBuffer> directArena = cache.directArena;
        
        final ByteBuf buf;
        if (directArena != null) {
            buf = directArena.allocate(cache, initialCapacity, maxCapacity);
        } else {
            buf = PlatformDependent.hasUnsafe() ?
                UnsafeByteBufUtil.newUnsafeDirectByteBuf(this, initialCapacity, maxCapacity) :
                new UnpooledDirectByteBuf(this, initialCapacity, maxCapacity);
        }
        
        return toLeakAwareBuffer(buf);
    }
}

// 使用池化内存
ByteBufAllocator allocator = PooledByteBufAllocator.DEFAULT;
ByteBuf buf = allocator.buffer(256);
try {
    // 使用 ByteBuf
} finally {
    buf.release();  // 释放内存
}
```

### 5.4 零拷贝

```java
/**
 * 1. Composite ByteBuf - 组合多个 ByteBuf
 * 
 * @author erik.zhou
 */
CompositeByteBuf compositeBuf = Unpooled.compositeBuffer();
compositeBuf.addComponents(true, header, body);

/**
 * 2. Slice - 切片（共享底层数组）
 * 
 * @author erik.zhou
 */
ByteBuf buf = Unpooled.buffer(10);
ByteBuf slice = buf.slice(0, 5);  // 不复制数据，共享底层数组

/**
 * 3. FileRegion - 文件传输
 * 
 * @author erik.zhou
 */
FileChannel fileChannel = new RandomAccessFile("file.txt", "r").getChannel();
FileRegion region = new DefaultFileRegion(fileChannel, 0, fileChannel.size());
channel.writeAndFlush(region);
```

---

## 6. 编解码器实现 🔥

### 6.1 解码器

```java
/**
 * ByteToMessageDecoder - 字节到消息解码器
 * 
 * @author erik.zhou
 */
public class MyDecoder extends ByteToMessageDecoder {
    
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        // 至少需要 4 个字节（消息长度）
        if (in.readableBytes() < 4) {
            return;
        }
        
        // 标记当前读索引
        in.markReaderIndex();
        
        // 读取消息长度
        int length = in.readInt();
        
        // 检查是否有足够的字节
        if (in.readableBytes() < length) {
            // 重置读索引
            in.resetReaderIndex();
            return;
        }
        
        // 读取消息内容
        byte[] content = new byte[length];
        in.readBytes(content);
        
        // 解码消息
        String message = new String(content, StandardCharsets.UTF_8);
        out.add(message);
    }
}

/**
 * ReplayingDecoder - 简化解码器
 * 
 * @author erik.zhou
 */
public class MyReplayingDecoder extends ReplayingDecoder<Void> {
    
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        // 不需要检查可读字节数
        int length = in.readInt();
        byte[] content = new byte[length];
        in.readBytes(content);
        
        String message = new String(content, StandardCharsets.UTF_8);
        out.add(message);
    }
}

/**
 * LineBasedFrameDecoder - 基于行的解码器
 * 
 * @author erik.zhou
 */
pipeline.addLast(new LineBasedFrameDecoder(1024));
pipeline.addLast(new StringDecoder());

/**
 * DelimiterBasedFrameDecoder - 基于分隔符的解码器
 * 
 * @author erik.zhou
 */
ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
pipeline.addLast(new DelimiterBasedFrameDecoder(1024, delimiter));
pipeline.addLast(new StringDecoder());

/**
 * LengthFieldBasedFrameDecoder - 基于长度字段的解码器
 * 
 * @author erik.zhou
 */
pipeline.addLast(new LengthFieldBasedFrameDecoder(
    1024,  // maxFrameLength
    0,     // lengthFieldOffset
    4,     // lengthFieldLength
    0,     // lengthAdjustment
    4      // initialBytesToStrip
));
```

### 6.2 编码器

```java
/**
 * MessageToByteEncoder - 消息到字节编码器
 * 
 * @author erik.zhou
 */
public class MyEncoder extends MessageToByteEncoder<String> {
    
    @Override
    protected void encode(ChannelHandlerContext ctx, String msg, ByteBuf out) throws Exception {
        byte[] content = msg.getBytes(StandardCharsets.UTF_8);
        // 写入消息长度
        out.writeInt(content.length);
        // 写入消息内容
        out.writeBytes(content);
    }
}

/**
 * MessageToMessageEncoder - 消息到消息编码器
 * 
 * @author erik.zhou
 */
public class IntegerToStringEncoder extends MessageToMessageEncoder<Integer> {
    
    @Override
    protected void encode(ChannelHandlerContext ctx, Integer msg, List<Object> out) throws Exception {
        out.add(String.valueOf(msg));
    }
}
```

### 6.3 编解码器

```java
/**
 * ByteToMessageCodec - 字节到消息编解码器
 * 
 * @author erik.zhou
 */
public class MyCodec extends ByteToMessageCodec<String> {
    
    @Override
    protected void encode(ChannelHandlerContext ctx, String msg, ByteBuf out) throws Exception {
        byte[] content = msg.getBytes(StandardCharsets.UTF_8);
        out.writeInt(content.length);
        out.writeBytes(content);
    }
    
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        if (in.readableBytes() < 4) {
            return;
        }
        
        in.markReaderIndex();
        int length = in.readInt();
        
        if (in.readableBytes() < length) {
            in.resetReaderIndex();
            return;
        }
        
        byte[] content = new byte[length];
        in.readBytes(content);
        
        String message = new String(content, StandardCharsets.UTF_8);
        out.add(message);
    }
}
```

---

## 7. 最佳实践

### 7.1 性能优化

```java
/**
 * 1. 使用池化内存
 * 
 * @author erik.zhou
 */
ServerBootstrap bootstrap = new ServerBootstrap();
bootstrap.option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT);
bootstrap.childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT);

/**
 * 2. 合理设置线程数
 * 
 * @author erik.zhou
 */
// Boss 线程数通常设置为 1
EventLoopGroup bossGroup = new NioEventLoopGroup(1);
// Worker 线程数设置为 CPU 核心数的 2 倍
EventLoopGroup workerGroup = new NioEventLoopGroup(Runtime.getRuntime().availableProcessors() * 2);

/**
 * 3. 配置 TCP 参数
 * 
 * @author erik.zhou
 */
bootstrap.option(ChannelOption.SO_BACKLOG, 1024)
    .option(ChannelOption.SO_REUSEADDR, true)
    .childOption(ChannelOption.SO_KEEPALIVE, true)
    .childOption(ChannelOption.TCP_NODELAY, true)
    .childOption(ChannelOption.SO_RCVBUF, 32 * 1024)
    .childOption(ChannelOption.SO_SNDBUF, 32 * 1024);

/**
 * 4. 及时释放 ByteBuf
 * 
 * @author erik.zhou
 */
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf buf = (ByteBuf) msg;
    try {
        // 处理消息
    } finally {
        ReferenceCountUtil.release(msg);
    }
}

/**
 * 5. 使用 writeAndFlush 批量写入
 * 
 * @author erik.zhou
 */
// 不推荐：每次都 flush
for (int i = 0; i < 100; i++) {
    ctx.writeAndFlush(msg);
}

// 推荐：批量 flush
for (int i = 0; i < 100; i++) {
    ctx.write(msg);
}
ctx.flush();
```

### 7.2 常见问题

```java
/**
 * 1. 内存泄漏问题
 * 
 * @author erik.zhou
 */
// 问题：ByteBuf 未释放导致内存泄漏
// 解决：使用 try-finally 确保释放
ByteBuf buf = ctx.alloc().buffer();
try {
    // 使用 buf
} finally {
    buf.release();
}

// 或者使用 SimpleChannelInboundHandler 自动释放
public class MyHandler extends SimpleChannelInboundHandler<String> {
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        // 消息会自动释放
    }
}

/**
 * 2. 粘包/拆包问题
 * 
 * @author erik.zhou
 */
// 问题：TCP 粘包/拆包导致消息不完整
// 解决：使用合适的解码器
pipeline.addLast(new LengthFieldBasedFrameDecoder(1024, 0, 4, 0, 4));
pipeline.addLast(new MyDecoder());

/**
 * 3. 线程安全问题
 * 
 * @author erik.zhou
 */
// 问题：在非 EventLoop 线程中操作 Channel
// 解决：使用 EventLoop 执行
if (ctx.channel().eventLoop().inEventLoop()) {
    // 在 EventLoop 线程中
    ctx.writeAndFlush(msg);
} else {
    // 不在 EventLoop 线程中，提交任务
    ctx.channel().eventLoop().execute(() -> {
        ctx.writeAndFlush(msg);
    });
}
```

---

## 8. 总结

Netty 框架的核心原理：

1. **Reactor 模型**：基于事件驱动的异步非阻塞 I/O 模型
2. **EventLoop**：单线程事件循环，处理 I/O 事件和任务
3. **Channel 和 Pipeline**：责任链模式处理网络事件
4. **ByteBuf**：高效的字节缓冲区，支持池化和零拷贝
5. **编解码器**：灵活的编解码框架，解决粘包/拆包问题

掌握这些核心原理，可以帮助我们：
- 开发高性能的网络应用
- 排查和解决网络编程问题
- 进行性能优化和自定义扩展

---

## 📚 参考资料

- [Netty 官方文档](https://netty.io/wiki/)
- [Netty GitHub](https://github.com/netty/netty)
- 《Netty 实战》
- 《Netty 权威指南》

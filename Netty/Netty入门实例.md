TCP服务：创建Netty项目
===
要求：
	1. Netty服务器在8889端口监听，客户端能发送消息给服务器：“hello，服务器！”。
	2. 服务器可以恢复消息给客户端：“hello，客户端！”。

思路：
	1. 编写服务端。
	2. 编写客户端。
	3. 对netty程序进行分析，研究netty模型的特点。
	
服务器端：
```java
package com.netty.netty.simple;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

/**
 * version 1.0
 * Created by Grtwwh2019
 * since 2020-05-23 15:01:58
 */
public class NettyServer {


    public static void main(String[] args) throws InterruptedException {
        // 创建BossGroup和WorkerGroup
        // 说明：
        // 1.创建两个线程组 bossGroup、workerGroup
        // 2.bossGroup只是处理accept（连接）请求，真正的与客户端进行业务处理会交给workerGroup完成
        // 3.两者都是无限循环
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            // 创建服务器端的启动对象，配置参数
            ServerBootstrap serverBootstrap = new ServerBootstrap();

            // 使用链式编程进行设置
            serverBootstrap.group(bossGroup, workerGroup) // 设置两个线程组
                    .channel(NioServerSocketChannel.class) // 使用NioServerSocketChannel作为服务器的通道实现
                    .option(ChannelOption.SO_BACKLOG, 128) // 设置线程队列等待连接的个数
                    .childOption(ChannelOption.SO_KEEPALIVE, true) // 设置保持活动连接状态
                    .childHandler(new ChannelInitializer<SocketChannel>() { // 创建一个通道初始化对象（匿名内部类）
                        // 给pipeline设置一个处理器
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            // pipeline()返回该channel关联的pipeline
                            // addLast：向pipeline的最后增加一个处理器
                            socketChannel.pipeline().addLast(new NettyServerHandler());
                        }
                    }); // 给我们的workerGroup的 EventLoop对应的管道 设置处理器
            System.out.println("服务器准备就绪...");

            // 绑定一个端口，并且进行同步，生成一个ChannelFuture对象（启动服务器）
            ChannelFuture channelFuture = serverBootstrap.bind(8889).sync();

            // 对关闭通道进行监听
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

服务器端的管道处理器：
```java
package com.netty.netty.simple;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.CharsetUtil;

/**
 * version 1.0
 * Created by Grtwwh2019
 * since 2020-05-23 15:21:55
 * 说明：
 * 1.自定义一个Handler需要继承Netty规定好的某个HandlerAdapter
 * 2.此时，自定义的Handler才能称为一个Handler（具有某些特定的功能）
 */
public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     * 读取数据事件（这里可以读取客户端发送的消息）
     *
     * @param ctx：ChannelHandlerContext，上下文对象，包含了管道pipeline，通道channel，地址
     * @param msg：Object，就是客户端发送的数据，默认是Object类型
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("server ctx = " + ctx);
        // 将msg转成一个ByteBuf（Nett提供的）
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("客户端发送的消息是：" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("客户端地址：" + ctx.channel().remoteAddress());
    }

    /**
     * 数据读取完毕的回调
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        // 将数据写入缓存并且刷新到管道（pipeline）
        // 一般会对发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello，客户端！", CharsetUtil.UTF_8));
    }

    /**
     * 处理异常，一般需要关闭通道
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.channel().close();
    }
}
```

客户端：
```java
package com.netty.netty.simple;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

/**
 * version 1.0
 * Created by Grtwwh2019
 * since 2020-05-23 15:52:49
 */
public class NettyClient {

    public static void main(String[] args) throws InterruptedException {
        // 客户端需要一个事件循环组
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            // 创建客户端启动对象
            // 注意客户端使用的是BootStrap
            Bootstrap bootstrap = new Bootstrap();
            // 设置相关参数
            bootstrap.group(group) // 设置线程组
                    .channel(NioSocketChannel.class) // 设置客户端通道的实现类型（反射）
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new NettyClientHandler()); // 加入自己的处理器
                        }
                    });

            System.out.println("客户端准备就绪...");

            // 启动客户端去连接服务器端
            // 关于ChannelFuture，涉及到netty的异步模型
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 8889).sync();
            // 给关闭通道进行监听
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }

    }
}
```

客户端的管道处理器：
```java
package com.netty.netty.simple;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.CharsetUtil;

/**
 * version 1.0
 * Created by Grtwwh2019
 * since 2020-05-23 16:16:30
 */
public class NettyClientHandler extends ChannelInboundHandlerAdapter {

    /**
     * 当通道就绪时，就会触发该方法
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("客户端 " + ctx);
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello，服务器端！", CharsetUtil.UTF_8));
    }

    /**
     * 当通道有读取事件时，会触发该方法
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("服务器回复的消息：" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("服务器的地址：" + ctx.channel().remoteAddress());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```


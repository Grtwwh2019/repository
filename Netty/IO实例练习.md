I/O模型
===

BIO实例：创建一个服务端接收客户端的连接，当有一个客户端连接时就启动一个线程与之通信，并使用线程池进行改善。（客户端使用telnet发送消息即可）
---
```java
// 服务端代码
/**
 * version 1.0
 * Created by Grtwwh2019
 * since 2020-05-08 15:06:49
 */
public class BIOServer {

    public static void main(String[] args) throws IOException {
        // 创建线程池
        ExecutorService executorService = Executors.newCachedThreadPool();
        ServerSocket serverSocket = new ServerSocket(6666);
        System.out.println("服务器已启动");
        while (true) {
            // 监听，等待客户端连接
            final Socket socket = serverSocket.accept();
            System.out.println("连接到一个客户端");
            // 如果有客户端连接，就创建一个线程，与之通信
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println("线程信息 id：" + Thread.currentThread().getId() + ";名字："
                            + Thread.currentThread().getName());
                    // 可以和客户端进行通讯
                    handler(socket);
                }
            });
        }

    }

    // handler，和客户端进行通讯
    private static void handler(Socket socket) {
        try {
            byte[] bytes = new byte[1024];
            // 通过socket获取一个输入流
            InputStream socketInputStream = socket.getInputStream();
            // 读取数据
            int len;
            while ((len = socketInputStream.read(bytes)) != -1) {
                // 输出客户端发送的数据
                System.out.println(new String(bytes, 0, len));
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            System.out.println("关闭和客户端的连接");
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

NIO实例：编写一个NIO案例，实现服务器端和客户端之间的数据简单通讯（非阻塞）
---
服务器端：
```java
 public class NIOServer {

    public static void main(String[] args) throws IOException {
        // 创建ServerSocketChannel
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 得到一个Selector对象
        Selector selector = Selector.open();
        // 绑定一个端口 8889，在服务器端监听
        serverSocketChannel.socket().bind(new InetSocketAddress(8889));
        // 设置为非阻塞
        serverSocketChannel.configureBlocking(false);
        // 把ServerSocketChannel注册到Selector，关心OP_ACCEPT事件
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        // 循环等待客户端连接
        while (true) {

            // 这里等待1秒，如果没有事件发生就继续
            if (selector.select(1000) == 0) { // 没有事件发生
                System.out.println("服务器等待了1秒，无连接");
                continue;
            }
            // 如果返回的>0，获取到相关的SelectionKey集合
            // 1.如果返回的>0，表示已经获取到关注的事件
            // 2.selector.selectedKeys();返回关注事件的集合
            // 3.通过selectionKeys反向获取通道
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            // 遍历SelectionKey，使用迭代器遍历
            Iterator<SelectionKey> keyIterator = selectionKeys.iterator();
            while (keyIterator.hasNext()) {
                // 获取到selectionKey
                SelectionKey selectionKey = keyIterator.next();
                // 根据key对应的通道发生的事件，做相应的处理
                if (selectionKey.isAcceptable()) { // 如果是OP_ACCEPT事件，有新的客户端来连接了
                    // 给该客户端生成一个SocketChannel
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    // 将SocketChannel设置为非阻塞
                    socketChannel.configureBlocking(false);
                    System.out.println("客户端连接生成，生成了SocketChannel " +socketChannel.hashCode());
                    // 将当前的socketChannel注册到Selector上，关注的事件为OP_READ，
                    // 同时给该socketChannel关联一个buffer
                    socketChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocate(1024));
                }
                if (selectionKey.isReadable()) { // 发生OP_READ事件
                    // 通过selectionkey反向获取对应的通道
                    SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
                    // 获取到该channel关联的buffer
                    ByteBuffer byteBuffer = (ByteBuffer) selectionKey.attachment();
                    // 把当前channel的数据读取到buffer
                    socketChannel.read(byteBuffer);
                    System.out.println("from 客户端：" + new String(byteBuffer.array()));

                }

                // 手动从集合中移除当前的selectionKey，防止重复操作
                keyIterator.remove();
            }
        }
    }
}
```
客户端：
```java
public class NIOClient {

    public static void main(String[] args) throws IOException {
        // 得到一个网络通道
        SocketChannel socketChannel = SocketChannel.open();
        // 设置非阻塞模式
        socketChannel.configureBlocking(false);
        // 提供服务器端的 IP 和 端口
        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 8889);
        // 连接服务器
        if (!socketChannel.connect(inetSocketAddress)) {
            while (!socketChannel.finishConnect()) {
                System.out.println("因为连接需要时间，客户端不会阻塞，可以做其他工作！");
            }
        }
        // 如果连接成功，就可以发送数据
        String strMsg = "hello nio example";
        // wrap()：根据直接字节数组的大小返回Buffer
        ByteBuffer buffer = ByteBuffer.wrap(strMsg.getBytes());
        // ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        // 发送数据，将buffer的数据写入到channel
        socketChannel.write(buffer);
        System.in.read();
    }
}
```
结果
```
每有一个客户端连接都会生成一个不同的SocketChannel。
```

NIO实例：群聊系统
---
要求：
* 编写一个NIO群聊系统，实现服务器端和客户端之间的简单数据通讯（非阻塞）。
* 实现多人群聊。
* 服务器端：可以监测用户上线、离线，并实现消息转发功能。
* 客户端：通过channel可以无阻塞发送消息给其它所有用户，同时可以接受其它用户发送的消息（由服务器转发得到）。
* 目的：进一步理解NIO非阻塞网络编程机制。
---
服务器端：
1. 服务器启动并监听8889。
2. 服务器接收客户端信息，并实现转发【处理上线和离线】。
```java
package com.netty.nio.groupChat;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

/**
 * version 1.0
 * Created by Grtwwh2019
 * since 2020-05-19 18:45:59
 */
public class GroupChatServer {

    // 定义相关属性
    private Selector selector;
    private ServerSocketChannel listenChannel; // 用来监听
    private static final int PORT = 8889;

    // 构造器
    // 初始化工作
    public GroupChatServer() {
        try {
            // 得到选择器
            selector = Selector.open();
            // 得到ServerSocketChannel
            listenChannel = ServerSocketChannel.open();
            // 绑定端口
            listenChannel.socket().bind(new InetSocketAddress(PORT));
            // 设置非阻塞模式
            listenChannel.configureBlocking(false);
            // 将该listenChannel注册到selector
            listenChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
        }
    }

    // 监听
    public void listen() {
        try {
            // 循环监听处理
            while (true) {
                int count = selector.select(2000);
                // 根据count返回的值进行处理
                if (count > 0) { // 有事件处理
                    // 遍历得到的SelectionKey集合
                    Set<SelectionKey> selectionKeys = selector.selectedKeys();
                    Iterator<SelectionKey> it = selectionKeys.iterator();
                    while (it.hasNext()) {
                        // 取出selectionKey
                        SelectionKey key = it.next();

                        if (key.isAcceptable()) { // 监听到的是ACCEPT，连接事件
                            // 处理连接
                            SocketChannel socketChannel = listenChannel.accept();
                            // 设置非阻塞
                            socketChannel.configureBlocking(false);
                            // 将该socketChannel注册到selector上
                            socketChannel.register(selector, SelectionKey.OP_READ);
                            // 提示
                            System.out.println(socketChannel.getRemoteAddress() + " 已上线");

                            socketChannel.configureBlocking(false);

                        }
                        if (key.isReadable()) { // 通道发生read事件，即通道是可读的
                            // 处理读（封装一个方法）
                            readDataFromClient(key);
                        }

                        // 移除当前的key，防止重复处理
                        it.remove();

                    }
                } else {
//                    System.out.println("等待中...");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
        }
    }

    // 读取客户端消息
    private void readDataFromClient(SelectionKey key) { // 通过SelectionKey反向获取Channel
        // 定义一个SocketChannel
        SocketChannel channel = null;
        try {
            // 取得关联的channel
            channel = (SocketChannel) key.channel();
            // 创建缓冲区buffer
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            // 读取数据
            int count = channel.read(buffer); // 读取到的数据数量
            // 根据count的值做处理
            if (count > 0) { // 读取到数据了
                // 把缓冲区的数据转成字符串并输出
                String msg = new String(buffer.array());
                // 输出该消息
                System.out.println("From Client: " + msg);
                // 向其他的客户端转发消息（排除发送方），封装一个方法来处理
                sendInfoToOtherClients(msg, channel);
            }
        } catch (IOException e) {
            try {
                System.out.println(channel.getRemoteAddress() + " 离线了");
                // 取消注册
                key.cancel();
                // 关闭通道
                channel.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        } finally {
        }

    }

    /**
     * 转发消息给其他的客户端(通道,一个客户端对应一个通道）
     * @param msg
     * @param self ： 发送方客户端
     */
    private void sendInfoToOtherClients(String msg, SocketChannel self) throws IOException {
        // 提示
        System.out.println("Server转发消息中...");
        // 遍历 所有注册到Selector上的SocketChannel，并排除发送方（self）
        // 注意selector.selectedKeys() 和 selector.keys()的区别
        Set<SelectionKey> keys = selector.keys();
        for (SelectionKey selectionKey : keys) {
            // 通过 key 取出对应的通道SocketChannel（转发的目标客户端）
            Channel targetChannel = selectionKey.channel();
            // 排除发送方（self）
            if (targetChannel instanceof SocketChannel && targetChannel != self) {
                // 转型
                SocketChannel destination = (SocketChannel) targetChannel;
                // 将msg存储到buffer
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                // 将buffer的数据写入到通道中
                destination.write(buffer);
            }
        }
    }

    public static void main(String[] args) {
        // 创建服务器对象
        GroupChatServer chatServer = new GroupChatServer();
        // 进行监听
        chatServer.listen();
    }
}
```


客户端：
1. 连接服务器
2. 发送消息
3. 接收服务器端的消息
```java
package com.netty.nio.groupChat;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Scanner;
import java.util.Set;

/**
 * version 1.0
 * Created by Grtwwh2019
 * since 2020-05-20 01:43:08
 */
public class GroupChatClient {

    // 定义相关的属性
    private final String HOST = "127.0.0.1"; // 服务器的IP
    private final int PORT = 8889; // 服务器的端口
    private Selector selector;
    private SocketChannel socketChannel;
    private String username;

    // 构造器，完成初始化工作（打开客户端）
    public GroupChatClient() {
        try {
            // 得到选择器
            selector = Selector.open();
            // 连接服务器
            socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", PORT));
            // 设置非阻塞
            socketChannel.configureBlocking(false);
            // 将socketChannel注册到Selector
            socketChannel.register(selector, SelectionKey.OP_READ);
            // 得到username（客户端的LocalAddress）
            username = socketChannel.getLocalAddress().toString().substring(1);
            // 提示
            System.out.println(username + " is OK!");
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
        }
    }

    // 向服务器发送消息
    public void sendInfo(String info) {
        info = username + " 说： " + info;
        try {
            // 发送消息
            socketChannel.write(ByteBuffer.wrap(info.getBytes()));
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
        }
    }

    // 读取从服务器端回复的消息
    public void readInfo() {
        try {
            int readChannels = selector.select();
            if (readChannels > 0) { // 有事件发生的通道
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> it = selectionKeys.iterator();
                while (it.hasNext()) {
                    SelectionKey key = it.next();
                    if (key.isReadable()) {
                        // 得到相关的通道（可读）
                        SocketChannel channel = (SocketChannel) key.channel();
                        // 得到一个buffer
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        // 从通道读取到buffer
                        channel.read(buffer);
                        // 把缓冲区（buffer）的数据转成字符串
                        String msg = new String(buffer.array());
                        // 输出消息
                        System.out.println(msg.trim());
                    }
                }
                it.remove();
            } else {
                System.out.println("无可用通道，等待中...");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
        }
    }


    public static void main(String[] args) {
        // 启动客户端
        GroupChatClient chatClient = new GroupChatClient();
        // 开启线程，每隔3秒就读取 从服务器端发送的数据
        new Thread() {
            public void run() {
                while (true) {
                    // 读取信息
                    chatClient.readInfo();
                    try {
                        Thread.currentThread().sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                    }
                }
            }
        }.start();

        // 发送数据给服务器端
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()) { // 读取一行
            String line = scanner.nextLine();
            chatClient.sendInfo(line);
        }
    }
}

```

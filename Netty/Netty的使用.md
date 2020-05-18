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
服务器端
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
客户端
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
结果：每有一个客户端连接都会生成一个不同的SocketChannel。

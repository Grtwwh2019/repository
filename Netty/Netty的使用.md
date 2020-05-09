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

NIO实例：
---

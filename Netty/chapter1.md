什么是Netty？
===
概念
---
Netty提供``异步的``、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序。   
Netty是一个基于``NIO``的客户、服务器端的编程框架。   

特点
---
* 设计    
  * 各种传输类型的统一API——阻塞和非阻塞套接字   
  * 基于灵活和可扩展的事件模型，允许明确的关注点分离    
  * 高度可定制的线程模型——单个线程，一个或多个线程池，如SEDA   
  * 真正的无连接数据报套接字支持
* 易用性
  * 文档良好的Javadoc、用户指南和示例    
* 性能    
  * 更好的吞吐量，更低的延迟    
  * 减少资源消耗
  * 减少内存拷贝    
* 安全性
  * 完整的 SSL / TLS 和 StartTLS 的支持    

基础
----
* Java BIO、NIO、AIO    
* Java 网络编程    
* 设计模式：观察者模式、命令模式、职责链模式   
* 数据结构：链表   
* Java 多线程编程   

Java BIO、NIO、AIO （进行基于C/S模式的网络编程）
====
BIO（IO）
---
BIO（Block-IO）：传统的IO，是同步阻塞的交互方式，在I、O完成之前，线程是阻塞状态的。它们的调用是线性的可靠的，BIO的代码简单直观，但是效率和扩展性存在瓶颈。    

流的分类
* 按数据单位的不同：字节流（8 bit）、字符流（16 bit）   
* input和output是在程序，也就是内存的角度来看的（按流向的不同）。   
  * input：`读取`外部的数据（磁盘、网络中的数据）到内存中。   
  * output：将内存的数据`输出`到磁盘、网络当中。    
* 按流的角色的不同：节点流、处理流    
  * 节点流：直接作用在数据上    
  * 处理流：作用在已有的流的基础上   

IO的基础类    
* 4个抽象基类
  * InputStream   
  * OutputStream    
  * Reader    
  * Writer    
* 4个基础节点流
  * FileInputStream   
  * FileOutputStream    
  * FileReader    
  * FileWriter    
* 常用的处理流
  * BufferedInputStream   
  * BufferedOutputStream    
  * BufferedReader    
  * BufferedWriter    
  * InputStreamReader   
  * OutputStreamWriter    
  * ObjectInputStream   
  * ObjectOutputStream    

* 简单的IO流代码    
  * FileReader：把磁盘中文件（hello.txt）里的内容读入到内存当中。    
  
    * 使用read()读取文件    
    ```Java
    @Test
      public void testFileReader() {
          File file = new File("src/IOTest/hello.txt");
          FileReader fileReader = null;
          try {
              // 提供具体的流
              fileReader = new FileReader(file);
              // 数据的读入
              // read()：返回读入的一个字符，如果到达文件末尾，则返回-1；
              int data;
              while ((data = fileReader.read()) != -1) {
                  System.out.print((char) data);
              }
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              // 流的关闭
              if (fileReader != null) {
                  try {
                      fileReader.close();
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
    ```
    * 使用read(int[] cbuf)读取文件，一次读入多个,返回每次读入到内存的字符的个数
    ```java
    @Test
    public void testFileReader1() {
        File file = new File("src/IOTest/hello.txt");
        FileReader fileReader = null;
        try {
            fileReader = new FileReader(file);
            // 使用read(int[] cbuf)：读取文件,一次读入多个,返回每次读入到内存的字符的个数
            char[] cbuf = new char[1024];
            // 每次读入的字符个数
            int len;
            while ((len = fileReader.read(cbuf)) != -1) {
                // public String(char value[], int offset, int count) {}
                String s = new String(cbuf, 0, len);
                System.out.print(s);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileReader != null) {
                try {
                    fileReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    ```
    
  * FileWriter：从内存中写出数据到磁盘文件。   
  
    * write(string)输出
    ```java
    @Test
    public void testFileWriter() {
        File file = new File("src/IOTest/hello1.txt");
        FileWriter fileWriter = null;
        try {
            // 提供FileWriter的对象，用于数据的输出
            // public FileWriter(File file, boolean append) {}
            // append：默认为false，若为true，则在文件末尾继续输出
            fileWriter = new FileWriter(file);
            // 写出的操作
            fileWriter.write("hello netty!");
            // 流的关闭
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileWriter != null) {
                try {
                    fileWriter.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    ```   
    
  * 文件的读入、写出（文件的复制）
  ```java
  @Test
  public void testFileReaderFileWriter() {
      // 文件的读入、写出（复制文件）
      File rfile = new File("src/IOTest/hello1.txt");
      File wfile = new File("src/IOTest/hello2.txt");
      FileReader fileReader = null;
      FileWriter fileWriter = null;
      try {
          // 创建输入流和输出流对象
          fileReader = new FileReader(rfile);
          fileWriter = new FileWriter(wfile);
          // 数据的读入和写出操作
          char[] cbuf = new char[1024];
          int len;
          while ((len = fileReader.read(cbuf)) != -1) {
              fileWriter.write(cbuf, 0, len);
          }
      } catch (IOException e) {
          e.printStackTrace();
      } finally {
          // 关闭流资源
          if (fileReader != null) {
              try {
                  fileReader.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
          if (fileWriter != null) {
              try {
                  fileWriter.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  ```    
  
  * FileInputStream、FileOutputStream的使用（复制字节文件）
  ```java
  @Test
    public void testFileInputStreamFileOutputStream() throws IOException {
        File inputfile = new File("src/IOTest/xszpLoad.jpg");
        File outputfile = new File("src/IOTest/zzj.jpg");
        // 创建字节流对象
        FileInputStream fileInputStream = new FileInputStream(inputfile);
        FileOutputStream fileOutputStream = new FileOutputStream(outputfile);
        // 读取数据
        byte[] bytes = new byte[1024];
        int len;
        while ((len = fileInputStream.read(bytes)) != -1) {
            // 输出数据
            fileOutputStream.write(bytes, 0, len);
        }
        fileInputStream.close();
        fileOutputStream.close();
    }
  ```
  
  
NIO（多路复用）
---
NIO（NonBlocak-IO）：Java4中引入了NIO框架，非阻塞IO，提供了Channel、Selector、Buffers等新抽象基类，可以构建多路复用的、同步非阻塞IO操作。与BIO的区别是在第一次发送请求时，线程并没有被阻塞，而是反复检查数据是否已经准备好，将大区域的阻塞分为多块更小的阻塞（自旋）。   

NIO的核心：
* Channels
* Buffers
* Selectors   


AIO
---
AIO（Asynchronous）：异步-非阻塞，基于事件和回调机制。

  






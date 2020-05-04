Java BIO、NIO、AIO （进行基于C/S模式的网络编程）
====
BIO（IO）
---
BIO（Block-IO）：传统的IO，是同步阻塞的交互方式，在I、O完成之前，线程是阻塞状态的。它们的调用是线性的可靠的，BIO的代码简单直观，但是效率和扩展性存在瓶颈。    

流的分类
* 按数据单位的不同：字节流（8 bit）、字符流（16 bit）   
* 按流的类型的不同：input（输入流）和output（输出流）是在程序，也就是内存的角度来看的（按流向的不同）。   
  * input：`读取`外部的数据（磁盘、网络中的数据）到内存中。   
  * output：将内存的数据`输出`到磁盘、网络当中。    
  `注意：`在每一次写入（input）数据完成后，刷新（flush）输出流（output）是非常重要的。因为缓冲区一般来说在接收到一定数据以后才会进行发送数据，此时有可能导致缓冲区接收到的数据不够大，缓冲区不进行数据发送，而服务器又等不到再次的数据请求，也不发送更多的数据到缓冲区，就会造成死锁状态。如果不刷新输出流，数据可能会丢失。
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
  * FileReader(readLine())    
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
  
  * 缓冲流，BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter  
  
  `提升流的输入、输出速度`  
  
  ```java
  @Test
    public void testBuffered() throws IOException {
        System.out.println("start: " + new SimpleDateFormat("yyyy-MM-dd hh:ss:mm").format(System.currentTimeMillis()));
        // 创建流
        FileInputStream fileInputStream = new FileInputStream("src/IOTest/xszpLoad.jpg");
        FileOutputStream fileOutputStream = new FileOutputStream("src/IOTest/newxxx.jpg");
        BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
        BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
        // 具体的读取、写入细节
        byte[] bytes = new byte[1024]; // 写1024个字节到bytes
        int len;
        while ((len = bufferedInputStream.read(bytes)) != -1) {
            bufferedOutputStream.write(bytes, 0, len);
            // bufferedOutputStream.flush(); // 清空缓冲区
        }
        System.out.println("finished: " + new SimpleDateFormat("yyyy-MM-dd hh:mm:ss").format(System.currentTimeMillis()));
        // 关闭的顺序，要求先关闭外层的流，再关闭内层的流
        // 而关闭外层流的同时，内层流也会自动关闭，因此可以省略内层流的关闭
        bufferedOutputStream.close();
        bufferedInputStream.close();
        // fileOutputStream.close();
        // fileInputStream.close();
    }
  ```
  
  * 转换流，InputStreamReader、OutputStreamWriter  
  输入的字节流转换为输入的字符流、输出的字符流转换为输出的字节流。  
  `提供字节流与字符流之间的转换`
  ```java
  @Test
    public void testTransferStream() throws IOException {
        FileInputStream fileInputStream = new FileInputStream("src/IOTest/hello1.txt");
        FileOutputStream fileOutputStream = new FileOutputStream("src/IOTest/hello3.txt");
        // 参数2指明了字符集，具体使用哪个字符集，取决于文件保存时使用的字符集
        InputStreamReader inputStreamReader = new InputStreamReader(fileInputStream, "UTF-8"); // 解码
        OutputStreamWriter outputStreamWriter = new OutputStreamWriter(fileOutputStream, "GBK"); // 编码
        char[] chars = new char[1024];
        int len;
        while ((len = inputStreamReader.read(chars)) != -1) {
            outputStreamWriter.write(chars, 0, len);
        }
        inputStreamReader.close();
        outputStreamWriter.close();
    }
  ```
  
  * 对象流，ObjectInputStream、ObjectOutputStream
  用于存储和读取基本数据类型数据或对象的处理流。可以把Java中的对象写入到数据源，也可以把对象从数据源还原回来。  
  序列化：将内存中的Java对象保存到磁盘中或者通过网络传输出去，使用ObjectOutputStream类保存基本数据类型数据或`对象`的机制。（输出）  
  反序列化：将磁盘文件中的对象还原为Java的一个对象，使用ObjectInputStream类读取基本数据类型数据或对象的机制。（输入）   
  ```java
  // 序列化
    @Test
    public void testObjectOutStream() throws IOException {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("src/IOTest/testObject.dat"));
        // 此时写出的文件并不是明文，因为其经过了序列化的过程。
        // 要想正常打开得到内容，需要进行反序列化。
        objectOutputStream.writeObject(new String("i love java!"));
        // 刷新操作
        objectOutputStream.flush();
        objectOutputStream.close();
    }

    // 反序列化
    @Test
    public void testObjectInputStream() throws IOException, ClassNotFoundException {
        FileInputStream fileInputStream = new FileInputStream("src/IOTest/testObject.dat");
        ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);
        // 对保存了Java对象的文件进行反序列化
        Object object = objectInputStream.readObject();
        System.out.println(object.toString());
        objectInputStream.close();
    }
  ```
  `注意：`如果序列化的是自定义的类，必须实现Serializable接口，同时也要保证该类的所有属性也是可序列化的，还需要显式声明一个serialVersionUID，用来对序列化对象进行版本控制，在反序列化时判断是否兼容。另外要注意的是，被static和transient修饰的成员变量是不可以进行序列化和反序列化的。    
  
`补充：`JDK7引入了“带资源的try”代码块，可以简洁的完成在finally中close资源（只要对象实现了Closeable接口）。
```java
@Test
    public void testFileWriter() {
        // public abstract class Writer implements Appendable, Closeable, Flushable {}
        File file = new File("src/IOTest/hello1.txt");
        // 现在不再需要finally子句，Java会自动的对该对象调用close()方法。
        try (FileWriter fileWriter = new FileWriter(file)) {
            // 写出的操作
            fileWriter.write("hello netty!");
            // 流的关闭
        } catch (IOException e) {
            e.printStackTrace();
        } 
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

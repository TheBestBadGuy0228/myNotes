# NIO

### java NIO 与 IO的主要区别

![](https://i.loli.net/2020/12/02/sArlWP7XLU5wd63.png)

* 传统IO面向的是流，是单向的阻塞I/O

![](https://i.loli.net/2020/12/02/xasDVuk2I8jJ95C.png)

**传统I/O是单向的只能定向传输，一个输入流一个输出流**

* NIO面向缓冲区

![](https://i.loli.net/2020/12/02/9snzwBX2kWmL5Zp.png)

**在NIO中channel相当于轨道，缓冲区相当于火车，他可以从两点往返运输**

### Java NIO核心组成部分：

* Channel
* Buffers
* Selectors

**在java NIO中，Selector是中央控制器，Buffer是承载数据的容器，而Channel可以说是最基础的门面，它是本地I/O设备与网络I/O设备的通讯桥梁**

#### Channel和Buffer

![](https://i.loli.net/2020/12/01/RV8m7IJepdvzEP1.png)

**基本上所有的额IO在NIO中都是从一个Channel开始。Channel有点像流。数据可以从Channel读到Buffer中，也可以从Buffer写到Channel中。**

**简而言之：channel负责传输，Buffer负责存储。**

* 主要的Channel的实现：

  * FileChannel------》从文件中读取数据
  * DataGramChannel--------》通过UDP读取网络中的数据
  * SocketChannel-----------》通过TCP读取网络中的数据
  * ServerSocketChannel------------》监听新进来的TCP连接，对每个新的连接都创建一个socketChannel

* 主要的Buffer实现：

  * byteBuffer--------》byte
  * CharBuffer--------》char
  * DoubleBuffer-----------》double
  * FloatBuffer---------》float
  * intBuffer----------》int
  * LongBuffer----------》long
  * ShortBuffer----------》short

  **这些buffer覆盖了你能通过IO发送的基本数据类型**

#### Selector

* Selector允许单个线程处理多个Channel，如果你的应用打开了多个连接（通道），但是每个连接的流量都很低，及使用selector就会很方便。例如在一个聊天服务器中

![](https://i.loli.net/2020/12/01/DNT1MIZ3bhJOyrW.png)

**单线程中使用一个selector处理三个Channel图示**

**要是用Selector得向Selector注册Channel，然后调用它的select（）方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子就如同有新的连接进来数据接收等**

### Channel

* java中的Channel类似于流，但是又有不同：
  * 既可以从channel中读取数据，又可以写数据到通道。但流的读写通常是单向的。
  * 通道可以异步得读写。
* channel实例：

```java
   /**
 * Channel：由于源节点与目标节点的连接。在java NIO中负责缓冲区中数据的传输。Channel本身不存储数据，因此需要配合缓冲区进行传输。
 *
 * 通道的主要实现类
 * java.nio.channels.Channel接口：
 *      FileChannel
 *      SocketChannel
 *      ServerSocketChannel
 *      DatagramChannel
 *
 * 获取通道
 * java针对支持的通道的类提供了getChannel（）方法
 *      本地IO
 *      FileInputStream/FileOutPutStream
 *      RandomAccess
 *      网络IO：
 *      Socket
 *      ServerSocket
 *      DatagramSocket
 *
 * 在JDK1.7中的NIO.2 针对各个通道提供了静态方法open（）
 * 在JDK1.7中的NIO.2 的File工具类的NewByteChannel（）
 *
 */
	@Test
    public void Test1() throws IOException {
        //创建一个随机访问的文件流从"D:/nio-data.txt"中读取，并指定读写权限
        RandomAccessFile afile = new RandomAccessFile("D:/nio-data.txt", "rw");
        //获取改文件的唯一fileChannel对象
        FileChannel inChannel = afile.getChannel();
        //分配一个新的字节缓冲区,容量为48个字节
        ByteBuffer buf = ByteBuffer.allocate(48);
        //从通道读取到给定缓冲区的字节序列
        int byteRead = inChannel.read(buf);
        //读取的字节数，可能为零，如果通道已达到流出端， 则为-1 ,当返回值为-1时则读取完毕
        while (byteRead != -1) {

            System.out.println("Read" + byteRead);
            //该限制设置为当前位置，然后将该位置设置为零。 如果标记被定义，则它被丢弃。
            //在通道读取或放置操作的序列之后，调用此方法来准备一系列通道写入或相对获取操作。
            //该方法的功能应该是把指针移到开始位置
            buf.flip();
            //public final boolean hasRemaining()
            //告诉当前位置和极限之间是否存在任何元素。
            //结果
            //true如果，并且只有在此缓冲区中至少有一个元素
            while (buf.hasRemaining()) {
                //获取缓冲区当前位置的字节
                System.out.println((char) buf.get());

            }
            //清除缓冲区
            buf.clear();

            byteRead = inChannel.read(buf);
        }
        //关闭此随机访问文件流并释放与流相关联的任何系统资源
        afile.close();

    }
```

### BUffer

* Buffer的基本用法
  * 写入数据到Buffer
  * 调用flip（）方法
  * 从Buffer中读取数据
  * 调用clear（）方法或者compact（）

**当向Buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip（）方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据。**

一旦读完所有数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区；调用clear（）或者compact（）方法。clear（）方法会清空整个缓冲区。compact（）方法只会清除已经读过得数据，任何未读的数据都会被移到缓冲区的起始处，新写入的数据会追加到后面。

* Buffer常用方法

![](https://i.loli.net/2020/12/09/u9gQcIGlXNtdKfU.png)

![](https://i.loli.net/2020/12/09/DXHqc7UK9ezTdPF.png)

* capacity/limit/position

![](https://i.loli.net/2020/12/04/ShGIW15suNy3TX6.png)

```
     * 缓冲区存取数据的核心方法：
     * put（）：存入数据到缓冲区
     * get（）：从缓冲区中获取数据
     * <p>
     * 缓冲区中的四个核心属性
     * <p>
     * capacity：容量，表阿斯缓冲区中最大存储数据的容量。一旦指定无法修改。
     * limit：界限，缓冲区中可以操作数据的大小。（limit后面的数据不能进行读写）
     * position：位置，表示缓冲区中正在操作数据的位置
     * 关系：position <= limit <= capacity
```

```java
       /**
     * Buffer（缓冲区）在Java NIO中负责数据的存取。缓冲区就是数组。用于存储不同的数据类型的数据
     * <p>
     * 根据不同的数据类型（Boolean除外），提供了相应类型的数据缓冲区：
     * ByteBuffer
     * CharBuffer
     * ShortBuffer
     * IntBuffer
     * LongBuffer
     * FloatBuffer
     * DoubleBuffer
     * 上述缓冲区的管理方式几乎一致，通过allocate（）方法获取
     * <p>
     * 缓冲区存取数据的核心方法：
     * put（）：存入数据到缓冲区
     * get（）：从缓冲区中获取数据
     * <p>
     * 缓冲区中的四个核心属性
     * <p>
     * capacity：容量，表阿斯缓冲区中最大存储数据的容量。一旦指定无法修改。
     * limit：界限，缓冲区中可以操作数据的大小。（limit后面的数据不能进行读写）
     * position：位置，表示缓冲区中正在操作数据的位置
     * <p>
     * 关系：0 <= mark <= position <= limit <= capacity
     * <p>
     * * mark:标记当前的position位置，reset（）：将position恢复到mark的位置
     */
 	@Test
    public void Test1() {
        String str = "abcde";
        //分配一个大小为1024个字节的缓冲区
        System.out.println("-------------allocate----------------");
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        System.out.println(byteBuffer.capacity());
        System.out.println(byteBuffer.limit());
        System.out.println(byteBuffer.position());

        //利用put（）存入数据到缓冲区
        byteBuffer.put(str.getBytes());

        System.out.println("--------------put()----------------------");
        System.out.println(byteBuffer.position());
        System.out.println(byteBuffer.limit());
        System.out.println(byteBuffer.capacity());

        //切换读取模式利用flip（）方法将position从数据末尾切换到数据起点；flip()函数的作用是将写模式转变为读模式
        byteBuffer.flip();
        System.out.println("--------------flip()----------------------");
        System.out.println(byteBuffer.position());
        System.out.println(byteBuffer.limit());
        System.out.println(byteBuffer.capacity());

        //利用get（）读取缓冲区中的数据
        System.out.println("--------------get()----------------------");
        byte[] bytes = new byte[byteBuffer.limit()];
        byteBuffer.get(bytes);
        System.out.println(new String(bytes, 0, bytes.length));
        System.out.println(byteBuffer.position());
        System.out.println(byteBuffer.limit());
        System.out.println(byteBuffer.capacity());

        //rewind()重复读/写；rewind()在读写模式下都可用，它单纯的将当前位置置0，同时取消mark标记，仅此而已；也就是说写模式下limit仍保持与Buffer容量相同，只是重头写而已；读模式下limit仍然与rewind()调用之前相同
        System.out.println("--------------rewind()----------------------");
        byteBuffer.rewind();
        System.out.println(byteBuffer.position());
        System.out.println(byteBuffer.limit());
        System.out.println(byteBuffer.capacity());

        //clear（）：清空缓冲区，但是缓冲区中的数据仍然存在，但是处于“被遗忘”状态，position和limit被设置为 初始状态
        System.out.println("--------------clear()----------------------");
        byteBuffer.clear();
        System.out.println(byteBuffer.position()); //0
        System.out.println(byteBuffer.limit()); //1024
        System.out.println(byteBuffer.capacity());//1024

    }

    @Test
    public void test2() {

        String s = new String("abcde");
        ByteBuffer buf = ByteBuffer.allocate(1024);
        buf.put(s.getBytes());
        buf.flip();
        byte[] bytes = new byte[buf.limit()];
        buf.get(bytes, 0, 2);
        System.out.println(new String(bytes, 0, 2));
        System.out.println(buf.position());

        //mark()：标记
        buf.mark();
        buf.get(bytes, 2, 2);
        System.out.println(new String(bytes, 2, 2));
        System.out.println(buf.position());
        buf.reset();
        System.out.println("-------------------reset-----------------------");
        System.out.println(buf.position());
        System.out.println("--------------------clear------------------------");
        buf.clear();
        System.out.println(buf.position());
        System.out.println(buf.limit());
        System.out.println(buf.capacity());
    }
```

* 直接缓冲区和非直接缓冲区

  * 直接缓冲区：通过allocateDirect（）方法分配直接缓冲区，将缓冲区建立在物理内存中；可以提高效率

    * 不需要从从用户地址空间copy到内核空间；直接由物理内存面向磁盘进行读写操作，提高了效率
    * 缺点：耗费物理内存资源，物理内存的数据不受程序控制由操作系统决定何时存储

    ![](https://i.loli.net/2020/12/09/yIJKsvRqCtwLpgF.png)

  ![](https://i.loli.net/2020/12/08/KjRTQOhnVExbmUH.png)						

  * 非直接缓冲区：通过allocate（）方法分配缓冲区，将缓冲区建立在JVM虚拟机的内存中；

  ![](https://i.loli.net/2020/12/08/CvAMK1Vs7e3BuWJ.png)

* 区别：
  * 字节缓冲区要么是直接的，要么是非直接的。如果为直接字节缓冲区，则 Java 虚拟机会尽最大努力直接在此缓冲区上执行本机 I/O 操作。也就是说，在每次调用基础操作系统的一个本机 I/O 操作之前（或之后），虚拟机都会尽量避免将缓冲区的内容复制到中间缓冲区中（或从中间缓冲区中复制内容）。
  * 直接字节缓冲区可以通过调用此类的 allocateDirect() 工厂方法来创建。此方法返回的缓冲区进行分配和取消分配所需成本通常高于非直接缓冲区。直接缓冲区的内容可以驻留在常规的垃圾回收堆之外，因此，它们对应用程序的内存需求量造成的影响可能并不明显。所以，建议将直接缓冲区主要分配给那些易受基础系统的本机 I/O 操作影响的大型、持久的缓冲区。一般情况下，最好仅在直接缓冲区能在程序性能方面带来明显好处时分配它们。
  * 直接字节缓冲区还可以通过 FileChannel 的 map() 方法 将文件区域直接映射到内存中来创建。该方法返回MappedByteBuffer 。 Java 平台的实现有助于通过 JNI 从本机代码创建直接字节缓冲区。如果以上这些缓冲区中的某个缓冲区实例指的是不可访问的内存区域，则试图访问该区域不会更改该缓冲区的内容，并且将会在访问期间或稍后的某个时间导致抛出不确定的异常。
  * 字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其 isDirect() 方法来确定。提供此方法是为了能够在性能关键型代码中执行显式缓冲区管理。

```java
    @Test
    public void test3(){
        //创建非直接缓冲区
        ByteBuffer allocate = ByteBuffer.allocate(1024);
        System.out.println(allocate.isDirect()); //false
        //创建直接缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(1024);
        System.out.println(byteBuffer.isDirect());//true

    }
```


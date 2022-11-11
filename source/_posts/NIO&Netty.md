---
title: NIO&Netty
img: https://pic1.zhimg.com/v2-9cd48539cd2c998fd21c6b24086b78de_1440w.jpg?source=172ae18b
categories: Java
---


### NIO

> [demo直达](https://github.com/xmxe/demo/tree/master/study-demo/src/main/java/com/xmxe/study_demo/nio)

NIO主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector(选择区) 。
IO是面向流的，NIO是面向缓冲区的。传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。


#### Buffer

NIO中的关键Buffer实现有：ByteBuffer, CharBuffer, DoubleBuffer,FloatBuffer,IntBuffer, LongBuffer, ShortBuffer。分别对应基本数据类型: byte, char,double, float,int, long, short。 
NIO中还有MappedByteBuffer, HeapByteBuffer,DirectByteBuffer等缓冲区,本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。 

Buffer顾名思义：缓冲区，实际上是一个容器，一个连续数组。Channel提供从文件、网络读取数据的渠道，但是读写的数据都必须经过Buffer.
向Buffer中写数据：从Channel到Buffer:fileChannel.read(buf);通过Buffer的put()方法 buf.put()
从Buffer中读取数据：从Buffer到Channel(channel.write(buf)) 使用get()方法 buf.get()

JAVA处理大文件，一般用BufferedReader,BufferedInputStream这类带缓冲的IO类，不过如果文件超大的话，更快的方式是采用MappedByteBuffer。MappedByteBuffer是NIO引入的文件内存映射方案，读写性能极高。NIO最主要的就是实现了对异步操作的支持。其中一种通过把一个套接字通道(SocketChannel)注册到一个选择器(Selector)中,不时调用后者的选择(select)方法就能返回满足的选择键(SelectionKey),键中包含了SOCKET事件信息。这就是select模型。
FileChannel提供了map方法来把文件影射为内存映像文件： MappedByteBuffer map(int mode,long position,long size); 可以把文件的从position开始的size大小的区域映射为内存映像文件，mode指出了可访问该内存映像文件的方式:
- READ_ONLY（只读）:试图修改得到的缓冲区将导致抛出 ReadOnlyBufferException.(MapMode.READ_ONLY)
- READ_WRITE（读/写）:对得到的缓冲区的更改最终将传播到文件；该更改对映射到同一文件的其他程序不一定是可见的。 (MapMode.READ_WRITE)
- PRIVATE（专用）:对得到的缓冲区的更改不会传播到文件，并且该更改对映射到同一文件的其他程序也不是可见的；相反，会创建缓冲区已修改部分的专用副本。 (MapMode.PRIVATE)


MappedByteBuffer是ByteBuffer的子类，其扩充了三个方法：
- force()：缓冲区是READ_WRITE模式下，此方法对缓冲区内容的修改强行写入文件；
- load()：将缓冲区的内容载入内存，并返回该缓冲区的引用；
- isLoaded()：如果缓冲区的内容在物理内存中，则返回真，否则返回假

```java
public void mappedByteBuffer(){
    File file = new File("D://data.txt");
    long len = file.length();
    byte[] ds = new byte[(int) len];
    try (RandomAccessFile randomAccessFile = new RandomAccessFile(file, "r");){
        MappedByteBuffer mappedByteBuffer = randomAccessFile.getChannel().map(FileChannel.MapMode.READ_ONLY, 0, len);
        for (int offset = 0; offset < len; offset++) {
            byte b = mappedByteBuffer.get();
            ds[offset] = b;
        }
        Scanner scan = new Scanner(new ByteArrayInputStream(ds)).useDelimiter(" ");
        while (scan.hasNext()) {
        	System.out.print(scan.next() + " ");
        }
    } catch (Exception e) {
       e.printStackTrace();
    }
        
/**
* map过程
* FileChannel提供了map方法把文件映射到虚拟内存，通常情况可以映射整个文件，如果文件比较大，可以进行分段映射。
* FileChannel中的几个变量：
*
* MapMode mode：内存映像文件访问的方式，共三种：
* MapMode.READ_ONLY：只读，试图修改得到的缓冲区将导致抛出异常。
* MapMode.READ_WRITE：读/写，对得到的缓冲区的更改最终将写入文件；但该更改对映射到同一文件的其他程序不一定是可见的。
* MapMode.PRIVATE：私用，可读可写,但是修改的内容不会写入文件，只是buffer自身的改变，这种能力称之为”copy on write”。
*
* position：文件映射时的起始位置。
* allocationGranularity：Memory allocation size for mapping buffers，通过native函数initIDs初始化。
*/
}
```

- capacity(缓冲区数组的总长度 即可以容纳的最大数据量；在缓冲区创建时被设定并且不能改变)。capacity作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

- position(下一个要操作的数据元素的位置,下一个要被读或写的元素的索引，每次读写缓冲区数据时都会改变改值，为下次读写作准备),当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后，position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。

- limit(缓冲区数组中不可操作的下一个元素的位置：limit<=capacity表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作。且极限是可以修改的)。limit在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。当切换Buffer到读模式时，limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）

- mark(标记，调用mark()来设置mark=position，再调用reset()可以让position恢复到标记的位置)
mark <= position <= limit <= capacity。position和limit的含义取决于Buffer处在读模式还是写模式。不管Buffer处在什么模式，capacity的含义总是一样的。

##### Buffer常用方法
```java
allocate(int capacity)//从堆空间中分配一个容量大小为capacity的byte数组作为缓冲区的byte数据存储器
allocateDirect(int capacity)//是不使用JVM堆栈而是通过操作系统来创建内存块用作缓冲区，它与当前操作系统能够更好的耦合，因此能进一步提高I/O操作速度。但是分配直接缓冲区的系统开销很大，因此只有在缓冲区较大并长期存在，或者需要经常重用时，才使用这种缓冲区
wrap(byte[] array)//这个缓冲区的数据会存放在byte数组中，bytes数组或buff缓冲区任何一方中数据的改动都会影响另一方。其实ByteBuffer底层本来就有一个bytes数组负责来保存buffer缓冲区中的数据，通过allocate方法系统会帮你构造一个byte数组
wrap(byte[] array, int offset, int length) //在上一个方法的基础上可以指定偏移量和长度，这个offset也就是包装后byteBuffer的position，而length呢就是limit-position的大小，从而我们可以得到limit的位置为length+position(offset)
limit(), limit(10)等//其中读取和设置这4个属性的方法的命名和jQuery中的val(),val(10)类似，一个负责get，一个负责set
reset()//把position设置成mark的值，相当于之前做过一个标记，现在要退回到之前标记的地方
clear()//position = 0;limit = capacity;mark = -1;  有点初始化的味道，但是并不影响底层byte数组的内容
flip()//limit = position;position = 0;mark = -1;  翻转，也就是让flip之后的position到limit这块区域变成之前的0到position这块，翻转就是将一个处于存数据状态的缓冲区变为一个处于准备取数据的状态 一般在从Buffer读出数据前调用。
rewind()//把position设为0，mark设为-1，不改变limit的值 一般在把数据重写入Buffer前调用 或者重新读取
remaining()//return limit - position;返回limit和position之间相对位置差
hasRemaining()//return position < limit返回是否还有未读内容
compact()//把从position到limit中的内容移到0到limit-position的区域内，position和limit的取值也分别变成limit-position、capacity。如果先将positon设置到limit，再compact，那么相当于clear()
get()//相对读，从position位置读取一个byte，并将position+1，为下次读写作准备
get(int index)//绝对读，读取byteBuffer底层的bytes中下标为index的byte，不改变position
get(byte[] dst, int offset, int length)//从position位置开始相对读，读length个byte，并写入dst下标从offset到offset+length的区域
put(byte b)//相对写，向position的位置写入一个byte，并将postion+1，为下次读写作准备
put(int index, byte b)//绝对写，向byteBuffer底层的bytes中下标为index的位置插入byte b，不改变position
put(ByteBuffer src)//用相对写，把src中可读的部分（也就是position到limit）写入此byteBuffer
put(byte[] src, int offset, int length)//从src数组中的offset到offset+length区域读取数据并使用相对写写入此byteBuffer
ByteOrder order()//检索此缓冲区的字节顺序。
ByteBuffer order(ByteOrder bo)//修改缓冲区的字节顺序。
ByteBuffer putInt(int value)//编写int值的相对put方法（可选操作） 。以当前字节顺序将包含给定int值的四个字节写入当前位置的缓冲区，然后将位置递增四。
byte[] array()//返回支持此缓冲区的字节数组（可选操作） 。对此缓冲区内容的修改将导致返回的数组的内容被修改，反之亦然。在调用此方法之前调用hasArray方法，以确保此缓冲区具有可访问的后台阵列。
    
Buffer.mark()
//通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position
//可以使用equals()和compareTo()方法两个Buffer。
equals()
//当满足下列条件时，表示两个Buffer相等：有相同的类型（byte、char、int等）。Buffer中剩余的byte、char等的个数相等。Buffer中所有剩余的byte、char等都相同。如你所见，equals只是比较Buffer的一部分，不是每一个在它里面的元素都比较。实际上，它只比较Buffer中的剩余元素。
compareTo()方法
//compareTo()方法比较两个Buffer的剩余元素(byte、char等)， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：第一个不相等的元素小于另一个Buffer中对应的元素 。所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。
```


#### Channel

NIO中的Channel的主要实现有： FileChannel(从文件中读写数据) 、DatagramChannel(通过UDP读写网络中的数据)、SocketChannel(通过TCP读写网络中的数据)、ServerSocketChannel(可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel)

```java
    /**
    * NIO读取文件 RandomAccessFile进行操作，也可以通FileInputStream.getChannel()获取channel进行操作
    */
    public void newIORead() {
        RandomAccessFile aFile = null;
        try {
            aFile = new RandomAccessFile("src/nio.txt", "rw");
            FileChannel fileChannel = aFile.getChannel();
            // buffer分配空间 根据Buffer实现类表明分配时的单位 下面代表分配1024个字节 CharBuffer就代表1024个字符
            ByteBuffer buf = ByteBuffer.allocate(1024);
            // 通道必须结合Buffer使用，不能直接向通道中读/写数据，
            // read()表示读channel数据写入到buffer，write()表示读取buffer数据写入到channel。
            int bytesRead = fileChannel.read(buf);
            while (bytesRead != -1) {
                // 在读模式下，可以读取之前写入到buffer的所有数据,调用flip()方法,position设回0，并将limit设成之前的position的值
                // 数据就是从position到limit的数据
                // capacity(缓冲区数组的总长度),position(下一个要操作的数据元素的位置),limit(缓冲区数组中不可操作的下一个元素的位置：limit<=capacity),mark(用于记录当前position的前一个位置或者默认是-1)
                buf.flip();
                // hasRemaining()用于判断当前位置(position)和限制(limit)之间是否有任何元素
                while (buf.hasRemaining()) {
                    System.out.print((char) buf.get());
                }
                // clear()方法：position将被设回0，limit设置成capacity，换句话说，Buffer被清空了，
                // 其实Buffer中的数据并未被清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。
                // 如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。
                // 如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先写些数据，那么使用compact()方法。
                // compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。
                // 现在Buffer准备好写数据了，但是不会覆盖未读的数据。
                buf.compact();
                bytesRead = fileChannel.read(buf);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (aFile != null) {
                    aFile.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * NIO写数据 通过FileChannel写入数据
     */
    public void FileChannelOnWrite() {
        try {
            RandomAccessFile accessFile = new RandomAccessFile("D://file1.txt", "rw");
            FileChannel fc = accessFile.getChannel();
            byte[] bytes = new String("write to file1.txt").getBytes();
            // 获得ByteBuffer的实例 类似allocate(int capacity)方法
            ByteBuffer byteBuffer = ByteBuffer.wrap(bytes);
            // 读取缓冲区数据写入到通道。
            fc.write(byteBuffer);
            // 清空缓存区 使得缓存区可以继续写入数据
            byteBuffer.clear();
            // 缓存区写入内容
            byteBuffer.put(new String(",a good boy").getBytes());
            // 写模式转化读模式
            byteBuffer.flip();
            // 读取缓冲区数据写入到通道。
            fc.write(byteBuffer);
            fc.close();
            accessFile.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中
     * （译者注：这个方法在JDK文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。
     */
    public void testTransferFrom() {
        try {
            RandomAccessFile fromFile = new RandomAccessFile("D://file1.txt", "rw");
            FileChannel fromChannel = fromFile.getChannel();
            RandomAccessFile toFile = new RandomAccessFile("D://file2.txt", "rw");
            FileChannel toChannel = toFile.getChannel();

            long position = 0;
            long count = fromChannel.size();
            toChannel.transferFrom(fromChannel, position, count);

            fromFile.close();
            toFile.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * transferTo()方法将数据从FileChannel传输到其他的channel中
     */
    public static void testTransferTo() {
        try {
            RandomAccessFile fromFile = new RandomAccessFile("D://file1.txt", "rw");
            FileChannel fromChannel = fromFile.getChannel();
            RandomAccessFile toFile = new RandomAccessFile("D://file3.txt", "rw");
            FileChannel toChannel = toFile.getChannel();

            long position = 0;
            long count = fromChannel.size();
            fromChannel.transferTo(position, count, toChannel);
            fromFile.close();
            toFile.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //-----------------------nio socket-------------------------------------

    /**
     * NIO新建socket client
     */
    public void newIOSocketClient() {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        SocketChannel socketChannel = null;
        try {
            // 通过 ServerSocketChannel.open()方法来创建一个新的ServerSocketChannel对象，
            // 该对象关联了一个未绑定ServerSocket的通道.通过调用该对象上的socket()方法可以获取与之关联的ServerSocket。
            socketChannel = SocketChannel.open();
            socketChannel.configureBlocking(false);
            socketChannel.connect(new InetSocketAddress("10.10.195.115", 8080));
            // 为了确定连接是否建立，可以调用finishConnect()的方法。
            if (socketChannel.finishConnect()) {
                int i = 0;
                while (true) {
                    TimeUnit.SECONDS.sleep(1);
                    String info = "I'm " + i++ + "-th information from client";
                    buffer.clear();
                    buffer.put(info.getBytes());
                    buffer.flip();
                    while (buffer.hasRemaining()) {
                        System.out.println(buffer);
                        socketChannel.write(buffer);
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (socketChannel != null) {
                    socketChannel.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * NIO socket server
     * 
     * @throws Exception
     */
    public void NIOSocketServer() throws Exception {
        ServerSocketChannel socketChannel = ServerSocketChannel.open();
        // 非阻塞模式
        socketChannel.configureBlocking(false);
        // socketChannel.socket().bind(new InetSocketAddress(9999));jdk 1.7之前
        socketChannel.bind(new InetSocketAddress(9999));
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        while (true) {
            // 通过 ServerSocketChannel.accept() 方法监听新进来的连接。
            // 在阻塞模式下当 accept()方法返回的时候,它返回一个包含新进来的连接的SocketChannel，否则accept()方法会一直阻塞到有新连接到达。
            // 在非阻塞模式下，在没有新连接的情况下，accept()会立即返回null，该模式下通常不会仅仅只监听一个连接,因此需在while循环中调用
            // accept()方法.
            SocketChannel channel = socketChannel.accept();
            if (channel != null) {
                InetSocketAddress remoteAddress = (InetSocketAddress) channel.getRemoteAddress();
                System.out.println(remoteAddress.getAddress());
                System.out.println(remoteAddress.getPort());
                channel.read(byteBuffer);
                byteBuffer.flip();
                while (byteBuffer.hasRemaining()) {
                    System.out.print((char) byteBuffer.get());
                }
            }
        }
    }

    /**
     * UDP发送方
     * 
     * @throws Exception
     */
    public void DatagramChannelsend() throws Exception {
        // 通过DatagramChannel的open()方法来创建。需要注意DatagramChannel的open（）方法只是打开获得通道，
        // 但此时尚未连接。尽管DatagramChannel无需建立连接（远端连接），但仍然可以通过isConnect（）检测当前的channel是否声明了远端连接地址。
        DatagramChannel channel = DatagramChannel.open();
        ByteBuffer byteBuffer = ByteBuffer.wrap(new String("i 'm client").getBytes());
        // 通过send（）方法将ByteBuffer中的内容发送到指定的SocketAddress对象所描述的地址。在阻塞模式下，调用线程会被阻塞至有数据包被加入传输队列。
        // 非阻塞模式下，如果发送内容为空则返回0，否则返回发送的字节数。发送数据报是一个全有或全无(all-or-nothing)的行为。
        // 如果传输队列没有足够空间来承载整个数据报，那么什么内容都不会被发送。
        // 请注意send()方法返回的非零值并不表示数据报到达了目的地，仅代表数据报被成功加到本地网络层的传输队列。
        // 此外，传输过程中的协议可能将数据报分解成碎片，被分解的数据报在目的地会被重新组合起来，接收者将看不到碎片。
        // 但是，如果有一个碎片不能按时到达，那么整个数据报将被丢弃。分解有助于发送大数据报，但也会会造成较高的丢包率。
        int bytesSent = channel.send(byteBuffer, new InetSocketAddress("127.0.0.1", 9999));
        System.out.println("bytesSent="+bytesSent);
    }

    /**
     * UDP接收方
     */
    public void receiveData() throws IOException {
        DatagramChannel channel = DatagramChannel.open();
        channel.socket().bind(new InetSocketAddress(9999));
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        byteBuffer.clear();
        // 通过receive（）方法接受DatagramChannel中数据。从该方法将传入的数据报的数据将被复制到ByteBuffer中，
        // 同时返回一个SocketAddress对象以指出数据来源。在阻塞模式下，receive（）将会阻塞至有数据包到来，
        // 非阻塞模式下，如果没有可接受的包则返回null。如果包内的数据大小超过缓冲区容量时，多出的数据会被悄悄抛弃
        SocketAddress address = channel.receive(byteBuffer);// receive data
        System.out.println(address);
        byteBuffer.flip();
        while (byteBuffer.hasRemaining()) {
            System.out.print((char) byteBuffer.get());
        }
    }
```

##### Channel常用方法 

```java
int read(ByteBuffer dst) //从Channel到中读取数据到ByteBuffer 
long read(ByteBuffer[] dsts) //将Channel到中的数据“分散”到ByteBuffer[] 
int write(ByteBuffer src) //将ByteBuffer到中的数据写入到Channel 
long write(ByteBuffer[] srcs) //将ByteBuffer[]到中的数据“聚集”到Channel 
long position() //返回此通道的文件位置 
FileChannel position(long p) //设置此通道的文件位置 
long size() //返回此通道的文件的当前大小 
FileChannel truncate(long s) //将此通道的文件截取为给定大小 
void force(boolean metaData) //强制将所有对此通道的文件更新写入到存储设备中
```

#### Selector

Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）因此，单个线程可以监听多个数据通道。

Selector的创建
```java
Selector selector = Selector.open();
```
为了将Channel和Selector配合使用，必须将Channel注册到Selector上，通过SelectableChannel.register()方法来实现，沿用nio创建socket
```java
ServerSocketChannel channel = ServerSocketChannel.open(); 
channel.socket().bind(new InetSocketAddress(PORT)); 
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_ACCEPT);
```
与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。
注意register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

1. Connect连接 
2. Accept接收 
3. Read读
4. Write写 

这四种事件用SelectionKey的四个常量来表示：1.SelectionKey.OP_CONNECT 2. SelectionKey.OP_ACCEPT 3. SelectionKey.OP_READ 4.SelectionKey.OP_WRITE

**SelectionKey**
一个SelectionKey键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系。
当向Selector注册Channel时，register()方法会返回一个SelectionKey对象。这个对象包含了一些你感兴趣的属性
interest集合：就像向Selector注册通道一节中所描述的，interest集合是你所选择的感兴趣的事件集合。可以通过SelectionKey读写interest集合。
ready集合：通道已经准备就绪的操作的集合。在一次选择(Selection)之后，你会首先访问这个ready set。可以这样访问ready集合： int readySet = selectionKey.readyOps();
可以用像检测interest集合那样的方法，来检测channel中什么事件或操作已经就绪。但是，也可以使用以下四个方法，它们都会返回一个布尔类型：

```java
selectionKey.isAcceptable(); 
selectionKey.isConnectable();
selectionKey.isReadable(); 
selectionKey.isWritable();
```
从SelectionKey访问Channel和Selector很简单:
```java
Channel channel = selectionKey.channel(); 
Selector selector = selectionKey.selector();
```
可以将一个对象或者更多信息附着到SelectionKey上，这样就能方便的识别某个给定的通道。例如，可以附加与通道一起使用的Buffer，或是包含聚集数据的某个对象。使用方法：
```java
selectionKey.attach(theObject); 
Object attachedObj = selectionKey.attachment();
```
还可以在用register()方法向Selector注册Channel的时候附加对象。如： 
```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

**SelectionKey常用方法**
```java
key.attachment(); //返回SelectionKey的attachment，attachment可以在注册channel的时候指定。
key.channel(); //返回该SelectionKey对应的channel。
key.selector(); //返回该SelectionKey对应的Selector。
key.interestOps(); //返回代表需要Selector监控的IO操作的bit mask
key.readyOps(); //返回一个bit mask，代表在相应channel上可以进行的IO操作。

```
通过Selector选择通道一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。

##### select()方法

```java
int select() //阻塞到至少有一个通道在你注册的事件上就绪了
int select(long timeout) //和select()一样，除了最长会阻塞timeout毫秒(参数)。
int selectNow() //不会阻塞，不管什么通道就绪都立刻返回（译者注：此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。
//select()方法返回的int值表示有多少通道已经就绪。亦即自上次调用select()方法后有多少通道变成就绪状态。如果调用select()方法，有一个通道变成就绪状态，返回了1，若再次调用select()方法，如果另一个通道就绪了，它会再次返回1。如果对第一个就绪的channel没有做任何操作，现在就有两个就绪的通道，但在每次select()方法调用之间，只有一个通道就绪了。一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectedKeys()方法，Set selectedKeys = selector.selectedKeys();当向Selector注册Channel时，Channel.register()方法会返回一个SelectionKey对象。这个对象代表了注册到该Selector的通道。注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中。
SelectionKey.channel()//方法返回的通道需要转型成你要处理的类型，如ServerSocketChannel或SocketChannel等。。

```

```java
    static class ServerSelector {
        private static final int BUF_SIZE = 1024;
        private static final int PORT = 8080;
        private static final int TIMEOUT = 3000;

        // public static void main(String[] args){
        //     selector();
        // }

        /**
         * 接收就绪处理
         * @param key
         * @throws Exception
         */
        public void handleAccept(SelectionKey key) throws Exception {
            ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();
            SocketChannel socketChannel = serverSocketChannel.accept();
            socketChannel.configureBlocking(false);
            socketChannel.register(key.selector(), SelectionKey.OP_READ, ByteBuffer.allocateDirect(BUF_SIZE));
        }

        /**
         * 读就绪处理
         * @param key
         * @throws Exception
         */
        public void handleRead(SelectionKey key) throws Exception {
            SocketChannel socketChannel = (SocketChannel) key.channel();
            ByteBuffer buf = (ByteBuffer) key.attachment();
            long bytesRead = socketChannel.read(buf);
            while (bytesRead > 0) {
                buf.flip();
                while (buf.hasRemaining()) {
                    System.out.print((char) buf.get());
                }
                System.out.println();
                buf.clear();
                bytesRead = socketChannel.read(buf);
            }
            if (bytesRead == -1) {
                socketChannel.close();
            }
        }

        /**
         * 写就绪处理
         * @param key
         * @throws Exception
         */
        public void handleWrite(SelectionKey key) throws Exception {
            ByteBuffer buf = (ByteBuffer) key.attachment();
            buf.flip();
            SocketChannel socketChannel = (SocketChannel) key.channel();
            while (buf.hasRemaining()) {
                socketChannel.write(buf);
            }
            buf.compact();
        }

        /**
         * 注册选择器
         */
        public void selector() {
            Selector selector = null;
            ServerSocketChannel serverSocketChannel = null;
            try {
                // 打开选择器
                selector = Selector.open();
                // 打开ServerSocketChannel通道
                serverSocketChannel = ServerSocketChannel.open();
                serverSocketChannel.socket().bind(new InetSocketAddress(PORT));
                // 设置非阻塞模式
                serverSocketChannel.configureBlocking(false);
                // 注册 监听接收
                serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
                while (true) {
                    if (selector.select(TIMEOUT) == 0) {
                        System.out.println("未检测到有通道就绪");
                        continue;
                    }
                    Iterator<SelectionKey> iter = selector.selectedKeys().iterator();
                    while (iter.hasNext()) {
                        SelectionKey key = iter.next();
                        if (key.isAcceptable()) {
                            // 接收就绪
                            handleAccept(key);
                        }
                        if (key.isReadable()) {
                            // 读就绪
                            handleRead(key);
                        }
                        if (key.isWritable() && key.isValid()) {
                            // 写就绪
                            handleWrite(key);
                        }
                        if (key.isConnectable()) {
                            // 连接就绪
                            System.out.println("isConnectable = true");
                        }
                        iter.remove();
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try {
                    if (selector != null) {
                        selector.close();
                    }
                    if (serverSocketChannel != null) {
                        serverSocketChannel.close();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static class ClientSelector {

        /*标识数字*/
        private static int flag = 0;
        /*缓冲区大小*/
        private static int BLOCK = 4096;
        /*接受数据缓冲区*/
        private static ByteBuffer sendbuffer = ByteBuffer.allocate(BLOCK);
        /*发送数据缓冲区*/
        private static ByteBuffer receivebuffer = ByteBuffer.allocate(BLOCK);
        /*服务器端地址*/
        private final static InetSocketAddress SERVER_ADDRESS = new InetSocketAddress(
                "localhost", 8888);
    
        // public static void main(String[] args) {
        //     selector();
        // }
        public void selector() throws Exception{
            // 打开socket通道
            SocketChannel socketChannel = SocketChannel.open();
            // 设置为非阻塞方式
            socketChannel.configureBlocking(false);
            // 打开选择器
            Selector selector = Selector.open();
            // 注册连接服务端socket动作
            socketChannel.register(selector, SelectionKey.OP_CONNECT);
            // 连接
            socketChannel.connect(SERVER_ADDRESS);
    
            int count = 0;
            while (true) {
                //选择一组键，其相应的通道已为 I/O 操作准备就绪。
                //此方法执行处于阻塞模式的选择操作。
                int selctorCount = selector.select();
                if(selctorCount <= 0) continue;
                //返回此选择器的已选择键集。
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                //System.out.println(selectionKeys.size());
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                while (iterator.hasNext()) {
                    SelectionKey selectionKey = iterator.next();
                    if (selectionKey.isConnectable()) {
                        System.out.println("client connect");
                        SocketChannel client = (SocketChannel) selectionKey.channel();
                        // 判断此通道上是否正在进行连接操作。
                        // 完成套接字通道的连接过程。
                        if (client.isConnectionPending()) {
                            client.finishConnect();
                            System.out.println("完成连接!");
                            sendbuffer.clear();
                            sendbuffer.put("Hello,Server".getBytes());
                            sendbuffer.flip();
                            client.write(sendbuffer);
                        }
                        client.register(selector, SelectionKey.OP_READ);
                    } else if (selectionKey.isReadable()) {
                        SocketChannel client = (SocketChannel) selectionKey.channel();
                        //将缓冲区清空以备下次读取
                        receivebuffer.clear();
                        //读取服务器发送来的数据到缓冲区中
                        count = client.read(receivebuffer);
                        if(count > 0){
                            String receiveText = new String( receivebuffer.array(),0,count);
                            System.out.println("客户端接受服务器端数据--:"+receiveText);
                            client.register(selector, SelectionKey.OP_WRITE);
                        }
    
                    } else if (selectionKey.isWritable()) {
                        sendbuffer.clear();
                        SocketChannel client = (SocketChannel) selectionKey.channel();
                        String sendText = "message from client--" + (flag++);
                        sendbuffer.put(sendText.getBytes());
                        //将缓冲区各标志复位,因为向里面put了数据标志被改变要想从中读取数据发向服务器,就要复位
                        sendbuffer.flip();
                        client.write(sendbuffer);
                        System.out.println("客户端向服务器端发送数据--："+sendText);
                        client.register(selector, SelectionKey.OP_READ);
                    }
                }
                selectionKeys.clear();
            }
        }
    }

```

[NIO之Selector选择器](https://www.cnblogs.com/snailclimb/p/9086334.html)

### Netty

> [demo直达](https://github.com/xmxe/demo/tree/master/study-demo/src/main/java/com/xmxe/study_demo/nio/netty)

[让Netty“榨干”你的CPU](https://mp.weixin.qq.com/s/uMak8HmIyl78hcEUtovAvg)
[Netty 简易实战，傻瓜都能看懂！](//https://mp.weixin.qq.com/s/W-KZFn40FnwIksP1zQP4RQ)
[Java NIO？看这一篇就够了！](https://blog.csdn.net/forezp/article/details/88414741)
[NIO系列教程](https://ifeve.com/overview/)
[Netty 底层的 IO 模型是什么？](https://mp.weixin.qq.com/s/jpWIhws9b2ASD5k6A1pBvg)
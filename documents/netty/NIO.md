JDK 1.4 开始引入。

Java io最核心的概念是流(Stream)，面向流编程

一个流不可能既是输入流又是输出流，要么是输入流，要么是输出流

Java nio核心概念：selector，channel，buffer，面向块(block)或者缓冲区(buffer)编程
> - buffer本身是一块内存，底层实现上是个数组，数据的读写都是通过buffer实现的
> - buffer既能读数据，又能写入数据.flip方法实现读写的反转(limit,capacity,position)
> - 除了数组外，buffer还提供了数据的结构化访问方式，并且可以追踪到系统的读写过程。Java中7中原生类型都有对应的buffer类型，比如IntBuffer,LongBuffer等，没有BooleanBuffer.

Channel 指的是向其写入或者读取数据的对象，类似于java.io中的stream，但是所有数据的读写都是通过buffer进行，永远不会出现直接向channel写入或读取数据的情况。与stream不同的是，channel是双向的。

由于channel是双向的，因此，其更能反应底层操作系统的真是情况，在Linux系统中，底层操作系统的通道就是双向的。

mark：对当前已经被读或写的位置进行标记，后续继续读写后，调用reset方法，又能重新从mark的位置开始读写。mark不会改变position

0 <= mark <= position <= limit <= capacity


slice方法生成的buffer对象与原buffer共享数据，任何一个修改数据，都将引起另一个的数据修改

一个普通的buffer可以在任何时刻转成一个readOnly，调用 asReadOnly 方法，但是只读的不能变成普通buffer



内存映射文件：MappedByteBuffer，通过fileChannel.map方法得到内存映射文件，内存是直接内存，但与堆内存有一个映射，可以直接操作堆内存来实现堆外内存的修改，由操作系统替我们进行同步。
> filechannel.lock 可以获得文件锁

scattering  与  gathering
> - 平时处理都是读写一个buffer，scattering可以传入一个buffer数组，一个满了，处理下一个

	public static void main(String[] args) throws IOException {
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        InetSocketAddress address = new InetSocketAddress(8999);
        serverSocketChannel.socket().bind(address);

        int messageLength = 2 + 3 + 4;
        ByteBuffer[] buffers = new ByteBuffer[3];
        buffers[0] = ByteBuffer.allocate(2);
        buffers[1] = ByteBuffer.allocate(3);
        buffers[2] = ByteBuffer.allocate(4);

        SocketChannel socketChannel = serverSocketChannel.accept();

        while (true) {
            int byteRead = 0;
            while (byteRead < messageLength) {
                long r = socketChannel.read(buffers);
                byteRead += r;
                System.out.println("byteread: " + byteRead);

                Arrays.asList(buffers).stream()
                        .map(buffer -> String.format("position: %d, limit: %d",
                                buffer.position(), buffer.limit()))
                        .forEach(System.out::println);
            }

            Arrays.asList(buffers).stream()
                    .forEach(buffer -> buffer.flip());

            int byteWrite = 0;
            while (byteWrite < messageLength) {
                long w = socketChannel.write(buffers);
                byteWrite += w;
            }

            Arrays.asList(buffers).stream()
                    .forEach(buffer -> buffer.clear());

            System.out.println(String.format("byteread: %d, bytewrite: %d, messagelength: %d",
                    byteRead, byteWrite, messageLength));
        }
    }




### 零拷贝
传统的IO模型，实现将磁盘的数据发送到网络通道，需要经历以下步骤

- 从用户空间发起读取命令，切换至内核空间
- 内核空间调用操作系统命令，将磁盘的文件读到内核缓存
- 将数据从内核缓存拷贝至用户缓存，切换至用户空间
- 用户空间发起写命令，将数据从用户缓存拷贝至内核缓存，并切换至内核空间
- 数据从内核缓存拷贝至socket缓存，从socket缓存发送数据
- 返回结果到用户空间，切换到用户空间

netty实际上的零拷贝

- 涉及的空间切换为：用户空间 -> 内核空间 -> 用户空间
- 从磁盘通过DMA将数据读到内核缓存，socket缓存保存内核缓存的地址以及数据长度(文件描述符,linux 2.4之后支持)，不实际进行数据的拷贝，发送数据时，从socket缓存读取数据，实际是拿到内核缓存的地址和长度，直接从内核缓存读取数据发送



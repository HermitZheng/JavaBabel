

## thrift 框架

RPC(Remote Procedure Call)：远程过程调用。他一种通过网络从远程计算机程序上请求

服务，而不需要了解底层网络技术的协议。



Thrift是一个跨语言的服务部署框架，最初由Facebook于2007年开发，2008年进入Apache开源项目。Thrift通过一个中间语言(IDL, 接口定义语言)来**定义RPC的接口和数据类型**，然后通过一个**编译器生成不同语言的代码**（目前支持C++,Java, Python, PHP, Ruby……共28种语言）, 并为 **RPC 服务的 Server 端和 Client 端直接生成了可用代码**。

Thrift支持众多通讯协议：

- TBinaryProtocol – 一种简单的二进制格式，简单，但没有为空间效率而优化。比文本协议处理起来更快，但更难于[调试](https://zh.wikipedia.org/wiki/调试)。
- TCompactProtocol – 更紧凑的二进制格式，处理起来通常同样高效。
- TJSONProtocol – 使用[JSON](https://zh.wikipedia.org/wiki/JSON)对数据编码。
- TSimpleJSONProtocol – 一种只写协议，它不能被Thrift解析，因为它使用JSON时丢弃了元数据。适合用脚本语言来解析。[[8\]](https://zh.wikipedia.org/wiki/Thrift#cite_note-8)

支持的*传输协议*有：

- TFileTransport – 该传输协议会写文件。
- TFramedTransport – 当使用一个非阻塞服务器时，要求使用这个传输协议。它按帧来发送数据，其中每一帧的开头是长度信息。
- TBufferedTransport--带缓存区的传输，维护一个 buffer，减少底层 Transport 的读写次数。
- TMemoryTransport – 使用[存储器映射输入输出](https://zh.wikipedia.org/wiki/存储器映射输入输出)。（Java的实现使用了一个简单的`ByteArrayOutputStream`。）
- TSocket – 使用阻塞的套接字I/O来传输。
- TZlibTransport – 用[zlib](https://zh.wikipedia.org/wiki/Zlib)执行压缩。用于连接另一个传输协议。

Thrift还提供众多的服务器，包括：

- TNonblockingServer – 一个多线程服务器，它使用非阻塞I/O（Java的实现使用了[NIO](https://zh.wikipedia.org/wiki/Java_NIO)通道）。TFramedTransport必须跟这个服务器配套使用。
- TSimpleServer – 一个单线程服务器，它使用标准的阻塞I/O。测试时很有用。
- TThreadPoolServer – 一个多线程服务器，它使用标准的阻塞I/O。


链接：https://www.jianshu.com/p/3a79bf355bfe

![img](http://static.zybuluo.com/Yano/e7m2cr7546sefoiw8sjth9yd/image.png)

Thrift network stack

```java
  +-------------------------------------------+
  | Server                                    |
  | (single-threaded, event-driven etc)       |
  +-------------------------------------------+
  | Processor                                 |
  | (compiler generated)                      |
  +-------------------------------------------+
  | Protocol                                  |
  | (JSON, compact etc)                       |
  +-------------------------------------------+
  | Transport                                 |
  | (raw TCP, HTTP etc)                       |
  +-------------------------------------------+
```

**Transport 传输层：传输层负责直接从网络中读取和写入数据，它定义了具体的网络传输协议；比如说TCP/IP传输等。**

`Transport layer`提供了一个从网络IO读写的简单抽象，可以使Thrift与底层解耦。open、close、read、write、flush

除了Transport接口，还有一个`ServerTransport`，用来在server端创建请求的连接。open、listen、accept、close

**Protocol 序列化：协议层定义了数据传输格式，负责网络传输数据的序列化和反序列化；比如说JSON、XML、二进制数据等。TProtocol主要实现了读写消息的编解码接口。**

`Protocol`定义了传输数据的序列化、反序列化机制（JSON、XML、binary、compact binary等）。

```java
writeMessageBegin(name, type, seq)
writeI16(i16)
name, type, seq = readMessageBegin()
bool = readBool()
```

**Processor 处理器：处理层是由具体的IDL（接口描述语言）生成的，Processor是TServer从Thrift框架转到用户逻辑的关键流程。**

`Processor`封装了读取输入流、写入输出流的能力，其中输入流、输出流都是`Protocol`的对象。接口很简单：

```java
interface TProcessor {
    bool process(TProtocol in, TProtocol out) throws TException
}
```

其中用户需要实现TProcessor接口。

**Server 服务端：整合上述组件，TServer负责接收Client的请求，并将请求转发到Processor进行处理。**

A Server pulls together all of the various features described above:

- Create a transport 生成传输对象
- Create input/output protocols for the transport 为传输对象生成输入输出序列化对象
- Create a processor based on the input/output protocols 在序列化对象基础上生成处理器
- Wait for incoming connections and hand them off to the processor 等待连接并调用处理器



参考[git.apache.org/thrift.git/tutorial/go/src](http://git.apache.org/thrift.git/tutorial/go/src)

```
func main() {
       flag.Usage = Usage
       server := flag.Bool("server", false, "Run server")
       protocol := flag.String("P", "binary", "Specify the protocol (binary, compact, json, simplejson)")
       framed := flag.Bool("framed", false, "Use framed transport")
       buffered := flag.Bool("buffered", false, "Use buffered transport")
       addr := flag.String("addr", "localhost:9090", "Address to listen to")
       secure := flag.Bool("secure", false, "Use tls secure transport")

       flag.Parse()

       var protocolFactory thrift.TProtocolFactory
       switch *protocol {
       case "compact":
              protocolFactory = thrift.NewTCompactProtocolFactory()
       case "simplejson":
              protocolFactory = thrift.NewTSimpleJSONProtocolFactory()
       case "json":
              protocolFactory = thrift.NewTJSONProtocolFactory()
       case "binary", "":
              protocolFactory = thrift.NewTBinaryProtocolFactoryDefault()
       default:
              fmt.Fprint(os.Stderr, "Invalid protocol specified", protocol, "\n")
              Usage()
              os.Exit(1)
       }

       var transportFactory thrift.TTransportFactory
       if *buffered {
              transportFactory = thrift.NewTBufferedTransportFactory(8192)
       } else {
              transportFactory = thrift.NewTTransportFactory()
       }

       if *framed {
              transportFactory = thrift.NewTFramedTransportFactory(transportFactory)
       }

       if *server {
              if err := runServer(transportFactory, protocolFactory, *addr, *secure); err != nil {
                     fmt.Println("error running server:", err)
              }
       } else {
              if err := runClient(transportFactory, protocolFactory, *addr, *secure); err != nil {
                     fmt.Println("error running client:", err)
              }
       }
}
func runClient(transportFactory thrift.TTransportFactory, protocolFactory thrift.TProtocolFactory, addr string, secure bool) error {
       var transport thrift.TTransport
       var err error
       if secure {
              cfg := new(tls.Config)
              cfg.InsecureSkipVerify = true
              transport, err = thrift.NewTSSLSocket(addr, cfg)
       } else {
              transport, err = thrift.NewTSocket(addr)
       }
       if err != nil {
              fmt.Println("Error opening socket:", err)
              return err
       }
       transport = transportFactory.GetTransport(transport)
       defer transport.Close()
       if err := transport.Open(); err != nil {
              return err
       }
       return handleClient(tutorial.NewCalculatorClientFactory(transport, protocolFactory))
}
func runServer(transportFactory thrift.TTransportFactory, protocolFactory thrift.TProtocolFactory, addr string, secure bool) error {
       var transport thrift.TServerTransport
       var err error
       if secure {
              cfg := new(tls.Config)
              if cert, err := tls.LoadX509KeyPair("server.crt", "server.key"); err == nil {
                     cfg.Certificates = append(cfg.Certificates, cert)
              } else {
                     return err
              }
              transport, err = thrift.NewTSSLServerSocket(addr, cfg)
       } else {
              transport, err = thrift.NewTServerSocket(addr)
       }
       
       if err != nil {
              return err
       }
       fmt.Printf("%T\n", transport)
       handler := NewCalculatorHandler()
       processor := tutorial.NewCalculatorProcessor(handler)
       server := thrift.NewTSimpleServer4(processor, transport, transportFactory, protocolFactory)

       fmt.Println("Starting the simple server... on ", addr)
       return server.Serve()
}
```

![image-20210104222610306](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20210104222610306.png)

**1、客户端调用IDL生成的client调用远程方法；**

**2、客户端协议层负责将请求的方法，参数等信息编码；**

**3、****客户端传输层将消息传递给服务端；**

**4、服务端传输层接受请求的消息；**

**5、TServer将请求转交Processor进行处理；**

**6、Processor使用协议层对消息进行解码；**

**7、根据解码的信息，Processor去调用对应的本地方法（Your Code）；**



1、server处理阻塞、非阻塞处理方式？

阻塞/非阻塞：进程/线程需要操作的数据如果尚未就绪，是否妨碍了当前进程/线程的后续操作。同步/异步：数据如果尚未就绪，是否需要等待数据结果。



- TNonblockingServer – 一个多线程服务器，它使用非阻塞I/O（Java的实现使用了[NIO](https://zh.wikipedia.org/wiki/Java_NIO)通道）。TFramedTransport必须跟这个服务器配套使用。
- TSimpleServer – 一个单线程服务器，它使用标准的阻塞I/O。测试时很有用。
- TThreadPoolServer – 一个多线程服务器，它使用标准的阻塞I/O。

![image-20210104222532313](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20210104222532313.png)



## 时序图

#### 6.1 创建Server的时序图

**![image-20210104222645709](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20210104222645709.png)**

使用SwiftServiceRunner.startThriftServer启动服务

在SwiftServiceRunner中会依次调用startThriftServiceImpl，enableSwiftPerfCounter

初始化ThriftServiceProcessor,初始化ThriftServer，然后注册ZK节，最后启动服务。

#### 6.2 创建Client的时序图

![image-20210104222653524](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20210104222653524.png)

使用SwiftClientFactory.getClient获取client

在SwiftClientFactory中，使用alwaysCreateClientProxy获取ClientProxy的实例clientProxy。

使用EndPointChooserImpls.getOrDefault获取endpointpool的选择方式chooser。

调用clientProxy.init初始化clientProxy，最后通过Proxy.newProxyInstance创建clientProxy的代理对象。

#### 6.3 Client调用时序图

![image-20210104222701443](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20210104222701443.png)

调用RPC函数log的流程：

调用log函数时，实际是代理对象的invoke函数，通过getEndpointPool获取该服务的endpointpool，然后通过创建clientProxy对象时候的chooser进行选择，选择一个endpoint。

通过borrowClient获取该endpoint的连接client，但是首先需要通过ProxyClientPool来判断server是Swift Server还是Thrfit Server，调用isNewServer方法，访问服务端的serverInfo服务，如果是Swift Server这个服务是存在的，能正常返回，Thrift Server没有这个服务，会有异常。

如果endpoint是Swift Server会从基于Swift的连接池中获取client连接，如果是Thrift Server会基于Thrift的连接连接池中获取client连接，返回给AlwaysCreateClientProxy，再返回给ClientProxy。通过method.invoke来调用client的log方法并得到结果result，通过returnClient返还client，最后返回调用结果result。
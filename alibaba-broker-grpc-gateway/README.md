RSocket Broker gRPC Gateway
===========================
RSocket Broker的gRPC Gateway，你可以通过gRPC接口访问RSocket服务。

### 工作原理

工作原理非常简单，就是将gRPC请求转换为RSocket请求，中间的桥梁就是Reactive gRPC。具体的步骤如下：

* 编写gRPC的proto文件，gateway支持各种RPC，包括Unary, Server Streaming, Client Streaming和Bidirectional streaming
* 执行"mvn compile"调用protobuf-maven-plugin生成reactor-grpc对应的Reactive服务接口
* 调用GrpcServiceRSocketImplBuilder动态构建gRPC到后端RSocket服务的Spring Bean

```
    @Bean
    public ReactorAccountServiceGrpc.AccountServiceImplBase grpcAccountService(UpstreamManager upstreamManager) throws Exception {
        return GrpcServiceRSocketImplBuilder
                .stub(ReactorAccountServiceGrpc.AccountServiceImplBase.class)
                .upstreamManager(upstreamManager)
                .build();
    }
```

* 启动RSocket Broker gRPC Gateway，gRPC服务就完成发布啦，你可以使用gRPC工具进行测试。

### gRPC Java客户端
对于gRPC服务调用的客户端，我们还是建议使用gRPC Reactive生成的对应Reactor Stub，接口也比较简单，样例代码如下：

```
public class ReactorServiceTest {
    private ManagedChannel channel;
    private ReactorAccountServiceGrpc.ReactorAccountServiceStub stub;

    @BeforeAll
    public void setUp() {
        channel = ManagedChannelBuilder.forAddress("localhost", 50051).usePlaintext().build();
        stub = ReactorAccountServiceGrpc.newReactorStub(channel);
    }

    @Test
    public void testSayHello() throws Exception {
        Mono<GetAccountRequest> request = Mono.just(GetAccountRequest.newBuilder().setId(1).build());
        ...
    }
 }
```

如果你要实现客户端的Load balance，可以结合RSocket Service Broker提供的服务发现客户端。

### Testing Tools
你可以使用以下工具进行gRPC服务测试，.evans.toml已经提供提供，同时justfile中包括grpccurl的调用方法。

* evans: https://github.com/ktr0731/evans  Finish streaming inputting with CTRL-D
* grpcurl: https://github.com/fullstorydev/grpcurl

### Todo

* 根据proto文件生成Reactive服务接口

### References

* Spring Boot starter module for gRPC framework: https://github.com/LogNet/grpc-spring-boot-starter
* Reactive stubs for gRPC: https://github.com/salesforce/reactive-grpc
* gRPC over HTTP2: https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md
* Byte Buddy: https://bytebuddy.net/


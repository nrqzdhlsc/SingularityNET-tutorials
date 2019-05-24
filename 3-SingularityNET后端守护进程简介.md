----

译者：BING

时间：20190524

原文链接：<https://dev.singularitynet.io/docs/concepts/daemon/>

---

> 学习守护进程相关内容，它是如何与SingularityNET市场和以太坊进行交互的。

[SingularityNET守护进程](https://github.com/singnet/snet-daemon)是一个适配器，服务可以通过它来与SingularityNET平台交互。

在软件架构的语境下，守护进程(Daemon)是一个[sidecar代理](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar)(参考sidecar模式) -- 一个部署在核心应用周围的进程（在本文的案例中，AI服务是核心应用），该进程会用来对架构进行抽象，比如日志和配置；也会用来对整个平台进行一定的抽象，如和智能合约的交互，甚至是否使用以太坊区块链的决策也可以抽象出来。Daemon的两个核心抽象任务是支付和请求翻译。为了准许支付，Dameon会与多方托管智能合约进行交互。在通过SingularityNET调用服务之前，用户必须：

1. 向多方托管合约中存一定的代币，
2. 和服务定义中指定的接受者开启一个支付通道，
3. 每次调用服务Daemon都会检查签名是否为真，
4. 支付通道内是否有足够的代币，
5. 支付通道的终止时间在特定的阈值之上，以确保开发者（服务提供者）能够取回用户消费的代币

在这些检查完成后，请求会发送到服务。Daemon也会跟踪不同客户端的支付状态。



![img](assets/daemon_diagram.jpg)



一旦Daemon完成了请求的验证，它会将请求翻译为AI服务需要的数据格式。 Daemon会暴露一个`gRPC`接口，所有的请求都基于`gRPC`和`protocol buffers`，但是Daemon能将请求翻译为AI服务需要的其他的格式：除了gRPC/Protobuf， 如JSON-RPC等。(注：process fork–based services不是很了解，后续再更新。 )这种翻译使得网络可以采用一种连贯的协议在服务间进行通信。守护进程Daemon和命令行也用`gRPC`和`Protobuf`通信。开发者可以部署多个AI服务的实例，每个实例有它自己的守护进程Daemon，并且所有的守护进程都会在注册中心上注册为访问端点。当多个实例并存时，它们可以被放进一个或多个实例组内（这么做的原因是可以将实例分组到同一个数据中心或者云端上的相同区域）。同一组内的守护进程可以通过[etcd](https://coreos.com/etcd/).共享支付状态。守护进程还提供了一些额外的**部署**相关和面向**管理**的特性：

- SSL终端。这可以由服务开发者提供证书和秘钥来实现，或者由 [Let’s Encrypt](https://letsencrypt.org/)提供服务。
- 日志，使用日志滚动以及可插拔日志钩子。当前邮件钩子已经提供了，其他的钩子也可以通过易用的API实现。
- Metrics, monitoring, and alerts. The daemon collects metrics about request calls, which service owners can use to optimize their resource usage. It also monitors daemon and service events, providing configurable alerts via email or web services.
- 性能度量，监控，警报。守护进程收集了关于请求的很多
- Rate limiting to prevent DoS attacks and to allow service owners to scale at their own speed and ability. The daemon uses the [token bucket](https://en.wikipedia.org/wiki/Token_bucket) algorithm.
- Heartbeat. A pull-based heartbeat service is provided, following the [gRPC health checking protocol](https://github.com/grpc/grpc/blob/master/doc/health-checking.md). The daemon will check that the heartbeat of the service is configured; this is used by monitoring services as well as the Marketplace DApp.

The latest version is , and can be [downloaded from the release page](https://github.com/singnet/snet-daemon/releases).
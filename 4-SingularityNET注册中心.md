----

译者：BING

时间：[20190524]

原文地址：https://dev.singularitynet.io/docs/concepts/registry/

------

> 本篇文章将解释注册合约是如何用来对外暴露关于AI服务的信息的，对外暴露后用户才能发现并购买服务。

SingularityNET注册合约是符合[ERC-165](https://eips.ethereum.org/EIPS/eip-165)的以太坊智能合约，这个类型的智能合约能够存储组织，服务以及类型。AI开发者可以使用注册合约来公布他们提供的服务的细节，同时用户可以使用注册合约来发现他们需要的服务。当用户在[市场DApp](https://beta.singularitynet.io/)上寻找服务时，这个DApp会从注册中心合约上读取服务。注册中心合约支持为服务打标记和类型，使得搜索、过滤成为可能。注册合约会提供**所有**需要的用来与平台上的AI服务交互的信息，要么是列出关于服务的全部信息；要么，当信息很长时，列出IPFS的哈希值。注册合约的源文件，ABI以及部署的信息在 [`singnet/platform-contracts`](https://github.com/singnet/platform-contracts)中可以看到。

### 接口

注册合约的接口，`IRegistry`，是注册合约的完整的说明书。注册合约和它的接口描述同时发布，其中接口合约是在 [`IRegistry.sol`](https://github.com/singnet/platform-contracts/blob/master/contracts/IRegistry.sol)中。接口包含满足[natspec](https://github.com/ethereum/wiki/wiki/Ethereum-Natural-Specification-Format)格式的文档，覆盖了所有的函数。开发者应当导入并指向接口，而不是具体实现。注册合约实现接口描述函数并完全支持[ERC-165](https://eips.ethereum.org/EIPS/eip-165).标准。

### 数据模型

注册合约存储了四类主要数据：组织，服务，类型仓库以及标签。它支持对这四类数据的CRUD操作，还包含一些查看函数以提取数据。

#### Organization

组织像是服务至上的一把伞，它是注册合约中数据层级的最高层。服务开发者能够且应当注册一个组织并且把他们所有的服务都放在这个组织下面。

一个组织的注册记录，需要有一个名字，拥有者的地址，成员的地址集合，服务的集合，以及类型仓库的集合。注册在组织下面的服务和类型仓库可以说是属于组织所有。成员列表是主要的访问管理结构；组织内的成员可以做任何事情，除了修改组织的所有者以及删除组织这类操作。

#### Service服务

一个服务就表示一个AI算法。**它在注册中心上的条目包含了调用这个AI服务所需要的所有信息**。其中条目包含**名字**，**标签**，以及**IPFS哈希值**。名字是用来发现的标识符，标签帮助用户在不知道名字的情况下发现服务，而**IPFS哈希值是指向存储在IPFS系统上的元数据文件链接**。DApps和智能合约能用`listServiceForTag`视图函数来发现服务。

#### 服务的元数据

基于性能以及gas成本的考量，所有的服务元数据链下存储在IPFS上。元数据包括：

- 基本信息如版本号，服务名称，描述等；
- 代码层次的信息，用于调用服务，如编码格式(protobuf或JSON)以及请求格式(gRPC, JSON-RPC或者进程)；
- 一个守护进程端点列表，聚合为一个或多个组；
- 定价信息；
- 一个服务API模型的IPFS哈希值。命令行工具提供了方便的API和库用于操作元数据，具体细节描述在[这里](https://dev.singularitynet.io/docs/concepts/service-metadata)。

#### 类型仓库

类型仓库是注册合约中服务开发者列出服务元数据的一部分，比如服务模型以及用到的数据类型。注册条目包括一个名字、多个标签、一个路径以及一个`URI`。名字和标签都是为了发现服务，而路径是个可选的标识符--用于组织内部管理。URI允许客户端(无论是终端用户还是通过SDK调用服务的应用)发现元数据。DApps和智能合约能用`listTypeRepositoriesForTag`视图函数来发现AI服务。URI和IPFS哈希值，以及主机本身能够用SingularityNET，服务开发者或者任意的IPFS服务来提供，比如[Infura](https://infura.io/).

#### 标签

标签完全是可选但很推荐用来做服务发现用的。服务和类型仓库能与标签联系到一起，通过使用相关的注册合约中的函数`addTagToServiceRegistration`。完成这步后，标签可以用来展示以及在市场DApp中辅助搜索。感谢注册合约中的逆序索引功能，其他的智能合约能够直接在注册中心进行搜索。这也是实现“API们的API”功能的基础，后序还会讨论这个问题。

### DApp整合

SingularityNET应用本质上是个富注册中心浏览器。它加载注册中心并生成UI，可以用来"把玩"注册在注册合约中的服务以及类型仓库。

### 命令行整合

SingularityNET命令行拥有所有的必需工具用来调用注册合约中的方法，可以参考命令行文档来阅读细节。

END.
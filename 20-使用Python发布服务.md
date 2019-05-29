----

原文链接：https://dev.singularitynet.io/tutorials/python/

译者：BING

时间：20190529

----

开始本教程之前，确保已经安装：

- *Docker (https://www.docker.com/)*

- *Metamask (https://metamask.io)*

    需要一个私钥-公钥20对来在SNET上注册服务。开启本教程之前先在MetaMask中生成密钥对。

----

从命令行运行本教程。

我们使用Python gRPC，更多细节参考： https://grpc.io/docs/。

本教程中我们会创建Python服务，并将其发布到SingularityNET上。

## 步骤1

使用提供的Dockerfile文件设置`ubuntu:18.04`镜像。

```bash
SNETD_VERSION="v0.1.7"
docker build \
    --build-arg language=python \
    --build-arg snetd_version=$SNETD_VERSION \
    -t snet_python_service https://github.com/singnet/dev-portal.git#master:/tutorials/docker

ETCD_HOST=$HOME/.snet/etcd/example-python-service/
ETCD_CONTAINER=/opt/singnet/etcd/
docker run -p 7000:7000 -v $ETCD_HOST:$ETCD_CONTAINER -ti snet_python_service bash
```

从现在开始我们使用Docker容器来走剩下的教程。

```bash
cd dev-portal/tutorials/python
```

## 步骤2

为服务项目创建骨架：

```bash
./create_project.sh PROJECT_NAME ORGANIZATION_ID SERVICE_ID SERVICE_PORT
```

`PROJECT_NAME` 是项目的短标签，它将用来命名项目的目录并且会作为`.proto`文件中的命名空间标签。

`ORGANIZATION_ID` 是组织的id值，你也将作为此组织中的一员。

`SERVICE_ID` 服务的id值。

`SERVICE_PORT`服务（本地）要监听的端口号。 

`create_project.sh`将创建目录，名字为 `PROJECT_NAME`，会提供一个空的服务实现。 

在本教程中我们将实现一个服务包含两种方法：

- `int div(int a, int b)`
- `string check(int a)`

我们通过命令行生成项目骨架：

```bash
./create_project.sh tutorial snet math-operations 7070
cd /opt/singnet/tutorial
```

## 步骤3

现在我们将修改骨架代码来实际实现基础服务。我们需啊哟编辑`./service_spec/tutorial.proto`并定义：

- 用户保存方法的输入输出的数据结构，以及
- 服务的RPC格式API

看看这个链接https://developers.google.com/protocol-buffers/docs/overview ，可以帮助你理解关于`.proto`文件的一切。

本教程中我们的`./service_spec/tutorial.proto`长这样：

```protobuf
syntax = "proto3";

package tutorial;

message IntPair {
    int32 a = 1;
    int32 b = 2;
}

message SingleInt {
    int32 v = 1;
}

message SingleString {
    string s = 1;
}

service ServiceDefinition {
    rpc div(IntPair) returns (SingleInt) {}
    rpc check(SingleInt) returns (SingleString) {}
}
```

每个`message`语句定义一个数据结构，用于表示API中的输入或输出。`service`语句定义了RPC API本身。

## 步骤4

为了实际实现我们的API，我们需要编辑`server.py`文件。

Look for `SERVICE_API` and replace `doSomething()` by our actual API methods:

查找`SERVICE_API`并且用实际的API方法替换`doSomething()`。

```python
class ServiceDefinition(pb2_grpc.ServiceDefinitionServicer):
    def __init__(self):
        self.a = 0
        self.b = 0
        self.response = None

    def div(self, request, context):
        self.a = request.a
        self.b = request.b
        self.response = pb2.SingleInt()
        self.response.v = int(self.a / self.b)
        return self.response

    def check(self, request, context):
        self.response = pb2.SingleString()
        self.response.s = "{}".format(request.v)
        return self.response
```

## 步骤5

现在我们编写一个客户端来测试本地服务(不用区块链)，编辑`client.py`文件。

查找`TEST_CODE`并用测试代码替换`doSomething()`。

```python
def doSomething(channel):
    a = 12
    b = 4
    if len(sys.argv) == 3:
        a = int(sys.argv[1])
        b = int(sys.argv[2])
    # Check the compiled proto file (.py) to get method names
    stub = pb2_grpc.ServiceDefinitionStub(channel)
    response = stub.div(pb2.IntPair(a=a, b=b))
    print("{}".format(response.v))
    return response
```

## 步骤6

编译protobuf文件：

```bash
./build.sh
```

## 步骤7

测试本地服务(不用区块链)：

```bash
python3 server.py &
python3 client.py 12 4
```

可以得到下面的输出：

```bash
python3 server.py &

# [1] 4217
# Server listening on 0.0.0.0:7070

python3 client.py 12 4

# 3
```

现在你就成功构建了gPRC Python服务，在容器中任何地方都可调用服务(它们不需要安装目录的任何东西)，或者在容器之外，安装Python gRPC库也可以完成调用。

下一步我们将服务发布到SingularityNET。

## 步骤8

现在必须遵循[发布服务](https://dev.singularitynet.io/tutorials/publish/) 教程来将这个服务发布出去或者使用脚本发布（下一步）。

需要一个`SNET CLI`身份(查看发布服务章节中的步骤3)。

## 步骤9

首先，确保你杀掉步骤7开始的服务进程。

然后发布并开启服务：

```bash
./publishAndStartService.sh PAYMENT_ADDRESS
```

将`PAYMENT_ADDRESS`替换为你自己的公钥(钱包)。

案例:

```bash
./publishAndStartService.sh 0x501e8c58E6C16081c0AbCf80Ce2ABb6b3f91E717
```

这将开启`SNET Daemon`和你的服务。如果事情都进展顺利的话，你可以看到区块链交易记录和后面的信息(分别是从：你的服务和`SNET Daemon`中发出）：

```bash
# [blockchain log]
# Server listening on 0.0.0.0:7070
# [daemon initial log]
# INFO[0002] Blockchain is enabled: instantiate payment validation interceptor 
# INFO[0002]                                               PaymentChannelStorageClient="&{ConnectionTimeout:5s RequestTimeout:3s Endpoints:[http://127.0.0.1:2379]}"
# INFO[0002] Default payment handler registered            defaultPaymentType=escrow
# DEBU[0002] starting daemon                              
```

可以再次确认服务是否正确发布，使用：

```bash
snet organization list-services snet
```

可选的是，你也可以取消发布服务：

```bash
snet service delete snet math-operations
```

实际上，因为这只是教程，你需要在完成测试后取消发布服务。

其他的`snet`命令和选项能在[这里](https://github.com/singnet/snet-cli)找到。

## 步骤10

可以通过下面的命令行发起请求测试服务：

`openChannel.sh`脚本能开启并初始化一个新的支付通道，它将能输出新的通道的id值(这个值会在`testServiceRequest.sh`中使用)：

```bash
./openChannel.sh

# [blockchain log]
# #channel_id
# 10
```

在这个例子中，通道的id是10。

现在可以运行`testServiceRequest.sh VALUE_A VALUE_B`:

```bash
./testServiceRequest.sh 12 4

# [blockchain log]
#   response:
#       v: 3
```

就这些了，记住按照步骤9中提到的方法删除服务。

```bash
snet service delete snet math-operations
```

END.
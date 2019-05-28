----

原文链接：https://dev.singularitynet.io/tutorials/publish/

译者：BING

时间：20190528

----

### 步骤1. 依赖

通过bash终端来运行这个教程。

这个教程中我们发布一个样例服务到SingularityNET，使用的是Ropsten测试网。

我们配置了一个Docker镜像，包括了所有的依赖，但是如果你喜欢可以自己安装依赖。

使用Docker镜像更容易(你不必是个Docker老手就足够跟着这个教程走一遍)。

如果你想自己安装依赖，看看这里的[要求](https://dev.singularitynet.io/docs/setup/requirements) 并直接跳到步骤三。

### 步骤2. 设置Docker容器

在进入下一步之前，确保已经安装：

- *Docker (https://docs.docker.com/install)*

如果你不熟悉Docker，可以先看看官方的新手指引。

注意本教程假定你的用户是docker组的成员，有权限和后台交互。为了确保这点，使用`sudo adduser $USER docker`或者读一下docker的[文档](https://docs.docker.com/install/linux/linux-postinstall/)。

直接从git仓库构建Docker镜像：

```bash
docker build -t snet_publish_service https://github.com/singnet/dev-portal.git#master:/tutorials/docker
```

设置环境变量(后面用到时会解释)：

```bash
ORGANIZATION_ID="$USER"-org
ORGANIZATION_NAME="The $USER's Organization"

SERVICE_ID=example-service
SERVICE_NAME="SNET Example Service"
SERVICE_IP=127.0.0.1
SERVICE_PORT=7000

DAEMON_HOST=0.0.0.0

USER_ID=$USER

# to secure payments
ETCD_HOST=$HOME/.snet/etcd/$SERVICE_ID/
ETCD_CONTAINER=/opt/singnet/etcd/

# to make your snet's configs persistent
SNET_CLI_HOST=$HOME/.snet/
SNET_CLI_CONTAINER=/root/.snet/
```

现在可以基于这个镜像来运行Docker容器了：

```bash
docker run \
    --name MY_SNET_SERVICE \
    -e ORGANIZATION_ID=$ORGANIZATION_ID \
    -e ORGANIZATION_NAME="$ORGANIZATION_NAME" \
    -e SERVICE_ID=$SERVICE_ID \
    -e SERVICE_NAME="$SERVICE_NAME" \
    -e SERVICE_IP=$SERVICE_IP \
    -e SERVICE_PORT=$SERVICE_PORT \
    -e DAEMON_HOST=$DAEMON_HOST \
    -e DAEMON_PORT=$SERVICE_PORT \
    -e USER_ID=$USER_ID \
    -p $SERVICE_PORT:$SERVICE_PORT \
    -v $ETCD_HOST:$ETCD_CONTAINER \
    -v $SNET_CLI_HOST:$SNET_CLI_CONTAINER \
    -ti snet_publish_service bash
```

这将进入到docker容器，后面的教程假定你在docker的容器命令行中工作。

可以使用`ctrl-d`退出，这会停止容器。如果你希望再次进入容器，使用`docker start snet_publish_service`就能从离开的地方重新开始。

### 步骤3. 设置`snet cli`并创建身份

#### 助记词身份设置：

选择你的助记词。`MY_MNEMONIC`是个字符串，用于生成公钥/私钥的密钥对：

```bash
snet identity create $USER_ID mnemonic --mnemonic "MY_MNEMONIC"
```

####  (可选)其他的身份类型选择：

可以使用已知的键创建身份。

`SNET CLI` 支持下面的身份类型：

- key - 16进制的私钥
- rpc - 和JSON-RPC挂你器一同使用
- ledger - 硬件钱包
- trezor -硬件钱包

参考这个链接了解更多关于 [SNET CLI](http://snet-cli-docs.singularitynet.io/)的细节。

### 步骤4. 获取ETH和AGI

需要一些ETH和AGI代币。

首先，通过 `snet account print` 获取账户地址。

然后，使用你的地址能获取Ropsten上的AGI和ETH代币，需要用到Github账户：

- AGI: https://faucet.singularitynet.io/
- ETH: https://faucet.metamask.io/

现在确保你在Ropsten网络上，使用下面的命令：

```bash
snet network ropsten
```

并检查余额，使用命令：

```bash
snet account balance
```

### 步骤5. 创建组织

为了能够发布服务，你需要成为一个组织的所有者或者成员。

你可以用下面的命令创建新的组织：

```bash
snet organization create "$ORGANIZATION_NAME" --org-id $ORGANIZATION_ID -y
```

万一`ORGANIZATION_ID` 已经用过了，可以自己选个新的名字。但要确保遵循了[命名规范指引]([https://github.com/nrqzdhlsc/SingularityNET-tutorials/blob/master/5-%E5%91%BD%E5%90%8D%E6%A0%87%E5%87%86.md](https://github.com/nrqzdhlsc/SingularityNET-tutorials/blob/master/5-命名标准.md))

如果必须用一个不同的`ORGANIZATION_ID`(而不是我们第二步提供的)，就需要正确更新这个值，这会在后面用到。

```bash
export ORGANIZATION_ID="new-org-id"
```

如果你想加入到已经存在的组织，比如`snet`，需要组织的所有者将你的公钥(账户)添加进来。

### 步骤6. 下载并配置example-service

在这个教程中，我们使用这里的[SingularityNET样例服务](https://github.com/singnet/example-service)。

- 复制git仓库

```bash
git clone --depth=1 https://github.com/singnet/example-service.git
cd example-service
```

- 安装依赖并编译protobuf文件：

```bash
pip3 install -r requirements.txt
sh buildproto.sh
```

服务已经可以运行了，但是首先我们需要将服务发布到SingularityNET，并配置`SNET DAEMON`。

### 步骤7. 准备服务元数据发布服务

首先我们需要创建服务元数据。可以运行下面的命令：

```bash
snet service metadata-init SERVICE_PROTOBUF_DIR SERVICE_DISPLAY_NAME PAYMENT_ADDRESS --endpoints SERVICE_ENDPOINT --fixed-price FIXED_PRICE
```

你需要指定下面的几个参数：

- `SERVICE_PROTOBUF_DIR` - 包含你的服务的protobuf文件的目录：在样例服务中是这个目录`service/service_spec/`
- `SERVICE_DISPLAY_NAME` - 展示服务的名字。你可以选择任何你想要的名字。
- `PAYMENT_ADDRESS` - 以太坊账号，用于接收服务的支付。你需要设置为自己的以太坊账户。
- `SERVICE_ENDPOINT` - 端点，用于连接服务。
- `FIXED_PRICE` - 用AGI计费，表示单次调用服务的价格。我们设置价格为$10^{-8}$AGI，记住$10^{-8}AGI = 1COG$

```bash
ACCOUNT=`snet account print`
snet service metadata-init service/service_spec/ "$SERVICE_NAME" $ACCOUNT --endpoints http://$SERVICE_IP:$SERVICE_PORT --fixed-price 0.00000001

# 描述服务并添加一个URL用于进一步了解service的信息
snet service metadata-add-description --json '{"description": "Description of my Service.", "url": "https://service.users.guide"}'
```

这命令会创建一个JSON配置文件：`service_metadata.json`。

关于服务元数据的细节可以参考[这里](https://dev.singularitynet.io/docs/concepts/service-metadata).。

### 步骤8. 发布服务到SingularityNET

现在可以发布服务(`service_metadata.json`文件会隐式使用)了：

```bash
snet service publish $ORGANIZATION_ID $SERVICE_ID -y
```

检查服务是否被正确发布：

```bash
snet organization info $ORGANIZATION_ID
```

### 步骤9. 运行服务(和SNET Daemon)

创建`SNET DAEMON`配置文件，命名为`snetd.config.json`。

```bash
cat > snetd.config.json << EOF
{
   "DAEMON_END_POINT": "$DAEMON_HOST:$DAEMON_PORT",
   "ETHEREUM_JSON_RPC_ENDPOINT": "https://ropsten.infura.io",
   "IPFS_END_POINT": "http://ipfs.singularitynet.io:80",
   "REGISTRY_ADDRESS_KEY": "0x5156fde2ca71da4398f8c76763c41bc9633875e4",
   "PASSTHROUGH_ENABLED": true,
   "PASSTHROUGH_ENDPOINT": "http://localhost:7003",
   "ORGANIZATION_ID": "$ORGANIZATION_ID",
   "SERVICE_ID": "$SERVICE_ID",
   "PAYMENT_CHANNEL_STORAGE_SERVER": {
       "DATA_DIR": "/opt/singnet/etcd/"
   },
   "LOG": {
       "LEVEL": "debug",
       "OUTPUT": {
          "TYPE": "stdout"
       }
   }
}
EOF
```

运行服务，并自动启动`SNET DAEMON`实例。

```bash
python3 run_example_service.py --daemon-config snetd.config.json
```

现在你的服务应该起来了并且可以运行了。

### 步骤10. 通过`SNET CLI`调用服务

开启一个新的终端，如果使用Docker，进入容器，使用：

```bash
docker exec -it MY_SNET_SERVICE bash
```

现在就能用好几个`SNET CLI`命令来与你的Ropsten网络账户交互了（参考 [SNET CLI](http://snet-cli-docs.singularitynet.io/) 了解更多细节）。

检查账户余额，并设置MPE支付通道以调用服务。

```bash
# 检查账户
snet account balance

# 向MPE合约中存储10COG
snet account deposit 0.00000010 -y

# 检查余额 - 10COG移到MPE
snet account balance

# 为服务开启支付通道
snet channel open-init $ORGANIZATION_ID $SERVICE_ID 0.00000010 +10days -y
```

`snet channel open-init`打开并初始化通道，并在通道中为`$ORGANIZATION_ID/SERVICE_ID`存入10个COG，通道的过期时间设置为10天(57600个区块，15秒出一个区块)。这个命令会打印出创建的通道的ID。使用下面的命令可以记录它的使用：

```bash
# 检查余额 - 10个COG移动从MPE移动到通道
snet account balance

# 检查通道余额(CHANNEL_ID通过'snet channel open-init'打印出来)
snet client get-channel-state <CHANNEL_ID> $SERVICE_IP:$SERVICE_PORT
```

调用服务：

```bash
snet client call $ORGANIZATION_ID $SERVICE_ID mul '{"a":12,"b":7}' -y
```

MPE支付通道改变了，用下面的命令查看资金：

```bash
# 1 COG被消费(签名了) 
snet client get-channel-state <CHANNEL_ID> $SERVICE_IP:$SERVICE_PORT
```

现在你已经消费了1个通道中的COG（服务成本在步骤7定义）用来调用服务。你可以持续调用服务，直到MPE通道用完资金。

作为服务提供者，你可以在任何时候取回客户消费的AGI代币，使用下面的命令：

```bash
snet treasurer claim-all --endpoint $SERVICE_IP:$SERVICE_PORT -y

# claimed funds are now in MPE
snet account balance

# 将资金从MPE合约移动到你的账户
snet account withdraw <AMOUNT_IN_AGI> -y
snet account balance
```

### 步骤11. 从MPE通道中取回未花费的资金(可选)

作为服务的用户，在通道过期前你不能取回未花费的资金。

一旦到期了，你就可以取回资金，通过使用`snet channel claim-timeout-all`：

```bash
# 显示通道中花费/未花费的AGI代币
snet client get-channel-state <CHANNEL_ID> $SERVICE_IP:$SERVICE_PORT
snet account balance

# 从所有过期的通道中移动资金到MPE
snet channel claim-timeout-all -y
snet account balance

# 将MPE中的资金一定到用户账户
snet account withdraw <AMOUNT_IN_AGI> -y
snet account balance
```

END.
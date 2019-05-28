-----

原文链接：https://dev.singularitynet.io/docs/development/local-singularitynet/

译者：BING

时间：20190528

-----

本教程会描述如何搭建完全可用的本地SingularityNET环境。你可以发布服务，调用服务并完全控制本地区块链网络用于开发和测试。

## 安装依赖

本文描述了在Ubuntu18.04中设置环境的过程。有些命令可能在其他Linunx发行版中有所不同。

> TIP: 在这里可以找到如何在Docker容器中本地运行SingularityNET平台的[指导](https://dev.singularitynet.io/docs/development/mpe-example1) ，以及如何在容器中运行简单的前端到后端的案例。

### Go工具集

- Go 1.10+
- Dep 0.4.1+
- Go Protobuf编译器
- Golint

部分代码用Go语言写成，所以你需要一些工具来编译Go代码，并且管理Go的依赖。

```bash
sudo apt-get install golang go-dep golang-goprotobuf-dev golint
```

### NodeJS工具集

- NodeJS 8+
- NPM

[Truffle](https://truffleframework.com/truffle)和[Ganache](https://truffleframework.com/ganache)用来开发和测试以太坊合约，因此NodeJS开发工具是需要的。

```bash
sudo apt-get install nodejs npm
```

### IPFS

IPFS用来保存服务的RPC模型，这些模型是通过SingularityNET平台发布的。遵循这里的指引下载并安装[IPFS](https://ipfs.io/docs/install)。下面的步骤会要求`ipfs`已经安装并能通过命令行运行。

### Python工具集

- Python 3.6.5
- Pip

部分代码是由Python来写的，因此需要Python解释器以及Pip作为包管理工具。

```bash
sudo apt-get install python3 python3-pip
```

### 其他

- libudev
- libusb 1.0

```bash
sudo apt-get install libudev-dev libusb-1.0-0-dev
```

## 部署本地环境

### 设置Go语言构建环境

Go编译器需要工作空间的路径暴露到`GOPATH`环境变量下。`SINGNET_REPOS`也暴露出来用于简化改变目录的命令

```python
mkdir -p singnet/src/github.com/singnet
cd singnet
mkdir log
export GOPATH=`pwd`
export SINGNET_REPOS=${GOPATH}/src/github.com/singnet
export PATH=${GOPATH}/bin:${PATH}
```

### 部署IPFS实例

IPFS在SingularityNET是用来保存发布的服务RPC模型。在本地环境下测试我们需要设置一个私有的IPFS实例。

初始化IPFS数据目录：

```bash
export IPFS_PATH=$GOPATH/ipfs
ipfs init
```

从默认的IPFS配置中移除默认的IPFS启动实例(参考IPFS[私有网络](https://github.com/ipfs/go-ipfs/blob/master/docs/experimental-features.md#private-networks))。

```bash
ipfs bootstrap rm --all
```

改变IPFS的API和网关端口，因为它和默认的`example-service`以及`snet-daemon`端口重复。

```bash
ipfs config Addresses.API /ip4/127.0.0.1/tcp/5002
ipfs config Addresses.Gateway /ip4/0.0.0.0/tcp/8081
```

### 配置平台合约

复制平台合约仓库：

```bash
cd $SINGNET_REPOS
git clone https://github.com/singnet/platform-contracts
cd platform-contracts
```

安装Ganache和依赖：

```bash
npm install
npm install ganache-cli
```

使用Truffle编译合约：

```bash
./node_modules/.bin/truffle compile
```

### 设置`snet`命令行交互

复制`snet-cli`仓库：

```bash
cd $SINGNET_REPOS
git clone https://github.com/singnet/snet-cli
cd snet-cli
```

在开发模式下安装区块链依赖以及`snet-cli`包：

```bash
# you need python 3.6 here, with python 3.5 you will get an error
./scripts/blockchain install
pip3 install -e .
```

### 构建`snet-daemon`

复制`snet-daemon`仓库：

```bash
cd $SINGNET_REPOS
git clone https://github.com/singnet/snet-daemon
cd snet-daemon
```

构建`snet-daemon`:

```bash
./scripts/install # install dependencies
./scripts/build linux amd64  # build project
```

## 开启环境并最终化snet配置

### 开启本地IPFS实例

开启IPFS守护进程：

```bash
ipfs daemon >$GOPATH/log/ipfs.log 2>&1 &
```

### 开启本地以太坊网络

开启本地以太坊网络。传递助记词用于生成确定的区块链环境：账户，私钥和行为。

```bash
cd $SINGNET_REPOS/platform-contracts
./node_modules/.bin/ganache-cli --mnemonic 'gauge enact biology destroy normal tunnel slight slide wide sauce ladder produce' >$GOPATH/log/ganache.log 2>&1 &
```

账户和私钥会用在后序的步骤。使用Truffle部署合约。

```bash
./node_modules/.bin/truffle migrate --network local
npm run package-npm
```

部署后打印出来的合约地址将会用在`snet`配置环节。

Truffle部署合约使用测试网络的第一个账户。SingularityNETToken合约是用这个账户来部署的，这个账户的余额会在部署期间保存所有SingularityNET上的代币。其他的待部署的合约是Registry和MultiPartyEscrow。Registry合约保存**组织和发布的服务列表**，MultiPartyEscrow合约是支付系统的一部分。

### 为本地环境配置`snet-cli`

```bash
# 首次运行snet命令创建默认配置
snet

# 添加本地以太坊网络到`snet`配置中，名字用local
snet network create local http://localhost:8545

# 创建第一个身份(snet-user = 第一个ganache身份)
snet identity create snet-user rpc --network local

# 切换到snet-user,将自动切换到本地网络
snet identity snet-user

# 切换到本地ipfs端点
snet set  default_ipfs_endpoint http://localhost:5002

# 为本地网络配置合约地址，kovan地址已经配置过
snet set current_singularitynettoken_at 0x6e5f20669177f5bdf3703ec5ea9c4d4fe3aabd14
snet set current_registry_at            0x4e74fefa82e83e0964f0d9f53c68e03f7298a8b2
snet set current_multipartyescrow_at    0x5c7a4290f6f8ff64c69eeffdfafc8644a4ec3a4e
```

END.
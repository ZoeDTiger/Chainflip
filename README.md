# Chainflip长期测试网Perseverance验证节点部署文档

## 情况说明：

激励测试网Soundcheck以于2022年1月19日结束，奖励在流动性引导池（LBP）建立后向参与者发放；目前为长期性测试网Perseverance，这个阶段没有奖励，但在未来主网启动之前，项目方会考虑这个问题。

## 先决条件：

1、需要两个以太坊账户，第一个账户将用于接收来自龙头的tFLIP和与Staking App交互，以质押和提取资金；第二个账户（验证者钱包/账户，需要提供私钥）将被验证者用来发送交易到Goerli以太坊区块链。

2、钱包需要至少0.1个gETH与10个tFLIP，gETH用于向以太坊区块链提交交易的gas费，tFLIP用于启动验证人节点，成为活跃的验证人则需要更多的tFLIP。

3、准备gETH RPC地址，可以自己运行geth客户端，也可以使用第三方远程客户端，如：https://www.alchemy.com/  、  https://www.infura.io/zh  、  https://rivet.cloud/

4、系统要求：
 
 操作系统：Ubuntu 20.04
 
 CPU：四核
 
 内存：8GB
 
 SSD：50GB
 
 带宽：建议使用1GBps
 
 防火墙端口：30333（TCP）和8078（TCP）

## 部署节点

### 1、通过APT Repo下载二进制文件

#### a、添加Chainflip APT Repo

sudo mkdir -p /etc/apt/keyrings

curl -fsSL repo.chainflip.io/keys/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/chainflip.gpg

#### b、验证密钥的真实性：

gpg --show-keys /etc/apt/keyrings/chainflip.gpg

<img width="386" alt="从终端看到以下输出" src="https://user-images.githubusercontent.com/100336530/207263177-6fc438f6-32ed-4209-b522-6769264ea975.png">

#### c、将Chainflip的Repo添加到apt sources列表中

echo "deb [signed-by=/etc/apt/keyrings/chainflip.gpg] https://repo.chainflip.io/perseverance/ focal main" | sudo tee /etc/apt/sources.list.d/chainflip.list

#### d、安装软件包

sudo apt-get update

sudo apt-get install -y chainflip-cli chainflip-node chainflip-engine

### 2、生成密钥

#### a、创建存储密钥的目录

sudo mkdir /etc/chainflip/keys

#### b、导入以太坊密钥：YOUR_VALIDATOR_WALLET_PRIVATE_KEY替换成你的实际私钥【强烈建议使用新的钱包私钥！！！】

echo -n "YOUR_VALIDATOR_WALLET_PRIVATE_KEY" |  sudo tee /etc/chainflip/keys/ethereum_key_file

#### c、验证者密钥：为了质押该节点，需要生成Chainflip密钥。签名钥匙的公钥（SS58）实际上也是验证人ID，将需要用它来质押和跟踪节点。

生成签名密钥：chainflip-node key generate

<img width="531" alt="包含助记词与SS58" src="https://user-images.githubusercontent.com/100336530/207264317-00e5a06b-8695-49af-b1b2-d259b4749266.png">

#### d、加载签名密钥：将其添加到你的验证器节点中，用你的密钥替换YOUR_CHAINFLIP_SECRET_SEED，不包括前两个字符（0x）保存在文件中

SECRET_SEED=YOUR_CHAINFLIP_SECRET_SEED

echo -n "${SECRET_SEED:2}" | sudo tee /etc/chainflip/keys/signing_key_file

#### e、生成节点密钥：用于验证器之间的安全通信

sudo chainflip-node key generate-node-key --file /etc/chainflip/keys/node_key_file

cat /etc/chainflip/keys/node_key_file

### 3、配置文件

#### a、创建引擎的配置文件

sudo mkdir -p /etc/chainflip/config

sudo vim /etc/chainflip/config/Default.toml

#### b、编辑配置：需要公网IP、gETH的wss与https地址，确保你没有使用主网的RPC。如下配置：

<img width="537" alt="确保你没有使用主网的RPC" src="https://user-images.githubusercontent.com/100336530/207268373-f134be85-b3c3-410b-bd7b-a53ae3eb81d5.png">

### 4、启动

#### a、启动chainflip节点：启动后开始同步区块，区块浏览器上可以找到最新的区块： https://blocks-perseverance.chainflip.io/

sudo systemctl start chainflip-node

#### b、检查节点状态：

sudo systemctl status chainflip-node

#### c、查看节点日志

tail -f /var/log/chainflip-node.log

#### d、启动引擎：节点完成同步后，需要启动 chainflip-engine

sudo systemctl start chainflip-engine

#### e、检查引擎状态

sudo systemctl status chainflip-engine

#### f、查看引擎日志

tail -f /var/log/chainflip-engine.log

<img width="854" alt="提示节点未质押" src="https://user-images.githubusercontent.com/100336530/207483260-857fd3ab-34a5-48a7-a4f1-1a023c15c18e.png">

#### g、设置服务器重启后自动启动

sudo systemctl enable chainflip-node

sudo systemctl enable chainflip-engine

#### h、修改配置文件后需要重启

sudo systemctl restart chainflip-engine

### 5、质押

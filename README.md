# Chainflip长期测试网Perseverance验证节点部署文档

## 情况说明：

激励测试网Soundcheck以于2022年1月19日结束，奖励在流动性引导池（LBP）建立后向参与者发放；目前为长期性测试网Perseverance，这个阶段没有奖励，但在未来主网启动之前，项目方会考虑这个问题。

## 先决条件：

1、需要两个以太坊账户，第一个账户将用于接收来自龙头的tFLIP和与Staking App交互，以质押和提取资金；第二个账户（验证者钱包/账户，需要提供私钥）将被验证者用来发送交易到Goerli以太坊区块链。

目前快速获得tFLIP的方法是通过流动性池：https://tflip-dex.thunderhead.world/

2、钱包需要至少0.1个gETH与10个tFLIP，gETH用于向以太坊区块链提交交易的gas费，tFLIP用于启动验证人节点，成为活跃的验证人则需要更多的tFLIP。

3、准备gETH RPC地址，可以自己运行geth客户端，也可以使用第三方远程客户端，如：https://www.alchemy.com/  、  https://www.infura.io/zh  、  https://rivet.cloud/

4、系统要求：
 
 操作系统：Ubuntu 20.04
 CPU：四核
 内存：8 GB
 磁盘：50 GB固态硬盘
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

#### f、查看引擎日志：初始启动时提示未质押

tail -f /var/log/chainflip-engine.log

<img width="854" alt="提示节点未质押" src="https://user-images.githubusercontent.com/100336530/207483260-857fd3ab-34a5-48a7-a4f1-1a023c15c18e.png">

#### g、设置服务器重启后自动启动

sudo systemctl enable chainflip-node

sudo systemctl enable chainflip-engine

#### h、修改配置文件后需要重启

sudo systemctl restart chainflip-engine

### 5、质押

#### a、通过验证者仪表板进行质押： https://stake-perseverance.chainflip.io/auctions

确保你的Metamask里有tFLIP。合约地址是0x8e71CEe1679bceFE1D426C7f23EAdE9d68e62650

转到 Perseverance Staking App-> "我的节点"

将你的Metamask钱包（质押钱包，即前文提到的"第一个账户"）与tFLIP连接起来

点击按钮 "+添加节点" -> 你应该看到 "注册新节点 "模式

输入你在生成密钥步骤中得到的验证者id —— 你的公钥(SS58) —— 以及你想要质押的tFLIP的数量。点击 "质押"

Metamask会要求你签署两个交易。第一笔是代币批准，第二笔是转让和质押你的tFLIP

一旦你成功质押，跳回你的终端并运行以下命令：sudo systemctl restart chainflip-engine

#### b、注册验证器密钥

节点添加成功后，需要设置验证器用来产生区块的密钥。如果不这样做，验证器将不能被选中并赢得拍卖。要设置密钥可以使用chainflip-cli。

##### 将节点注册为验证器（节点必须是完全同步的）：

sudo chainflip-cli --config-path /etc/chainflip/config/Default.toml register-account-role Validator

<img width="739" alt="节点注册为验证器" src="https://user-images.githubusercontent.com/100336530/207487515-3a12205a-ad68-4dee-9cf3-b66314a67283.png">

##### 激活账户：等待一段时间再提交激活，否则会提示错误

sudo chainflip-cli --config-path /etc/chainflip/config/Default.toml activate

<img width="847" alt="提示错误" src="https://user-images.githubusercontent.com/100336530/207496692-7052126c-b90a-4d37-9d35-fac26f256f2d.png">

##### 轮换验证器密钥：

sudo chainflip-cli --config-path /etc/chainflip/config/Default.toml rotate

##### 设置验证节点名称：

sudo chainflip-cli --config-path /etc/chainflip/config/Default.toml vanity-name my-name

### 6、提取与退出

在拍卖没有进行的时候，可以从最低有效投标中提取多余的资金，也可以退出你的节点，与提取不同，取消抵押将使你的节点和所有抵押的tFLIP退出。

#### a、退出拍卖

这样就可以把节点排除在下一次拍卖之外。在最后一次拍卖结束后，所有剩余的tFLIP都将记入余额，仍然需要按照接下来的步骤来领取这些资金

sudo chainflip-cli --config-path /etc/chainflip/config/Default.toml retire

#### b、提取tFLIP

要从系统中提取tFLIP，需要使用索赔程序。这是一个多步骤的过程，需要与Chainflip CLI和Staking App互动。

##### 申请索赔证书：FLIP_AMOUNT是想提取的tFLIP的金额，ETH_ADDRESS是你想用来接收tFLIP的ETH地址。

sudo chainflip-cli --config-path /etc/chainflip/config/Default.toml claim FLIP_AMOUNT ETH_ADDRESS

申请了证书，继续进入Staking App进行claim。

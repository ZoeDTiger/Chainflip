# Chainflip 长期测试网Perseverance验证人部署文档

# 情况说明：

1、激励测试网Soundcheck以于2022年1月19日结束，奖励在流动性引导池（LBP）建立后向参与者发放；长期性测试网Perseverance，这个阶段没有奖励，但在未来主网启动之前，项目方会考虑这个问题。

# 先决条件：

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

# 部署节点

1、通过APT Repo下载二进制文件

a、添加Chainflip APT Repo

sudo mkdir -p /etc/apt/keyrings

curl -fsSL repo.chainflip.io/keys/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/chainflip.gpg

b、验证密钥的真实性：

gpg --show-keys /etc/apt/keyrings/chainflip.gpg

<img width="386" alt="从终端看到以下输出" src="https://user-images.githubusercontent.com/100336530/207263177-6fc438f6-32ed-4209-b522-6769264ea975.png">

c、将Chainflip的Repo添加到apt sources列表中

echo "deb [signed-by=/etc/apt/keyrings/chainflip.gpg] https://repo.chainflip.io/perseverance/ focal main" | sudo tee /etc/apt/sources.list.d/chainflip.list

d、安装软件包

sudo apt-get update

sudo apt-get install -y chainflip-cli chainflip-node chainflip-engine


# Chainflip 长期测试网Perseverance验证人部署文档

# 情况说明：

1、激励测试网Soundcheck以于2022年1月19日结束，奖励在流动性引导池（LBP）建立后向参与者发放；长期性测试网Perseverance，这个阶段没有奖励，但在未来主网启动之前，项目方会考虑这个问题。

# 先决条件：

1、需要两个以太坊账户，第一个账户将用于接收来自龙头的tFLIP和与Staking App交互，以质押和提取资金；第二个账户（验证者钱包/账户，需要提供私钥）将被验证者用来发送交易到Goerli以太坊区块链。

2、钱包需要至少0.1个gETH与10个tFLIP，gETH用于向以太坊区块链提交交易的gas费，tFLIP用于启动验证人节点，成为活跃的验证人则需要更多的tFLIP。

3、准备gETH RPC地址，可以自己运行geth客户端，也可以使用第三方远程客户端，如：https://www.alchemy.com/    https://www.infura.io/zh    https://rivet.cloud/

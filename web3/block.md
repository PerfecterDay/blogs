## 长安链
长安链归集地址：
认购：0xD1C02F8d906fCFfdc34c19c531758A7Fd13A5bb2
赎回：0x85e411be787300333926B8C69e61905199b45b40

余额合约地址： 3dea0074963defa796ad46c22933c44788736e16
多签合约地址： 4adf0408613770fc5a95c087ee21e9ba77e4681a


## 以太坊
eth归集地址：
认购：0xF73D131e1B888fcBc729b73cDC36823e1Bb89a81
赎回：0x9aF4Bf7E6bAc7F10FE984EdD386717C62b6745BE


余额合约地址： 0xA71fE9a5999F445B67D4616542739b3AA2966334
多签合约地址： 0xF1f80BD53Fe880D3aCD7b0964AEed4f472Af9f51

RPC地址： http://10.187.224.48:8096/eth/sepolia ----> 反向代理 http://hk_server/eth/sepolia (hk_server:10.4.168.230:3000)


## 常用查询
查询交易信息：
http://10.187.224.48:8097/chain1/transaction/186418bc4cedec8eca03e467fa0f8327fb4a8cccf46c4f7c807dc51c24a5e8b9

https://sepolia.etherscan.io/token/0x9aF4Bf7E6bAC7F10FE984EdD386717C62b6745BE


curl -X POST https://ethereum-sepolia-rpc.publicnode.com \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc":"2.0",
    "method":"eth_getBalance",
    "params": ["0xF73D131e1B888fcBc729b73cDC36823e1Bb89a81", "latest"],
    "id":1
  }'

## 测试环境
shiyukun/Shiyukun5713$
gjsyShare@20210422


./MiniFTClient -h 10.189.36.1 -u wangzhongzhu
/opt/linux-client/12.04/now

http://10.187.224.48
nginx 配置 ：/data/stablecoin-project/ETH/nginx-proxy/domestic


## 代码
EthContractService
Web3j
ChainClient

授权：proposeGrantRole
proposeBatchGrantRole
approveProposal
proposeAddToWhitelist
proposeRemoveFromWhitelist
proposeFreeze
proposeUnfreeze
proposePause
proposeUnpause
proposeForceTransfer
addSuperAdmin
updateSuperAdmins
removeSuperAdmin
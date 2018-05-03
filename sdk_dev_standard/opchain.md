# 与链通信



## 3 与链通信

Onotology链支持Restful、RPC和Websocket连接。
|  连接方式    | 端口  |
|:--------    |:--   |
|   restful   | 20334|
|   rpc       | 20336|
|   websocket | 20335|

### 3.1 基础接口

* boolean      sendRawTransaction(Transaction tx)
> Note: 参数是交易实例

* boolean      sendRawTransaction(String hexData)
> Note: 参数是交易实例的十六进制字符串形式

* Object       sendRawTransactionPreExec(String hexData)
> Note: 预执行不会修改链上的数据，不用参与共识

* Transaction  getTransaction(String txhash)
> Note: 参数是交易hash

* Object       getTransactionJson(String txhash)
> Note: 参数是交易hash

* int          getGenerateBlockTime()
> Note: 返回出块时间

* int          getNodeCount()
> Note: 返回区块链节点数

* int          getBlockHeight()
> Note: 返回区块高度

* Block        getBlock(int height)
> Note: 根据区块高度获得区块

* Block        getBlock(String hash)
> Note: 根据区块hash获得区块

* Object       getBalance(String address)
> Note: 根据账户address获得余额

* Object       getBlockJson(int height)
> Note: 根据区块高度获得区块数据的JSON格式数据

* Object       getBlockJson(String hash)
> Note: 根据区块hash获得区块数据的JSON格式数据

* Object       getContract(String hash)
> Note: 根据合约hash获得合约代码

* Object       getContractJson(String hash)
> Note: 根据合约hash获得合约代码

* Object       getSmartCodeEvent(int height)
> Note: 根据区块高度获得合约事件

* Object       getSmartCodeEvent(String hash)
> Note: 根据区块高度获得合约事件

* int          getBlockHeightByTxHash(String hash)
> Note: 根据区块hash获得区块高度

* String       getStorage(String codehash, String key)
> Note: codeHash是部署的合约的codeAddress,key要使用十六进制字符串

* Object       getMerkleProof(String txhash)
> Note: 获得交易hash的Merkle证明，证明该交易存在链上

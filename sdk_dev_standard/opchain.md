# 与链通信

## 3 与链通信

Ontology链支持Restful、RPC和Websocket连接。

|  连接方式    | 端口  |
|:--------    |:--   |
|   restful   | 20334|
|   rpc       | 20336|
|   websocket | 20335|

### 3.1 基础接口

1 boolean sendRawTransaction(Transaction tx)
* 描述: 发送交易
* 输入参数: 交易实例
* 输出参数: true 代表成功, false 代表失败

* 异常处理

|   错误码 |  发生场景        |
|:--------| :------     |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

2 boolean sendRawTransaction(String hexData)
* 描述: 发送交易
* 输入参数: 交易实例的十六进制字符串形式
* 输出参数: true 代表成功, false 代表失败
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------      |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

3 Object sendRawTransactionPreExec(String hexData)
* 描述: 发送预执行交易，预执行不会修改链上的数据，不用参与共识
* 输入参数: 交易实例的十六进制字符串形式
* 输出参数: 对象
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------                                               |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

4 Transaction  getTransaction(String txhash)
* 描述: 根据交易hash获得交易
* 输入参数: 交易hash，去掉前面的"0x"
* 输出参数: 交易Transaction
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------                                               |
|58023    | Invalid url |
|53001    |Transaction deserialize failed|
|4XXXX    | Ontology错误|

5 Object getTransactionJson(String txhash)
* 描述: 根据交易hash获得交易json数据
* 输入参数: 交易hash，去掉前面的"0x"
* 输出参数: 交易json对象
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------       |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

6 int getGenerateBlockTime()
* 描述: 返回出块时间
* 输入参数
* 输出参数: 出块时间
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------     |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

7 int getNodeCount()
* 描述: 获得已连接的节点数量
* 输入参数
* 输出参数: 出块时间
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------    |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

8 int getBlockHeight()
* 描述: 获得区块高度
* 输入参数
* 输出参数: 当前的区块高度
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------     |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

9 Block getBlock(int height)
* 描述: 根据高度获得区块
* 输入参数: 区块高度
* 输出参数: 该高度对应的区块
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------       |
|58023    | Invalid url |
|53002    | Block deserialize failed|
|4XXXX    | Ontology错误|

10 Block getBlock(String hash)
* 描述
根据区块hash获得区块
* 输入参数: 区块hash
* 输出参数: 该区块hash对应的区块
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------       |
|58023    | Invalid url |
|53002    | Block deserialize failed|
|4XXXX    | Ontology错误|

11 Object   getBalance(String address)
* 描述: 根据账户address获得余额
* 输入参数: 账户地址
* 输出参数: 账户余额
* 异常处理

|   错误码 |  发生场景        |
|:--------| :------      |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

12 Object getBlockJson(int height)
* 描述: 根据区块高度获得区块数据的JSON格式数据
* 输入参数: 区块高度
* 输出参数: 该区块高度对应的区块json数据
* 异常处理

|   错误码 |  发生场景      |
|:--------| :------    |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

13 Object getBlockJson(String hash)
* 描述: 根据区块hash获得区块数据的JSON格式数据
* 输入参数: 区块hash
* 输出参数: 该区块hash对应的区块json对象
* 异常处理

|   错误码 |  发生场景      |
|:--------| :------     |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

14 Object getContract(String hash)
* 描述: 根据合约hash获得合约代码
* 输入参数: 合约hash
* 输出参数: 该合约hash对应的合约代码
* 异常处理

|   错误码 |  发生场景      |
|:--------| :------                                               |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

15 Object getContractJson(String hash)
* 描述: 根据合约hash获得合约代码json数据
* 输入参数: 合约hash
* 输出参数: 该合约hash对应的合约代码
* 异常处理

|   错误码 |  发生场景      |
|:--------| :------                                               |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

16 Object getSmartCodeEvent(int height)
* 描述: 根据区块高度获得合约事件
* 输入参数: 区块高度
* 输出参数: 事件对象
* 异常处理

|   错误码 |  发生场景      |
|:--------| :------     |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

17 Object getSmartCodeEvent(String hash)
* 描述: 根据交易hash获得合约事件
* 输入参数: 交易hash
* 输出参数: 事件对象
* 异常处理

|   错误码 |  发生场景      |
|:--------| :------       |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

18 int getBlockHeightByTxHash(String hash)
* 描述: 根据交易hash获得区块高度
* 输入参数: 交易hash
* 输出参数: 区块高度
* 异常处理

|   错误码 |  发生场景      |
|:--------| :------      |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

19 String getStorage(String codehash, String key)
* 描述: 获得合约的key存储的数据
* 输入参数: codeHash是部署的合约的codeAddress,key要使用十六进制字符串
* 输出参数: 合约的key存储的数据，String类型的值
* 异常处理

|   错误码 |  发生场景      |
|:--------| :------      |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

20 Object getMerkleProof(String txhash)
* 描述: 获得交易hash的Merkle证明，证明该交易存在链上
* 输入参数: 交易hash
* 输出参数: MerkleProof证明对象
* 异常处理

|   错误码 |  发生场景      |
|:--------| :------     |
|58023    | Invalid url |
|4XXXX    | Ontology错误|

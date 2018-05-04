# 数字资产ONT&ONG 交易

## 接口列表

| 方法名 | 参数 | 返回值类型 | 描述 |
|:--|:---|:---|:--|
| sendTransfer       |String assetName, String sendAddr, String password, String recvAddr, long amount      | String|两个地址之间转移资产|
|sendTransferToMany  |String assetName, String sendAddr, String password, String[] recvAddr, long[] amount  |String |给多个地址转移资产|
|sendTransferFromMany|String assetName, String[] sendAddr, String[] password, String recvAddr, long[] amount|String |多个地址向某个地址转移资产|
|sendOngTransferFrom |String sendAddr, String password, String to, long amount                              |String |转移ong资产|

## 接口定义

#### sendTransfer

* 描述
转移资产
* 输入参数
String assetName, String sendAddr, String password, String recvAddr, long amount
assetName: 资产名，
sendAddr: 发送方地址，
recvAddr: 接收方地址，
amount: 转移的数量
* 输出参数
交易hash
* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58012    | asset name error |
|58004    | param error|
|58023    | Invalid url|   
|未知      | Ontology内部错误|                                  

#### sendTransferToMany
* 描述
向多个账户转移资产
* 输入参数
String assetName, String sendAddr, String password, String[] recvAddr, long[] amount
assetName: 资产名，
sendAddr: 发送方地址，
recvAddr: 接收方地址，
amount: 转移的数量
amount和recvAddr一一对应
* 输出参数
交易hash
* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58012    | asset name error |
|58004    | param error|
|58023    | Invalid url|   
|4XXXX    | Ontology错误|  

#### sendTransferFromMany
* 描述
从多个账户向某个账户转移资产
* 输入参数
String assetName, String[] sendAddr, String[] password, String recvAddr, long[] amount
assetName: 资产名，
sendAddr: 发送方地址，
recvAddr: 接收方地址，
amount: 转移的数量
amount、sendAddr和password一一对应
* 输出参数
交易hash
* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58012    | asset name error |
|58004    | param error|
|58023    | Invalid url|   
|4XXXX    | Ontology错误|

#### sendOngTransferFrom
* 描述
提取ong到某个账户
* 输入参数
String sendAddr, String password, String to, long amount
sendAddr: 发送方地址，
to: 接收方地址，
amount: 转移的数量
* 输出参数
交易hash
* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58012    | asset name error |
|58004    | param error|
|58023    | Invalid url|   
|4XXXX    | Ontology错误|

## 交易说明

ont和ong合约address
```
private final String ontContract = "ff00000000000000000000000000000000000001";
private final String ongContract = "ff00000000000000000000000000000000000002";
```

State类字段如下：
```
public class State implements Serializable {
    public byte version;
    public Address from;
    public Address to;
    public BigInteger value;
    ...
  }
```

Transfers类字段如下：
```
public class Transfers implements Serializable {
    public byte version = 0;
    public State[] states;

    public Transfers(State[] states){
        this.states = states;
    }
    ...
  }
```

Contarct字段如下

```
public class Contract implements Serializable {
    public byte version;
    public byte[] code = new byte[0];
    public Address constracHash;
    public String method;
    public byte[] args;

    public Contract(byte version,byte[] code,Address constracHash, String method,byte[] args){
        this.version = version;
        if (code != null) {
            this.code = code;
        }
        this.constracHash = constracHash;
        this.method = method;
        this.args = args;
    }
    ....
  }
```
* 构造参数paramBytes

```
State state = new State(senderAddress, receiveAddress, new BigInteger(amount));
Transfers transfers = new Transfers(new State[]{state});
Contract contract = new Contract((byte) 0,null, Address.parse(contractAddr), "transfer", transfers.toArray());
byte[] paramBytes = contarct.toArray();
```
* 构造交易
请参考智能合约构造交易部分
* 发送交易
请参考智能合约发送交易部分

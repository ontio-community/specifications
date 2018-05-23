<h1 align="center">数字资产ONT&ONG交易 </h1>
<p align="center" class="version">Version 0.7.0 </p>

## 接口列表

| 方法名 | 参数 | 返回值类型 | 描述 |
|:--|:---|:---|:--|
| sendTransfer       |String assetName, String sendAddr, String password, String recvAddr, long amount,long gas | String|两个地址之间转移资产|
|sendTransferToMany  |String assetName, String sendAddr, String password, String[] recvAddr, long[] amount,long gas|String |给多个地址转移资产|
|sendTransferFromMany|String assetName, String[] sendAddr, String[] password, String recvAddr, long[] amount,long gas|String |多个地址向某个地址转移资产|
|sendOngTransferFrom |String sendAddr, String password, String to, long amount,long gas|String |提取ong资产|
|sendApprove|String assetName ,String sendAddr, String password, String recvAddr, long amount,long gas|||
|sendTransferFrom|String assetName ,String sendAddr, String password, String fromAddr, String toAddr, long amount,long gas|||
|sendAllowance|String assetName,String fromAddr,String toAddr|long|查询fromAddr授权给toAddr的数额|
|sendBalanceOf|String assetName, String address|long|查询余额|
|queryName|String assetName|String|查询资产名详细信息|
|querySymbol|String assetName|String|查询资产symbol信息|
|queryDecimals|String assetName|long|查询资产的精度|
|queryTotalSupply|String assetName|long|查询资产的总供应量|


## 接口定义

#### sendTransfer

* 描述

转移资产

* 输入参数

String assetName, String sendAddr, String password, String recvAddr, long amount

assetName: 资产名，

sendAddr: 发送方地址，

password: 发送方密码，

recvAddr: 接收方地址，

amount: 转移的数量

* 输出参数
交易hash
* 异常处理

|   错误码 |  描述信息        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology内部错误|                                  

#### sendTransferToMany
* 描述
向多个账户转移资产

* 输入参数

String assetName, String sendAddr, String password, String[] recvAddr, long[] amount,long gas

assetName: 资产名，

sendAddr: 发送方地址，

password:发送方密码，

recvAddr: 接收方地址数组，

amount: 转移的数量数组

gas: gas数量

amount和recvAddr一一对应

* 输出参数
交易hash
* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|  
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology错误|  

#### sendTransferFromMany
* 描述
从多个账户向某个账户转移资产

* 输入参数

String assetName, String[] sendAddr, String[] password, String recvAddr, long[] amount,long gas

assetName: 资产名，

sendAddr: 发送方地址数组，

password: 发送方密码数组，

recvAddr: 接收方地址，

amount: 转移的数量数组

gas: gas数量

amount、sendAddr和password一一对应

* 输出参数

交易hash

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology错误|

#### sendOngTransferFrom

* 描述

提取ong到某个账户

* 输入参数

String sendAddr, String password, String to, long amount,long gas

sendAddr: 发送方地址，

password: 发送方密码，

to: 接收方地址，

amount: 转移的数量

gas: gas数量

* 输出参数

交易hash

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology错误|

#### sendApprove

* 描述

授权其他账户转移一定量的资产

* 输入参数
String assetName ,String sendAddr, String password, String recvAddr, long amount,long gas

assetName: 资产名称

sendAddr: 发送方地址，

password: 发送方密码，

recvAddr: 接收方地址，

amount: 转移的数量

gas: gas数量

* 输出参数

交易hash

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology错误|

#### sendTransferFrom

* 描述

和sendApprove一起使用，交易发起方从某个账户转移一定量的资产到另一个地址

* 输入参数

String assetName ,String sendAddr, String password, String fromAddr, String toAddr, long amount,long gas

assetName: 资产名称

sendAddr: 发送方地址，

password: 发送方密码，

fromAddr: 资产转出方地址，

toAddr: 资产接收方地址，

amount: 转移的数量

gas: gas数量

* 输出参数

交易hash

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology错误|

#### sendAllowance

* 描述

和sendApprove一起使用，查询授权转移的资产数量

* 输入参数
String assetName,String fromAddr,String toAddr

assetName: 资产名

fromAddr: 授权资产转出方地址

toAddr: 授权资产转入方地址

* 输出参数

授权转移的资产数量

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology错误|

#### sendBalanceOf

* 描述

查询余额

* 输入参数

String assetName, String address

assetName: 资产名

address: 账户地址

* 输出参数

资产余额

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology错误|

#### queryName

* 描述

查询资产名详细信息

* 输入参数

String assetName

* 输出参数

资产名详细信息

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology错误|

#### querySymbol

* 描述

查询资产symbol信息

* 输入参数

String assetName

assetName: 资产名称

* 输出参数

资产symbol信息

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology错误|

#### queryDecimals

* 描述

查询精度

* 输入参数

String assetName

assetName: 资产名

* 输出参数

资产精度

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology错误|

#### queryTotalSupply

* 描述

查询资产总供应量

* 输入参数

String assetName

assetName: 资产名

* 输出参数

总供应量

* 异常处理

|   错误码 |  发生场景        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
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
    public Address from;
    public Address to;
    public long value;
    ...
  }
```

Transfers类字段如下：
```
public class Transfers implements Serializable {
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
    public Address contractHash;
    public String method;
    public byte[] args;

    public Contract(byte version,byte[] code,Address contractHash, String method,byte[] args){
        this.version = version;
        if (code != null) {
            this.code = code;
        }
        this.contractHash = contractHash;
        this.method = method;
        this.args = args;
    }
    ....
  }
```
* 构造参数paramBytes

```
State state = new State(senderAddress, receiveAddress, amount);
Transfers transfers = new Transfers(new State[]{state});
Contract contract = new Contract((byte) 0,null, Address.parse(contractAddr), "transfer", transfers.toArray());
byte[] paramBytes = contarct.toArray();
```
* 构造交易
请参考智能合约构造交易部分
* 发送交易
请参考智能合约发送交易部分

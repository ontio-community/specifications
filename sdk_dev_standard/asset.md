# 数字资产ONT&ONG 交易


## 接口列表

| 方法名 | 参数 | 返回值类型 | 描述 |
|:--|:---|:---|:--|
| sendTransfer       |String assetName, String sendAddr, String password, String recvAddr, long amount      | String|两个地址之间转移资产|
|sendTransferToMany  |String assetName, String sendAddr, String password, String[] recvAddr, long[] amount  |String |给多个地址转移资产|
|sendTransferFromMany|String assetName, String[] sendAddr, String[] password, String recvAddr, long[] amount|String |多个地址向某个地址转移资产|
|sendOngTransferFrom |String sendAddr, String password, String to, long amount                              |String |转移ong资产|


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
State state = new State(senderAddress, receiveAddress, new BigInteger(String.valueOf(amount)));
Transfers transfers = new Transfers(new State[]{state});
Contract contract = new Contract((byte) 0,null, Address.parse(contractAddr), "transfer", transfers.toArray());
byte[] paramBytes = contarct.toArray();
```
* 构造交易

```
public InvokeCode makeInvokeCodeTransaction(String codeAddr,String method,byte[] params, byte vmtype, Fee[] fees) throws SDKException {
        if(vmtype == VmType.NEOVM.value()) {
            Contract contract = new Contract((byte) 0, null, Address.parse(codeAddr), "", params);
            params = Helper.addBytes(new byte[]{0x67}, contract.toArray());
//            params = Helper.addBytes(params, new byte[]{0x69});
//            params = Helper.addBytes(params, Helper.hexToBytes(codeAddress));
        }else if(vmtype == VmType.WASMVM.value()) {
            Contract contract = new Contract((byte) 1, null, Address.parse(codeAddr), method, params);
            params = contract.toArray();
        }
        InvokeCode tx = new InvokeCode();
        tx.attributes = new Attribute[1];
        tx.attributes[0] = new Attribute();
        tx.attributes[0].usage = AttributeUsage.Nonce;
        tx.attributes[0].data = UUID.randomUUID().toString().getBytes();
        tx.code = params;
        tx.gasLimit = 0;
        tx.vmType = vmtype;
        tx.fee = fees;
        return tx;
    }
```
* 发送交易

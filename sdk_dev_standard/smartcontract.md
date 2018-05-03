# 本体 SDK 开发标准




## 4 智能合约交易

目前Ontology链上可以运行Native、NEO和WASM合约，SDK要实现NEO和WASM合约的部署和调用交易。

* Transaction类字段如下

```
public abstract class Transaction extends Inventory {
    public byte version = 0;
    public final TransactionType txType;
    public int nonce = new Random().nextInt();
    public Attribute[] attributes;
    public Fee[] fee = new Fee[0];
    public long networkFee;
    public Sig[] sigs = new Sig[0];
  }
```
* 虚拟机类型

```
Native(0xff),
NEOVM(0x80),
WASMVM(0x90);
```

### 2.1 Neo合约构造交易

1. 读取合约abi文件

```
InputStream is = new FileInputStream("C:\\ZX\\IdContract.abi.json");
byte[] bys = new byte[is.available()];
is.read(bys);
is.close();
String abi = new String(bys);
AbiInfo abiinfo = JSON.parseObject(abi, AbiInfo.class);
```
2. 构造参数

将函数参数转换成虚拟机可以执行的字节码，详细的字节码数据请查看本文当末尾。
假设调用某合约中函数需要如下参数：
函数名，参数1，参数2
转换成虚拟机能够识别的字节码：
* 反序转换参数
 如果遇到数组或者集合类型的数据，将数组或集合中的数据反序遍历并压入栈中，然后将该数组或者集合的大小压入栈中，然后在压入OP_PACK(0xC1)字节码;

* 将参数压入栈中进行的操作（以Java中的数据类型为例）
 如果参数是boolean类型数据。

```
 //true对应的字节码是OP_1(0x51),false对应的字节码是OP_0(0x00)
 public ScriptBuilder push(boolean b) {
    if(b == true) {
        return add(ScriptOp.OP_1);
    }
    return add(ScriptOp.OP_0);
}
```

如果参数是BigInteger

需要要将参数按照小端序转换成byte[],然后将byte[]转换成BigInteger对象，在进行如下操作：
判断是不是-1,如果是，往栈中压入OP_1NEGATE(0x4F)
判断是不是0,如果是，往栈中压入OP_0(0x00)
判断是不是大于0并且小于等于16,如果是，往栈中压入ScriptOp.OP_1.getByte() - 1 + number.byteValue()
其他的情况，往栈中压入该值的字节数组

```
public ScriptBuilder push(BigInteger number) {
//判断是不是-1
if (number.equals(BigInteger.ONE.negate())) {
    return add(ScriptOp.OP_1NEGATE);
}
//判断是不是0
if (number.equals(BigInteger.ZERO)) {
    return add(ScriptOp.OP_0);
}
//判断是不是大于0并且小于等于16
if (number.compareTo(BigInteger.ZERO) > 0 && number.compareTo(BigInteger.valueOf(16)) <= 0) {
    return add((byte) (ScriptOp.OP_1.getByte() - 1 + number.byteValue()));
}
return push(number.toByteArray());
}
```

如果参数是byte数组
如果字节数组的长度小于OP_PUSHBYTES75，写入数组长度，然后写入数据数据
如果字节数组的长度小于0x100，往栈中压入OP_PUSHDATA1(0x4C)，写入数组长度，然后写入数据数据
如果字节数组的长度小于0x10000，往栈中压入OP_PUSHDATA2(0x4D)，写入数组长度，然后写入数据数据（详见下面例子）
如果字节数组的长度小于0x100000000L，往栈中压入OP_PUSHDATA4(0x4E)，写入数组长度，然后写入数据数据（详见下面例子）

```
 public ScriptBuilder push(byte[] data) {
    if (data == null) {
    	throw new NullPointerException();
    }
    if (data.length <= (int)ScriptOp.OP_PUSHBYTES75.getByte()) {
        ms.write((byte)data.length);
        ms.write(data, 0, data.length);
    } else if (data.length < 0x100) {
        add(ScriptOp.OP_PUSHDATA1);
        ms.write((byte)data.length);
        ms.write(data, 0, data.length);
    } else if (data.length < 0x10000) {
        add(ScriptOp.OP_PUSHDATA2);
		ms.write(ByteBuffer.allocate(2).order(ByteOrder.LITTLE_ENDIAN).putShort((short)data.length).array(), 0, 2);
        ms.write(data, 0, data.length);
    } else if (data.length < 0x100000000L) {
        add(ScriptOp.OP_PUSHDATA4);
        ms.write(ByteBuffer.allocate(4).order(ByteOrder.LITTLE_ENDIAN).putInt(data.length).array(), 0, 4);
        ms.write(data, 0, data.length);
    } else {
        throw new IllegalArgumentException();
    }
    return this;
}
```

* Java转换参数的示例：

```
byte[] params = createCodeParamsScript(list);
public byte[] createCodeParamsScript(List<Object> list) {
ScriptBuilder sb = new ScriptBuilder();
try {
    for (int i = list.size() - 1; i >= 0; i--) {
        Object val = list.get(i);
        if (val instanceof byte[]) {
            sb.push((byte[]) val);
        } else if (val instanceof Boolean) {
            sb.push((Boolean) val);
        } else if (val instanceof Integer) {
            sb.push(new BigInteger(Int2Bytes_LittleEndian((int)val)));
        } else if (val instanceof List) {
            List tmp = (List) val;
            createCodeParamsScript(sb, tmp);
            sb.push(new BigInteger(String.valueOf(tmp.size())));
            sb.pushPack();
        } else {
        }
    }
} catch (Exception e) {
    e.printStackTrace();
}
return sb.toArray();
}
```

3. 构造交易

```
//根据虚拟机类型构造交易
if(vmtype == VmType.NEOVM.value()) {
   Contract contract = new Contract((byte) 0, null, Address.parse(codeAddr), "", params);
   params = Helper.addBytes(new byte[]{0x67}, contract.toArray());
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
//Contract构造方法
public Contract(byte version,byte[] code,Address constracHash, String method,byte[] args){
    this.version = version;
    if (code != null) {
        this.code = code;
    }
    this.constracHash = constracHash;
    this.method = method;
    this.args = args;
}

```
4. 交易签名
a 将交易对象序列化成字节数据
b 对交易的字节数组进行两次sha256运算得到txhash
c 对txHash进行签名

```
//将交易对象中的字段值转换成字节数组txBytes
ByteArrayOutputStream ms = new ByteArrayOutputStream();
BinaryWriter writer = new BinaryWriter(ms);
writer.writeByte(version);
writer.writeByte(txType.value());
writer.writeInt(nonce);
serializeExclusiveData(writer);
writer.writeSerializableArray(attributes);
writer.writeSerializableArray(fee);
writer.writeLong(networkFee);
writer.flush();
byte[] txBytes = ms.toByteArray();
//对交易字节数组进行两次sha256
String txhash = Digest.sha256(Digest.sha256(txBytes))
//对txhash做签名
byte[] signature = tx.sign(accounts[i][j], getWalletMgr().getSignatureScheme());
//给Transaction中的字段赋值
tx.sigs = sigs;
```

交易签名结构体如下
```
public class Sig implements Serializable {
    public byte[][] pubKeys = null;
    public int M;
    public byte[][] sigData;
 }
```

属性说明
    pubKeys 签名的公钥
    M 需要的签名的公钥数
    sigData 签名数据

* 签名示例：

```
public Transaction signTx(Transaction tx, Account[][] accounts) throws Exception{
  Sig[] sigs = new Sig[accounts.length];
  for (int i = 0; i < accounts.length; i++) {
      sigs[i] = new Sig();
      sigs[i].pubKeys = new byte[accounts[i].length][];
      sigs[i].sigData = new byte[accounts[i].length][];
      for (int j = 0; j < accounts[i].length; j++) {
          sigs[i].M++;
          byte[] signature = tx.sign(accounts[i][j], getWalletMgr().getSignatureScheme());
          sigs[i].pubKeys[j] = accounts[i][j].serializePublicKey();
          sigs[i].sigData[j] = signature;
      }
  }
  tx.sigs = sigs;
  return tx;
}
```

5. 发送交易
  1 将交易实例转换成字节数组
  txBytes
  ```
  byte[] txBytes = toArray();
  default byte[] toArray() {
        try (ByteArrayOutputStream ms = new ByteArrayOutputStream()) {
	        try (BinaryWriter writer = new BinaryWriter(ms)) {
	            serialize(writer);
	            writer.flush();
	            return ms.toByteArray();
	        }
        } catch (IOException ex) {
			throw new UnsupportedOperationException(ex);
		}
    }
  ```

  2 将txBytes转换成十六进制字符串
  ```
  String txHex = toHexString(txBytes);
  public static String toHexString(byte[] value) {
        StringBuilder sb = new StringBuilder();
        for (byte b : value) {
            int v = Byte.toUnsignedInt(b);
            sb.append(Integer.toHexString(v >>> 4));
            sb.append(Integer.toHexString(v & 0x0f));
        }
        return sb.toString();
    }
  ```
  3. 发送交易(以restful为例)

```
public String sendTransaction(boolean preExec, String userid, String action, String version, String data) throws RestfulException {
        Map<String, String> params = new HashMap<String, String>();
        if (userid != null) {
            params.put("userid", userid);
        }
        if (preExec) {
            params.put("preExec", "1");
        }
        Map<String, Object> body = new HashMap<String, Object>();
        body.put("Action", action);
        body.put("Version", version);
        body.put("Data", data);
        try {
            return http.post(url + UrlConsts.Url_send_transaction, params, body);
        } catch (Exception e) {
            throw new RestfulException("Invalid url:" + url + "," + e.getMessage(), e);
        }
    }
```

### 2.2 Wasm合约构造交易

  1 构造调用合约中的方法需要的参数；

依次将参数的值和类型放入map集合中，然后转换成json字符串

  ```
  //合约函数中需要的参数json字符串
  public String buildWasmContractJsonParam(Object[] objs) {
        List params = new ArrayList();
        for (int i = 0; i < objs.length; i++) {
            Object val = objs[i];
            if (val instanceof String) {
                Map map = new HashMap();
                map.put("type","string");
                map.put("value",val);
                params.add(map);
            } else if (val instanceof Integer) {
                Map map = new HashMap();
                map.put("type","int");
                map.put("value",String.valueOf(val));
                params.add(map);
            } else if (val instanceof Long) {
                Map map = new HashMap();
                map.put("type","int64");
                map.put("value",String.valueOf(val));
                params.add(map);
            } else if (val instanceof int[]) {
                Map map = new HashMap();
                map.put("type","int_array");
                map.put("value",val);
                params.add(map);
            } else if (val instanceof long[]) {
                Map map = new HashMap();
                map.put("type","int_array");
                map.put("value",val);
                params.add(map);
            } else {
                continue;
            }
        }
        Map result = new HashMap();
        result.put("Params",params);
        return JSON.toJSONString(result);
    }
  ```

  2 构造交易；
  ```
  //需要的参数：合约hash，合约函数名，虚拟机类型，费用实例
  Transaction tx = ontSdk.getSmartcodeTx().makeInvokeCodeTransaction(codeAddress,"add",params.getBytes(),VmType.WASMVM.value(),new Fee[0]);
  ```
>参数说明: codeAddress是智能合约address，“add”是调用的合约函数名，params.getBytes()参数的字节形式，VmType.WASMVM.value() wasm合约类型值，

  3 交易签名(如果是预执行不需要签名)；
    和Neo合约中一样

* 示例：

```
//设置要调用的合约地址codeAddress
ontSdk.getSmartcodeTx().setCodeAddress(codeAddress);
String funcName = "add";
//构造合约函数需要的参数
String params = ontSdk.getSmartcodeTx().buildWasmContractJsonParam(new Object[]{20,30});
//指定虚拟机类型构造交易
Transaction tx = ontSdk.getSmartcodeTx().makeInvokeCodeTransaction(ontSdk.getSmartcodeTx().getCodeAddress(),funcName,params.getBytes(),VmType.WASMVM.value(),new Fee[0]);
//发送交易
ontSdk.getConnectMgr().sendRawTransaction(tx.toHexString());

```

## 4 智能合约使用说明

具体实现可以参考java-sdk中智能合约调用相关文章https://ontio.github.io/documentation/ontology_java_sdk_smartcontract_zh.html

### 4.1 部署合约

> Note: Neo和Wasm合约的部署一样。
* java 例子

```
InputStream is = new FileInputStream("IdContract.avm");
byte[] bys = new byte[is.available()];
is.read(bys);
is.close();
//将字节数组转换成十六进制字符串
code = Helper.toHexString(bys);
ontSdk.setCodeAddress(Helper.getCodeAddress(code,VmType.NEOVM.value()));
//部署合约
String txhash = ontSdk.getSmartcodeTx().makeDeployCodeTransaction(code, true, "name", "1.0", "1", "1", "1", VmType.NEOVM.value());
System.out.println("txhash:" + txhash);
//等待出块
Thread.sleep(6000);
DeployCodeTransaction t = (DeployCodeTransaction) ontSdk.getConnectMgr().getTransaction(txhash);
```
* 构造部署交易参数说明

| 参数      | 字段   | 类型  | 描述 |             说明 |
| ----- | ------- | ------ | ------------- | ----------- |
| 输入参数 | codeHexStr| String | 合约code十六进制字符串 | 必选 |
|        | needStorage    | Boolean | 是否需要存储   | 必选 |
|        | name    | String  | 名字       | 必选|
|        | codeVersion   | String | 版本       |  必选 |
|        | author   | String | 作者     | 必选 |
|        | email   | String | emal     | 必选 |
|        | desp   | String | 描述信息     | 必选 |
|        | VmType   | byte | 虚拟机类型     | 必选 |
| 输出参数 | tx   | Transaction  | 交易实例  |  |

### 4.2 调用合约

#### 4.2.1 Neo合约调用

 * 基本流程：
 1. 读取智能合约的abi文件；
 2. 构造调用智能合约函数；
 3. 构造交易；
 4. 交易签名(预执行不需要签名)；
 5. 发送交易。

 * 示例
 ```
 //读取智能合约的abi文件
 InputStream is = new FileInputStream("C:\\ZX\\NeoContract1.abi.json");
 byte[] bys = new byte[is.available()];
 is.read(bys);
 is.close();
 String abi = new String(bys);

 //解析abi文件
 AbiInfo abiinfo = JSON.parseObject(abi, AbiInfo.class);
 System.out.println("codeHash:"+abiinfo.getHash());
 System.out.println("Entrypoint:"+abiinfo.getEntrypoint());
 System.out.println("Functions:"+abiinfo.getFunctions());
 System.out.println("Events"+abiinfo.getEvents());

 //设置智能合约codeAddress
 ontSdk.setCodeAddress(abiinfo.getHash());

 //获取账号信息
 Identity did = ontSdk.getWalletMgr().getIdentitys().get(0);
 AccountInfo info = ontSdk.getWalletMgr().getAccountInfo(did.ontid,"passwordtest");

 //构造智能合约函数
 AbiFunction func = abiinfo.getFunction("AddAttribute");
 System.out.println(func.getParameters());
 func.setParamsValue(did.ontid.getBytes(),"key".getBytes(),"bytes".getBytes(),"values02".getBytes(),Helper.hexToBytes(info.pubkey));
 System.out.println(func);
 //调用智能合约，sendInvokeSmartCodeWithSign方法封装好了构造交易，签名交易，发送交易步骤
 String hash = ontSdk.getSmartcodeTx().sendInvokeSmartCodeWithSign(did.ontid, "passwordtest", func, (byte) VmType.NEOVM.value()););
 ```
 * AbiInfo结构(NEO合约调用的时候需要，WASM合约不需要)

```
public class AbiInfo {
    public String hash;
    public String entrypoint;
    public List<AbiFunction> functions;
    public List<AbiEvent> events;
}
public class AbiFunction {
    public String name;
    public String returntype;
    public List<Parameter> parameters;
}
public class Parameter {
    public String name;
    public String type;
    public String value;
}
```
#### 4.2.2 WASM合约调用
* 基本流程：
  1. 构造调用合约中的方法需要的参数；
  2. 构造交易；
  3. 交易签名(如果是预执行不需要签名)；
  4. 发送交易。

* 示例：

```
//设置要调用的合约地址codeAddress
ontSdk.getSmartcodeTx().setCodeAddress(codeAddress);
String funcName = "add";
//构造合约函数需要的参数
String params = ontSdk.getSmartcodeTx().buildWasmContractJsonParam(new Object[]{20,30});
//指定虚拟机类型构造交易
Transaction tx = ontSdk.getSmartcodeTx().makeInvokeCodeTransaction(ontSdk.getSmartcodeTx().getCodeAddress(),funcName,params.getBytes(),VmType.WASMVM.value(),new Fee[0]);
//发送交易
ontSdk.getConnectMgr().sendRawTransaction(tx.toHexString());
```

### 4.3 智能合约执行过程推送

创建websocket线程，解析推送结果。


* 1 设置websocket链接


```
//lock 全局变量,同步锁
public static Object lock = new Object();

//获得ont实例
String ip = "http://127.0.0.1";
String wsUrl = ip + ":" + "20335";
OntSdk wm = OntSdk.getInstance();
wm.setWesocket(wsUrl, lock);
wm.setDefaultConnect(wm.getWebSocket());
wm.openWalletFile("OntAssetDemo.json");

```


* 2 启动websocket线程


```
//false 表示不打印回调函数信息
ontSdk.getWebSocket().startWebsocketThread(false);

```

* 3 启动结果处理线程


```
Thread thread = new Thread(
                    new Runnable() {
                        @Override
                        public void run() {
                            waitResult(lock);
                        }
                    });
            thread.start();
            //将MsgQueue中的数据取出打印
            public static void waitResult(Object lock) {
                    try {
                        synchronized (lock) {
                            while (true) {
                                lock.wait();
                                for (String e : MsgQueue.getResultSet()) {
                                    System.out.println("RECV: " + e);
                                    Result rt = JSON.parseObject(e, Result.class);
                                    //TODO
                                    MsgQueue.removeResult(e);
                                    if (rt.Action.equals("getblockbyheight")) {
                                        Block bb = Serializable.from(Helper.hexToBytes((String) rt.Result), Block.class);
                                        //System.out.println(bb.json());
                                    }
                                }
                            }
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
```


* 4 每6秒发送一次心跳程序，维持socket链接


```
for (;;){
                    Map map = new HashMap();
                    if(i >0) {
                        map.put("SubscribeEvent", true);
                        map.put("SubscribeRawBlock", false);
                    }else{
                        map.put("SubscribeJsonBlock", false);
                        map.put("SubscribeRawBlock", true);
                    }
                    //System.out.println(map);
                    ontSdk.getWebSocket().setReqId(i);
                    ontSdk.getWebSocket().sendSubscribe(map);     
                Thread.sleep(6000);
            }
```


* 5 推送结果事例详解


以调用存证合约的put函数为例，

//存证合约abi.json文件部分内容如下

```
{
    "hash":"0x27f5ae9dd51499e7ac4fe6a5cc44526aff909669",
    "entrypoint":"Main",
    "functions":
    [

    ],
    "events":
    [
        {
            "name":"putRecord",
            "parameters":
            [
                {
                    "name":"arg1",
                    "type":"String"
                },
                {
                    "name":"arg2",
                    "type":"ByteArray"
                },
                {
                    "name":"arg3",
                    "type":"ByteArray"
                }
            ],
            "returntype":"Void"
        }
    ]
}
```

当调用put函数保存数据时，触发putRecord事件，websocket 推送的结果是{"putRecord", "arg1", "arg2", "arg3"}的十六进制字符串

例子如下：

```
RECV: {"Action":"Log","Desc":"SUCCESS","Error":0,"Result":{"Message":"Put","TxHash":"8cb32f3a1817d88d8562fdc0097a0f9aa75a926625c6644dfc5417273ca7ed71","ContractAddress":"80f6bff7645a84298a1a52aa3745f84dba6615cf"},"Version":"1.0.0"}
RECV: {"Action":"Notify","Desc":"SUCCESS","Error":0,"Result":[{"States":["7075745265636f7264","507574","6b6579","7b2244617461223a7b22416c6772697468656d223a22534d32222c2248617368223a22222c2254657874223a2276616c75652d7465737431222c225369676e6174757265223a22227d2c2243416b6579223a22222c225365714e6f223a22222c2254696d657374616d70223a307d"],"TxHash":"8cb32f3a1817d88d8562fdc0097a0f9aa75a926625c6644dfc5417273ca7ed71","ContractAddress":"80f6bff7645a84298a1a52aa3745f84dba6615cf"}],"Version":"1.0.0"}
```




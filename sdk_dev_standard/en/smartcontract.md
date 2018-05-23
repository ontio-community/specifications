<h1 align="center">Smart Contract Development Standard</h1>
<p align="center" class="version">Version 0.7.0 </p>

## 4 Smart contract transaction

Currently, Native, NEO, and WASM contracts can be run on the Ontology chain. The SDK can implement the NEO and WASM contract deployment and invocation transactions, and at the same time implement the invocation transaction of the Native contract.
> Note: The following code are reference code

* Transaction class field

```
public abstract class Transaction extends Inventory {
    public byte version = 0;
    public final TransactionType txType;
    public int nonce = new Random().nextInt();
    public Attribute[] attributes;
    public long gasPrice = 0;
    public long gasLimit = 0;
    public Address payer = new Address();
    public Sig[] sigs = new Sig[0];
  }
```
* Virtual machine type

```
Native(0xff),
NEOVM(0x80),
WASMVM(0x90);
```

### 4.1 Neo contract construction transaction

1. Read the contract abi file

```
InputStream is = new FileInputStream("C:\\ZX\\IdContract.abi.json");
byte[] bys = new byte[is.available()];
is.read(bys);
is.close();
String abi = new String(bys);
AbiInfo abiinfo = JSON.parseObject(abi, AbiInfo.class);
```
2. Construct parameters

For converting the function parameters into bytecodes that can be executed by the virtual machine, detailed bytecode data can refer to the [attachment](appendix.md).

Assume that calling a function in a contract requires the following parameters:
Function name, parameter 1, parameter 2
Convert to a bytecode that the virtual machine can recognize:
- Reverse parameters
 If encountering array or set type data, the data in the array or set is traversed in reverse order and pushed into the stack. Next, pushing the size of the array or set into the stack, and finally pushing the OP_PACK (0xC1) byte code;

Pushing parameters into the stack (take Java as an example)
  - If the parameter is a boolean data type
  ```
   //The bytecode corresponding to true is OP_1(0x51), and the bytecode corresponding to false is OP_0(0x00)
   public ScriptBuilder push(boolean b) {
      if(b == true) {
          return add(ScriptOp.OP_1);
      }
      return add(ScriptOp.OP_0);
  }
  ```

  - If the parameter is BigInteger,
we need to convert the parameters in accordance with the little endian to byte[], and then convert byte[] to a BigInteger object, finally execute as follows:
Judge if it is -1, if yes, push OP_1NEGATE(0x4F) into the stack
Judge if it is 0, if yes, push OP_0 (0x00) into the stack
Judge if it is greater than 0 and less than or equal to 16, if yes, push ScriptOp.OP_1.getByte() - 1 + number.byteValue() into the stack.
In other cases, push the byte array of the value into the stack;

  
  ```
  public ScriptBuilder push(BigInteger number) {
  //Judge if it is -1
  if (number.equals(BigInteger.ONE.negate())) {
      return add(ScriptOp.OP_1NEGATE);
  }
  //Judge if it is 0
  if (number.equals(BigInteger.ZERO)) {
      return add(ScriptOp.OP_0);
  }
  //Judge if it is greater than 0 and less than or equal to 16
  if (number.compareTo(BigInteger.ZERO) > 0 && number.compareTo(BigInteger.valueOf(16)) <= 0) {
      return add((byte) (ScriptOp.OP_1.getByte() - 1 + number.byteValue()));
  }
  return push(number.toByteArray());
  }
  ```

  - if the parameter is a byte array
If the length of the byte array is less than OP_PUSHBYTES75, write the length of the array and then write the data
If the length of the byte array is less than 0x100, push OP_PUSHDATA1(0x4C) into the stack, write the length of the array, and then write the data
If the length of the byte array is less than 0x10000, push OP_PUSHDATA2(0x4D) into the stack, write the length of the array, and then write the data (see the following example)
If the length of the byte array is less than 0x100000000L, push OP_PUSHDATA4(0x4E) into the stack, write the length of the array, and then write the data (see the following example)

  ```
   public ScriptBuilder push(byte[] data) {
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

3. Construct transaction:
Construct transactions based on different virtual machine types
```
//Construct transactions based on virtual machine types
if(vmtype == VmType.NEOVM.value()) {
   Contract contract = new Contract((byte) 0, null, Address.parse(codeAddr), "", params);
   params = Helper.addBytes(new byte[]{0x67}, contract.toArray());
}else if(vmtype == VmType.WASMVM.value()) {
    Contract contract = new Contract((byte) 1, null, Address.parse(codeAddr), method, params);
    params = contract.toArray();
}
InvokeCode tx = new InvokeCode();
tx.code = params;
tx.vmType = vmtype;
...
```

4. Transaction signature:
a serialize transaction objects to byte data
For the serialization method, please refer to [smartcontract](smartcontract.md)
b Calculate sha256 twice for the transaction's byte array to get txhash
c Sign the txhash

```
//Convert field values ​​of transaction objects to byte array txBytes
byte[] txBytes = tx.getByteArray();
//Calculate sha256 twice for the transaction byte array
String txhash = Digest.sha256(Digest.sha256(txBytes))
//Sign the txhash
byte[] signature = tx.sign(accounts[i][j], getWalletMgr().getSignatureScheme());
//Assign a value to the field in transaction
tx.sigs = sigs;
```

The transaction signature structure is as follows
```
public class Sig implements Serializable {
    public byte[][] pubKeys = null;
    public int M;
    public byte[][] sigData;
 }
```

Attribute description
    pubKeys: Signed Public Key
    M: The number of public keys required
    sigData: Signature data

5. Send transactions
1 Convert transaction instance to byte array
  txBytes
  ```
  //F is conversion function
  byte[] txBytes = F(tx);
  ```

  2 Convert txBytes to a hexadecimal string
  ```
  String txHex = toHexString(txBytes);
  ```
  3 Send transactions (restful example)
Input parameter description
preExec true represents "pre-execution"，false represents "non-pre-execution."
action "sendrawtransaction" represents "send transaction"
version version number, indicating the protocol version number
data transaction parameters
```
public String sendTransaction(boolean preExec, String action, String version, String data) throws RestfulException {
        Map<String, String> params = new HashMap<String, String>();
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

### 4.2 Wasm contract construction transaction

  1 Constructs the parameters required by the method in the calling contract;

In order to put the value and type of the parameter into the set, and then convert to a json string

  ```
  //The parameter json string required in the contract function
  public String buildWasmContractJsonParam(Object[] objs) {
        List params = new ArrayList();
        for (int i = 0; i < objs.length; i++) {
            Object val = objs[i];
            Map map = new HashMap();
            map.put("type",val.Type());
            map.put("value",val.value());
            params.add(map);
            ...
        }
        Map result = new HashMap();
        result.put("Params",params);
        return JSON.toJSONString(result);
    }
  ```

  2 Construct transaction
Input parameter description: codeAddress is the address of smart contract，method is the contract function name，params is the parameter byte form，VmType.WASMVM.value() is the wasm contract type value.
Process:
a Construct params based on virtual machine type
b Instantiate InvokeCode
```
//Required parameters: contract hash, contract function name, virtual machine type, cost instance

public InvokeCode makeInvokeCodeTransaction(String codeAddr,String method,byte[] params, byte vmtype, Fee[] fees) throws SDKException {
        //Constructs params based on virtual machine type
        if(vmtype == VmType.NEOVM.value()) {
            Contract contract = new Contract((byte) 0, null, Address.parse(codeAddr), "", params);
            params = Helper.addBytes(new byte[]{0x67}, contract.toArray());
        }else if(vmtype == VmType.WASMVM.value()) {
            Contract contract = new Contract((byte) 1, null, Address.parse(codeAddr), method, params);
            params = contract.toArray();
        }
        //Instantiate InvokeCode
        InvokeCode tx = new InvokeCode();
        tx.code = params;  
        tx.vmType = vmtype;
        ...
        return tx;
    }
```

  3 Transaction signature(no signature required for pre-execution)；
   The same as Neo contract

* Example：

```
//Set the contract address - codeAddress
ontSdk.getSmartcodeTx().setCodeAddress(codeAddress);
String funcName = "add";
//Construct parameters required in the contract
String params = ontSdk.getSmartcodeTx().buildWasmContractJsonParam(new Object[]{20,30});
//Specify the virtual machine type to construct transaction
Transaction tx = ontSdk.getSmartcodeTx().makeInvokeCodeTransaction(ontSdk.getSmartcodeTx().getCodeAddress(),funcName,params.getBytes(),VmType.WASMVM.value(),new Fee[0]);
//Send transaction
ontSdk.getConnectMgr().sendRawTransaction(tx.toHexString());

```

### 4.3 Smart contract execution process push

Create a websocket thread and parse the push result.


* 1 Set websocket link


```
//lock global variable, synchronization lock
public static Object lock = new Object();

//Get ont instance
String ip = "http://127.0.0.1";
String wsUrl = ip + ":" + "20335";
OntSdk wm = OntSdk.getInstance();
wm.setWesocket(wsUrl, lock);
wm.setDefaultConnect(wm.getWebSocket());
wm.openWalletFile("OntAssetDemo.json");
```


* 2 Start websocket thread


```
//false means the program does not need to print callback function information
ontSdk.getWebSocket().startWebsocketThread(false);

```

* 3 Start result processing thread


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
                                    Result rt = JSON.parseObject(e, Result.class);
                                    //TODO
                                    MsgQueue.removeResult(e);
                                    if (rt.Action.equals("getblockbyheight")) {
                                        Block bb = Serializable.from(Helper.hexToBytes((String) rt.Result), Block.class);
                                    }
                                    ...
                                }
                            }
                        }
                    } catch (Exception e) {
                        throw new Exception("waitResult error" + e.getMessage());
                    }
                }
```


* 4 Send a heartbeat every 6 seconds to maintain the socket link


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
                ontSdk.getWebSocket().setReqId(i);
                ontSdk.getWebSocket().sendSubscribe(map);     
                Thread.sleep(6000);
            }
```


* 5 Push Events in detail


Take calling the put function of the attest contract as an example.

//The content of attest contract abi.json is as follows:

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

When calling the put function to save the data, the putRecord event is triggered, and the result of the websocket push is a hexadecimal string of {"putRecord", "arg1", "arg2", "arg3"}
Example：

```
RECV: {
    "Action": "Log",
    "Desc": "SUCCESS",
    "Error": 0,
    "Result": {
        "Message": "Put",
        "TxHash": "8cb32f3a1817d88d8562fdc0097a0f9aa75a926625c6644dfc5417273ca7ed71",
        "ContractAddress": "80f6bff7645a84298a1a52aa3745f84dba6615cf"
    },
    "Version": "1.0.0"
}
RECV: {
    "Action": "Notify",
    "Desc": "SUCCESS",
    "Error": 0,
    "Result": [
        {
            "States": [
                "7075745265636f7264",
                "507574",
                "6b6579",
                "7b2244617461223a7b22416c6772697468656d223a22534d32222c2248617368223a22222c2254657874223a2276616c75652d7465737431222c225369676e6174757265223a22227d2c2243416b6579223a22222c225365714e6f223a22222c2254696d657374616d70223a307d"
            ],
            "TxHash": "8cb32f3a1817d88d8562fdc0097a0f9aa75a926625c6644dfc5417273ca7ed71",
            "ContractAddress": "80f6bff7645a84298a1a52aa3745f84dba6615cf"
        }
    ],
    "Version": "1.0.0"
}
```

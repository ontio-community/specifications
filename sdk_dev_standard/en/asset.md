# Digital Asset ONT&ONG Transaction

## Interface list

| Method Name | Parameters | Return Type | Description |
|:--|:---|:---|:--|
| sendTransfer       |String assetName, String sendAddr, String password, String recvAddr, long amount      | String|Transfer assets between two addresses|
|sendTransferToMany  |String assetName, String sendAddr, String password, String[] recvAddr, long[] amount  |String |Transfer assets between multiple addresses|
|sendTransferFromMany|String assetName, String[] sendAddr, String[] password, String recvAddr, long[] amount|String |Multiple addresses transfer assets to one address|
|sendOngTransferFrom |String sendAddr, String password, String to, long amount |String |Transfer ong assets|

## Interface definition

#### sendTransfer

* Description
Transfer assets
* Input parameters
String assetName, String sendAddr, String password, String recvAddr, long amount
assetName: Asset name，
sendAddr: Sender address，
password: Sender password，
recvAddr: Receiver address，
amount: The amount of transfer
* Output parameters
Transaction hash
* Exception handling

| Error code | Occurrence scenario |                              
|:--------| :------                                               
|58012    | asset name error |
|58004    | param error |
|58023    | Invalid url |   
| Unknown | Ontology internal error |                                  

#### sendTransferToMany
* Description
Transfer assets to multiple accounts
* Input parameters
String assetName, String sendAddr, String password, String[] recvAddr, long[] amount
assetName: Asset name，
sendAddr: Sender address，
password: Sender password，
recvAddr: Array of receiver address，
amount: The amount array of transfer
Amount and recvAddr correspond one by one 
* Output parameters
Transaction hash
* Exception handling

| Error code | Occurrence scenario |                              
|:--------| :------                                               
|58012    | asset name error |
|58004    | param error |
|58023    | Invalid url |   
| Unknown | Ontology error |  

#### sendTransferFromMany
* Description
Transfer assets from multiple accounts to one account
* Input parameters
String assetName, String[] sendAddr, String[] password, String recvAddr, long[] amount
assetName: Asset name，
sendAddr: Array of sender address，
password: Array of sender password，
recvAddr: Receiver address，
amount: The amount array of transfer
Amount, sendAddr and password correspond one by one
* Output parameters
Transaction hash
* Exception handling

| Error code | Occurrence scenario |                              
|:--------| :------                                               
|58012    | asset name error |
|58004    | param error|
|58023    | Invalid url|   
|4XXXX    | Ontology error|

#### sendOngTransferFrom
* Description
Extract ong to one account
* Input parameters
String sendAddr, String password, String to, long amount
sendAddr: Sender address，
password: Sender password，
to: Receiver address，
amount: The amount of transfer
* Output parameters
Transaction hash
* Exception handling

| Error code | Occurrence scenario |                              
|:--------| :------                                               
|58012    | asset name error |
|58004    | param error|
|58023    | Invalid url|   
|4XXXX    | Ontology error|

## Transaction description

Ont and ong contract address
```
private final String ontContract = "ff00000000000000000000000000000000000001";
private final String ongContract = "ff00000000000000000000000000000000000002";
```

State class field：
```
public class State implements Serializable {
    public byte version;
    public Address from;
    public Address to;
    public BigInteger value;
    ...
  }
```

Transfers class field：
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
Contract field:

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
* Construct parameters paramBytes

```
State state = new State(senderAddress, receiveAddress, new BigInteger(amount));
Transfers transfers = new Transfers(new State[]{state});
Contract contract = new Contract((byte) 0,null, Address.parse(contractAddr), "transfer", transfers.toArray());
byte[] paramBytes = contarct.toArray();
```
* Construct transaction
Please refer to the section of construction transaction of smart contract 
* Send transaction 
Please refer to the section of send transaction of smart contract 

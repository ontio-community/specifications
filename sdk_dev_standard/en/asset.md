<h1 align="center">Digital Asset ONT&ONG Transaction</h1>
<p align="center" class="version">Version 0.7.0 </p>

## Interface list

| Method Name | Parameters | Return Type | Description |
|:--|:---|:---|:--|
| sendTransfer       |String assetName, String sendAddr, String password, String recvAddr, long amount      | String|Transfer assets between two addresses|
|sendTransferToMany  |String assetName, String sendAddr, String password, String[] recvAddr, long[] amount  |String |Transfer assets between multiple addresses|
|sendTransferFromMany|String assetName, String[] sendAddr, String[] password, String recvAddr, long[] amount|String |Multiple addresses transfer assets to one address|
|sendOngTransferFrom |String sendAddr, String password, String to, long amount |String |Transfer ong assets|
|sendApprove|String assetName ,String sendAddr, String password, String recvAddr, long amount,long gas|||
|sendTransferFrom|String assetName ,String sendAddr, String password, String fromAddr, String toAddr, long amount,long gas|||
|sendAllowance|String assetName,String fromAddr,String toAddr|long|Query the amount of toAddr which is authorized by fromAddr|
|sendBalanceOf|String assetName, String address|long|Check Balance|
|queryName|String assetName|String|Query details about asset name|
|querySymbol|String assetName|String|Query the information about asset symbol|
|queryDecimals|String assetName|long|Query the precision of asset|
|queryTotalSupply|String assetName|long|Query the total supply of asset|

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
|58101    | asset name error |
|58402    | Invalid url|  
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology internal error|  
                                

#### sendTransferToMany
* Description
Transfer assets to multiple accounts

* Input parameters

String assetName, String sendAddr, String password, String[] recvAddr, long[] amount,long gas

assetName: Asset name，

sendAddr: Sender address，

password: Sender password，

recvAddr: Array of receiver address，

amount: The amount array of transfer

gas: The number of gas

Amount and recvAddr correspond one by one

* Output parameters

Transaction hash

* Exception handling

| Error code | Occurrence scenario |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|  
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology internal error|  

#### sendTransferFromMany
* Description

Transfer assets from multiple accounts to one account

* Input parameters

String assetName, String[] sendAddr, String[] password, String recvAddr, long[] amount, long gas

assetName: Asset name，

sendAddr: Array of sender address，

password: Array of sender password，

recvAddr: Receiver address，

amount: The amount array of transfer

gas: The number of gas

Amount, sendAddr and password correspond one by one

* Output parameters

Transaction hash

* Exception handling

| Error code | Occurrence scenario |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|  
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology internal error|  

#### sendOngTransferFrom

* Description

Extract ong to one account

* Input parameters

String sendAddr, String password, String to, long amount, long gas

sendAddr: Sender address，

password: Sender password，

to: Receiver address，

amount: The amount of transfer

gas: The number of gas

* Input parameters

Transaction hash

* Exception handling

|   Error code |  Occurrence scenario        |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology error|

#### sendApprove

* Description

Authorize other accounts to transfer a certain amount of assets

* Input parameters

String assetName ,String sendAddr, String password, String recvAddr, long amount,long gas

assetName: Asset name,

sendAddr: Sender address，

password: Sender password，

recvAddr: Receiver address，

amount: The amount of transfer

gas: The number of gas

* Output parameters

Transaction hash

* Exception handling

| Error code | Occurrence scenario |                              
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|  
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology internal error|  

#### sendTransferFrom

* Description

Used with sendApprove, the transaction initiator transfers a certain amount of assets from one account to another

* Input parameters

String assetName ,String sendAddr, String password, String fromAddr, String toAddr, long amount,long gas

assetName: Asset name

sendAddr: Sender address，

password: Sender password，

fromAddr: Asset transferee address，

toAddr: Asset receiver address，

amount: The amount of transfer

gas: The number of gas

* Output parameters

Transaction hash

* Exception handling

| Error code | Occurrence scenario |                               
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|58105    |OntAsset Error,amount is less than or equal to zero|   
|51015    |Account Error,encryptedPriKey address password not match.|
|4XXXX    | Ontology error|

#### sendAllowance

* Description

Used with sendApprove to query the number of assets authorized for transfer

* Input parameters
String assetName,String fromAddr,String toAddr

assetName: Asset name

fromAddr: Authorized asset transferee address

toAddr: Authorized asset receiver address

* Output parameters

The number of assets authorized for transfer

* Exception handling

| Error code | Occurrence scenario |                             
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology error|

#### sendBalanceOf

* Description

Check balance

* Input parameters

String assetName, String address

assetName: Asset name

address: Asset address

* Output parameters

Asset balance

* Exception handling

| Error code | Occurrence scenario |                           
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology error|

#### queryName

* Description

Query details about asset name

* Input parameters

String assetName

* Output parameters

Details about asset name

* Exception handling

| Error code | Occurrence scenario |                             
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology error|

#### querySymbol

* Description

Query asset symbol information

* Input parameters

String assetName

assetName: Asset name

* Output parameters

Asset symbol information

* Exception handling

| Error code | Occurrence scenario |                           
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology error|

#### queryDecimals

* Description

Query precision

* Input parameters

String assetName

assetName: Asset name

* Output parameters

Asset precision

* Exception handling

| Error code | Occurrence scenario |                            
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
|4XXXX    | Ontology error|

#### queryTotalSupply

* Description

Query the total supply of assets

* Input parameters

String assetName

assetName: Asset name

* Output parameters

The total supply of assets

* Exception handling

| Error code | Occurrence scenario |                           
|:--------| :------                                               
|58101    | asset name error |
|58402    | Invalid url|   
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
    public Address from;
    public Address to;
    public long value;
    ...
  }
```

Transfers class field：
```
public class Transfers implements Serializable {
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

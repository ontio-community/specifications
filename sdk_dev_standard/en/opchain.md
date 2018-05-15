<h1 align="center">Communication with the Chain Standard</h1>
<p align="center" class="version">Version 0.7.0 </p>

## 3 Communication with the chain

The Ontology chain supports Restful, RPC, and Websocket connections.

|  Connection    | Port  |
|:--------    |:--   |
|   restful   | 20334|
|   rpc       | 20336|
|   websocket | 20335|

### 3.1 Basic Interface

1 boolean sendRawTransaction(Transaction tx)
* Description
Send transaction
* Input parameters
Transaction instance
* Output parameters
true represents success
false represents failure
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------     |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

2 boolean sendRawTransaction(String hexData)
* Description
Send transaction
* Input parameters
The hexadecimal string form of the transaction instance
* Output parameters
true represents success
false represents failure
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------      |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

3 Object sendRawTransactionPreExec(String hexData)
* Description
Send pre-executed transactions. Pre-execution will not modify the data on the chain and it does not participate in the consensus.
* Input parameters
The hexadecimal string form of the transaction instance
* Output parameters
objects
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------                                               |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

4 Transaction  getTransaction(String txhash)
* Description
Get transaction by transaction hash
* Input parameters
transaction hash，remove the previous "0x"
* Output parameters
Transaction
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------                                               |
|58023    | Invalid URL |
|53001    |Transaction deserialize failed|
|4XXXX    | Ontology error|

5 Object getTransactionJson(String txhash)
* Description
Get transaction json data by transaction hash
* Input parameters
Transaction hash，remove the previous "0x"
* Output parameters
Transaction json objects
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------       |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

6 int getGenerateBlockTime()
* Description
Return block-generated time
* Input parameters
* Output parameters
Block-generated time
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------     |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

7 int getNodeCount()
* Description
Get the number of connected nodes
* Input parameters
* Output parameters
Block-generated time
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------    |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

8 int getBlockHeight()
* Description
Get block height
* Input parameters
* Output parameters
Current block height
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------     |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

9 Block getBlock(int height)
* Description
Get block by height
* Input parameters
Block height
* Output parameters
The block corresponding to the height
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------       |
|58023    | Invalid URL |
|53002    | Block deserialize failed|
|4XXXX    | Ontology error|

10 Block getBlock(String hash)
* Description
Get block by block hash
* Input parameters
Block hash
* Output parameters
The block corresponding to the block hash
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------       |
|58023    | Invalid URL |
|53002    | Block deserialize failed|
|4XXXX    | Ontology error|

11 Object   getBalance(String address)
* Description
Get balance by account address
* Input parameters
Account address
* Output parameters
Account balance
* Exception handling

| Error Code |  Occurrence Scenario        |
|:--------| :------      |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

12 Object getBlockJson(int height)
* Description
Get JSON format data of block data by block height
* Input parameters
Block height
* Output parameters
Block json data corresponding to the block height
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------    |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

13 Object getBlockJson(String hash)
* Description
Get JSON format data of block data by block hash
* Input parameters
Block hash
* Output parameters
Block json data corresponding to the block hash
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------     |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

14 Object getContract(String hash)
* Description
Get contract code by contract hash
* Input parameters
Contract hash
* Output parameters
Contract code corresponding to the contract hash
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------                                               |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

15 Object getContractJson(String hash)
* Description
Get contract code json data by contract hash
* Input parameters
Contract hash
* Output parameters
Contract code (Json) corresponding to the contract hash
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------                                               |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

16 Object getSmartCodeEvent(int height)
* Description
Get contract event by block height
* Input parameters
Block height
* Output parameters
Event object
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------     |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

17 Object getSmartCodeEvent(String hash)
* Description
Get contract event by transaction hash
* Input parameters
Transaction hash
* Output parameters
Event object
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------       |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

18 int getBlockHeightByTxHash(String hash)
* Description
Get block height by transaction hash
* Input parameters
Transaction hash
* Output parameters
Block height
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------      |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

19 String getStorage(String codehash, String key)
* Description
Get the stored data by contract's key
* Input parameters
codeHash is the codeAddress of the deployed contract, and key is a hexadecimal string
* Output parameters
The data stored by the contract's key, and the value type is string
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------      |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

20 Object getMerkleProof(String txhash)
* Description
Get Merkle proof of transaction hash to prove the transaction exists on the chain
* Input parameters
Transaction hash
* Output parameters
MerkleProof proof object
* Exception handling

|  Error Code |  Occurrence Scenario        |
|:--------| :------     |
|58023    | Invalid URL |
|4XXXX    | Ontology error|

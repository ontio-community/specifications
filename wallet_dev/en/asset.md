
[中文](../cn/asset.md) | Enlish

<h1 align="center">Digital Asset Management </h1>
<p align="center" class="version">Version 0.9.0 </p>


# Wallet

Wallet is a data storing file in JSON format. In Ontology, Wallet can store not only the digital identity but also digital assets.

[Wallet Specification](Wallet_Specification_en.md)


## Create a Wallet

Users could create their wallet from scratch.

Users only need to pass the name of their wallets.

````
import {Wallet} from 'ontology-ts-sdk';
var wallet = Wallet.create('my_wallet')
````

## Create a random account

We can generate a random private key with specific keypair algorithm and elliptic curve. There are three kinds of algorithms we support:

* ECDSA
* SM2
* EDDSA

ECDSA is the default one. You can check TS SDK API reference for info.

Then we can create the account and add it to the wallet.

````
import {Account, Crypto} from 'ontology-ts-sdk';
import { Crypto } from 'ontology-ts-sdk';

cont keyType = Crypto.KeyType.ECDSA;

const keyParameters = new Crypto.KeyParameters(Crypto.CurveLabel.SECP256r1);

const privateKey = Crypto.PrivateKey.random(keyType, keyParameters)

var account = Account.create( privateKey, password, name );

wallet.addAccount(account)

````

|  Param    | Desc  |
|:--------    |:--   |
|   privateKey      | An instance of class PrivateKey..|
|   password      | User's password to encrypt the private key. |
|   name       |  Name of the account. |
|   amount   | ONG  Need to multiply 1e9 to keep precision.|
|   gasPrice | Gas price.|
|   gasLimit | Gas limit.|
|   payer | Payer's address to pay for the transaction gas.|


## Create an account from mnemonic code



## Create an account from WIF 


## import account


## import and export keystore


# Account
Account is used to manage user's assets.


##  Create an Account

````
import {Account} from 'ontology-ts-sdk'
//@param {PrivateKey} The user's private key
//@param {string} The user's password
//@param {string} Optional. Name of the account
//@param {object} Optional parameter. The encryption algorithm object.
var account = Account.create(privateKey, password, label, params)
````

## Import an Account

Users can import an account by the backup data.

This method will check the password and the private key, an error will be thrown if they are not match.

````
import { Account } from 'ontology-ts-sdk'
//@param label {srint} Name of the account
//@param encryptedPrivateKey {PrivateKey} The encrypted private key
//@param password {string} The password used to decrypt private key
//@param address {Address} The address of the account
//@param saltBase64 {string} The salt in base64 format
//@param params {ScryptParams} Optional scrypt params to decrypt private key
var account;
try {
    account = Account.importAccount(label, encryptedPrivateKey, password, address, saltBase64, params);
} catch(error) {
    //password or private key incorrect
}
````



# Native Asset

There are two kinds of native asset in Ontology: ONT and ONG.

In order to transfer native asset, we can create the specific transaction and send it to the blockchain. After the transaction has been packaged in the block, the transaction will succeed.

Type of native asset:
````
TOKEN_TYPE = {
  ONT : 'ONT',  //Ontology Token
  ONG : 'ONG'   //Ontology Gas
}
````

## Query Balance

We can use RESTful API, RPC API and WebSocket API to query the balance. Here we use RESTful API as example.

### Example:

````typescript
const address = new Address('AXpNeebiUZZQxLff6czjpHZ3Tftj8go2TF');
const rest = new RestClient();
rest.getBalance(address).then(res -> {
	console.log(res)
})
````
The result contains balance of ONT and ONG.


## Transfer asset

### Create transfer transaction

First we need to create the transaction for transfer.

````typescript
import {OntAssetTxBuilder} from 'ontology-ts-sdk'
//supppose we have an account with enough ONT and ONG
//Sender's address
const from = account.address;
//Receiver's address
const to = new Address('AXpNeebiUZZQxLff6czjpHZ3Tftj8go2TF')
//Amount to send
const amount = 100
//Asset type
const assetType = 'ONT'
//Gas price and gas limit are to compute the gas costs of the transaction.
const gasPrice = '500';
const gasLimit = '20000';
//Payer's address to pay for the transaction gas
const payer = from;
const tx = OntAssetTxBuilder.makeTransferTx(assetType, from, to, amount, gasPrice, gasLimit, payer);
````

|  Param    | Desc  |
|:--------    |:--   |
|   assetType      | ONT or ONG.|
|   from      | Sender's address to withdraw ONG.|
|   to       |  Receiver's address to receive ONG.|
|   amount   | ONG  Need to multiply 1e9 to keep precision.|
|   gasPrice | Gas price.|
|   gasLimit | Gas limit.|
|   payer | Payer's address to pay for the transaction gas.|




## Withdraw ong

### Create withdraw transaction

Withdraw generated ONG from user's account address and send to other address. They can be the same address.

````typescript
import {OntAssetTxBuilder} from 'ontology-ts-sdk'

//suppose we have an account already
const from = account.address;
const to = account.address;
const amount = 10 * 1e9;
const gasPrice = '500';
const gasLimit = '20000';
const payer = account.address;
const tx = OntAssetTxBuilder.makeWithdrawOngTx(from, to, amount, payer, gasPrice, gasLimit);
````

|  Param    | Desc  |
|:--------    |:--   |
|   from      | Sender's address to withdraw ONG.|
|   to       |  Receiver's address to receive ONG.|
|   amount   | ONG  Need to multiply 1e9 to keep precision.|
|   gasPrice | Gas price.|
|   gasLimit | Gas limit.|
|   payer | Payer's address to pay for the transaction gas.|



## Sign and Send transaction

We can use RESTful API, RPC API, or WebSocket API to send a transaction. Here we use RESTful API as an example.


> RestClient.getSmartCodeEvent

````typescript
//sign transaction before send it
import {RestClient, CONST, TransactionBuilder} from 'ontology-ts-sdk'

//we already got the transaction we created before

//we have to sign the transaction before sent it
//Use user's private key to sign the transaction
TransactionBuilder.signTransaction(tx, privateKey)

const rest = new RestClient(CONST.TEST_ONT_URL.REST_URL);
rest.sendRawTransaction(tx.serialize()).then(res => {
	console.log(res)	
})

````

> Use WebSocket API and wait for the transaction notice.

````typescript
import {RestClient, CONST, TransactionBuilder} from 'ontology-ts-sdk'

//we already got the transaction we created before

//we have to sign the transaction before sent it
//Use user's private key to sign the transaction
TransactionBuilder.signTransaction(tx, privateKey)

const rest = new RestClient(CONST.TEST_ONT_URL.REST_URL);
rest.sendRawTransaction(tx.serialize()).then(res => {
	console.log(res)	
})

````

The result may look like:

```
{ 
  Action: 'sendrawtransaction',
  Desc: 'SUCCESS',
  Error: 0,
  Result: 'dfc598649e0f3d9ff94486a80020a2775e1d474b843255f8680a3ac862c58741',
  Version: '1.0.0' 
}
```

The `Result` of the response is the transaction hash, it can be used to query the event of the transaction.

Then we can query the balance to check if the withdraw succeeded.

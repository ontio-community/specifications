<h1 align="center">Digital Identity SDK Development Standard</h1>
<p align="center" class="version">Version 0.7.0 </p>

For an introduction of digital identity, please refer to [ONT ID Identity Protocol and Trust Framework](https://github.com/ontio/ontology-DID).

## Interface list

|   Method Name      |  Parameters                                                                     |  Return Type  | Description         | Necessary |
|:--------          | :------                                                                  |:------------ |:-------     |:-------     |
|sendRegister       |  Identity ident, String password                                         |    Identity  |Register ontId on the Blockchain|Y
|sendRegister       |   String  password                                                       |    Identity  |           |
|sendRegister       | String label,String password                                             |    Identity  |           |
|sendRegister       |String password, Map<String, Object> attrsMap                             |    Identity  |           |
|sendAddPubKey      |String ontid, String password, String newpubkey                           |    String    |Add a public key to ontId|
|sendAddPubKey      |String password, String ontid, String newpubkey, String recoveryScriptHash|    String    |           |
|sendRemovePubKey   |String ontid, String password, String removepk                            |    String    |Delete a public key    |
|sendRemovePubKey   |String ontid, String password, byte[] key, String recoveryScriptHash      |    String    |           |
|sendAddRecovery    |String ontid, String password, String recoveryScriptHash                  |    String    |Add recovery |
|sendChangeRecovery |String ontid, String password, String newRecoveryScriptHash               |    String    |Modify recovery  |
|sendUpdateAttribute|String ontid, String password, byte[] path, byte[] type, byte[] value     |    String    |Update attribute     |
|sendGetDDO         |String ontid                                                              |    String    |Get DDo           |
|createOntIdClaim   |String signerOntid,String password, String context, Map<String, Object> claimMap, Map metaData|    String    | Create ontId declaration |
|verifyOntIdClaim   |String claim                                                              |    String    |Verify ontId declaration           |
|getProof           |String txhash                                                             |    Object    | Get merkle proof           |
|verifyMerkleProof  |String claim                                                              |    boolean   |Verify merkle proof    |
|sendRemoveAttribute|String ontid, String password, byte[] path                                |    String    |Delete attribute       |
|sendGetPublicKeyId |String ontid,String password                                              |    String    |Get public key Id         |
|sendGetPublicKeyStatus|String ontid,String password,byte[] pkId                               |    String    |Get public key status     |

## Interface definition

#### sendRegister

* Description
Register ontId on the blockchain
* Input parameters
Identity ident, String password
* Output parameters

* Exception handling

|   Error Code           |  Occurrence Scenario        |                              
|:--------          | :------                                               
|10001       |  XXX                                       

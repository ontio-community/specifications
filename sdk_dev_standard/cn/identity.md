<h1 align="center">数字身份SDK开发标准</h1>
<p align="center" class="version">Version 0.7.0 </p>


数字身份相关介绍可参考[ONT ID 身份标识协议及信任框架](https://github.com/ontio/ontology-DID)。

## 接口列表

|   方法名           |  参数                                                                     |   返回值类型  | 描述         | 必需 |
|:--------          | :------                                                                  |:------------ |:-------     |:-------     |
|sendRegister       |  Identity ident, String password                                         |    Identity  |向链上注册ontId|是
|sendRegister       |   String  password                                                       |    Identity  |           |
|sendRegister       | String label,String password                                             |    Identity  |           |
|sendRegister       |String password, Map<String, Object> attrsMap                             |    Identity  |           |
|sendAddPubKey      |String ontid, String password, String newpubkey                           |    String    |给ontId添加公钥|
|sendAddPubKey      |String password, String ontid, String newpubkey, String recoveryScriptHash|    String    |           |
|sendRemovePubKey   |String ontid, String password, String removepk                            |    String    |删除公钥    |
|sendRemovePubKey   |String ontid, String password, byte[] key, String recoveryScriptHash      |    String    |           |
|sendAddRecovery    |String ontid, String password, String recoveryScriptHash                  |    String    |添加recovery |
|sendChangeRecovery |String ontid, String password, String newRecoveryScriptHash               |    String    |修改recovery  |
|sendUpdateAttribute|String ontid, String password, byte[] path, byte[] type, byte[] value     |    String    |更新属性     |
|sendGetDDO         |String ontid                                                              |    String    |获得DDo           |
|createOntIdClaim   |String signerOntid,String password, String context, Map<String, Object> claimMap, Map metaData|    String    | 创建ontId声明 |
|verifyOntIdClaim   |String claim                                                              |    String    |验证ontId声明           |
|getProof           |String txhash                                                             |    Object    | 获得merkle证明          |
|verifyMerkleProof  |String claim                                                              |    boolean   |验证merkle证明    |
|sendRemoveAttribute|String ontid, String password, byte[] path                                |    String    |删除属性       |
|sendGetPublicKeyId |String ontid,String password                                              |    String    |获得公钥Id         |
|sendGetPublicKeyStatus|String ontid,String password,byte[] pkId                               |    String    |获得公钥状态    |

## 接口定义

#### sendRegister

* 描述
向链上注册ontId
* 输入参数
Identity ident, String password
* 输出参数

* 异常处理

|   错误码           |  发生场景        |                              
|:--------          | :------                                               
|10001       |  XXX                                       




# 附件

### 错误码定义

| 返回代码 | 描述信息 | 说明 |
| :---- | ----------------------------- | ----------------- |
| 0 | SUCCESS | 成功 |
| 41001 | SESSION_EXPIRED | 会话无效或已过期（ 需要重新登录） |
| 41002 | SERVICE_CEILING | 达到服务上限 |
| 41003 | ILLEGAL_DATAFORMAT | 不合法数据格式 |
| 41004 | INVALID_VERSION| 不合法的版本 |
| 42001 | INVALID_METHOD | 无效的方法 |
| 42002 | INVALID_PARAMS | 无效的参数 |
| 43001 | INVALID_TRANSACTION | 无效的交易 |
| 43002 | INVALID_ASSET | 无效的资产 |
| 43003 | INVALID_BLOCK | 无效的块 |
| 44001 | UNKNOWN_TRANSACTION | 找不到交易 |
| 44002 | UNKNOWN_ASSET | 找不到资产 |
| 44003 | UNKNOWN_BLOCK | 找不到块 |
| 44004 | UNKNWN_CONTRACT | 找不到合约 |
| 45001 | INTERNAL_ERROR | 内部错误 |
| 47001 | SMARTCODE_ERROR| 智能合约错误 |
|51001  |  InvalidParams |Account Error,invalid params|
|51002  |  UnsupportedKeyType |Account Error,unsupported key type|
|51003  |  InvalidMessage |Account Error,invalid message|
|51004  |  WithoutPrivate |Account Error,account without private key cannot generate signature|
|51005  |  InvalidSM2Signature |Account Error,invalid SM2 signature parameter, ID (String) excepted|
|51006  |  AccountInvalidInput |Account Error,account without public key cannot verify signature|
|51007  |  AccountWithoutPublicKey |Account Error,unknown key type|
|51008  |  UnknownKeyType |Account Error,null input|
|51009  |  NullInput |Account Error,invalid data|
|51010  |  InvalidData |Account Error,invalid params|
|51011  |  Decoded3bytesError |Account Error,decoded 3 bytes error|
|51012  |  DecodePrikeyPassphraseError |Account Error,decode prikey passphrase error|
|51013  |  PrikeyLengthError |Account Error,Prikey length error|
|52001  |  InputError |Uint256 Error,input error|
|52002  |  ChecksumNotValidate |Base58 Error,Checksum does not validate|
|52003  |  InputTooShort |Base58 Error,Input too short|
|52004  |  UnknownCurve |Curve Error,unknown curve|
|52005  |  UnknownCurveLabel |Curve Error,unknown curve label|
|52006  |  UnknownAsymmetricKeyType |keyType Error,unknown asymmetric key type|
|52007  |  InvalidSignatureData |Signature Error,invalid signature data: missing the ID parameter for SM3withSM2|
|52008  |  InvalidSignatureDataLen |Signature Error,invalid signature data length|
|52009  |  MalformedSignature |Signature Error,malformed signature|
|52010  |  UnsupportedSignatureScheme |Signature Error,unsupported signature scheme:|
|53001  |  TxDeserializeError |Core Error,Transaction deserialize failed|
|53002  |  BlockDeserializeError |Core Error,Block deserialize failed|
|58001  |  SendRawTxError |SmartCodeTx Error,sendRawTransaction error|
|58002  |  TypeError |SmartCodeTx Error,type error|
|58003  |  NullCodeHash |OntIdTx Error,null codeHash|
|58004  |  ParamError |OntIdTx Error,param error|
|58005  |  DidNull |OntIdTx Error,SendDid or receiverDid is null in metaData|
|58006  |  NotExistCliamIssuer |OntIdTx Error,Not exist cliam issuer|
|58007  |  NotFoundPublicKeyId |OntIdTx Error,not found PublicKeyId|
|58008  |  PublicKeyIdErr |OntIdTx Error,PublicKeyId err|
|58009  |  BlockHeightNotMatch |OntIdTx Error,BlockHeight not match|
|58010  |  NodesNotMatch |OntIdTx Error,nodes not match|
|58011  |  ResultIsNull |OntIdTx Error,result is null|
|58012  |  AssetNameError |OntAsset Error,asset name error|
|58013  |  DidError |OntAsset Error,Did error|
|58014  |  NullPkId |OntAsset Error,null pkId|
|58015  |  NullClaimId |OntAsset Error,null claimId|
|58016  |  NullKeyOrValue |RecordTx Error,null key or value|
|58017  |  NullKey |RecordTx Error,null  key|
|58018  |  GetAccountByAddressErr |WalletManager Error,getAccountByAddress err|
|58019  |  WebsocketNotInit |OntSdk Error,websocket not init|
|58020  |  ConnRestfulNotInit |OntSdk Error,connRestful not init|
|58021  |  SetParamsValueValueNumError |AbiFunction Error,setParamsValue value num error|
|58022  |  InvalidUrl |Interfaces Error,Invalid url:|
|58023  |  AESailed |ECIES Error,AES failed initialisation -|
|59000  |  OtherError| other error|

### NeoVM字节码

```
public enum ScriptOp {
    // Constants
    OP_0(0x00), // An empty array of bytes is pushed onto the stack. (This is not a no-op: an item is added to the stack.)
    OP_FALSE(OP_0),
    OP_PUSHBYTES1(0x01), // 0x01-0x4B The next opcode bytes is data to be pushed onto the stack
    OP_PUSHBYTES75(0x4B),
    OP_PUSHDATA1(0x4C), // The next byte contains the number of bytes to be pushed onto the stack.
    OP_PUSHDATA2(0x4D), // The next two bytes contain the number of bytes to be pushed onto the stack.
    OP_PUSHDATA4(0x4E), // The next four bytes contain the number of bytes to be pushed onto the stack.
    OP_1NEGATE(0x4F), // The number -1 is pushed onto the stack.
    //OP_RESERVED(0x50), // Transaction is invalid unless occuring in an unexecuted OP_IF branch
    OP_1(0x51), // The number 1 is pushed onto the stack.
    OP_TRUE(OP_1),
    OP_2(0x52), // The number 2 is pushed onto the stack.
    OP_3(0x53), // The number 3 is pushed onto the stack.
    OP_4(0x54), // The number 4 is pushed onto the stack.
    OP_5(0x55), // The number 5 is pushed onto the stack.
    OP_6(0x56), // The number 6 is pushed onto the stack.
    OP_7(0x57), // The number 7 is pushed onto the stack.
    OP_8(0x58), // The number 8 is pushed onto the stack.
    OP_9(0x59), // The number 9 is pushed onto the stack.
    OP_10(0x5A), // The number 10 is pushed onto the stack.
    OP_11(0x5B), // The number 11 is pushed onto the stack.
    OP_12(0x5C), // The number 12 is pushed onto the stack.
    OP_13(0x5D), // The number 13 is pushed onto the stack.
    OP_14(0x5E), // The number 14 is pushed onto the stack.
    OP_15(0x5F), // The number 15 is pushed onto the stack.
    OP_16(0x60), // The number 16 is pushed onto the stack.


    // Flow control
    OP_NOP(0x61), // Does nothing.
    OP_JMP(0x62),
    OP_JMPIF(0x63),
    OP_JMPIFNOT(0x64),
    OP_CALL(0x65),
    OP_RET(0x66),
    OP_APPCALL(0x67),
    OP_SYSCALL(0x68),
    OP_VERIFY(0x69), // Marks transaction as invalid if top stack value is not true. True is removed, but false is not.
    OP_HALT(0x6A), // Marks transaction as invalid.


    // Stack
    OP_TOALTSTACK(0x6B), // Puts the input onto the top of the alt stack. Removes it from the main stack.
    OP_FROMALTSTACK(0x6C), // Puts the input onto the top of the main stack. Removes it from the alt stack.
    OP_2DROP(0x6D), // Removes the top two stack items.
    OP_2DUP(0x6E), // Duplicates the top two stack items.
    OP_3DUP(0x6F), // Duplicates the top three stack items.
    OP_2OVER(0x70), // Copies the pair of items two spaces back in the stack to the front.
    OP_2ROT(0x71), // The fifth and sixth items back are moved to the top of the stack.
    OP_2SWAP(0x72), // Swaps the top two pairs of items.
    OP_IFDUP(0x73), // If the top stack value is not 0, duplicate it.
    OP_DEPTH(0x74), // Puts the number of stack items onto the stack.
    OP_DROP(0x75), // Removes the top stack item.
    OP_DUP(0x76), // Duplicates the top stack item.
    OP_NIP(0x77), // Removes the second-to-top stack item.
    OP_OVER(0x78), // Copies the second-to-top stack item to the top.
    OP_PICK(0x79), // The item n back in the stack is copied to the top.
    OP_ROLL(0x7A), // The item n back in the stack is moved to the top.
    OP_ROT(0x7B), // The top three items on the stack are rotated to the left.
    OP_SWAP(0x7C), // The top two items on the stack are swapped.
    OP_TUCK(0x7D), // The item at the top of the stack is copied and inserted before the second-to-top item.


    // Splice
    OP_CAT(0x7E), // Concatenates two strings.
    OP_SUBSTR(0x7F), // Returns a section of a string.
    OP_LEFT(0x80), // Keeps only characters left of the specified point in a string.
    OP_RIGHT(0x81), // Keeps only characters right of the specified point in a string.
    OP_SIZE(0x82), // Returns the length of the input string.


    // Bitwise logic
    OP_INVERT(0x83), // Flips all of the bits in the input.
    OP_AND(0x84), // Boolean and between each bit in the inputs.
    OP_OR(0x85), // Boolean or between each bit in the inputs.
    OP_XOR(0x86), // Boolean exclusive or between each bit in the inputs.
    OP_EQUAL(0x87), // Returns 1 if the inputs are exactly equal, 0 otherwise.
    //OP_EQUALVERIFY(0x88), // Same as OP_EQUAL, but runs OP_VERIFY afterward.
    //OP_RESERVED1(0x89), // Transaction is invalid unless occuring in an unexecuted OP_IF branch
    //OP_RESERVED2(0x8A), // Transaction is invalid unless occuring in an unexecuted OP_IF branch

    // Arithmetic
    // Note: Arithmetic inputs are limited to signed 32-bit integers, but may overflow their output.
    OP_1ADD(0x8B), // 1 is added to the input.
    OP_1SUB(0x8C), // 1 is subtracted from the input.
    OP_2MUL(0x8D), // The input is multiplied by 2.
    OP_2DIV(0x8E), // The input is divided by 2.
    OP_NEGATE(0x8F), // The sign of the input is flipped.
    OP_ABS(0x90), // The input is made positive.
    OP_NOT(0x91), // If the input is 0 or 1, it is flipped. Otherwise the output will be 0.
    OP_0NOTEQUAL(0x92), // Returns 0 if the input is 0. 1 otherwise.
    OP_ADD(0x93), // a is added to b.
    OP_SUB(0x94), // b is subtracted from a.
    OP_MUL(0x95), // a is multiplied by b.
    OP_DIV(0x96), // a is divided by b.
    OP_MOD(0x97), // Returns the remainder after dividing a by b.
    OP_LSHIFT(0x98), // Shifts a left b bits, preserving sign.
    OP_RSHIFT(0x99), // Shifts a right b bits, preserving sign.
    OP_BOOLAND(0x9A), // If both a and b are not 0, the output is 1. Otherwise 0.
    OP_BOOLOR(0x9B), // If a or b is not 0, the output is 1. Otherwise 0.
    OP_NUMEQUAL(0x9C), // Returns 1 if the numbers are equal, 0 otherwise.
    //OP_NUMEQUALVERIFY(0x9D), // Same as OP_NUMEQUAL, but runs OP_VERIFY afterward.
    OP_NUMNOTEQUAL(0x9E), // Returns 1 if the numbers are not equal, 0 otherwise.
    OP_LESSTHAN(0x9F), // Returns 1 if a is less than b, 0 otherwise.
    OP_GREATERTHAN(0xA0), // Returns 1 if a is greater than b, 0 otherwise.
    OP_LESSTHANOREQUAL(0xA1), // Returns 1 if a is less than or equal to b, 0 otherwise.
    OP_GREATERTHANOREQUAL(0xA2), // Returns 1 if a is greater than or equal to b, 0 otherwise.
    OP_MIN(0xA3), // Returns the smaller of a and b.
    OP_MAX(0xA4), // Returns the larger of a and b.
    OP_WITHIN(0xA5), // Returns 1 if x is within the specified range (left-inclusive), 0 otherwise.


    // Crypto
    OP_RIPEMD160(0xA6), // The input is hashed using RIPEMD-160.
    OP_SHA1(0xA7), // The input is hashed using SHA-1.
    OP_SHA256(0xA8), // The input is hashed using SHA-256.
    OP_HASH160(0xA9), // The input is hashed twice: first with SHA-256 and then with RIPEMD-160.
    OP_HASH256(0xAA), // The input is hashed two times with SHA-256.
    //OP_CODESEPARATOR(0xAB), // All of the signature checking words will only match signatures to the data after the most recently-executed OP_CODESEPARATOR.
    OP_CHECKSIG(0xAC), // The entire transaction's outputs, inputs, and script (from the most recently-executed OP_CODESEPARATOR to the end) are hashed. The signature used by OP_CHECKSIG must be a valid signature for this hash and public key. If it is, 1 is returned, 0 otherwise.
    //OP_CHECKSIGVERIFY(0xAD), // Same as OP_CHECKSIG, but OP_VERIFY is executed afterward.
    OP_CHECKMULTISIG(0xAE), // For each signature and public key pair, OP_CHECKSIG is executed. If more public keys than signatures are listed, some key/sig pairs can fail. All signatures need to match a public key. If all signatures are valid, 1 is returned, 0 otherwise. Due to a bug, one extra unused value is removed from the stack.
    //OP_CHECKMULTISIGVERIFY(0xAF), // Same as OP_CHECKMULTISIG, but OP_VERIFY is executed afterward.


    // Array
    OP_ARRAYSIZE(0xC0),
    OP_PACK(0xC1),
    OP_UNPACK(0xC2),
    OP_DISTINCT(0xC3),
    OP_SORT(0xC4),
    OP_REVERSE(0xC5),
    OP_CONCAT(0xC6),
    OP_UNION(0xC7),
    OP_INTERSECT(0xC8),
    OP_EXCEPT(0xC9),
    OP_TAKE(0xCA),
    OP_SKIP(0xCB),
    OP_PICKITEM(0xCC),
    OP_ALL(0xCD),
    OP_ANY(0xCE),
    OP_SUM(0xCF),
    OP_AVERAGE(0xD0),
    OP_MAXITEM(0xD1),
    OP_MINITEM(0xD2),
    ;
    private byte value;

    ScriptOp(int v) {
        value = (byte)v;
    }

    ScriptOp(ScriptOp v) {
        value = v.value;
    }

    public byte getByte() {
        return value;
    }

    public static ScriptOp valueOf(int v) {
        for (ScriptOp op : ScriptOp.values()) {
            if (op.value == (byte)v) {
                 return op;
            }
        }
        return null;
    }
}
```

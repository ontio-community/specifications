<h1 align="center">账户基本功能标准</h1>
<p align="center" class="version">Version 0.7.0 </p>


## 1.1 公私钥对生成

目前Ontology支持的算法

| ID | Algorithm |
|:--|:--|
|0x12|ECDSA|
|0x13|SM2|
|0x14|EdDSA|

ONT可签名方案说明( with 前面是散列算法，后面是签名算法)，支持的signature schemes：
```
SHA224withECDSA
SHA256withECDSA
SHA384withECDSA
SHA512withECDSA
SHA3-224withECDSA
SHA3-256withECDSA
SHA3-384withECDSA
SHA3-512withECDSA
RIPEMD160withECDSA
SM3withSM2
SHA512withEdDSA
```

公私钥和签名的序列化方法请参考https://github.com/ontio/ontology-crypto/wiki/ECDSA




Account类功能：
* 生成公私钥对：指定hash散列算法和签名算法，获得加密算法框架实例并初始化，产生公私钥对。
* 根据公钥计算U160地址和base58地址
* 公钥、私钥序列化
* 私钥加解密

```
public class Account {
    private KeyType keyType; //签名算法
    private Object[] curveParams;//椭圆曲线域参数
    private PrivateKey privateKey;//私钥
    private PublicKey publicKey;//公钥
    private Address addressU160;//U160地址，公钥转换而来
    private SignatureScheme signatureScheme;//签名scheme
```

* java获得公私钥对的示例
方法一，随机生成公私钥：
```
public Account(SignatureScheme scheme) throws Exception {
        Security.addProvider(new BouncyCastleProvider());
        KeyPairGenerator gen;
        AlgorithmParameterSpec paramSpec;
        signatureScheme = scheme;

        if (scheme == SignatureScheme.SHA256WITHECDSA) {
            this.keyType = KeyType.ECDSA;
            this.curveParams = new Object[]{Curve.P256.toString()};
        } else if (scheme == SignatureScheme.SM3WITHSM2) {
            this.keyType = KeyType.SM2;
            this.curveParams = new Object[]{Curve.SM2P256V1.toString()};
        }

        switch (scheme) {
            case SHA256WITHECDSA:
            case SM3WITHSM2:
                if (!(curveParams[0] instanceof String)) {
                    throw new Exception(ErrorCode.InvalidParams);
                }
                String curveName = (String) curveParams[0];
                paramSpec = new ECGenParameterSpec(curveName);
                gen = KeyPairGenerator.getInstance("EC", "BC");
                break;
            default:
                //should not reach here
                throw new Exception(ErrorCode.UnsupportedKeyType);
        }
        gen.initialize(paramSpec, new SecureRandom());
        KeyPair keyPair = gen.generateKeyPair();
        this.privateKey = keyPair.getPrivate();
        this.publicKey = keyPair.getPublic();
        this.addressU160 = Address.addressFromPubKey(serializePublicKey());
    }
```

方法二，根据指定私钥生成公钥：


```
public Account(byte[] data, SignatureScheme scheme) throws Exception {
        Security.addProvider(new BouncyCastleProvider());
        signatureScheme = scheme;
        if (scheme == SignatureScheme.SM3WITHSM2) {
            this.keyType = KeyType.SM2;
            this.curveParams = new Object[]{Curve.SM2P256V1.toString()};
        } else if (scheme == SignatureScheme.SHA256WITHECDSA) {
            this.keyType = KeyType.ECDSA;
            this.curveParams = new Object[]{Curve.P256.toString()};
        }
        switch (scheme) {
            case SHA256WITHECDSA:
            case SM3WITHSM2:
                BigInteger d = new BigInteger(1, data);
                ECNamedCurveParameterSpec spec = ECNamedCurveTable.getParameterSpec((String) this.curveParams[0]);
                ECParameterSpec paramSpec = new ECNamedCurveSpec(spec.getName(), spec.getCurve(), spec.getG(), spec.getN());
                ECPrivateKeySpec priSpec = new ECPrivateKeySpec(d, paramSpec);
                KeyFactory kf = KeyFactory.getInstance("EC", "BC");
                this.privateKey = kf.generatePrivate(priSpec);
                org.bouncycastle.math.ec.ECPoint Q = spec.getG().multiply(d).normalize();
                ECPublicKeySpec pubSpec = new ECPublicKeySpec(
                        new ECPoint(Q.getAffineXCoord().toBigInteger(), Q.getAffineYCoord().toBigInteger()),
                        paramSpec);
                this.publicKey = kf.generatePublic(pubSpec);
                this.addressU160 = Address.addressFromPubKey(serializePublicKey());
                break;
            default:
                throw new Exception(ErrorCode.UnsupportedKeyType);
        }
    }
```


## 1.2 账户地址生成

```
对于普通账户	： address = 0x01 + dhash160(pubkey)[1:]
对于多重签名账户：涉及三个量 n, m, pubkeys.
n为总公钥数，pubkeys为公钥的列表，m为需要的签名数量, 沿用Neo对多重签名的限制： n<=24
依次序列化n，m，和pubkeys 得到byte数组 multi_pubkeys
address = 0x02 + dhash160(multi-pubkeys)[1:]


对于合约账户				： address = CodeType + dhash160(code)[1:]
neovm 类合约				： address = 0x80 + dhash160(code)[1:]
wasm 类合约				： address = 0x90 + dhash160(code)[1:]
后续其他vm类型合约也可以向后面拓展.
```

交易在验签时根据地址前缀识别对应的签名算法，进行验签。 交易在执行时根据地址前缀识别对应的虚拟机类型，启动对应的vm运行合约。
示例：

```
//根据公钥计算的address
public static Address addressFromPubKey(byte[] publicKey) {
        ScriptBuilder sb = new ScriptBuilder();
        sb.emitPushByteArray(publicKey);
        sb.add(ScriptOp.OP_CHECKSIG);
        return Address.toScriptHash(sb.toArray());
    }
    //根据多重公钥计算address
   public static Address addressFromMultiPubKeys(int m, byte[]... publicKeys) throws Exception {
           return Address.toScriptHash(Program.ProgramFromMultiPubKey(m,publicKeys));
       }
       public static byte[] ProgramFromMultiPubKey(int m, byte[]... publicKeys) throws Exception {
               int n = publicKeys.length;

               if (m <= 0 || m > n || n > Common.MULTI_SIG_MAX_PUBKEY_SIZE) {
                   throw new SDKException(ErrorCode.ParamError);
               }
               try (ScriptBuilder sb = new ScriptBuilder()) {
                   sb.emitPushInteger(BigInteger.valueOf(m));
                   publicKeys = sortPublicKeys(publicKeys);
                   for (byte[] publicKey : publicKeys) {
                       sb.emitPushByteArray(publicKey);
                   }
                   sb.emitPushInteger(BigInteger.valueOf(publicKeys.length));
                   sb.add(ScriptOp.OP_CHECKMULTISIG);
                   return sb.toArray();
               }
           }
    //根据合约hex获得智能合约的address
    public static Address AddressFromVmCode(String codeHexStr) {
            Address code = Address.toScriptHash(Helper.hexToBytes(codeHexStr));
            return code;
        }
```
## 1.3 公私钥序列化
私钥序列化：私钥转byte[]
公钥序列化：keyType(1 byte) + Curve(1 byte) +  PublicKey Encoded

```
    public byte[] serializePrivateKey() throws Exception {
        switch (this.keyType) {
            case ECDSA:
                BCECPrivateKey pri = (BCECPrivateKey) this.privateKey;
                byte[] d = new byte[32];
                if (pri.getD().toByteArray().length == 33) {
                    System.arraycopy(pri.getD().toByteArray(), 1, d, 0, 32);
                } else {
                    return pri.getD().toByteArray();
                }
                return d;
            default:
                // should not reach here
                throw new Exception(ErrorCode.UnknownKeyType);
        }
    }

    public enum KeyType {
       ECDSA(0x12),
       SM2(0x13),
       EDDSA(0x14);
    }
    public enum Curve {
        P224(1, "P-224"),
        P256(2, "P-256"),
        P384(3, "P-384"),
        P512(4, "P-512"),
        SM2P256V1(20, "sm2p256v1");
    }
    public byte[] serializePublicKey() {
            ByteArrayOutputStream bs = new ByteArrayOutputStream();
            BCECPublicKey pub = (BCECPublicKey) publicKey;
            try {
                switch (this.keyType) {
                    case ECDSA:
                        //bs.write(this.keyType.getLabel());
                        //bs.write(Curve.valueOf(pub.getParameters().getCurve()).getLabel());
                        bs.write(pub.getQ().getEncoded(true));
                        break;
                    case SM2:
                        bs.write(this.keyType.getLabel());
                        bs.write(Curve.valueOf(pub.getParameters().getCurve()).getLabel());
                        bs.write(pub.getQ().getEncoded(true));
                        break;
                    default:
                        // Should not reach here
                        throw new Exception(ErrorCode.UnknownKeyType);
                }
            } catch (Exception e) {
                // Should not reach here
                e.printStackTrace();
                return null;
            }
            return bs.toByteArray();
        }
```

## 1.4 私钥加解密

采用AES的CTR 模式,参数如下：

| Notation | Description |
|:-- |:---|
| A      |The account address    |
|sk      |The private key |
|H       |Hash function|
|IV      |Initial vector used in block cipher                    |
|a[i:j] |the sub-array of byte array a from the index i to j-1   |


| Parameter | Description |
|:-- |:---|
| r      |the block size   |  8 |
|N      |the CPU/Memory cost |  16384 |
|p       |parallelization| 8 |
|dkLen      |byte length of the output               | fixed to 64
|salt |a randomly generated byte sequence  |  |
|P |the user chosen password |  |

h = H(H(A))
salt = h[0:4]
k = scrypt(r, N, p, dkLen, salt, P), where dkLen is fixed to 64
IV = k[0:16]
e = k[32:64], as the AES key
c = AESEncrypt(IV, e, sk), output c

加密：
```
    public String exportGcmEncryptedPrikey(String passphrase,byte[] salt, int n) throws Exception {
        int N = n;
        int r = 8;
        int p = 8;
        int dkLen = 64;
        if (salt.length != 16) {
            throw new SDKException(ErrorCode.ParamError);
        }
        Security.addProvider(new BouncyCastleProvider());
        byte[] derivedkey = SCrypt.generate(passphrase.getBytes(StandardCharsets.UTF_8), salt, N, r, p, dkLen);
        byte[] derivedhalf2 = new byte[32];
        byte[] iv = new byte[12];
        System.arraycopy(derivedkey, 0, iv, 0, 12);
        System.arraycopy(derivedkey, 32, derivedhalf2, 0, 32);
        try {
            SecretKeySpec skeySpec = new SecretKeySpec(derivedhalf2, "AES");
            Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
            cipher.init(Cipher.ENCRYPT_MODE, skeySpec, new GCMParameterSpec(128,iv));
            cipher.updateAAD(getAddressU160().toBase58().getBytes());
            byte[] encryptedkey = cipher.doFinal(serializePrivateKey());
            return new String(Base64.getEncoder().encode(encryptedkey));
        } catch (Exception e) {
            e.printStackTrace();
            throw new SDKException(ErrorCode.EncriptPrivateKeyError);
        }
    }
```

解密：
```
    public static String getGcmDecodedPrivateKey(String encryptedPriKey, String passphrase,String address, byte[] salt, int n, SignatureScheme scheme) throws Exception {
        if (encryptedPriKey == null) {
            throw new SDKException(ErrorCode.EncryptedPriKeyError);
        }
        if (salt.length != 16) {
            throw new SDKException(ErrorCode.ParamError);
        }

        byte[] encryptedkey = Base64.getDecoder().decode(encryptedPriKey);

        int N = n;
        int r = 8;
        int p = 8;
        int dkLen = 64;

        byte[] derivedkey = SCrypt.generate(passphrase.getBytes(StandardCharsets.UTF_8), salt, N, r, p, dkLen);
        byte[] derivedhalf2 = new byte[32];
        byte[] iv = new byte[12];
        System.arraycopy(derivedkey, 0, iv, 0, 12);
        System.arraycopy(derivedkey, 32, derivedhalf2, 0, 32);

        byte[] rawkey = new byte[0];
        try {
            SecretKeySpec skeySpec = new SecretKeySpec(derivedhalf2, "AES");
            Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
            cipher.init(Cipher.DECRYPT_MODE, skeySpec, new GCMParameterSpec(128,iv));
            cipher.updateAAD(address.getBytes());
            rawkey = cipher.doFinal(encryptedkey);
        } catch (Exception e) {
            e.printStackTrace();
            throw new SDKException(ErrorCode.encryptedPriKeyAddressPasswordErr);
        }
        Account account = new Account(rawkey, scheme);
        if (!address.equals(account.getAddressU160().toBase58())) {
            throw new SDKException(ErrorCode.encryptedPriKeyAddressPasswordErr);
        }
        return Helper.toHexString(rawkey);
    }

```

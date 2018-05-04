# 本体 SDK 开发标准



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
        KeyType keyType;
        signatureScheme = scheme;
        switch (scheme) {
            case SHA256WITHECDSA:
                keyType = KeyType.ECDSA;
                Object[] params = new Object[]{Curve.P256.toString()};
                curveParams = params;
                if (!(params[0] instanceof String)) {
                    throw new Exception(ErrorCode.InvalidParams);
                }
                String curveName = (String) params[0];
                paramSpec = new ECGenParameterSpec(curveName);//指定用于生成椭圆曲线 (EC) 域参数的参数集。
                gen = KeyPairGenerator.getInstance("EC", "BC");
                break;
            default:
                //should not reach here
                throw new Exception(ErrorCode.UnsupportedKeyType);
        }
        gen.initialize(paramSpec, new SecureRandom());
        KeyPair keyPair = gen.generateKeyPair();//随机生成公私钥对
        this.privateKey = keyPair.getPrivate();
        this.publicKey = keyPair.getPublic();
        this.keyType = keyType;
        this.addressU160 = Address.addressFromPubKey(serializePublicKey());
    }
```

方法二，根据指定私钥生成公钥：


```
//生成私钥
byte[] privateKey = ECC.generateKey();
//根据私钥生成公钥
public Account(byte[] data, SignatureScheme scheme) throws Exception {
        Security.addProvider(new BouncyCastleProvider());
        signatureScheme = scheme;
        switch (scheme) {
            case SHA256WITHECDSA:
                this.keyType = KeyType.ECDSA;
                Object[] params = new Object[]{Curve.P256.toString()};
                curveParams = params;
                BigInteger d = new BigInteger(1, data);
                ECNamedCurveParameterSpec spec = ECNamedCurveTable.getParameterSpec((String) params[0]);
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
      try {
          byte[] bys = Digest.hash160(publicKey);
          bys[0] = 0x01;
          Address u160 = new Address(bys);
          return u160;
      } catch (Exception e) {
          throw new UnsupportedOperationException(e);
      }
  }
  public static Address addressFromPubKey(ECPoint publicKey) {
        try (ByteArrayOutputStream ms = new ByteArrayOutputStream()) {
            try (BinaryWriter writer = new BinaryWriter(ms)) {
                writer.writeVarBytes(Helper.removePrevZero(publicKey.getXCoord().toBigInteger().toByteArray()));
                writer.writeVarBytes(Helper.removePrevZero(publicKey.getYCoord().toBigInteger().toByteArray()));
                writer.flush();
                byte[] bys = Digest.hash160(ms.toByteArray());
                bys[0] = 0x01;
                Address u160 = new Address(bys);
                return u160;
            }
        } catch (IOException ex) {
            throw new UnsupportedOperationException(ex);
        }
    }
    //根据多重公钥计算address
   public static Address addressFromMultiPubKeys(int m, byte[]... publicKeys) throws Exception {
        if (m <= 0 || m > publicKeys.length || publicKeys.length > 24) {
            throw new IllegalArgumentException();
        }
        try (ByteArrayOutputStream ms = new ByteArrayOutputStream()) {
            try (BinaryWriter writer = new BinaryWriter(ms)) {
                writer.writeByte((byte) publicKeys.length);
                writer.writeByte((byte) m);

                Arrays.sort(publicKeys, (a, b) -> Helper.toHexString(a).compareTo(Helper.toHexString(b)));
                for (int i = 0; i < publicKeys.length; i++) {
                    System.out.println(Helper.toHexString(publicKeys[i]));
                    writer.writeVarBytes(publicKeys[i]);
                }
                writer.flush();
                byte[] bys = Digest.hash160(ms.toByteArray());
                bys[0] = 0x02;
                Address u160 = new Address(bys);
                return u160;
            }
        } catch (IOException ex) {
            throw new UnsupportedOperationException(ex);
        }
    }
    //根据合约hex和虚拟机类型获得智能合约的address
    public static String getCodeAddress(String codeHexStr,byte vmtype){
        Address code = Address.toScriptHash(Helper.hexToBytes(codeHexStr));
        byte[] hash = code.toArray();
        hash[0] = vmtype;
        String codeHash = Helper.toHexString(hash);
        return codeHash;
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
            bs.write(this.keyType.getLabel());
            try {
                switch (this.keyType) {
                    case ECDSA:
                        BCECPublicKey pub = (BCECPublicKey) publicKey;
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
    public String exportCtrEncryptedPrikey(String passphrase, int n) {
        int N = n;  // the CPU/Memory cost
        int r = 8; // block size
        int p = 8; // parallelization
        int dkLen = 64; //byte length of the output
        Address script_hash = Address.addressFromPubKey(serializePublicKey());
        String address = script_hash.toBase58();

        byte[] addresshashTmp = Digest.sha256(Digest.sha256(address.getBytes()));
        byte[] addresshash = Arrays.copyOfRange(addresshashTmp, 0, 4);
        //使用SCRYPT生成密钥
        byte[] derivedkey = SCrypt.generate(passphrase.getBytes(StandardCharsets.UTF_8), addresshash, N, r, p, dkLen);

        byte[] derivedhalf2 = new byte[32];
        byte[] iv = new byte[16];  //初始化向量 (IV)
        System.arraycopy(derivedkey, 0, iv, 0, 16);
        System.arraycopy(derivedkey, 32, derivedhalf2, 0, 32);
        try {
            SecretKeySpec skeySpec = new SecretKeySpec(derivedhalf2, "AES");
            Cipher cipher = Cipher.getInstance("AES/CTR/NoPadding");
            cipher.init(Cipher.ENCRYPT_MODE, skeySpec, new IvParameterSpec(iv));
            byte[] encryptedkey = cipher.doFinal(serializePrivateKey());
            return new String(Base64.getEncoder().encode(encryptedkey));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
```

解密：
```
    public static String getCtrDecodedPrivateKey(String encryptedPriKey, String passphrase, String address, int n, SignatureScheme scheme) throws Exception {
        if (encryptedPriKey == null) {
            throw new NullPointerException();
        }
        byte[] encryptedkey = Base64.getDecoder().decode(encryptedPriKey);

        int N = n;
        int r = 8;
        int p = 8;
        int dkLen = 64;

        byte[] addresshashTmp = Digest.sha256(Digest.sha256(address.getBytes()));
        byte[] addresshash = Arrays.copyOfRange(addresshashTmp, 0, 4);

        byte[] derivedkey = SCrypt.generate(passphrase.getBytes(StandardCharsets.UTF_8), addresshash, N, r, p, dkLen);
        byte[] derivedhalf2 = new byte[32];
        byte[] iv = new byte[16];
        System.arraycopy(derivedkey, 0, iv, 0, 16);
        System.arraycopy(derivedkey, 32, derivedhalf2, 0, 32);

        SecretKeySpec skeySpec = new SecretKeySpec(derivedhalf2, "AES");
        Cipher cipher = Cipher.getInstance("AES/CTR/NoPadding");
        cipher.init(Cipher.DECRYPT_MODE, skeySpec, new IvParameterSpec(iv));
        byte[] rawkey = cipher.doFinal(encryptedkey);
        return Helper.toHexString(rawkey);
    }

```
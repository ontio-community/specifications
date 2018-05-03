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

* java获得公私钥对的示例

方法一：
根据加密算法名和算法需要的参数生成公私钥对。
基本流程：
指定hash散列算法和签名算法；
获得加密算法框架实例并初始化；
产生公私钥对；
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
        this.keyType = keyType;
        this.addressU160 = Address.addressFromPubKey(serializePublicKey());
    }
```

方法二：
根据私钥生成公钥

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

方法三：
根据公钥获得Account
```
private void parsePublicKey(byte[] data) throws Exception {
        if (data == null) {
            throw new Exception(ErrorCode.NullInput);
        }
        if (data.length < 2) {
            throw new Exception(ErrorCode.InvalidData);
        }
        this.privateKey = null;
        this.publicKey = null;
        this.keyType = KeyType.fromLabel(data[0]);
        switch (this.keyType) {
            case ECDSA:
            case SM2:
                Curve c = Curve.fromLabel(data[1]);
                this.curveParams = new Object[]{c.toString()};
                ECNamedCurveParameterSpec spec = ECNamedCurveTable.getParameterSpec(c.toString());
                ECParameterSpec param = new ECNamedCurveSpec(spec.getName(), spec.getCurve(), spec.getG(), spec.getN());
                ECPublicKeySpec pubSpec = new ECPublicKeySpec(
                        ECPointUtil.decodePoint(
                                param.getCurve(),
                                Arrays.copyOfRange(data, 2, data.length)),
                        param);
                KeyFactory kf = KeyFactory.getInstance("EC", "BC");
                this.publicKey = kf.generatePublic(pubSpec);
                break;
            default:
                throw new Exception(ErrorCode.UnknownKeyType);
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
  public static Address addressFromMultiPubKeys(int m, ECPoint... publicKeys) {
        if(m<=0 || m > publicKeys.length || publicKeys.length > 24){
            throw new IllegalArgumentException();
        }
        try (ByteArrayOutputStream ms = new ByteArrayOutputStream()) {
            try (BinaryWriter writer = new BinaryWriter(ms)) {
                writer.writeByte((byte)publicKeys.length);
                writer.writeByte((byte)m);
                ECPoint[] ecPoint = Arrays.stream(publicKeys).sorted((o1, o2) -> {
                    if (o1.getXCoord().toString().compareTo(o2.getXCoord().toString()) <= 0) {
                        return -1;
                    }
                    return 1;
                }).toArray(ECPoint[]::new);
                for(ECPoint publicKey:ecPoint) {
                    writer.writeVarBytes(Helper.removePrevZero(publicKey.getXCoord().toBigInteger().toByteArray()));
                    writer.writeVarBytes(Helper.removePrevZero(publicKey.getYCoord().toBigInteger().toByteArray()));
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

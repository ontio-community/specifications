<h1 align="center">Account Basic Function Standard </h1>
<p align="center" class="version">Version 0.7.0 </p>


## 1.1 Public and Private Key Pair Generation

Current Ontology supported algorithms:

| ID | Algorithm |
|:--|:--|
|0x12|ECDSA|
|0x13|SM2|
|0x14|EdDSA|

ONT signable schemes description( hash algorithm + with + signature algorithm)，supported signature schemes：
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

For serialization method of public and private keys and signatures, please refer to https://github.com/ontio/ontology-crypto/wiki/ECDSA


Account class function:
* Generate public and private key pairs: Specify a hash algorithm and signature algorithm, obtain an instance of the encryption algorithm framework and initialize it, and generate public and private key pairs.
* Calculate U160 address and base58 address based on public key
* Serialization of public and private key
* Private key encryption and decryption

```
public class Account {
    private KeyType keyType; //Signature algorithm
    private Object[] curveParams;//Elliptic curve domain parameters
    private PrivateKey privateKey;//Private key
    private PublicKey publicKey;//Public key
    private Address addressU160;//U160 address, transferred from public key
    private SignatureScheme signatureScheme;//Signature scheme
```

* An example of how java obtains public and private key pairs.
Method 1, random generation of public and private keys:

```
public Account(SignatureScheme scheme) throws Exception {
        Security.addProvider(new BouncyCastleProvider());
        KeyPairGenerator gen;
        AlgorithmParameterSpec paramSpec;
        KeyType keyType;
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
        KeyPair keyPair = gen.generateKeyPair();//Generate public and private key pairs randomly
        this.privateKey = keyPair.getPrivate();
        this.publicKey = keyPair.getPublic();
        this.keyType = keyType;
        this.addressU160 = Address.addressFromPubKey(serializePublicKey());
    }
```

Method 2: Generate a public key based on the specified private key:


```
//Generate private key
byte[] privateKey = ECC.generateKey();
//Generate a public key based on the specified private key
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


## 1.2 Account Address Generation

```
For normal accounts: address = 0x01 + dhash160(pubkey)[1:]
For multi-signature accounts：involves three variables n, m, pubkeys.
n is the total number of public keys, pubkeys is a list of public keys, m is the number of signatures required, and the Neo restriction on multiple signatures is used: n<=24
Serialize n, m, and pubkeys in order to get the byte array multi_pubkeys
address = 0x02 + dhash160(multi-pubkeys)[1:]

For the contract accounts	    ： address = CodeType + dhash160(code)[1:]
neovm contract				： address = 0x80 + dhash160(code)[1:]
wasm contract				： address = 0x90 + dhash160(code)[1:]
Subsequent other vm type contracts can also be extended later.
```

When the transaction is verified, the corresponding signature algorithm is identified according to the address prefix.

When executing the transaction, the corresponding virtual machine type is identified according to the address prefix, and the corresponding VM will start to run the contract.
Example:

```
//Address calculated by public key
public static Address addressFromPubKey(byte[] publicKey) {
        ScriptBuilder sb = new ScriptBuilder();
        sb.emitPushByteArray(publicKey);
        sb.add(ScriptOp.OP_CHECKSIG);
        return Address.toScriptHash(sb.toArray());
    }
    //Address calculated by multiple public keys
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
    //Get smart contract's address based on contract hex and virtual machine type
    public static Address AddressFromVmCode(String codeHexStr) {
                Address code = Address.toScriptHash(Helper.hexToBytes(codeHexStr));
                return code;
            }
```
## 1.3 Serialization of Public and Private Keys
Serialization of Private Key：Private key to byte[].
Serialization of Public Key：keyType(1 byte) + Curve(1 byte) +  PublicKey Encoded

```
    public byte[] serializePrivateKey() throws Exception {
            switch (this.keyType) {
                case ECDSA:
                case SM2:
                    BCECPrivateKey pri = (BCECPrivateKey) this.privateKey;
                    String curveName = Curve.valueOf(pri.getParameters().getCurve()).toString();
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

## 1.4 Private key encryption and decryption

Using AES's CTR mode, the parameters are as follows:

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

Encryption：
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

Decryption：
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

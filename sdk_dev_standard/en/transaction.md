<h1 align="center">Serialize and deserialize Standard</h1>
<p align="center" class="version">Version 0.7.0 </p>

## Serialization and deserialization standard

1. writeVarInt(long v)

Serialize to bytes according to the actual length of v

|v size|serialization result|length|
|:--|:--|:--|
| v < 0xFD | (byte)v | 1 bytes |
| v <= 0xFFFF | 0xfd(1 byte) + (LittleEndian)v in 2 bytes | 3 bytes |
| v <= 0xFFFFFFFF | 0xfe(1 byte) + (LittleEndian)v in 4 bytes | 5 bytes |
| v > 0xFFFFFFFF | 0xff(1 byte) + (LittleEndian)v in 8 bytes | 9 bytes |

Java example:
```
if (v < 0xFD) {
            writeByte((byte)v);
        } else if (v <= 0xFFFF) {
            writeByte((byte)0xFD);
            writeShort((short)v);
        } else if (v <= 0xFFFFFFFF) {
        	writeByte((byte)0xFE);
            writeInt((int)v);
        } else {
            writeByte((byte)0xFF);
            writeLong(v);
        }
```
2. readVarInt(long max)

The first byte is the length of max

long fb = Byte.toUnsignedLong(readByte());

|The size of first byte |Read method|
|:--|:--|
|fb == 0xFD| Read the next 2 bytes |
|fb == 0xFE| Read the next 4 bytes |
|fb == 0xFF| Read the next 8 bytes |

Java example:
```
long fb = Byte.toUnsignedLong(readByte());
        long value;
        if (fb == 0xFD) {
            value = Short.toUnsignedLong(readShort());
        } else if (fb == 0xFE) {
            value = Integer.toUnsignedLong(readInt());
        } else if (fb == 0xFF) {
            value = readLong();
        } else {
			value = fb;
        }
```

3. WriteVarBytes(byte[] v)

Serialization result: The length of v transfers to byte array + v

Java example:
```
writeVarInt(v.length);
writer.write(v);
```
4. WriteString(String v)
Serialization result: Serialization of length of byte array transferred by v + v transfers to byte array

Java example:
```
byte[] vBytes = v.getBytes("UTF-8")
writeVarInt(vBytes.length);
writer.write(vBytes);
```
5. readVarBytes()
First read the first byte,
If the first byte is 0xFD, read the next 2 bytes
If the first byte is 0xFE, read the next 4 bytes
If the first byte is 0xFF, read the next 8 bytes
Otherwise, read 1 byte
Java example
```
readBytes((int)readVarInt(0X7fffffc7));
```
> Note: 0X7fffffc7 is the maximum value. If a value is greater than it, an exception should be thrown.
6. ReadBytes(int count)
Read a fixed-length byte array

Java example:
```
byte[] buffer = new byte[count];
reader.readFully(buffer);
```

## Serialization and deserialization of concrete classes

* Serialization and deserialization of Block

Block class field
```
public class Block extends Inventory {

    public int version;
    public UInt256 prevBlockHash;
    public UInt256 transactionsRoot;
    public UInt256 blockRoot;
    public int timestamp;
    public int height;
    public long consensusData;
    public byte[] consensusPayload;
    public Address nextBookkeeper;
    public String[] sigData;
    public byte[][] bookkeepers;
    public Transaction[] transactions;
    public UInt256 hash;
    private Block _header = null;
    ...
  }
```

Serialize and deserialize in the following order (Java example)
```
writer.writeInt(version);
writer.writeSerializable(prevBlockHash);
writer.writeSerializable(transactionsRoot);
writer.writeSerializable(blockRoot);
writer.writeInt(timestamp);
writer.writeInt(height);
writer.writeLong(consensusData);
writer.writeVarBytes(consensusPayload);
writer.writeSerializable(nextBookkeeper);
writer.writeVarInt(bookkeepers.length);
for(int i=0;i<bookkeepers.length;i++) {
    writer.writeVarBytes(bookkeepers[i]);
}
writer.writeVarInt(sigData.length);
for (int i = 0; i < sigData.length; i++) {
    writer.writeVarBytes(Helper.hexToBytes(sigData[i]));
}
writer.writeInt(transactions.length);
for(int i=0;i<transactions.length;i++) {
    writer.writeSerializable(transactions[i]);
}
```

* Serialization and deserialization of transaction
Transaction field:
```
public byte version = 0;
public final TransactionType txType;
public int nonce = new Random().nextInt();
public Attribute[] attributes;
public Fee[] fee = new Fee[0];
public long networkFee;
public Sig[] sigs = new Sig[0];
```
The order of serialization is as follows, and the order of deserialization please refer to serialization
```
writer.writeByte(version);
writer.writeByte(txType.value());
writer.writeInt(nonce);
serializeExclusiveData(writer);//子类自有字段的序列化
writer.writeSerializableArray(attributes);
writer.writeSerializableArray(fee);
writer.writeLong(networkFee);
writer.writeSerializableArray(sigs);
```

* Serialization and deserialization of DeployCode transactions
The DeployCode transaction is a subclass of Transaction. The serialization order of the fields inherited from the transaction does not change.
DeployCode own field:

```
public byte[] code;
public byte vmType;
public boolean needStorage;
public String name;
public String version;
public String author;
public String email;
public String description;
```
The serialization order of DeployCode's own fields is as follows, and the deserialization order is consistent with the serialization order.
```
writer.writeByte(vmType);
writer.writeVarBytes(code);
writer.writeBoolean(needStorage);
writer.writeVarString(name);
writer.writeVarString(version);
writer.writeVarString(author);
writer.writeVarString(email);
writer.writeVarString(description);
```

* Serialization and deserialization of InvokeCode transactions

InvokeCode own field:
```
public long gasLimit;
public byte vmType;
public byte[] code;
```

The InvokeCode transaction is a subclass of Transaction. The serialization order of the fields inherited from the transaction does not change.
The serialization order of InvokeCode's own fields is as follows, and the deserialization order is consistent with the serialization order.
```
writer.writeLong(gasLimit);
writer.writeByte(vmType);
writer.writeVarBytes(code);
```

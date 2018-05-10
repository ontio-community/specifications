# 本体 SDK 开发标准

## 序列化和反序列化标准

1. writeVarInt(long v)

根据v的实际长度序列化成字节

|v大小|序列化结果|长度|
|:--|:--|:--|
| v < 0xFD | (byte)v | 1 bytes |
| v <= 0xFFFF | 0xfd(1 byte) + (LittleEndian)v in 2 bytes | 3 bytes |
| v <= 0xFFFFFFFF | 0xfe(1 byte) + (LittleEndian)v in 4 bytes | 5 bytes |
| v > 0xFFFFFFFF | 0xff(1 byte) + (LittleEndian)v in 8 bytes | 9 bytes |

Java语言示例
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

第一个字节是max的长度

long fb = Byte.toUnsignedLong(readByte());

|第一个字节大小|读取方法|
|:--|:--|
|fb == 0xFD| 读取后面2个字节 |
|fb == 0xFE| 读取后面4个字节 |
|fb == 0xFF| 读取后面8个字节 |

Java 代码示例
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

序列化结果：v的长度转换字节数组 + v

Java语言示例
```
writeVarInt(v.length);
writer.write(v);
```
4. WriteString(String v)

序列化结果：v转换成字节数组的长度序列化 + v转换成字节数组
Java语言示例
```
byte[] vBytes = v.getBytes("UTF-8")
writeVarInt(vBytes.length);
writer.write(vBytes);
```
5. readVarBytes()

首先读取字节数组的长度
  读取第一个字节，
  如果第一个字节是0xFD,就读后面的2个字节
  如果第一个字节是0xFE,就读后面的4个字节
  如果第一个字节是0xFF,就读后面的8个字节
  否则，就读1个字节
其次读取指定长度的字节

Java代码示例
```
int length = (int)readVarInt(0X7fffffc7)
readBytes(length);
```
> Note: 0X7fffffc7是最大值，如果比该值大直接抛异常

6. ReadBytes(int count)

读取固定长度的字节数组
Java代码示例
```
byte[] buffer = new byte[count];
reader.readFully(buffer);
```

## 具体类的序列化和反序列化

* Block的序列化和反序列化

Block类字段如下
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

按照下面的顺序进行序列化和反序列化(以Java代码序列化为例)：
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

* Transaction的序列化和反序列化
Transaction的字段如下
```
public byte version = 0;
public final TransactionType txType;
public int nonce = new Random().nextInt();
public Attribute[] attributes;
public Fee[] fee = new Fee[0];
public long networkFee;
public Sig[] sigs = new Sig[0];
```
序列化顺序是，反序列化请参考序列化
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

* DeployCode交易的序列化和反序列化
DeployCode交易是Transaction的子类，DeployCode继承自Transaction的字段的序列化顺序不变。
DeployCode自有字段如下
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
DeployCode自有字段的序列化顺序如下,反序列化顺序和序列化顺序一致。
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

* InvokeCode交易的序列化和反序列化
InvokeCode自有字段如下
```
public long gasLimit;
public byte vmType;
public byte[] code;
```

InvokeCode交易是Transaction的子类，InvokeCode继承自Transaction的字段的序列化顺序不变。
InvokeCode自有字段的序列化顺序如下,反序列化顺序和序列化顺序一致。
```
writer.writeLong(gasLimit);
writer.writeByte(vmType);
writer.writeVarBytes(code);
```

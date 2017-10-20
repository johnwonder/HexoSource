title: scala_bytestring
date: 2017-04-21 09:58:41
tags: scala
---


## scala  bytestring

上代码：

```java

object TestByteString extends  App {


  implicit val byteOrder: ByteOrder = ByteOrder.BIG_ENDIAN
  val bsb = new ByteStringBuilder();
  bsb.append(ByteString ( int2Bytes(2) ));

  val bs = bsb.result();

  println(bs.iterator.getInt)

  def int2Bytes(value: Int): Array[Byte] = {

   Array(((value >> 24) & 0xFF).toByte, ((value >> 16) & 0xFF).toByte, ((value >> 8) & 0xFF).toByte, (value & 0xFF).toByte)
 }
}

```

我们来看看ByteIterator的getInt方法：

```java
def getInt(implicit byteOrder: ByteOrder): Int = {
if (byteOrder == ByteOrder.BIG_ENDIAN)
  ((next() & 0xff) << 24
    | (next() & 0xff) << 16
    | (next() & 0xff) << 8
    | (next() & 0xff) << 0)
else if (byteOrder == ByteOrder.LITTLE_ENDIAN)
  ((next() & 0xff) << 0
    | (next() & 0xff) << 8
    | (next() & 0xff) << 16
    | (next() & 0xff) << 24)
else throw new IllegalArgumentException("Unknown byte order " + byteOrder)
}
```

相当于遍历byte数组再左移再用或运算

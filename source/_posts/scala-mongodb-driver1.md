title: scala_mongodb-driver1
date: 2017-07-06 21:23:21
tags: scala
---


前两天研究scala mongodb groupby的问题，又有了另外种解决方案：

在看driver core源码的时候 发现传入的表达式在encode的时候有如下判断：

```java
final class BuildersHelper {

  @SuppressWarnings("unchecked")
  static <TItem> void encodeValue(final BsonDocumentWriter writer, final TItem value, final CodecRegistry codecRegistry) {
      if (value == null) {
          writer.writeNull();
      } else if (value instanceof Bson) {
          ((Encoder) codecRegistry.get(BsonDocument.class)).encode(writer,
                                                                   ((Bson) value).toBsonDocument(BsonDocument.class, codecRegistry),
                                                                   EncoderContext.builder().build());
      } else {
          ((Encoder) codecRegistry.get(value.getClass())).encode(writer, value, EncoderContext.builder().build());
      }
  }

  private BuildersHelper() {
  }
}
```

发现会判断value是否为Bson，那么想到Filter也是Bson类型，所以有了如下表达式：

```java
  Filters.and(Filters.all("$code"),Filters.all("$userType"))
```

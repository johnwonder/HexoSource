title: scala的Either类型
date: 2017-06-13 21:48:44
tags: scala
---

这一段要弄清楚：

```python
  def averageLineCountWontCompile(url1: URL, url2: URL): Either[String, Int] =
  for {
    source1 <- getContent(url1).right
    source2 <- getContent(url2).right
    lines1 = source1.getLines().size
    lines2 = source2.getLines().size
  }
   yield (lines1 + lines2) / 2
```

参考博文：
[类型 Either](http://udn.yyuap.com/doc/guides-to-scala-book/chp7-the-either-type.html)

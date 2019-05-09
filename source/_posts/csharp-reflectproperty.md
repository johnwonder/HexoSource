title: csharp-reflectproperty
date: 2015-10-20 09:16:19
tags: c#
---

##   C#反射Encoding.UTF8属性

###

1.0源码:
```
	//先获取UTF8静态属性
	PropertyInfo property = typeof(Encoding).GetProperty("UTF8", BindingFlags.Static | BindingFlags.GetProperty | BindingFlags.Public);
	    //获取属性值
            object objUTF8 = property.GetValue(null,null);
	    //获取GetString函数列表
            IEnumerable<MethodInfo> infos  = typeof(Encoding).GetMethods(BindingFlags.Public | BindingFlags.Instance)
    .Where(m => m.IsVirtual && m.Name == "GetString");

            byte[] bys = new byte[] {};
	    //调用第一个GetString函数
            object obj =  infos.First().Invoke(objUTF8, new object[] { bys });
```

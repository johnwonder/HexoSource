title: csharp-stringintern1
date: 2015-11-06 10:24:50
tags:
---

读了[深入解析String#intern](http://http://tech.meituan.com/in_depth_understanding_string_intern.html)这篇文章，带着对C#字符串驻留的疑问，在C#里做了如下测试：

```
         string s1 = "1";
         string s11 = string.Intern(s1);
         string s2 = new string(new char[] { '1' });

            Console.WriteLine(s1 == s2);//True
            Console.WriteLine(ReferenceEquals(s1, s2));//False
            Console.WriteLine(Object.ReferenceEquals(string.Intern(s1),s2));//False
            Console.WriteLine(Object.ReferenceEquals(s11 ,s2));//False
            Console.WriteLine(Object.ReferenceEquals(s1,  String.Intern(s2)));//True

            String s22 = "2";
            String s3 = s22 + "2";
            String s4 = "22";
            string s5 = string.Intern(s3);

            Console.WriteLine(s3 == s4);//true
            Console.WriteLine(ReferenceEquals(s3, s4));//false
            Console.WriteLine(ReferenceEquals(s5, s4));//true

            //让我们来修改下s5的值 看看会不会对s4 造成影响
            s5 = "33";
            Console.WriteLine(s4);//还是输出22 即使同一个String实例，但是利用任何一个对String实例的引用所进行的修改操作都不会切实地影响到该实例的状态


            //不过同样值得注意的是，使用Intern方法让一个字符串存活于驻留池中也有一个副作用：即使已经不存在任何其它引用指向驻留池中的字符串了，这个字符串仍然不一定会被垃圾回收掉。也就是说即使驻留池中的字符串已经没有用处了，它可能也要等到CLR终结时才被销毁。当您使用Intern方法的时候，也应该考虑到这个特殊的行为
            Console.Read();
```

这两篇文章 [C#中字符串的内存分配与驻留池](http://kb.cnblogs.com/page/102225/), [C# 字符串驻留](http://blog.sina.com.cn/s/blog_7b60d05f0101s25l.html) 也有阐述。

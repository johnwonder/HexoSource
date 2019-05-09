title: csharp_delegate_event
date: 2017-09-08 11:23:24
tags: c#
---

## c#委托和事件的区别

在学习object-c的委托过程中，顺便复习了下c#的委托和事件。

事件就是一个狭义的委托,也就是事件是一个用于事件驱动模型的专用委托.你可以在客户代码中直接调用委托来激发委托指向的函数,

而事件不可以，事件的触发只能由服务代码自己触发。

也就是说在你的代码里委托你不但可以安排谁是它的调用函数,还可以直接调用它,

而事件不能直接调用,只能通过某些操作触发。

除此之此，事件拥有委托的所有功能，包括多播特性。即事件可以有多个事件处理函数,委托同样也可以是个多播委托.

```csharp
class Program1
  {
      static void OtherClassMethod()
      {
          Console.WriteLine("Delegate an other class's method");
      }

      static void Main(string[] args)
      {
          var test = new TestDelegate();
          test.delegateMethod = new TestDelegate.DelegateMethod(test.NonStaticMethod);
          test.delegateMethod += new TestDelegate.DelegateMethod(TestDelegate.StaticMethod);
          test.delegateMethod += Program1.OtherClassMethod;

          test.delegateEvent();//不能直接调用
          test.delegateMethod();//委托可以直接调用
          //test.RunDelegateMethods();
          //http://www.cnblogs.com/hyddd/archive/2009/07/26/1531538.html
          Console.Read();
      }
  }

  class TestDelegate
  {
      public delegate void DelegateMethod();  //声明了一个Delegate Type

      public DelegateMethod delegateMethod;   //声明了一个Delegate对象

      public event DelegateMethod delegateEvent;

      public static void StaticMethod()
      {
          Console.WriteLine("Delegate a static method");
      }

      public void NonStaticMethod()
      {
          Console.WriteLine("Delegate a non-static method");
      }

      public void RunDelegateMethods()
      {
          if (delegateMethod != null)
          {
              Console.WriteLine("---------");
              delegateMethod.Invoke();
              Console.WriteLine("---------");
          }
      }
  }
```

参考资料：
[快速理解C#高级概念(二) 事件与委托的区别](http://www.cnblogs.com/lilin123/archive/2012/12/20/2826514.html)

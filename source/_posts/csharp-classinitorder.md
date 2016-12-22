title: csharp-classinitorder
date: 2015-10-30 15:01:30
tags:
---

读了[Java重写方法与初始化的隐患](http://www.jianshu.com/p/cdc5adb40bb7#)这篇文章，带着对C#初始化的疑问，在C#里做了如下测试：

```
 	public class BaseClass
    {
    	 //4
         private int mSuperX = 2;

         //3
         private static int ms = 2;

         //5
         public BaseClass()
         {
         		//调用了子类setX
                setX(99);
         }

        public virtual void setX(int x) {
            mSuperX = x;
        }
    }

    public class SubClass : BaseClass {

    //2
    private int mSubX = 1;

    //1
    private static int m = 2;

    //6
    public SubClass()
    {
    }

    public override void setX(int x) {
        base.setX(x);
        mSubX = x;
        Console.WriteLine("SubX is assigned " + x);
    }

    public void printX() {
        Console.WriteLine("SubX = " + mSubX);
    }
```

so,C#里面 子类static成员 -> 子类普通成员初始化和初始化块 ->父类static成员 -> 父类static成员 -> 父类普通成员初始化和初始化块 -> 父类构造方法 ->子类构造方法。

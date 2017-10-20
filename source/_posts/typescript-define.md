title: typescript_define
date: 2017-08-07 11:24:26
tags: typescript
---

## typescript声明文件

今天在看lodash的声明文件，看到下面这一段有点疑问：

```js
declare var _: _.LoDashStatic;

declare namespace _ {

}
```

如果我在namespace里导出一个function,如下：

```js
declare namespace _ {
  export function test():void;
}
```

那vscode会提示错误Duplicate identifier,我们回到TypeScript官网文档看下：

与类型相比，你可能已经理解了什么是值。 值是运行时名字，可以在表达式里引用。 比如 let x = 5;创建一个名为x的值。

同样，以下方式能够创建值：

let，const，和var声明
包含值的namespace或module声明
enum声明
class声明
指向值的import声明
function声明

只要不产生冲突就是合法的。 一个普通的规则是值总是会和同名的其它值产生冲突除非它们在不同命名空间里。

也就是说值冲突了，但是如果我把declare var 换成declare class 那就是可以的

```js
declare namespace _ {
  export function test():void;
}

declare class _ {

}
```

typescript官网是这么说的：

```
Note that in this example, we added a value to the static side of C (its constructor function). This is because we added a value, and the container for all values is another value (types are contained by namespaces, and namespaces are contained by other namespaces).
```

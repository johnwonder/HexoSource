title: angular2源码之forwardRef
date: 2017-08-04 13:37:00
tags: angular
---

今天抽空看了下angular2的依赖注入部分的源码，看到forward_ref.ts这个文件，源码位置如下

[forward_ref.ts](https://github.com/angular/angular/blob/master/packages/core/src/di/forward_ref.ts)

## forwardRef

```js
import {Type} from '../type';
import {stringify} from '../util';

//混合类型
//相当于可以是任意方法来实现这个接口
export interface ForwardRefFn { (): any; }

//forwardRefFn加入属性__forward_ref__
export function forwardRef(forwardRefFn: ForwardRefFn): Type<any> {
  (<any>forwardRefFn).__forward_ref__ = forwardRef;
  (<any>forwardRefFn).toString = function() { return stringify(this()); };
  return (<Type<any>><any>forwardRefFn);
  //TypeScript也使用尖括号来表示类型断言，JSX的语法带来了解析的困难。因此，TypeScript在 .tsx文件里禁用了使用尖括号的类型断言。
  //类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 TypeScript会假设你，程序员，已经进行了必须的检查。
}

//如果type是function 而且有__forward_ref__属性 且 __forward_ref__为forwardRef,那么就执行这个type
export function resolveForwardRef(type: any): any {
  if (typeof type === 'function' && type.hasOwnProperty('__forward_ref__') &&
      type.__forward_ref__ === forwardRef) {
    return (<ForwardRefFn>type)();
  } else {
    return type;
  }
}
```

编译为js后是这样的：

```js
export function forwardRef(forwardRefFn) {
  forwardRefFn.__forward_ref__ = forwardRef;
  forwardRefFn.toString = function () { return stringify(this()); };
  return forwardRefFn;
}

export function resolveForwardRef(type) {
    if (isFunction(type) && type.hasOwnProperty('__forward_ref__') &&
        type.__forward_ref__ === forwardRef) {
        return type();
    }
    else {
        return type;
    }
}
```

### Type

```js
//const是对let的一个增强，它能阻止对一个变量再次赋值。
export const Type = Function;

export function isType(v: any): v is Type<any> {
return typeof v === 'function';
}

//类可以创建出类型，所以你能够在允许使用接口的地方使用类。
export interface Type<T> extends Function { new (...args: any[]): T; }

```

注意到在导出interface Type<T>时，我们定义了constructor,constructor存在于类的静态部分

涉及到的知识点：

1.typescript的[interface](https://www.tslang.cn/docs/handbook/interfaces.html)

2.typescript的[export](https://www.tslang.cn/docs/handbook/modules.html)

3.typescript的[const](https://www.tslang.cn/docs/handbook/variable-declarations.html)

4.typescript的[继承](https://www.tslang.cn/docs/handbook/classes.html)

5.typescript的[类型别名](https://www.tslang.cn/docs/handbook/advanced-types.html)

title: angular源码分析之运行流程
date: 2019-03-12 15:03:12
tags: angular
---


梳理下angular的运行流程：

JQLitePrototype.ready

JQLite.on
此处涉及到expandoStore 的问题
on里面主要是addHandler 放到eventFns里


等到onload后 再调用

angularInit
bootstrap
doBootstrap
createInjector
injectionArgs
invoke -> $RootScopeProvider


以表达式{{person.name}} 文本插值来梳理：

collectDirectives -> addTextInterpolateDirective -> $interpolate

$interpolate函数中调用parse函数，parse函数里
addInterceptor(parsedExpression, interceptorFn)
parsedExpression 为解析后的表达式，interceptorFn 为 parseStringifyInterceptor

parsedExpression外面再封装一层 regularInterceptedExpression函数

regularInterceptedExpression函数的watchDelegate 为inputsWatchDelegate。

$interpolate函数返回的 interpolateFn实际是 调用 extend函数过后的函数：

```js
//返回interpolationFn方法
        return extend(function interpolationFn(context) {
            var i = 0;
            var ii = expressions.length;
            var values = new Array(ii);

            try {
              //根据表达式来解析
              for (; i < ii; i++) {
                //调用parseFns得到value
                values[i] = parseFns[i](context);
              }

              return compute(values);
            } catch (err) {
              $exceptionHandler($interpolateMinErr.interr(text, err));
            }

          }, {
          // all of these properties are undocumented for now
          exp: text, //just for compatibility with regular watchers created via $watch
          expressions: expressions,
          $$watchDelegate: function(scope, listener) {
            var lastValue;
            return scope.$watchGroup(parseFns, function interpolateFnWatcher(values, oldValues) {
              var currValue = compute(values);
              if (isFunction(listener)) {
                listener.call(this, currValue, values !== oldValues ? lastValue : currValue, scope);
              }
              lastValue = currValue;
            });
          }
        });
```

所以此时的调用文本指令的时候，此时scope已经为childScope:

```js
scope.$watch(interpolateFn, function interpolateFnWatchAction(value) {
             node[0].nodeValue = value;
           });
```

这里的interpolateFn 就是上面的function interpolationFn(context) 函数，

因为带有watchDelegate,所以直接调用watchDelegate返回:

```js
//如果被解析过后的watch函数有watch的代理那就返回这个代理watch函数
//
//会创建一个watcher
//watcher里放入listener(类似spring里的监听器)
//数据变化时会触发listener
$watch: function(watchExp, listener, objectEquality, prettyPrintExpression) {
        var get = $parse(watchExp);//watchExp为函数时直接返回watchExp

        if (get.$$watchDelegate) {
          return get.$$watchDelegate(this, listener, objectEquality, get, watchExp);
        }
      }
```

从而进入
scope.$watchGroup :

```js
function $watchGroup: function(watchExpressions, listener) {

  if (watchExpressions.length === 1) {
          // Special case size of one
          return this.$watch(watchExpressions[0], function watchGroupAction(value, oldValue, scope) {
            newValues[0] = value;
            oldValues[0] = oldValue;
            listener(newValues, (value === oldValue) ? newValues : oldValues, scope);
          });
        }
}

```

此时watchExpressions就是 parseFns ，那继续进入watch,那此时的watchDelegate就是

inputsWatchDelegate。

```js
function inputsWatchDelegate(scope, listener, objectEquality, parsedExpression, prettyPrintExpression) {
   var inputExpressions = parsedExpression.inputs;
   var lastResult;

   if (inputExpressions.length === 1) {
     var oldInputValueOf = expressionInputDirtyCheck; // init to something unique so that equals check fails
     inputExpressions = inputExpressions[0];
     return scope.$watch(function expressionInputWatch(scope) {
       var newInputValue = inputExpressions(scope);
       if (!expressionInputDirtyCheck(newInputValue, oldInputValueOf)) {
         lastResult = parsedExpression(scope, undefined, undefined, [newInputValue]);
         oldInputValueOf = newInputValue && getValueOf(newInputValue);
       }
       return lastResult;
     }, listener, objectEquality, prettyPrintExpression);
   }
 }
```

实际这个插值的watchers就只有一个，就是这个expressionInputWatch

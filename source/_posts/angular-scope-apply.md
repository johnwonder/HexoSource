title: angular_scope_apply
date: 2017-04-18 13:59:00
tags: angular
---

## angular scope apply方法

在bootstrap函数中我们会调用scope.$apply函数，代码如下：

```js
var injector = createInjector(modules, config.strictDi);
    injector.invoke(['$rootScope', '$rootElement', '$compile', '$injector',
       function bootstrapApply(scope, element, compile, injector) {
        scope.$apply(function() { //rootScope
          element.data('$injector', injector);
          compile(element)(scope);
        });
      }]
    );
return injector;
```

我们来看看scope.$apply函数内部是如何调用的：

$apply函数是在$rootScopeProvider中定义的：

### $apply函数
```js
$apply: function(expr) {
      try {
        beginPhase('$apply');
        try {
          return this.$eval(expr);
        } finally {
          clearPhase();
        }
      } catch (e) {
        $exceptionHandler(e);
      } finally {
        try {
          $rootScope.$digest();
        } catch (e) {
          $exceptionHandler(e);
          throw e;
        }
      }
}
```

### $eval函数
```js
$eval: function(expr, locals) {
       return $parse(expr)(this, locals);
}
```

### $parse函数
```js
function $parse(exp, interceptorFn, expensiveChecks) {
      var parsedExpression, oneTime, cacheKey;

      expensiveChecks = expensiveChecks || runningChecksEnabled;

      switch (typeof exp) {
        case 'string':
          exp = exp.trim();
          cacheKey = exp;

          var cache = (expensiveChecks ? cacheExpensive : cacheDefault);
          parsedExpression = cache[cacheKey];

          if (!parsedExpression) {
            if (exp.charAt(0) === ':' && exp.charAt(1) === ':') {
              oneTime = true;
              exp = exp.substring(2);
            }
            var parseOptions = expensiveChecks ? $parseOptionsExpensive : $parseOptions;
            var lexer = new Lexer(parseOptions);
            var parser = new Parser(lexer, $filter, parseOptions);
            parsedExpression = parser.parse(exp);
            if (parsedExpression.constant) {
              parsedExpression.$$watchDelegate = constantWatchDelegate;
            } else if (oneTime) {
              parsedExpression.$$watchDelegate = parsedExpression.literal ?
                  oneTimeLiteralWatchDelegate : oneTimeWatchDelegate;
            } else if (parsedExpression.inputs) {
              parsedExpression.$$watchDelegate = inputsWatchDelegate;
            }
            if (expensiveChecks) {
              parsedExpression = expensiveChecksInterceptor(parsedExpression);
            }
            cache[cacheKey] = parsedExpression;
          }
          return addInterceptor(parsedExpression, interceptorFn);

        case 'function':
          return addInterceptor(exp, interceptorFn);

        default:
          return addInterceptor(noop, interceptorFn);
      }
}
```

### addInterceptor函数
```js
//interceptorFn传空就直接返回parsedExpression
function addInterceptor(parsedExpression, interceptorFn) {
      if (!interceptorFn) return parsedExpression;
      var watchDelegate = parsedExpression.$$watchDelegate;
      var useInputs = false;

      var regularWatch =
          watchDelegate !== oneTimeLiteralWatchDelegate &&
          watchDelegate !== oneTimeWatchDelegate;

      var fn = regularWatch ? function regularInterceptedExpression(scope, locals, assign, inputs) {
        var value = useInputs && inputs ? inputs[0] : parsedExpression(scope, locals, assign, inputs);
        return interceptorFn(value, scope, locals);
      } : function oneTimeInterceptedExpression(scope, locals, assign, inputs) {
        var value = parsedExpression(scope, locals, assign, inputs);
        var result = interceptorFn(value, scope, locals);
        // we only return the interceptor's result if the
        // initial value is defined (for bind-once)
        return isDefined(value) ? result : value;
      };

      // Propagate $$watchDelegates other then inputsWatchDelegate
      if (parsedExpression.$$watchDelegate &&
          parsedExpression.$$watchDelegate !== inputsWatchDelegate) {
        fn.$$watchDelegate = parsedExpression.$$watchDelegate;
      } else if (!interceptorFn.$stateful) {
        // If there is an interceptor, but no watchDelegate then treat the interceptor like
        // we treat filters - it is assumed to be a pure function unless flagged with $stateful
        fn.$$watchDelegate = inputsWatchDelegate;
        useInputs = !parsedExpression.inputs;
        fn.inputs = parsedExpression.inputs ? parsedExpression.inputs : [parsedExpression];
      }

      return fn;
    }
```

参考资料：
[AngularJs学习笔记--Scope](http://www.cnblogs.com/lcllao/archive/2012/09/23/2698651.html)

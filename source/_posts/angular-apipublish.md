title: angular源码分析之publishExternalAPI
date: 2017-01-02 20:12:23
tags: angular
---


## publishExternalAPI

### 发布扩展API
```js

function publishExternalAPI(angular) {
  //调用extend函数
  //使得外部可以调用诸如angular.bootstrap等函数
  extend(angular, {
    'bootstrap': bootstrap,
    'copy': copy,
    'extend': extend,
    'merge': merge,
    'equals': equals,
    'element': jqLite,
    'forEach': forEach,
    'injector': createInjector,
    'noop': noop,
    'bind': bind,
    'toJson': toJson,
    'fromJson': fromJson,
    'identity': identity,
    'isUndefined': isUndefined,
    'isDefined': isDefined,
    'isString': isString,
    'isFunction': isFunction,
    'isObject': isObject,
    'isNumber': isNumber,
    'isElement': isElement,
    'isArray': isArray,
    'version': version,
    'isDate': isDate,
    'lowercase': lowercase,
    'uppercase': uppercase,
    'callbacks': {$$counter: 0},
    'getTestability': getTestability,
    '$$minErr': minErr,
    '$$csp': csp,
    'reloadWithDebugInfo': reloadWithDebugInfo
  });

 //返回angular.module方法
 //传递(name, requires, configFn)
 angularModule = setupModuleLoader(window);

 //调用angular.module方法
 //angular.module = function(name, requires, configFn)
 //angular.module维护了一个 modules变量
 //ng name参数
 //['ngLocale'] 是 requires参数
 //['$provide',function(){}] 是 configFn参数
 angularModule('ng', ['ngLocale'], ['$provide',
   function ngModule($provide) {
     // $$sanitizeUriProvider needs to be before $compileProvider as it is used by it.
     $provide.provider({
       $$sanitizeUri: $$SanitizeUriProvider
     });
     //用 injector.invoke时候放入providerCache
     $provide.provider('$compile', $CompileProvider).
       directive({
           a: htmlAnchorDirective,
           input: inputDirective,
           textarea: inputDirective,
           form: formDirective,
           script: scriptDirective,
           select: selectDirective,
           style: styleDirective,
           option: optionDirective,
           ngBind: ngBindDirective,
           ngBindHtml: ngBindHtmlDirective,
           ngBindTemplate: ngBindTemplateDirective,
           ngClass: ngClassDirective,
           ngClassEven: ngClassEvenDirective,
           ngClassOdd: ngClassOddDirective,
           ngCloak: ngCloakDirective,
           ngController: ngControllerDirective,
           ngForm: ngFormDirective,
           ngHide: ngHideDirective,
           ngIf: ngIfDirective,
           ngInclude: ngIncludeDirective,
           ngInit: ngInitDirective,
           ngNonBindable: ngNonBindableDirective,
           ngPluralize: ngPluralizeDirective,
           ngRepeat: ngRepeatDirective,
           ngShow: ngShowDirective,
           ngStyle: ngStyleDirective,
           ngSwitch: ngSwitchDirective,
           ngSwitchWhen: ngSwitchWhenDirective,
           ngSwitchDefault: ngSwitchDefaultDirective,
           ngOptions: ngOptionsDirective,
           ngTransclude: ngTranscludeDirective,
           ngModel: ngModelDirective,
           ngList: ngListDirective,
           ngChange: ngChangeDirective,
           pattern: patternDirective,
           ngPattern: patternDirective,
           required: requiredDirective,
           ngRequired: requiredDirective,
           minlength: minlengthDirective,
           ngMinlength: minlengthDirective,
           maxlength: maxlengthDirective,
           ngMaxlength: maxlengthDirective,
           ngValue: ngValueDirective,
           ngModelOptions: ngModelOptionsDirective
       }).
       directive({
         ngInclude: ngIncludeFillContentDirective
       }).
       directive(ngAttributeAliasDirectives).
       directive(ngEventDirectives);
     $provide.provider({
       $anchorScroll: $AnchorScrollProvider,
       $animate: $AnimateProvider,
       $animateCss: $CoreAnimateCssProvider,
       $$animateJs: $$CoreAnimateJsProvider,
       $$animateQueue: $$CoreAnimateQueueProvider,
       $$AnimateRunner: $$AnimateRunnerFactoryProvider,
       $$animateAsyncRun: $$AnimateAsyncRunFactoryProvider,
       $browser: $BrowserProvider,
       $cacheFactory: $CacheFactoryProvider,
       $controller: $ControllerProvider,
       $document: $DocumentProvider,
       $exceptionHandler: $ExceptionHandlerProvider,
       $filter: $FilterProvider,
       $$forceReflow: $$ForceReflowProvider,
       $interpolate: $InterpolateProvider,
       $interval: $IntervalProvider,
       $http: $HttpProvider,
       $httpParamSerializer: $HttpParamSerializerProvider,
       $httpParamSerializerJQLike: $HttpParamSerializerJQLikeProvider,
       $httpBackend: $HttpBackendProvider,
       $xhrFactory: $xhrFactoryProvider,
       $jsonpCallbacks: $jsonpCallbacksProvider,
       $location: $LocationProvider,
       $log: $LogProvider,
       $parse: $ParseProvider,
       $rootScope: $RootScopeProvider,
       $q: $QProvider,
       $$q: $$QProvider,
       $sce: $SceProvider,
       $sceDelegate: $SceDelegateProvider,
       $sniffer: $SnifferProvider,
       $templateCache: $TemplateCacheProvider,
       $templateRequest: $TemplateRequestProvider,
       $$testability: $$TestabilityProvider,
       $timeout: $TimeoutProvider,
       $window: $WindowProvider,
       $$rAF: $$RAFProvider,
       $$jqLite: $$jqLiteProvider,
       $$HashMap: $$HashMapProvider,
       $$cookieReader: $$CookieReaderProvider
     });
   }
 ]);
```

### angular变量

publishExternalAPI中使用到了angular变量，我们看看它在哪定义的
```js
  var
    msie,             // holds major version number for IE, or NaN if UA is not IE.
    jqLite,           // delay binding since jQuery could be loaded after us.
    jQuery,           // delay binding
    slice             = [].slice,
    splice            = [].splice,
    push              = [].push,
    toString          = Object.prototype.toString,
    getPrototypeOf    = Object.getPrototypeOf,
    ngMinErr          = minErr('ng'),

    /** @name angular */
    //默认在window变量下
    angular           = window.angular || (window.angular = {}),
    angularModule,
    uid               = 0;
```

### extend扩展函数

publishExternalAPI中使用到了extend函数，我们看看它在哪定义的
```js

//内部调用baseExtend函数
function extend(dst) {
  //slice函数
  //一个新的字符串。包括字符串 stringObject 从 start 开始（包括 start）到 end 结束（不包括 end）为止的所有字符。
  return baseExtend(dst, slice.call(arguments, 1), false);
}
//扩展dst对象
//deep是否深度拷贝
//默认false
function baseExtend(dst, objs, deep) {
  var h = dst.$$hashKey;

  for (var i = 0, ii = objs.length; i < ii; ++i) {
    var obj = objs[i];
    if (!isObject(obj) && !isFunction(obj)) continue;
    var keys = Object.keys(obj);
    for (var j = 0, jj = keys.length; j < jj; j++) {
      var key = keys[j];
      var src = obj[key];

      if (deep && isObject(src)) {
        if (isDate(src)) {
          dst[key] = new Date(src.valueOf());
        } else if (isRegExp(src)) {
          dst[key] = new RegExp(src);
        } else if (src.nodeName) {
          dst[key] = src.cloneNode(true);
        } else if (isElement(src)) {
          dst[key] = src.clone();
        } else {
          if (!isObject(dst[key])) dst[key] = isArray(src) ? [] : {};
          baseExtend(dst[key], [src], true);
        }
      } else {
        dst[key] = src;
      }
    }
  }

  setHashKey(dst, h);
  return dst;
}
```

publishExternalAPI最终在源码最后调用，到这时才定义了angular.module方法。

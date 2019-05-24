title: angular源码剖析之Provider系列-$http
date: 2019-05-22 22:21:52
tags: angular
---

## 前言

虽然现在angular.js已经不怎么流行，也逐渐被react,vue等主流框架取代，但是angular.js中的一些
设计我觉得非常值得学习，我们不仅仅要跟随潮流，也得了解曾经优秀流行的框架是怎么设计的，为什么要这么
设计，这样我觉得学习新技术才会思路更清晰，才能更快上手。
我们在工作中为了尽快完成领导交给我们的任务不得不去熟悉使用现有框架提供给我们的福利，
却忽略了基础知识或者说底层技术的掌握，框架就是把各种基础函数和技术封装好让软件开发的效率提升，所以现在我们的很多开发人员只停留在使用层面却
不去想这个功能建立在哪些基础知识点上，

## $http简介

先来看官方文档的简介:
The $http service is a core AngularJS service that facilitates communication with
 the remote HTTP servers via the browser's XMLHttpRequest object or via JSONP.

 $http服务是AngularJs的核心服务，是通过浏览器的XMLHttpRequest对象或者JSONP来跟远程http服务
 进行通信的.也就是说$http内部必然是通过封装的ajax来完成他的设计的。
 $http是基于$q服务暴露出来的deferred/promise api,所以要知道$http的高级使用得熟悉这些api.


## $http核心

1.扩展配置
2.把请求拦截器和响应拦截器放入promise链
3.把核心方法serverRequest放入promise链

核心方法里最终返回了promise对象
```js
function $http(requestConfig) {

      //忽略部分代码
      //扩展了requestConfig
      var config = extend({
        method: 'get',
        transformRequest: defaults.transformRequest,
        transformResponse: defaults.transformResponse,
        paramSerializer: defaults.paramSerializer
      }, requestConfig);

      config.headers = mergeHeaders(requestConfig);
      config.method = uppercase(config.method);
      config.paramSerializer = isString(config.paramSerializer) ?
          $injector.get(config.paramSerializer) : config.paramSerializer;

      //请求拦截器
      var requestInterceptors = [];
      //响应拦截器
      var responseInterceptors = [];
      //调用$q的静态方法when
      var promise = $q.when(config);

      // apply interceptors
      //加入拦截器
      forEach(reversedInterceptors, function(interceptor) {
        if (interceptor.request || interceptor.requestError) {
          //unshift在顶部插入
          //unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。
          //随后在chain的时候一次取出两个元素
          requestInterceptors.unshift(interceptor.request, interceptor.requestError);
        }
        if (interceptor.response || interceptor.responseError) {
          //push在尾部插入
          //push() 方法可向数组的末尾添加一个或多个元素，并返回新的长度。
          responseInterceptors.push(interceptor.response, interceptor.responseError);
        }
      });

      promise = chainInterceptors(promise, requestInterceptors);
      //核心请求放入promise链
      promise = promise.then(serverRequest);
      promise = chainInterceptors(promise, responseInterceptors);

      //忽略部分代码
      return promise;
    }
```

## serverRequest方法

serverRequest调用sendReq方法发送请求。
```js
//放在promise里
    function serverRequest(config) {
      //配置的请求头
      var headers = config.headers;
      //转换请求数据
      var reqData = transformData(config.data, headersGetter(headers), undefined, config.transformRequest);

      // strip content-type if data is undefined
      if (isUndefined(reqData)) {
        forEach(headers, function(value, header) {
          if (lowercase(header) === 'content-type') {
            delete headers[header];
          }
        });
      }

      if (isUndefined(config.withCredentials) && !isUndefined(defaults.withCredentials)) {
        config.withCredentials = defaults.withCredentials;
      }

       //关键的步骤 发送请求
      // send request
      //返回新的promise供外部调用
      return sendReq(config, reqData).then(transformResponse, transformResponse);
    }
```


```js
function sendReq(config, reqData) {
      //创建defrred对象
      var deferred = $q.defer(),
          promise = deferred.promise,
          cache,
          cachedResp,
          reqHeaders = config.headers,
          //构造请求url
          url = buildUrl(config.url, config.paramSerializer(config.params));

      //
      $http.pendingRequests.push(config);
      promise.then(removePendingReq, removePendingReq);
      //配置了cache属性且是get 方法时
      if ((config.cache || defaults.cache) && config.cache !== false &&
          (config.method === 'GET' || config.method === 'JSONP')) {
        cache = isObject(config.cache) ? config.cache
              : isObject(defaults.cache) ? defaults.cache
              : defaultCache;
      }

      if (cache) {
        //cachedResp是个promise对象
        cachedResp = cache.get(url);
        //如果存在缓存那么就存缓存中获取响应结果
        if (isDefined(cachedResp)) {
          if (isPromiseLike(cachedResp)) {
            // cached request has already been sent, but there is no response yet
            cachedResp.then(resolvePromiseWithResult, resolvePromiseWithResult);
          } else {
            // serving from cache
            if (isArray(cachedResp)) {
              resolvePromise(cachedResp[1], cachedResp[0], shallowCopy(cachedResp[2]), cachedResp[3]);
            } else {
              resolvePromise(cachedResp, 200, {}, 'OK');
            }
          }
        } else {
          // put the promise for the non-transformed response into cache as a placeholder
          cache.put(url, promise);
        }
      }

      // if we won't have the response in cache, set the xsrf headers and
      // send the request to the backend
      if (isUndefined(cachedResp)) {
        var xsrfValue = urlIsSameOrigin(config.url)
            ? $$cookieReader()[config.xsrfCookieName || defaults.xsrfCookieName]
            : undefined;
        if (xsrfValue) {
          reqHeaders[(config.xsrfHeaderName || defaults.xsrfHeaderName)] = xsrfValue;
        }
        //done函数里调用了promise
        //调用httpBackend
        //事件处理器
        //上传事件处理器
        //done方法在httpBackend中回调
        $httpBackend(config.method, url, reqData, done, reqHeaders, config.timeout,
            config.withCredentials, config.responseType,
            createApplyHandlers(config.eventHandlers),
            createApplyHandlers(config.uploadEventHandlers));
      }
      //返回promise函数
      return promise;

      function createApplyHandlers(eventHandlers) {
        if (eventHandlers) {
          var applyHandlers = {};
          forEach(eventHandlers, function(eventHandler, key) {
            applyHandlers[key] = function(event) {
              if (useApplyAsync) {
                $rootScope.$applyAsync(callEventHandler);
              } else if ($rootScope.$$phase) {
                callEventHandler();
              } else {
                $rootScope.$apply(callEventHandler);
              }

              function callEventHandler() {
                eventHandler(event);
              }
            };
          });
          return applyHandlers;
        }
      }

      //重要的一步
      //注册到$httpBackend()方法
      //缓存响应
      //处理原始$http promise
      //调用$apply
      /**
       * Callback registered to $httpBackend():
       *  - caches the response if desired
       *  - resolves the raw $http promise
       *  - calls $apply
       */
       //status 响应状态 response 响应对象  
       //headersString 响应头
       //statusText 状态文本
      function done(status, response, headersString, statusText) {
      }

    }
```

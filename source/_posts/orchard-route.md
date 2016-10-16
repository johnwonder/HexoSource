title: orchard-route
date: 2015-09-30 09:07:54
tags:
---

##   Orchard路由解析1

###  DefaultOrchardShell类

DefaultOrchardShell中跟路由相关的就是Active函数。看代码：
{% codeblock lang:c# %}
 var allRoutes = new List<RouteDescriptor>();
            //注册的时候 只有 Orchard.Setup.Routes这个类
            allRoutes.AddRange(_routeProviders.SelectMany(provider => provider.GetRoutes()));//SellContainerFactory中注册到容器
{% endcodeblock %}

然后再通过routePublisher 加入RouteCollection集合
{% codeblock lang:c# %}
 //封装成ShellRoute对象
                    //传入shellSettings
                    var shellRoute = new ShellRoute(routeDescriptor.Route, _shellSettings, _workContextAccessor, _runningShellTable) {
                        IsHttpRoute = routeDescriptor is HttpRouteDescriptor,
                        SessionState = sessionStateBehavior
                    };
                    //_routeCollection 为RouteTable.Routes
                    _routeCollection.Add(routeDescriptor.Name, shellRoute);
{% endcodeblock %}
publish的时候会先对RouteArray排序，根据Priority
{% codeblock lang:c# %}
 var routesArray = routes
                .OrderByDescending(r => r.Priority)
                .ToArray();
{% endcodeblock %}
模块的RouteBase对象都封装成ShellRoute对象，以后都在ShellRoute中解析。

接下来我们来看看安装完后 首页是如何定位到Controller的。

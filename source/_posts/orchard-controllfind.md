title: orchard中Controller是如何找到的
date: 2015-09-30 16:05
tags:
---

##   Orchard 控制器解析1

###  CompositionStrategy类

CompositionStrategy中跟控制器相关的就是Compose函数。看代码：
```
 var allRoutes = new List<RouteDescriptor>();
 var controllers = BuildBlueprint(features, IsController, BuildController, excludedTypes);

//通过Feature设置AreaName
 private static ControllerBlueprint BuildController(Type type, Feature feature) {
            var areaName = feature.Descriptor.Extension.Id;

            var controllerName = type.Name;
            if (controllerName.EndsWith("Controller"))
                controllerName = controllerName.Substring(0, controllerName.Length - "Controller".Length);

            return new ControllerBlueprint {
                Type = type,
                Feature = feature,
                AreaName = areaName,//AreaName在这里加上 便于后面 ControllFactory中去寻找
                ControllerName = controllerName,
            };
        }
```

## ShellContainerFactory类:
然后再通过ShellContainerFactory 加入orchard容器中

```
      foreach (var item in blueprint.Controllers) {
                        //这里很关键是通过AreaName和ControllerName注入
                        var serviceKeyName = (item.AreaName + "/" + item.ControllerName).ToLowerInvariant();
                        var serviceKeyType = item.Type;
                        RegisterType(builder, item)
                            .EnableDynamicProxy(dynamicProxyContext)
                            .Keyed<IController>(serviceKeyName)//这里供OrchardControllerFactory解析
                            .Keyed<IController>(serviceKeyType)
                            .WithMetadata("ControllerType", item.Type)
                            .InstancePerDependency()
                            .OnActivating(e => {
                                // necessary to inject custom filters dynamically
                                // see FilterResolvingActionInvoker
                                //需要动态注入filter
                                var controller = e.Instance as Controller;
                                if (controller != null)
                                    controller.ActionInvoker = (IActionInvoker)e.Context.ResolveService(new TypedService(typeof(IActionInvoker)));
                            });
                    }
```

## OrchardControllerFactory

然后控制器工厂OrchardControllerFactory类通过RouteData寻找Controller
路由数据就是在ShellRoute中找到匹配的路由数据

```
      protected override Type GetControllerType(RequestContext requestContext, string controllerName) {
            var routeData = requestContext.RouteData;

            // Determine the area name for the request, and fall back to stock orchard controllers
            var areaName = routeData.GetAreaName();
            //这里通过routeData里的area controllername 然后去寻找Metadata匹配
            // Service name pattern matches the identification strategy
            var serviceKey = (areaName + "/" + controllerName).ToLowerInvariant();

            // Now that the request container is known - try to resolve the controller information
            Meta<Lazy<IController>> info;
            var workContext = requestContext.GetWorkContext();
            if (TryResolve(workContext, serviceKey, out info)) {
                return (Type) info.Metadata["ControllerType"];
            }

            return null;
        }
```

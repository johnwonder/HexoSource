title: Orchard-Layout-Create
date: 2015-09-28 10:55:24
tags: orchard
---

##   Orchard安装界面的布局创建过程

###  LayoutAwareViewEngine类

   这个类是布局创建的入口，什么时候调用这个类呢？看同级目录ThemeAwareness中的ThemedViewResultFilter.cs。

从类命名就可以看出这是一个视图结果筛选器，在这里面把视图引擎集合重新定义了一下。

```
 viewResultBase.ViewEngineCollection = new ViewEngineCollection(new[] { _layoutAwareViewEngine });
```
   当然_layoutAwareViewEngine是通过Autofac这个强大的IOC容器去resolve出来的。Autofac的解析我们以后会分析。
实际就是调用了LayoutAwareViewEngine类。
   那么这个过滤器是什么时候去调用的呢？就靠FilterProvider这个抽象类了。FilterProvider实现了IFilterProvider接口。
IFilterProvider接口是在SetupMode类中注册的

```
            builder.RegisterType<ThemedViewResultFilter>().As<IFilterProvider>().InstancePerLifetimeScope();
            builder.RegisterType<ThemeFilter>().As<IFilterProvider>().InstancePerLifetimeScope();
```
   为什么要在SetupMode中注册呢？因为执行安装的时候没有把Orchard.Framework的ShellFeature给包装进来，看代码：

```
      var descriptor = new ShellDescriptor {
                SerialNumber = -1,
                Features = new[] {
                    new ShellFeature { Name = "Orchard.Setup" },
                    new ShellFeature { Name = "Shapes" },//这里 用于找 模板
                    new ShellFeature { Name = "Orchard.jQuery" },
                },
            };
```

   如果不是安装过程 ，那么直接在CreateContainerFactory.CreateContainer中通过IDependency接口去调用

```
 //实现IDependency接口的依赖都可以使用动态代理 AOP
                    foreach (var item in blueprint.Dependencies.Where(t => typeof(IDependency).IsAssignableFrom(t.Type))) {
                        var registration = RegisterType(builder, item)
                            .EnableDynamicProxy(dynamicProxyContext)
                            .InstancePerLifetimeScope();
```

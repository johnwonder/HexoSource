title: Orchard 源码解析
date: 2015-07-03 13:49:35
tags:
---

## 悦读从Orchard.Web项目开始：

### Global.asax 入口文件:
定义一个IOrchardHost的泛型类Starter.
宿主初始化HostInitialization函数。

函数内部调用OrchardStarter类的CreateHost函数返回IOrchardHost对象。
IOrchardHost对象调用 `Initialize()`  – `BeginRequest` – `EndRequest`




title: SpringCloud Ribbon知识点
date: 2019-05-08 18:02:04
tags: SpringCloud
---

Feign
Spring Cloud Netflix 的微服务都是以 HTTP 接口的形式暴露的，所以可以用 Apache 的 HttpClient 或 Spring 的 RestTemplate 去調用

而 Feign 是一個使用起來更加方便的 HTTP 客戶端，它用起來就好像調用本地方法一樣，完全感覺不到是調用的遠程方法

总结起来就是：发布到注册中心的服务方接口，是 HTTP 的，也可以不用 Ribbon 或者 Feign，直接浏览器一样能够访问

只不过 Ribbon 或者 Feign 调用起来要方便一些，最重要的是：它俩都支持软负载均衡

注意：spring-cloud-starter-feign 里面已经包含了 spring-cloud-starter-ribbon（Feign 中也使用了 Ribbon）

如何理解客户端Ribbon 
zuul也有负载均衡的功能，它是针对外部请求做负载，那客户端ribbon的负载均衡又是怎么一回事？

客户端ribbon的负载均衡，解决的是服务发起方（在Eureka注册的服务）对被调用的服务的负载，比如我们查询商品服务要调用显示库存和商品明细服务，通过商品服务的接口将两个服务组合，可以减少外部应用的请求，比如手机App发起一次请求即可，可以节省网络带宽，也更省电。

ribbon是对服务之间调用做负载，是服务之间的负载均衡，zuul是可以对外部请求做负载均衡

[Spring Cloud 客服端负载均衡 Ribbon](https://www.cnblogs.com/liferecord/p/6893786.html)
[SpringCloud系列之服务消费Ribbon和Feign区别](https://blog.csdn.net/q_0718/article/details/80269864)
[Spring Cloud OpenFeign详解](https://blog.csdn.net/taiyangdao/article/details/81359394)

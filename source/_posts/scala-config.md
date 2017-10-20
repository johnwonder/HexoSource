title: scala_config
date: 2017-06-21 16:20:06
tags: scala
---

## scala typesafe config库读取配置

这两天在看一个akka remote actor future超时的问题

之前的超时是在remoteActor里配置的

```java
  def remoteActor = TypedActor.context.system.actorSelection(actorInfo.remoteActorPath)
```

然后调用的时候是用
```java
Await.result(remoteActor.?(GetCarMonthlyInfo(date,parkCode))(Timeout.intToTimeout(5000000)), timeout)
```

结果发现还是会出现```Future times out in 5000 ms```

问题出在哪里呢？

其实出在TypedActor本身，默认配置读取的jar包里的reference.conf
```java
typed {
      # Default timeout for typed actor methods with non-void return type
      timeout = 5s
    }
```
```java
TypedActor(system).DefaultReturnTimeout
```

这里的配置会和应用的配置文件合并

```java
final val config: Config = {
     //cfg 应用配置文件
     //defaultReference 默认配置
      val config = cfg.withFallback(ConfigFactory.defaultReference(classLoader))
      config.checkValid(ConfigFactory.defaultReference(classLoader), "akka")
      config
    }
```

路径：config-1.2.1-sources.jar!\com\typesafe\config\impl\AbstractConfigValue.java
```java
public AbstractConfigValue withFallback(ConfigMergeable mergeable) {
    if (ignoresFallbacks()) {
        return this;
    } else {
        ConfigValue other = ((MergeableValue) mergeable).toFallbackValue();

        if (other instanceof Unmergeable) {
            return mergedWithTheUnmergeable((Unmergeable) other);
        } else if (other instanceof AbstractConfigObject) {
            return mergedWithObject((AbstractConfigObject) other);
        } else {
            return mergedWithNonObject((AbstractConfigValue) other);
        }
    }
}
```

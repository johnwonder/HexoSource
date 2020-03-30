title: spring5源码解析之加载bean过程前奏
date: 2020-03-30 20:48:01
tags: Spring
---

Java生态圈里的Spring框架众所周知，研究它源码的大神也多如牛毛，小弟不才，也对它的源码很感兴趣，
不光是要一窥框架设计之精妙，也要在平时的开发过程中能借鉴其中的设计思想。今天这篇文章我们就来
聊聊它加载bean的过程中有什么值得学习的地方。

首先我们可以写个简单的demo,来一步步调试和窥探：

```Java
public static void main( String[] args )
    {
      String XMLPath = "spring-config.xml";
      //
      ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-config.xml");
      Landlord landlord = (Landlord) applicationContext.getBean("landlord", Landlord.class);
      landlord.service();
    }
```

spring-config.xml配置文件中我们就定义了一个简单的bean元素，那么我们先自己简单地想下Spring内部要解决哪些
问题它才能获取到这个实例，我们要带着问题去阅读源码才会有所收获。我首先想到的就是如下这些：

1. 它是如何定位xml配置文件并读取的
2. 如何解析xml配置文件中的bean元素和属性
3. 解析完了以后这些配置放到哪里呢
4. getBean的时候又是去哪获取的呢

## 定位xml配置文件

我们在把一个文件名放入ClassPathXmlApplicationContext构造函数的时候，它内部就开始了定位逻辑，

因为它支持多个配置文件路径，所以我们传入一个路径的时候它会new一个String数组，调用其他构造函数，

```Java
    public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
        this(new String[] {configLocation}, true, null);
    }
    //这里我们学到了可变参数的用途
  	public ClassPathXmlApplicationContext(String... configLocations) throws BeansException {
  		this(configLocations, true, null);
  	}
    public ClassPathXmlApplicationContext(
    String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
    throws BeansException {
    //省略部分代码
    //AbstractRefreshableConfigApplicationContext的setConfigLocations方法
    setConfigLocations(configLocations);
  }
```

最后就调用了```setConfigLocations```，我们跟进去发现它竟然是抽象父类的一个方法，也就是说这个函数
我们可以在ClassPathXmlApplicationContext类中自己实现，这就是一个很好的设计之处。
AbstractRefreshableApplicationContext的setConfigLocations方法如下：
```Java
  /**
	 * Set the config locations for this application context.
	 * <p>If not set, the implementation may use a default as appropriate.
	 */
	public void setConfigLocations(@Nullable String... locations) {
  if (locations != null) {

      this.configLocations = new String[locations.length];
      for (int i = 0; i < locations.length; i++) {
        this.configLocations[i] = resolvePath(locations[i]).trim();
      }
    }
    else {
      this.configLocations = null;
    }
  }
```

它的注释说的很明白，如果我们不调用setConfigLocations,那它就会根据情况用一个默认路径了，后果不看设想。。
显然我们要重点关注的是```resolvePath```这个函数,

```Java
  /**
  	 * Resolve the given path, replacing placeholders with corresponding
  	 * environment property values if necessary. Applied to config locations.
  	 * @param path the original file path
  	 * @return the resolved file path
  	 * @see org.springframework.core.env.Environment#resolveRequiredPlaceholders(String)
  	 */
    protected String resolvePath(String path) {
  		return getEnvironment().resolveRequiredPlaceholders(path);
  	}
```

我们又要看它的注释了，很清楚，根据给定的path路径字符串，如果必要的话会用环境变量替换它里面占位符，然后赋给配置路径，
比如我们把```spring-config.xml``` 换成${java.version}spring-config.xml传入，那么它会把${java.version}用我们环境变量替换掉
具体要看```org.springframework.core.env.Environment#resolveRequiredPlaceholders(String)```里面的代码了。

我们下回再解！

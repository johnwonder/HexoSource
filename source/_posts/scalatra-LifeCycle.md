title: scalatra_LifeCycle
date: 2017-03-18 23:09:15
tags: scala
---

## scalatra是如何启动的

```java

  class ScalatraListener extends ServletContextListener {

  import org.scalatra.servlet.ScalatraListener._

  private[this] val logger: Logger = Logger[this.type]

  private[this] var cycle: LifeCycle = _

  private[this] var servletContext: ServletContext = _

  override def contextInitialized(sce: ServletContextEvent): Unit = {
    try {
      //contextInitialized 后赋值给servletContext
      configureServletContext(sce)
      configureCycleClass(Thread.currentThread.getContextClassLoader)
    } catch {
      case e: Throwable =>
        logger.error("Failed to initialize scalatra application at " + sce.getServletContext.getContextPath, e)
        throw e
    }
  }

  def contextDestroyed(sce: ServletContextEvent): Unit = {
    if (cycle != null) {
      logger.info("Destroying life cycle class: %s".format(cycle.getClass.getName))
      cycle.destroy(servletContext)
    }
  }

  protected def configureExecutionContext(sce: ServletContextEvent): Unit = {
  }

  //寻找LifeCycle
  protected def probeForCycleClass(classLoader: ClassLoader): (String, LifeCycle) = {

  }

  protected def configureServletContext(sce: ServletContextEvent): Unit = {
    //定义servletContext 从sce获取
    servletContext = sce.getServletContext
  }

  protected def configureCycleClass(classLoader: ClassLoader): Unit = {

  }
  }

  object ScalatraListener {

  // DO NOT RENAME THIS CLASS NAME AS IT BREAKS THE ENTIRE WORLD
  // TOGETHER WITH THE WORLD IT WILL BREAK ALL EXISTING SCALATRA APPS
  // RENAMING THIS CLASS WILL RESULT IN GETTING SHOT, IF YOU SURVIVE YOU WILL BE SHOT AGAIN
  val DefaultLifeCycle: String = "ScalatraBootstrap"
  val OldDefaultLifeCycle: String = "Scalatra"
  val LifeCycleKey: String = "org.scalatra.LifeCycle"

  }
```

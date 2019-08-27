Logback介绍
============
logback是java的日志开源组件，是log4j创始人写的，性能比log4j要好，目前主要分为3个模块:

* logback-core:核心代码模块
* logback-classic:log4j的一个改良版本，同时实现了slf4j的接口，这样你如果之后要切换其他日志组件也是一件很容易的事
* logback-access:访问模块与Servlet容器集成提供通过Http来访问日志的功能

logback的使用
===============
引入maven依赖
::
  <!--这个依赖直接包含了 logback-core 以及 slf4j-api的依赖-->
  <dependency>
       <groupId>ch.qos.logback</groupId>
       <artifactId>logback-classic</artifactId>
       <version>1.2.3</version>
  </dependency>
然后就可以直接在代码中使用slf4j的接口获取Logger输出日志了。
一般使用log4j的写法如下:
::
  //这是slf4j的接口，由于我们引入了logback-classic依赖，所以底层实现是logback
  private static final Logger logger = LoggerFactory.getLogger(Test.class);
  
  public static void main(String[] args) throws InterruptedException {
      logger.info("hello world");
  }

使用lombok的@Slf4j注解可以很方便的省略这个步骤，直接用log就可以了。
::
  //加入lombok的依赖 
  <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
  </dependency>

配置详解
========
配置获取顺序
++++++++++++++
logback在启动的时候，会按照下面的顺序加载配置文件

* 如果java程序启动时指定了logback.configurationFile属性，就用该属性指定的配置文件。如java -Dlogback.configurationFile=/path/to/mylogback.xml Test ，这样执行Test类的时候就会加载/path/to/mylogback.xml配置
* 在classpath中查找  logback.groovy 文件
* 在classpath中查找  logback-test.xml 文件
* 在classpath中查找  logback.xml 文件
* 如果是 jdk6+,那么会调用ServiceLoader 查找 com.qos.logback.classic.spi.Configurator接口的第一个实现类
* 自动使用ch.qos.logback.classic.BasicConfigurator,在控制台输出日志

上面的顺序表示优先级，使用java -D配置的优先级最高，只要获取到配置后就不会再执行下面的流程。

SLF4j的日志输出级别
+++++++++++++++++++
在slf4j中，从小到大的日志级别依旧是trace、debug、info、warn、error

configuration节点相关属性
++++++++++++++++++++++++++
========== =========== ==================================================================
属性名称   默认值      介绍
========== =========== ==================================================================
debug      false       要不要打印 logback内部日志信息，true则表示要打印。建议开启
scan       true        配置发送改变时，要不要重新加载
scanPeriod 1 seconds   检测配置发生变化的时间间隔。如果没给出时间单位，默认时间单位是毫秒
========== =========== ==================================================================

configuration子节点介绍
+++++++++++++++++++++++
1. contextName节点
------------------
设置日志上下文名称，后面输出格式中可以通过定义 %contextName 来打印日志上下文名称

2.property节点
--------------
用来设置相关变量,通过key-value的方式配置，然后在后面的配置文件中通过 ${key}来访问

3.appender 节点
---------------	
日志输出组件，主要负责日志的输出以及格式化日志。常用的属性有name和class

======== ========= =================================================================================
属性名称 默认值    介绍
======== ========= =================================================================================
name     无默认值  appender组件的名称，后面给logger指定appender使用
class    无默认值  appender的具体实现类。常用的有 ConsoleAppender、FileAppender、RollingFileAppender
======== ========= =================================================================================

**ConsoleAppender**：向控制台输出日志内容的组件，只要定义好encoder节点就可以使用。

**FileAppender**：向文件输出日志内容的组件，用法也很简单，不过由于没有日志滚动策略，一般很少使用

**RollingFileAppender**：向文件输出日志内容的组件，同时可以配置日志文件滚动策略，在日志达到一定条件后生成一个新的日志文件。
appender节点中有一个子节点filter，配置具体的过滤器，比如上面的例子配置了一个内置的过滤器ThresholdFilter，然后设置了level的值为DEBUG。这样用这个appender输出日志的时候都会经过这个过滤器，日志级别低于DEBUG的都不会输出来。

4.logger以及root节点
--------------------
root节点和logger节点其实都是表示Logger组件。个人觉的可以把他们之间的关系可以理解为父子关系，root是最顶层的logger，正常情况getLogger("name/class")没有找到对应logger的情况下，都是使用root节点配置的logger。

如果配置了logger，并且通过getLogger("name/class")获取到这个logger，输出日志的时候，就会使用这个logger配置的appender输出，同时还会使用rootLogger配置的appender。我们可以使用logger节点的additivity="false"属性来屏蔽rootLogger的appender。这样就可以不使用rootLogger的appender输出日志了。

关于logger的获取，一般logger是配置name的。我们再代码中经常通过指定的CLass来获取Logger，比如这样LoggerFactory.getLogger(Test.class);,其实这个最后也是转成对应的包名+类名的字符串com.kongtrio.Test.class。假设有一个logger配置的那么是com.kongtrio，那么通过LoggerFactory.getLogger(Test.class)获取到的logger就是这个logger。

也就是说，name可以配置包名，也可以配置自定义名称。


实现原理
==========
slf4j是什么
++++++++++++++
slf4j只是一套标准，通俗来讲，就是定义了一系列接口，它并不提供任何的具体实现。所以，我们使用这套接口进行开发，可以任意的切换底层的实现框架。

比如，一开始项目用的是log4j的实现，后来发现log4j的性能太差了，想换成logback，由于我们代码中都是面向slf4j接口的，这样我们只要吧log4j的依赖换成logback就可以了。

logback-classic启动原理
++++++++++++++++++++++++
我们在调用LoggerFactory.getLogger(Test.class)时，这些接口或者类都是slf4j的，那么，它是怎么切换到logback的实现的呢？

在ogback-classic底下，有一个slf4j的包.slf4j在初始化时会调用org.slf4j.StaticLoggerBinder进行初始化。因此，每个要实现slf4j的日志组件项目，都要有org.slf4j.StaticLoggerBinder的具体实现。这样slf4j才会在初始化的关联到具体的实现。

logback-logback.xml 配置示例
==============================
::

  <?xml version="1.0" encoding="UTF-8" ?>
  <configuration>
      <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
          <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
              <pattern>
                  %d{yyyy-MM-dd HH:mm:ss} %highlight(%-5level) %cyan([%-50.50class]) : %boldYellow(%msg) %n
              </pattern>
          </encoder>
      </appender>
  
      <!--info日志文件输出-->
      <appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <!--只拦截info日志的配置-->
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>ERROR</level>
              <!--匹配规则，如果匹配上（上面level配置）就否认-->
              <onMatch>DENY</onMatch>
              <!--如果匹配不上就接收-->
              <onMismatch>ACCEPT</onMismatch>
              <!--LevelFilter里的FilterReply中定义三个规则，另外有个NEUTRAL，意思是跳过这个，然后继续后面的-->
          </filter>
          <encoder>
              <pattern>
                  %d{yyyy-MM-dd HH:mm:ss} %-5level [%-50.50class] : %msg%n
              </pattern>
          </encoder>
          <!--滚动策略：每天滚动生成-->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <!--保存路径-->
              <fileNamePattern>/home/qchailang/IdeaProjects/life/security/src/test/log/info.%d.log</fileNamePattern>
          </rollingPolicy>
      </appender>
  
      <!--error日志文件输出-->
      <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>ERROR</level>
          </filter>
          <encoder>
              <pattern>
                  %d{yyyy-MM-dd HH:mm:ss} %-5level [%-50.50class] : %msg%n
              </pattern>
          </encoder>
          <!--滚动策略-->
          <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
              <!--保存路径-->
              <fileNamePattern>/home/qchailang/IdeaProjects/life/security/src/test/log/error.%d.log</fileNamePattern>
          </rollingPolicy>
      </appender>
  
  
      <root level="info">
          <!--控制台输出-->
          <appender-ref ref="consoleLog"/>
          <!--info输出-->
          <appender-ref ref="fileInfoLog"/>
          <!--error输出-->
          <appender-ref ref="fileErrorLog"/>
      </root>
  </configuration>

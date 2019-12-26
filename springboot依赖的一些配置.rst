springboot依赖的一些配置
+++++++++++++++++++++++++
spring-boot-dependencies、
spring-boot-starter-parent、
io.spring.platform

springboot里会引入很多springboot starter依赖，这些依赖的版本号统一管理，springboot有两种方案可以选择。

1.继承parent：

2.import导入的方式

例1：继承parent：
在pom.xml里添加

.. code:: java

  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>2.2.2.RELEASE</version>
  </parent>

例2：import导入的方式

.. code:: java

  <dependencyManagement>
      <dependencies>
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-dependencies</artifactId>
          <version>2.2.2.RELEASE</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
      </dependencies>
  </dependencyManagement>

例3：import导入的方式

.. code:: java

  <dependencyManagement>
      <dependencies>
        <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.2.2.RELEASE</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
      </dependencies>
  </dependencyManagement>

例3：import导入的方式

.. code:: java

  <dependencyManagement>
      <dependencies>
        <dependency>
          <groupId>io.spring.platform</groupId>
          <artifactId>platform-bom</artifactId>
          <version>Cairo-SR8</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
      </dependencies>
  </dependencyManagement>

spring-boot-dependencies、spring-boot-starter-parent、io.spring.platform三者的关系,区别．
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
1.spring-boot-starter-parent继承spring-boot-dependencies

2.io.spring.platform继承spring-boot-starter-parent

spring-boot-dependencies
-------------------------

spring-boot-dependencies主要的作用了，管理控制依赖版本号，管理控制插件版本号以及引入了3个辅助插件。

从继承的源点spring-boot-dependencies开始看
1.pom.xml里的dependencyManagement节点

dependencyManagement节点的作用是统一maven引入依赖JAR包的版本号，可以看出spring-boot-dependencies最重要的一个作用就是对springboot可能用到的依赖JAR包做了版本号的控制管理
2.pom.xml里的pluginManagement节点

pluginManagement节点的作用是统一maven引入插件的版本号，可以看出spring-boot-dependencies另一个作用是对springboot可能用到的插件做了版本号的控制管理
3.pom.xml里的plugins节点
spring-boot-dependencies引入（或覆盖）了三个插件：

maven-help-plugin：用于获取有关项目或系统的帮助信息；
xml-maven-plugin：处理XML相关
build-helper-maven-plugin：用于设置主源码目录、测试源码目录、主资源文件目录、测试资源文件目录等

这三个插件共同完成了一件事，将spring-boot-dependencies（springboot在项目里使用到的依赖）输出到XML，并且打包install到仓库。

spring-boot-starter-parent
----------------------------

spring-boot-starter-parent继承spring-boot-dependencies
1.pom.xml里的properties节点

spring-boot-starter-parent在properties节点里添加了一些预设配置

java.version：jdk的版本号

<java.version>1.8</java.version>

resource.delimiter：设定占位符为@

<resource.delimiter>@</resource.delimiter>

project.build.sourceEncoding、project.reporting.outputEncoding：设置编码为UTF-8

<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

maven.compiler.source、maven.compiler.target：设置编译打包的jdk版本

<maven.compiler.source>${java.version}</maven.compiler.source>
<maven.compiler.target>${java.version}</maven.compiler.target>

2.pom.xml里的dependencyManagement节点

覆盖了spring-boot-dependencies的spring-core依赖引入，去掉了spring-core里的commons-logging依赖

.. code:: java

  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>${spring.version}</version>
    <exclusions>
      <exclusion>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
      </exclusion>
    </exclusions>
  </dependency>

3.pom.xml里的bulid->resources节点

设置了application.properties配置文件的读取目录在/src/main/resources目录下

pom.xml里的pluginManagement节点

覆盖了spring-boot-dependencies的一些插件版本控制管理：

io.spring.platform
--------------------
io.spring.platform继承spring-boot-starter-parent
1.pom.xml里的properties节点

io.spring.platform一个最大的作用便是将经过集成测试的各类依赖版本号进行整合。

在平时开发中，需要某个JAR包依赖往往是习惯性的找最新版本，或是根据经验选择一个版本；

单对某个JAR包来讲，没有任何问题，但当过多的JAR包依赖整合到一起的时候，就可能会出现各自版本不适配的情况产生，产生BUG漏洞的场景将大大增加；

io.spring.platform所做的事情就是将做过集成测试JAR包依赖整合到一起，大大降低了漏洞出现的可能性。
2.pom.xml里的dependencyManagement节点

覆盖所有父节点里的依赖引入并增加部分新的依赖，使用properties节点里的版本号

继承方式和import方式更改依赖版本号的问题
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
例如使用io.spring.platform时，它会管理各类经过集成测试的依赖版本号。

但有的时候，我们想使用指定的版本号，这个时候就需要去覆盖io.spring.platform的版本号。

一、继承的方式：

使用这个配置，可以通过property覆盖内部的依赖。例如，在pom.xml中升级Spring Data release train。

.. code:: java

  <properties>
    spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
  </properties>

二、import的方式

这种方式不能使用property的形式覆盖原始的依赖项。要达到同样的效果，需要在dependencyManagement里面的spring-boot-dependencies之前添加依赖的东西。例如，要升级Spring Data release train，pom.xml应该是这样的：

.. code:: java

  <dependencyManagement>
      <dependencies>
          <!-- Override Spring Data release train provided by Spring Boot -->
          <dependency>
              <groupId>org.springframework.data</groupId>
              <artifactId>spring-data-releasetrain</artifactId>
              <version>Fowler-SR2</version>
              <scope>import</scope>
              <type>pom</type>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-dependencies</artifactId>
              <version>2.1.3.RELEASE</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
  </dependencyManagement>

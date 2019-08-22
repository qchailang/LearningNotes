**********************************
SpringBoot+MariaDB+MyBatis+Druid整合
**********************************
druid官网： https://github.com/alibaba/druid

POM文件中加入的内容
===========
.. code:: java

    <!--SpringBoot需加入的内容-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.1.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <druid-spring-boot-starter.version>1.1.10</druid-spring-boot-starter.version>
        <mybatis-spring-boot-starter.version>1.3.2</mybatis-spring-boot-starter.version>
        <spring-boot-starter.version>2.1.0.RELEASE</spring-boot-starter.version>
        <mariadb-java-client.version>2.3.0</mariadb-java-client.version>
    </properties>

    <dependencies>
        <!--druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid-spring-boot-starter.version}</version>
        </dependency>

        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis-spring-boot-starter.version}</version>
        </dependency>

        <!--mariadb-->
        <dependency>
            <groupId>org.mariadb.jdbc</groupId>
            <artifactId>mariadb-java-client</artifactId>
            <version>${mariadb-java-client.version}</version>
        </dependency>
    </dependencies>

application.yml加入的内容
====================
.. code:: java

 spring:
   datasource:
     druid:
       driver-class-name: org.mariadb.jdbc.Driver
       url: jdbc:mariadb://localhost/kulvv
       username: qchailang
       password: tianhua
       validationQuery: SELECT 1
       web-stat-filter:
         session-stat-enable: false
       max-active: 10
       initial-size: 1
 # mmapper xml文件路径与类别名设置。
 mybatis:
   mapper-locations: classpath:mapper/*Mapper.xml # Mapper xml文件路径。一般在resources目录下。
   type-aliases-package: com.kulvv.life.entity # 这样设置后，在写xml文件时类的包名可省略。

@Mapper和@MapperScan注解的用法
========================
在每个Mapper文件加上@Mapper注解，指定这是一个Mapper文件，但Mapper文件过多时，在每个Mapper文件上都加@Mapper注解也很麻烦，这时可用@MapperScan注解。每个Mapper文件上不用再加@Mapper注解，只用将@MapperScan注解加到启动类上，然后指定Mapper文件的路径就行了。

Mapper xml文件详解
==================
.. code:: java

  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.kulvv.life.security.mapper.UserInfoMapper">
      <select id="getUserByUsername" parameterType="String" resultType="UserInfoDO">
          SELECT id, username, password, status, descn
          FROM user
          where username = #{username}
      </select>
  </mapper>

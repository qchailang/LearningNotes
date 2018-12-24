**********************************
SpringBoot+MariaDB+MyBatis+Druid整合
**********************************
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

druid官网： https://github.com/alibaba/druid

@Mapper和@MapperScan注解的用法
========================
 在每个Mapper文件加上@Mapper注解，指定这是一个Mapper文件，但Mapper文件过多时，在每个Mapper文件上都加@Mapper注解也很麻烦，这时可用@MapperScan注解。每个Mapper文件上不用再加@Mapper注解，只用将@MapperScan注解加到启动入口类上，然后指定Mapper文件的路径就行了。


Spring Cloud Eureka注册中心(Greenwich版)
===========
**eureka现已闭源，停止更新，处于维护阶段。不过目前已经很稳定，依旧可以使用。也可以使用 Consul、zookeeper等替换**

新建maven工程作为父工程，管理依赖的版本.

pom文件主要内容
    .. code:: java

        <properties>
	        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	        <spring.boot.version>2.1.4.RELEASE</spring.boot.version>
	        <spring.cloud.version>Greenwich.SR1</spring.cloud.version>
	        <maven.compiler.source>1.8</maven.compiler.source>
	        <maven.compiler.target>1.8</maven.compiler.target>
        </properties>

        <dependencyManagement>
	        <dependencies>
	            <dependency>
	                <groupId>org.springframework.boot</groupId>
	                <artifactId>spring-boot-dependencies</artifactId>
	                <version>${spring.boot.version}</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	            <dependency>
	                <groupId>org.springframework.cloud</groupId>
	                <artifactId>spring-cloud-dependencies</artifactId>
	                <version>${spring.cloud.version}</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	        </dependencies>
        </dependencyManagement>
Eureka Server
----------
#. 新建maven module工程作为Eureka注册中心

   pom文件主要内容:
     .. code:: java

              <dependencies>
                 <dependency>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
                 </dependency>
              </dependencies>

   application文件内容.
	  .. code:: java

	    server:
	      port: 8080

	    //springcloud应用都要指定application.name
	    spring:
	      application:
	       name: eureka-server

	    eureka:
	      instance:
	        hostname: localhost
	      client:
	      //registerWithEureka和fetchRegistry=false 表明自己是一个eureka server.
	        registerWithEureka: false
	        fetchRegistry: false
	        serviceUrl:
	          defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
#. 在启动类上加入@EnableEurekaServer注解来启用EurekaServer.
     
   .. code:: java

     @SpringBootApplication
     @EnableEurekaServer //启用EurekaServer
     public class Application {
	     public static void main(String[] args) {
	 	    SpringApplication.run(Application.class, args);
	     }
     }


Eureka Client
----------
#. 新建maven module工程作为Eureka Client
   
   pom文件主要内容：
    .. code:: java

     <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
     </dependencies>

   application文件内容:
    .. code:: java

     server:
       port: 8082

     //服务与服务之间相互调用一般都是根据这个name 。
     spring:
       application:
         name: service-hi

     eureka:
       client:
         serviceUrl:
      //指定服务注册中心的地址
         defaultZone: http://localhost:8080/eureka/
#. 在启动类上加入@EnableEurekaClient注解来启用EurekaClient.
   
   .. code:: java

    @SpringBootApplication
    @EnableEurekaClient //启用EurekaClient服务
    public class Application {
	    public static void main(String[] args) {
	        SpringApplication.run(Application.class, args);
	    }   
    }

**@EnableDiscoveryClient与@EnableEurekaClient的区别**

这两个注解非常的类似，都是用于启用服务发现的。
也就是说discovery service有许多种实现（eureka、consul、zookeeper等等），@EnableDiscoveryClient在spring-cloud-commons包中, 而@EnableEurekaClient是在spring-cloud-netflix包中的。
如果选用的注册中心是eureka，那么就推荐@EnableEurekaClient，如果是其他的注册中心，那么使用@EnableDiscoveryClient。

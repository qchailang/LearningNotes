Spring Cloud Config分布式配置中心
============
配制Config Server
----------

在远程git仓库https://gitee.com/qchailang/springcloud.git中存在一个文件夹config，在该文件夹中编写配置文件

#. **pom需引入的依赖**

   .. code:: java

	 <dependency>
	    <groupId>org.springframework.cloud</groupId>
	    <artifactId>spring-cloud-config-server</artifactId>
	 </dependency>

#. **application.yml文件内容:**

   .. code:: java

	  server:
	  port: 8087
	  spring:
	    application:
	      name: config-server
	    cloud:
	      config:
	        server:
	          git:
	            // git仓库的地址
	            uri: https://gitee.com/qchailang/springcloud.git
	            // 配置仓库路径，这里是文件夹搜索路径
	            searchPaths: config
	            // git仓库的用户名和密码，如果是public仓库，可以不写
	            username: qchailang
	            password: tian123hua
	            // 指定分支，不指定默认是master
	            default-label: master
  
#. **启动类加上@EnableConfigServer主解，启用spring cloud config 服务.**
   
   .. code:: java

    @SpringBootApplication
    @EnableConfigServer //用spring cloud config 服务
    public class ConfigServerApplication {
        public static void main(String[] args) {
            SpringApplication.run(ConfigServerApplication.class, args);
        }
    }

#. **http请求地址和资源文件映射如下:**
   
   * /{application}/{profile}[/{label}]
   * /{application}-{profile}.yml
   * /{label}/{application}-{profile}.yml
   * /{application}-{profile}.properties
   * /{label}/{application}-{profile}.properties
     
   浏览器访问示例:

   * http://localhost:8087/springcloud-config-client-dev.properties
   * http://localhost:8087/springcloud-config-client/dev

配制Config 客户端
----------

#. **pom文件中需引入的依赖**
   
   .. code:: java

     <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
     </dependency>

#. **由于很多配置是需要在服务启动前加载的，比如数据库连接。所以我们要使用bootstrap.yml这个配置，bootstrap配置的优先级高于application，会在服务启动前加载．**
   
   *bootstrap.yml中的内容*
   
   .. code:: java

    // bootstrap的优先级高于application，很多配置都是要在服务启动前加载,所以这里要使用bootstrap.
	spring:
	  cloud:
	    config:
	      # 对应你的配置文件名称,如果没有设置就使用spring.application.name的值.
	      name: springcloud-config-client
	      // 启动什么环境下的配置,对应配置文件的test、dev、prod
	      profile: dev  
	      // 配置服务的URL
	      uri: http://localhost:8087/
	      # 对应git的branch(可选)
	      label: master

   *application.yml中的内容*
   
   .. code:: java

     server:
       port: 8088

     spring:
       application:
         name: springcloud-config-client

#. **测试类，通过属性注入的方式来加载配置：**
   
   .. code:: java

     @RestController
     public class HelloController {
         // 属性注入
         @Value("${name}")
         String name;

         @RequestMapping(value = "/hi")
         public String hi(){
             return "hi "+name;
         }
     }

集成eureka实现高可用集群
-----------

#. **改造Config Server项目，集成eureka，**
   
   *pom文件增加依赖:*
     .. code:: java

       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>

   *application.yml文件增加Eureka相关的配置:*
     .. code:: java

       eureka:
         client:
           serviceUrl:
           #      指定服务注册中心的地址
             defaultZone: http://localhost:8080/eureka/

        #   心跳检测检测与续约时间
        #   【测试】时将值设置设置小些，保证服务关闭后注册中心能及时踢出服务
        instance:
          #     每间隔1s，向服务端发送一次心跳，证明自己依然”存活“
          lease-renewal-interval-in-seconds: 1
          #    告诉服务端，如果我2s之内没有给你发心跳，就代表我“死”了，将我踢出掉。
          lease-expiration-duration-in-seconds: 2

   **启动类中添加注解@EnableEurekaClient**
   
#. **改造Config Client项目，集成eureka.**
   
   *pom文件中增加的依赖*
     .. code:: java

       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>

   *修改bootstrap.yml属性配置，主要添加spring.cloud.config.discovery和eureka相关配置:*
     .. code:: java

       spring:
         cloud:
           config:
             # 对应你的配置文件名称,如果没有设置就使用spring.application.name的值.
	         name: springcloud-config-client
             #启动什么环境下的配置
             profile: dev
             # 配置服务的URL
             # 配置服务的URL【如果使用eureka，则不再写URL，使用下面的discovery的形式】
             # uri: http://localhost:8087/
             discovery:
               enabled: true
               service-id: config-server # 指定配置中心的服务名称
       eureka:
         instance:
           lease-renewal-interval-in-seconds: 1
           lease-expiration-duration-in-seconds: 2
         client:
           serviceUrl:
             # 指定服务注册中心的地址
             defaultZone: http://localhost:8080/eureka/
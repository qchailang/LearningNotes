Spring Cloud 用Feign实现负载均衡
===========
#. 新建maven module工程,

   pom 文件主要内容:
    .. code:: java

      <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        // 引入eureka-client
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        // 引入 feign
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>

#. 声明一个interface接口，开启Feign负载均衡
   
   HelloService.java文件内容
    .. code:: java

     @FeignClient("service-hi")
     public interface HelloService {
         @GetMapping("/")
         String hi(@RequestParam(value = "name") String name);
     }

#. 在启动类上加入@EnableFeignClients注解来启用Feign服务.
   
   .. code:: java

    @SpringBootApplication
    @EnableEurekaClient
    @EnableFeignClients
    public class FeignApplication {
        public static void main(String[] args) {
            SpringApplication.run(FeignApplication.class, args);
        }
    }

#. application.yml文件内容同前. 
   
#. 新建控制器类，内容如下：
   
   .. code:: java

    @RestController
    public class HelloController {
        @Autowired
        private HelloService helloService;
        @GetMapping("/")
        public String hi(String name) {
            return helloService.hi(name);
        }
    }


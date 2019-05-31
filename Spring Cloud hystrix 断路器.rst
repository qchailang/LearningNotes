Spring Cloud hystrix 断路器
==========
**hystrix 目前已经停止开发，处于维护阶段。不过已经很稳定，项目中依旧可以使用。官方也提供了一个替代方案：Resilience4j**

#. feign继承了hystrix，但是默认没有开启，所以我们要在配置文件中开启hystrix功能：
   
   application.yml增加的内容:
     .. code:: java

      //启用hystrix
      feign:
        hystrix:
          enabled: true
#. 虽然feign依赖了hystrix，但是依赖不全，我们还是要手动引入hystrix的依赖：
   
   pom文件增加的内容:
     .. code:: java

       // 引入 hystrix
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
       </dependency>
#. 实现Feign实现负载均衡的接口中的方法,此类中的方法在出现故障时调用.
   
   HelloService.java中的内容
   
   .. code:: java

    @FeignClient(value = "service-hi", fallback = HelloServiceImpl.class)
    public interface HelloService {
    @GetMapping("/")
        String hi(@RequestParam(value = "name") String name);
    }


   HelloServiceImpl.java中的内容

   .. code:: java

    @Component
    public class HelloServiceImpl implements HelloService {
        @Override
        public String hi(String name) {
            return "fallback... " + name;
        }
    }


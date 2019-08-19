引入的依赖
::
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  <dependency>
      <groupId>org.springframework.session</groupId>
      <artifactId>spring-session-data-redis</artifactId>
  </dependency>  

application.yml
::
  spring:
    redis:
      timeout: 6000ms
      password: tianhua
      cluster:
        max-redirects: 3  # 获取失败 最大重定向次数
        nodes:
          - 172.18.0.2:6379
          - 172.18.0.3:6379
          - 172.18.0.4:6379
          - 172.18.0.5:6379
          - 172.18.0.6:6379
          - 172.18.0.7:6379
      lettuce:
        pool:
          max-active: 1000 # 连接池最大连接数（使用负值表示没有限制）
          max-idle: 10 # 连接池中的最大空闲连接
          min-idle: 5 # 连接池中的最小空闲连接
          max-wait: -1 # 连接池最大阻塞等待时间（使用负值表示没有限制）

Redis配置文件:RedisConfig.java
::
  @Configuration
  @AutoConfigureAfter(RedisAutoConfiguration.class)
  public class RedisConfig {
  
      @Bean
      public RedisTemplate<String, Serializable> redisJson2Template(LettuceConnectionFactory redisConnectionFactory) {
          RedisTemplate<String, Serializable> template = new RedisTemplate<>();
          template.setConnectionFactory(redisConnectionFactory);
          template.setKeySerializer(new StringRedisSerializer());
          template.setValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class));
          return template;
      }
  }
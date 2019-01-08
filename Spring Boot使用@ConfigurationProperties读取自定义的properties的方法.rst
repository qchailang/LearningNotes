*********************************************************
Spring Boot使用@ConfigurationProperties读取自定义的properties的方法
*********************************************************
 Spring Boot可使用注解的方式将自定义的properties文件映射到实体bean(属性配置类)中
#. 新建一个属性配置类,加上@Component,@@ConfigurationProperties(prefix = "kulvv.website")注解。**prefix** 对应properties文件相应属性值的前缀，在需要使用这些属性的类中直接注入这个属性配置类。
   
.. code:: java

    @Component
    @ConfigurationProperties(prefix = "kulvv.website")
    public class WebSiteProperties {
      private String name;
      private String url;

      public String getName() {
          return name;
      }

      public void setName(String name) {
          this.name = name;
      }

      public String getUrl() {
          return url;
      }

      public void setUrl(String url) {
          this.url = url;
      }
    }

 application.yml文件内容

  .. code:: java

      kulvv:
        website:
          name: 酷威科技
          url: www.kulvv.com

*********************************************************
Spring Boot使用注解读取自定义的properties的方法
*********************************************************
Spring Boot可使用注解的方式将自定义的properties文件映射到实体bean(属性配置类)中

#. 新建一个属性配置类.在需要使用这些属性的类中直接注入这个属性配置类。
 | @Component 指定这是一个组件,可自动将这个bean加入到spring容器中,这样在别的类中直接注入就可以使用．
 | @ConfigurationProperties(prefix = "kulvv.website"),**prefix** 对应properties文件相应属性值的前缀，
 | @PropertySource("bb.properties") 指定从bb.properties文件中读取属性值．这个注解只能读取.properties文件的内容．
   
.. code:: java

    @Component
    @ConfigurationProperties(prefix = "kulvv.website")
    @PropertySource("conf/bb.properties")
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

 bb.yml文件内容

  .. code:: java

     kulvv.dbname: dbtest
     kulvv.path: mariadb.org

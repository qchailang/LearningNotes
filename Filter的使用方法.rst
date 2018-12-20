**********
Filter使用方法
**********
过滤器的缺点: 
===========
#. 只能获取request与response对象,并不能获取要访问的类与方法. 
#. 因为实现的Filter是servlet的,并不是spring的.
#. 要获取访问的类与方法,可以使用spring提供的拦截器intercept

方法一
======
 | 此方法适合自己写的过滤器.
 | 此方法会拦截所有的路径.
#. 写一个类实现Filter接口,在类上面加上@Component注解.
   
.. code:: java

	 @Component
	 public class MyTestFilter implements Filter {
	 	//实现接口中的方法
	 }
方法二
======
 | 使用servlet3.0提供的@WebFilter注解.此方法比方法一更加灵活.
 | 可使用urlPatterns参数配置拦截的路径.
#. 在application启动入口类中添加注解@ServletComponentScan
#. 写一个类实现Filter接口,在类上面加上@WebFilter(urlPatterns = “/*”)注解

.. code:: java

	//application启动入口类
	@SpringBootApplication
	@ServletComponentScan
	public class DemoApplication extends SpringBootServletInitializer {

	    public static void main(String[] args) {
	        SpringApplication.run(DemoApplication.class, args);
	    }

	    @Override
	    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
	        return application.sources(DemoApplication.class);
	    }
	}

	//实现Filter接口的类
	/*
 	* @WebFilter将一个实现了javax.servlet.Filter接口的类定义为过滤器
 	* 属性filterName声明过滤器的名称,可选
 	* 属性urlPatterns指定要过滤的URL模式,也可使用属性value来声明.必选属性
 	*/
	@WebFilter(urlPatterns = "/*")
	public class MyTestFilter implements Filter {
		//实现接口的方法
	}
方法三
======
 | 第三方的过滤器或自己写的过滤器适合使用此方法.
 | 使用此方法可以更加灵活的配置过滤器.如配置 **过滤器拦截的路径**, **过滤器的启动顺序**.
 | 配置类只是在一个普通类上加个@Configuration注解.
#. 使用第三方的过滤器或自己写一个过滤器类,不用加@Compoent或@WebFilter注解.
#. 写一个配置类,在配置类中将过滤器注册到过滤器链中.

.. code-block:: java

	@Configuration
	public class WebFilterConfig {
	    @Bean
	    public FilterRegistrationBean myTestFilter() {
	        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
	        MyTestFilter myTestFilter = new MyTestFilter();
	        filterRegistrationBean.setFilter(myTestFilter);
	        filterRegistrationBean.setOrder(1);
	        List<String> urls = new ArrayList<>();
	        urls.add("/*");
	        filterRegistrationBean.setUrlPatterns(urls);
	        return filterRegistrationBean;
	    }
	}



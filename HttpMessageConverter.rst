Http请求响应报文其实都是字符串，当请求报文到java程序会被封装为一个ServletInputStream流，开发人员再读取报文，响应报文则通过ServletOutputStream流来输出响应报文。
从流中只能读取到原始的字符串报文，同样输出流也是。那么在报文到达SpringMVC / SpringBoot和从SpringMVC / SpringBoot出去，都存在一个字符串到java对象的转化问题。这一过程，在SpringMVC / SpringBoot中，是通过HttpMessageConverter来解决的

如何自定义一个HttpMessageConverter，就用Java原生序列化为例，叫作JavaSerializationConverter，基本仿照ProtobufHttpMessageConverter来写。

首先继承AbstractHttpMessageConverter，泛型类这里有几种方式可以选择：

最广的可以选择Object，不过Object并不都是可以序列化的，但是可以在覆盖的supports方法中进一步控制，因此选择Object是可以的
最符合的是Serializable，既完美满足泛型定义，本身也是个Java序列化/反序列化的充要条件
自定义的基类Bean，有些技术规范要求自己代码中的所有bean都继承自同一个自定义的基类BaseBean，这样可以在Serializable的基础上再进一步控制，满足自己的业务要求
这里选择Serializable作为泛型基类。

其次是选择一个MediaType，使得springmvc能够根据Accept和Content-Type唯一确定是要使用JavaSerializationConverter，所以这个MediaType不能是通用的text/plain、application/json、*/*这种，得特殊一点，这里就用application/x-java-serialization;charset=UTF-8。因为Java序列化是二进制数据，charset不是必须的，但是MediaType的构造方法中需要指定一个charset，这里就用UTF-8。

添加一个converter的方式有三种，代码以及说明如下：

    // 添加converter的第一种方式，代码很简单，也是推荐的方式
    // 这样做springboot会把我们自定义的converter放在顺序上的最高优先级（List的头部）
    // 即有多个converter都满足Accpet/ContentType/MediaType的规则时，优先使用我们这个
    @Bean
    public JavaSerializationConverter javaSerializationConverter() {
        return new JavaSerializationConverter();
    }

    // 添加converter的第二种方式
    // 通常在只有一个自定义WebMvcConfigurerAdapter时，会把这个方法里面添加的converter(s)依次放在最高优先级（List的头部）
    // 虽然第一种方式的代码先执行，但是bean的添加比这种方式晚，所以方式二的优先级 大于 方式一
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // add方法可以指定顺序，有多个自定义的WebMvcConfigurerAdapter时，可以改变相互之间的顺序
        // 但是都在springmvc内置的converter前面
        converters.add(new JavaSerializationConverter());
    }

    // 添加converter的第三种方式
    // 同一个WebMvcConfigurerAdapter中的configureMessageConverters方法先于extendMessageConverters方法执行
    // 可以理解为是三种方式中最后执行的一种，不过这里可以通过add指定顺序来调整优先级，也可以使用remove/clear来删除converter，功能强大
    // 使用converters.add(xxx)会放在最低优先级（List的尾部）
    // 使用converters.add(0,xxx)会放在最高优先级（List的头部）
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new JavaSerializationConverter());
    }

替换默认的HttpMessageConverter，就是将其改成使用FastJsonHttpMessageConverter来处理Java对象与HttpInputMessage/HttpOutputMessage间的转化。
 @Configuration
public class HttpMessageConverterConfig {

    //引入Fastjson解析json，不使用默认的jackson
    //必须在pom.xml引入fastjson的jar包，并且版必须大于1.2.10
    @Bean
    public HttpMessageConverters fastJsonHttpMessageConverters() {
        //1、定义一个convert转换消息的对象
        FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();

        //2、添加fastjson的配置信息
        FastJsonConfig fastJsonConfig = new FastJsonConfig();

        SerializerFeature[] serializerFeatures = new SerializerFeature[]{
                //    输出key是包含双引号
//                SerializerFeature.QuoteFieldNames,
                //    是否输出为null的字段,若为null 则显示该字段
//                SerializerFeature.WriteMapNullValue,
                //    数值字段如果为null，则输出为0
                SerializerFeature.WriteNullNumberAsZero,
                //     List字段如果为null,输出为[],而非null
                SerializerFeature.WriteNullListAsEmpty,
                //    字符类型字段如果为null,输出为"",而非null
                SerializerFeature.WriteNullStringAsEmpty,
                //    Boolean字段如果为null,输出为false,而非null
                SerializerFeature.WriteNullBooleanAsFalse,
                //    Date的日期转换器
                SerializerFeature.WriteDateUseDateFormat,
                //    循环引用
                SerializerFeature.DisableCircularReferenceDetect,
        };

        fastJsonConfig.setSerializerFeatures(serializerFeatures);
        fastJsonConfig.setCharset(Charset.forName("UTF-8"));

        //3、在convert中添加配置信息
        fastConverter.setFastJsonConfig(fastJsonConfig);

        //4、将convert添加到converters中
        HttpMessageConverter<?> converter = fastConverter;

        return new HttpMessageConverters(converter);
    }
}
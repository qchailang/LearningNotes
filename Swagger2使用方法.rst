Swagger2使用方法
===========
#. Maven 依赖
     
   .. code:: java

         <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
#. 常用注解
   
::

 @Api：用在类上，说明该类的作用
 @ApiOperation：用在方法上，说明方法的作用
 @ApiModelProperty：用在model属性上，描述一个model属性的作用
 @ApiParam：用在方法的参数上，对单个参数的描述
 @ApiImplicitParams：用在方法上包含一组参数说明
 @ApiImplicitParam：用在 @ApiImplicitParams 注解中，指定一个请求参数的各个方面
    paramType：参数放在哪个地方
      · header --> 请求参数的获取：@RequestHeader
      · query -->请求参数的获取：@RequestParam
      · path（用于restful接口）--> 请求参数的获取：@PathVariable
    name：参数名
    dataType：参数类型
    required：参数是否必须传
    value：参数的意思
    defaultValue：参数的默认值
 @ApiResponses：用于表示一组响应
 @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
    code：数字，例如400
    message：信息，例如"请求参数没填好"
    response：抛出异常的类
 @ApiModel：描述一个Model的信息，用对象来接收参数（这种一般用在post创建的时候，使用@RequestBody这样的场景，请求参数无法使用@ApiImplicitParam注解进行描述的时候）
 @ApiProperty：用对象接收参数时，描述对象的一个字段
 @ApiIgnore：使用该注解忽略这个API
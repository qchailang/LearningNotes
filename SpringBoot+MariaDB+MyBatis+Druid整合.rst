**********************************
SpringBoot+MariaDB+MyBatis+Druid整合
**********************************
druid官网： https://github.com/alibaba/druid

POM文件中加入的内容
===========
.. code:: java

    <!--SpringBoot需加入的内容-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.1.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <druid-spring-boot-starter.version>1.1.10</druid-spring-boot-starter.version>
        <mybatis-spring-boot-starter.version>1.3.2</mybatis-spring-boot-starter.version>
        <spring-boot-starter.version>2.1.0.RELEASE</spring-boot-starter.version>
        <mariadb-java-client.version>2.3.0</mariadb-java-client.version>
    </properties>

    <dependencies>
        <!--druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid-spring-boot-starter.version}</version>
        </dependency>

        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis-spring-boot-starter.version}</version>
        </dependency>

        <!--mariadb-->
        <dependency>
            <groupId>org.mariadb.jdbc</groupId>
            <artifactId>mariadb-java-client</artifactId>
            <version>${mariadb-java-client.version}</version>
        </dependency>
    </dependencies>

application.yml加入的内容
====================
.. code:: java

 spring:
   datasource:
     druid:
       driver-class-name: org.mariadb.jdbc.Driver
       url: jdbc:mariadb://localhost/kulvv
       username: qchailang
       password: tianhua
       validationQuery: SELECT 1
       web-stat-filter:
         session-stat-enable: false
       max-active: 10
       initial-size: 1
 # mapper xml映射配置文件路径与类别名设置。
 mybatis:
   mapper-locations: classpath:mapper/*Mapper.xml # Mapper xml映射配置文件路径。一般放resources目录下。
   type-aliases-package: com.kulvv.life.entity # 这样设置后，在写xml映射配置文件时类的包名可省略。

@Mapper和@MapperScan注解的用法
========================
在每个Mapper文件加上@Mapper注解，指定这是一个Mapper文件，但Mapper文件过多时，在每个Mapper文件上都加@Mapper注解也很麻烦，这时可用@MapperScan注解。每个Mapper文件上不用再加@Mapper注解，只用将@MapperScan注解加到一个配置类上，然后指定Mapper文件的路径就行了。

Mybatis Mapper动态代理开发步骤
=======================
* 编写DAO接口，UserMapper.
* 在resources目录下编写Dao对应的映射配制文件，UserMapper.xml
* xml 映射配置文件规范

  + UserMapper.xml映射配制文件中namespace与UserMapper接口的类路径一致。
  + UserMapper.xml映射配制文件中定义的每个sql语句的id要与UserMapper接口方法名一致。
  + UserMapper.xml映射配制文件中定义的每个sql语句的parameterType与UserMapper接口方法的输入参数类型一致。
  + UserMapper.xml映射配制文件中定义的每个sql语句的resultType的类型与UserMapper口方法的返回参数类型一致

Mybatis注解方式开发步骤
==========================
编写DAO接口，UserMapper。在接口的方法中配置相应的注解，不需配置映射文件。

示例: OrderMapper.java 一对一关联查询

 column对应数据库（select 语句返回结果）字段，property对应pojo类属性。

.. code:: java

 package com.kulvv.dao;
 public interface OrderMapper {
     @Select("select * from t_order where id=#{id}")
     @Results({
             @Result(id=true, property = "id", column = "id"),
             @Result(property = "uid", column = "uid"),
             @Result(property = "price", column = "price"),
             @Result(property = "user", column = "uid", one = @One(select = "com.kulvv.dao.UserMapper.getUserById"))
     })
     Order getOrderById(int id);
 
     @Select("select * from t_order where uid=#{uid}")
     @Results({
             @Result(id=true, property = "id", column = "id"),
             @Result(property = "uid", column = "uid"),
             @Result(property = "price", column = "price")
     })
     List<Order> getOrdersByUid(int uid);
 }

示例: UserMapper.java 一对多关联查询

.. code:: java

  package com.kulvv.dao;
  public interface UserMapper {
      @Select("select * from t_user where id=#{id}")
      @Results({
              @Result(id=true, property = "id", column = "id"),
              @Result(property = "name", column = "name"),
              @Result(property = "sex", column = "sex"),
              @Result(property = "age", column = "age"),
              @Result(property = "orderList", column = "id", many = @Many(select = "com.kulvv.dao.OrderMapper.getOrdersByUid"))}
      )
      User getUserById(int id);
  }

Mapper xml映射配置文件详解
============================
.. code:: java

  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.kulvv.life.security.mapper.UserInfoMapper">
      <select id="getUserByUsername" parameterType="String" resultType="UserInfoDO">
          SELECT id, username, password, status, descn
          FROM user
          where username = #{username}
      </select>
      ...
  </mapper>
Mapper文件中包含的元素有：
-------------------------
* cache – 配置给定命名空间的缓存。
* cache-ref – 从其他命名空间引用缓存配置。
* resultMap – 映射复杂的结果对象，确定实体类属性与表中字段对应关系。
* sql – 可以重用的 SQL 块,也可以被其他语句引用。
* insert – 映射插入语句
* update – 映射更新语句
* delete – 映射删除语句
* select – 映射查询语句

示例：select – 映射查询语句
----------------------------
.. code:: java

    <select id="queryUserById" resultType="User" parameterType="Long">
    	SELECT * FROM tb_user WHERE id = #{id}
    </select>

id：是当前命名空间下的Statement的唯一标识，与接口方法名一致（必须属性）；

parameterType：输入的参数类型（可以省略）；

resultType：将结果集映射为的java对象类型（必须属性）；

标签内部：编写SQL语句

示例：insert – 获取自增Id
--------------------------
.. code:: java

  <!-- useGeneratedKeys: 开启自增id回填
  keyColumn: 指定数据库表中列名
  keyProperty: 指定对象的属性名
  如果keyColumn 和 keyProperty 相同，则keyColumn可以省略不写 -->
  <insert id="saveUser" parameterType="cn.zto.mybatis.pojo.User" useGeneratedKeys="true" keyColumn="id" keyProperty="id">
  	INSERT INTO ......
  </insert>

parameterType的传入参数
-----------------------
在<insert>,<update>,<select>,<delete>标签中,可以通过parameterType指定输入参数的类型。

传入类型有三种：

* 简单类型，string、long、integer等
* Pojo类型，User等
* HashMap类型。
 parameterType参数是可以省略的，MyBatis框架可以根据接口中方法的参数来判断输入参数的实际数据类型.

ResultType结果输出
-------------------
resultType属性存在<select>标签中.负责将查询结果进行映射。

使用resultType属性为实体类类型时，只有查询出来的列名和实体类中的属性名一致，才可以映射成功. 如果查询出来的列名和pojo中的属性名全部不一致，就不会创建实体类对象.但是只要查询出来的列名和实体类中的属性有一个一致，就会创建实体类对象。

输出类型有三种：

* 简单类型，string、long、integer等
* Pojo类型，User等
* HashMap类型。

示例:
------
..code:: java

  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.lion.mapper.UserMapper">
    <select id="selectUser" parameterType="int" resultType="User">
      select * from users where id = #{id}
    </select>
    
    <delete id="deleteUser" parameterType="int">
        delete from users where id = #{id}
    </delete>
    
    
    <update id="updateUser" parameterType="User">
        update users set name = #{name},age = #{age} where id = #{id}
    </update>
     
    <insert id="addUser" parameterType="User">
        insert into users(name,age) values(#{name},#{age})
    </insert>
    
    <select id="selectAll" resultType="User" >
        select * from users
    </select>
  </mapper>

resultMap – 映射复杂的结果对象
-------------------------------
使用ResultMap可以解决两大问题：
* MyBatis框架中是根据表中字段名到实体类定位同名属性的.如果出现了实体类属性名与表中字段名不一致的情况,则无法自动进行对应.此时可以使用resultMap来重新建立实体类与字段名之间对应关系.
* 完成高级查询，比如说，一对一、一对多、多对多。

ResultMap 元素内置标签
+++++++++++++++++++++++
#. constructor - 用于在实例化类时，注入结果到构造方法中
  ① idArg - ID 参数标记出作为ID 的结果可以帮助提高整体性能
  ② arg - 将被注入到构造方法的一个普通结果
#. id - 一个ID结果，标记出作为ID的结果可以帮助提高整体性能
#. result - 注入到字段或javaBean属性的普通结果
#. association - 一个复杂类型的关联；许多结果将包装成这个类型
  嵌套结果映射 - 集合可以指定为一个resultMap元素，或者引用一个
#. collection  -  一个复杂类型的集合
  嵌套结果映射 - 集合可以指定一个resultMap元素，或者引用一个
#. discriminator - 使用结果值来决定使用那个resultMap
   case - 基于某些值的结果映射
   嵌套结果映射 - 一个case 也是一个映射它本身的结果，因此可以包含很多相同的元素，或者它可以参照一个外部的resultMap

示例：字段名与实体类属性之间建立对应关系
+++++++++++++++++++++++++++++++++++++
.. code:: java

   <resultMap id="userResultMap" type="User">
     <id property="id" column="user_id" />
     <result property="username" column="user_name"/>
     <result property="password" column="hashed_password"/>
   </resultMap>

   <select id="selectUsers" resultMap="userResultMap">
     select user_id, user_name, hashed_password
     from some_table
     where id = #{id}
   </select>  

示例： 多对一 或 一对一 映射
+++++++++++++++++++++++++++
一对一的关系可以使用"association"，"javaType"进行配置。
* association:用于将关联信息映射到单个pojo上。
* property:要将关联信息映射到要查询的用户信息中的那个属性。
* javaType:关联信息映射到orders的属性的类型，是User类型。

.. code:: java

  <resultMap id="orderAll" type="com.kulvv.pojo.Order">
      <id column="id" property="id" />
      <result column="uid" property="uid" />
      <result column="price" property="price" />
      <association property="user" javaType="com.kulvv.pojo.User">
          <id column="uid" property="id" />
          <result column="name" property="name" />
          <result column="sex" property="sex" />
          <result column="age" property="age" />
      </association>
  </resultMap>

  <select id="getOrderAll" resultMap="orderAll">
      select o.uid, o.id, u.`name`, o.price, u.sex, u.age
      from t_order o
      LEFT JOIN t_user u
      on o.uid = u.id
  </select>

示例： 一对多 映射(mybatis不提供多对多映射，如果是多对多关系需要转换为一对多和多对一关系)
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
一对多关系可以使用"collection"，"fType"进行配置。

property是集合("多")的名称，ofType是集合的元素类型。

.. code:: java

  <resultMap id="userAll" type="com.kulvv.pojo.User">
      <id column="id" property="id" />
      <result column="name" property="name" />
      <result column="sex" property="sex" />
      <result column="age" property="age" />
      <collection property="orderList" ofType="com.kulvv.pojo.Order">
          <id column="id" property="uid" />
          <result column="oid" property="id" />
          <result column="price" property="price" />
      </collection>
  </resultMap>

  <select id="getUserAll" resultMap="userAll">
      select u.id, u.name, u.sex, u.age, o.id oid, o.price
      from t_user u
      LEFT join t_order o
      on u.id = o.uid
  </select>

ResultMap 元素标签属性
+++++++++++++++++++++++
* type: 结果集映射的java对象类的完全限定名，或者一个类型别名
* id: resultMap的唯一标识，对应select标签中的属性resultMap。
* autoMapping: 如果设置这个属性，Mybatis将会为这个ResultMap开启或者关闭自动映射，缺省为 true 。

'#{}'与'${}'的区别
-----------------------
* #{} 实现的是sql语句的预处理参数，之后执行sql中用?号代替，使用时不需要关注数据类型，Mybatis自动实现数据类型的转换。并且可以防止SQL注入。使用一个参数时，可以使用任意参数名称进行接收， #{}只是做占位符，与参数名无关。
* ${} 实现sql语句的直接拼接，不做数据类型转换，需要自行判断数据类型。如果是字符串，需要手动添加引号。不能防止SQL注入。
在大多数情况下,我们都是采用#{}读取参数内容.但是在一些特殊的情况下,我们还是需要使用${}读取参数的.

比如 有两张表,分别是emp_2017 和 emp_2018 .如果需要在查询语句中动态指定表名,就只能使用${}
::
  <select>
      select *  from emp_${year}
  <select>

再比如.需要动态的指定查询中的排序字段,此时也只能使用${}
::
  <select>
       select  *  from dept order by ${name}
  </select>

SQL片段
--------
我们在java代码中会将公用的一些代码提取出来需要的地方直接调用方法即可，在Mybatis也是有类似的概念，那就是SQL片段。

在Mybatis中使用<sql id="xxxxx" />标签定义SQL片段，在需要的地方通过<include refid="xxxxx"/>引用

.. code:: java

  <sql id="userColumns">id,user_name,password,name,age,sex,birthday,created,updated</sql>

  <select id="queryUserById" resultMap="userResultMap" parameterType="Long">
  	SELECT <include refid="userColumns"/> FROM tb_user WHERE id = #{id}
  </select>
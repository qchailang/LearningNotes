#{}和${}的区别总结：

#{}：

    是预编译
    编译成占位符
    可以防止sql注入
    自动判断数据类型
    一个参数时，可以使用任意参数名称进行接收

${}:

    非预编译
    sql的直接拼接
    不能防止sql注入
    需要判断数据类型，如果是字符串，需要手动添加引号。
    一个参数时，参数名称必须是value，才能接收参数。
 ———————————————— 
版权声明：本文为CSDN博主「Demon_gu」的原创文章，遵循CC 4.0 by-sa版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_23329167/article/details/82744301


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

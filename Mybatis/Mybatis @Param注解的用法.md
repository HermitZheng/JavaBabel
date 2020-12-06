### Mybatis @Param注解的用法



@Param注解一般用于Mapper Interface中，写在方法的参数之前。

例如：

```java
@Mapper
public interface UserMapper {
    int insert(@Param("username") String username, @Param("password") String password);
}
```

并不是所有时候都会用到@Param注解，一般在以下的一些场景中会用到这个注解。



#### 1. 方法中只有一个参数

一般java 接口不使用 @Param 注解，同时 mapper 文件也不需要使用 parameterType 这个参数，Mybatis会自动对唯一参数进行类型匹配；当参数类型为实体类时，也会自动识别并对属性进行匹配。

当然也可以使用注解，此时XML文件中#{}的属性名必须与@Param注解中定义的一致。

例如：@Param("username")，#{username}

此时XML中可以不写ParameterType，参数的类型将根据@Param注解的参数类型决定。



#### 2.  方法中存在多个参数，需要使用@Param注解

例如：

Interface：

```java
@Mapper
public interface UserMapper {
    int insert(@Param("username") String username, @Param("password") String password);
}
```

XML：

```xml
<insert id="insert" parameterType="com.pojo.User">
    insert into user(user_name, user_pass)
    values (#{userName,jdbcType=VARCHAR}, #{userPass,jdbcType=VARCHAR})
</insert>
```



#### 3. 方法中存在多个参数，不使用@Param注解

可以将多个参数封装到JavaBean中，或者使用**Map<key, value>**进行封装。

同时在XML中需要写明：parameterType="java.util.Map

此时在XML中可以使用 #{key} 来获取参数。

但是这使得对方法的参数信息表达不够明确。

个人认为当存在不相关的多个参数时，还是使用注解更为明确。



#### 4. 方法参数要取别名，需要使用@Param注解

例如：

Interface：

```java
@Mapper
public interface UserMapper {
    int insert(@Param("name") String username, @Param("pass") String password);
}
```

XML：

```xml
<insert id="insert" parameterType="com.pojo.User">
    insert into user(user_name, user_pass)
    values (#{name,jdbcType=VARCHAR}, #{pass,jdbcType=VARCHAR})
</insert>
```



#### 5. XML 中的 SQL 使用了 $

$会产生SQL注入的问题，但是当需要传入列名或表名时不得不使用$，这时就需要使用@Param注解。

例如：

Interface：

```java
@Mapper
public interface UserMapper {
    List<User> getAllUsers(@Param("order_by")String order_by);
}
```

XML：

```xml
<select id="getAllUsers" resultType="org.javaboy.helloboot.bean.User">
    select * from user
    <if test="order_by!=null and order_by!=''">
        order by ${order_by} desc
    </if>
</select>
```



#### 6. 动态SQL

**动态SQL指的是根据不同的查询条件 , 生成不同的Sql语句.**

例如：

Interface：

```java
@Mapper
public interface UserMapper {
    List<User> getUserById(@Param("id")Integer id);
}
```

XML：

```xml
<select id="getUserById" resultType="org.javaboy.helloboot.bean.User">
    select * from user
    <if test="id!=null">
        where id=#{id}
    </if>
</select>
```


### Mybatis绑定错误 Invalid bound statement (not found)

相关错误信息：

![image-20200206135044368](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200206135044368.png)

Spring、SpringMVC、Mybatis整合时发生的错误，错误原因在于配置文件以及Mapper.xml文件，有多种可能的错误原因。



#### 原因一：

Mapper Interface和xml文件的定义对应不上，需要检查包名，namespace，函数名称等能否对应上。

具体信息如下：

1. 检查xml文件所在的package名称是否和interface对应的package名称是否一一对应

   例如：

![image-20200206135825138](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200206135825138.png)![image-20200206135848956](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200206135848956.png)



2. 检查xml文件的namespace是否和xml文件的package名称是否一一对应

   例如：

![image-20200206135951307](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200206135951307.png)

![image-20200206140046921](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200206140046921.png)

![image-20200206135848956](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200206135848956.png)



3. 检查函数名称能否对应上

   例如：

   ![image-20200206140836758](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200206140836758.png)

![image-20200206140311241](C:\Users\Hanabi\AppData\Roaming\Typora\typora-user-images\image-20200206140311241.png)



4. 去掉xml文件中的中文注释，**格式化一下xml文件中的代码，规范xml格式**

   

5. 随意在xml文件中加一个空格或者空行然后保存

   ​	原因：实际上是触发了IDE的自动编译功能。由于xml文件在编译的时候，不一定总能立即从源目录复制到class文件的编译目录，因此真正确定有效的方式是将正确的xml文件复制到class输出目录。



#### 原因二：

Maven的静态资源过滤问题。

maven默认会把src/main/resources下的所有配置文件以及src/main/java下的所有java文件打包或发布到target\classes下面.

但是现实我们可能会在src/main/java下面也放置一些配置文件如mybatis mapper配置文件等，如果不做一些额外配置，那我们打包后的项目可能找不到这些必须的资源文件，因此在pom.xml中增加类似如下配置：

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```



#### 原因三：

MapperLocation的配置：



在application.properties配置文件中写上：

```properties
mybatis.mapperLocations=classpath:mapper/*.xml
```



或者在spring-dao.xml（spring或者mybatis的contex中）

在property属性中配置mapperLocation的位置：

```XML
<!-- 配置SqlSessionFactory对象 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 注入数据库连接池 -->
    <property name="dataSource" ref="dataSource"/>
    <!-- 配置MyBaties全局配置文件:mybatis-config.xml -->
    <property name="configLocation" value="classpath:mybatis/mybatis-config.xml"/>
    
    <!--mapper.xml所在位置-->
    <property name="mapperLocations" value="classpath*:mapper/*Mapper.xml" />
    
</bean>
```


# Java多环境项目及Maven的基本命令与原理

## 多环境配置

Java项目在开发过程中会有多个环境设置，比如**开发dev**环境、**测试test**环境、**生产prod**环境等等，所以在项目中我们要去相应的做配置，而不是在部署的时候手动的去改配置。

### 配置profile节点

在`pom.xml`中添加如下代码（与dependencies元素同级）：

```xml
<profiles>
    <profile>
        <!-- 开发环境 -->
        <id>dev</id>
        <properties>
            <!-- 节点名字environment随意 用于下方指定 -->
            <environment>development</environment> 
        </properties>
        <activation>
            <!-- 默认激活该profile节点-->
            <activeByDefault>true</activeByDefault> 
        </activation>
    </profile>
    
    <!-- 测试环境 -->
    <profile>
        <id>test</id>
        <properties>
            <environment>test</environment>
        </properties>
        <!--通过filter，我们可以将不同环境目录下的application.properties文件中的参数值加载到maven中-->
		<!--如果我们有多个properties可以在添加一个filter即可。-->
        <build>
            <filters>
                <filter>
                    src/main/resources/profiles/application.properties
                </filter>
            </filters>
        </build>
    </profile>
    
    <!-- 预演环境 -->
    <profile>
        <id>prev</id>
        <properties>
            <environment>preview</environment>
        </properties>
    </profile>
    
    <!-- 生产环境 -->
    <profile>
        <id>prod</id>
        <properties>
            <environment>production</environment>
        </properties>
    </profile>
</profiles>
```

### 添加与profile配置相对应的配置文件（properties）目录

```
- src
	- main
		- java
		- resources
			- profiles
				- dev.properties（或者dev/xxx.properties）
				- test.properties
				- prod.properties
				...
            - mybatis
            ...
    - test
```

### 配置resource节点（在build节点下）

```xml
<resources>
    <resource>
        <!--打包时包含src/main/resources目录下所有文件以及子目录 -->
        <directory>src/main/resources</directory> 
        <excludes> <!--打包时排除节点-->
            <!--打包时排除src/main/resources/profiles/dev/下所有-->
            <exclude>profiles/dev/*</exclude> 
            <exclude>profiles/test/*</exclude><!-- 同上 -->
            <exclude>profiles/prev/*</exclude><!-- 同上 -->
            <exclude>profiles/prod/*</exclude><!-- 同上 -->
        </excludes>
    </resource>
    <resource>
        <!-- 打包时包含src/main/resources/profiles/${environment}下所有文件,environment变量值和上面随意写的一样 -->
        <directory>src/main/resources/profiles/${environment}</directory>
        <!-- 打包文件输出位置 这里得说一下我这里什么都不写 位置就是上方directory节点中配置的路径-->
        <targetPath>
        </targetPath>
    </resource>
</resources>
```

### 打包发布项目

在IDEA的右侧Maven工具栏中，有Profiles选项可以选择需要的环境，并进行打包等一系列操作。打完的包会在**target**目录中。





## Maven基本知识

### Maven的生命周期

Maven 构建生命周期定义了一个项目构建跟发布的过程。我们在开发项目的时候，不断地在编译、测试、打包、部署等过程，maven的生命周期就是对所有构建过程抽象与统一。

Maven 中定义了三种标准的生命周期：**清理**（clean），**默认**（default）(有时候也称为**构建**)，和**站点**（site）。 这三种生命周期互相独立。每种生命周期包含一些步骤，**这些步骤是有序的**。

> 1、clean 生命周期：清理项目，包含三个步骤。

​	1）pre-clean：执行清理前需要完成的工作。

​	2）clean：清理上一次构建生成的文件。

​	3）post-clean：执行清理后需要完成的工作。

> 2、default 生命周期：构建项目，它有 20 多个步骤，下面仅例举几个重要的 phase 如下:

​	1）validate：验证工程是否正确，所有需要的资源是否可用。

​	2）compile：编译项目的源代码。

​	3）test：使用合适的单元测试框架来测试已编译的源代码。这些测试不需要已打包和布署。

​	4）Package：把已编译的代码打包成可发布的格式，比如 jar。

​	5）verify：运行所有检查，验证包是否有效且达到质量标准。

​	6）install：把包安装到 maven 本地仓库，可以被其他工程作为依赖来使用。

​	7）Deploy：在集成或者发布环境下执行，将最终版本的包拷贝到远程的 repository，使得其他的开发者或者工程可以共享。

> 3、site 生命周期：建立和发布项目站点，步骤如下:

​	1）pre-site：生成项目站点之前需要完成的工作。

​	2）site：生成项目站点文档。

​	3）post-site：生成项目站点之后需要完成的工作。

​	4）site-deploy：将项目站点发布到服务器。

#### 顺序执行

**指定某个生命周期的阶段**

执行 **`mvn install`** 命令，将完成 validate、 compile、test、package、verify、install 阶段，并将 package 生成的包发布到本地仓库中。

执行 **`mvn deploy`** 的时候，它会自动运行 deploy 步骤之前的 validate、compile、test、package 等一系列步骤。

**指定多个不同构建生命周期的阶段**

执行 **`mvn clean deploy`** 命令，首先完成的 clean lifecycle，将以前构建的文件清理，然后再执行 default lifecycle 的 validate、compile、test、package、verify、install、deploy 阶段，将 package 阶段创建的包发布到远程仓库中。

maven 的生命周期是**独立**的，即可以直接运行 mvn clean install site 这三套生命周期, 这等于分别运行 mvn clean, mvn install, mvn site。



### Maven常用命令与参数

```
// 清除，然后再编译
mvn clean compile
```

```
// 激活id为xxx的profile（如果有多个，使用逗号隔开）
mvn -Pxxx,xxx
mvn -Pdev,test
```

```
// 指定java全局属性
mvn -Dxxx=yyy
mvn -Dmaven.test.skip=true
```

```
// 仅在当前项目模块执行命令,不构建子模块
mvn -N, --non-recursive 
```

```
// 显示maven允许的debug信息
mvn -X
```

```
// 强制更新snapshot类型的插件或依赖库(否则maven一天只会更新一次snapshot依赖)
mvn -U
```

```
// 创建Maven的普通Java项目
mvn archetype:generate -DgroupId=packageName -DartifactId=projectName
mvn archetype:generate -DgroupId=com.zhuqiu -DartifactId=MavenTest
```

```
// 创建Maven的Web项目
mvn archetype:generate -DgroupId=packageId -DartifactId=webappName -DarchetypeArtifactId=maven-archetype-webapp
```

```
// 指定为dev开发环境，先进行clean，之后跳过测试并deploy，同时强制更新插件和依赖
mvn clean deploy -Dmaven.test.skip=true -U -Pdev
```

```
// 设置项目名称、路径、打包方式并deploy
mvn deploy:deploy-file -DgroupId=com.zhuqiu -DartifactId=client -Dversion=0.1.0 -Dpackaging=jar -Dfile=/Project/client-0.1.0.jar -DdownloadSources=true -DdownloadJavadocs=true
```


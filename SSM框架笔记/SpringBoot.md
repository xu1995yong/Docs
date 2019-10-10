# Spring Boot



## 1. Spring Boot的特性

1. 能够快速创建基于Spring的应用程序
2. 能够直接使用java main方法启动内嵌的Tomcat、Jetty服务器运行Spring boot程序，不需要部署war包文件
3. 提供约定的starter POM来简化Maven配置，让Maven的配置变得简单。
4. 根据项目的Maven依赖配置，Spring boot自动配置Spring、Spring mvc等
5. 提供了程序的健康检查等功能。
6. 基本可以完全不使用XML配置文件，采用注解配置。

## 2. Spring Boot四大核心

1. 自动配置：针对很多Spring应用程序和常见的应用功能，Spring Boot能自动提供相关配置。
2. 起步依赖：告诉Spring Boot需要什么功能， 它就能引入需要的依赖库
3. Actuator：让你能够深入运行中的Spring Boot应用程序，一探Spring Boot程序的内部信息。
4. 命令行界面(CLI)：这是Spring Boot的可选特性，主要针对Groovy语言使用。

## 3. Spring Boot的父级依赖和起步依赖

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.5.RELEASE</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>	
```

## Spring Boot与Spring MVC的区别









## n. Spring Boot之CommandLineRunner接口和ApplicationRunner接口

在开发中可能会有这样的情景。需要在容器启动的时候执行一些内容。比如读取配置文件，数据库连接之类的。SpringBoot给我们提供了两个接口来帮助我们实现这种需求。这两个接口分别为CommandLineRunner和ApplicationRunner。他们的执行时机为容器启动完成的时候。

这两个接口中有一个run方法，我们只需要实现这个方法即可。

##### CommandLineRunner接口

```java
@Component
public class CommandLineRunnerImpl_1 implements CommandLineRunner {
	@Order(value = 1)
	@Override
	public void run(String... args) throws Exception {
		System.out.println(Thread.currentThread().getName() + "  1");
	}

}
```

##### ApplicationRunner接口

```java
@Component
public class ApplicationRunnerImpl_1 implements ApplicationRunner {
 	@Order(value = 1)
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(args);
        System.out.println("这个是测试ApplicationRunner接口");
    }
}
```

 @Order注解

如果有多个实现类，而你需要他们按一定顺序执行的话，可以在实现类上加上@Order注解。@Order(value=整数值)。SpringBoot会按照@Order中的value值从小到大依次执行。


## 1. SpringBoot的热部署

使用spring-boot-devtools模块可以实现：在程序代码发生改变时，自动进行项目的重新启动和代码的重新部署。

工作原理：在spring-boot-devtools中使用了两个ClassLoader，一个ClassLoader加载那些不会改变的类（例如第三方的Jar包依赖），另一个ClassLoader加载会发生改变的类，称为RestartClassLoader。这样在有代码发生改变时，只使用RestartClassLoader加载发生改变的类。

要使用spring-boot-devtools，只需在pom.xml中添加以下代码：

```xml
<!--添加spring-boot-devtools的依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
    <scope>true</scope>
</dependency>
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>   <!--修改此处-->
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```

# Spring Boot 的数据访问

## 1. Druid连接池

Druid是Java语言中最好的数据库连接池。Druid能够提供强大的监控和扩展功能。


### 1. 使用Druid连接池
1. 在 Spring Boot 项目中加入druid-spring-boot-starter依赖
    Druid Spring Boot Starter 用于帮助你在Spring Boot项目中轻松集成Druid数据库连接池和监控。
    ```xml
    <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid-spring-boot-starter</artifactId>
       <version>1.1.10</version>
    </dependency>
    ```

2. 指定数据源

   - 在application.properties文件中添加数据库连接信息，并指定数据源。

   ```properties
   spring.datasource.url=jdbc:mysql://192.168.100.2:3306/e3mall
   spring.datasource.username=root
   spring.datasource.password=root
   spring.datasource.driver-class-name= com.mysql.jdbc.Driver
   #指定使用 Druid数据源
   spring.datasource.type=com.alibaba.druid.pool.DruidDataSource 
   ```

   - 在application.properties文件中自定义Druid连接池

    ```properties
    spring.datasource.druid.initial-size=50
    spring.datasource.druid.min-idle: 5
    spring.datasource.druid.max-active: 100
    spring.datasource.druid.query-timeout: 6000  
    spring.datasource.druid.transaction-query-timeout: 6000  
    spring.datasource.druid.remove-abandoned-timeout: 1800  
    spring.datasource.druid.filter-class-names: stat
    spring.datasource.druid.filters: stat,config
	 ```


### 2. Druid的监控统计功能

1. 在application.properties文件中添加如下代码：

    ```properties
    # WebStatFilter配置，说明请参考Druid Wiki，配置_配置WebStatFilter
    spring.datasource.druid.web-stat-filter.enabled=true
    spring.datasource.druid.web-stat-filter.url-pattern=/*
    spring.datasource.druid.web-stat-	filter.exclusions=*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*
    
    # StatViewServlet配置，说明请参考Druid Wiki，配置_StatViewServlet配置
    spring.datasource.druid.stat-view-servlet.enabled= true
    spring.datasource.druid.stat-view-servlet.url-pattern= /druid/*
    spring.datasource.druid.stat-view-servlet.reset-enable= false
    spring.datasource.druid.stat-view-servlet.login-username= admin
    spring.datasource.druid.stat-view-servlet.login-password= admin
    ```

2. 访问

   http://localhost:8080/druid/index.html

### 3. Druid多数据源配置
https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter
https://my.oschina.net/u/3681868/blog/1813011
## 2. 整合Mybatis

### 1. 注解版

1. 配置数据库连接池
2. 添加依赖

    ```xml
    <!-- 引入 mysql 驱动依赖-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <!--引入 Mybatis 的starter-->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
    </dependency>
    ```


3. 创建POJO类

   ```java
   //省略
   ```

4. 构造数据访问层

   - 创建Mapper接口

       ```java
       public interface UserMapper {

        @Select("select * from tb_user  where id = #{id}")
        public TbUser getUserById(Integer id);

        @Delete("delete from tb_user where id = #{id}")
        public int delectUserById(Integer id);
       }
       ```

   - 在Spring中注册Mapper

     ```java
     //方法一：在SpringBoot启动类中添加Mapper扫描的包路径
     @MapperScan(basePackages = "com.example.demo.mapper")
     //方法二：在每个Mapper接口中添加Mapper注解
     @Mapper
     public interface UserMapper {
         //省略
     }
     ```

5. 使用Mapper
   在Service中使用@Autowired注解导入UserMapper类的对象，并调用其方法。

### 2. 配置文件版
1. 配置数据库连接池
2. 添加依赖

3. 创建POJO类

4. 构造数据访问层

   - 编写mybatis的全局配置文件

     在resources文件夹中创建mybatis-config.xml文件，并添加以下信息：

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE configuration
       PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-config.dtd">
     <configuration>
     
     </configuration>
     ```

   - 创建Mapper接口

     ```java
     public interface UserMapper {
     	public TbUser getUserById(Integer id);
     	public int deleteUserById(Integer id);
     }
     ```

   - 编写Mybatis的映射文件

     在resources/mapper文件夹中创建UserMapper.xml映射文件，并添加以下信息：

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="com.example.demo.mapper.UserMapper">
     	<select id="getUserById" resultType="com.example.demo.pojo.TbUser">
     		select * from tb_user where id = #{id}
     	</select>
     	<delete id="deleteUserById">
     		delete from tb_User where id = #{id}
     	</delete>
     </mapper>
     ```

   - 指明mybatis配置文件的路径

     在resources/application.properties中添加如下信息：

     ```properties
     mybatis.config-location=classpath:mybatis/mybatis-config.xml 
     mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
     ```

5. 使用Mapper



# SpringBoot与Dubbo整合

## 一、前期工作
### 1.安装zookeeper
### 2.创建SpringBoot项目


## 二、配置Provider

### 1.添加Dubbo、zookeeper、zkclient的maven依赖

```xml
<dependency>
	<groupId>com.alibaba.spring.boot</groupId>
	<artifactId>dubbo-spring-boot-starter</artifactId>
	<version>2.0.0</version>
</dependency>
<dependency>
	<groupId>org.apache.zookeeper</groupId>
	<artifactId>zookeeper</artifactId>
	<version>3.3.3</version>
	<exclusions>
		<exclusion>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
		</exclusion>
		<exclusion>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>com.github.sgroschupf</groupId>
	<artifactId>zkclient</artifactId>
	<version>0.1</version>
</dependency>
```

### 2.在SpringBootApplication加入@EnableDubboConfiguration注解
```java
@SpringBootApplication
@EnableDubboConfiguration
public class SpringBootDubboProviderApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringBootDubboProviderApplication.class, args);
	}
}
```

### 3.在项目配置文件中添加
	spring.dubbo.application.name=provider
	spring.dubbo.server=true
	spring.dubbo.registry.address=zookeeper://192.168.230.106:2181?client=zkclient   

### 4.编写要暴露的服务接口，并打jar包，提供给consumer

### 5.实现接口，并添加Service注解
```java
//注意，这里的service注解是com.alibaba.dubbo.config.annotation.Service
@Component
@Service(version = "1.0")
public class HelloServiceImpl implements HelloService {

	@Override
	public String getHello() {
		return "Hello,world!";
	}
}
```

## 三、配置Consumer

### 1.添加Dubbo、zookeeper、zkclient的maven依赖
	//同上

### 2.在SpringBootApplication加入@EnableDubboConfiguration注解
	//同上

### 3.在项目配置文件中添加
	spring.dubbo.application.name=consumer
	spring.dubbo.server=true
	spring.dubbo.registry.address=zookeeper://192.168.230.106:2181?client=zkclient

### 4.导入要使用的服务接口
	//要注意包名必须与provider提供的相同

### 5.使用Reference注解引用接口，并调用接口的方法
	public class HelloController {
	
		@Reference(version = "1.0")
		private HelloService helloService;
	
		@RequestMapping("/hello")
		public String getHello() {
			return helloService.getHello();
		}
	}
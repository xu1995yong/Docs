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




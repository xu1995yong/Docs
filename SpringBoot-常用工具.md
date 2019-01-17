# 1. SpringBoot的热部署

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

# 2. SpringBoot中的日志

1. 开启DEBUG模式

   在application. properties文件中添加：

   ```properties
   debug=true
   ```

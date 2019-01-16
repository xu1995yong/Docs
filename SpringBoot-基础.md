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

##### @Order注解

如果有多个实现类，而你需要他们按一定顺序执行的话，可以在实现类上加上@Order注解。@Order(value=整数值)。SpringBoot会按照@Order中的value值从小到大依次执行。



##### 







​	 
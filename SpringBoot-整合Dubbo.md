# SpringBoot与Dubbo整合

## 一、前期工作
### 1.安装zookeeper
### 2.创建SpringBoot项目


## 二、配置Provider

### 1.添加Dubbo、zookeeper、zkclient的maven依赖

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

### 2.在SpringBootApplication加入@EnableDubboConfiguration注解
	@SpringBootApplication
	@EnableDubboConfiguration
	public class SpringBootDubboProviderApplication {
		public static void main(String[] args) {
			SpringApplication.run(SpringBootDubboProviderApplication.class, args);
		}
	}

### 3.在项目配置文件中添加
	spring.dubbo.application.name=provider
	spring.dubbo.server=true
	spring.dubbo.registry.address=zookeeper://192.168.230.106:2181?client=zkclient   

### 4.编写要暴露的服务接口，并打jar包，提供给consumer

### 5.实现接口，并添加Service注解
	//注意，这里的service注解是com.alibaba.dubbo.config.annotation.Service
	@Component
	@Service(version = "1.0")
	public class HelloServiceImpl implements HelloService {

		@Override
		public String getHello() {
			return "Hello,world!";
		}
	}

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
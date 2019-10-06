###  在IOC容器中装配Bean（4种方式）

##### 方式一：基于XML配置

1. 创建Bean的实现类

   ```java
   public class Person {
   	private String name;
   	private int age;
       public Person(String name, int age) {this.name = name;this.age = age;}
   	public Person() {}
   	public String getName() {return name;}
   	public void setName(String name) {this.name = name;}
   	public int getAge() {return age;}
   	public void setAge(int age) {this.age = age;}
   	@Override
   	public String toString() {
   		return "Person [name=" + name + ", age=" + age + "]";
   	}
   }
   ```

2. 使用xml装载bean

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans      http://www.springframework.org/schema/beans/spring-beans.xsd">
   
   	<bean id="person" class="com.xu.beans.Person">
           <!--为bean的属性赋初值-->
   		<property name="name" value="zhangsan"></property>
   		<property name="age" value="10"></property>
   	</bean>
   </beans>
   ```

3. 获取bean

   ```java
   ApplicationContext applicationContext = new ClassPathXmlApplicationContext("application.xml");
   Person person = (Person) applicationContext.getBean("person");
   ```

##### 方式二：基于注解配置

1. 编写bean的实现类并在实现类上添加注解

    ```java
    //添加了注解的实现类可以被Spring容器识别，Spring容器自动将POJO转化为容器管理的Bean。
    //默认Bean的name为第一个字母小写的类名
    /**
    * @Repository 用于对DAO实现类进行标注。
    * @Service 用于对Service实现类进行标注。
    * @Controller 用于对Controller实现类进行标注
    * @Component 可用于所有POJO的标注
    */
    @Component
    public class Person {
        //（同方式一中的Person类）
    }
    ```
2. 在配置文件中定义组件扫描的包

   ```xml
   <context:component-scan base-package="com.xu.beans"></context:component-scan>
   ```

3. 加载配置文件并获取bean

   ```java
   ApplicationContext applicationContext = new ClassPathXmlApplicationContext("application.xml");
   Person person = (Person) applicationContext.getBean("person");
   System.out.println(person);
   ```

##### 方式三：基于Java类配置

1. 编写bean的实现类

   ```java
   //（同方式一中的Person类）
   ```

2.  编写Beans的配置类

   ```java
   @Configuration 
   //普通的POJO只要标注@Configuration注解，就可以为Spring容器提供bean定义的信息。每个标注了@Bean注解的类方法都相当于提供了一个Bean的定义信息。其中Bean的类型由方法返回值的类型决定，名称默认和方法名相同，也可以通过@Bean注解的name属性显式指定。
   public class BeansConfig {
   
       //Spring会对配置类种所有标注@Bean的方法进行改造（AOP增强），将对Bean生命周期管理的逻辑植入进来。所以即使调用new BeansConfig().person_1()方法，不是简单的执行person_1种的定义的方法逻辑，而是从Spring容器中返回相应的Bean的单例。
   	@Bean(name = "person1")
   	public Person person_1() {
   		return new Person();
   	}
   
   	@Bean(name = "person2")
   	public Person person_2() {
   		return new Person("zhangsan", 20);
   	}
   }
   ```
3. 获取Bean 

   ```java
    ApplicationContext ctx = new AnnotationConfigApplicationContext(BeansConfig.class);//在此处传入bean的配置类的class对象
       Person person = (Person) ctx.getBean("person1");
       System.out.println(person);
   
       //加载多个配置类
       AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
       ctx.register(BeansConfig_1.class);
       ctx.register(BeansConfig_2.class);
       ctx.refresh();
       Person person = (Person) ctx.getBean("person1");
       System.out.println(person);
   ```
   
   
   
   


## SpringAOP的使用

### 1.导包

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.2.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>aopalliance</groupId>
    <artifactId>aopalliance</artifactId>
    <version>1.0</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.6.8</version>
</dependency>
```

### 2.定义目标接口及其实现类

```java
//目标接口
public interface HelloService {
    public void sayHello();
}
//实现类
public class HelloServiceImpl implements HelloService {
    public void sayHello() {
        System.out.println("hello!");
    }
}
```

### 3.定义通知

```java
public class MyAdvice {
    // 前置通知
    public void before() {
        System.out.println("before！");
    }
    // 后置通知(如果出现异常不会调用)
    public void afterReturning() {
        System.out.println("afterReturning！");
    }
    // 环绕通知
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("这是环绕通知之前的部分!!");
        Object proceed = pjp.proceed();// 调用目标方法
        System.out.println("这是环绕通知之后的部分!!");
        return proceed;
    }
    // 异常通知
    public void afterException() {
        System.out.println("afterException");
    }
    // 后置通知(出现异常也会调用)
    public void after() {
        System.out.println("after");
    }
}
```

### 4.将通知织入目标方法

方法一：使用xml文件配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- 准备工作: 导入aop(约束)命名空间 -->
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                           http://www.springframework.org/schema/aop
                           http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">
    <!-- 1.配置目标对象 -->
    <bean name="HelloService" class="com.xu.service.HelloServiceImpl"></bean>
    <!-- 2.配置通知对象 -->
    <bean name="myAdvice" class="com.xu.advice.MyAdvice"></bean>
    <!-- 3.配置将通知织入目标对象 -->
    <aop:config>
        <!-- 配置切入点 -->
        <!-- execution(* com.xu.service.*ServiceImpl.*(..)) 代表 com.xu.service包下的以ServiceImpl为结尾的所有实现类下的所有方法（任何返回值、任何参数）， -->	
        <aop:pointcut expression="execution(* com.xu.service.*ServiceImpl.*(..))" id="pointCut" />
        <aop:aspect ref="myAdvice">
            <aop:before method="before" pointcut-ref="pointCut" />
            <aop:after-returning method="afterReturning" pointcut-ref="pointCut" />
            <aop:after method="after" pointcut-ref="pointCut" />
            <aop:after-throwing method="afterException" pointcut-ref="pointCut" />
            <aop:around method="around" pointcut-ref="pointCut" />

        </aop:aspect>
    </aop:config>
</beans>
```





5. 
































































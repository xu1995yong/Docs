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

方法二：使用注解

```java

```


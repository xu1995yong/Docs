# Spring AOP

## JAVA中的几种代理

### 静态代理

在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。

```java
//1.被代理接口
public interface HelloService {
    public void echo(String msg);
}
//2.被代理接口的实现类
public class HelloServiceImpl implements HelloService {
    public void echo(String msg) {
        System.out.println("echo:" + msg);
    }
}
//3.代理类
public class HelloServiceProxy implements HelloService {
    private HelloService helloService;
    public HelloServiceProxy(HelloService helloService) {
        this.helloService = helloService;
    }
    public void echo(String msg) {
        helloService.echo(msg);
    }
}
//4.测试
public class Test {
    public static void main(String args[]) {
        HelloService helloService = new HelloServiceImpl();
        HelloService helloServiceProxy = new HelloServiceProxy(helloService);
        helloServiceProxy.echo("hello");
    }
}
```

### JDK动态代理
被代理类的源码是在程序运行期间由JVM根据反射等机制动态的生成，代理类和被代理类的关系是在程序运行时确定。

JDK动态代理缺点：只能为接口创建代理实例，即要求被代理对象必须存在接口。

```java
//1.被代理接口 -> 同上
//2.被代理接口的实现类 -> 同上
//3.实现InvocationHandler接口
//InvocationHandler是被调用类的调用处理器，每个代理类的实例都关联了一个handler当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。
public class HelloServiceHandler implements InvocationHandler {
    private HelloService helloService;
    public HelloServiceHandler(HelloService target) {
        this.helloService = target;
    }
    //将事务处理的横切代码编织到业务代码中
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println(proxy instanceof HelloService);
        System.out.println("开始事务" + helloService.getClass().getName() + "." + method.getName());
        //通过java的反射机制间接调用
        Object obj = method.invoke(helloService, args);
        System.out.println("结束事务");
        return obj;
    }
}
//4.获取被代理对象并测试
public class JDKProxyTest {
    @Test
    public void jdkProxyTest() {
        HelloService target = new HelloServiceImpl();
        HelloServiceHandler handler = new HelloServiceHandler(target);
        HelloService proxy = (HelloService) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), handler);
        proxy.echo("Hello");
    }
}
```
### CGLib代理
CGLib采用底层的字节码技术，可以为被代理类创建子类（不需要被代理类的接口），然后再子类中采用方法拦截的技术拦截所有父类方法的调用并顺势织入横切逻辑。

```java
//1.被代理类 -> 同上
//2.实现方法拦截器
//方法拦截器，在调用目标方法时，CGLib会回调MethodInterceptor接口方法拦截
public class CGlibProxy implements MethodInterceptor {	
    public Object getProxy(Class superClass) {
        Enhancer enhancer = new Enhancer();
        // 设置产生的代理对象的父类。
        enhancer.setSuperclass(superClass);
        enhancer.setCallback(this);
        Object obj = enhancer.create();
        return obj;
    }
    //拦截所有被代理类方法的调用
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println(obj instanceof HelloServiceImpl); //true
        System.out.println("开始事务" + obj.getClass().getName() + "." + method.getName());
        Object result = methodProxy.invokeSuper(obj, args);
        System.out.println("结束事务");
        return result;
    }
}
//3.获取被代理对象并测试
public class CglibProxyTest {
    @Test
    public void proxy() {
        CGlibProxy proxy = new CGlibProxy();
        HelloServiceImpl helloService = (HelloServiceImpl) proxy.getProxy(HelloServiceImpl.class);
        helloService.echo("hello");
    }
}
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





## SpringAOP相关概念

1、横切关注点

对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点

2、切面（aspect）

类是对物体特征的抽象，切面就是对横切关注点的抽象

3、连接点（joinpoint）

被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器

4、切入点（pointcut）

对连接点进行拦截的定义

5、通知（advice）

所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类

6、目标对象

代理的目标对象

7、织入（weave）

将切面应用到目标对象并导致代理对象创建的过程

8、引入（introduction）

在不修改代码的前提下，引入可以在**运行期**为类动态地添加一些方法或字段
































































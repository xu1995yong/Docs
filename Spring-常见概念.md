## 常见概念

1. 控制反转（IOC）

控制反转是一种设计思想。在传统的Java程序中，直接通过new关键字创建对象，这样导致程序的强耦合性。为了降低耦合，将对象的创建过程控制权交给了容器，由容器进行注入组合对象。而程序只是被动的接受对象

2. 依赖注入（DI)
3. 面向切面编程（AOP)





### BeanFactory 和 ApplicationContext的区别





## SpringAOP相关概念

1、

对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点

2、

类是对物体特征的抽象，切面就是对横切关注点的抽象

3、连接点（joinpoint）

一段代码中的一些具有边界性质的点。Spring只支持方法类型的连接点，即仅能在方法调用前、方法调用后、方法抛出异常时等。

4. 切入点（pointcut）

对连接点进行拦截的定义

5. 增强（advice）：要织入到目标类连接点上的代码，增强分为前置、后置、异常、最终、环绕增强五类。
6. 切面（aspect）：
7. 目标对象：增强逻辑的织入目标类。
8. 织入（weave）：将增强添加到目标类的具体连接点上的过程。
9. 引入（introduction）：一种特殊的增强，在不修改代码的前提下，引入可以在**运行期**为类动态地添加一些方法或字段
10. 代理类：类被AOP织入增强后产生的代理类。





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
//实现InvocationHandler接口
//InvocationHandler是被调用类的调用处理器，每个代理类的实例都关联了一个handler当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。
public class HelloServiceHandler implements InvocationHandler {
    private HelloService helloService;
    public HelloServiceHandler(HelloService target) {
        this.helloService = target;
    }
    //将事务处理的横切代码编织到业务代码中。 Method method是真实对象中调用方法的Method类
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println(proxy instanceof HelloService);
        System.out.println("开始事务" + helloService.getClass().getName() + "." + method.getName());
        //通过java的反射机制间接调用。事先不知道调用目标对象的哪个方法
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





## Spring中的事务



### 事务的传播行为

事务的传播行为是指，当前事务方法调用另一个事务方法时，当前方法的事务如何传播到被调用的方法中。

Spring中定义有7种事务的传播行为：

- PROPAGATION_REQUIRED：加入到当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。 

- PROPAGATION_SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。 

- PROPAGATION_MANDATORY：使用当前事务，如果当前没有事务，就抛出异常。 

- PROPAGATION_REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。 

- PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 

- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。 

- PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。
##静态代理

	在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。
	1.被代理接口
	public interface HelloService {
		public void echo(String msg);
	}
	2.被代理接口的实现类
	public class HelloServiceImpl implements HelloService {
		public void echo(String msg) {
			System.out.println("echo:" + msg);
		}
	}
	3.代理类
	public class HelloServiceProxy implements HelloService {
		private HelloService helloService;
		public HelloServiceProxy(HelloService helloService) {
			this.helloService = helloService;
		}
		public void echo(String msg) {
			helloService.echo(msg);
		}
	}
	4.测试
	public class Test {
		public static void main(String args[]) {
			HelloService helloService = new HelloServiceImpl();
			HelloService helloServiceProxy = new HelloServiceProxy(helloService);
			helloServiceProxy.echo("hello");
		}
	}

##JDK动态代理
	被代理类的源码是在程序运行期间由JVM根据反射等机制动态的生成，代理类和被代理类的关系是在程序运行时确定。
	1.被代理接口 -> 同上
	2.被代理接口的实现类 -> 同上
	3.实现InvocationHandler接口
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
	4.获取被代理对象并测试
	public class JDKProxyTest {
		@Test
		public void jdkProxyTest() {
			HelloService target = new HelloServiceImpl();
			HelloServiceHandler handler = new HelloServiceHandler(target);
			HelloService proxy = (HelloService) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), handler);
			proxy.echo("Hello");
		}
	}
	JDK动态代理缺点：只能为接口创建代理实例，即要求被代理对象必须存在接口。

##CGLib代理
	CGLib采用底层的字节码技术，可以为被代理类创建子类（不需要被代理类的接口），然后再子类中采用方法拦截的技术拦截所有父类方法的调用并顺势织入横切逻辑。
	1.被代理类 -> 同上
	2.实现方法拦截器
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
	3.获取被代理对象并测试
	public class CglibProxyTest {
		@Test
		public void proxy() {
			CGlibProxy proxy = new CGlibProxy();
			HelloServiceImpl helloService = (HelloServiceImpl) proxy.getProxy(HelloServiceImpl.class);
			helloService.echo("hello");
		}
	}
##  SpringIOC源码解析

###  refresh()

```java
//AbstractApplicationContext类
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();
        // Tell the subclass to refresh the internal bean factory.
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);
        // Allows post-processing of the bean factory in context subclasses.
        postProcessBeanFactory(beanFactory);
        // Invoke factory processors registered as beans in the context.
        invokeBeanFactoryPostProcessors(beanFactory);
        //注册所有的BeanPostProcessor bean，该方法必须在任何应用程序bean实例化之前调用。
        registerBeanPostProcessors(beanFactory);
        // Initialize message source for this context.
        initMessageSource();
        // Initialize event multicaster for this context.
        initApplicationEventMulticaster();
        // Initialize other special beans in specific context subclasses.
        onRefresh();
        // Check for listener beans and register them.
        registerListeners();

        //初始化所有剩余的非懒加载的单实例bean。极其重要的方法！
        finishBeanFactoryInitialization(beanFactory);

        // Last step: publish corresponding event.
        finishRefresh();
    }
}
```
### finishBeanFactoryInitialization()
```java
//AbstractApplicationContext类
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 调用beanFactory的方法，初始化所有剩余的非懒加载的单实例bean
    beanFactory.preInstantiateSingletons();
}
```

### preInstantiateSingletons()

```java
//DefaultListableBeanFactory类
public void preInstantiateSingletons(){
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
    for (String beanName : beanNames) {
        //根据beanName获取对应的BeanDefinition对象，BeanDefinition对象中保存了bean的定义信息
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            //判断是否是FactoryBean；是否是实现FactoryBean接口的Bean；
            if (isFactoryBean(beanName)) {
            }
            else {
                //不是工厂Bean。调用AbstractBeanFactory的getBean(beanName)创建对象
                getBean(beanName);
            }
        }
    }
}
```
### getBean()
```java
//AbstractBeanFactory
//根据beanName获取bean，ctx.getBean()就是该方法
public Object getBean(String name) throws BeansException { 
    return doGetBean(name, null, null, false);
}
protected <T> T doGetBean(String name,Class<T> requiredType,Object[] args, boolean typeCheckOnly) {
    final String beanName = transformedBeanName(name);
    //尝试从缓存中获取该单实例Bean。如果能获取说明该Bean之前被创建过,所有创建过的单实例Bean都会被缓存起来
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }
    //缓存中获取不到，开始该单实例Bean的对象创建流程。
    else {
        // 如果当前Bean依赖其他Bean，则调用getBean()先把依赖的Bean先创建出来
        String[] dependsOn = mbd.getDependsOn();
        if (dependsOn != null) {
            for (String dep : dependsOn) {
                getBean(dep);
            }
        }
        //正式启动单实例Bean的创建流程
        if (mbd.isSingleton()) {
            //此处调用getSingleton()方法获取bean实例。
            //第二个参数传入ObjectFactory类的匿名子类，重写的getObject()方法中调用了createBean()方法
            sharedInstance = getSingleton(beanName, () -> {
                return createBean(beanName, mbd, args);
            });
            bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
        }
    }
    return (T) bean;
}
```
### getSingleton()

```java
//DefaultSingletonBeanRegistry类
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            //getObject()直接调用AbstractAutowireCapableBeanFactory类createBean()方法
            singletonObject = singletonFactory.getObject();
            newSingleton = true;
            if (newSingleton) {
                addSingleton(beanName, singletonObject);//将创建的Bean添加到缓存中singletonObjects
            }
        }
        return singletonObject;
    }
}
```

### createBean()

```java
//AbstractAutowireCapableBeanFactory类
protected Object createBean(String beanName, RootBeanDefinition mbd,Object[] args){
    RootBeanDefinition mbdToUse = mbd;
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    return beanInstance;
}
```
### doCreateBean()
```java
protected Object doCreateBean(String beanName,RootBeanDefinition mbd,Object[] args){
    BeanWrapper instanceWrapper = null;
    if (instanceWrapper == null) {
        //利用工厂方法或者利用反射对象的构造器创建出Bean实例。
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    final Object bean = instanceWrapper.getWrappedInstance();
    Class<?> beanType = instanceWrapper.getWrappedClass();
// Allow post-processors to modify the merged bean definition.
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}
    // Eagerly cache singletons to be able to resolve circular references
		// even when triggered by lifecycle interfaces like BeanFactoryAware.
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
    Object exposedObject = bean;
    populateBean(beanName, mbd, instanceWrapper); //调用Bean属性的set()方法赋值
    exposedObject = initializeBean(beanName, exposedObject, mbd);
    //注册bean的destory()方法,包括DisposableBean接口的destroy()和配置文件中<bean>的destroy-method属性指定的销毁方法
    registerDisposableBeanIfNecessary(beanName, bean, mbd);
    return exposedObject;
}
```
### populateBean()
```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);
    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}

```
### initializeBean()
```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    invokeAwareMethods(beanName, bean);//调用bean实现的aware接口的方法
    //调用BeanPostProcessor的postProcessBeforeInitialization()方法
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    //调用InitializingBean的afterPropertiesSet()方法
    //之后调用配置文件中<bean>的init-method属性指定的初始化方法
    invokeInitMethods(beanName, wrappedBean, mbd);

    // 调用BeanPostProcessor的postProcessAfterInitialization()方法
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);

    return wrappedBean;
}

```



### obtainFreshBeanFactory()

```java
//AbstractApplicationContext
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory();//抽象方法
    return getBeanFactory();
}
```





```java
//AbstractRefreshableApplicationContext中的实现
protected final void refreshBeanFactory() throws BeansException {
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    //新建一个DefaultListableBeanFactory对象并返回
    DefaultListableBeanFactory beanFactory = createBeanFactory();
    beanFactory.setSerializationId(getId());
    customizeBeanFactory(beanFactory);
    loadBeanDefinitions(beanFactory); //抽象方法，将bean的定义加载到给定的bean工厂中
    synchronized (this.beanFactoryMonitor) {
        this.beanFactory = beanFactory;
    }
}
```

```java
//AbstractXmlApplicationContext中的实现
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) {
    // Create a new XmlBeanDefinitionReader for the given BeanFactory.
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    // Configure the bean definition reader with this context's resource loading environment.
    beanDefinitionReader.setEnvironment(this.getEnvironment());
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    // Allow a subclass to provide custom initialization of the reader,
    // then proceed with actually loading the bean definitions.
    initBeanDefinitionReader(beanDefinitionReader);
    loadBeanDefinitions(beanDefinitionReader);
}
```





## bean的生命周期

![img](https://images2015.cnblogs.com/blog/801753/201608/801753-20160809105632527-898343609.jpg)

1. Spring容器从XML文件中读取bean的定义，之后调用bean的构造器实例化bean。
2. Spring根据bean的定义注入所有的属性。
3. 如果bean实现了BeanNameAware接口，Spring会调用其setBeanName(String name)方法，传递bean的ID。
4. 如果Bean实现了BeanFactoryAware接口，Spring会调用其setBeanFactory(BeanFactory beanFactory)方法，传递bean所属的beanfactory。
5. 如果beanfactory中注册了BeanPostProcessors，Spring会调用其postProcessBeforeInitialization()方法。
6. 如果bean实现了IntializingBean接口，Spring会调用其afterPropertySet()方法
7. 调用配置文件中<bean>的init-method属性指定的初始化方法
8. 如果beanfactory中注册了BeanPostProcessors，Spring会调用其postProcessAfterInitialization()方法。
9. Bean初始化成功
10. 如果bean实现了DisposableBean接口，Spring会调用其destroy()方法。
11. 调用配置文件中<bean>的destroy-method属性指定的销毁方法。





## Spring中循环依赖问题

循环依赖即循环引用，就是两个或以上的bean互相持有对方，最终形成闭环。比如A依赖于B，B依赖于C，C依赖于A。

Spring中循环依赖场景有： 构造器的循环依赖 、field属性的循环依赖。

### Spring中循环依赖的检测



### Spring中解决循环依赖

Spring解决循环依赖的理论依据其实是Java基于引用传递，即获取java对象的引用时，对象的字段可以延后设置的。

Spring中将Bean的创建过程分为三步，都在AbstractAutowireCapableBeanFactory类的doCreateBean()方法中。

1. createBeanInstance(beanName, mbd, args)：创建bean的实例。
2. populateBean(beanName, mbd, instanceWrapper)：属性填充
3. initializeBean(beanName, exposedObject, mbd)：调用配置文件中指定的init-method初始化对象。

Spring为了解决单例的循环依赖问题，使用了三级缓存。定义在DefaultSingletonBeanRegistry类中。

```java
/**用于存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用 */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/**存放早期的 bean 对象（尚未填充属性），用于解决循环依赖 */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

/**存放 bean 工厂对象，用于解决循环依赖 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```







```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    //先从singletonObjects缓存中获取对象
    Object singletonObject = this.singletonObjects.get(beanName);
    //如果从singletonObjects缓存中获取不到，并且当前对象正在创建中。说明当前对象还没有创建完全，
    //即存在循环依赖的问题。此时从earlySingletonObjects缓存中获取对象。
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            //如果earlySingletonObjects缓存中获取不到，且允许创建早期应用
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```









## Spring AOP源码分析



### AutoProxyCreator的注册











### 生成代理对象

在bean的初始化过程中，会调用bean对应的BeanPostProcessors。AbstractAutoProxyCreator是BeanPostProcessors的子类，重写了接口的postProcessAfterInitialization方法。在该方法中会对bean进行包装，也就是创建该bean的代理对象。

创建代理的流程为：首先获取所有的Advisor，然后调用AbstractAutoProxyCreator对象的createProxy()方法。在该方法中，首先创建一个proxyFactory对象，然后调用proxyFactory对象的getProxy()方法，在该方法中，获取了JdkDynamicAopProxy对象，该对象实现了JDK的InvocationHandler接口，所以能使用动态代理创建代理对象。

#### initializeBean()

```java
//AbstractAutowireCapableBeanFactory
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
            invokeAwareMethods(beanName, bean);
            return null;
        }, getAccessControlContext());
    }
    else {
        invokeAwareMethods(beanName, bean);
    }
    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
    try {
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean;
}
```

#### applyBeanPostProcessorsAfterInitialization()

```java
//AbstractAutowireCapableBeanFactory
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {

    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```

#### postProcessAfterInitialization()

```java
//AbstractAutoProxyCreator
public Object postProcessAfterInitialization(Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (!this.earlyProxyReferences.contains(cacheKey)) {
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

#### wrapIfNecessary()

```java
//AbstractAutoProxyCreator
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
        return bean;
    }
    if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
        return bean;
    }
    if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
        this.advisedBeans.put(cacheKey, Boolean.FALSE);
        return bean;
    }
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        //创建bean的代理对象
        Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }

    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}
```

#### createProxy()

```java
//AbstractAutoProxyCreator
protected Object createProxy(Class<?> beanClass,String beanName,Object[] specificInterceptors, TargetSource targetSource) {

    if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
        AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
    }

    ProxyFactory proxyFactory = new ProxyFactory();
    proxyFactory.copyFrom(this);

    if (!proxyFactory.isProxyTargetClass()) {
        if (shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        }
        else {
            evaluateProxyInterfaces(beanClass, proxyFactory);
        }
    }

    Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
    proxyFactory.addAdvisors(advisors);
    proxyFactory.setTargetSource(targetSource);
    customizeProxyFactory(proxyFactory);

    proxyFactory.setFrozen(this.freezeProxy);
    if (advisorsPreFiltered()) {
        proxyFactory.setPreFiltered(true);
    }

    return proxyFactory.getProxy(getProxyClassLoader());
}
```

#### getProxy()

```java
//ProxyFactory
public Object getProxy(@Nullable ClassLoader classLoader) {
    return createAopProxy().getProxy(classLoader); 
}

//ProxyCreatorSupport是ProxyFactory的父类
protected final synchronized AopProxy createAopProxy() {
    if (!this.active) {
        activate();
    }
    return getAopProxyFactory().createAopProxy(this); //DefaultAopProxyFactory
}

//DefaultAopProxyFactory
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if(config.isOptimize()||config.isProxyTargetClass()||hasNoUserSuppliedProxyInterfaces(config)){
        Class<?> targetClass = config.getTargetClass();
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);//jdk动态代理
        }
        return new ObjenesisCglibAopProxy(config);//cglib代理
    }
    else {
        return new JdkDynamicAopProxy(config);//config
    }
}
```

#### getProxy()

```java
//JdkDynamicAopProxy
public Object getProxy(@Nullable ClassLoader classLoader) {
    Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
    findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
    //这里的this指的是JdkDynamicAopProxy对象，该对象实现了JDK的InvocationHandler接口
    return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```

### 执行目标方法

#### invoke()

```java
//JdkDynamicAopProxy
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
        if (!this.equalsDefined && AopUtils.isEqualsMethod(method)) {
            return equals(args[0]);
        }
        else if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)) {
            return hashCode();
        }
        else if (method.getDeclaringClass() == DecoratingProxy.class) {
            return AopProxyUtils.ultimateTargetClass(this.advised);
        }
        //获取目标方法的拦截器链.
        List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
        if (chain.isEmpty()) {
            //如果拦截器链为空，则直接对目标方法进行反射调用
            Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
            retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
        }
        else {
            //创建Invocation对象
            invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
            // Proceed to the joinpoint through the interceptor chain.
            retVal = invocation.proceed();
        }

        Class<?> returnType = method.getReturnType();
        if (retVal != null && retVal == target &&
            returnType != Object.class && returnType.isInstance(proxy) &&
            !RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
            // Special case: it returned "this" and the return type of the method
            // is type-compatible. Note that we can't help if the target sets
            // a reference to itself in another returned object.
            retVal = proxy;
        }
        return retVal;
    }
    finally {
        if (target != null && !targetSource.isStatic()) {
            // Must have come from TargetSource.
            targetSource.releaseTarget(target);
        }
        if (setProxyContext) {
            // Restore old proxy.
            AopContext.setCurrentProxy(oldProxy);
        }
    }
}

```



```java
public Object proceed() throws Throwable {
    //	We start with an index of -1 and increment early.
    if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
        return invokeJoinpoint();
    }

    Object interceptorOrInterceptionAdvice =
        this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
    if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
        // Evaluate dynamic method matcher here: static part will already have
        // been evaluated and found to match.
        InterceptorAndDynamicMethodMatcher dm =
            (InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
        Class<?> targetClass = (this.targetClass != null ? this.targetClass : this.method.getDeclaringClass());
        if (dm.methodMatcher.matches(this.method, targetClass, this.arguments)) {
            return dm.interceptor.invoke(this);
        }
        else {
            // Dynamic matching failed.
            // Skip this interceptor and invoke the next in the chain.
            return proceed();
        }
    }
    else {
        // It's an interceptor, so we just invoke it: The pointcut will have
        // been evaluated statically before this object was constructed.
        return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
    }
}
```















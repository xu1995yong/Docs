## SpringAOP基础概念

AOP 即 **面向切面编程**,，通过[预编译](https://baike.baidu.com/item/预编译/3191547)方式和运行期实现在不修改[源代码](https://baike.baidu.com/item/源代码)的情况下动态代理实现程序功能的统一维护的一种技术。它与 `OOP`( `Object-Oriented Programming`, 面向对象编程) 相辅相成,，提供了与 `OOP` 不同的抽象软件结构的视角。在 `OOP` 中,我们以类(class)作为我们的基本单元, 而 `AOP` 中的基本单元是 **Aspect(切面)**。



1. 连接点（joinpoint）：一段代码中的一些具有边界性质的点。Spring只支持方法类型的连接点，即仅能在方法调用前、方法调用后、方法抛出异常时等。
2. 切点（pointcut）：一系列需要增强的方法的集合。
3. 增强（advice）：要织入到目标类切点上的代码，**增强分为前置、后置、异常、最终、环绕增强五类**。
4. 切面（aspect）：切面是切点和增强的组合。
5. 目标对象：增强逻辑的织入目标类。
6. 织入（weave）：将增强添加到目标类的具体连接点上的过程。
7. 引入（introduction）：一种特殊的增强，在不修改代码的前提下，引入可以在**运行期**为类动态地添加一些方法或字段
8. 代理类：类被AOP织入增强后产生的代理类。



## JDK动态代理与CGLib代理

**JDK动态代理**

被代理类的源码是在程序运行期间由JVM根据反射等机制动态生成，其代理对象必须是某个接口的实现。代理类和被代理类的关系是在程序运行时确定。

JDK动态代理需要实现InvocationHandler接口。InvocationHandler是被调用类的调用处理器，每个代理类的实例都关联了一个handler。当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。

JDK动态代理缺点：只能为接口创建代理实例，即要求被代理对象必须存在接口。

**CGLib代理**

CGLib采用底层的字节码技术，可以为被代理类创建子类，而不需要被代理类的接口，然后再子类中采用方法拦截的技术拦截所有父类方法的调用并顺势织入横切逻辑。但是CGLib代理无法增强Final方法





## Spring AOP源码分析

### AutoProxyCreator的注册



### 生成代理对象


AbstractAutoProxyCreator实现了BeanPostProcessor接口，在Bean对象的初始化过程中，会调用该接口的postProcessBeforeInitialization()和postProcessAfterInitialization()方法。在AbstractAutoProxyCreator中重写了postProcessAfterInitialization()方法来实现对Bean的代理。

代理对象的创建流程为：首先获取所有的Advisor，然后判断这些Advisor能否应用于当前Bean，判断依据就是每个Advisor的节点表达式与当前Bean的类型匹配，如果当前Bean没有找到匹配的Advisor，说明当前Bean不需要代理，直接返回当前Bean。

如果找到了匹配的Advisor，则调用AbstractAutoProxyCreator对象的createProxy()方法。在该方法中，首先创建一个proxyFactory对象，然后将匹配的Advisor封装到该proxyFactory对象中，之后则调用proxyFactory对象的getProxy()方法来获取代理对象。

在proxyFactory对象的getProxy()方法中，首先会进行代理方式的选择。判断依据为目标对象是否实现了接口，如果实现了接口，默认会选择JDK动态代理，这种情况下用户也可以强制指定使用CGLIB代理。如果目标对象没有实现接口，则只能使用CGLIB代理。

确定了代理方式后就会调用该代理对象的getProxy()方法创建代理。在JdkDynamicAopProxy对象的getProxy()方法中，首先会获取被代理对象的接口，然后调用Java包中的Proxy类的newProxyInstance()方法来创建代理。



### 执行目标方法

由于JdkDynamicAopProxy类实现了JDK的InvocationHandler接口，所以对于JdkDynamicAopProxy类型的代理对象，在调用目标Bean增强过的方法时，会调用JdkDynamicAopProxy对象的invoke()方法。

在invoke()方法中，首先会获取当前被调用方法的拦截器链，然后对拦截器链进行封装，最后就是按照拦截器的顺序，挨个调用每个拦截器，来对目标方法实施增强。

 



#### AbstractAutowireCapableBeanFactory类

##### applyBeanPostProcessorsBeforeInstantiation()

```java
//AbstractAutowireCapableBeanFactory类
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}
```

#### postProcessBeforeInstantiation()

```java
public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
    Object cacheKey = getCacheKey(beanClass, beanName);
    if (!StringUtils.hasLength(beanName) || !this.targetSourcedBeans.contains(beanName)) {
        if (this.advisedBeans.containsKey(cacheKey)) {
            return null;
        }
        if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
            this.advisedBeans.put(cacheKey, Boolean.FALSE);
            return null;
        }
    }

    // Create proxy here if we have a custom TargetSource.
    // Suppresses unnecessary default instantiation of the target bean:
    // The TargetSource will handle target instances in a custom fashion.
    TargetSource targetSource = getCustomTargetSource(beanClass, beanName);
    if (targetSource != null) {
        if (StringUtils.hasLength(beanName)) {
            this.targetSourcedBeans.add(beanName);
        }
        Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(beanClass, beanName, targetSource);
        Object proxy = createProxy(beanClass, beanName, specificInterceptors, targetSource);
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }
    return null;
}
```



###### applyBeanPostProcessorsAfterInitialization()

```java
//AbstractAutowireCapableBeanFactory
//该方法在bean创建完之后调用
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
            //如果需要，则包装bean
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
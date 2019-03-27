## SpringIOC基础概念

1. 控制反转（IOC）

控制反转是一种设计思想。在传统的Java程序中，直接通过new关键字创建对象，这样导致程序的强耦合性。为了降低耦合，将对象的创建过程控制权交给了容器，由容器进行注入组合对象。而程序只是被动的接受对象

2. 依赖注入（DI)





### BeanFactory 和 ApplicationContext的区别

BeanFactory是Spring中最低层的接口，提供了最简单的容器的功能。BeanFactory使用延迟加载所有的Bean，Bean在获取时才被实例化。

ApplicationContext继承于BeanFactory，并且提供了更多高级功能，比如MessageSource（国际化资源接口）、ResourceLoader（资源加载接口）、ApplicationEventPublisher（应用事件发布接口）等。ApplicationContext在启动时就实例化所有的非延迟加载的Bean。

ApplicationContext的三个实现类：

- ClassPathXmlApplication：把上下文文件当成类路径资源
- FileSystemXmlApplication：从文件系统中的XML文件载入上下文定义信息
- XmlWebApplicationContext：从Web系统中的XML文件载入上下文定义信息





##  ApplicationContext的初始化流程

![img](http://dl.iteye.com/upload/attachment/386364/473a5937-be61-3e4b-b1b2-4a5dd351d10a.jpg)

###  refresh()

```java
//AbstractApplicationContext类
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 刷新上下文前的准备工作
        prepareRefresh();
        // 刷新BeanFactory
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
#### prepareRefresh()

```java
//刷新前的环境准备
protected void prepareRefresh() {
    this.closed.set(false);
    this.active.set(true);
    //初始化属性源，默认空实现
    initPropertySources();
    // 验证属性
    getEnvironment().validateRequiredProperties();
    //
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

#### obtainFreshBeanFactory()

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    //关闭之前的BeanFactory，创建新的BeanFactor，并导入BeanDefinition。该方法默认空实现，
    //并在AbstractRefreshableApplicationContext类重写
    refreshBeanFactory();
    //返回创建的BeanFactory，默认空实现，并在AbstractRefreshableApplicationContext类重写
    return getBeanFactory();
}
```

##### refreshBeanFactory()

```java
//AbstractRefreshableApplicationContext类中重写的refreshBeanFactory()方法
protected final void refreshBeanFactory() throws BeansException {
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    //创建DefaultListableBeanFactory
    DefaultListableBeanFactory beanFactory = createBeanFactory();
    beanFactory.setSerializationId(getId());
    //自定义BeanFactory，包括是否允许BeanDefinition的覆盖及是否允许循环依赖
    customizeBeanFactory(beanFactory);
    //导入BeanDefinition，默认空实现，并在AbstractXmlApplicationContext类中重写
    loadBeanDefinitions(beanFactory);
    synchronized (this.beanFactoryMonitor) {
        this.beanFactory = beanFactory;
    }
}

public final ConfigurableListableBeanFactory getBeanFactory() {
    synchronized (this.beanFactoryMonitor) {
        return this.beanFactory;
    }
}
```

#### prepareBeanFactory()

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // Tell the internal bean factory to use the context's class loader etc.
    beanFactory.setBeanClassLoader(getClassLoader());
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // Configure the bean factory with context callbacks.
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    // BeanFactory interface not registered as resolvable type in a plain factory.
    // MessageSource registered (and found for autowiring) as a bean.
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // Register early post-processor for detecting inner beans as ApplicationListeners.
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    // Detect a LoadTimeWeaver and prepare for weaving, if found.
    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        // Set a temporary ClassLoader for type matching.
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    // Register default environment beans.
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```

#### finishBeanFactoryInitialization()

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // Initialize conversion service for this context.
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
        beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // Register a default embedded value resolver if no bean post-processor
    // (such as a PropertyPlaceholderConfigurer bean) registered any before:
    // at this point, primarily for resolution in annotation attribute values.
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }

    // Stop using the temporary ClassLoader for type matching.
    beanFactory.setTempClassLoader(null);

    // Allow for caching all bean definition metadata, not expecting further changes.
    beanFactory.freezeConfiguration();

    // 调用beanFactory的方法，初始化所有剩余的非懒加载的单实例bean
    beanFactory.preInstantiateSingletons();
}
```

#### finishRefresh()

```java
protected void finishRefresh() {
    // 清除资源缓存
    clearResourceCaches();
    // 初始化生命周期处理器
    initLifecycleProcessor();
    // 
    getLifecycleProcessor().onRefresh();
    // 发布ContextRefreshedEvent事件
    publishEvent(new ContextRefreshedEvent(this));
    LiveBeansView.registerApplicationContext(this);
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


## Bean的获取过程

### getBean()

```java
//AbstractBeanFactory
//根据beanName获取bean，ctx.getBean()就是该方法
public Object getBean(String name) throws BeansException { 
    return doGetBean(name, null, null, false);
}
```
### doGetBean()
```java
protected <T> T doGetBean(String name,Class<T> requiredType,Object[] args, boolean typeCheckOnly) {
    //将传入的name转化为对应的beanName，因为传入的name可能为bean的别名，或者FactoryBean
    final String beanName = transformedBeanName(name);
    //尝试从缓存中获取Bean。如果能获取说明该Bean之前被创建过,所有创建过的单实例Bean都会被缓存起来
    //也是为了解决循环依赖问题
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        //
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }
    else {
        //如果该bean是原型模式的bean且正在创建中，说明存在原型模式的bean的循环依赖，则抛出异常。
        if (isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }
        //如果当前BeanFactory中不包含该BeanDefinition，且存在parentBeanFactory，则到parentBeanFactory中加载该Bean
        BeanFactory parentBeanFactory = getParentBeanFactory();
        if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
            String nameToLookup = originalBeanName(name);
            if (parentBeanFactory instanceof AbstractBeanFactory) {
                return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
                    nameToLookup, requiredType, args, typeCheckOnly);
            }
            else if (args != null) {
                return (T) parentBeanFactory.getBean(nameToLookup, args);
            }
        }
        final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
        checkMergedBeanDefinition(mbd, beanName, args);
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
            //
            bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
        }
    }
    return (T) bean;
}
```
### 从缓存中获取bean

#### getSingleton()

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

#### getObjectForBeanInstance()

```java
//该方法用于判断获得的bean是否是FactoryBean，如果不是FactoryBean的对象，或者用户想要获取的就是工厂bean本身，则直接返回该bean实例。如果是，则调用FactoryBean的getObject()方法来获取FactoryBean产生的bean。
protected Object getObjectForBeanInstance(
    Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {
    //判断name是否以&开头，如果是说明用户想要获取的是工厂bean本身
    if (BeanFactoryUtils.isFactoryDereference(name)) {
        if (beanInstance instanceof NullBean) {
            return beanInstance;
        }
        //此时如果得到的bean实例不是FactoryBean的对象，则抛出异常
        if (!(beanInstance instanceof FactoryBean)) {
            throw new BeanIsNotAFactoryException((name), beanInstance.getClass());
        }
    }
    //如果得到的bean不是FactoryBean的对象，或者用户想要获取的就是工厂bean本身，则直接返回该bean实例
    if (!(beanInstance instanceof FactoryBean) || BeanFactoryUtils.isFactoryDereference(name)) {
        return beanInstance;
    }
    Object object = null;
    if (mbd == null) {
        //尝试从FactoryBean的缓存中获取FactoryBean产生的bean实例
        object = getCachedObjectForFactoryBean(beanName);
    }
    if (object == null) {
        //否则将获取bean的工作交给给getObjectFromFactoryBean()
        FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
        if (mbd == null && containsBeanDefinition(beanName)) {
            mbd = getMergedLocalBeanDefinition(beanName);
        }
        boolean synthetic = (mbd != null && mbd.isSynthetic());
        object = getObjectFromFactoryBean(factory, beanName, !synthetic);
    }
    return object;
}
```



### 创建单例bean

#### getSingleton()

```java
//DefaultSingletonBeanRegistry类
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    synchronized (this.singletonObjects) {
        //尝试从缓存中获取bean
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            //将当前正要加载的bean记录在singletonsCurrentlyInCreation缓冲中
            beforeSingletonCreation(beanName);
            //getObject()直接调用AbstractAutowireCapableBeanFactory类createBean()方法
            singletonObject = singletonFactory.getObject();
            //从singletonsCurrentlyInCreation缓冲中删除该bean
            afterSingletonCreation(beanName);
            newSingleton = true;
            if (newSingleton) {
                //将创建的Bean添加到缓存中singletonObjects,
                //并从earlySingletonObjects缓存、singletonFactories缓存中删除该bean
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}
```

#### 正式创建bean

##### createBean()

```java
//AbstractAutowireCapableBeanFactory类
protected Object createBean(String beanName, RootBeanDefinition mbd,Object[] args){
    RootBeanDefinition mbdToUse = mbd;
    Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
    if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
        mbdToUse = new RootBeanDefinition(mbd);
        mbdToUse.setBeanClass(resolvedClass);
    }
    //方法覆盖
    mbdToUse.prepareMethodOverrides();
    //给 BeanPostProcessors 一个机会来返回代理对象代理真正的实例。是AOP的基础
    Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
    if (bean != null) {
        return bean;
    }
    //
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    return beanInstance;
}
```
##### doCreateBean()

Spring中将Bean的创建过程分为三步，都在AbstractAutowireCapableBeanFactory类的doCreateBean()方法中。

1. createBeanInstance(beanName, mbd, args)：创建bean的实例。
2. populateBean(beanName, mbd, instanceWrapper)：属性填充
3. initializeBean(beanName, exposedObject, mbd)：调用配置文件中指定的init-method初始化对象。

```java
//AbstractAutowireCapableBeanFactory类
protected Object doCreateBean(String beanName , RootBeanDefinition mbd, Object[] args){
    BeanWrapper instanceWrapper = null;
    if (instanceWrapper == null) {
        //利用工厂方法或者利用反射对象的构造器创建出Bean实例。
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    final Object bean = instanceWrapper.getWrappedInstance();
    Class<?> beanType = instanceWrapper.getWrappedClass();
    // 允许PostProcessor修改merged bean definition.
    synchronized (mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            mbd.postProcessed = true;
        }
    }
    // 判断是否需要提早暴露该bean，依据为是否是单例、是否允许循环引用、当前bean是否在创建中
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                      isSingletonCurrentlyInCreation(beanName));
    //如果允许提早暴露该bean，则创建一个匿名ObjectFactory，并使该ObjectFactory的getObject()方法调用getEarlyBeanReference()方法。 并将创建出的bean传入getEarlyBeanReference()方法中。
    if (earlySingletonExposure) {
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
###### createBeanInstance()
```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
    // Make sure bean class is actually resolved at this point.
    Class<?> beanClass = resolveBeanClass(mbd, beanName);

    if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
       
    }

    Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
    if (instanceSupplier != null) {
        return obtainFromSupplier(instanceSupplier, beanName);
    }

    if (mbd.getFactoryMethodName() != null) {
        return instantiateUsingFactoryMethod(beanName, mbd, args);
    }

    boolean resolved = false;
    boolean autowireNecessary = false;
    if (args == null) {
        synchronized (mbd.constructorArgumentLock) {
            if (mbd.resolvedConstructorOrFactoryMethod != null) {
                resolved = true;
                autowireNecessary = mbd.constructorArgumentsResolved;
            }
        }
    }
    if (resolved) {
        if (autowireNecessary) {
            return autowireConstructor(beanName, mbd, null, null);
        }
        else {
            return instantiateBean(beanName, mbd);
        }
    }

    // Candidate constructors for autowiring?
    Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
        mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
        return autowireConstructor(beanName, mbd, ctors, args);
    }

    // Preferred constructors for default construction?
    ctors = mbd.getPreferredConstructors();
    if (ctors != null) {
        return autowireConstructor(beanName, mbd, ctors, null);
    }

    // No special handling: simply use no-arg constructor.
    return instantiateBean(beanName, mbd);
}

```



###### populateBean()
```java
//AbstractAutowireCapableBeanFactory类
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    boolean continueWithPropertyPopulation = true;
    //给InstantiationAwareBeanPostProcessor最后一次机会在属性设置前来改变bean
    //如可以用来支持属性注入的类型
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                    continueWithPropertyPopulation = false;
                    break;
                }
            }
        }
    }
    //BeanPostProcessor使属性注入停止
    if (!continueWithPropertyPopulation) {
        return;
    }

    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

    if (mbd.getResolvedAutowireMode() == AUTOWIRE_BY_NAME || mbd.getResolvedAutowireMode() == AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
        // Add property values based on autowire by name if applicable.
        if (mbd.getResolvedAutowireMode() == AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }
        // Add property values based on autowire by type if applicable.
        if (mbd.getResolvedAutowireMode() == AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }
        pvs = newPvs;
    }
    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}

```
###### initializeBean()
```java
protected Object initializeBean(final String beanName, final Object bean,RootBeanDefinition mbd) {
    invokeAwareMethods(beanName, bean);//调用bean实现的aware接口的方法
    //调用BeanPostProcessor的postProcessBeforeInitialization()方法
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    //调用InitializingBean的afterPropertiesSet()方法
    //之后调用配置文件中<bean>的init-method属性指定的初始化方法
    invokeInitMethods(beanName, wrappedBean, mbd);
    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
    try {
        // 调用BeanPostProcessor的postProcessAfterInitialization()方法
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean
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

///注入属性，初始化



## Spring中循环依赖问题

循环依赖即循环引用，就是两个或以上的bean互相持有对方，最终形成闭环。比如A依赖于B，B依赖于C，C依赖于A。

循环依赖场景有： 构造器的循环依赖 、字段中的循环依赖。**Spring只能解决单例bean的字段中的循环依赖问题。**对于Prototype类型的bean，由于其依赖需要创建新的对象，所以其中的循环依赖是无法解决的。

### Spring中循环依赖的检测

对于所有的bean，由于其在创建之前，都会将beanName加入到相应bean类型的CurrentlyInCreation缓存中，直到bean创建结束删除，所以如果在CurrentlyInCreation缓存中存在该bean，证明产生了循环依赖。

### Spring中解决循环依赖

对于prototype类型的bean的循环依赖，会直接抛出异常。
```java
if (isPrototypeCurrentlyInCreation(beanName)) {
    throw new BeanCurrentlyInCreationException(beanName);
}
```


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

doCreateBean()方法中，在调用createBeanInstance()方法来创建bean的实例后，populateBean()方法进行属性填充之前，会判断是否需要提早暴露该bean，**依据为是否是单例、是否允许循环引用、当前bean是否在创建中**。如果允许提早暴露该bean，则创建一个匿名ObjectFactory，并使该ObjectFactory的getObject()方法调用getEarlyBeanReference()方法。 并将创建出的bean传入getEarlyBeanReference()方法中。如下：

#### doCreateBean()

```java
//AbstractAutowireCapableBeanFactory类的doCreateBean()方法局部

if (instanceWrapper == null) {
    instanceWrapper = createBeanInstance(beanName, mbd, args);
}
//-----------------
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                                  isSingletonCurrentlyInCreation(beanName));
if (earlySingletonExposure) {
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
//-----------------
Object exposedObject = bean;
try {
    populateBean(beanName, mbd, instanceWrapper);
    exposedObject = initializeBean(beanName, exposedObject, mbd);
}
```

#### getEarlyBeanReference()

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
    Object exposedObject = bean;
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = 				(SmartInstantiationAwareBeanPostProcessor) bp;
                exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
            }
        }
    }
    return exposedObject;
}
```

这样当其他bean的创建过程需要依赖该bean时，就需要调用getBean()方法获取该bean，而在获取bean的流程中，会尝试从缓存中获取bean。这样就可以从singletonFactories缓存中获取该bean早期暴露的工厂。从工厂中获取到该bean的早期对象后，就将该bean对象放入earlySingletonObjects缓存中，并从singletonFactories缓存中删除，这样是为了保证单例bean的唯一性。如下：

#### getSingleton()

```java
//在获取bean的流程中，先尝试从缓存中获取bean
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
              //获取bean早期暴露的工厂对象
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    //调用ObjectFactory的子类的getObject()方法，该方法会调用AbstractAutowireCapableBeanFactory对象的getEarlyBeanReference()方法
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

这样其他bean获取了该bean的早期引用后，就可以继续其创建过程，当创建过程结束后，就会从earlySingletonObjects缓存和singletonFactories缓存中删除。

```java
//DefaultSingletonBeanRegistry类
protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        this.singletonObjects.put(beanName, singletonObject);
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}
```







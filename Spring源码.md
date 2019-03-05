 

##  refresh()

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
### finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory)
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
### getBean(String name)
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
### getSingleton(String beanName, ObjectFactory<?> singletonFactory)

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
protected Object doCreateBean(String beanName,RootBeanDefinition mbd,Object[] args){
    BeanWrapper instanceWrapper = null;
    if (instanceWrapper == null) {
        //利用工厂方法或者利用反射对象的构造器创建出Bean实例。
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    final Object bean = instanceWrapper.getWrappedInstance();
    Class<?> beanType = instanceWrapper.getWrappedClass();

    Object exposedObject = bean;
    populateBean(beanName, mbd, instanceWrapper); //调用Bean属性的set()方法赋值
    exposedObject = initializeBean(beanName, exposedObject, mbd);
    //注册bean的destory()方法,包括DisposableBean接口的destroy()和配置文件中<bean>的destroy-method属性指定的销毁方法
    registerDisposableBeanIfNecessary(beanName, bean, mbd);
    return exposedObject;
}
```
```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);
    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}

```
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





### bean的生命周期

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



BeanFactory 和 ApplicationContext的区别















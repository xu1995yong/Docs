##  SpringIOC源码解析

![img](http://dl.iteye.com/upload/attachment/386364/473a5937-be61-3e4b-b1b2-4a5dd351d10a.jpg)



![img](http://dl.iteye.com/upload/attachment/386366/c31e8412-f49d-3be5-a960-5fcc24d35894.jpg)

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















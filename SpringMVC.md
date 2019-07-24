## MVC架构

**MVC模式**把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）。

- 控制器（Controller）- 负责转发请求和请求处理。
- 视图（View） - 展现应用数据、与用户交互。
- 模型（Model） - 封装应用数据，并向外提供接口。





## Servlet



在根容器创建与子容器创建之间，还会创建监听器、过滤器等，完整的加载顺序为;

- ServletContext -> context-param -> listener-> filter -> servlet





## WEB应用的分层



## SpringMVC中的线程安全性

在Springmvc中，Controller都是单例的，也是非线程安全的。这意味着每个request过来，系统都会用原有的instance去处理，这样导致了两个结果：一是我们不用每次创建Controller，二是减少了对象创建和垃圾收集的时间。由于只有一个Controller的实例，当多个线程调用的时候，它里面的成员变量就不是线程安全的了，会发生窜数据的问题。



## Spring MVC中的DispatcherServlet
前端控制器模式（Front Controller Pattern）是用来提供一个集中的请求处理机制，所有的请求都将由一个单一的处理程序处理。

DispatcherServlet提供Spring MVC的集中访问点，而且负责职责的分派，而且与Spring IoC容器无缝集成，从而可以获得Spring的所有好处。DispatcherServlet主要用作职责调度工作，本身主要用于控制流程，主要职责如下：

1、文件上传解析，如果请求类型是multipart将通过MultipartResolver进行文件上传解析；
2、通过HandlerMapping，将请求映射到处理器（返回一个HandlerExecutionChain，它包括一个处理器、多个HandlerInterceptor拦截器）；
3、通过HandlerAdapter支持多种类型的处理器(HandlerExecutionChain中的处理器)；
4、通过ViewResolver解析逻辑视图名到具体视图实现；
5、本地化解析；
6、渲染具体的视图等；
7、如果执行过程中遇到异常将交给HandlerExceptionResolver来解析。





## SpringMVC的初始化

在Spring中，ApplicationContext对象的创建都是由用户执行。但是在WEB应用中，无法手动创建ApplicationContext，这时就需要根据WEB容器的特点，将ApplicationContext对象的创建工作添加到web容器的创建工作中。

### Spring MVC中父容器的初始化

web容器初始化时首先加载web.xml文件，之后会调用ServletContextListener接口的contextInitialized()方法完成Listener的初始化。为了实现SpringMVC的初始化，需要在web.xml中注册ContextLoaderListener，并实现ServletContextListener接口，重写其contextInitialized()方法。ContextLoaderListener类还继承了ContextLoader类。

#### contextInitialized()

```java
//ContextLoaderListener类
public void contextInitialized(ServletContextEvent event) {
    initWebApplicationContext(event.getServletContext());//从Servlet事件中得到ServletContext
}
```
#### initWebApplicationContext()

```java
//ContextLoader类
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
    if (this.context == null) {
        //通过反射创建WebApplicationContext，默认为XmlWebApplicationContext
        this.context = createWebApplicationContext(servletContext);
    }
    if (this.context instanceof ConfigurableWebApplicationContext) {
        ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
        if (!cwac.isActive()) {
            if (cwac.getParent() == null) {
                ApplicationContext parent = loadParentContext(servletContext);
                cwac.setParent(parent);
            }
            //设置IOC容器各个参数，然后通过refresh()方法启动容器的初始化
            configureAndRefreshWebApplicationContext(cwac, servletContext);
        }
    }
    //将根上下文保存到servletContext中
    servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
    //使用ContextClassLoader
    ClassLoader ccl = Thread.currentThread().getContextClassLoader();
    if (ccl == ContextLoader.class.getClassLoader()) {
        currentContext = this.context;
    }
    else if (ccl != null) {
        currentContextPerThread.put(ccl, this.context);
    }
    return this.context;
}
```
#### createWebApplicationContext()

```java
//ContextLoader
protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
    Class<?> contextClass = determineContextClass(sc);  
    //通过反射创建XmlWebApplicationContext
    return (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
}
```
#### configureAndRefreshWebApplicationContext()

```java
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
    if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
        String idParam = sc.getInitParameter(CONTEXT_ID_PARAM);
        if (idParam != null) {
            wac.setId(idParam);
        }
        else {
            // Generate default id...
            wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
                      ObjectUtils.getDisplayString(sc.getContextPath()));
        }
    }
    wac.setServletContext(sc);//将ServletContext保存到该WebApplicationContext中
    //设置配置文件的位置参数。即web.xml中定义的contextConfigLocation
    String configLocationParam = sc.getInitParameter(CONFIG_LOCATION_PARAM);
    if (configLocationParam != null) {
        wac.setConfigLocation(configLocationParam);
    }
    ConfigurableEnvironment env = wac.getEnvironment();
    if (env instanceof ConfigurableWebEnvironment) {
        ((ConfigurableWebEnvironment) env).initPropertySources(sc, null);
    }
    customizeContext(sc, wac);
    wac.refresh();//启动容器的初始化
}

```

### Spring MVC中子容器的初始化

![img](https://img-blog.csdn.net/20160422092959447?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

子容器的初始化的初始化过程涉及三个类：HttpServletBean、FrameworkServlet和DispatcherServlet。DispatcherServlet通过继承FrameworkServlet和HttpServletBean而继承了HttpServlet。

在Servlet容器初始化时会加载web.xml文件，之后会调用Servlet生命周期的init()方法初始化其中注册的Servlet。为完成dispatcherServlet的初始化，需要在web.xml中注册dispatcherServlet。

```xml
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```



#### init()

init()方法在整个系统启动时运行，且只运行一次。init方法中包括对容器（WebApplicationContext）的初始化、组件和外部资源的初始化等等。HttpServletBean中重写了init()方法。

```java
//HttpServletBean
public final void init() throws ServletException {
    //读取web.xml中DispatchServlet定义中的<init-param>，并注入到dispatcherServlet中
    PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
    if (!pvs.isEmpty()) {
        //this即该dispatcherServlet对象
        BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
        ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
        bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
        initBeanWrapper(bw);
        bw.setPropertyValues(pvs, true);
    }
    // 调用子类的initServletBean进行具体的初始化，该方法在FrameworkServlet中实现
    initServletBean();
}
```

#### initServletBean()

```java
//FrameworkServlet
protected final void initServletBean() throws ServletException {
    long startTime = System.currentTimeMillis();
    //初始化子容器WebApplicationContext
    this.webApplicationContext = initWebApplicationContext();
    initFrameworkServlet();//空方法，没有实现
}
```
#### initWebApplicationContext()

```java
//FrameworkServlet
protected WebApplicationContext initWebApplicationContext() {
    //获取ServletContext中保存的父容器WebApplicationContext
    WebApplicationContext rootContext =
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;

    if (this.webApplicationContext != null) {
        // A context instance was injected at construction time -> use it
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                // The context has not yet been refreshed -> provide services such as
                // setting the parent context, setting the application context id, etc
                if (cwac.getParent() == null) {
                    // The context instance was injected without an explicit parent -> set
                    // the root application context (if any; may be null) as the parent
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        // No context instance was injected at construction time -> see if one
        // has been registered in the servlet context. If one exists, it is assumed
        // that the parent context (if any) has already been set and that the
        // user has performed any initialization such as setting the context id
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        // 没有为此servlet定义上下文实例，则创建WebApplicationContext
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        //onRefresh()用于执行各组件的初始化，该方法在DispatchServlet中实现。
        onRefresh(wac);
    }
//将当前上下文存到ServletContext中
    if (this.publishContext) {
        // Publish the context as a servlet context attribute.
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
    }

    return wac;
}
```

#### createWebApplicationContext(WebApplicationContext parent)

```java
//FrameworkServlet
protected WebApplicationContext createWebApplicationContext(WebApplicationContext parent) {
    return createWebApplicationContext((ApplicationContext) parent);
}
protected WebApplicationContext createWebApplicationContext(ApplicationContext parent) {
    //获取XmlWebApplicationContext的Class对象
    Class<?> contextClass = getContextClass();
    //根据获取的Class对象，通过反射创建XmlWebApplicationContext对象
    ConfigurableWebApplicationContext wac =
        (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);

    wac.setEnvironment(getEnvironment());
    wac.setParent(parent);//设置该XmlWebApplicationContext的父容器
    String configLocation = getContextConfigLocation();
    if (configLocation != null) {
        wac.setConfigLocation(configLocation);//设置配置文件的路径
    }
    configureAndRefreshWebApplicationContext(wac);
    return wac;
}
```

#### configureAndRefreshWebApplicationContext()

```java
//FrameworkServlet
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
    if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
        if (this.contextId != null) {
            wac.setId(this.contextId);
        }
        else {
            wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
                      ObjectUtils.getDisplayString(getServletContext().getContextPath()) + "/" + getServletName());
        }
    }

    wac.setServletContext(getServletContext());
    wac.setServletConfig(getServletConfig());
    wac.setNamespace(getNamespace());
    wac.addApplicationListener(new SourceFilteringListener(wac, new ContextRefreshListener()));

    // The wac environment's #initPropertySources will be called in any case when the context
    // is refreshed; do it eagerly here to ensure servlet property sources are in place for
    // use in any post-processing or initialization that occurs below prior to #refresh
    ConfigurableEnvironment env = wac.getEnvironment();
    if (env instanceof ConfigurableWebEnvironment) {
        ((ConfigurableWebEnvironment) env).initPropertySources(getServletContext(), getServletConfig());
    }

    postProcessWebApplicationContext(wac);//空实现
    applyInitializers(wac);
    wac.refresh();
}
```



### SpringMVC中的父子容器

父容器：保存数据源、服务层Service、DAO层、事务的bean。

子容器：保存MVC相关的controller的bean。

![img](https://img-blog.csdn.net/20170324164207194?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSkpfbmFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## SpringMVC请求流程分析

HttpServletBean主要参与容器的创建工作，没有涉及请求处理过程。

请求处理流程都是从Servlet接口的service()方法开始的，该方法在FrameworkServlet中重写。

### service()

```java
//该方法将PATCH类型的请求交给processRequest()方法处理，其他请求交给父类HttpServlet处理
protected void service(HttpServletRequest request, HttpServletResponse response){
    HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
    if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
        processRequest(request, response);
    }
    else {
        super.service(request, response);
    }
}
```

在父类HttpServlet中，根据请求的类型，将不同的请求交给不同的方法处理。例如GET请求交给doGet()方法处理。而在FrameworkServlet中，又重写了不同请求的相应方法，这些方法都将请求交给processRequest(request, response)方法处理。

### doGet()、doPost()

```java
protected final void doGet(HttpServletRequest request, HttpServletResponse response){
    processRequest(request, response);
}
protected final void doPost(HttpServletRequest request, HttpServletResponse response){
    processRequest(request, response);
}
```

### processRequest()

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response){
    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
        doService(request, response);//模板方法模式，该方法在DispatcherServlet类中实现        
    }
    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
        logResult(request, response, failureCause, asyncManager);
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

### doService()

```java
//DispatcherServlet类
protected void doService(HttpServletRequest request, HttpServletResponse response) {
    Map<String, Object> attributesSnapshot = null;
    //判断request是不是include类型的请求。如果是，则对request的attribute做快照备份，方便之后的还原
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }
    //对request设置属性。包括WebApplicationContext、localeResolver、themeResolver等
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());
    //将flashMap加入到request中
    if (this.flashMapManager != null) {
        FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
        if (inputFlashMap != null) {
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
        }
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
    }
    try {
        doDispatch(request, response);//将请求交给doDispatch()方法处理
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Restore the original attribute snapshot, in case of an include.
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
    }
}
```

### doDispatch()

```java
//DispatcherServlet类
protected void doDispatch(HttpServletRequest request, HttpServletResponse response){
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;
        //检查是不是上传请求
        processedRequest = checkMultipart(request);
        multipartRequestParsed = (processedRequest != request);
        // 根据请求调用已加载的HandlerMapping对象的getHandler()方法，获取相应的Handler和拦截器组成
        // HandlerExecutionChain。如果无法找到，则报错返回
        mappedHandler = getHandler(processedRequest);
        if (mappedHandler == null) {
            noHandlerFound(processedRequest, response);
            return;
        }
        //根据handler获取相应的HandlerAdapter
        HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
        //处理GET、HEAD请求的lastModified属性
        String method = request.getMethod();
        boolean isGet = "GET".equals(method);
        if (isGet || "HEAD".equals(method)) {
            long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
            if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                return;
            }
        }
        //执行HandlerExecutionChain中Interceptor拦截器的preHandler()方法
        if (!mappedHandler.applyPreHandle(processedRequest, response)) {
            return;
        }
        //执行HandlerAdapter中的handler处理请求。Controller就是在该处执行。
        mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

        if (asyncManager.isConcurrentHandlingStarted()) {
            return;
        }
        //设置默认View
        applyDefaultViewName(processedRequest, mv);
        //执行HandlerExecutionChain中Interceptor拦截器的postHandler()方法
        mappedHandler.applyPostHandle(processedRequest, response, mv);
        //处理返回结果。包括处理异常、渲染页面、
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // Instead of postHandle and afterCompletion
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            // 删除上传请求的资源.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```

## SpringMVC中的九大组件

- HandlerMapping：根据request查找相应的处理器handler和Interceptors。
- HandlerAdapter：调用handler处理request。
- HandlerExceptionResolver：根据异常设置ModelAndView，之后再交给render方法渲染。
- ViewResolver：
- RequestToViewNameTranslator：
- LOcaleResolver：
- ThemeResolver：
- MultipartResolver：
- FlashMapManager：

## SpringMVC中的HandlerMapping

![HandlerMappingæ ¸å¿ç±»å¾](http://www.51gjie.com/Images/image1/tyu4scq1.x2u.jpg)

### AbstractHandlerMapping

#### getHandler(HttpServletRequest request)

```java
//org.springframework.web.servlet.handler.AbstractHandlerMapping
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    Object handler = getHandlerInternal(request);//由子类实现
    if (handler == null) {
        handler = getDefaultHandler();  //获取默认的handler
    }
    if (handler == null) {
        return null;
    }
    //如果找到的handler是String类型，就以该String为名到SpringMVC的容器中查找该bean
    if (handler instanceof String) { 
        String handlerName = (String) handler;
        handler = obtainApplicationContext().getBean(handlerName);
    }
	//使用找到的handler创建出HandlerExecutionChain类型的变量，然后将adaptedInterceptors和MappedInterceptor添加到HandlerExecutionChain中
    HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);

    if (CorsUtils.isCorsRequest(request)) {
        CorsConfiguration globalConfig = this.corsConfigurationSource.getCorsConfiguration(request);
        CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
        CorsConfiguration config = (globalConfig != null ? globalConfig.combine(handlerConfig) : handlerConfig);
        executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
    }

    return executionChain;
}
```

#### getHandlerExecutionChain() 

```java
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
    HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ?
                                   (HandlerExecutionChain) handler : 
                                   new HandlerExecutionChain(handler));

    String lookupPath = this.urlPathHelper.getLookupPathForRequest(request);
    for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
        if (interceptor instanceof MappedInterceptor) {
            MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
            if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
                chain.addInterceptor(mappedInterceptor.getInterceptor());
            }
        }
        else {
            chain.addInterceptor(interceptor);
        }
    }
    return chain;
}
```

## SpringMVC中的HandlerAdapter
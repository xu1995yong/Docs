## 为什么使用Dubbo

#### 单一应用架构

当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。

#### 垂直应用架构

当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。

#### 分布式服务架构

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。

#### 流动计算架构

当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。

## 完整的RPC同步调用流程

1. 服务消费方（client）以本地调用方式调用服务；
2. client stub（客户端存根）接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体； 
3. client stub找到服务地址，并将消息发送到服务端；
4. server stub收到消息后进行解码； 
5. server stub根据解码结果调用本地的服务； 
6. 本地服务执行并将结果返回给server stub； 
7. server stub将返回结果打包成消息并发送至消费方； 
8. client stub接收到消息，并进行解码； 
9. 服务消费方得到最终结果。



## Dubbo的架构

![dubbo-architucture](http://dubbo.apache.org/docs/zh-cn/user/sources/images/dubbo-architecture.jpg)

### 节点角色说明

| 节点        | 角色说明                               |
| ----------- | -------------------------------------- |
| `Provider`  | 暴露服务的服务提供方                   |
| `Consumer`  | 调用远程服务的服务消费方               |
| `Registry`  | 服务注册与发现的注册中心               |
| `Monitor`   | 统计服务的调用次数和调用时间的监控中心 |
| `Container` | 服务运行容器                           |

### Dubbo启动流程

1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。 向 `/dubbo/XXXService/providers` 目录下写入自己的 URL 地址。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。订阅 `/dubbo/XXXService/providers` 目录下的提供者 URL 地址。并向 `/dubbo/XXXService/consumers` 目录下写入自己的 URL 地址
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 监控中心启动时: 订阅 `/dubbo/XXXService` 目录下的所有提供者和消费者 URL 地址。服务消费者和提供者定时每分钟发送一次统计数据到监控中心。

### Dubbo 架构的特点

#### 连通性

- 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
- 监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
- 服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
- 服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
- 注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
- 注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
- 
- 注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

#### 健壮性

- 监控中心宕掉不影响使用，只是丢失部分采样数据
- 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
- 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
- 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯，或者通过直连的方式通信
- 服务提供者无状态，任意一台宕掉后，不影响使用
- 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

#### 伸缩性

- 注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
- 服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者



## Dubbo中的协议

### dubbo协议

Dubbo 缺省协议采用单一长连接和 NIO 异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。但Dubbo 缺省协议不适合传送大数据量的服务，比如传文件，传视频等。

**特性**

- 连接个数：单连接
- 连接方式：长连接
- 传输协议：TCP
- 传输方式：NIO 异步传输
- 序列化：Hessian 二进制序列化
- 适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用 dubbo 协议传输大文件或超大字符串。
- 适用场景：常规远程服务方法调用



### http协议

基于 HTTP 表单的远程调用协议，采用 Spring 的 HttpInvoker 实现。

**特性**

- 连接个数：多连接
- 连接方式：短连接
- 传输协议：HTTP
- 传输方式：同步传输
- 序列化：表单序列化
- 适用范围：传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或URL传入参数，暂不支持传文件。
- 适用场景：需同时给应用程序和浏览器 JS 使用的服务。



### rmi协议

RMI 协议采用 JDK 标准的 `java.rmi.*` 实现，采用阻塞式短连接和 JDK 标准序列化方式。

注意：如果正在使用 RMI 提供服务给外部访问 [[1\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/rmi.html#fn1)，同时应用里依赖了老的 common-collections 包 [[2\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/rmi.html#fn2) 的情况下，存在反序列化安全风险 [[3\]](http://dubbo.apache.org/zh-cn/docs/user/references/protocol/rmi.html#fn3)。

**特性**

- 连接个数：多连接
- 连接方式：短连接
- 传输协议：TCP
- 传输方式：同步传输
- 序列化：Java 标准二进制序列化
- 适用范围：传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。
- 适用场景：常规远程服务方法调用，与原生RMI服务互操作







## Dubbo中的注册中心

注册中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表

### Zookeeper注册中心

支持以下功能：

- 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息
- 当注册中心重启时，能自动恢复注册数据，以及订阅请求
- 当会话过期时，能自动恢复注册数据，以及订阅请求
- 当设置 `<dubbo:registry check="false" />` 时，记录失败注册和订阅请求，后台定时重试
- 可通过 `<dubbo:registry username="admin" password="1234" />` 设置 zookeeper 登录信息
- 可通过 `<dubbo:registry group="dubbo" />` 设置 zookeeper 的根节点，不设置将使用无根树
- 支持 `*` 号通配符 `<dubbo:reference group="*" version="*" />`，可订阅服务的所有分组和所有版本的提供者







## Dubbo中的负载均衡

### 基于权重的随机负载均衡（Random ）

- 按权重设置随机概率。
- 在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

### 基于轮询的负载均衡（RoundRobin ）

- 按公约后的权重设置轮询比率。
- 存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。

### 基于最少活跃数的负载均衡（LeastActive ）

- **最少活跃调用**，相同活跃数的随机，活跃数指调用前后计数差。
- 越快的提供者收到的请求越多，越慢的提供者收到的请求越少，因为越慢的提供者的调用前后计数差会越大。

### 基于一致性 Hash的负载均衡（ConsistentHash ）

- 相同参数的请求总是发到同一提供者。
- 当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。
- 算法参见：<http://en.wikipedia.org/wiki/Consistent_hashing>
- 缺省只对第一个参数 Hash，如果要修改，请配置 `<dubbo:parameter key="hash.arguments" value="0,1" />`
- 缺省用 160 份虚拟节点，如果要修改，请配置 `<dubbo:parameter key="hash.nodes" value="320" />`



## Dubbo中的集群容错

在集群调用失败时，Dubbo 提供了多种容错方案。

**Failover Cluster**

失败自动切换，当出现失败，重试其它服务器 [[1\]](https://dubbo.incubator.apache.org/zh-cn/docs/user/demos/fault-tolerent-strategy.html#fn1)。通常用于读操作，但重试会带来更长延迟。可通过 `retries="2"` 来设置重试次数(不含第一次)。

**Failfast Cluster**

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

**Failsafe Cluster**

失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

**Failback Cluster**

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

**Forking Cluster**

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 `forks="2"` 来设置最大并行数。

**Broadcast Cluster**

广播调用所有提供者，逐个调用，任意一台报错则报错 [[2\]](https://dubbo.incubator.apache.org/zh-cn/docs/user/demos/fault-tolerent-strategy.html#fn2)。通常用于通知所有提供者更新缓存或日志等本地资源信息。







## Dubbo服务暴露过程

首先在Spring容器启动时，XML文件中定义的<dubbo:service />标签会解析成ServiceBean对象，ServiceBean类实现了InitializingBean接口，在ServiceBean的初始化过程中，会调用InitializingBean接口的afterPropertiesSet()方法，来完成自身的初始化过程。

ServiceBean类还实现了ApplicationListener接口并重写了onApplicationEvent()方法来监听ContextRefreshedEvent。当Spring容器初始化完成之后，会回调ServiceBean对象的onApplicationEvent()方法。

在onApplicationEvent()方法中，如果判断当前ServiceBean没有暴露过，并且可以暴露，就会执行export()方法。在exprt()方法中又调用父类ServiceConfig的export()方法执行暴露过程。

在ServiceConfig类的doExportUrls()方法中，首先会导入注册中心的url信息，之后遍历xml文件中该ServiceBean指定的所有协议，并对每个协议都执行服务暴露。

在针对每个协议执行服务暴露时，首先会根据scope的值，进行服务暴露。默认scopse为null，即同时执行本地暴露和远程暴露。在执行远程暴露过程中，首先会根据当前ServiceBean的具体实现对象ref、服务的接口以及registryUrl创建一个Invoker，然后调用Protocol类的export()方法来暴露服务。

首先调用RegistryProtocol类的export方法，在该export()方法中又首先调用DubboProtocol类的export()方法。DubboProtocol类的export()方法中会创建Netty服务器，并监听Dubbo协议指定的端口。之后返回到RegistryProtocol类的export方法，该方法会将当前服务的url注册到zookeeper注册中心。这样就完成了整个的服务暴露过程。



### ServiceBean类

#### onApplicationEvent()

```java
//ServiceBean类重写了ApplicationListener接口的onApplicationEvent()方法
public void onApplicationEvent(ContextRefreshedEvent event) {
    if (!isExported() && !isUnexported()) {
        export();
    }
}
```

#### export()

```java
public void export() {
    super.export();//调用父类ServiceConfig的export()方法
    publishExportEvent();
}
```

### ServiceConfig类

####  export()

```java
//ServiceConfig中的export()方法
public synchronized void export() {
    if (delay != null && delay > 0) {
        delayExportExecutor.schedule(this::doExport, delay, TimeUnit.MILLISECONDS);
    } else {
        doExport();
    }
}
```

####  doExport()

```java
protected synchronized void doExport() {
    exported = true;
    if (path == null || path.length() == 0) {
        path = interfaceName;
    }
    ProviderModel providerModel = new ProviderModel(getUniqueServiceName(), ref, interfaceClass);
    ApplicationModel.initProviderModel(getUniqueServiceName(), providerModel);
    doExportUrls();
}
```

#### doExportUrls()

 ```java
private void doExportUrls() {
    List<URL> registryURLs = loadRegistries(true);//导入注册中心的url信息
    //遍历xml文件中该服务指定的所有协议，执行服务暴露
    for (ProtocolConfig protocolConfig : protocols) {
        doExportUrlsFor1Protocol(protocolConfig, registryURLs);
    }
}
 ```

#### doExportUrlsFor1Protocol()

```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    String name = protocolConfig.getName(); //默认采用Dubbo协议
    if (name == null || name.length() == 0) {
        name = Constants.DUBBO;
    }
    //省略一系列参数设置
    //省略

    //根据参数生成url
    URL url = new URL(name, host, port, (contextPath == null || contextPath.length() == 0 ? "" : contextPath + "/") + path, map);

    if (ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
        .hasExtension(url.getProtocol())) {
        url = ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class)
            .getExtension(url.getProtocol()).getConfigurator(url).configure(url);
    }
    //根据scope的值，进行服务暴露。默认scopse为null，即同时执行本地暴露和远程暴露。
    String scope = url.getParameter(Constants.SCOPE_KEY);
    //如果scope配置为none则不暴露，只有scope的值不等于none时，才执行暴露
    if (!Constants.SCOPE_NONE.equalsIgnoreCase(scope)) {
        //当scope的值不等于remote时，执行本地暴露
        if (!Constants.SCOPE_REMOTE.equalsIgnoreCase(scope)) {
            exportLocal(url);
        }
        //当scope的值不等于local时，执行远程暴露
        if (!Constants.SCOPE_LOCAL.equalsIgnoreCase(scope)) {
            if (registryURLs != null && !registryURLs.isEmpty()) {
                for (URL registryURL : registryURLs) {
                    url = url.addParameterIfAbsent(Constants.DYNAMIC_KEY, registryURL.getParameter(Constants.DYNAMIC_KEY));

                    // For providers, this is used to enable custom proxy to generate invoker
                    String proxy = url.getParameter(Constants.PROXY_KEY);
                    if (StringUtils.isNotEmpty(proxy)) {
                        registryURL = registryURL.addParameter(Constants.PROXY_KEY, proxy);
                    }
                    //根据服务的具体实现对象ref、服务的接口以及registryUrl获取Invoker
                    //invoker是对服务的具体实现的代理
                    Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(Constants.EXPORT_KEY, url.toFullString()));
                    DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
                    //调用Protocol生成的适配类的export方法来暴露服务
                    Exporter<?> exporter = protocol.export(wrapperInvoker);
                    exporters.add(exporter);
                }
            } else {
                Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, url);
                DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);

                Exporter<?> exporter = protocol.export(wrapperInvoker);
                exporters.add(exporter);
            }

            MetadataReportService metadataReportService = null;
            if ((metadataReportService = getMetadataReportService()) != null) {
                metadataReportService.publishProvider(url);
            }
        }
    }
    this.urls.add(url);
}
```

### RegistryProtocol类

#### export()

```java
//RegistryProtocol类
public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
    URL registryUrl = getRegistryUrl(originInvoker);
    URL providerUrl = getProviderUrl(originInvoker);

    final URL overrideSubscribeUrl = getSubscribedOverrideUrl(providerUrl);
    final OverrideListener overrideSubscribeListener = new OverrideListener(overrideSubscribeUrl, originInvoker);
    overrideListeners.put(overrideSubscribeUrl, overrideSubscribeListener);

    providerUrl = overrideUrlWithConfig(providerUrl, overrideSubscribeListener);
    //暴露服务
    final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);
    final Registry registry = getRegistry(originInvoker);
    final URL registeredProviderUrl = getRegisteredProviderUrl(providerUrl, registryUrl);
    //
    ProviderInvokerWrapper<T> providerInvokerWrapper = ProviderConsumerRegTable.registerProvider(originInvoker,registryUrl, registeredProviderUrl);
    //to judge if we need to delay publish
    boolean register = registeredProviderUrl.getParameter("register", true);
    if (register) {
        //将服务的url注册到zookeeper注册中心
        register(registryUrl, registeredProviderUrl);
        providerInvokerWrapper.setReg(true);
    }
    registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);
    exporter.setRegisterUrl(registeredProviderUrl);
    exporter.setSubscribeUrl(overrideSubscribeUrl);
    return new DestroyableExporter<>(exporter);
}
```

#### doLocalExport()

```java
private <T> ExporterChangeableWrapper<T> doLocalExport(final Invoker<T> originInvoker, URL providerUrl) {
    String key = getCacheKey(originInvoker);
    ExporterChangeableWrapper<T> exporter = (ExporterChangeableWrapper<T>) bounds.get(key);
    if (exporter == null) {
        synchronized (bounds) {
            exporter = (ExporterChangeableWrapper<T>) bounds.get(key);
            if (exporter == null) {
                final Invoker<?> invokerDelegete = new InvokerDelegate<T>(originInvoker, providerUrl);
                exporter = new ExporterChangeableWrapper<T>((Exporter<T>) protocol.export(invokerDelegete), originInvoker);//调用DubboProtocol的export()方法
                bounds.put(key, exporter);
            }
        }
    }
    return exporter;
}
```

### DubboProtocol类

#### export()

```java
//DubboProtocol类
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    URL url = invoker.getUrl();
    String key = serviceKey(url);
    DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
    exporterMap.put(key, exporter);
    //export an stub service for dispatching event
    Boolean isStubSupportEvent = url.getParameter(Constants.STUB_EVENT_KEY, Constants.DEFAULT_STUB_EVENT);
    Boolean isCallbackservice = url.getParameter(Constants.IS_CALLBACK_SERVICE, false);
    if (isStubSupportEvent && !isCallbackservice) {
        String stubServiceMethods = url.getParameter(Constants.STUB_EVENT_METHODS_KEY);
        if (stubServiceMethods == null || stubServiceMethods.length() == 0) {

        } else {
            stubServiceMethodsMap.put(url.getServiceKey(), stubServiceMethods);
        }
    }
    openServer(url);//创建Netty服务器，并监听指定的端口
    optimizeSerialization(url);
    return exporter;
}
```



## Dubbo服务引用过程

在消费端的Spring容器初始化时，会将XML文件中定义的 `<dubbo:reference/>`标签解析为ReferenceBean类的对象。而ReferenceBean类实现了FactoryBean接口，并重写了其getObject()方法，即ReferenceBean是一个FactoryBean。当在Spring容器中获取以`<dubbo:reference/>` 标签的id属性为名称的bean时，就会调用ReferenceBean类重写的getObject()方法返回该FactoryBean创建的对象。

在ReferenceBean类的getObject()方法中，调用父类ReferenceConfig类的get()方法来获取指定的Bean。在get()方法中会调用init()方法。在init()方法中，首先会进行相应的配置解析比如当前应用名、注册中心地址、当前服务的接口名及接口内的方法名、服务消费者 ip 地址等，并将配置存储到 Hashmap 中，之后会调用createProxy()方法并传入存储了相应配置的map来创建代理对象。

在createProxy()方法中，首先会根据 url 的协议、scope值以及 injvm 等参数检测是否是本地引用，如果不是则执行远程引用过程。远程引用时会先判断是否是点对点接连，如果不是点对点连接，则加载注册中心的url地址，之后会调用RegistryProtocol的 refer()方法。

在RegistryProtocol的refer()方法中，首先会根据url获取注册中心实例，之后在注册中心的 consumers 目录下注册自己，然后订阅注册中心中的 providers 节点下的信息。

之后会调用DubboProtocol的refer()方法，在该方法 中，会建立与当前服务的提供者的连接。

最后调用 ProxyFactory 的getProxy()方法并传入刚创建的Invoker实例来生成最终的代理类，

Init()方法调用完成后，初始化过程结束，get()方法就将创建出来的代理对象返回。





### ReferenceBean类

#### getObject()

```java
//ReferenceBean类
public Object getObject() {
    return get();//调用父类ReferenceConfig类的get()方法
}
```

### ReferenceConfig类

#### get()

```java
//ReferenceConfig类
public synchronized T get() {
    checkAndUpdateSubConfigs();
    if (destroyed) {
        throw new IllegalStateException("Already destroyed!");
    }
    if (ref == null) {
        init();
    }
    return ref;
}
```

#### init()

```java
//ReferenceConfig类
private void init() {
    initialized = true;
    checkStubAndLocal(interfaceClass);
    checkMock(interfaceClass);
 
    //参数解析
    Map<String, Object> attributes = null;
    if (methods != null && !methods.isEmpty()) {
        attributes = new HashMap<String, Object>();
        for (MethodConfig methodConfig : methods) {
            appendParameters(map, methodConfig, methodConfig.getName());
            String retryKey = methodConfig.getName() + ".retry";
            if (map.containsKey(retryKey)) {
                String retryValue = map.remove(retryKey);
                if ("false".equals(retryValue)) {
                    map.put(methodConfig.getName() + ".retries", "0");
                }
            }
            attributes.put(methodConfig.getName(), convertMethodConfig2AyncInfo(methodConfig));
        }
    }
    //获取服务消费者 ip 地址
    String hostToRegistry = ConfigUtils.getSystemProperty(Constants.DUBBO_IP_TO_REGISTRY);
    if (hostToRegistry == null || hostToRegistry.length() == 0) {
        hostToRegistry = NetUtils.getLocalHost();
    } 
    map.put(Constants.REGISTER_IP_KEY, hostToRegistry);
    //
    ref = createProxy(map);

    ConsumerModel consumerModel = new ConsumerModel(getUniqueServiceName(), interfaceClass, ref, interfaceClass.getMethods(), attributes);
    ApplicationModel.initConsumerModel(getUniqueServiceName(), consumerModel);
}
```

#### createProxy()

```java
private T createProxy(Map<String, String> map) {
    URL tmpUrl = new URL("temp", "localhost", 0, map);
    final boolean isJvmRefer;
    //根据 url 的协议、scope 以及 injvm 等参数检测是否需要本地引用
    if (isInjvm() == null) {
        if (url != null && url.length() > 0) {  
            isJvmRefer = false;
        } else {
            isJvmRefer = InjvmProtocol.getInjvmProtocol().isInjvmRefer(tmpUrl);
        }
    } else {
        isJvmRefer = isInjvm();
    }
    if (isJvmRefer) {
        URL url = new URL(Constants.LOCAL_PROTOCOL, NetUtils.LOCALHOST, 0, interfaceClass.getName()).addParameters(map);
        invoker = refprotocol.refer(interfaceClass, url);
    } else {
        if (url != null && url.length() > 0) { // user specified URL, could be peer-to-peer address, or register center's address.
            String[] us = Constants.SEMICOLON_SPLIT_PATTERN.split(url);
            if (us != null && us.length > 0) {
                for (String u : us) {
                    URL url = URL.valueOf(u);
                    if (url.getPath() == null || url.getPath().length() == 0) {
                        url = url.setPath(interfaceName);
                    }
                    if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
                        urls.add(url.addParameterAndEncoded(Constants.REFER_KEY, StringUtils.toQueryString(map)));
                    } else {
                        urls.add(ClusterUtils.mergeUrl(url, map));
                    }
                }
            }
        } else { // assemble URL from register center's configuration
            checkRegistry();
            List<URL> us = loadRegistries(false);
            if (us != null && !us.isEmpty()) {
                for (URL u : us) {
                    URL monitorUrl = loadMonitor(u);
                    if (monitorUrl != null) {
                        map.put(Constants.MONITOR_KEY, URL.encode(monitorUrl.toFullString()));
                    }
                    urls.add(u.addParameterAndEncoded(Constants.REFER_KEY, StringUtils.toQueryString(map)));
                }
            }

        }

        if (urls.size() == 1) {
            invoker = refprotocol.refer(interfaceClass, urls.get(0));
        } else {
            List<Invoker<?>> invokers = new ArrayList<Invoker<?>>();
            URL registryURL = null;
            for (URL url : urls) {
                invokers.add(refprotocol.refer(interfaceClass, url));
                if (Constants.REGISTRY_PROTOCOL.equals(url.getProtocol())) {
                    registryURL = url; // use last registry url
                }
            }
            if (registryURL != null) { // registry url is available
                // use RegistryAwareCluster only when register's cluster is available
                URL u = registryURL.addParameter(Constants.CLUSTER_KEY, RegistryAwareCluster.NAME);
                // The invoker wrap relation would be: RegistryAwareClusterInvoker(StaticDirectory) -> FailoverClusterInvoker(RegistryDirectory, will execute route) -> Invoker
                invoker = cluster.join(new StaticDirectory(u, invokers));
            } else { // not a registry url, must be direct invoke.
                invoker = cluster.join(new StaticDirectory(invokers));
            }
        }
    }

    Boolean c = check;
    if (c == null && consumer != null) {
        c = consumer.isCheck();
    }
    if (c == null) {
        c = true; // default true
    }
    if (c && !invoker.isAvailable()) {
        // make it possible for consumer to retry later if provider is temporarily unavailable
        initialized = false;

    }

    MetadataReportService metadataReportService = null;
    if ((metadataReportService = getMetadataReportService()) != null) {
        URL consumerURL = new URL(Constants.CONSUMER_PROTOCOL, map.remove(Constants.REGISTER_IP_KEY), 0, map.get(Constants.INTERFACE_KEY), map);
        metadataReportService.publishConsumer(consumerURL);
    }
    // create service proxy
    return (T) proxyFactory.getProxy(invoker);
}
```

### RegistryProtocol类

#### refer()

```java
//RegistryProtocol
public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
    url = url.setProtocol(url.getParameter(REGISTRY_KEY, DEFAULT_REGISTRY)).removeParameter(REGISTRY_KEY);
    //根据url获取注册中心对象
    Registry registry = registryFactory.getRegistry(url);
    if (RegistryService.class.equals(type)) {
        return proxyFactory.getInvoker((T) registry, type, url);
    }

    // group="a,b" or group="*"
    Map<String, String> qs = StringUtils.parseQueryString(url.getParameterAndDecoded(REFER_KEY));
    String group = qs.get(Constants.GROUP_KEY);
    if (group != null && group.length() > 0) {
        if ((COMMA_SPLIT_PATTERN.split(group)).length > 1 || "*".equals(group)) {
            return doRefer(getMergeableCluster(), registry, type, url);
        }
    }
    return doRefer(cluster, registry, type, url);
}
```

#### doRefer()

```java
private <T> Invoker<T> doRefer(Cluster cluster, Registry registry, Class<T> type, URL url) {
    RegistryDirectory<T> directory = new RegistryDirectory<T>(type, url);
    directory.setRegistry(registry);
    directory.setProtocol(protocol);
    // all attributes of REFER_KEY
    Map<String, String> parameters = new HashMap(directory.getUrl().getParameters());
    URL subscribeUrl = new URL(CONSUMER_PROTOCOL, parameters.remove(REGISTER_IP_KEY), 0, type.getName(), parameters);
    //向注册中心注册自己
    if (!ANY_VALUE.equals(url.getServiceInterface()) && url.getParameter(REGISTER_KEY, true)) {
        registry.register(getRegisteredConsumerUrl(subscribeUrl, url));
    }
    directory.buildRouterChain(subscribeUrl);
    //从注册中心订阅服务
    directory.subscribe(subscribeUrl.addParameter(CATEGORY_KEY,PROVIDERS_CATEGORY + "," + CONFIGURATORS_CATEGORY + "," + ROUTERS_CATEGORY));

    Invoker invoker = cluster.join(directory);
    ProviderConsumerRegTable.registerConsumer(invoker, url, subscribeUrl, directory);
    return invoker;
}
```

### DubboProtocol类

#### refer()

```java
//DubboProtocol类
public <T> Invoker<T> refer(Class<T> serviceType, URL url) throws RpcException {
    optimizeSerialization(url);
    // create rpc invoker.
    DubboInvoker<T> invoker = new DubboInvoker<T>(serviceType, url, getClients(url), invokers);
    invokers.add(invoker);
    return invoker;
}
```

#### getClients()

```java
private ExchangeClient[] getClients(URL url) {
    // whether to share connection
    boolean service_share_connect = false;
    int connections = url.getParameter(Constants.CONNECTIONS_KEY, 0);
    // if not configured, connection is shared, otherwise, one connection for one service
    if (connections == 0) {
        service_share_connect = true;
        connections = 1;
    }

    ExchangeClient[] clients = new ExchangeClient[connections];
    for (int i = 0; i < clients.length; i++) {
        if (service_share_connect) {
            clients[i] = getSharedClient(url);
        } else {
            clients[i] = initClient(url);
        }
    }
    return clients;
}
```



## Dubbo服务调用过程

从Spring容器中获取的ConsumerBean实际上是一个代理对象。该代理对象是根据Dubbo配置文件的设置，对Invoker层层封装的结果。Invoker 是 Dubbo 的核心模型，代表一个可执行体。在服务消费方，Invoker 用于执行远程调用。

当调用该ConsumerBean的某个方法时，会调用 InvocationHandler 接口的实现类 InvokerInvocationHandler 的 invoke()方法。InvokerInvocationHandler 中封装了Invoker类型的成员变量，比如MockClusterInvoker，MockClusterInvoker 内部封装了服务降级逻辑。所以在InvokerInvocationHandler 对象的invoke()方法中又调用了MockClusterInvoker 对象的invoke()方法。

MockClusterInvoker对象中又封装了Invoker类型的成员变量，比如 FailoverClusterInvoker，所以在MockClusterInvoker 对象的invoke()方法中会调用FailoverClusterInvoker对象的invoke()方法。该方法在其父类AbstractClusterInvoker类中定义。该invoke()方法会获取所有可用的Invoker，并初始化负载均衡策略，之后会调用其doInvoke()方法，该方法在FailoverClusterInvoker类中重写。

在FailoverClusterInvoker类的doInvoke()方法中，主要封装了失败重试的逻辑。在该方法中，首先会获取配置文件中的失败重试次数，然后在一个for循环当中失败重试。在for循环中首先会通过负载均衡机制选择一个Invoker，然后调用该Invoker 的 invoke() 方法。

如果采用Dubbo协议，最终该调用会转到DubboInvoker对象的invoke()方法，该方法在其父类AbstractInvoker类中定义。在该invoke()方法中会设置一些参数，之后调用AbstractInvoker类的doInvoke()方法。该方法在DubboInvoker中重写。

在DubboInvoker的doInvoke()方法中，首先会选择一个服务引用过程中已经创建好的客户端，然后判断该次RPC调用的类型，比如Oneway、Async、Sync。

- 如果是Oneway类型的调用，则调用已选择的客户端的request()方法发送请求，并返回一个空结果。
- 如果是异步调用则调用已选择的客户端的request()方法发送请求，并暂时返回一个空结果。
- 如果是同步调用，就调用已选择的客户端的request()方法将请求发送出去。之后将服务提供者返回的数据封装成ResponseFuture类型的对象，并调用该对象的get()方法返回调用的值。



### InvokerInvocationHandler类

```java
private final Invoker<?> invoker;

public InvokerInvocationHandler(Invoker<?> handler) {
    this.invoker = handler;
}

@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    String methodName = method.getName();
    Class<?>[] parameterTypes = method.getParameterTypes();
    if (method.getDeclaringClass() == Object.class) {
        return method.invoke(invoker, args);
    }
    if ("toString".equals(methodName) && parameterTypes.length == 0) {
        return invoker.toString();
    }
    if ("hashCode".equals(methodName) && parameterTypes.length == 0) {
        return invoker.hashCode();
    }
    if ("equals".equals(methodName) && parameterTypes.length == 1) {
        return invoker.equals(args[0]);
    }

    return invoker.invoke(createInvocation(method, args)).recreate();
}

private RpcInvocation createInvocation(Method method, Object[] args) {
    RpcInvocation invocation = new RpcInvocation(method, args);
    if (RpcUtils.hasFutureReturnType(method)) {
        invocation.setAttachment(Constants.FUTURE_RETURNTYPE_KEY, "true");
        invocation.setAttachment(Constants.ASYNC_KEY, "true");
    }
    return invocation;
}

```



```java
public Result invoke(Invocation invocation) throws RpcException {
    Result result = null;

    String value = directory.getUrl().getMethodParameter(invocation.getMethodName(), Constants.MOCK_KEY, Boolean.FALSE.toString()).trim();
    if (value.length() == 0 || value.equalsIgnoreCase("false")) {
        //no mock
        result = this.invoker.invoke(invocation);
    } else if (value.startsWith("force")) {

        //force:direct mock
        result = doMockInvoke(invocation, null);
    } else {
        //fail-mock
        try {
            result = this.invoker.invoke(invocation);
        } catch (RpcException e) {
            if (e.isBiz()) {
                throw e;
            }


            result = doMockInvoke(invocation, e);
        }
    }
    return result;
}
```

### AbstractClusterInvoker类

#### invoke()

```java
//AbstractClusterInvoker类
public Result invoke(final Invocation invocation) throws RpcException {
    checkWhetherDestroyed();

    // binding attachments into invocation.
    Map<String, String> contextAttachments = RpcContext.getContext().getAttachments();
    if (contextAttachments != null && contextAttachments.size() != 0) {
        ((RpcInvocation) invocation).addAttachments(contextAttachments);
    }

    List<Invoker<T>> invokers = list(invocation);
    LoadBalance loadbalance = initLoadBalance(invokers, invocation);
    RpcUtils.attachInvocationIdIfAsync(getUrl(), invocation);
    return doInvoke(invocation, invokers, loadbalance);
}
```





### FailoverClusterInvoker类

#### doInvoke()

```java
//FailoverClusterInvoker类
public Result doInvoke(Invocation invocation, final List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
    List<Invoker<T>> copyInvokers = invokers;
    checkInvokers(copyInvokers, invocation);
    String methodName = RpcUtils.getMethodName(invocation);
    //获取重试次数
    int len = getUrl().getMethodParameter(methodName, Constants.RETRIES_KEY, Constants.DEFAULT_RETRIES) + 1;
    if (len <= 0) {
        len = 1;
    }
    RpcException le = null; // last exception.
    List<Invoker<T>> invoked = new ArrayList<Invoker<T>>(copyInvokers.size()); // invoked invokers.
    Set<String> providers = new HashSet<String>(len);
    for (int i = 0; i < len; i++) { //在一个for循环当中，失败重试
        if (i > 0) {
            checkWhetherDestroyed();
            // 在进行重试前重新列举 Invoker，这样做的好处是，如果某个服务挂了，
            // 通过调用 list 可得到最新可用的 Invoker 列表
            copyInvokers = list(invocation);
            checkInvokers(copyInvokers, invocation);
        }
        //通过负载均衡机制，选择一个Invoker
        Invoker<T> invoker = select(loadbalance, invocation, copyInvokers, invoked);
        invoked.add(invoker);
        RpcContext.getContext().setInvokers((List) invoked);
        try {
            Result result = invoker.invoke(invocation);
            return result;
        } catch (RpcException e) {
            if (e.isBiz()) { // biz exception.
                throw e;
            }
            le = e;
        } catch (Throwable e) {
            le = new RpcException(e.getMessage(), e);
        } finally {
            providers.add(invoker.getUrl().getAddress());
        }
    }
}

```





```java
//AbstractInvoker类
public Result invoke(Invocation inv) throws RpcException {
    // if invoker is destroyed due to address refresh from registry, let's allow the current invoke to proceed
    RpcInvocation invocation = (RpcInvocation) inv;
    invocation.setInvoker(this);
    if (attachment != null && attachment.size() > 0) {
        invocation.addAttachmentsIfAbsent(attachment);
    }
    Map<String, String> contextAttachments = RpcContext.getContext().getAttachments();
    if (contextAttachments != null && contextAttachments.size() != 0) {
        /**
             * invocation.addAttachmentsIfAbsent(context){@link RpcInvocation#addAttachmentsIfAbsent(Map)}should not be used here,
             * because the {@link RpcContext#setAttachment(String, String)} is passed in the Filter when the call is triggered
             * by the built-in retry mechanism of the Dubbo. The attachment to update RpcContext will no longer work, which is
             * a mistake in most cases (for example, through Filter to RpcContext output traceId and spanId and other information).
             */
        invocation.addAttachments(contextAttachments);
    }
    if (getUrl().getMethodParameter(invocation.getMethodName(), Constants.ASYNC_KEY, false)) {
        invocation.setAttachment(Constants.ASYNC_KEY, Boolean.TRUE.toString());
    }
    RpcUtils.attachInvocationIdIfAsync(getUrl(), invocation);


    try {
        return doInvoke(invocation);
    } catch (InvocationTargetException e) { // biz exception
        Throwable te = e.getTargetException();
        if (te == null) {
            return new RpcResult(e);
        } else {
            if (te instanceof RpcException) {
                ((RpcException) te).setCode(RpcException.BIZ_EXCEPTION);
            }
            return new RpcResult(te);
        }
    } catch (RpcException e) {
        if (e.isBiz()) {
            return new RpcResult(e);
        } else {
            throw e;
        }
    } catch (Throwable e) {
        return new RpcResult(e);
    }
}
```



```java
//DubboInvoker类
protected Result doInvoke(final Invocation invocation) throws Throwable {
    RpcInvocation inv = (RpcInvocation) invocation;
    final String methodName = RpcUtils.getMethodName(invocation);
    inv.setAttachment(Constants.PATH_KEY, getUrl().getPath());
    inv.setAttachment(Constants.VERSION_KEY, version);

    ExchangeClient currentClient;
    //选择一个客户端
    if (clients.length == 1) {
        currentClient = clients[0];
    } else {
        currentClient = clients[index.getAndIncrement() % clients.length];
    }
    try {
        boolean isAsync = RpcUtils.isAsync(getUrl(), invocation);
        boolean isAsyncFuture = RpcUtils.isReturnTypeFuture(inv);
        boolean isOneway = RpcUtils.isOneway(getUrl(), invocation);
        int timeout = getUrl().getMethodParameter(methodName, Constants.TIMEOUT_KEY, Constants.DEFAULT_TIMEOUT);
        if (isOneway) { //单向通信，异步无返回值
            boolean isSent = getUrl().getMethodParameter(methodName, Constants.SENT_KEY, false);
            currentClient.send(inv, isSent);
            RpcContext.getContext().setFuture(null);
            return new RpcResult();
        } else if (isAsync) {//异步通信有返回值
            ResponseFuture future = currentClient.request(inv, timeout);
            // For compatibility
            FutureAdapter<Object> futureAdapter = new FutureAdapter<>(future);
            RpcContext.getContext().setFuture(futureAdapter);

            Result result;
            if (isAsyncFuture) {
                // register resultCallback, sometimes we need the async result being processed by the filter chain.
                result = new AsyncRpcResult(futureAdapter, futureAdapter.getResultFuture(), false);
            } else {
                result = new SimpleAsyncRpcResult(futureAdapter, futureAdapter.getResultFuture(), false);
            }
            return result;
        } else {
            //同步的方式
            RpcContext.getContext().setFuture(null);
            return (Result) currentClient.request(inv, timeout).get();//调用底层的Netty客户端，发送请求
        }
    }  
}

```









6 dubbo中的rpc如何实现。

1、dubbo几个协议的对比，dubbo的底层实现（从这里扯到了netty，socket，三次握手， 

2、dubbo的服务注册过程，服务调用过程，讲一讲底层怎么实现的。（面到这里我基本知道我凉了。。）



5、rpc的调用过程？dubbo的服务调用过程？基本组件？  

 

 

 

 

 








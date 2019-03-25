---
layout: post
title:  "SpringCloud学习笔记及常用配置"
date:   2019-03-24 08:08:08
categories: framework
tags: java springcloud 微服务
excerpt: SpringCloud学习笔记及常用配置
mathjax: true
---

* content
{:toc}

### 注册中心（Eurake、consul）

#### 概述

主要是提供服务的注册、发现。所有的服务在启动时会去向注册中心注册服务，并且会定期的会向注册中发送心跳包，所以注册中心会维护一份存活的服务实例的列表

![avatar](https://github.com/hongmong/hongmong.github.io/blob/master/_posts/image/2019-03-25%20105302.png?raw=true)

* 服务注册：服务提供者在启动的时候会通过发送Rest请求的发昂是将自己注册到Eurake Server上。
* 服务续约：在注册完服务后，服务提供者会维护一个心跳来持续告诉Eurake Server：“我还活着”
* 获取服务：当我们启动消费者的时候，他会发送一个Rest请求给注册中心，来获取注册中心的服务清单。并且会定期的重新请求来更新此清单
* 服务下线：当服务实例正常的关闭时，它会触发一个服务下线的Rest请求给注册中心，告诉注册中心：“我要下线了”
* 失效剔除：注册中心在启动的时候会创建一个定时任务，默认每隔一段时间将当前清单中没有续约的服务剔除出去。

#### 常用配置
> 服务端
```yaml
eureka:
    server:
        # see discussion about enable-self-preservation:
        # https://github.com/jhipster/generator-jhipster/issues/3654
        #是否开启注册中心的保护机制，Eureka 会统计15分钟之内心跳失败的比例低于85%将会触发保护机制，不剔除服务提供者
        enable-self-preservation: false
        #读取对等节点服务器复制的超时的时间，单位为毫秒，默认为200
        peer-node-connect-timeout-ms: 1000
        #集群节点之间读取超时时间，单位为毫秒，默认为200
        peer-node-read-timeout-ms: 1000
        #清理无效节点的时间间隔
        eviction-interval-timer-in-ms: 6000
```

> 客户端
```yaml
eureka:
    client:
        enabled: true
        healthcheck:
            enabled: true
        #是否获取注册中心服务清单
        fetch-registry: true
        #是否将自身的实例注册到注册中心
        register-with-eureka: true
        #定时将实例信息（如果变化了）复制到注册中心的间隔
        instance-info-replication-interval-seconds: 10
        #服务清单的更新时间
        registry-fetch-interval-seconds: 10
        #读取注册中心信息的超时时间
        eureka-server-read-timeout-seconds: 8
        #获取实例时是否过滤，只保留UP状态的实例
        filter-only-up-instances: true
    instance:
        appname: msgcenter
        instanceId: msgcenter:${spring.application.instance-id:${random.value}}
        #服务续约任务的调用间隔时间
        lease-renewal-interval-in-seconds: 5
        #服务时效的时间
        lease-expiration-duration-in-seconds: 10
        #状态页Url
        status-page-url-path: ${management.endpoints.web.base-path}/info
        #健康检查Url
        health-check-url-path: ${management.endpoints.web.base-path}/health
        #是否优先使用ip地址作为主机名的标识
        prefer-ip-address: false
```

### 网关（zuul）
#### 概述

自身也是一个微服务，跟其它服务Service-1，Service-2, ... Service-N一样，都注册在eureka server上，可以相互发现，zuul能感知到哪些服务在线，同时通过配置路由规则，可以将请求自动转发到指定的后端微服务上，对于一些公用的预处理（比如：权限认证，token合法性校验）,可以放在所谓的过滤器(ZuulFilter)里处理，这样后端服务以后新增了服务，zuul层几乎不用修改。每一个进入Zuul的Http请求都会经过一系列的过滤器处理得到请求响应。
    
![avatar](https://github.com/hongmong/hongmong.github.io/blob/master/_posts/image/2019-03-25%20105303.png?raw=true)

>zuul的内置过滤器
>![avatar](https://github.com/hongmong/hongmong.github.io/blob/master/_posts/image/2019-03-25%20135103.png?raw=true)
>zuul实现路由功能也是通过过滤器实现的。上图中RibbonRoutingFilter是做关于serviceId（也就是现在项目中使用的服务注册）的请求转发SimpleHostRoutingFilter是承担那些自定义Url路由的转发功能，这个过滤器是直接创建一个HttpClient进行访问的，不会经过Hytrix和Ribbon组件

>从客户端访问的请求，通过网关最终到达我们的服务，需要经过这些组件，而这些组件的作用和执行顺序如下图
>![avatar](https://github.com/hongmong/hongmong.github.io/blob/master/_posts/image/2019-03-25%20135104.png?raw=true)

#### 自定义过滤器

```java
public class MyFilter extends ZuulFilter {
    private static Logger log= LoggerFactory. getLogger(MyFilter. class);
    /**
     * 过滤器的类型
     *  pre:可以在请求被路由之前调用
     *  routing:在路由请求时调用
     *  post：在routing和error过滤之后被调用
     *  error：处理请求发生错误时被调用
     * @return
     */
    @Override
    public String filterType() {
        return "pre";
    }
    /**
     * 过滤器的执行顺序，当请求在一个阶段中存在多个过滤器时，
     * 需要根据该方法返回的值来依次执行，数值越小优先级越高
     * @return
     */
    @Override
    public int filterOrder() {
        return 0;
    }
    /**
     * 判断该过滤器是否需要被执行
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return false;
    }
    /**
     * 过滤器的具体逻辑
     * @return
     * @throws ZuulException
     */
    @Override
    public Object run() throws ZuulException {
        RequestContext ctx= RequestContext. getCurrentContext();
        HttpServletRequest request= ctx. getRequest();
        log. info(" send {} request to {}",request. getMethod(), request. getRequestURL(). toString());
        Object accessToken= request. getParameter(" accessToken");
        if( accessToken== null) {
            log.warn(" access token is empty");
            //
            ctx.setSendZuulResponse(false);
            //设置返回code
            ctx.setResponseStatusCode(401);
            return null;
        }else {
            log. info(" access token ok");
            return null;
        }

    }
}
```
#### 常用配置

> 路由规则配置

```yaml
#See http://cloud.spring.io/spring-cloud-netflix/spring-cloud-netflix.html
zuul:
    #忽略路由路径匹配，如果我们希望[/test]不被路由，可以这样配置
    ignored-patterns: /**/test/**
    #关闭Zuul的重试功能
    retryable: false
    routes:
        test: #自定义路由名称
            #传统路由 http://gateway/test/api --> http://localhost:10006/api
            path: /test/**
            url: http://localhost:10006/
        payment:
            #和注册中心整合，面向服务路由
            path: /payment/**
            serviceId: payment
        payment-ext:
            path: /payment/ext/**
            serviceId: payment-ext
        #利用网关做本地跳转，需要在网关增加/local的接口才能正常访问，否则404
        forward-test:
            path: /api-b/**
            url: forward:/local
    #可关闭某一个过滤器包括内置的过滤器
    MyFilter:#过滤器名称
        pre:#过滤器类型
            disable: true
```

>超时配置

```yaml
zuul: # those values must be configured depending on the application specific needs
    host:
        #httpClient最大连接数
        max-total-connections: 1000
        #httpclient每一个路由最大连接数
        max-per-route-connections: 100
        #如果Zuul使用服务发现，则需要使用ribbon.ReadTimeout和ribbon.SocketTimeout功能区属性配置这些超时 。
        #如果您通过指定URL配置了Zuul路由，则需要使用 zuul.host.connect-timeout-millis和zuul.host.socket-timeout-millis。
        socket-timeout-millis: 5000
        connect-timeout-millis: 15000
    ribbon-isolation-strategy: thread
更多设置请参考：http://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html
```


### hystrix（熔断器）
熔断器，在远程服务超时或异常时，可提供服务降级，服务熔断的功能。除此之外，Hystrix还可以请求缓存、请求合并
#### 常用配置
```yaml
hystrix:
    threadpool:
        default:
            #设置线程池的线程数
            coreSize: 10
            #设置线程池的最大值，只有allowMaximumSizeToDivergeFromCoreSize=true时才生效
            maximumSize: 5000
            #该属性允许配置maximumSize生效，当coreSize小于maximumSize时创建一个最大值为maximumSize并发的线程池
            allowMaximumSizeToDivergeFromCoreSize: true
            #设置线程池的最大队列大小。当设置-1时，线程将使用SynchronousQueue实现队列，否则使用LinkedBlockingQueue实现的队列
            #该属性只有初始化的时候才有用，无法通过动态刷新的方式调整
            maxQueueSize: -1

    command:
        default:
            circuitBreaker:
                #设置断路器是否起作用
                enabled: true
                #当在配置时间窗口内达到此数量的失败后，进行短路，默认20个
                #例如， 默认该值为20的 时候，如果滚动时间窗（ 默认10 秒）内仅收到了19个请求，即使这19个请求都失败了，断路器也不会打开。
                requestVolumeThreshold: 2
                #短路多久以后开始尝试是否恢复，默认5s
                sleepWindowInMilliseconds: 5000
                #在滚动时间窗中，出错百分比阈值，当达到此阈值后，开始短路。默认50%
                errorThresholdPercentage: 5
                #是否打开断路器
                forceOpen: false
                #是否关闭断路器，当forceOpen打开时，此参数无效
                forceClose: false
            metrics:
                rollingStats:
                    #设置滚动时间窗的长度
                    timeInMilliseconds: 10000
                    #设置滚动的统计窗口被分成的桶（bucket）的数目
                    numBuckets: 10
            execution:
                timeout:
                    #是否有超时限制
                    enabled: true
                isolation:
                    #hystrix默认隔离策略，默认THREAD在固定大小线程池中，以单独线程执行，并发请求数受限于线程池大小。
                    #SEMAPHORE在调用线程中执行，通过信号量来限制并发量
                    strategy: THREAD
                    thread:
                        #hystrix超时设置
                        timeoutInMilliseconds: 15000
                        #hystrix是否执行时在超时发生时被中断线程
                        interruptOntimeout: true
```

### ribbon（客户端负载均衡）
```yaml
ribbon:
#    使用OKhttp请求，默认HttpClient
#    okhttp:
#        enabled: true
    #ribbon请求连接的超时时间- 限制3秒内必须请求到服务，并不限制服务处理的返回时间
    ConnectTimeout: 3000
    #请求处理的超时时间 下级服务响应最大时间,超出时间消费方（路由也是消费方）返回timeout
    ReadTimeout: 10000
    #对所有操作都进行重试
    OkToRetryOnAllOperations: true
    #切换实例的重试次数
    MaxAutoRetriesNextServer: 0
    #对当前实例的重试次数
    MaxAutoRetries: 0
    #在responseCode为在配置之后，才触发重试
    retryableStatusCodes: 404,500
    #设置ribbon负载均衡策略，默认轮询，其他包括
    #RandomRule 随机选择
    #WeightedResponseTimeRule 根据权重来挑选实例，每个实例权重为总响应时间与实际自身的平均响应时间的差的累积所得
    #也可自定义一个负载策略
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
    #是否从注册中心获取服务列表
    eureka:
        enabled: true

payment:
    ribbon:
        #ribbon请求连接的超时时间- 限制3秒内必须请求到服务，并不限制服务处理的返回时间
        ConnectTimeout: 3000
        #请求处理的超时时间 下级服务响应最大时间,超出时间消费方（路由也是消费方）返回timeout
        ReadTimeout: 10000
        #对所有操作都进行重试
        OkToRetryOnAllOperations: true
        #切换实例的重试次数
        MaxAutoRetriesNextServer: 0
        #对当前实例的重试次数
        MaxAutoRetries: 0

farmfriend:
    ribbon:
        listOfServers: http://dev.farmfriend.com.cn
```

### feign
feiin是一个声明式的Http客户端，可以让我们定义远程接口的格式像定义一个接口类一样。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果

```java
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

### zipkin
随着服务的越来越多，对调用链的分析会越来越复杂，如服务之间的调用关系、某个请求对应的调用链、调用之间消费的时间等，对这些信息进行监控就成为一个问题。在实际的使用中我们需要监控服务和服务之间通讯的各项指标，这些数据将是我们改进系统架构的主要依据。因此分布式的链路跟踪就变的非常重要

### ELK
统一的日志收集平台（elastricsearch+logstash+kibana）
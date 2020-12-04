# Spring Cloud 组件 Open Feign相关问题

## 1、如何使用？

1. 首先，调用以及被调用的微服务双方都应该被**注册到注册中心**。

2. Spring Boot启动APP上标注 `@EnableFeignClients`注解。

3. 编写远程调用接口并标注`@FeignClient`注解。（括号内添加所要调用的微服务名称）

4. 接口中的方法为实际想要调用的服务的方法签名，并使用`@PostMapping`注解映射为一个post类型的HTTP请求。


## 2、实现远程调用的原理？（新浪）（百度）

核心原理就是**通过一系列的封装和处理**，将以Java注解的方式定义的远程调用API接口，最终转化为HTTP的请求与响应结果。     

![](C:\Users\lok666\Desktop\feigen远程调用原理图.png)

从上图可以看到，Feign通过**处理注解**，**将请求模板化**，当实际调用的时候，传入参数，根据参数再应用到请求上，进而转化成真正的 Request 请求。

1. 微服务启动时，feign对添加了`@FeignClient`的接口扫描，创建远程接口的本地JDK Proxy代理实例。然后注入到Spring IOC容器中。**当远程接口的方法被调用，由Proxy代理实例去完成真正的远程访问，并且返回结果。**

2. Feign的方法处理器 `MethodHandler` 。它用来解析方法上的`url`，以及`@postMapping`注解中包含的数据，并生成一个http请求模板。

3. 在 `MethodHandler` 中具体由`requestTemplate`发起request请求。Request 经过编码交给`httpclient`发送到远程调用服务。

## 3、如何解决远程调用的负载均衡问题？（百度）

feign内部有Ribbon实现了客户端的负载均衡。从注册中心读取所有可用的服务提供者，在客户端每次调用接口时采用如轮询负载均衡算法选出一个服务提供者调用，因此，`Ribbon`是一个客户端负载均衡器。

常见的负载均衡策略有：

1. 轮询：可能会导致性能较弱的服务器过载
2. 加权轮询：可以为每台服务器分配权重，能力较弱的服务器分配较少的连接请求
3. 最少连接数：
4. 加权最少连接：

# Spring Cloud 组件 Gateway相关问题

## 1、网关定义？

挡在众多微服务前面的一堵墙，用来**管理、授权、流量限制**等等。可以保护后台的微服务。Spring Cloud Gateway旨在为微服务架构提供一种简单而有效的统一的API路由管理方式。Spring Cloud Gateway作为Spring Cloud生态系中的网关，目标是**替代ZUUL**，其不仅提供统一的路由方式，并且基于Filter链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。

## 2、作用？

1. 作为所有微服务的请求入口。
2. 对所有的API进行统一的管理、分发。
3. 作为所有后端服务的聚合点。

## 3、几个重要的概念说一说？（美团）

1. Route（路由）：是网关的基本构建模块，由一个ID、一个目标URI、一组断言、一组过滤器实现。如果断言为真，则会路由至指定的服务中。
2. Predicate （断言）：即一种**匹配规则**。可以根据多种情况进行匹配。我最常用的是根据path或Host进行匹配。
3. Filter（过滤器）：负责对过来的请求按照某种规则进行修改

```yml
    gateway:
    # 路由的示例
      routes:
        - id: product_route
          uri: lb://mall-product
          predicates:
            # 根据path进行匹配
            - Path=/api/product/**
            # 根据host进行匹配，可以一次匹配多个host。
            - Host=catmall.com, item.catmall.com
          filters:
            - RewritePath=/api/(?<segment>.*),/$\{segment}
```

# 

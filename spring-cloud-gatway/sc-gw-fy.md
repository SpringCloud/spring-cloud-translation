---
layout:     post
title:      Spring Cloud Gateway翻译
date:       2017-11-09 14:00:00 +0800
summary:    Spring Cloud Gateway翻译
toc: true
categories:
- Spring Cloud Gateway

tags:
- Spring Cloud Gateway
- Spring Cloud Gateway
---

**摘要**: 本文主要对Spring Cloud Gateway v2.0.0.M3版本文档进行翻译。 
<!--more-->


## spring-cloud-gateway

This project provides an API Gateway built on top of the Spring Ecosystem, including: Spring 5, Spring Boot 2 and Project Reactor. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.

>这个项目提供了一个构建在Spring生态系统之上的API网关，包括：Spring 5，Spring Boot 2和Project Reactor。 Spring Cloud Gateway旨在提供一种简单而有效的API路由方式，并为其提供横切关注点，例如：安全，监控/指标，和弹性。

## How to Include Spring Cloud Gateway

To include Spring Cloud Gateway in your project use the starter with group org.springframework.cloud and artifact id spring-cloud-starter-gateway. See the Spring Cloud Project page for details on setting up your build system with the current Spring Cloud Release Train.

> 要在项目中包含Spring Cloud Gateway，请使用组org.springframework.cloud和工件id spring-cloud-starter-gateway。请参阅Spring Cloud Project页面，以获取有关使用当前Spring Cloud Release Train设置构建系统的详细信息。

If you include the starter, but, for some reason, you do not want the gateway to be enabled, set spring.cloud.gateway.enabled=false.

> 如果你项目中包含了Spring Cloud Gateway这个starter，但是由于某种原因，你不想让它在你的项目中生效，你可以设置spring.cloud.gateway.enabled=false


## Glossary

* Route: Route the basic building block of the gateway. It is defined by an ID, a destination URI, a collection of predicates and a collection of filters. A route is matched if aggregate predicate is true.

> 路由：路由是网关的基本构建模块。它由一个ID，一个目标URI，一组断言和一个过滤器的集合定义。如果聚合谓词为真，则路由匹配

* Predicate: This is a Java 8 Function Predicate. The input type is a Spring Framework ServerWebExchange. This allows developers to match on anything from the HTTP request, such as headers or parameters.

> 谓词：这是一个Java 8函数谓词。输入类型是一个Spring框架的ServerWebExchange。这允许开发人员匹配来自HTTP请求的任何内容，例如标题或参数

* Filter: These are instances Spring Framework GatewayFilter constructed in with a specific factory. Here, requests and responses can be modified before or after sending the downstream request.

>  过滤器：这是实例Spring Framework GatewayFilter与特定工厂构建的实例。这里，可以在发送下游请求之前或之后修改请求和响应


## How It Works



Clients make requests to Spring Cloud Gateway. If the Gateway Handler Mapping determines that a request matches a Route, it is sent to the Gateway Web Handler. This handler runs sends the request through a filter chain that is specific to the request. The reason the filters are divided by the dotted line, is that filters may execute logic before the proxy request is sent or after. All "pre" filter logic is executed, then the proxy request is made. After the proxy request is made, the "post" filter logic is executed.


## Route Predicate Factories

Spring Cloud Gateway matches routes as part of the Spring WebFlux HandlerMapping infrastructure. Spring Cloud Gateway includes many built-in Route Predicate Factories. All of these predicates match on different attributes of the HTTP request. Multiple Route Predicate Factories can be combined and are combined via logical and.


## After Route Predicate Factory


The After Route Predicate Factory takes one parameter, a datetime. This predicate matches requests that happen after the current datetime.

application.yml

```yml
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: after_route
        uri: http://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

This route matches any request after Jan 20, 2017 17:42 Mountain Time (Denver).


## Before Route Predicate Factory

The Before Route Predicate Factory takes one parameter, a datetime. This predicate matches requests that happen before the current datetime.

application.yml

```yml
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: before_route
        uri: http://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```

This route matches any request before Jan 20, 2017 17:42 Mountain Time (Denver).

## Between Route Predicate Factory

The Between Route Predicate Factory takes two parameters, datetime1 and datetime2. This predicate matches requests that happen after datetime1 and before datetime2. The datetime2 parameter must be after datetime1.

application.yml


```yml
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: between_route
        uri: http://example.org
        predicates:
        - Betweeen=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```

This route matches any request after Jan 20, 2017 17:42 Mountain Time (Denver) and before Jan 21, 2017 17:42 Mountain Time (Denver). This could be useful for maintenance windows.

## Cookie Route Predicate Factory

The Cookie Route Predicate Factory takes two parameters, the cookie name and a regular expression. This predicate matches cookies that have the given name and the value matches the regular expression.

application.yml

```java
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: cookie_route
        uri: http://example.org
        predicates:
        - Cookie=chocolate, ch.p
```

This route matches the request has a cookie named chocolate who’s value matches the ch.p regular expression.

## Header Route Predicate Factory

The Header Route Predicate Factory takes two parameters, the header name and a regular expression. This predicate matches with a header that has the given name and the value matches the regular expression.

application.yml

```yml
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: header_route
        uri: http://example.org
        predicates:
        - Header=X-Request-Id, \d+
```

This route matches if the request has a header named X-Request-Id whos value matches the \d+ regular expression (has a value of one or more digits).

## Host Route Predicate Factory

The Host Route Predicate Factory takes one parameter: the host name pattern. The pattern is an Ant style pattern with . as the separator. This predicates matches the Host header that matches the pattern.

application.yml


```yml
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: host_route
        uri: http://example.org
        predicates:
        - Host=**.somehost.org
```
This route would match if the request has a Host header has the value www.somehost.org or beta.somehost.org.


## Method Route Predicate Factory

The Method Route Predicate Factory takes one parameter: the HTTP method to match.

application.yml

```yml
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: method_route
        uri: http://example.org
        predicates:
        - Method=GET
```

This route would match if the request method was a GET.

## Path Route Predicate Factory

The Path Route Predicate Factory takes one parameter: a Spring PathMatcher pattern.


```yml
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: host_route
        uri: http://example.org
        predicates:
        - Path=/foo/{segment}
```

This route would match if the request path was, for example: /foo/1 or /foo/bar.

This predicate extracts the URI template variables (like segment defined in the example above) as a map of names and values and places it in the ServerWebExchange.getAttributes() with a key defined in PathRoutePredicate.URL_PREDICATE_VARS_ATTR. Those values are then available for use by[GatewayFilter Factories](https://github.com/spring-cloud/spring-cloud-gateway/blob/v2.0.0.M3/docs/src/main/asciidoc/spring-cloud-gateway.adoc#gateway-route-filters)

## Query Route Predicate Factory

The Query Route Predicate Factory takes two parameters: a required param and an optional regexp.

application.yml

```yml
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: query_route
        uri: http://example.org
        predicates:
        - Query=baz
```

This route would match if the request contained a baz query parameter.

application.yml


```yml
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: query_route
        uri: http://example.org
        predicates:
        - Query=foo, ba.
```

This route would match if the request contained a foo query parameter whose value matched the ba. regexp, so bar and baz would match.



## RemoteAddr Route Predicate Factory（远程地址路由Predicate工厂）


 远程地址路由Predicate工厂需要维护一个CIDR-notation的字符串列表，该列表的大小大于1，比如192.168.0.1/16 (192.168.0.1一个ip地址，16 是一个子网掩码。

配置文件application.yml如下：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: remoteaddr_route
        uri: http://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24

```

这个路由将会匹配比如像192.168.1.10这样的远程请求。

## GatewayFilter Factories（网关过滤器工厂链）

路由过滤器能够以某种方式允许修改进入的Http请求或输出的对外的Http响应。
路由过滤器的作用域是一个特定的路由。 Spring Cloud Gateway包含许多内置的GatewayFilter工厂。

### AddRequestHeader GatewayFilter Factory（添加请求头的过滤器工厂）

添加请求头的过滤器工厂可以在请求头中添加一对键值对参数。

配置文件application.yml：

```

spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: add_request_header_route
        uri: http://example.org
        filters:
        - AddRequestHeader=X-Request-Foo, Bar
```

以上的配置会将X-Request-Foo：Bar头添加到所有匹配请求的下游请求头。

### AddRequestParameter GatewayFilter Factory（添加请求参数的过滤器工厂）

添加请求参数的过滤器工厂可以在请求中添加一对请求参数的键值对。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: add_request_parameter_route
        uri: http://example.org
        filters:
        - AddRequestParameter=foo, bar
```

以上的配置会将foo：bar参数添加到所有匹配请求的下游请求中。

### AddResponseHeader GatewayFilter Factory（添加响应头的过滤器工厂）

添加响应头的过滤器工厂可以在响应头中添加键值对。

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: add_request_header_route
        uri: http://example.org
        filters:
        - AddResponseHeader=X-Response-Foo, Bar

```

以上的配置会将X-Response-Foo, Bar头添加到所有匹配请求的下游请求的相响应头中。


### Hystrix GatewayFilter Factory（熔断过滤器工厂）

Hystrix GatewayFilter Factory采用单个名称的参数，即HystrixCommand的名称。 （未来版本中可能会添加更多选项）。

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: hytstrix_route
        uri: http://example.org
        filters:
        - Hystrix=myCommandName

```

以上的配置将使用命令名称myCommandName将剩余的过滤器包装在HystrixCommand中。

### PrefixPath GatewayFilter Factory（前缀路径过滤器工厂）

前缀路径过滤器工厂使用的是一个简单的prefix参数。

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: prefixpath_route
        uri: http://example.org
        filters:
        - PrefixPath=/mypath

```

这将前缀/mypath到所有匹配的请求的路径。 所以/hello的请求将被发送到/mypath/hello。

### RequestRateLimiter GatewayFilter Factory（请求限流过滤器工厂）

请求限流过滤器工厂需要三个参数：replenishRate, burstCapacity 和keyResolverName.


- replenishRate 允许用户每秒处理多少个请求。

- burstCapacity TODO：文件的爆发能力

- keyResolver是一个实现KeyResolver接口的bean。在配置中，使用SpEL通过名称引用bean。＃{@myKeyResolver}是引用名为myKeyResolver的bean的SpEL表达式。


KeyResolver.java

```
public interface KeyResolver {
	Mono<String> resolve(ServerWebExchange exchange);
}

```


KeyResolver接口允许可插入策略派生出限制请求的密钥。 在未来的里程碑版本中，将会有一些KeyResolver具体实现类。

Redis的实现基于Stripe工作的。 它需要使用spring-boot-starter-data-redis-reactive 的起步依赖。

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: requestratelimiter_route
        uri: http://example.org
        filters:
        - RequestRateLimiter=10, 20, #{@userKeyResolver}

```

Config.java

```
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

以上的配置定义了每个用户10个请求速率限制。 KeyResolver是一个简单的获取用户请求参数（注意：这不建议用于生产）。

### RedirectTo GatewayFilter Factory（重定向过滤器工厂）

重定向过滤器工厂接受一个状态和一个url参数。 该状态是一个300系列重定向http代码，如301.该网址应该是一个有效的网址。 并将它们组合成一个Location的header.

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: prefixpath_route
        uri: http://example.org
        filters:
        - RedirectTo=302, http://acme.org

```

以上的配置将使用Location：http：//acme.org标头发送状态302以执行重定向。


### RemoveNonProxyHeaders GatewayFilter Factory（去掉非代理头的过滤器工厂）

去掉非代理头的过滤器工厂从转发的请求中删除请求头。 被删除的请求头的默认列表来自IETF。

默认删除请求头如下：

- Connection
- Keep-Alive
- Proxy-Authenticate
- Proxy-Authorization
- TE
- Trailer
- Transfer-Encoding
- Upgrade

要改变这一点，请将spring.cloud.gateway.filter.remove-non-proxy-headers.headers属性设置为要删除的标题名称列表。

### RemoveRequestHeader GatewayFilter Factory（去掉请求头的过滤器工厂）

去掉请求头的过滤器工厂需要一个名称的参数，这个名称参数是需要去掉的请求头名。

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: removerequestheader_route
        uri: http://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo

```

以上的配置将在下游发送之前删除X-Request-Foo头。

### RemoveResponseHeader GatewayFilter Factory （去掉响应头的过滤器工厂）

去掉响应头的过滤器工厂需要一个名称的参数，这个名称参数是需要去掉的响应头名。

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: removeresponseheader_route
        uri: http://example.org
        filters:
        - RemoveResponseHeader=X-Response-Foo
```

这将在返回到网关客户端之前从响应中删除X-Response-Foo头。

### RewritePath GatewayFilter Factory（重些url路径的过滤器工厂）

重些url网关过滤器工厂采用路径正则表达式参数和一个替换参数。 使用的是Java正则表达式来灵活地重写请求路径。

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: rewritepath_route
        uri: http://example.org
        - Path=/foo/**
        filters:
        - RewritePath=/foo/(?<segment>.*), /$\{segment}
```

以上的配置，对于/foo/bar的请求路径，将在进行下游请求之前将路径设置为/ bar。 注意由于YAML规范，"$\" 被 "$" 替换。

### SecureHeaders GatewayFilter Factory（安全头过滤工厂）

安全头过滤工厂给响应添加了一些列的安全头，这些安全头在这篇文章有详细的介绍，文章地址：https://blog.appcanary.com/2017/http-security-headers.html

以下的安全头被采纳，并且有默认值：

- X-Xss-Protection:1; mode=block
- Strict-Transport-Security:max-age=631138519
- X-Frame-Options:DENY
- X-Content-Type-Options:nosniff
- Referrer-Policy:no-referrer
- Content-Security-Policy:default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline'
- X-Download-Options:noopen
- X-Permitted-Cross-Domain-Policies:none

如果要更改默认值，在spring.cloud.gateway.filter.secure-headers命名空间中设置适当的属性：

要更改的属性：

- xss-protection-header
- strict-transport-security
- frame-options
- content-type-options
- referrer-policy
- content-security-policy
- download-options
- permitted-cross-domain-policies

### SetPath GatewayFilter Factory（设置路径过滤器工厂）

设置路径过滤器工厂需要路径模板参数。 它提供了一种简单的方法来通过允许路径的模板化段来操纵请求路径。 使用了Spring框架的uri模板。 允许多个匹配段。

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: setpath_route
        uri: http://example.org
        predicates:
        - Path=/foo/{segment}
        filters:
        - SetPath=/{segment}

```

对于/foo/bar的请求路径，这将在进行下游请求之前将路径设置为/bar。

### SetResponseHeader GatewayFilter Factory（设置响应头网关过滤器工厂）

设置响应头网关过滤器工厂需要参数名和参数的值的键值对参数。

配置文件application.yml：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: setresponseheader_route
        uri: http://example.org
        filters:
        - SetResponseHeader=X-Response-Foo, Bar

```

这个网关过滤器将用给定的响应头的值替换掉原来请求响应头的值，注意不是添加而是替换。 在上面的配置中，如果下游服务器使用X-Response-Foo：1234进行响应，则将被X-Response-Foo：Bar取代，网关客户端收到的响应头为X-Response-Foo：Bar而不是X-Response-Foo：1234。


### SetStatus GatewayFilter Factory (请求响应状态网关过滤器工厂)
The SetStatus GatewayFilter Factory takes a single `status` parameter. It must be a valid Spring `HttpStatus`. It may be the integer value `404` or the string representation of the enumeration `NOT_FOUND`.

> 请求响应网关过滤器工厂需要一个`status`参数。该参数值必须是一个有效的Spring `HttpStatus`。该值可以是整型`404`或者是表示枚举类型`NOT_FOUND`的字符串。

application.yml

```java
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: setstatusstring_route
        uri: http://example.org
        filters:
        - SetStatus=BAD_REQUEST
      - id: setstatusint_route
        uri: http://example.org
        filters:
        - SetStatus=401
```

In either case, the HTTP status of the response will be set to 401.
> 如上配置下，无论什么情况，http的响应状态都会被置为401

## Global Filters （全局过滤器）

The `GlobalFilter` interface has the same signature as `GatewayFilter`. These are special filters that are conditionally applied to all routes. (This interface and usage are subject to change in future milestones).

> `GlobalFilter`全局过滤器接口有和`GatewayFilter`相同的签名。这些特殊的过滤器有条件性的应用到全部路由中。（这类接口和用法将会在未来里程碑版本中发生变化）

### Combined Global Filter and GatewayFilter Ordering （组合全局过滤器和网关过滤器定序）

TODO: document ordering

> 接下来要做：文档化定序方式

### Forward Routing Filter （转发路由过滤器）

The `ForwardRoutingFilter` looks for a URI in the exchange attribute `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`. If the url has a `forward` scheme (ie `forward:///localendpoint`), it will use the Spring `DispatcherHandler` to handler the request. The unmodified original url is appended to the list in the `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR` attribute.

> `ForwardRoutingFilter`通过以`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`为key在交换属性中获得URI（统一资源标识符）。如果这个url(统一资源定位符)是`forward`协议，（比如：`forward:///localendpoint`），那么就会通过Spring`DispatcherHandler` 去处理该请求。这个未修改的原始路径会被添加到以`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`为key的列表中。

### LoadBalancerClient Filter (客户端负载均衡过滤器)

The `LoadBalancerClientFilter` looks for a URI in the exchange attribute `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`. If the url has a `lb` scheme (ie `lb://myservice`), it will use the Spring Cloud `LoadBalancerClient` to resolve the name (`myservice` in the previous example) to an actual host and port and replace the URI in the same attribute. The unmodified original url is appended to the list in the `ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR` attribute.

> `LoadBalancerClientFilter`通过以`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`为key在交换属性中获得URI。如果这个url是`lb`协议（比如`lb://myservice`），那么就会通过Spring Cloud `LoadBalancerClient`去处理这个`myservice`（这个名字是与前面的例子中匹配的）找到真实的主机和端口，然后在uri的同类属性中进行替换。这个未修改的原始路径会被添加到以`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`为key的LinkedHashSet列表中。

### Netty Routing Filter （Netty路由过滤器）

The Netty Routing Filter runs if the url located in the `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` exchange attribute has a `http` or `https` scheme. It uses the Netty `HttpClient` to make the downstream proxy request. The response is put in the `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR` exchange attribute for use in a later filter. (There is an experimental `WebClientHttpRoutingFilter` that performs the same function, but does not require netty)

> 如果在交换属性中以`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`为key获得的url是以`http`或者是`https`为请求协议，那么Netty路由过滤器就会生效工作。其是使用Netty`HttpClient`去生成一个下游的代理请求。请求的响应根据`ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR`添加进交换属性中以供后面的过滤器使用。（有一个实验性的过滤器`WebClientHttpRoutingFilter`执行相同的函数，但是不需要Netty）

### Netty Write Response Filter （Netty响应过滤器）

The `NettyWriteResponseFilter` runs if there is a Netty `HttpClientResponse` in the `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR` exchange attribute. It is run after all other filters have completed and writes the proxy response back to the gateway client response. (There is an experimental `WebClientWriteResponseFilter` that performs the same function, but does not require netty)

> 如果在交换属性中以`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`为key获得的请求响应是一个Netty的响应 `HttpClientResponse`，那么Netty响应过滤器`NettyWriteResponseFilter`就会生效工作。其是在所有的其他过滤器处理完成之后开始工作的，并且写入代理请求返回给网关客户端响应。（有一个实验性的过滤器`WebClientWriteResponseFilter`执行相同的函数，但是不需要Netty）

### RouteToRequestUrl Filter (路由到请求地址过滤器)

The `RouteToRequestUrlFilter` runs if there is a `Route` object in the `ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR` exchange attribute. It creates a new URI, based off of the request URI, but updated with the URI attribute of the `Route` object. The new URI is placed in the `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` exchange attribute`.

> 如果在交换属性中以`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`为key获得一个`Route`对象，那么路由请求地址过滤器`RouteToRequestUrlFilter`就会生效工作。其创建一个新的URI是基于请求的URI，但是根据`Route`对象的URI属性进行更新。新的URI会以`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`为key放置到交换属性当中。

### Websocket Routing Filter （Websocket路由过滤器）

The Websocket Routing Filter runs if the url located in the `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR` exchange attribute has a `ws` or `wss` scheme. It uses the Spring Web Socket infrastructure to forward the Websocket request downstream.

> 如果在交换属性中以`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`为key获得的url是以`ws`或者是`wss`为请求协议，那么Netty路由过滤器就会生效工作。其使用Spring Web Socket底层代码处理，来将Websocket请求转发到下游。

## Configuration （配置）

Configuration for Spring Cloud Gateway is driven by a collection of `RouteDefinitionLocator` s.

> Spring Cloud Gateway的配置是被一系列的`RouteDefinitionLocator`类来管理的。

RouteDefinitionLocator.java

```java
public interface RouteDefinitionLocator {
	Flux<RouteDefinition> getRouteDefinitions();
}
```

By default, a `PropertiesRouteDefinitionLocator` loads properties using Spring Boot's `@ConfigurationProperties` mechanism.

> 默认方式下，`PropertiesRouteDefinitionLocator`是通过使用Spring Boot的 `@ConfigurationProperties` 原理来进行加载配置的。

The configuration examples above all use a shortcut notation that uses positional arguments rather than named ones. The two examples below are equivalent:

> 以上的配置例子全部都是使用一个指定位置参数（译者注：“_genkey_”+i）的快捷方式，而不是指定命名的。下面两个例子是等价的：

application.yml

```java
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: setstatus_route
        uri: http://example.org
        filters:
        - name: SetStatus
          args:
            status: 401
      - id: setstatusshortcut_route
        uri: http://example.org
        filters:
        - SetStatus=401
```

For some usages of the gateway, properties will be adequate, but some production use cases will benefit from loading configuration from an external source, such as a database. Future milestone versions will have `RouteDefinitionLocator` implementations based off of Spring Data Repositories such as: Redis, MongoDB and Cassandra.

> 在gateway网关的一些使用方法上，properties配置会是合适的，但是有些生产使用案例中使用从外部来源加载配置将会更好，比如从一个数据库。未来的里程碑版本将会有 `RouteDefinitionLocator`基于Spring Data Repositories来实现，例如：Redis, MongoDB and Cassandra。

### Fluent Java Routes API （流式Java路由API）

To allow for simple configuration in Java, there is a fluent API defined in the `Routes` class.

> 为了在Java中更简单的配置，在`Routes`类中定义了流式API。

GatewaySampleApplication.java

```java
// static imports from GatewayFilters and RoutePredicates
@Bean
public RouteLocator customRouteLocator(ThrottleGatewayFilterFactory throttle) {
    return Routes.locator()
            .route("test")
                .predicate(host("**.abc.org").and(path("/image/png")))
                .addResponseHeader("X-TestHeader", "foobar")
                .uri("http://httpbin.org:80")
            .route("test2")
                .predicate(path("/image/webp"))
                .add(addResponseHeader("X-AnotherHeader", "baz"))
                .uri("http://httpbin.org:80")
            .route("test3")
                .order(-1)
                .predicate(host("**.throttle.org").and(path("/get")))
                .add(throttle.apply(tuple().of("capacity", 1,
                    "refillTokens", 1,
                    "refillPeriod", 10,
                    "refillUnit", "SECONDS")))
                .uri("http://httpbin.org:80")
            .build();
}
```

This style also allows for more custom predicate assertions. The predicates defined by `RouteDefinitionLocator` beans are combined using logical `and`. By using the fluent Java API, you can use the `and()`, `or()` and `negate()` operators on the `Predicate` class.

> 这类风格也允许更多的自定义predicates断言。被 `RouteDefinitionLocator` 实例定义的predicates通过使用逻辑 `and` 来组合。通过使用Java流式API，你能使用`and()`, `or()` 和 `negate()`来操作`Predicate`类。

## Actuator API （执行器API）

TODO: document the `/gateway` actuator endpoint

> 接下来要做：`/gateway`执行器端点文档

## Developer Guide （开发者指南）

TODO: overview of writing custom integrations

> 接下来要做：编写自定义集成概述

### Writing Custom Route Predicate Factories （编写自定义路由Predicate工厂）

TODO: document writing Custom Route Predicate Factories

> 接下来要做：文档化编写自定义路由Predicate工厂

### Writing Custom GatewayFilter Factories （编写自定义GatewayFilter工厂）

TODO: document writing Custom GatewayFilter Factories

> 接下来要做：文档化编写自定义GatewayFilter工厂

### Writing Custom Global Filters （编写自定义全局过滤器）

TODO: document writing Custom Global Filters

> 接下来要做：文档化编写自定义全局过滤器

### Writing Custom Route Locators and Writers （编写自定义路由定位器和写入器）

TODO: document writing Custom Route Locators and Writers

> 接下来要做：文档化编写自定义路由定位器和写入器

## Building a Simple Gateway Using Spring MVC （使用Spring MVC构建一个简单的网关）

Spring Cloud Gateway provides a utility object called `ProxyExchange` which you can use inside a regular Spring MVC handler as a method parameter. It supports basic downstream HTTP exchanges via methods that mirror the HTTP verbs, or forwarding to a local handler via the `forward()` method.

> Spring Cloud Gateway 提供了名为 `ProxyExchange` 一个实用化对象，你可以使用它来内置一个正确的Spring MVC处理器作为方法参数。它支持通过真实的HTTP的方法来替换下游的内部HTTP，或者通过 `forward()` 方法转发到一个本地的处理器。

Example (proxying a request to "/test" downstream to a remote server):

> 例如下：（代理请求"/test"的请求到下游的一个远程服务）

```java
@RestController
@SpringBootApplication
public class GatewaySampleApplication {

	@Value("${remote.home}")
	private URI home;

	@GetMapping("/test")
	public ResponseEntity<?> proxy(ProxyExchange<Object> proxy) throws Exception {
		return proxy.uri(home.toString() + "/image/png").get();
	}

}
```

There are convenience methods on the `ProxyExchange` to enable the handler method to discover and enhance the URI path of the incoming request. For example you might want to extract the trailing elements of a path to pass them downstream:

> 在`ProxyExchange`有简单的方式去通过处理器方法发现并完善请求的URI路径。比如你可能想提取路径后的元素，以便将它们传递到下游：


```java
@GetMapping("/proxy/path/**")
public ResponseEntity<?> proxyPath(ProxyExchange<?> proxy) throws Exception {
  String path = proxy.path("/proxy/path/");
  return proxy.uri(home.toString() + "/foos/" + path).get();
}
```

All the features of Spring MVC are available to Gateway handler methods. So you can inject request headers and query parameters, for instance, and you can constrain the incoming requests with declarations in the mapping annotation. See the documentation for `@RequestMapping` in Spring MVC for more details of those features.

> 所有的Spring MVC的特性都可以使用到网关的处理器方法。因此你可以注入请求头和查询参数，你可以使用映射注解声明来约束请求。 查看Spring MVC中的 `@RequestMapping` 文档可以了解更多的特性。

Headers can be added to the downstream response using the `header()` methods on `ProxyExchange`.

> 通过 `ProxyExchange` 的 `header()` 方法可以添加头信息到下游的响应中。

You can also manipulate response headers (and anything else you like in the response) by adding a mapper to the `get()` etc. method. The mapper is a `Function` that takes the incoming `ResponseEntity` and converts it to an outgoing one.

> 通过向 `get()` 之类的方法添加一个映射，你也可以操作响应头（包括在响应中你想要的）。这个映射是一个需要输入`ResponseEntity` 和转化指定输出的 `Function` 函数。 

First class support is provided for "sensitive" headers ("cookie" and "authorization" by default) which are not passed downstream, and for "proxy" headers (`x-forwarded-*`).

> 最高优先级的类支持处理让 `sensitive` 敏感头信息（`cookie`和`authorization`是默认的）不往下游传递，并且对于 `proxy` 代理头信息处理为（`x-forwarded-*`）。
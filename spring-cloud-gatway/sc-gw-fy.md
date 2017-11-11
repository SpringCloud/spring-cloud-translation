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



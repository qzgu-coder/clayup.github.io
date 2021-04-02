---
layout: post # 使用post模版
title:  "SpringCloud.2 - 服务熔断-服务转发"
date:   2021-04-01
categories: SpringCloude SpringBoot Java
tags:   SpringCloud SpringBoot Java

---


### 服务熔断

>在微服务架构中通常会有多个服务层调用，基础服务的故障可能会导致级联故障，进而造成整个系统不可用的情况，这种现象被称为服务雪崩效应。服务雪崩效应是一种因“服务提供者”的不可用导致“服务消费者”的不可用,并将不可用逐渐放大的过程。如果下图所示：A作为服务提供者，B为A的服务消费者，C和D是B的服务消费者。A不可用引起了B的不可用，并将不可用像滚雪球一样放大到C和D时，雪崩效应就形成了。

- 在 `application.yml` 导入依赖

```yaml
feign:
  hystrix:
    enabled: true
```

- 实现上一部分的`LabelClient`的接口

```java
@Component
public class LabelClientImpl implements LabelClient {
	@Override
	public Result findById(String id) {
		return new Result(false, StatusCode.ERROR, "熔断器启动了");
	}
}
```

- 修改LabelClient的注解

```java
@FeignClient(value="tensquare‐base",fallback = LabelClientImpl.class)
```

- 启动测试

## Zuul（网关）

Zuul是Netflix开源的微服务网关，他可以和Eureka,Ribbon,Hystrix等组件配合使用。Zuul组件的核心是一系列的过滤器，这些过滤器可以完成以下功能：

身份认证和安全: 识别每一个资源的验证要求，并拒绝那些不符的请求审查与监控

动态路由：动态将请求路由到不同后端集群

压力测试：逐渐增加指向集群的流量，以了解性能

负载分配：为每一种负载类型分配对应容量，并弃用超出限定值的请求

静态响应处理：边缘位置进行响应，避免转发到内部集群

多区域弹性：跨域AWS Region进行请求路由，旨在实现ELB(ElasticLoad Balancing)使用多样化Spring Cloud对Zuul进行了整合和增强。

### 使用

- 创建微服务网关model，pom添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
<dependency>    
```

- 新建`application.yml`配置文件

```yaml
server:
  port: 9011
spring:
  application:
    name: tensquare-gateway
eureka:
  client:
    service-url:
      defaultZone: http://localhost:6868/eureka
    fetch-registry: true
  instance:
    prefer-ip-address: true
zuul:
  host:
    socket-timeout-millis: 60000
    connect-timeout-millis: 60000
  routes:
    tensquare-gathering: #活动
      path: /gathering/** #配置请求URL的请求规则
      serviceId: tensquare-gathering #指定Eureka注册中心中的服务id
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 6000

ribbon:
  ConnectTimeout: 500
  ReadTimeout: 3000

```

至此可以在浏览器请求

```http
http://localhsot:9011/gathering/{后面更你的服务url路径}
```

### 过滤器

在有些时候我们需要对服务请求进行鉴权，这是我们可以通过zuul提供的过滤器来鉴权

`filterType`：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：

- pre ：可以在请求被路由之前调用
- route ：在路由请求时候被调用
- post ：在route和error过滤器之后被调用
- error ：处理请求时发生错误时被调用

filterOrder ：通过int值来定义过滤器的执行顺序

shouldFilter ：返回一个boolean类型来判断该过滤器是否要执行，所以通过此函数可实现过滤器的开关。在上例中，我们直接返回true，所以该过滤器总是生效

run ：过滤器的具体逻辑。

```java
@Component
public class WebFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        System.out.println("zuul filter running ... ");
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();

        String authorization = request.getHeader("Authorization");
        if (authorization != null) {
            context.addZuulRequestHeader("Authorization", authorization);
        }
        return null;
    }
}

```
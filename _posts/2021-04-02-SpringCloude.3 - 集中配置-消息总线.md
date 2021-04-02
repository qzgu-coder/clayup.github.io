---
layout: post # 使用post模版
title:  "SpringCloud.3 - 集中配置-消息总线"
date:   2021-04-02
categories: SpringCloude SpringBoot Java
tags:   SpringCloud SpringBoot Java
---


## SpringCloudConfig（集中配置）

>在分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。在Spring Cloud中，有分布式配置中心组件spring cloud config ，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程Git仓库中。在spring cloud config 组件中，分两个角色，一是config server，二是configclient。
>
>Config Server是一个可横向扩展、集中式的配置服务器，它用于集中管理应用程序各个环境下的配置，默认使用Git存储配置文件内容，也可以使用SVN存储，或者是本地文件存储。
>
>Config Client是Config Server的客户端，用于操作存储在Config Server中的配置内容。微服务在启动时会请求Config Server获取配置文件的内容，请求到后再启动容器。
>
>详细内容看在线文档： https://springcloud.cc/spring-cloud-config.html

### 配置服务端

- 在gitee新建仓库
- 按照以下格式建立新的配置文件

```cmd
{application}-{profile}.yml 或 {application}-{profile}.properties
```

application为应用名称 profile指的开发环境（用于区分开发环境，测试环境、生产环境等）

- 复制仓库地址

### 实现 - 服务端

- 创建config 模块，添加依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
	</dependency>
</dependencies
```

- 创建ConfigServerApplication

```java
@EnableConfigServer //开启配置服务
@SpringBootApplication
public class ConfigServerApplication {
public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}
```

- 添加application.yml文件

```yaml
spring:
  application:
    name: tensquare‐config
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/{you_git_name}/{you_config_repository}.git
server:
  port: 12000
```

- 浏览器测试

```http
http://localhost:12000/base-dev.yml
```

### 实现 - 客户端

- 在客户端清除application.yml文件，添加bootstrap.yml文件

```yaml
spring:
  cloud:
    config:
      enabled: true
      name: base
      profile: dev
      label: master
      uri: http://127.0.0.1:12000 # 你的config地址
```

- 添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
<dependency>    
```

- 测试服务

## SpringCloudBus（消息总线组件）

> 如果我们更新码云中的配置文件，那客户端工程是否可以及时接受新的配置信息呢？我们现在来做有一个测试，修改一下码云中的配置文件中mysql的端口 ，然后测试http://localhost:9001/label 数据依然可以查询出来，证明修改服务器中的配置并没有更新立刻到工程，只有重新启动程序才会读取配置。 那我们如果想在不重启微服务的情况下更新配置如何来实现呢? 我们使用SpringCloudBus来实现配置的自动更新。

### 实现 - 服务端

-  修改config 模块的pom.xml

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-bus</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>	
	<artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```

- 修改application.yml，添加配置

```yaml
spirng: 
  rabbitmq:
    host: 192.168.52.128
management: #暴露触发消息总线的地址
  endpoints:
    web:
      exposure:
        include: bus‐refresh
```

### 实现 - 客户端

- 修改客户端pom.xml

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-bus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- 修改码云application.yml

```yaml
spring: 
  rabbitmq: 
    host: 192.168.52.128
```

- 测试 配置文件修改之后 会不会自动更新配置文件

  使用 postman请求，观察控制台

```http
http://127.0.0.1:12000/actuator/bus-refresh
```

请求新的配置文件

```http
http://localhost:12000/base-dev.yml
```

### 自定义配置文件读取

- 修改码云上的配置文件，增加自定义配置

```yaml
sms: 
  ip: 192.168.0.17
```

- 在服务模块中新建controller

  注意增加注解：`@RefreshScope`用于刷新配置

```java
@RefreshScope
@RestController
public class TestController {
	@Value("${sms.ip}")
	private String ip;
    
	@RequestMapping(value = "/ip", method = RequestMethod.GET)
	public String ip() {
		return ip;
	}
}
```




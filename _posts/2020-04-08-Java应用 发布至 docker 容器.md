---
layout: post # 使用post模版
title:  "Java 应用发布至 docker 容器 "
date:   2021-04-08
categories: Java docker
tags:    Java docker
---
本文将介绍如何把Java 应用发布至 docker 容器


- IDEA 安装插件`Docker`

<img src="/images/posts/java/idea-docker-plugins.png" alt="idea-docker-plugins" style="zoom:67%;" />

- 在项目路径根路径编写Dockefile

```dockerfile
FROM java:8
EXPOSE 8080
ARG JAR_FILE
ADD target/${JAR_FILE} /covid-19-1.0.0-SNAPSHOT.jar
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' >/etc/timezone
ENTRYPOINT ["java", "-jar","/covid-19-1.0.0-SNAPSHOT.jar"]
```

- 引入`pom`文件

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.0.0</version>
    <configuration>
        <dockerHost>{you.docker.url}</dockerHost>
        <!-- URL示例： http://192.168.52.128:2375 docker需要开启远程登录，并在docker宿主机放行相关端口 -->
        <imageName>{you.project.name}</imageName>
        <forceTags>true</forceTags>
        <!--dockerfile的位置-->
        <dockerDirectory>${project.basedir}/</dockerDirectory>
        <!--jar名称配置，用在dockerfile中，需要自己写的可不用此配置-->
        <buildArgs>
            <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
        <!--jar源位置-->
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```

最后依次点击下图，就可以发布到`docker`

<img src="/images/posts/java/idea-docker-plugins.png" alt="idea-docker-plugins" style="zoom:67%;" />

- 在docker中查看镜像

```cmd
 # 查看镜像
 docker images 
```


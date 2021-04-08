---
layout: post # 使用post模版
title:  "SpringBoot.1 - MongoDB整合 "
date:   2021-04-08
categories: SpringBoot Java MongoDB
tags:    SpringBoot Java MongoDB
---
本文将介绍在`SpringBoot`中使用`MongoDB`的流程

## 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
    <optional>true</optional>
</dependency>
```

## 配置

`application.yml`

```yaml
spring: 
  data:
    mongodb:
      host: 192.168.52.128 # mongodb 地址
      database: covid      # 数据库名
```

## Pojo配置

```java
@Document(collection = "crawler_record")
public class CrawlerRecord implements Serializable {
    @Id
    private String _id;
    private String content;
    private Date updateTime;

    public String get_id() {
        return _id;
    }

    public void set_id(String _id) {
        this._id = _id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public Date getUpdateTime() {
        return updateTime;
    }

    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }
}
```

## 数据层

```java
public interface CrawlerRecordRepository extends MongoRepository<CrawlerRecord, String> {
}
```

## 调用

此时就可以在service中注入服务调用

```java
@Autowired
private CrawlerRecordRepository crawlerRecordRepository;

public void test(){
        CrawlerRecord crawlerRecord = new CrawlerRecord();
        crawlerRecord.setContent(result);

        crawlerRecord.setUpdateTime(DateUtil.parserYMDHMS(DateUtil.formatYMDHMS(new Date())));
        crawlerRecordRepository.insert(crawlerRecord);
        System.out.println(crawlerRecordRepository.findAll().size());
}
```




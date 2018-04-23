---
layout: post
title: spring-boot项目在外部tomcat环境下部署
date:   2018-04-23 15:36:00 +0800
categories: 框架
tag: SpringBoot
author: Michael Wang
source_id: 180423153600
---

* content
{:toc}

spring-boot默认提供内嵌的tomcat，所以打包直接生成jar包，用`java -jar`命令就可以启动。
但是，有时候我们更希望一个tomcat来管理多个项目，这种情况下就需要项目是war格式的包而不是jar格式的包。
spring-boot同样提供了解决方案，只需要简单的几步更改就可以了，这里提供maven项目的解决方法：<br/>

1. 将项目的启动类Application.java继承SpringBootServletInitializer并重写configure方法

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}
```

2.在pom.xml文件中，将project下面的packaging标签中的jar改为war.

```xml
<packaging>war</packaging>
```

3.还是在pom.xml文件中，dependencies下面添加.
```xml
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
</dependency>
```
这样，只需要以上3步就可以打包成war包，并且部署到tomcat中了。需要注意的是这样部署的request url需要在端口后加上项目的名字才能正常访问。spring-boot更加强大的一点就是：即便项目是以上配置，依然可以用内嵌的tomcat来调试，启动命令和以前没变，还是`mvn spring-boot:run`.
<br/>
如果需要在springboot中加上request前缀，需要在application.properties中添加server.contextPath=/prefix/即可。其中prefix为前缀名。这个前缀会在war包中失效，取而代之的是war包名称，如果war包名称和prefix相同的话，那么调试环境和正式部署环境就是一个request地址了。

## 参考
[spring-boot项目在外部tomcat环境下部署](https://blog.csdn.net/james_wade63/article/details/51009423)
[深入浅出ES6（十四）：let和const](http://www.infoq.com/cn/articles/es6-in-depth-let-and-const)<br/>
[JavaScript中var、let、const区别？](https://www.zhihu.com/question/52662013)<br/>
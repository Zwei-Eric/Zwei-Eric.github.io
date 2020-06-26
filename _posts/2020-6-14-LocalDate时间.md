---
layout: post
title: "不是吧，阿sir，还在用Date类型表示时间？"
categories: java
---





## 一、Date、TimeStamp数据类型

本来在springboot项目中时间类型用的Date类型，但试了好几次插入时间结果都是错误的。回想起来有文章介绍过Date类型和TimeStampe类型都已经过时，使用不方便不说，也不是线程安全的，所以考虑用LocalDate/LocalDateTime类型来代替。



## 二、 LocalDate、LocalDateTime类型

LocalDate和LocalDateTime都是java8中引入的一系列新时间类型中的一种，可读性佳，修改方便，可以自定义输出的格式，同时新的时间类是线程安全的，不用考虑并发的问题。

在项目中遇到的问题是时间字符串的解析出错，而用LocalDate时间类可以非常方便地解析字符串。

```java
LocalDateTime time = LocalDateTime.parse("2020-06-14 19:58", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm"));
System.out.println(time);

//输出为2020-06-14T19:58
```



## 三、在SpringBoot+Mybatis项目中使用LocalDate

在使用LocalDate时间类型的时，项目需要增加一些配置才能正常运行，基本上都是解决类型解析的问题。

1. 在pom文件中增加thymeleaf对java8适配的依赖

   ```xml
   <dependency>
   	<groupId>org.thymeleaf.extras</groupId>
   	<artifactId>thymeleaf-extras-java8time</artifactId>
   	<version>${thymeleaf-extras-java8time.version}</version>
   </dependency>
   ```

   在thymeleaf的模板文件中，需要用`temporals.format()`来标注对LocalDate类型的解析。第一个参数是要上传的参数，第二个参数是时间字符串的格式。

   ```html
   <input name="birthday" type="text" class="form-control" placeholder="yyyy-mm-dd" th:value="${emp!=null}?${#temporals.format(emp.birthday, 'yyyy-MM-dd')}">
   ```

   2.在pom文件中增加mybatis对LocalDate类型解析的依赖，需要的包是jsr310，如下代码所示。而在前后端分离的项目中，后端把数据json化，需要添加json时间格式化的依赖，才能正确地把localDate类型转换成格式化时间字符串。

   
   
   ```xml
   <dependency>
   	<groupId>org.mybatis</groupId>
   	<artifactId>mybatis-typehandlers-jsr310</artifactId>
   	<version>1.0.0</version>
</dependency>
   
   <dependency>
   	<groupId>com.fasterxml.jackson.datatype</groupId>
   	<artifactId>jackson-datatype-jsr310</artifactId>
   	<version>${jackson.version}</version>
   </dependency>
   ```
   
   
   
   3.在项目的类Bean中，需要给LocalDate的成员变量加上`DateTimeFormat(pattern = "yyyy-MM-dd")` 注解来表示时间格式的解析规则，否则从前端返回数据注入到容器中会出现解析错误的情况。注意这时输入字符串的格式必须严格和规定格式相同：yyyy-MM-dd，否则还是会报解析错误400代码。
   
   从后端转成前端所需的json数据则需要`@JsonFormat(pattern = "yyyy-MM-dd hh:mm:ss")`注解
   
   

```java
import java.time.LocalDate;
public class Employee {
...
	@DateTimeFormat(pattern = "yyyy-MM-dd")
	@JsonFormat(pattern = "yyyy-MM-dd hh:mm:ss")
	private LocalDate birthday;
...
}
```

数据库端不需要做修改，可以继续使用DATE数据类型，也能正确地从LocalDate转换到DATE类型。

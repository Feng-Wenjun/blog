---
layout: post
title: spring学习笔记
date: 2021-12-07 12:07:00 +0800
category: tutorial
thumbnail: /style/image/thumbnail.png
icon: book
---
* content
  {:toc}


## 需要逐渐掌握的思想

"依赖注入"是实现"控制反转"的一种方法,如果说"控制反转"是一种设计思想,那"依赖注入"是就是这种思想的一种实现,通过注入参数或实例的方式实现控制反转

## 常用的依赖导入

一个就能搞到javaweb中需要的很多包

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
    </dependencies>
```

## 注解

***springboot中大量采用注解替代xml 一定要多了解注解底层源码***

###注意事项
使用注解开发需要导入aop包，并导入context约束，增加对注解的支持
有`<context:component-scan base-package="org.example"/>`后不需要`<context:annotation-config/>`
因为前者包含后者

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

***疑问***：输入`<context:annotation-config/>`idea没能成功自动填充 会报错

不是自己的类使用不了 即无法ref

### @Autowired和@Resource

Autowired和Resource都能放置在set方法和属性上面实现自动注入
两者主要自动装配对象
都使用反射 即使不加set也能使用反射机制注入

@Autowired默认按类型自动装配（bytype），如果有多个相同类型则byname

如果`@Autowired`自动装配的环境复杂（比如有多个同类型对象）
应使用`@Qualifier(value="objName")`用beanid去唯一指定一个bean对象

@Resource默认按名称自动装配（byname），如果找不到名称则bytype
是Java自带的注解而非Spring的注解，在jdk8后被取消 jdk11中被Autowired替代

### @Component

写在类的上方，说明这个类被spring管理了，变成bean！

#### @Value

写在属性上方，可以代替简单类型的赋值

#### 衍生注解

对应web mvc三层架构
dao @Repository
service @Service
controller @Controller
功能完全一致，都是将类注册到spring容器中并装配

### @Scope

写在类的上方表明作用域，常用的有单例模式和原型模式
`@Scope("prototype")`

![img.png](img.png)
![img_1.png](img_1.png)


| Scope | Description |
| - | - |
| singleton | （默认）将每个 Spring IoC 容器的单个 bean 定义范围限定为单个对象实例。 |
| prototype | 将单个 bean 定义的作用域限定为任意数量的对象实例。 |
| request | 将单个 bean 定义的范围限定为单个 HTTP 请求的生命周期。也就是说，每个 HTTP 请求都有一个在单个 bean 定义后面创建的 bean 实例。仅在可感知网络的 Spring ApplicationContext中有效。 |
| session | 将单个 bean 定义的范围限定为 HTTP Session的生命周期。仅在可感知网络的 Spring ApplicationContext上下文中有效。 |
| application | 将单个 bean 定义的范围限定为ServletContext的生命周期。仅在可感知网络的 Spring ApplicationContext上下文中有效。 |
| websocket | 将单个 bean 定义的范围限定为WebSocket的生命周期。仅在可感知网络的 Spring ApplicationContext上下文中有效。 |

### 使用Java的方式配置Spring

这种方法在springboot中随处可见

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

等价于

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

使用 `AnnotationConfigApplicationContext` 实例化 Spring 容器

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

#### @Configuration

`@Configuration`表示这是一个配置类，等价于`applicationContext.xml`

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

#### @Import的使用

`@Import(Config.class)` 允许从另一个配置类中加载@Bean定义
等价于xml的`<import>`

#### @ComponentScan

`@ComponentScan("com.test.pojo")`加在配置类上方
等价于
`<context:component-scan base-package="com.test.pojo"/>`

## AOP

AOP底层即是代理模式
【SpringAOP】和【SpringMVC】面试必问
·

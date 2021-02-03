## 一、lombok 介绍

### 简介

`lombok`产生就是为了省去我们手动创建getter和setter方法等等一些基本组件代码的麻烦，它能够在编译的时候帮助我们生成getter和setter方法。

### 常见注解

- `@EqualsAndHashCode` 注解在类，生成hashCode和equals方法
- `@slf4j`日志注解，自动生成该类的 log 静态常量，要打日志就可以直接打，不用再手动 new log 静态常量了，除了 @Slf4j 之外，lombok 也提供其他日志框架的变种注解可以用，像是 @Log、@Log4j...等，他们都是帮我们创建一个静态常量 log，只是使用的库不一样而已，SpringBoot默认支持的就是 slf4j + logback 的日志框架，所以也不用再多做啥设定，直接就可以用在 SpringBoot project上，log 系列注解最常用的就是 @Slf4j。
- `@Data`注解在类上；等于同时加上@Getter/@Setter，@ToString，@EqualsAndHashCode，@RequiredArgsConstructor。@Data 是使用频率最高的 lombok 注解，通常 @Data 会加在一个值可以被更新的对象上，像是日常使用的 DTO 、Entity ，就很适合加上 @Data 注解，也就是 @Data for mutable class

- `@Setter`：注解在属性上；为属性提供 setting 方法

- `@Getter`：注解在属性上；为属性提供 getting 方法

- `@ToString` 注解在类，添加toString方法

- `@NoArgsConstructor`：注解在类上；为类提供一个无参的构造方法

- `@AllArgsConstructor`：注解在类上；为类提供一个全参的构造方法

- `@RequiredArgsConstructor` 注解在类，为类中需要特殊处理的字段生成构造方法，比如final和被@NonNull注解的字段

- `@Cleanup`: 可以关闭流

- `@Builder`： 被注解的类加个构造者模式，自动生成流式 set 值写法，从此之后再也不用写一堆 setter 了。注意，虽然只要加上 @Builder 注解，我们就能够用流式写法快速设定对象的值，但是 setter 还是必须要写不能省略的，因为 Spring 或是其他框架有很多地方都会用到对象的 getter/setter 对他们取值/赋值

  所以通常是 @Data 和 @Builder 会一起用在同个类上，既方便我们流式写代码，也方便框架做事

- `@Synchronized` ： 加个同步锁

- `@SneakyThrows `: 等同于try/catch 捕获异常

- `@NonNull`: 如果给参数加个这个注解 参数为null会抛出空指针异常

- `@Value` : 注解和@Data类似，区别在于它会把所有成员变量默认定义为private final修饰，并且不会生成set方法。

### 依赖安装

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.12</version>
</dependency>
```

### IDEA 插件安装

如果IDEA没有安装`lombok`插件的话是会报错的。

IDEA官方插件网址：<https://plugins.jetbrains.com/>。

搜索`lombok`并下载。

![image.png](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/10-Lombok/images/bVcKGEO.png)

下载之后在`IDEA`中添加插件，步骤如下：

打开setting，选择`plugins`，选择`install plugin from disk...`，然后选择下载的`lombok`插件文件安装就可以了。

![image.png](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/10-Lombok/images/bVcKGFY.png)

<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
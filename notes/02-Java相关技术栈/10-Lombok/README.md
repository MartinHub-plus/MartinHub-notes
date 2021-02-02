## 一、lombok 介绍

### 简介

`lombok`产生就是为了省去我们手动创建getter和setter方法等等一些基本组件代码的麻烦，它能够在编译的时候帮助我们生成getter和setter方法。

### 常见注解

- `@Setter` 注解在类或字段。注解在类时为所有字段生成setter方法，注解在字段上时只为该字段生成setter方法
- `@Getter` 使用方法同`@Setter`，区别在于生成的是getter方法
- `@ToString` 注解在类，添加toString方法
- `@EqualsAndHashCode` 注解在类，生成hashCode和equals方法
- `@NoArgsConstructor` 注解在类，生成无参的构造方法
- `@RequiredArgsConstructor` 注解在类，为类中需要特殊处理的字段生成构造方法，比如final和被@NonNull注解的字段
- `@AllArgsConstructor` 注解在类，生成包含类中所有字段的构造方法
- `@Data` 注解在类，为类的所有字段注解@ToString、@EqualsAndHashCode、@Getter的便捷方法，同时为所有非final字段注解@Setter

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
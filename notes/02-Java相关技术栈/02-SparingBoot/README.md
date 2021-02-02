## 一、Spring Boot 项目结构

### （1）分层

- **表示层**：

  > controller - 负责页面访问控制，对外暴露Rest API接口。

- **业务层**：

  > service / biz：
  >
  > &emsp;服务 contract 是接口；
  >
  > &emsp;impl 是服务实现；
  >
  > entity：
  >
  > &emsp;实体 vo 是服务可对外公开的实体；
  > &emsp;dto 是数据传输对象，可在服务间传递；
  > &emsp;qo 是查询对象，可以认为是查询条件的封装；

- **持久层**：

  > dao ：数据访问对象，选择比较熟悉的 mybatis 作为 ORM 工具；
  >
  > domain ：数据对象实体 DO，通常和数据表、视图或其他业务对象一一对应；

- **公共模块**：

  > common ：公共类，如枚举，常量、业务无关的通用公共实体等；
  >
  > util ：常用实用的帮助类，如反射、字符串、集合、枚举、正则、缓存、队列等；
  >
  > config ：自定义的配置项，可从配置文件读取；

### （2）Spring 框架的重要内容

> IOC : 控制反转
>
> AOP : 面向切面编程

**注** ：Spring Boot 框架封装 spring、spring mvc 等框架之后，使用变得十分简单。

<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
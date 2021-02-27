## Java 必须掌握的 20+ 种 Spring 常用注解

**Spring部分** 

**1、声明bean的注解**

> @Component 添加到容器中
>
> @Service 在业务逻辑层使用（service层）
>
> @Repository 在数据访问层使用（dao层）
>
> @Controller 在展现层使用，控制器的声明（C）

**2、注入bean的注解**

> @Autowired：由Spring提供
>
> @Inject：由JSR-330提供
>
> @Resource：由JSR-250提供

都可以注解在set方法和属性上，推荐注解在属性上（一目了然，少写代码）。

**3、java配置类相关注解**

> @Configuration 声明当前类为配置类，相当于xml形式的Spring配置（类上）
>
> @Bean 注解在方法上，声明当前方法的返回值为一个bean，替代xml中的方式（方法上）
>
> @Configuration 声明当前类为配置类，其中内部组合了@Component注解，表明这个类是一个bean（类上）
>
> @ComponentScan 用于对Component进行扫描，相当于xml中的（类上）
>
> @WishlyConfiguration 为@Configuration与@ComponentScan的组合注解，可以替代这两个注解

**4、切面（AOP）相关注解**

Spring支持AspectJ的注解式切面编程。

> @Aspect 声明一个切面（类上）
> 使用@After、@Before、@Around定义建言（advice），可直接将拦截规则（切点）作为参数。
>
> @After 在方法执行之后执行（方法上）
> @Before 在方法执行之前执行（方法上）
> @Around 在方法执行之前与之后执行（方法上）
>
> @PointCut 声明切点

在java配置类中使用@EnableAspectJAutoProxy注解开启Spring对AspectJ代理的支持（类上）

**5、@Bean的属性支持**

> @Scope 设置Spring容器如何新建Bean实例（方法上，得有@Bean）
> 其设置类型包括：
>
> Singleton （单例,一个Spring容器中只有一个bean实例，默认模式）,
> Protetype （每次调用新建一个bean）,
> Request （web项目中，给每个http request新建一个bean）,
> Session （web项目中，给每个http session新建一个bean）,
> GlobalSession（给每一个 global http session新建一个Bean实例）
>
> @StepScope 在Spring Batch中还有涉及
>
> @PostConstruct 由JSR-250提供，在构造函数执行完之后执行，等价于xml配置文件中bean的initMethod
>
> @PreDestory 由JSR-250提供，在Bean销毁之前执行，等价于xml配置文件中bean的destroyMethod

**6、@Value注解**

> @Value 为属性注入值（属性上）

支持如下方式的注入：
》注入普通字符

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解1.PNG) 

》注入操作系统属性

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解2.PNG)

》注入表达式结果

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解3.PNG)

》注入其它bean属性

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解4.PNG)

》注入文件资源

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解5.PNG)

》注入网站资源

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解6.PNG)

》注入配置文件

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解7.PNG)

注入配置使用方法

① 编写配置文件（test.properties）

book.name=《三体》

② @PropertySource 加载配置文件(类上)

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解8.PNG)

③ 还需配置一个PropertySourcesPlaceholderConfigurer的bean。

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解9.PNG)

**7、环境切换**

> @Profile 通过设定Environment的ActiveProfiles来设定当前context需要使用的配置环境。（类或方法上）
>
> @Conditional Spring4中可以使用此注解定义条件话的bean，通过实现Condition接口，并重写matches方法，从而决定该bean是否被实例化。（方法上）

**8、异步相关**

> @EnableAsync 配置类中，通过此注解开启对异步任务的支持，叙事性AsyncConfigurer接口（类上）
>
> @Async 在实际执行的bean方法使用该注解来申明其是一个异步任务（方法上或类上所有的方法都将异步，需要@EnableAsync开启异步任务）

**9、定时任务相关**

> @EnableScheduling 在配置类上使用，开启计划任务的支持（类上）
>
> @Scheduled 来申明这是一个任务，包括cron,fixDelay,fixRate等类型（方法上，需先开启计划任务的支持）

**10、@Enable\*注解说明**

这些注解主要用来开启对xxx的支持。

> @EnableAspectJAutoProxy 开启对AspectJ自动代理的支持
>
> @EnableAsync 开启异步方法的支持
>
> @EnableScheduling 开启计划任务的支持
>
> @EnableWebMvc 开启Web MVC的配置支持
>
> @EnableConfigurationProperties 开启对@ConfigurationProperties注解配置Bean的支持
>
> @EnableJpaRepositories 开启对SpringData JPA Repository的支持
>
> @EnableTransactionManagement 开启注解式事务的支持
>
> @EnableTransactionManagement 开启注解式事务的支持
>
> @EnableCaching 开启注解式的缓存支持

**11、测试相关注解**

@RunWith 运行器，Spring中通常用于对JUnit的支持

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解10.PNG)

@ContextConfiguration 用来加载配置ApplicationContext，其中classes属性用来加载配置类

![img](https://gitee.com/MartinHub/MartinHub-notes/raw/master/notes/02-Java相关技术栈/日常记录/images/注解11.PNG)

**SpringMVC部分** 

> @EnableWebMvc 在配置类中开启Web MVC的配置支持，如一些ViewResolver或者MessageConverter等，若无此句，重写WebMvcConfigurerAdapter方法（用于对SpringMVC的配置）。
>
> @Controller 声明该类为SpringMVC中的Controller
>
> @RequestMapping 用于映射Web请求，包括访问路径和参数（类或方法上）
>
> @ResponseBody 支持将返回值放在response内，而不是一个页面，通常用户返回json数据（返回值旁或方法上）
>
> @RequestBody 允许request的参数在request体中，而不是在直接连接在地址后面。（放在参数前）
>
> @PathVariable 用于接收路径参数，比如@RequestMapping(“/hello/{name}”)申明的路径，将注解放在参数中前，即可获取该值，通常作为Restful的接口实现方法。
>
> @RestController 该注解为一个组合注解，相当于@Controller和@ResponseBody的组合，注解在类上，意味着，该Controller的所有方法都默认加上了@ResponseBody。
>
> @ControllerAdvice 通过该注解，我们可以将对于控制器的全局配置放置在同一个位置，注解了@Controller的类的方法可使用@ExceptionHandler、@InitBinder、@ModelAttribute注解到方法上，
> 这对所有注解了 @RequestMapping的控制器内的方法有效。
>
> @ExceptionHandler 用于全局处理控制器里的异常
>
> @InitBinder 用来设置WebDataBinder，WebDataBinder用来自动绑定前台请求参数到Model中。
>
> @ModelAttribute 本来的作用是绑定键值对到Model里，在@ControllerAdvice中是让全局的@RequestMapping 都能获得在此处设置的键值对。

<br/>

<br/>

## Spring Boot 项目结构

**（1）分层** 

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
  >
  > &emsp;dto 是数据传输对象，可在服务间传递；
  >
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

**（2）Spring 框架的重要内容** 

> IOC : 控制反转
>
> AOP : 面向切面编程

**注** ：Spring Boot 框架封装 spring、spring mvc 等框架之后，使用变得十分简单。

**（3）注解说明** 

> @RestController = @Controller + @ResponseBody；
>
> &emsp;方法返回 object，会转化为 json 格式；
>
> @RequestBody: 从请求体中获取数据；
>
> @RequestParam: 从请求头中获取数据；
>
> 

## 配置文件

```yaml
<!--导入配置文件处理器，配置文件绑定就会有输入提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

(注意： 配置好了重新启动main)

**application.yml写入配置**  

```yaml
server:
  port: 8081
 
person:
  lastName: zhangsan
  age: 18
  boss: false
  birth: 2017/12/12
  maps: {k1: v1,k2: 12}
  lists:
    - list
    - zhaoliu
  dog:
    name: 小狗
    age: 2
```

**@Component 添加到容器中**

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import java.util.Date;
import java.util.List;
import java.util.Map;
import lombok.Data;
 
@Component
@ConfigurationProperties(prefix = "person")
@Data
public class Person {
 
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;
 
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
 
    @Override
    public String toString() {
        return "Person{" +
                "lastName='" + lastName + '\'' +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", lists=" + lists +
                ", dog=" + dog +
                '}';
    }
}
```

## JSR 303 基本的校验规则

**空检查** 

> @Null 验证对象是否为null 
>
> @NotNull 验证对象是否不为null, 无法查检长度为0的字符串 
>
> @NotBlank 检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格. 
>
> @NotEmpty 检查约束元素是否为NULL或者是EMPTY.

 **Booelan检查**  

> @AssertTrue 验证 Boolean 对象是否为 true 
>
> @AssertFalse 验证 Boolean 对象是否为 false长度检查 
>
> @Size(min=, max=) 验证对象（Array,Collection,Map,String）长度是否在给定的范围之内 
>
> @Length(min=, max=) Validates that the annotated string is between min and max included.

 **日期检查** 

> @Past 验证 Date 和 Calendar 对象是否在当前时间之前，验证成立的话被注释的元素一定是一个过去的日期 
>
> @Future 验证 Date 和 Calendar 对象是否在当前时间之后 ，验证成立的话被注释的元素一定是一个将来的日期 
>
> @Pattern 验证 String 对象是否符合正则表达式的规则，被注释的元素符合制定的正则表达式，regexp:正则表式 flags: 指定 Pattern.Flag 的数组，表示正则表达式的相关选项。

**数值检查** 

建议使用在Stirng,Integer类型，不建议使用在int类型上，因为表单值为“”时无法转换为int，但可以转换为Stirng为”“,Integer为null 

> @Min 验证 Number 和 String 对象是否大等于指定的值 
>
> @Max 验证 Number 和 String 对象是否小等于指定的值 
>
> @DecimalMax 被标注的值必须不大于约束中指定的最大值. 这个约束的参数是一个通过BigDecimal定义的最大值的字符串表示.小数存在精度 
>
> @DecimalMin 被标注的值必须不小于约束中指定的最小值. 这个约束的参数是一个通过BigDecimal定义的最小值的字符串表示.小数存在精度 
>
> @Digits 验证 Number 和 String 的构成是否合法 
>
> @Digits(integer=,fraction=) 验证字符串是否是符合指定格式的数字，interger指定整数精度，fraction指定小数精度。 
>
> @Range(min=, max=) 被指定的元素必须在合适的范围内 
>
> @Range(min=10000,max=50000,message=”range.bean.wage”) 
>
> @Valid 递归的对关联对象进行校验, 如果关联对象是个集合或者数组,那么对其中的元素进行递归校验,如果是一个map,则对其中的值部分进行校验.(是否进行递归验证) 
>
> @CreditCardNumber信用卡验证 
>
> @Email 验证是否是邮件地址，如果为null,不进行验证，算通过验证。 
>
> @ScriptAssert(lang= ,script=, alias=) 
>
> @URL(protocol=,host=, port=,regexp=, flags=)



<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
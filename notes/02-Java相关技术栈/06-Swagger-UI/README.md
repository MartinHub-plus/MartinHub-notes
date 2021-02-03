## 一、Swagger使用介绍

### （1）**Swagger依赖信息**:

```properties
<!--接口文档生成工具swagger-->
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger2</artifactId>
   <version>2.9.2</version>
</dependency>
<!--接口文档生成工具界面swagger-ui-->
<dependency>
   <groupId>io.springfox</groupId>
   <artifactId>springfox-swagger-ui</artifactId>
   <version2.9.2</version>
</dependency>
<!—以下两个依赖用于生成文档，自定义注解使用-->
<!--实现Swagger导出离线API文档-->
<dependency>
   <groupId>io.github.swagger2markup</groupId>
   <artifactId>swagger2markup</artifactId>
   <version>1.3.3</version>
   <scope>test</scope>
</dependency>
<!--Swagger 自定义map参数-->
<dependency>
   <groupId>org.javassist</groupId>
   <artifactId>javassist</artifactId>
   <version>3.21.0-GA</version>
</dependency>

```

### （2）Swagger基本注解介绍

#### > @EnableSwagger2 

> 开启Swagger 
>
> @EnableSwagger2：放在启动类上或Swagger配置类上。

举例1：

```java
@EnableSwagger2
public class SwaggerApplication {
   public static void main(String[] args) {
      SpringApplication.run(SwaggerApplication.class, args);
   }
}
```

举例2：

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    // 定义分隔符,配置Swagger多包
    private static final String splitor = ";";
    @Bean
    public Docket buildDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                //将api的元素设置为包含在json resource listing响应中
                .apiInfo(buildApiInfo())
                //设置ip和端口,或者域名
                //.host("127.0.0.1:8080")
                //选择那些路径和api会生成document
                .select()
                //要扫描的API(Controller)基础包(多个包用 ; 隔开)
                .apis(basePackage("com.erp.user.storage.controller"
                        + splitor + "com.erp.user.preview.controller"
                        + splitor + "com.erp.user.swaggerDemo.controller"
                        ))
                //错误路径不监控
                .paths(Predicates.not(PathSelectors.regex("/error.*")))
                //对根下所有路径进行监控
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo buildApiInfo() {
        return new ApiInfoBuilder()
                //文档标题
                .title("用户服务API")
                //文档描述
                .description("用户服务调试接口文档")
                //联系人
               // .contact(new Contact("name", "url", "email"))
                //版本号
                .version("0.0.1")
                //更新此API的许可证信息
                //.license("Apache 2.0")
                //更新此API的许可证Url
                //.licenseUrl("http://www.apache.org/licenses/LICENSE-2.0.html")
                //更新服务条款URL
                //.termsOfServiceUrl("NO terms of service")
                .build();
    }

    /**
     * @param basePackage
     * @description 重写basePackage方法，使能够实现多包访问，复制贴上去
     * @returen com.google.common.base.Predicate<springfox.documentation.RequestHandler>
     */
    public static Predicate<RequestHandler> basePackage(final String basePackage) {
        return input -> declaringClass(input).transform(handlerPackage(basePackage)).or(true);
    }

    private static Function<Class<?>, Boolean> handlerPackage(final String basePackage) {
        return input -> {
            // 循环判断匹配
            for (String strPackage : basePackage.split(splitor)) {
                boolean isMatch = input.getPackage().getName().startsWith(strPackage);
                if (isMatch) {
                    return true;
                }
            }
            return false;
        };
    }

    private static Optional<? extends Class<?>> declaringClass(RequestHandler input) {
        return Optional.fromNullable(input.declaringClass());
    }
}
```

#### > @Api 

> 请求类的说明
>
> @Api：放在请求的类上，与 @Controller 并列，说明类的作用，如用户模块，用户类等。
>
> tags="说明该类的作用"
>
> value="该参数没什么意义，所以不需要配置"
>
> description = "用户基本信息操作"

举例：

```java
@Api(value = "用户Controller",tags = "用户信息测试")
@RestController
@RequestMapping("/user")
public class UserController {
//TODO
}
```

#### > @ApiOperation

> 方法的说明
>
> @ApiOperation："用在请求的方法上，说明方法的作用"
>
> value="说明方法的作用"
>
> notes="方法的备注说明"
>
> tags="说明该方法的作用，参数是个数组，可以填多个"
>
> 格式：tags={"作用1","作用2"}
>
> httpMethod="设置显示的请求方式GET、POST、PUT、DELETE"

举例：

```java
@ApiOperation(value = "查询用户信息",notes = "方法的备注说明，如果有可以写在这里",httpMethod = "GET")
@RequestMapping(value = "/findUser")
public User findUser(@RequestParam String userId){
    //TODO
}
```

#### > @ApiImplicitParams / @ApiImplicitParam

> 方法参数的说明
>
> @ApiImplicitParams：用在请求的方法上，包含一组参数说明
>
> &emsp; @ApiImplicitParam：对单个参数的说明      
>
> &emsp;&emsp;&emsp;name：参数名
>
> &emsp;&emsp;&emsp;value：参数的汉字说明、解释
>
> &emsp;&emsp;&emsp;required：参数是否必须传
>
> &emsp;&emsp;&emsp;paramType：参数放在哪个地方
>
> &emsp;&emsp;&emsp;&emsp;· header --> 请求参数的获取：@RequestHeader
>
> &emsp;&emsp;&emsp;&emsp;· query --> 请求参数的获取：@RequestParam
>
> &emsp;&emsp;&emsp;&emsp;· path（用于restful接口）--> 请求参数的获取：@PathVariable
>
> &emsp;&emsp;&emsp;&emsp;· body（请求体）-->  @RequestBody User user
>
> &emsp;&emsp;&emsp;&emsp;· form（普通表单提交）     
>
> &emsp;&emsp;&emsp;dataType：参数类型，默认String，其它值dataType="int"       
>
> &emsp;&emsp;&emsp;defaultValue：参数的默认值

举例：多个参数

```java
@ApiOperation(value = "查询用户",notes = "查询用户的所有信息",httpMethod = "POST")
@ApiImplicitParams({
        @ApiImplicitParam(name = "userId",value = "用户Id",dataType = "String",paramType = "form",required = true),
        @ApiImplicitParam(name = "userName",value = "用户名",dataType = "String",paramType = "form",required = true),
        @ApiImplicitParam(name = "password",value = "密码",dataType = "String",paramType = "form",required = true),})
@RequestMapping("/findList")
public List<User> findList(String userId, String userName, String password,Integer age){
//TODO
}
```

举例：单个参数

```java
@ApiOperation(value = "查询用户信息",notes = "根据id查询用户信息",httpMethod = "GET")
@ApiImplicitParam(name = "userId",value = "用户id",paramType = "query",dataType = "String",example = "根据id查询用户",required = true)
@RequestMapping(value = "/findUser")
public User findUser(@RequestParam String userId){
    //TODO
}
```

#### > @ApiResponses / @ApiResponse 

> 方法返回值的说明
>
> @ApiResponses：方法返回对象的说明
>
> &emsp;@ApiResponse：每个参数的说明
>
> &emsp;&emsp;code：数字，例如400
>
> &emsp;&emsp;message：信息，例如"请求参数没填好"
>
> &emsp;&emsp;response：抛出异常的类

举例：

```java
@ApiOperation(value = "修改密码", notes = "方法的备注说明，如果有可以写在这里",httpMethod = "POST")
@ApiResponses({
        @ApiResponse(code = 400, message = "请求参数没填好"),
        @ApiResponse(code = 404, message = "请求路径找不到")
})
@RequestMapping(value = "/findUser")
public User findUser(@RequestParam String userId){
    //TODO
}

```

#### > @ApiModel 

> 用于JavaBean上面，表示一个JavaBean
>
> @ApiModel：用于JavaBean的类上面，表示此 JavaBean 整体的信息（这种一般用在post创建的时候，使用 @RequestBody 这样的场景，请求参数无法使用 @ApiImplicitParam 注解进行描述的时候 ）

举例：

```java
@ApiModel("用户信息类")
public class User {
    @ApiModelProperty(value="用户id",name="id",dataType = "Integer",example="用户id",required = true)
    private String id;
}
```

#### > @ApiModelProperty 

> 用在JavaBean的属性上面，说明属性含义
>
> @ApiModelProperty：用在属性上，描述实体类的属性
>
> &emsp;value="用户名" 描述参数的意义
>
> &emsp;name="name" 参数的变量名
>
> &emsp;example="举例说明 "
>
> &emsp;hidden="true" 隐藏
>
> &emsp;dataType="重写属性类型"
>
> &emsp;required=true 参数是否必填

举例：

```java
@ApiModel("用户信息类")
public class User {
    @ApiModelProperty(value="用户id",name="id",dataType = "Integer",example="用户id",required = true)
    private String id;
    @ApiModelProperty(value="用户名",name="userName")
    private String username;
    @ApiModelProperty("密码")
    private String password;

```

#### > @ApiParam 

> 用于方法，参数，字段说明 表示对参数的要求和说明
>
> @ApiParam：用在属性上，描述实体类的属性
>
> &emsp;name="参数名称"
>
> &emsp;value="参数的简要说明"
>
> &emsp;defaultValue="参数默认值"
>
> &emsp;required="true" 表示属性是否必填，默认为false

举例：

```java
public User addUser(@ApiParam(value = "新增用户参数", required = true) @RequestBody User param) {
//TODO
}

```

#### > @ApiIgnore

> 用于类或者方法上，不被显示在页面上

#### > @Profile({“dev”, “test”})

> 用于配置类上，表示只对开发和测试环境有用



<br/>

<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
## Druid数据源

### 1.Druid数据源是什么？

Druid是阿里巴巴开源的一个数据源，主要用于java数据库连接池，相比spring推荐的DBCP和hibernate推荐的C3P0、Proxool数据库连接池，Druid在市场上占有绝对的优势；

### 2.为什么选择Druid作为数据库连接池？

这里直接给出一个链接：

[大话数据库连接池](https://links.jianshu.com/go?to=http%3A%2F%2Frainbowhorse.site%2F2018%2F02%2F06%2F%25E5%25A4%25A7%25E8%25AF%259D%25E6%2595%25B0%25E6%258D%25AE%25E5%25BA%2593%25E8%25BF%259E%25E6%258E%25A5%25E6%25B1%25A0%2F)

文章从市场占有率、性能上比较C3P0、DBCP、HikariCP和Druid，说明了Druid数据源由于有强大的监控特性、可拓展性等特点值得作者推荐。虽说 HikariCP 的性能比 Druid 高，但是因为 Druid 包括很多维度的统计和分析功能，所以大家都选择使用Druid 的更多；

### 3.如何结合SpringBoot使用？

要先知道如何使用一个工具，当然首先要从文档开始，关于文档可以从官网知道，可以参考[https://github.com/alibaba/druid](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2Fdruid)；

(1) 添加Druid依赖

```pom
<properties>
	<druid-version>1.1.10</druid-version>
</properties>
<dependencies>
    ......
    <!--alibaba druid datasource-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>${druid-version}</version>
    </dependency>
    ......
</dependencies>
```

(2) 添加配置

springboot支持yml和properties等配置文件，本文采用application.properties配置。

- application.properties

  ```properties
  # DataSource settings
  spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
  spring.datasource.url = jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF8
  spring.datasource.username = root
  spring.datasource.password = root
  spring.datasource.driverClassName = com.mysql.jdbc.Driver
  #连接池的配置信息
  spring.datasource.initialSize=5
  spring.datasource.minIdle=5
  spring.datasource.maxActive=20
  # 配置获取连接等待超时的时间
  spring.datasource.maxWait=60000
  # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
  spring.datasource.timeBetweenEvictionRunsMillis=60000
  # 配置一个连接在池中最小生存的时间，单位是毫秒
  spring.datasource.minEvictableIdleTimeMillis=300000
  spring.datasource.validationQuery=SELECT 1 FROM DUAL
  spring.datasource.testWhileIdle=true
  spring.datasource.testOnBorrow=false
  spring.datasource.testOnReturn=false
  # 打开PSCache，并且指定每个连接上PSCache的大小
  spring.datasource.poolPreparedStatements=true
  spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
  # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
  spring.datasource.filters=stat,wall,log4j
  # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
  spring.datasource.connectionProperties=druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000

  ```

- druid-monitor.properties

  ```properties
  #是否启用StatFilter默认值true
  spring.datasource.druid.web-stat-filter.enabled=true
  #多个白名单IP以逗号分隔
  druid.monitor.allow=127.0.0.1
  #多个黑名单IP以逗号分隔
  druid.monitor.deny=0.0.0.0
  #druid监控管理界面登录帐号
  druid.monitor.loginUsername=admin
  #druid监控管理界面登录密码
  druid.monitor.loginPassword=password
  #是否开启重置功能
  druid.monitor.resetEnable=false

  ```

  要知道具体配置有啥影响的话，我找了一篇采坑的文章，可以看一下：

  [使用druid连接池带来的坑testOnBorrow=false](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Flx348321409%2Farticle%2Fdetails%2F76095751)

  (3) 添加配置类

  为方便以后拓展，这里提供一个数据源配置接口，druid配置也只是这个接口的一个实现类，方便以后切换不同的数据源；

  - DbConfig.java

    ```java
    package com.ijustone.service.core.db.config;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.springframework.jdbc.datasource.DataSourceTransactionManager;
    import javax.sql.DataSource;
    public interface DbConfig {

        /**
         * 定义数据源
         *
         * @return
         * @throws Exception
         */
        DataSource  dataSource() throws Exception;

        /**
         * 定义session工厂
         *
         * @param dataSource
         * @return
         * @throws Exception
         */
        SqlSessionFactory sessionFactory(DataSource dataSource) throws Exception;

        /**
         * 定义失误管理器
         *
         * @param dataSource
         * @return
         */
        DataSourceTransactionManager transactionManager(DataSource dataSource);

    }
    ```

  下面是druid配置实现类

  - MyDruirdConfig.java

    ```java
    package com.ijustone.service.core.db.config.impl;

    import com.alibaba.druid.pool.DruidDataSource;
    import com.ijustone.service.core.db.config.DbConfig;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.mybatis.spring.SqlSessionFactoryBean;
    import org.mybatis.spring.annotation.MapperScan;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Primary;
    import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
    import org.springframework.jdbc.datasource.DataSourceTransactionManager;
    import org.springframework.transaction.annotation.EnableTransactionManagement;

    import javax.sql.DataSource;

    @Configuration
    @EnableTransactionManagement
    @MapperScan(basePackages = MyDruirdConfig.PACKAGE, sqlSessionFactoryRef = "sessionFactory")
    public class MyDruirdConfig implements DbConfig {
        public static final String PACKAGE = "com.ijustone.service.**.mapper";

        public static final String MAPPER = "classpath:com/ijustone/service/**/mapper/**/*Mapper.xml";

        @Value("${spring.datasource.url}")
        private String dbUrl;

        @Value("${spring.datasource.username}")
        private String username;

        @Value("${spring.datasource.password}")
        private String password;

        @Value("${spring.datasource.driverClassName}")
        private String driverClassName;

        @Value("${spring.datasource.initialSize}")
        private int initialSize;

        @Value("${spring.datasource.minIdle}")
        private int minIdle;

        @Value("${spring.datasource.maxActive}")
        private int maxActive;

        @Value("${spring.datasource.maxWait}")
        private int maxWait;

        @Value("${spring.datasource.testWhileIdle:true}")
        private boolean testWhileIdle;

        @Value("${spring.datasource.timeBetweenEvictionRunsMillis:60000}")
        private int timeBetweenEvictionRunsMillis;

        @Value("${spring.datasource.validationQuery}")
        private String validationQuery;

        /**
         * 指明是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个.<br/>
         * 注意: 设置为true后如果要生效,validationQuery参数必须设置为非空字符串
         */
        @Value("${spring.datasource.testOnBorrow:true}")
        private boolean testOnBorrow;

        /**
         * 指明是否在归还到池中前进行检验<br/>
         * 注意: 设置为true后如果要生效,validationQuery参数必须设置为非空字符串
         */
        @Value("${spring.datasource.testOnReturn:false}")
        private boolean testOnReturn;

        @Value("${spring.datasource.minEvictableIdleTimeMillis:300000}")
        private int minEvictableIdleTimeMillis;

        /**
         * 当开启时, 将为每个连接创建一个statement池,并且被方法创建的PreparedStatements将被缓存起来:
         */
        @Value("${spring.datasource.poolPreparedStatements:false}")
        private boolean poolPreparedStatements;

        /**
         * 不限制  statement池能够同时分配的打开的statements的最大数量,如果设置为0表示不限制
         */
        @Value("${spring.datasource.maxOpenPreparedStatements:10}")
        private int maxPoolPreparedStatementPerConnectionSize;

        @Value("${spring.datasource.defaultAutoCommit:false}")
        private boolean defaultAutoCommit;

        @Value("${spring.datasource.filters:stat}")
        private String filters;

        /**
         * 当建立新连接时被发送给JDBC驱动的连接参数
         */
        @Value("${spring.datasource.connectionProperties:druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000}")
        private String connectionProperties;

        /**
         * 定义数据源
         * 注意@Primary注解表示：自动装配时当出现多个Bean候选者时，被注解为@Primary的Bean将作为首选者，否则将抛出异常
         *
         * @return
         * @throws Exception
         */
        @Bean(name = "dataSource")
        @Primary
        @Override
        public DataSource dataSource() throws Exception {
            DruidDataSource datasource = new DruidDataSource();

            datasource.setUrl(this.dbUrl);
            datasource.setUsername(this.username);
            datasource.setPassword(this.password);
            datasource.setDriverClassName(this.driverClassName);

            datasource.setInitialSize(this.initialSize);
            datasource.setMinIdle(this.minIdle);
            datasource.setMaxActive(this.maxActive);
            datasource.setMaxWait(this.maxWait);
            datasource.setTimeBetweenEvictionRunsMillis(this.timeBetweenEvictionRunsMillis);
            datasource.setMinEvictableIdleTimeMillis(this.minEvictableIdleTimeMillis);
            datasource.setValidationQuery(this.validationQuery);
            datasource.setTestWhileIdle(this.testWhileIdle);
            datasource.setTestOnBorrow(this.testOnBorrow);
            datasource.setTestOnReturn(this.testOnReturn);
            datasource.setPoolPreparedStatements(this.poolPreparedStatements);
            datasource.setMaxPoolPreparedStatementPerConnectionSize(this.maxPoolPreparedStatementPerConnectionSize);
            datasource.setDefaultAutoCommit(this.defaultAutoCommit);
            datasource.setFilters(this.filters);
            datasource.setConnectionProperties(this.connectionProperties);
            return datasource;
        }

        /**
         * 定义session工厂
         * 注：ualifier的意思是合格者，通过这个标示，表明了哪个实现类才是我们所需要的，
         * 我们修改调用代码，添加@Qualifier注解，需要注意的是@Qualifier的参数名称必须为我们之前定义@Service注解的名称之一！
         *
         * @param dataSource
         * @return
         * @throws Exception
         */
        @Bean(name = "sessionFactory")
        @Primary
        @Override
        public SqlSessionFactory sessionFactory(@Qualifier("dataSource") DataSource dataSource) throws Exception {
            SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
            sessionFactory.setDataSource(dataSource);

            PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

            sessionFactory.setMapperLocations(resolver.getResources(MyDruirdConfig.MAPPER));
            return sessionFactory.getObject();
        }

        /**
         * 定义事务管理器
         *
         * @param dataSource
         * @return
         */
        @Bean(name = "transactionManager")
        @Override
        public DataSourceTransactionManager transactionManager(@Qualifier("dataSource") DataSource dataSource) {
            return new DataSourceTransactionManager(dataSource);
        }
    }
    ```

  我们配置了Druid的监听器

  - DruidMonitorConfiguration.java

    ```java
    package com.ijustone.service.core.druid.monitor;

    import com.alibaba.druid.support.http.StatViewServlet;
    import com.alibaba.druid.support.http.WebStatFilter;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.boot.web.servlet.FilterRegistrationBean;
    import org.springframework.boot.web.servlet.ServletRegistrationBean;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.PropertySource;

    @Configuration
    @PropertySource(value = "classpath:config/druid-monitor.properties")
    @ConfigurationProperties
    public class DruidMonitorConfiguration {

        @Value("${druid.monitor.allow:127.0.0.1}")
        private String allow;
        @Value("${druid.monitor.deny}")
        private String deny;
        @Value("${druid.monitor.loginUsername:admin}")
        private String loginUsername;
        @Value("${druid.monitor.loginPassword:password}")
        private String loginPassword;
        @Value("${druid.monitor.resetEnable:false}")
        private String resetEnable;

        @Bean
        public ServletRegistrationBean druidStatViewServlet() {
            ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
            servletRegistrationBean.addInitParameter("allow", this.allow);
            servletRegistrationBean.addInitParameter("deny", this.deny);
            servletRegistrationBean.addInitParameter("loginUsername", this.loginUsername);
            servletRegistrationBean.addInitParameter("loginPassword", this.loginPassword);
            servletRegistrationBean.addInitParameter("resetEnable", this.resetEnable);
            return servletRegistrationBean;
        }

        @Bean
        public FilterRegistrationBean druidStatFilter() {
            FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
            filterRegistrationBean.addUrlPatterns("/*");
            filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
            return filterRegistrationBean;
        }
    }
    ```

  ### 4.如何查看监控页面？

  访问[http://localhost:8010/druid](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8010%2Fdruid)就可以了，如果工程集成了SpringSecurity等权限工程的话是需要额外配置的，这里给一个踩坑链接:([https://www.2cto.com/kf/201803/731528.html](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.2cto.com%2Fkf%2F201803%2F731528.html)).

<div align="center"> <img  src="https://gitee.com/MartinHub/MartinHub-notes/raw/master/images/weixin.png" width="200"/> </div>
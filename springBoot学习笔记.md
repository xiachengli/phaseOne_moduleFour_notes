##### 简介

	###### 约定优于配置

约定优于配置，也称按约定编程，是一种软件设计范式，旨在减少软件开发人员需做决定的数量，获得简单的好处而不失灵活性

从框架提供的约定（功能）来说，如果框架的约定（功能）满足你的预期那就不必再进行配置，否，可自定义配置使功能满足预期。开发人员只需规定应用中不符约定的部分，减少了配置文件的编写

###### SpringBoot基础

目的：简化spring应用程序的开发

spring的缺点

- 大量配置
- 项目的依赖管理：在环境搭建时，不仅需要分析要导入库的坐标，还要考虑与导入库有依赖关系的库坐标

SpringBoot解决了spring的问题

- 起步依赖

  简单来说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的实现。本质上是一个Maven项目对象模型，定义了对其他库的传递依赖

- 自动配置

  我们只需引入需要的包，不需要进行配置，springBoot会自动

  帮我们将bean注册进ioc容器a

优点：简单、快速搭建项目；对主流开发框架的无配置集成；极大的提高了开发，部署效率；

###### 单元测试与热部署

单元测试

- 引入maven坐标

  ```xml
  <dependency>  <groupId>org.springframework.boot</groupId>  <artifactId>spring-boot-starter-test</artifactId>  <scope>test</scope> </dependency>
  
  ```

  (使用Spring Initializer搭建的项目，自动引入了spring-boot-starter-test 测试依赖启动器)

- 编写单元测试类和启动方法

  ```java
  /**
  *@RunWith用于指定Junit运行环境，是junit提供给其他框架测试环境接口扩展
  *为了便于使用spring的依赖注入，spring提供了SpringJUnit4ClassRunner作为Junit测试环境
  *SpringRunner继承了SpringJUnit4ClassRunner
  */
  @RunWith(SpringRunner.class) 
  @SpringBootTest  
  class SpringbootDemoApplicationTests {
      
     @Autowired   
     private DemoController demoController;
      
  ֺ   @Test   
      void contextLoads() {      
      	String demo = demoController.demo();      
      	System.out.println(demo);   
  	}
  }
  ```

热部署

- 引入Maven坐标

  ```xml
  <!--热部署：开发过程中，修改代码后，无需手动重启项目-->
  <dependency>   
      <groupId>org.springframework.boot</groupId>   
      <artifactId>spring-boot-devtools</artifactId> 
  </dependency>
  ```

  （引入包之后可能并没有效果，具体请查看相关IDEA的配置）

###### 全局配置文件

SpringBoot默认会加载src/main/resources目录或者类路径的config目录下的下列三个文件作为全局配置文件（按顺序加载）

- application.yml
- application.yaml
- application.properties

###### 属性值的注入

- @ConfiguraProperties

- @Value

  ```java
  @Component //1 放入IOC容器
  @ConfigurationProperties(prefix = "person") //2 批量设置
  public class Person {  
      private int id;          
         
      public void setId(int id) {     //3 提供setter()方法
          this.id = id;    
      } 
  }
   
  //@Value不仅可以注入属性，还可以直接赋值
  @Component //1 放入IOC容器
  public class Person { 
      @Value("${person.id}")    //2EL表达式拿到值
      private int id;      
  }
  
  ```

###### 自定义配置

自定义配置文件：SpringBoot无法识别我们自定义的配置文件，所以需要@PropertySource加载配置文件

自定义配置类：使用@Configuration注解自定义一个配置类，SpringBoot会自动扫描和识别配置类

```java
//如果这里使用@Componen注解 可以省略@EnableConﬁgurationProperties 
@Configuration    // 配置类
 @PropertySource("classpath:test.properties")   // 自定义配置文件位置及名称
 @EnableConfigurationProperties(MyProperties.class) // 开启属性注入功能
 @ConfigurationProperties(prefix = "test")      // 属性注入  
 public class MyProperties {      
     private int id;      
     private String name;      
     //省略setter()方法
 }

  @Configuration  //配置类
  public class MyConfig {     
  @Bean        //将方法返回结果放入IOC容器，bean的id默认为方法名   
  public MyService myService(){          
          return new MyService();      
      } 
  }

```



###### 随机数设置

随机数设置

 SpringBoot内嵌了RandomValuePropertySource类用于随机值注入

 语法：${random.xx}，xx指定随机数类型和范围

```properties
my.secret=${random.value}         // 随机值 
my.number=${random.int}           // 随机整数 
my.bignumber=${random.long}      // 随机long型整数
my.uuid=${random.uuid}            // 随机uuid 
my.number.less.than.ten=${random.int(10)}  //小于10的随机整数
my.number.in.range=${random.int[1024,65536]} //范围在[1024,65536]之间的随机整数
```



---

##### 原理深入及源码刨析

###### 依赖管理

版本号：spring-boot-starter-parent作为父项目管理依赖，底层有一个父依赖spring-boot-dependencies，查看dependencies源文件如下，可以看到该文件将常用技术框架的版本号进行了统一管理

```xml
<properties>  
    <activemq.version>5.15.11</activemq.version>...
    <solr.version>8.2.0</solr.version>    
    <mysql.version>8.0.18</mysql.version>    
    <kafka.version>2.3.1</kafka.version>   
    <spring-amqp.version>2.2.2.RELEASE</spring-amqp.version>    
    <spring-restdocs.version>2.0.4.RELEASE</spring-restdocs.version>   
    <spring-retry.version>1.2.4.RELEASE</spring-retry.version>    
    <spring-security.version>5.2.1.RELEASE</spring-security.version>   
    <spring-session-bom.version>Corn-RELEASE</spring-session-bom.version>    
    <spring-ws.version>3.0.8.RELEASE</spring-ws.version>    
    <sqlite-jdbc.version>3.28.0</sqlite-jdbc.version>    
    <sun-mail.version>${jakarta-mail.version}</sun-mail.version>   
    <tomcat.version>9.0.29</tomcat.version>   
    <thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>    
    <thymeleaf-extras-data-attribute.version>2.0.1</thymeleaf-extras-dataattribute.version>    ...  
</properties>

```

jar包：场景启动器，提供了开发某一功能所需的jar包。如开发web项目，只需引入spring-boot-starter-web启动器，不需要人为导入tomcat，servlet等等

###### 自动配置

找到springBoot的入口方法，可以看到类上有@SpringBootApplication这个注解，该注解能够扫描spring组件并自动配置springBoot

@SpringBootApplciation是一个组合注解

```java
@Target({ElementType.TYPE})//适用范围：类、接口、注解或枚举
@Retention(RetentionPolicy.RUNTIME)//生命周期：运行时
@Documented//表明该注解可以记录在javadoc中
@Inherited//可被子类继承
@SpringBootConfiguration//标明该类是一个配置类
@EnableAutoConfiguration//开启自动配置功能
@ComponentScan(//包扫描
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
}


//@SpringBootConfiguration
@Target({ElementType.TYPE}) @Retention(RetentionPolicy.RUNTIME) @Documented @Configuration //配置类，可被组件扫描器扫描
public @interface SpringBootConfiguration { }

/**
*@EnableAutoConfiguration借助@Import来收集所有符合自动配*置条件的bean定义，并加载到IOC容器
*/
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage //自动配置包
@Import({AutoConfigurationImportSelector.class})//导入自动配置类扫描
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}

/**
*@AutoConfigurationPackage将主程序类所在的包及其子包下的组*件扫描到spring容器中
*/
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})//向容器中导入Register组件类
public @interface AutoConfigurationPackage {
}

/***
*@Import({AutoConfigurationImportSelector.class})
*将AutoConfigurationImportSelector组件导入到IOC容器中
*该类可以帮助springboot应用将所有符合条件的@Configuration
*配置都加载到IOC容器中
*
*selectImports()告诉springBott需要导入哪些组件
*selectImports（0方法执行的过程中，会使用内部工具类*SpringFactoriesLoader,查找classpath下所有jar包中的
*META-INF/spring.factories,然后通过反射生成配置类并放入IOC容器中
*/

//@ComponentScan包扫描器
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    @AliasFor("basePackages")
    String[] value() default {};
}

```

###### 自定义starter

- 引入Maven坐标

  ```xml
  <dependencies>    
      <dependency>        <groupId>org.springframework.boot</groupId> 
          <artifactId>spring-boot-autoconfigure</artifactId>        <version>2.2.2.RELEASE</version>    </dependency> </dependencies>
  
  ```

- 编写JavaBean、编写配置类

  ```java
  @EnableConfigurationProperties(SimpleBean.class) 
  @ConfigurationProperties(prefix = "simplebean") 
  public class SimpleBean {
      private int id;    
      private String name;
      public int getId() {        return id;    }
      public void setId(int id) {        this.id = id;    }
      public String getName() {        return name;    }
      public void setName(String name) {        this.name = name;    }
      @Override    
      public String toString() {        
          return "SimpleBean{" +                
              "id=" + id +                ",
              name='" + name + '\'' +                '}';    } 
  }
  
  @Configuration
  @ConditionalOnClass 
  public class MyAutoConfiguration {
      static {        System.out.println("MyAutoConfiguration init....");    }
      @Bean    
      public SimpleBean simpleBean(){        
          return new SimpleBean();    
      }
  }
  
  
  ```

  

- resources下创建META-INF/spring.factories

  ```xml
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\配置类的全限定类名
  
  ```

  

###### 执行原理

主程序类的main()方法中有一个run方法，SpringApplication.run()方法内部执行了两个操作，分别是SpringAppliaction实例化和调用run()方法启动项目

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {  
    return (new SpringApplication(primarySources)).run(args);                                                                                        
}


```

SpringApplication的初始化过程主要包括4部分

1. this.webApplicationType = WebApplicationType.deduceFromClasspath()

   用于判断当前webApplicationType应用的类型。deduceFromClasspath()方法用于查看classpath类路径下是否存在某个特征类，从而判断当前webApplicationType类型是Servlet应用（Spring5之前的传统MVC应用）还是Reactive应用(Spring5开始出现的webFlux交互应用)

2. this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class))

   用于SpringApplication应用的初始化设置。在初始化器设置过程中，会使用spring 类加载器从类路径下的META-INF/spring.factories文件中获取所有可用的应用初始化器类ApplicationContextInitializer

3. this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class))

   用于SpringApplication应用的监听器设置。使用SpringFactoriesLoader从类路径下的META-INF/spring.factories文件中获取所有可用的监听器类ApplicationListener

4. this.mainApplicationClass = this.deduceMainApplicationClass()

   用于推断并设置项目main()方法启动的主程序启动类

调用run()方法

1. 获取并启动监听器

   this.getRunListeners(args)和listeners.starting()主要用来初始化SpringApplicationRunListeners监听器并运行

2. 根据springApplicationRunListeners以及参数来准备环境

   this.prepareEnviroment(listeners,applicationArguments)方法主要用来对项目运行环境进行预设置

3. 创建spring容器

   根据webApplicationType进行判断，确定容器类型，如果该类型为servlet，会通过反射装载对应的字节码，接着使用之前初始化设置的context(应用上下文环境)，enviroment(项目运行环境)、listeners(监听器)，applicationArgyments(项目参数)和printedBanner(项目图标信息)进行应用上下文的组装配置，并刷新配置

4. spring容器前置处理

   容器刷新之前的准备动作。设置容器环境，包括各种变量等等，其中包含一个非常关键的操作：将启动类注入容器，为后续开启自动化配置奠定基础

5. 刷新容器

   通过refresh()方法对IOC容器进行初始化，同时向JVM运行时注册一个关机钩子，在JVM关机时会关闭这个上下文

6. spring容器后置处理

   扩展接口，设计模式中的模板方法，默认为空实现

7. 发出结束执行的事件

   获取EventPublishingRunListener监听器，并执行其started方法，并且将创建的spring容器传进去，创建一个ApplicationStartedEvent时间，并执行ConfigurableApplicationContext的publishEvnet方法，在spring容器中发布事件

8. 执行Runners

   用于调用项目中自定义的执行器xxRunner类，使得在项目启动后立即执行一些特定程序。springBoot提供的执行器接口有ApplicationRunner和CommandLineRunner两种

   ![image-20200516213611653](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200516213611653.png)

---



##### SpringBoot+MyBatis

1、引入Maven坐标，这里使用MySQL数据库

```xml
<dependency>    
    <groupId>org.mybatis.spring.boot</groupId>    
    <artifactId>mybatis-spring-boot-starter</artifactId>    
    <version>2.1.2</version>
</dependency>
<dependency>      
     <groupId>mysql</groupId>    
    <artifactId>mysql-connector-java</artifactId>    
    <scope>runtime</scope>
</dependency>

```

2、配置数据库连接

```properties
  # MySQL数据库连接
  spring.datasource.url=jdbc:mysql://localhost:3306/springbootdata? serverTimezone=UTC  
  spring.datasource.username=root  
  spring.datasource.password=root
 

```

3、注解方式使用 或 xml配置

```java
 /**
 *注解方式
 *方式1：每个mapper文件上加@Mapper注解
 *方式2：一劳永逸，启动类上加@MapperScan("com.xcl.blog.mapper")
 */
 @Mapper  
 public interface CommentMapper {         
     @Select("SELECT * FROM t_comment WHERE id =#{id}")      
     public Comment findById(Integer id);  
 }


```

```xml
<!--xml方式-->
<?xml version="1.0" encoding="UTF-8" ?>  
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"              "http://mybatis.org/dtd/mybatis-3-mapper.dtd"> 
    
<mapper namespace="com.lagou.mapper.ArticleMapper">    
   <select id="selectArticle" resultType="Article">        
    select * from Article    
   </select> 
</mapper>

在全局配置文件中加入mapper.xml路径
mybatis.mapper-locations=classpath:mapper/*.xml 

//mybatis.type-aliases-package=com.xml.pojo

```



##### SpringBoot+JPA

1. 引入Maven坐标

   ```xml
   <dependency> 
       <groupId>org.springframework.boot</groupId> 
       <artifactId>spring-boot-starter-data-jpa</artifactId> 
   </dependency>
   
   
   ```

2. 编写pojo

   ```java
   @Entity(name = "t_comment")  // 指定表名
   public class Comment {
       @Id   // 主键  
       @GeneratedValue(strategy = GenerationType.IDENTITY) // ᦡID生成策略  
       private Integer id;    
       private String content;    
       private String author;
       @Column(name = "a_id")  //表字段名（当属性名与字段名不一致时需要手动指定）   
       private Integer aId;    
       //getter和setter
   }
   
   
   ```

3. 编写Repository接口

   ```java
   public interface CommentRepository extends JpaRepository<Comment,Integer> {
   }
    
   
   ```

##### SpringBoot+Redis

1. 引入Maven坐标

   ```xml
   <dependency>  <groupId>org.springframework.boot</groupId>  <artifactId>spring-boot-starter-data-redis</artifactId> </dependency>
    
   
   ```

2. 配置

   ```properties
   spring.redis.host=127.0.0.1
   spring.redis.port=6379
   spring.redis.password=
   
   
   ```

3. 注解方式使用 或 API(借助redisTemplate）方式使用

##### SpringBoot+Thymeleaf

1. 引入Maven坐标

   ```xml
   <dependency>   
       <groupId>org.springframework.boot</groupId>   
       <artifactId>spring-boot-starter-thymeleaf</artifactId> 
   </dependency>
   
   ```

2. 配置

   ```properties
   spring.thymeleaf.cache = true  #启用缓存模板     
   spring.thymeleaf.encoding = UTF-8   #模板编码
   spring.thymeleaf.mode = HTML5  #应用于模板的模板模式
   spring.thymeleaf.prefix = classpath:/templates/  #指定模板页面的存放路径
   spring.thymeleaf.suffix = .html #指定模板页面名称的后缀
   
   ```

   静态资源的访问：使用spring Initializer方式创建的springboot项目，默认生成一个resources目录，在resources目录下新建public、resources、static三个子目录下，springBoot默认会挨个从public、resources、static中查找静态资源

##### 国际化页面

springBoot默认识别的语言配置文件为类路径下(resources)的messages.properties。其他语言国际化文件的名称必须要个按照文件前缀名_ 语言_国家.properties的形式命名

1. 编写国际化文件(resources目录下)

2. 指定国际化文件基础名

   ```
    spring.messages.basename=i18n.login
   
   ```

3. 自定义区域信息解析器

   ```java
   @Configuration //配置类
   public class MyLocaleResovel implements LocaleResolver {//实现区域解析器
     @Override    
       public Locale resolveLocale(HttpServletRequest httpServletRequest) {                		String l = httpServletRequest.getParameter("l");        // 
           String header= httpServletRequest.getHeader("Accept-Language");    
         if(!StringUtils.isEmpty(l)){            
             String[] split = l.split("_");            
             locale=new Locale(split[0],split[1]);        
         }else {            
             // Accept-Language: en-US,en;q=0.9          ,zh-CN;q=0.8,zh;q=0.7     
             String[] splits = header.split(",");            
             String[] split = splits[0].split("-");            
             locale=new Locale(split[0],split[1]);        
         }       
         return locale;
      } 
       
       //加入IOC容器
        @Bean   
       public LocaleResolver localeResolver(){        
           return new MyLocalResovel();   
       } 
   
   }
   
   ```

4. 国际化页面使用

   ```HTML
   <a class="btn btn-sm" th:href="@{/toLoginPage(l='zh_CN')}">中文</a>    
   <a class="btn btn-sm" th:href="@{/toLoginPage(l='en_US')}">English</a>
   
   ```

##### SpringBoot缓存管理

###### 默认缓存

spring框架支持透明地向应用程序添加缓存，其管理缓存的核心是将缓存应用于操作数据的方法，从而减少操作数据的执行次数，同时不会对程序本身造成任何干扰

springBoot继承了spring框架的缓存管理功能，通过使用@EnableCaching注解开启基于注解的缓存，然后在Service类或其方法上添加@Cacheable注解，对查询结果进行缓存

```java
@Service 
public class CommentService {
    @Autowired    
    private CommentRepository commentRepository;
    
    /**
    *Select语句执行流程
    *先查询缓存Cache，到cacheNames指定的名称空间中使用key	*获取Cache值，如果没有则查询数据库且将查询结果进行缓存，		*有就直接返回
    *
    *将查询结果Comment存放在comment的名称空间中
    *SimpleKeyGenerator生成key的默认策略
   	*	没有参数 new SimpleKey()
   	*	有一个参数 参数值
    * 	多个参数 new SimpleKey(params)	
    */
    @Cacheable(cacheNames = "comment",unless = "#result==null")    
    public Comment findCommentById(Integer id){        
        Optional<Comment> comment = commentRepository.findById(id);       
        if(comment.isPresent()){           
            Comment comment1 = comment.get();          
            return comment1;        
        }        
        return null;   
    }
    
    
    /**
    *方法执行完后生效。使缓存失效，方法执行后的结果被保存在缓	 *存中
    */
    @CachePut(cacheNames = "comment",key = "#result.id")    
    public Comment updateComment(Comment comment) {  
        commentRepository.updateComment(comment.getAuthor(), comment.getaId());    
        return comment;   
    }
    
    
    /**
    *删除缓存数据
    */
    @CacheEvict(cacheNames = "comment")    
    public void deleteComment(int comment_id) {   
        commentRepository.deleteById(comment_id);    
    }
}

```

底层结构：springBoot默认装配的是SimpleCacheConfiguration，其使用的CacheManager是ConcurrentMapCacheManager,使用concurrentMap当底层数据结构

@Cacheable常用属性

![image-20200516224100096](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200516224100096.png)

常用的SPEL表达式

![image-20200517001718524](C:\Users\x1850\AppData\Roaming\Typora\typora-user-images\image-20200517001718524.png)

###### 整合redis

如果程序中没有定义类型为CacheManager的Bean组件或者名为cacheResolver的cacheResolver缓存解析器，springBoot将尝试选择并启用以下缓存组件（按照指定的顺序）

1. Generic
2. JCache
3. EnCache2.X
4. Hazelcast
5. Infinispan
6. Couchbase
7. Redis
8. Caffeine
9. Simple（默认缓存组件）

如果在项目中添加某个缓存管理组件（例如redis）,springBoot会选择并启用对应的缓存管理器。

基于注解的Redis缓存实现

1. 引入Maven坐标

   ```xml
   <dependency> 
       <groupId>org.springframework.boot</groupId> 
       <artifactId>spring-boot-starter-data-redis
       </artifactId> 
   </dependency>
   
   
   ```

   当添加了redis相关的启动器后，springboot会使用RedisCacheConfiguration当做生效的自动配置类进行缓存相关的自动装配，容器中使用的缓存管理器是RedisCacheManager,这个缓存管理器创建的Cache为RedisCache，进而控制redis进行数据的缓存

2. redis服务连接配置

   ```properties
   spring.redis.host=127.0.0.1
   spring.redis.port=6379
   spring.redis.password=
   
   ```

3. 开启注解@EnableCaching，使用@Cacheable/@CachePut/@CacheEvict

   注：缓存对象需要实现序列化接口

基于API的缓存实现

通过redis提供的API调用相关方法实现数据缓存管理，同时，这种方法还可以手动管理缓存的有效期

```java
 @Autowired    
 private RedisTemplate redisTemplate;

 redisTemplate.opsForValue().set("comment_"+id,comment,1,TimeUnit.DAYS); 
  redisTemplate.delete("comment_"+comment_id); 
 

```

##### 自定义redis缓存序列化机制

默认使用JdkSerializationRedisSerializer序列化方式，所以要进行缓存的数据必须实现JDk自带的序列化接口（Serializable）

基于API的序列化方式，自定义RedisTemplate

```java
@Configuration 
public class RedisConfig {
    @Bean    
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory                                                               redisConnectionFactory) {       
        RedisTemplate<Object, Object> template = new RedisTemplate();  
        template.setConnectionFactory(redisConnectionFactory); 
        //使用JSON格式序列化对象
        Jackson2JsonRedisSerializer jacksonSeial = new Jackson2JsonRedisSerializer(Object.class);
        
        //解决转换异常的问题
           ObjectMapper om = new ObjectMapper(); 
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);              om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);    
        jacksonSeial.setObjectMapper(om); 
        //设置模板的序列化方式
        template.setDefaultSerializer(jacksonSeial);        
        return template;    
    } 
}    
 

```

基于注解的序列化方式，自定义RedisCacheManager

```java
@Bean 
public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {  
    RedisSerializer<String> strSerializer = new StringRedisSerializer(); 
    Jackson2JsonRedisSerializer jacksonSeial = new 
        Jackson2JsonRedisSerializer(Object.class); 
    ObjectMapper om = new ObjectMapper();  
    om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY); 
    om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL); 
    jacksonSeial.setObjectMapper(om);
    RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig() 
        .entryTtl(Duration.ofDays(1))           
        .serializeKeysWith(RedisSerializationContext.SerializationPair     
                           .fromSerializer(strSerializer))     
        .serializeValuesWith(RedisSerializationContext.SerializationPair  
                             .fromSerializer(jacksonSeial))  
        .disableCachingNullValues();  
    RedisCacheManager cacheManager = RedisCacheManager   
        .builder(redisConnectionFactory).cacheDefaults(config).build(); 
    return cacheManager; 
}

```



# 初步学习二

[toc]

## docker的简单命令

1. 查询关键字

   ```
   docker search xxx
   ```

2. 拉取镜像

   ```
   docker pull xxx:tag    
   tag可选,默认是latest
   ```

3. 查看所有的镜像

   ```
   docker images
   ```

4. 删除镜像

   ```
   docker rmi 镜像id
   ```

5. 运行

   ```
   docker run --name container-name -d image-name 
   eg:docker run –name myredis –d redis
   --name：自定义容器名 
   -d：后台运行
   image-name:指定镜像模板
   ```

6. 停止

   ```
   docker stop container-name/container-id
   ```

7. 启动

   ```
   docker start container-name/container-id
   ```

8. 查看列表

   ```
   docker ps 
   默认是查看运行的
   加上 -a 可以查看所有的
   ```

9. 删除

   ```
   docker start container-name/container-id
   ```

10. 端口映射

    ```
     -p 6379:6379
     eg:docker run -d -p 6379:6379 --name myredis docker.io/redis
     -p:虚拟机端口:容器端口
    ```

### 关于云服务配置端口映射后无法访问的解决方案

1. 使用命令: docker exec -it 运行的tomcat容器ID /bin/bash 进入到tomcat的目录
2. 删除原有的webapps目录,应该是空的
3. webapps.dist重命名成webapps即可

### 在docker中安装mysql：5.7

如果按照前面所述的方法安装mysql，查看日志文件会发现如下错误，原因是必须设置以下三种状态的一种

```
[ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
	You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
```

可以查看官方文档https://hub.docker.com/_/mysql

以下是一个使用实例

```shell
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
eg:docker run -p 3306:3306 --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

## 数据访问

1. 配置数据源

   ```yml
   spring:
     datasource:
       username: root
       password: 123456
       url: jdbc:mysql://139.9.128.222:3306/jdbc
       driver-class-name: com.mysql.jdbc.Driver
   ```

   springboot 2.3.0默认使用的是class com.zaxxer.hikari.HikariDataSource数据源

### 使用druid数据源

导入依赖

```
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.20</version>
</dependency>
```

配置文件

```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://xxxxxxx:3306/jdbc
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true

    filters: stat,wall,slf4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

配置druid的servlet后台和filter

```java
//导入druid数据源
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
       return  new DruidDataSource();
    }

    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    //之后可以访问http://localhost:8080/druid登陆后台
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();

        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        initParams.put("allow","");//默认就是允许所有访问
        initParams.put("deny","192.168.15.21");

        bean.setInitParameters(initParams);
        return bean;
    }


    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");

        bean.setInitParameters(initParams);

        bean.setUrlPatterns(Arrays.asList("/*"));

        return  bean;
    }
}		
```

### 整合mybatis

**使用注解**

1. 编写mapper文件
2. 直接调用即可

```java
//指定这是一个操作数据库的mapper
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);

    @Delete("delete from department where id=#{id}")
    public int deleteDeptById(Integer id);

    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDept(Department department);

    @Update("update department set departmentName=#{departmentName} where id=#{id}")
    public int updateDept(Department department);
}
```

添加一些自定义配置

```java
@org.springframework.context.annotation.Configuration
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer(){

            @Override
            public void customize(Configuration configuration) {
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

**使用xml配置**

1. 编写mapper接口文件
2. 编写mapper对应的xml文件
3. 在yml文件中配置mybatis配置的文件的路径

```yml
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml 指定全局配置文件的位置
  mapper-locations: classpath:mybatis/mapper/*.xml  指定sql映射文件的位置
```

### 使用jpa

1）、编写一个实体类（bean）和数据表进行映射，并且配置好映射关系；

```java
//使用JPA注解配置映射关系
@Entity //告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "tbl_user") //@Table来指定和哪个数据表对应;如果省略默认表名就是user；
public class User {

    @Id //这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增主键
    private Integer id;

    @Column(name = "last_name",length = 50) //这是和数据表对应的一个列
    private String lastName;
    @Column //省略默认列名就是属性名
    private String email;
```

2）、编写一个Dao接口来操作实体类对应的数据表（Repository）

```java
//继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User,Integer> {
}

```

3）、基本的配置JpaProperties

```yaml
spring:  
 jpa:
    hibernate:
#     更新或者创建数据表结构
      ddl-auto: update
#    控制台显示SQL
    show-sql: true
```

4）、直接调用UserRepository的方法即可。

## 缓存使用

### @Cacheable注解

1. 在主程序中添加@EnableCaching注解，开启缓存

2. 在需要的方法添加@Cacheable注解，表示使用缓存

   需要注意的是，需要添加一个缓存名，表示这条数据存储在的缓存空间

   ```java
   public @interface Cacheable {
   	//value和cacheNames是同一个东西，两者互为别名
      @AliasFor("cacheNames")
      String[] value() default {};
   
      @AliasFor("value")
      String[] cacheNames() default {};
   	//表示缓存数据的key
      String key() default "";
   	//key的一个生成器，于key两者只能使用一者
      String keyGenerator() default "";
   	//设置缓存管理器
      String cacheManager() default "";
   	
      String cacheResolver() default "";
   	//在满足给定条件才添加缓存
      String condition() default "";
   	//满足给定条件就不添加缓存
      String unless() default "";
   	//同步与异步
      boolean sync() default false;
   
   }
   ```

**在缓存的注解中，允许使用spel**

| **名字**        | **位置**           | **描述**                                                     | **示例**             |
| --------------- | ------------------ | ------------------------------------------------------------ | -------------------- |
| methodName      | root object        | 当前被调用的方法名                                           | #root.methodName     |
| method          | root object        | 当前被调用的方法                                             | #root.method.name    |
| target          | root object        | 当前被调用的目标对象                                         | #root.target         |
| targetClass     | root object        | 当前被调用的目标对象类                                       | #root.targetClass    |
| args            | root object        | 当前被调用的方法的参数列表                                   | #root.args[0]        |
| caches          | root object        | 当前方法调用使用的缓存列表（如@Cacheable(value={"cache1",  "cache2"})），则有两个cache | #root.caches[0].name |
| *argument name* | evaluation context | 方法参数的名字. 可以直接 #参数名 ，也可以使用 #p0或#a0 的形式，0代表参数的索引； | #iban 、 #a0 、 #p0  |
| result          | evaluation context | 方法执行后的返回值（仅当方法执行之后的判断有效，如‘unless’，’cache put’的表达式 ’cache evict’的表达式beforeInvocation=false） | #result              |

大概原理

1. 根据自动配置类注册cacheManager
2. 第一次查询的时候根据cacheName查询组件，若不存在就创建
3. 如果没有设置key，就根据keyGenerator生成key，之后根据key去缓存中查询，不存在就执行接下来的方法获得数据后，在保存，如果存在，直接从cache中获取并返回

###  @CachePut注解

一般用在更新方法上，作用是先执行方法，之后将结果缓存。使用的时候注意key的值。

### @CacheEvict注解

一般用在删除方法上，作用是清除缓存。

```java
//指定这个属性，表示清除cacheNames的所有缓存键值对
boolean allEntries() default false;
//默认是在方法执行后删除，如果指定为true，表示在方法执行前就会执行缓存清除操作
boolean beforeInvocation() default false;
```

### @CacheConfig注解

标注在类上，用于统一指定这个类的一些缓存属性，比如cacheNames，keyGenerator等等

### @Caching注解

是三者的结合，可以进行一些复杂的缓存

```java
public @interface Caching {

   Cacheable[] cacheable() default {};

   CachePut[] put() default {};

   CacheEvict[] evict() default {};

}
```

### 整合redis

1. 导入依赖，默认版本和springboot的版本一致

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2. 在配置文件中配置redis的地址，端口号默认是6379

   ```properties
   spring.redis.host=xxx.xxx.xxxx.xxx
   ```

简单的测试

```java
@SpringBootTest
class SpringbootCacheApplicationTests {
    @Autowired
    StringRedisTemplate template;

    @Test
    void contextLoads() {
        template.opsForValue().append("1","1");
    }
}
```

**在RedisAutoConfiguration配置类中，默认存在两个组件RedisTemplate<Object, Object>和StringRedisTemplate。其实StringRedisTemplate是RedisTemplate的子类。**

```java
public class StringRedisTemplate extends RedisTemplate<String, String> {}
```

关于更多操作redis的方法，关注template.opsfxxx

下面的5种方法，就是对应了redis的五种数据类型

```java
template.opsForValue();//String 
template.opsForHash();//Hash
template.opsForList();//List
template.opsForSet();//Set
template.opsForZSet();//ZSet
```

在将对象保存到redis时,就涉及到序列化的问题，由于redis默认的value序列化方法是jdk的序列化，不是很方便查看，因此如果想要配置json的序列化方式，需要手动编写代码。

```java
//系统代码
public static RedisCacheConfiguration defaultCacheConfig(@Nullable ClassLoader classLoader) {

   DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();

   registerDefaultConverters(conversionService);

   return new RedisCacheConfiguration(Duration.ZERO, true, true, CacheKeyPrefix.simple(),
          //key的序列化
         SerializationPair.fromSerializer(RedisSerializer.string()),
          //value的序列化
         SerializationPair.fromSerializer(RedisSerializer.java(classLoader)), conversionService);
}
```

因此要实现json的序列化，需要自己写代码，以下是一个实例

```java
@Configuration
public class MyRedisConfig {

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer =
                new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // 配置序列化
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer));

        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }
}
```

## rabbitmq消息队列

\# 表示匹配0个或多个单词， \* 表示匹配一个单词

### 在docker安装rabbitmq

​	注意映射两个端口5672和15672 

15672端口用于浏览器访问后台

访问ip：15672 就可以进入后台，用户名和密码默认是guest

```shell
docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq cc86
```

### 在IDEA中使用rabbitmq

导入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-amqp -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>2.3.0.RELEASE</version>
</dependency>

```

在properties中配置host的ip，默认是localhost

```
spring.rabbitmq.host=xxx.xxx.xxx.xxx
```

配置用户名和密码，默认也都是guest

```
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

配置端口，在我的springboot版本2.3.0中没有自动配置，手动配置5672

```
spring.rabbitmq.port=5672
```

设置虚拟主机，默认不写是“/”

```
spring.rabbitmq.virtual-host=
```

**简单使用**

通过rabbitTemplate来操纵对rabbitmq的操作。

```java
//传入exchange，routingKey和自定义的message（比较复杂）
rabbitTemplate.(String exchange, String routingKey和自定义的message, Message message);
//使用这个方法会自动序列化，比较简单方便
rabbitTemplate.convertAndSend(String exchange, String routingKey, Object message);
//接收消息队列中的数据
rabbitTemplate.receiveAndConvert(String queueName);
```

由于在序列化对象的时候默认还是使用jdk的序列化，不方便查看，所以最好还是使用json的序列化方式。

只需要配置一个json的消息转化器就可以了

```java
@Configuration
public class MyAmqpConfig {
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

要监听消息队列，只需要开启注解即可

1. ```java
   @EnableRabbit使用这个注解开启注解模式
   ```

2. ```java
   //queue里写消息队列的名称，每当消息队列接受到消息的时候，就能够取出消息
   @RabbitListener(queues = "")
   ```

**如果要想创建消息队列或者是交换器，可以通过amqpAdmin来创建。**

## Elasticsearch检索

1. docker安装

   由于elasticsearch据说默认会占用2G的堆内存，如果服务器配置不足，可以像下方一样进行限制

   ```xhell
   docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name ES01 elasticsearch
   ```

2. 访问9200端口，如果能够出现一串json，就说明安装成功。

**可以查看官方文档来学习ES**：https://www.elastic.co/guide/cn/elasticsearch/guide/cn/foreword_id.html

### 在IDEA中使用ES

导入依赖（下面第一个是springdata的，第二个是用jest来使用，不过已经不推荐使用了）

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>

 <!-- https://mvnrepository.com/artifact/io.searchbox/jest -->
        <dependency>
            <groupId>io.searchbox</groupId>
            <artifactId>jest</artifactId>
            <version>5.3.3</version>
        </dependency>
```

需要注意的是ES存在版本对应关系，需要注意es版本和spring-data-elasticsearch版本的关系。

## 异步任务

1. ```
   @EnableAsync//开启异步任务注解
   ```

2. ```
   @Async//标注一个方法为异步模式的
   ```

## 定时任务

1. ```
   @EnableScheduling//开启定时任务
   ```

2. ```
   @Scheduled(cron = "0 * * * * MON-FRI")//书写cron表达式，配置定时规则
   ```

| **字段** | **允许值**             | **允许的特殊字符** |
| -------- | ---------------------- | ------------------ |
| 秒       | 0-59                   | , -  * /           |
| 分       | 0-59                   | , -  * /           |
| 小时     | 0-23                   | , -  * /           |
| 日期     | 1-31                   | , -  * ? / L W C   |
| 月份     | 1-12                   | , -  * /           |
| 星期     | 0-7或SUN-SAT  0,7是SUN | , -  * ? / L C #   |

| **特殊字符** | **代表含义**               |
| ------------ | -------------------------- |
| ,            | 枚举                       |
| -            | 区间                       |
| *            | 任意                       |
| /            | 步长                       |
| ?            | 日/星期冲突匹配            |
| L            | 最后                       |
| W            | 工作日                     |
| C            | 和calendar联系后计算过的值 |
| #            | 星期，4#2，第2个星期四     |
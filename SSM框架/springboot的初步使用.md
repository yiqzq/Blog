## springboot的初步使用

[toc]

## 二刷章节

P18 自动装配源码分析



## 项目的创建

推荐使用官方Spring Initializer来创建一个springboot项目。

<img src="https://i.loli.net/2020/05/22/1ujN5wdCKDPrJIn.png" alt="image-20200522180023973" style="zoom:67%;" />

创建完成之后，只需要直接编写controller就可以使用了，省去了SSM复杂的配置过程。

**当前的文件结构如下**

![image-20200522180151137](https://i.loli.net/2020/05/22/uci43EzQVThbrBA.png)

**resources文件夹中目录结构**

- static：保存所有的静态资源； js css images；
-  templates：保存所有的模板页面；（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页 面）；可以使用模板引擎（freemarker、thymeleaf）；
-  application.properties：Spring Boot应用的配置文件；可以修改一些默认设置；



需要注意的是，controller包一定要和DemoApplication主程序在同一个包下，如果放在外面的话，即使添加了注解，spring也无法扫描到组件。

下面是简单的controller文件

```java
package com.yiqzq.demo.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author yiqzq
 * @date 2020/5/22 17:42
 */
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(){
        return "hello world!";
    }
}
```

之后直接启动主程序即可，springboot内部集成了tomcat

```java
package com.yiqzq.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

## springboot的配置文件

### YAML文件

#### 语法特点

- 大小写敏感
- 通过缩进表示层级关系
- **禁止使用tab缩进，只能使用空格键** 
- 缩进的空格数目不重要，只要相同层级左对齐即可
- 使用#表示注释

#### 支持的数据结构

- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）

- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）

- #### 纯量（scalars）：单个的、不可再分的值

#### 基本用法

k:(空格)v：表示一对键值对

以空格的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

**对象写法**

```yaml
friends:
	lastName: zhangsan 
	age: 20
```

```yaml
friends: {lastName: zhangsan,age: 18}
```



**数组写法**

用- 值表示数组中的一个元素

```yaml
brand:
   - audi
   - bmw
   - ferrari
```

```yaml
brand: [audi,bmw,ferrari]
```

### 配置文件的占位符

1. 随机数

   ```xml
   ${random.value}、${random.int}、${random.long}
   ${random.int(10)}、${random.int[1024,65536]}
   ```

2. 使用前面配置的属性，没有的话，可以使用：来指定默认值

   选择之前person的hello字段，如果没有，默认位hello

   ```
   person.dog.name=${person.hello:hello}_dog
   ```

### 激活不同的profile

​	1、在配置文件中指定  spring.profiles.active=dev

​	2、命令行：

​		java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；

​		可以直接在测试的时候，配置传入命令行参数

​	3、虚拟机参数；

​		-Dspring.profiles.active=dev

### 配置文件的加载顺序

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

–file:./config/

–file:./

–classpath:/config/

–classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这四个位置全部加载主配置文件；**互补配置**；

同时还可以通过命令行参数，来指定新的配置文件，用于互补

### 外部配置加载顺序

**1.命令行参数**

所有的配置都可以在命令行上进行指定

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc

多个配置用空格分开； --配置项=值



2.来自java:comp/env的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

5.RandomValuePropertySource配置的random.*属性值



==**由jar包外向jar包内进行寻找；**==

==**优先加载带profile**==

**6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**

**7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**



==**再来加载不带profile**==

**8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件**

**9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件**



10.@Configuration注解类上的@PropertySource

11.通过SpringApplication.setDefaultProperties指定的默认属性

更多信息，参考[官方文档](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/htmlsingle/#boot-features-external-config)4.2



## 向spring注入bean

### 通过从全局配置文件中读取@ConfigurationProperties

主要是通过一个注解来实现的

```
@ConfigurationProperties(prefix = "")
```

使用的时候需要绑定prefix。

对于properties和yml文件，在同名的情况下，properties优先级高于yml文件

### 通过@Value注解绑定属性

```
@Value("")
```

 **@ConfigurationProperties和@Value的区别**

${key}从环境变量、配置文件中获取值

#{SpEL}

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

### 加载指定的配置文件@PropertySource

```
@PropertySource(value = {"classpath:xxx.properties"})
```

### 导入Spring的配置文件@ImportResource

```
@ImportResource(locations = {"classpath:xxx.xml"})
```

### 使用全注解方式添加bean（官方推荐）

主要是使用@Configuration和@Bean联合使用

```java
/**
 * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 * 在配置文件中用<bean><bean/>标签添加组件
 *
 */
@Configuration
public class MyAppConfig {
    //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

## SpringBoot自动配置原理

Spring Boot启动的时候会通过**@EnableAutoConfiguration**注解找到**META-INF/spring.factories**配置文件中的所有自动配置类，并对其进行加载，而这些自动配置类都是以AutoConfiguration结尾来命名的，它实际上就是一个JavaConfig形式的Spring容器配置类，它能通过以**Properties**结尾命名的类中取得在全局配置文件中配置的属性如：server.port，而XxxxProperties类是通过**@ConfigurationProperties**注解与全局配置文件中对应的属性进行绑定的。

## 日志功能

### 日志的选择

市面上的日志框架；

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....

| 日志门面  （日志的抽象层）                                   | 日志实现                                             |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| ~~JCL（Jakarta  Commons Logging）~~    SLF4j（Simple  Logging Facade for Java）    **~~jboss-logging~~** | Log4j  JUL（java.util.logging）  Log4j2  **Logback** |

SpringBoot：底层是Spring框架，Spring框架默认是用JCL

**==SpringBoot选用 SLF4j和logback；==**

### 适配不同的框架

由于不同的框架采用的日志框架可能不同，因此在springboot为了统一框架，一般会采用适配的方式，添加一个中间依赖来适配其他框架达到统一的目的。

可以看到，为了适配不同的框架，对于不同的（比如说log4j，jul）框架，添加了adaptation layer 依赖。

![image-20200524123936213](C:\Users\39268\AppData\Roaming\Typora\typora-user-images\image-20200524123936213.png)

### 日志的使用和配置

```
Logger logger = LoggerFactory.getLogger(getClass());

logger.trace();		
logger.debug();
logger.info();
logger.warn();
logger.error();

```

**简单的配置**

只需要在properties中设置即可

```
//设置级别，可以精确到每一个类，springboot默认是全局info
logging.level=
//日志文件输出为止
logging.file.path=
//日志文件名称
logging.file.name=
```

### 指定自己的配置文件

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

只需要在类路径下写下自定义配置文件即可。

推荐使用带有-spring后缀的文件名，这样子写可以被springboot识别到，可以写入一些springboot的高级功能。

## Web开发

### 静态资源的导入

1. 对于像bootstrap，js这些静态资源，一种方法就是通过jar中引入，一般使用的方式就是通过[webjars](https://www.webjars.org/)这个网站上导入maven依赖，然后获取。

   举个例子：http://localhost:8080/webjars/jquery/3.1.1/jquery.js

2. 另外自己的图片等静态资源存放位置

一般存放在一下位置就可以被自动识别到（ResourceProperties.java中可以找到配置）

```
//同时，这也是静态资源文件夹的优先级
{ 	
"classpath:/META-INF/resources/",
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/" 
};
```

比如要访问classpath:/static/pic.pig下的文件，只需要输入http://localhost:8080/pic.pig即可。

**注意：如果导入之后无法访问的，可以尝试重启maven**

3）、欢迎页； 静态资源文件夹下的所有index.html页面；被"/**"映射；

​	localhost:8080/   找index页面

4）、所有的 **/favicon.ico  都是在静态资源文件下找；

### 模板引擎thymeleaf

#### 导入依赖		

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

```java
public class ThymeleafProperties {

   private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;
	//实现了像spring视图解析器一样的拼接功能，只需要把html页面放过在classpath:/templates/路径下即可
   public static final String DEFAULT_PREFIX = "classpath:/templates/";

   public static final String DEFAULT_SUFFIX = ".html";
   }
```

#### thymeleaf的用法

在html页面中导入

```html
<html xmlns:th="http://www.thymeleaf.org">
```

在IDEA开发中，发现thymeleaf没有代码提示，可以查看plugins一栏有没有启用thymeleaf

```properties
Simple expressions:（表达式语法）
    Variable Expressions: ${...}：获取变量值；OGNL；
    		1）、获取对象的属性、调用方法
    		2）、使用内置的基本对象：
    			#ctx : the context object.
    			#vars: the context variables.
                #locale : the context locale.
                #request : (only in Web Contexts) the HttpServletRequest object.
                #response : (only in Web Contexts) the HttpServletResponse object.
                #session : (only in Web Contexts) the HttpSession object.
                #servletContext : (only in Web Contexts) the ServletContext object.
                
                ${session.foo}
            3）、内置的一些工具对象：
#execInfo : information about the template being processed.
#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
#uris : methods for escaping parts of URLs/URIs
#conversions : methods for executing the configured conversion service (if any).
#dates : methods for java.util.Date objects: formatting, component extraction, etc.
#calendars : analogous to #dates , but for java.util.Calendar objects.
#numbers : methods for formatting numeric objects.
#strings : methods for String objects: contains, startsWith, prepending/appending, etc.
#objects : methods for objects in general.
#bools : methods for boolean evaluation.
#arrays : methods for arrays.
#lists : methods for lists.
#sets : methods for sets.
#maps : methods for maps.
#aggregates : methods for creating aggregates on arrays or collections.
#ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

    Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
    	补充：配合 th:object="${session.user}：
   <div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
    </div>
    
    Message Expressions: #{...}：获取国际化内容
    Link URL Expressions: @{...}：定义URL；
    		@{/order/process(execId=${execId},execType='FAST')}
    Fragment Expressions: ~{...}：片段引用表达式
    		<div th:insert="~{commons :: main}">...</div>
    		
Literals（字面量）
      Text literals: 'one text' , 'Another one!' ,…
      Number literals: 0 , 34 , 3.0 , 12.3 ,…
      Boolean literals: true , false
      Null literal: null
      Literal tokens: one , sometext , main ,…
Text operations:（文本操作）
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:（数学运算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:（布尔运算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:（比较运算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:条件运算（三元运算符）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
Special tokens:
    No-Operation: _ 
```

### 如何修改SpringBoot的默认配置

模式：

​	1）、SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来；

​	2）、在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置

​	3）、在SpringBoot中会有很多的xxxCustomizer帮助我们进行定制配置

### web实战问题

#### 设置首页

- 将首页文件index.html放在静态资源文件下,springboot可以识别到

- 如何使得放在templates文件夹下的首页得到访问

  - 写一个空的Controller方法

    ```java
    @RequestMapping({"/", "/index.html"})
    public String index() {
        return "index";
    }
    ```

  - 手动配置config

    ```java
    public class Mycaonfig implements WebMvcConfigurer {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("index");
        }
    }
    ```

    

#### 国际化

之前的用法

1）、编写国际化配置文件；

2）、使用ResourceBundleMessageSource管理国际化资源文件

3）、在页面使用fmt:message取出国际化内容

**国际化乱码问题**

设置default encoding

<img src="https://i.loli.net/2020/05/25/R8zQ2bhKdrBIFGk.png" alt="image-20200525175509264" style="zoom:80%;" />

**springboot使用国际化**

1. 编写国际化配置文件

   <img src="https://i.loli.net/2020/05/25/eoYkq8fB7xyuU2b.png" alt="image-20200525180055617" style="zoom:80%;" />

2. 在html中使用thymeleaf语法使用，导入国际化文件的字段

**实现点击链接切换国际化**

1. 给链接带上区域信息

2. 后台获取请求附带的区域信息，手动书写LocaleResolver，并返回对应的Locale。

   

```java
//往容器中添加自己的组件，方法名不能换，必须是localeResolver，因为spring底层自动化配置的时候会检测，容器中有没有自己写的Bean，没有才会添加
@Bean
public LocaleResolver localeResolver() {
    return new MyLocaleResolver();
}
-------------------------------------------------------

public class MyLocaleResolver implements LocaleResolver {

    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String str = request.getParameter("locale");
        Locale locale = Locale.getDefault();
        if (!StringUtils.isEmpty(str)) {
            String[] s = str.split("_");
            locale = new Locale(s[0], s[1]);
        }
        System.out.println(locale);
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {

    }
}

```

#### 设置IDEA的热部署

##### 方法1

1. 首先，需要thymeleaf的缓存设置为false。

```
spring.thymeleaf.cache=false
```

2. 然后使用下面两条命令，均可以。
   `ctrl + shift + F9`：Recompile （这个一般是修改了java代码，如controller等文件，重新编译）
   `ctrl + F9`：Build Project （这个，当你修改了静态页面，如html页面，重新build）

##### 方法二

1. 同样首先禁用缓存
2. 在setting中设置

<img src="https://i.loli.net/2020/05/26/QybFh2T6POHsuzW.png" alt="image-20200526112455767" style="zoom:80%;" />

3. 然后 Shift+Ctrl+Alt+/，选择Registry

<img src="https://i.loli.net/2020/05/26/snC4pL1zjWvYrc8.png" alt="image-20200526112619098" style="zoom:80%;" />

4. 重启即可

#### 请求转发之后css样式丢失

把静态资源的相对路径改为绝对路径

举个例子，第一个是相对路径，第二个是绝对路径

```
<link href="asserts/css/signin.css" rel="stylesheet">
```

```
<link href="/asserts/css/signin.css" rel="stylesheet">
```

#### 登录请求

```java
@Controller
public class LoginController {
    @RequestMapping(value = "/user/login", method = RequestMethod.POST)
    public String login(String username, String password, Map<String, Object> map) {
        if (check(username, password)) {
            return "dashboard";
        } else {
            map.put("loginmsg", "用户名或密码错误!");
            return "index";
        }
    }

    private boolean check(String username, String password) {
        if ("admin".equals(username) && "123456".equals(password)) {
            return true;
        } else {
            return false;
        }
    }
}
```

可以使用thymeleaf模板引擎实现错误消息提示

```html
<p style="color: red" th:text="${loginmsg}" th:if="${not #strings.isEmpty(loginmsg)}"></p>
```

为了实现登陆之后重复提交表单，可以重定向到新的页面。

有一个问题，由于springmvc直接用redirect重定向是不会经过模板引擎的解析的

```
return "redirect:/dashboard.html";
```

所以，为了方便起见，一般都是设定一个中间请求，之后再转发。

```java
//解析转发
public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/main.html").setViewName("dashboard");
}
——————————————————————————————————————————————————————————————————————
  public String login(String username, String password, Map<String, Object> map) {
        if (check(username, password)) {
            //中间请求
            return "redirect:/main.html";
        } else {
            map.put("loginmsg", "用户名或密码错误!");
            return "index";
        }
    }
```

#### 登录请求的拦截器

1. 编写拦截器

   ```java
   package com.yiqzq.demo.component;
   
   import org.springframework.web.servlet.HandlerInterceptor;
   import org.springframework.web.servlet.ModelAndView;
   import org.thymeleaf.util.StringUtils;
   
   import javax.servlet.http.HttpServletRequest;
   import javax.servlet.http.HttpServletResponse;
   import javax.servlet.http.HttpSession;
   
   /**
    * @author yiqzq
    * @date 2020/5/26 13:38
    */
   public class LoginHandlerInterceptor implements HandlerInterceptor {
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           HttpSession session = request.getSession();
           String username = (String) session.getAttribute("username");
           if (!StringUtils.isEmpty(username)) {
               return true;
           } else {
               request.setAttribute("loginmsg", "没有权限,请先登录");
               request.getRequestDispatcher("/").forward(request, response);
               return false;
           }
       }
   
       @Override
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
   
       }
   
       @Override
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
   
       }
   }
   ```

2. 把拦截器配置进容器中

   需要注意的是，可能会拦截掉静态资源，我这里的本地是没没有什么问题的，如果有问题，建议添加`excludePathPatterns("/static/**","webjars/**")`

   ``

   ```java
   @Override
   public void addInterceptors(InterceptorRegistry registry) {
       registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**")
               .excludePathPatterns("/", "/index.html","/user/login");
   }
   ```

#### thymeleaf公共页面元素抽取

```html
1、抽取公共片段
<div th:fragment="copy">
&copy; 2011 The Good Thymes Virtual Grocery
</div>

2、引入公共片段
<div th:insert="~{footer :: copy}"></div>
~{templatename::selector}：模板名::选择器
~{templatename::fragmentname}:模板名::片段名

3、默认效果：
insert的公共片段在div标签中
如果使用th:insert等属性进行引入，可以不用写~{}：
行内写法可以加上：[[~{}]];[(~{})]；
```



三种引入公共片段的th属性：

**th:insert**：将公共片段整个插入到声明引入的元素中

**th:replace**：将声明引入的元素替换为公共片段

**th:include**：将被引入的片段的内容包含进这个标签中

```html
<footer th:fragment="copy">
&copy; 2011 The Good Thymes Virtual Grocery
</footer>

引入方式
<div th:insert="footer :: copy"></div>
<div th:replace="footer :: copy"></div>
<div th:include="footer :: copy"></div>

效果
<div>
    <footer>
    &copy; 2011 The Good Thymes Virtual Grocery
    </footer>
</div>

<footer>
&copy; 2011 The Good Thymes Virtual Grocery
</footer>

<div>
&copy; 2011 The Good Thymes Virtual Grocery
</div>
```

#### RESTful风格修改

需要加上如下的配置

```
spring.mvc.hiddenmethod.filter.enabled=true
```

**比如说发送put请求修改数据**

1. 配置HiddenHttpMethodFilter
2. 创建post表单
3. 创建一个input项，name=“_method ”，value是请求方式

#### springboot错误处理机制

原理：

​	可以参照ErrorMvcAutoConfiguration；错误处理的自动配置；

  	给容器中添加了以下组件

​	1、DefaultErrorAttributes：添加model数据

```java
帮我们在页面共享信息；
@Override
	public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes,
			boolean includeStackTrace) {
		Map<String, Object> errorAttributes = new LinkedHashMap<String, Object>();
		errorAttributes.put("timestamp", new Date());
		addStatus(errorAttributes, requestAttributes);
		addErrorDetails(errorAttributes, requestAttributes, includeStackTrace);
		addPath(errorAttributes, requestAttributes);
		return errorAttributes;
	}
```



​	2、BasicErrorController：处理默认/error请求

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    
    @RequestMapping(produces = "text/html")//产生html类型的数据；浏览器发送的请求来到这个方法处理
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
        
        //去哪个页面作为错误页面；包含页面地址和页面内容
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
	}

	@RequestMapping
	@ResponseBody    //产生json数据，其他客户端来到这个方法处理；
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<Map<String, Object>>(body, status);
	}
```



​	3、ErrorPageCustomizer：

```java
	@Value("${error.path:/error}")
	private String path = "/error";  系统出现错误以后来到error请求进行处理；（web.xml注册的错误页面规则）
```



​	4、DefaultErrorViewResolver：

```java
@Override
	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status,
			Map<String, Object> model) {
		ModelAndView modelAndView = resolve(String.valueOf(status), model);
		if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
			modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
		}
		return modelAndView;
	}

	private ModelAndView resolve(String viewName, Map<String, Object> model) {
        //默认SpringBoot可以去找到一个页面？  error/404
		String errorViewName = "error/" + viewName;
        
        //模板引擎可以解析这个页面地址就用模板引擎解析
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders
				.getProvider(errorViewName, this.applicationContext);
		if (provider != null) {
            //模板引擎可用的情况下返回到errorViewName指定的视图地址
			return new ModelAndView(errorViewName, model);
		}
        //模板引擎不可用，就在静态资源文件夹下找errorViewName对应的页面   error/404.html
		return resolveResource(errorViewName, model);
	}
```



​	步骤：

​		一但系统出现4xx或者5xx之类的错误；ErrorPageCustomizer就会生效（定制错误的响应规则）；就会来到/error请求；就会被**BasicErrorController**处理；

​		1）响应页面；去哪个页面是由**DefaultErrorViewResolver**解析得到的；

```java
protected ModelAndView resolveErrorView(HttpServletRequest request,
      HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
    //所有的ErrorViewResolver得到ModelAndView
   for (ErrorViewResolver resolver : this.errorViewResolvers) {
      ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
      if (modelAndView != null) {
         return modelAndView;
      }
   }
   return null;
}
```

### 2）、如果定制错误响应：

#### 	**1）、如何定制错误的页面；**

​			**1）、有模板引擎的情况下；error/状态码;** 【将错误页面命名为  错误状态码.html 放在模板引擎文件夹里面的 error文件夹下】，发生此状态码的错误就会来到  对应的页面；

​			我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态码.html）；		

​			页面能获取的信息；

​				timestamp：时间戳

​				status：状态码

​				error：错误提示

​				exception：异常对象

​				2.x版本要使用exception，需要配置server.error.include-exception=true这个属性，默认为false

​				message：异常消息

​				errors：JSR303数据校验的错误都在这里

​			2）、没有模板引擎（模板引擎找不到这个错误页面），静态资源文件夹下找；

​			3）、以上都没有错误页面，就是默认来到SpringBoot默认的错误提示页面；

#### 	2）、如何定制错误的json数据；

​		1）、自定义异常处理&返回定制json数据；

```java
@ControllerAdvice
public class MyExceptionHandler {

    @ResponseBody
    @ExceptionHandler(UserNotExistException.class)
    public Map<String,Object> handleException(Exception e){
        Map<String,Object> map = new HashMap<>();
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        return map;
    }
}
//没有自适应效果...
```



​		2）、转发到/error进行自适应响应效果处理

```java
 @ExceptionHandler(UserNotExistException.class)
    public String handleException(Exception e, HttpServletRequest request){
        Map<String,Object> map = new HashMap<>();
        //传入我们自己的错误状态码  4xx 5xx，否则就不会进入定制错误页面的解析流程
        /**
         * Integer statusCode = (Integer) request
         .getAttribute("javax.servlet.error.status_code");
         */
        request.setAttribute("javax.servlet.error.status_code",500);
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        //转发到/error
        return "forward:/error";
    }
```

#### 	3）、将我们的定制数据携带出去；

出现错误以后，会来到/error请求，会被BasicErrorController处理，响应出去可以获取的数据是由getErrorAttributes得到的（是AbstractErrorController（ErrorController）规定的方法）；

​	1、完全来编写一个ErrorController的实现类【或者是编写AbstractErrorController的子类】，放在容器中；

​	2、页面上能用的数据，或者是json返回能用的数据都是通过errorAttributes.getErrorAttributes得到；

​			容器中DefaultErrorAttributes.getErrorAttributes()；默认进行数据处理的；

自定义ErrorAttributes

```java
//给容器中加入我们自己定义的ErrorAttributes
@Component
public class MyErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
        Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
        map.put("company","atguigu");
        return map;
    }
}
```

最终的效果：响应是自适应的，可以通过定制ErrorAttributes改变需要返回的内容，

## 配置嵌入式Servlet容器

1. 在配置文件中修改

```properties
server.port=8081
server.context-path=/crud

server.tomcat.uri-encoding=UTF-8

//通用的Servlet容器设置
server.xxx
//Tomcat的设置
server.tomcat.xxx
```

2. 编写一个WebServerFactoryCustomizer：嵌入式的Servlet容器的定制器；来修改Servlet容器的配置

```java
public class MyWebServerFactoryCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        factory.setPort(8888);
    }
}
------------------------------------------------------------------
 @Bean
    public MyWebServerFactoryCustomizer webServerFactoryCustomizer() {
        return new MyWebServerFactoryCustomizer();
    }
```

## 注册servlet，filter，listener到tomcat中

```java
@Configuration
public class MyServletConfig {
    @Bean
    public ServletRegistrationBean<HttpServlet> myServlet() {
        ServletRegistrationBean<HttpServlet> servlet = new ServletRegistrationBean<HttpServlet>(
                new MyServlet(), "/myServlet");
        servlet.setLoadOnStartup(1);
        return servlet;
    }

    @Bean
    public ServletListenerRegistrationBean<ServletContextListener> myListener() {
        return new ServletListenerRegistrationBean<ServletContextListener>(new MyListener());
    }

    @Bean
    public FilterRegistrationBean<Filter> myFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<Filter>();
        //注册 过滤器
        filterRegistrationBean.setFilter(new MyFilter());
        //设置拦截路径
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/hello", "/myServlet"));
        return filterRegistrationBean;
    }


}
```

```java
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException, IOException {
        System.out.println("MyFilter 执行了");
        //放行
        filterChain.doFilter(request, response);
    }

    @Override
    public void destroy() {

    }
}
```

```java
public class MyListener implements ServletContextListener {

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("contextInitialized servlet 容器启动---------");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("contextDestroyed 服务器关闭-----------");
    }
}
```

```java
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException, IOException {
        resp.getWriter().write("MyServlet");
    }
}
```


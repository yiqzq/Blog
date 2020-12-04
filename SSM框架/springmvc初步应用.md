# springmvc初步应用

[toc]

## mvc

mvc即model，view，controller。

model一般指dao，service

view即jsp，html页面

controller指ervlet

## /和/*的区别

/ 拦截所有请求，不拦截jsp

/* 拦截所有请求，拦截jsp

## 使用步骤

导入依赖

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.2</version>
    </dependency>
    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
</dependencies>
```

配置web.xml文件

需要注意的是，要添加contextConfigLocation这个标签指示spring的配置文件的位置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
<filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

配置spring-servlet配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!--包扫描-->
    <context:component-scan base-package="com.yiqzq.controller"/>
    <!--    过滤静态资源-->
    <mvc:default-servlet-handler/>
    <!--    注解驱动-->
    <mvc:annotation-driven/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

注解写一个简单的controller

```java
package com.yiqzq.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * @author yiqzq
 * @date 2020/5/17 10:55
 */
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public String Hello(Model model) {
        model.addAttribute("msg", "hello");
        return "/hello";
    }
}
```

访问网页即可

## restful风格

普通的使用键值对方式传值，而restful使用/分割来表示传输的数据，可以通过method来区分请求，看似路径相同，其实可以执行不同的请求

可以使用@PathVariable 注解来获得路径的值

```java
@RequestMapping("/add/{a}/{b}")
public String test01(@PathVariable int a,@PathVariable int b, Model model){
    model.addAttribute("msg", a+b);
    return "/hello";
}
```

```java
//以下两个注解是等价的
@RequestMapping(method = RequestMethod.GET)
@GetMapping()
```

## 结果跳转

也可以使用servlet的api，也就是使用request和response

```java
public String test02(HttpServletRequest req, HttpServletResponse res){
        return "/hello";
}
```

也可以使用springmvc的方式实现

其中默认的就是请求转发

对于重定向，可以使用下面的方式,需要注意的是，由于配置了视图解析器，所以需要加上.jsp，不然找不到对应的文件

```java
return "redirect:/index.jsp";
```

@RequestParam 用于匹配前端传过来的数据

## Json

前后端分离一般采用的是json来传输数据

 @ResponseBody注解会跳过视图解析器，返回一个字符串，@RestController也能有同样的效果，只不过这个是放在类上的， @ResponseBody是放在方法上的

## 拦截器

```java
public class MyInterceptor implements HandlerInterceptor {
    //false 不放行 
    // true 放行,执行下一个拦截器
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return false;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```
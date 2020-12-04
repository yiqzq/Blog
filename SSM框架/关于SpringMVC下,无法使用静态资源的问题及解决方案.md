[toc]



### 原因

在我们配置`SpringMVC`的时候,在`web.xml`中配置过一个`DispatcherServlet`,代码如下

```java
	<!--配置springmvc DispatcherServlet，前端控制器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/springmvc-config.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <!-- 拦截器配置，拦截所有请求 -->
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

我们在`url-pattern`拦截了所有的请求(除了`jsp`),然后`SpringMVC`就会在容器中寻找对应的处理器去处理这个请求,这也就导致了如果请求的是静态资源(图片，`js`等),那么就会无法访问成功。 

#### 顺带一提，关于为什么没有拦截掉jsp页面请求的原因

因为在`tomcat`的配置文件`web.xml`中，配置了两个`servlet`

```xml
	<servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
        <init-param>
            <param-name>debug</param-name>
            <param-value>0</param-value>
        </init-param>
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
<!-- The mapping for the default servlet -->
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
	
	<servlet>
        <servlet-name>jsp</servlet-name>
        <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
        <init-param>
            <param-name>fork</param-name>
            <param-value>false</param-value>
        </init-param>
        <init-param>
            <param-name>xpoweredBy</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>3</load-on-startup>
    </servlet>

    <!-- The mappings for the JSP servlet -->
    <servlet-mapping>
        <servlet-name>jsp</servlet-name>
	        <url-pattern>*.jsp</url-pattern>
        <url-pattern>*.jspx</url-pattern>
    </servlet-mapping>

```

然后如果我们配置了`DispatcherServlet`，就会覆盖掉默认的`DefaultServlet`，导致静态资源解析失败，但是如果传过来的是`jsp`文件，因为一个优先级的原因，如图	<img src="C:\Users\39268\Desktop\QQ截图20200303170932.png" style="zoom:50%;" />

所以`tomcat`配置的`JspServlet`会优先`DispatcherServlet`执行，也就解释了为什么当请求`jsp`页面时，没有被拦截的原因。

### 解决方案

1. 为`spring`的配置文件中配置

   ```xml
    <mvc:default-servlet-handler/>
   ```

   加上这行代码，`SpringMVC`在运行的时候，会有![](C:\Users\39268\Desktop\QQ截图20200303173351.png)

使用`SimpleUrlHandlerMapping`来处理这个映射关系，映射关系如如下图![image-20200303173613277](C:\Users\39268\AppData\Roaming\Typora\typora-user-images\image-20200303173613277.png)

我们可以看到，这个`map`可以拦截所有的请求，用`DefaultServletHttpRequestHandler`来处理，然后这个`handler`实际上就会使用`tomcat`默认的`servlet`来处理，这样就能够加载静态资源了。

当然这还没有结束，如果只是单纯的加这一行代码，**结果就是虽然能够访问静态资源了，但是所有用注解@RequestMapping()全都失效了。**

原因也很简单，缺少了一个关于注解的`mapping`，因为下图的这两个，都不能够解析注解

![image-20200303190057672](C:\Users\39268\AppData\Roaming\Typora\typora-user-images\image-20200303190057672.png)	

所以，只需要再手动配置了一个关于注解的`Mapping`即可。

​	这里有两种方式

```java
	<!-- 1. 配置RequestMappingHandlerMapping -->
    <!-- 配置annotation类型的处理映射器，它根据请求查找映射 -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
    </bean>
    <!-- 配置annotation类型的处理器适配器，完成对@RequestMapping标注方法的调用 -->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
    <!-- 配置视图解析器 -->

```

又或者是直接配置注解驱动

```xml
  <mvc:annotation-driven/>
```


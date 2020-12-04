# SSM整合

[toc]

文件结构

<img src="https://i.loli.net/2020/05/18/TKAsaBUHiv2Gue9.png" alt="image-20200518143640648" style="zoom:80%;" />

1. 导入依赖

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>org.example</groupId>
       <artifactId>SSM-demo</artifactId>
       <version>1.0-SNAPSHOT</version>
       <dependencies>
           <!--       单元测试-->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>
           <!--        数据库驱动-->
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>8.0.15</version>
           </dependency>
           <!--        连接池-->
           <dependency>
               <groupId>com.mchange</groupId>
               <artifactId>c3p0</artifactId>
               <version>0.9.5.2</version>
           </dependency>
           <!--        jsp,servlet,jstl的原生组件-->
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
           <!--mybatis-->
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.2</version>
           </dependency>
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis-spring</artifactId>
               <version>2.0.2</version>
           </dependency>
           <!--        spring-->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-webmvc</artifactId>
               <version>5.1.9.RELEASE</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-jdbc</artifactId>
               <version>5.2.2.RELEASE</version>
           </dependency>
           <!--        aop-->
           <dependency>
               <groupId>org.aspectj</groupId>
               <artifactId>aspectjweaver</artifactId>
               <version>1.9.4</version>
           </dependency>
       </dependencies>
       <!--静态资源过滤-->
       <build>
           <resources>
               <resource>
                   <directory>src/main/java</directory>
                   <includes>
                       <include>**/*.properties</include>
                       <include>**/*.xml</include>
                   </includes>
                   <filtering>false</filtering>
               </resource>
               <resource>
                   <directory>src/main/resources</directory>
                   <includes>
                       <include>**/*.properties</include>
                       <include>**/*.xml</include>
                   </includes>
                   <filtering>false</filtering>
               </resource>
           </resources>
       </build>
   
   </project>
   ```

2. 配置web.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
            version="4.0">
       <servlet>
           <servlet-name>DispatcherServlet</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <!--一定要注意:我们这里加载的是总的配置文件，之前被这里坑了！-->
               <param-value>classpath:applicationContext.xml</param-value>
           </init-param>
           <load-on-startup>1</load-on-startup>
       </servlet>
       <servlet-mapping>
           <servlet-name>DispatcherServlet</servlet-name>
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   
       <!--encodingFilter-->
       <filter>
           <filter-name>encodingFilter</filter-name>
           <filter-class>
               org.springframework.web.filter.CharacterEncodingFilter
           </filter-class>
           <init-param>
               <param-name>encoding</param-name>
               <param-value>utf-8</param-value>
           </init-param>
       </filter>
       <filter-mapping>
           <filter-name>encodingFilter</filter-name>
           <url-pattern>/*</url-pattern>
       </filter-mapping>
   
       <!--Session过期时间-->
       <session-config>
           <session-timeout>15</session-timeout>
       </session-config>
   </web-app>
   ```

3. 数据库properties文件

   ```properties
   driver=com.mysql.cj.jdbc.Driver
   username=root
   password=123456
   url=jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone = GMT
   ```

4. mybatis的配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
   
       <typeAliases>
           <package name="com.yiqzq.pojo"/>
       </typeAliases>
       <mappers>
           <mapper class="com.yiqzq.dao.BookMapper"/>
       </mappers>
   </configuration>
   ```

5. spring和mybatis的整合文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          https://www.springframework.org/schema/context/spring-context.xsd">
   
       <!-- 配置整合mybatis -->
       <!-- 1.关联数据库文件 -->
       <context:property-placeholder location="classpath:db.properties"/>
   
       <!-- 2.数据库连接池 -->
       <!--数据库连接池
           dbcp 半自动化操作 不能自动连接
           c3p0 自动化操作（自动的加载配置文件 并且设置到对象里面）
       -->
       <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
           <!-- 配置连接池属性 -->
           <property name="driverClass" value="${driver}"/>
           <property name="jdbcUrl" value="${url}"/>
           <property name="user" value="${username}"/>
           <property name="password" value="${password}"/>
   
           <!-- c3p0连接池的私有属性 -->
           <property name="maxPoolSize" value="30"/>
           <property name="minPoolSize" value="10"/>
           <!-- 关闭连接后不自动commit -->
           <property name="autoCommitOnClose" value="false"/>
           <!-- 获取连接超时时间 -->
           <property name="checkoutTimeout" value="10000"/>
           <!-- 当获取连接失败重试次数 -->
           <property name="acquireRetryAttempts" value="2"/>
       </bean>
   
       <!-- 3.配置SqlSessionFactory对象 -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           <!-- 注入数据库连接池 -->
           <property name="dataSource" ref="dataSource"/>
           <!-- 配置MyBaties全局配置文件:mybatis-config.xml -->
           <property name="configLocation" value="classpath:mybatis-config.xml"/>
       </bean>
   
       <!-- 4.配置扫描Dao接口包，动态实现Dao接口注入到spring容器中 -->
       <!--解释 ：https://www.cnblogs.com/jpfss/p/7799806.html-->
       <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
           <!-- 注入sqlSessionFactory -->
           <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
           <!-- 给出需要扫描Dao接口包 -->
           <property name="basePackage" value="com.yiqzq.dao"/>
       </bean>
   
   </beans>
   ```

6. spring和springmvc的整合部分

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
   
       <!-- 配置SpringMVC -->
       <!-- 1.开启SpringMVC注解驱动 -->
       <mvc:annotation-driven/>
       <!-- 2.静态资源默认servlet配置-->
       <mvc:default-servlet-handler/>
   
       <!-- 3.配置jsp 显示ViewResolver视图解析器 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
           <property name="prefix" value="/WEB-INF/jsp/"/>
           <property name="suffix" value=".jsp"/>
       </bean>
   
       <!-- 4.扫描web相关的bean -->
       <context:component-scan base-package="com.yiqzq.controller"/>
   
   </beans>
   ```

7. spring-service文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
       <context:component-scan base-package="com.yiqzq.service"/>
       <!--BookServiceImpl注入到IOC容器中-->
       <bean id="BookServiceImpl" class="com.yiqzq.service.BookServiceImpl">
           <property name="bookMapper" ref="bookMapper"/>
       </bean>
   
       <!-- 配置事务管理器 -->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <!-- 注入数据库连接池 -->
           <property name="dataSource" ref="dataSource"/>
       </bean>
   </beans>
   ```

8. 总的spring文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/cache"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd">
   
           <import resource="spring-dao.xml"/>
           <import resource="spring-service.xml"/>
           <import resource="spring-mvc.xml"/>
   
   </beans>
   ```
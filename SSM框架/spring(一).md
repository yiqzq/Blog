[toc]

## spring的优点

1. 开源免费
2. 轻量级，非入侵式的容器
3. 控制反转，面向切面编程
4. 支持事务处理

## 普通的web结构

用户->业务service层->dao层

## spring的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
</beans>
```

## 导入的maven依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.1.9.RELEASE</version>
</dependency>
```

## IOC创建对象的方式

1. 默认创建方式使用的是无参构造器，每一个property标签调用的其实是set方法。

   ```xml
    <property name="" value=""/>
   ```

2. 使用有参构造方法，使用index索引表示参数

```xml
<bean id="user01" class="com.yiqzq.pojo.User">
   <constructor-arg index="0" value="sjy"/>
   <constructor-arg index="1" value="1"/>
</bean>
```

2. 使用有参构造方法，使用type

   对应一个具体的有参构造方法，如果参数类型相同，则按照配置的先后顺序赋值

   注意：这里的基本类型最好使用包装类

   同时，这里创建bean的去寻找构造方法的方式是匹配参数类型的

   ```xml
   <constructor-arg type="java.lang.String" value="sjy"/>
   <constructor-arg type="java.lang.String" value="1"/>
   ```

3. 使用有参构造器，使用构造方法参数 name

## 依赖注入的方式

一.目前使用最广泛的 @Autowired：自动装配

二.setter 方法注入

三:构造器注入

四丶静态工厂的方法注入

 五丶实例工厂的方法注入

## 一些元素依赖注入的方式

Properties，List，Map，Set

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

空串

```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

null值

```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

p名称空间

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="classic" class="com.example.ExampleBean">
        <property name="email" value="someone@somewhere.com"/>
    </bean>

    <bean name="p-namespace" class="com.example.ExampleBean"
        p:email="someone@somewhere.com"/>
</beans>
```

c名称空间

和p名称空间类似，只不过p名称空间是针对setter注入，c名称空间针对构造器注入

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>

    <!-- traditional declaration with optional argument names -->
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg name="thingTwo" ref="beanTwo"/>
        <constructor-arg name="thingThree" ref="beanThree"/>
        <constructor-arg name="email" value="something@somewhere.com"/>
    </bean>

    <!-- c-namespace declaration with argument names -->
    <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
        c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>

</beans>
```

## bean的生命周期

1.  The Singleton Scope
2.   The Prototype Scope

## 基于注解开发的方式

开启配置：\<context:annotation-config/>

包扫描：  <context:component-scan base-package=""/>

1.**@Required**

   该**@Required**注解适用于bean属性setter方法，并表示受影响的bean属性必须在XML配置文件在配置时进行填充。否则，容器会抛出一个BeanInitializationException异常。不过现在已经不推荐使用了。

2.**@Autowired**

   Autowired顾名思义，就是自动装配，其作用是为了消除代码Java代码里面的getter/setter与bean属性中的property。

Autowired首先根据type类型去容器中匹配，如果有多个，就会根据name去匹配，如果没有就匹配不成功。这时候就需要Qualifier来指定具体的bean。

3.@Component,@Service, @Controller,@Repository

用户表示bean的注入

4.@Value

bean属性的注入

## AOP

动态代理

**使用xml配置aop**

1. 写切面和通知

   ```java
   public class Log {
       public void beforeLog() {
           System.out.println("执行前...");
       }
   
       public void afterLog() {
           System.out.println("执行后...");
       }
   }
   ```

2. 写目标对象

   ```java
   public interface Animal {
       void show();
   }
   
   public class Cat implements Animal {
       public void show() {
           System.out.println("cat 猫!");
       }
   }
   
   ```

3. 导入aop依赖

   ```xml
    xmlns:aop="http://www.springframework.org/schema/aop"
    
    
    http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd
   ```

4. 在spring中配置bean

   ```xml
   <bean id="cat" class="com.yiqzq.pojo.Cat"/>
   <bean id="log" class="com.yiqzq.log.Log"/>
   ```

5. 书写aop配置

   ```xml
   <aop:config>
       <!--要引入的切面-->
       <aop:aspect ref="log">
           <!--写切入点-->
           <aop:pointcut id="p1" expression="execution(* com.yiqzq.pojo.Cat.*(..))"/>
           <!--配置具体的-->
           <aop:before method="beforeLog" pointcut-ref="p1"/>
           <aop:after method="afterLog" pointcut-ref="p1"/>
       </aop:aspect>
   </aop:config>
   ```

aop的顺序

环绕前-方法前-环绕后-方法后

## Spring整合mybatis

1. 导包

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-jdbc</artifactId>
           <version>5.2.2.RELEASE</version>
       </dependency>
       <dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis-spring</artifactId>
           <version>2.0.2</version>
       </dependency>
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-webmvc</artifactId>
           <version>5.1.9.RELEASE</version>
       </dependency>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.12</version>
       </dependency>
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <version>1.16.12</version>
       </dependency>
       <dependency>
           <groupId>org.aspectj</groupId>
           <artifactId>aspectjweaver</artifactId>
           <version>1.9.4</version>
       </dependency>
       <dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis</artifactId>
           <version>3.5.2</version>
       </dependency>
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>8.0.15</version>
       </dependency>
   
   </dependencies>
   ```

2. 写mybatis的配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <typeAliases>
        <package name="com.yiqzq.pojo"/>
    </typeAliases>
</configuration>
```

3. 写spring的配置文件

   主要时配置数据源，sqlsessionfactory，sqlsession（这里用SqlSessionTemplate）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:db.properties"/>
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!--                <property name="driverClassName" value="${driver}"/>-->
        <!--                <property name="url" value="${url}"/>-->
        <!--                <property name="username" value="${username}"/>-->
        <!--                <property name="password" value="${password}"/>-->
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url"
                  value="jdbc:mysql://localhost:3306/mybatis?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false&amp;serverTimezone = GMT"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="mybatis-config.xml"/>
        <property name="mapperLocations" value="com/yiqzq/dao/*.xml"/>
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>
    <!--这个是接口文件的实现类-->
    <bean id="userMapper" class="com.yiqzq.dao.UserMapperImpl">
        <property name="sqlSession" ref="sqlSession"/>
    </bean>
</beans>
```

4. 写实体类

   ```java
   public class User {
       private int id;
       private String name;
       private String pwd;
   }
   ```

5. 写接口方法

   ```java
   public interface UserMapper {
       public List<User> getUsers();
   }
   ```

6. 接口方法的sql实现

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <!--namespace是对应的java接口类 -->
   <mapper namespace="com.yiqzq.dao.UserMapper">
       <!--id是对用的方法名-->
       <select id="getUsers" resultType="user">
       select * from mybatis.user
     </select>
   </mapper>	
   ```

7. 可以配置在spring中的具体实现，需要组合sqlsession

   这里官方也给了第二种方法，继承sqlSessionDaoSupport，就可以通过方法getsession（）获得sqlsession，而不需要在视spring中注册sqlsession

   ```java
   public class UserMapperImpl implements UserMapper{
       private SqlSessionTemplate sqlSession;
   
       public void setSqlSession(SqlSessionTemplate sqlSession) {
           this.sqlSession = sqlSession;
       }
   
       public List<User> getUsers() {
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           return mapper.getUsers();
       }
   }
   ```

8. 编写测试类即可

   ```java
   public void test01() {
       ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");
       UserMapper mapper = context.getBean("userMapper", UserMapper.class);
       List<User> users = mapper.getUsers();
       for (User user : users) {
           System.out.println(user);
       }
   ```

## Spring事务管理

- **编程式事务管理：** 通过Transaction Template手动管理事务，实际应用中很少使用，
- **使用XML配置声明式事务：** 推荐使用（代码侵入性最小），实际是通过AOP实现

**使用方法**

```xml
<!--    配置声明式事务-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--    配置事务通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <!--给哪些方法配置事务,并设置传播特性-->
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>

<!--    配置事务切入-->
<aop:config>
    <aop:pointcut id="txPointCut" expression="execution(* com.yiqzq.dao.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
</aop:config>
```
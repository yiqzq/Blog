[toc]

## 导入的依赖

```xml
<dependencies>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.15</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.2</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
```

## 构建 SqlSessionFactory

每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。

1. 从XML种配置

   XML 配置文件中包含了对 MyBatis 系统的核心设置，包括获取数据库连接实例的数据源（DataSource）以及决定事务作用域和控制方式的事务管理器（TransactionManager）。

**简单实例**

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

2. 使用Java代码构建

   **简单实例**

   ```java
   DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
   TransactionFactory transactionFactory = new JdbcTransactionFactory();
   Environment environment = new Environment("development", transactionFactory, dataSource);
   Configuration configuration = new Configuration(environment);
   configuration.addMapper(BlogMapper.class);
   SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
   ```

## 通过SqlSessionFactory 中获取 SqlSession

SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。

**一个简单的工具类**

```java
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static  SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }

}
```

## 编写sql对应的xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace是对应的java接口类 -->
<mapper namespace="org.mybatis.example.BlogMapper">
    <!--id是对用的方法名-->
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

**当然，mybatis还同时支持注解书写sql语句的功能**

```java
package org.mybatis.example;
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  Blog selectBlog(int id);
}
```

**总结一下流程**

1. 设置Java接口类
2. 通过xml的方式实现接口类
3. 在mybatis的核心配置文件中注册对应的mapper
4. 通过sqlSession就可以调用具体的方法了

## SqlSessionFactoryBuilder和SqlSessionFactory和SqlSession

### SqlSessionFactoryBuilder

这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但最好还是不要一直保留着它，以保证所有的 XML 解析资源可以被释放给更重要的事情。

### SqlSessionFactory

SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

### SqlSession

每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。 一定要记得关闭。

## CRUD操作

注意：增删改需要提交事务

```java
   session.commit();
```

```
   session.commit();
```

parameterType：传入参数列表

当传入一个Java对象的时候，使用#{field}来获取值，field必须与对象属性字段一致

也可以传入一个Map\<String,Object>，那么field就需要与key一致

resultType：返回值类型

## mybatis的配置文件属性

配置文件中标签需要遵循以下的顺序

```xml
    (properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorFactory?,plugins?,environments?,databaseIdProvider?,mappers?)"
```

- configuration（配置）
  - properties（属性）
  - settings（设置）
  - typeAliases（类型别名）
  - typeHandlers（类型处理器）
  - objectFactory（对象工厂）
  - plugins（插件）
  - environments（环境配置）
    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）
  - databaseIdProvider（数据库厂商标识）
  - mappers（映射器）

### properties（属性）

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。

 **properties 元素的子元素中设置**

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

 **在Java 属性文件中配置**

```xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

如果一个属性在不只一个地方进行了配置，那么，MyBatis 将按照下面的顺序来加载：

- 首先读取在 properties 元素体内指定的属性。
- 然后根据 properties 元素中的 resource 属性读取类路径下属性文件，或根据 url 属性指定的路径读取属性文件，并覆盖之前读取过的同名属性。
- 最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性。

因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的则是 properties 元素中指定的属性。

### settings（设置）

**主要是配置一些mybatis的一些参数**

比如以下

| 设置名             | 描述      | 有效值   | 默认值 |
| ----------------- | ----------------------------------------------------------- |------------ | ----- |
| cacheEnabled       | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true/false | true   |
| lazyLoadingEnabled | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true/false | false  |
| mapUnderscoreToCamelCase | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true/false | False |

### typeAliases（类型别名）

**类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写**

按照如下配置，可以使用alias中的单词，替换type中的全限定类名

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```

同时，mybatis还支持批量注册别名，按照如下配置，每一个在包 `domain.blog` 中的 Java Bean，在没有注解的情况下，会使用 Bean 的**首字母小写的非限定类名**来作为它的别名。

```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

如果需要修改，则要采用注解

```java
@Alias("author")
public class Author {
    ...
}
```

### environments（环境配置）

MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中。

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

- 默认使用的环境 ID（比如：default="development"）。
- 每个 environment 元素定义的环境 ID（比如：id="development"）。
- 事务管理器的配置（比如：type="JDBC"）。
- 数据源的配置（比如：type="POOLED"）。

事务管理器：在 MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）：

数据源：有三种内建的数据源类型（也就是 type="[UNPOOLED|POOLED|JNDI]"）：

### mappers（映射器）

主要有以下三种

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

```xml
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

```xml
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

## 结果映射

`resultMap` 元素是 MyBatis 中最重要最强大的元素。

```xml
//这是个简单的例子，只是将字段映射到了HashMap上
<select id="selectUsers" resultType="map">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

使用as （sql语法）解决java对象和数据库表字段不一致的问题

```xml
<select id="selectUsers" resultType="User">
  select
    user_id             as "id",
    user_name           as "userName",
    hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>
```

正宗的resultMap

```xml
//type是要映射到的java对象
//在result中，property是java属性  column是与之对应的表字段
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="user_name"/>
  <result property="password" column="hashed_password"/>
</resultMap>

<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```

## log4j的配置文件

```properties
#将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/sjy.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

**log4j的简单使用**

```java
public class UserMapperTest {
    Logger logger= Logger.getLogger(UserMapperTest.class);

    @Test
    public void testLog(){
        logger.info("start");
    }
}
```

## 复杂查询的处理

### 嵌套 Select 查询

1. 建立实体类

   ```java
   public class Student {
       String sid;
       String name;
       Integer age;
       Teacher teacher;
   }
   ```

   ```java
   public class Teacher {
       String tid;
       String name;
   }
   ```

2. 同时建立对应数据库中的表

3. 编写Mapper文件，并准备对应的xml配置文件书写sql

   ```java
   public interface StudentMapper {
       public List<Student> getStudents();
   }
   ```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yiqzq.dao.StudentMapper">
    <select id="getStudents" resultMap="studentwithTeacher">
    select * from student
</select>
    <resultMap id="studentwithTeacher" type="student">
        <association property="teacher" javaType="Teacher" column="tid" select="getTeacher"/>
    </resultMap>
    <select id="getTeacher" resultType="teacher">
        select * from teacher where tid=#{tid}
    </select>
</mapper>
```

这里的联合查询涉及到association的使用

关联（association）元素处理“有一个”类型的关系。 这里表示一个学生有一个老师指导。

上面使用的是，嵌套 Select 查询：通过执行另外一个 SQL 映射语句来加载期望的复杂类型。

```xml
//property是java实体类中的字段名 javaType是实体类对应的java对象类 column是要嵌套查询的字段 select是嵌套查询语句
<association property="teacher" javaType="Teacher" column="tid" select="getTeacher"/>
```

| 属性     | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| `column` | 数据库中的列名，或者是列的别名。一般情况下，这和传递给 `resultSet.getString(columnName)` 方法的参数一样。 注意：在使用复合主键的时候，你可以使用 `column="{prop1=col1,prop2=col2}"` 这样的语法来指定多个传递给嵌套 Select 查询语句的列名。这会使得 `prop1` 和 `prop2` 作为参数对象，被设置为对应嵌套 Select 语句的参数。 |
| `select` | 用于加载复杂类型属性的映射语句的 ID，它会从 column 属性指定的列中检索数据，作为参数传递给目标 select 语句。 具体请参考下面的例子。注意：在使用复合主键的时候，你可以使用 `column="{prop1=col1,prop2=col2}"` 这样的语法来指定多个传递给嵌套 Select 查询语句的列名。这会使得 `prop1` 和 `prop2` 作为参数对象，被设置为对应嵌套 Select 语句的参数。 |

4. 编写测试类

   ```java
   public void test1() {
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
           List<Student> students = mapper.getStudents();
           for (Student s : students) {
               System.out.println(s);
           }
           sqlSession.close();
       }
   ```

### 嵌套结果映射

```java
public interface StudentMapper {
    public List<Student> getStudents2();
}
```

实体类都相同，只需要重新写一下xml文件

```xml
<select id="getStudents2" resultMap="studentwithTeacher2">
    select student.sid sid,student.name sname,teacher.name tname
    from student,teacher where student.tid = teacher.tid
</select>
<resultMap id="studentwithTeacher2" type="student">
    <result property="sid" column="sid"/>
    <result property="name" column="sname"/>
    <association property="teacher" javaType="Teacher">
        <result property="name" column="tname"/>
    </association>
</resultMap>
```

collection也是相似的，具体看官方文档https://mybatis.org/mybatis-3/zh/sqlmap-xml.html

ofType ：用来将JavaBean（或字段）属性的类型和集合存储的类型区分开来

```
private List<Post> posts;
<collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
```

## 动态sql简单使用

1. 实体类

```java
@Data
public class Blog {
    private int id;
    private String title;
    private String author;
    private Date createTime;
    private int views;
}
```

2. 映射接口

```java
public interface BlogMapper {
    List<Blog> getBlogs();

    List<Blog> getBlogsIF(HashMap<String, String> map);
}
```

3. 接口实现类

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yiqzq.dao.BlogMapper">
    <select id="getBlogs" resultType="blog">
        select * from Blog
    </select>

    <select id="getBlogsIF" resultType="blog">
        select * from Blog where 1=1
        <if test="title!=null">
            and title like #{title}
        </if>
        <if test="author!=null">
            and author like #{author}
        </if>
    </select>
</mapper>
```

4. 测试类

   ```java
   public class BlogMapperTest {
       @Test
       public void test01(){
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
           HashMap<String,String>map=new HashMap<String, String>();
           map.put("title","%java%");
           map.put("author","%sjy%");
           List<Blog> blogs = mapper.getBlogsIF(map);
           for (Blog b:blogs){
               System.out.println(b);
           }
           sqlSession.close();
       }
   }
   ```

其他关于choose，where，set标签的也是类似的。

## 缓存

一级缓存和二级缓存

一级缓存：默认开启，是sqlsession级别的，只在一次会话中

二级缓存：需要手动开启，是namespace级别的

- 映射语句文件中的所有 select 语句的结果将会被缓存。
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

关于二级缓存的问题

1. 所保存的数据需要实现序列化接口，否则会报错。并且由于反序列化时深拷贝，再次查询使用==判断是不相等的。
2. 只有当会话结束或者会话提交的时候，会提交到二级缓存。

**缓存顺序**

1. 先看二级缓存
2. 再看一级缓存
3. 查询数据库，并保存到一级缓存
# Mybatis



## Mybatis工作原理

1）读取 MyBatis 配置文件：mybatis-config.xml 为 MyBatis 的全局配置文件，配置了 MyBatis 的运行环境等信息，例如数据库连接信息。

2）加载映射文件。映射文件即 SQL 映射文件，该文件中配置了操作数据库的 SQL 语句，需要在 MyBatis 配置文件 mybatis-config.xml 中加载。mybatis-config.xml 文件可以加载多个映射文件，每个文件对应数据库中的一张表。

3）构造会话工厂：通过 MyBatis 的环境等配置信息构建会话工厂 SqlSessionFactory。

![MyBatis框架的执行流程图](http://c.biancheng.net/uploads/allimg/190704/5-1ZF4130T31N.png)
图 2 MyBatis 框架的执行流程图

4）创建会话对象：由会话工厂创建 SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法。

5）Executor 执行器：MyBatis 底层定义了一个 Executor 接口来操作数据库，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。

6）MappedStatement 对象：在 Executor 接口的执行方法中有一个 MappedStatement 类型的参数，该参数是对映射信息的封装，用于存储要映射的 SQL 语句的 id、参数等信息。

7）输入参数映射：输入参数类型可以是 Map、List 等集合类型，也可以是基本数据类型和 POJO 类型。输入参数映射过程类似于 JDBC 对 preparedStatement 对象设置参数的过程。

8）输出结果映射：输出结果类型可以是 Map、 List 等集合类型，也可以是基本数据类型和 POJO 类型。输出结果映射过程类似于 JDBC 对结果集的解析过程。

通过上面的讲解，读者对 MyBatis 框架应该有了一个初步的了解，在后续的学习中将慢慢加深理解。



---



## MyBatis的核心组件：SqlSessionFactoryBuilder、SqlSessionFactory、SqlSession和SQL Mapper

- SqlSessionFactoryBuilder（构造器）：它会根据配置或者代码来生成 SqlSessionFactory，采用的是分步构建的 Builder 模式。

- SqlSessionFactory（工厂接口）：依靠它来生成 SqlSession，使用的是工厂模式。

- SqlSession（会话）：一个既可以发送 SQL 执行返回结果，也可以获取 Mapper 的接口。在现有的技术中，一般我们会让其在业务逻辑代码中“消失”，而使用的是 MyBatis 提供的 SQL Mapper 接口编程技术，它能提

高代码的可读性和可维护性。

- SQL Mapper（映射器）:MyBatis 新设计存在的组件，它由一个 [Java](http://c.biancheng.net/java/) 接口和 XML 文件（或注解）构成，需要给出对应的 SQL 和映射规则。它负责发送 SQL 去执行，并返回结果。

用一张图来展示 MyBatis 核心组件之间的关系，如图 1 所示。

![MyBatis核心组件](http://c.biancheng.net/uploads/allimg/190704/5-1ZF4161629250.png)
图 1 MyBatis 核心组件


注意，无论是映射器还是 SqlSession 都可以发送 SQL 到数据库执行，下面学习这些组件的用法。

## MyBatis SqlSessionFactory及其常见创建方式

使用 MyBatis 首先是使用配置或者代码去生产 SqlSessionFactory，而 MyBatis 提供了构造器 SqlSessionFactoryBuilder。

它提供了一个类 org.apache.ibatis.session.Configuration 作为引导，采用的是 Builder 模式。具体的分步则是在 Configuration 类里面完成的，当然会有很多内容，包括你很感兴趣的插件。

在 MyBatis 中，既可以通过读取配置的 XML 文件的形式生成 SqlSessionFactory，也可以通过 [Java](http://c.biancheng.net/java/) 代码的形式去生成 SqlSessionFactory。

笔者强烈推荐采用 XML 的形式，因为代码的方式在需要修改的时候会比较麻烦。当配置了 XML 或者提供代码后，MyBatis 会读取配置文件，通过 Configuration 类对象构建整个 MyBatis 的上下文。

注意，SqlSessionFactory 是一个接口，在 MyBatis 中它存在两个实现类：SqlSessionManager 和 DefaultSqlSessionFactory。

一般而言，具体是由 DefaultSqlSessionFactory 去实现的，而 SqlSessionManager 使用在多线程的环境中，它的具体实现依靠 DefaultSqlSessionFactory，它们之间的关系如图 1 所示。

![SqlSessionFactory的生成](http://c.biancheng.net/uploads/allimg/190704/5-1ZF416213S21.png)
图 1 SqlSessionFactory 的生成


每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的，而 SqlSessionFactory 唯一的作用就是生产 MyBatis 的核心接口对象 SqlSession，所以它的责任是唯一的。我们往往会采用单例模式处理它，下面讨论使用配置文件和 Java 代码两种形式去生成 SqlSessionFactory 的方法。

## 

### 使用 XML 构建 SqlSessionFactory

首先，在 MyBatis 中的 XML 分为两类，一类是基础配置文件，通常只有一个，主要是配置一些最基本的上下文参数和运行环境；另一类是映射文件，它可以配置映射关系、SQL、参数等信息。

先看一份简易的基础配置文件，我们把它命名为 mybatis-config.xml，放在工程类路径下，其内容如下所示。

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases><!--别名-->
        <typeAliases alias="user" type="com.mybatis.po.User"/>
    </typeAliases>
    <!-- 数据库环境 -->
    <environments default="development">
        <environment id="development">
            <!-- 使用JDBC的事务管理 -->
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <!-- MySQL数据库驱动 -->
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <!-- 连接数据库的URL -->
                <property name="url"
                    value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8" />
                <property name="username" value="root" />
                <property name="password" value="1128" />
            </dataSource>
        </environment>
    </environments>
    <!-- 将mapper文件加入到配置文件中 -->
    <mappers>
        <mapper resource="com/mybatis/mapper/UserMapper.xml" />
    </mappers>
</configuration>
```

我们描述一下 MyBatis 的基础配置文件：

- <typeAlias> 元素定义了一个别名 user，它代表着 com.mybatis.po.User 这个类。这样定义后，在 MyBatis 上下文中就可以使用别名去代替全限定名了。
- <environment> 元素的定义，这里描述的是数据库。它里面的 <transactionManager> 元素是配置事务管理器，这里采用的是 MyBatis 的 JDBC 管理器方式。
- <dataSource> 元素配置数据库，其中属性 type="POOLED" 代表采用 MyBatis 内部提供的连接池方式，最后定义一些关于 JDBC 的属性信息。
- <mapper> 元素代表引入的那些映射器，在谈到映射器时会详细讨论它。


有了基础配置文件，就可以用一段很简短的代码来生成 SqlSessionFactory 了，如下所示。

```JAVA
SqlSessionFactory factory = null;
String resource = "mybatis-config.xml";
InputStream is;
try {
    InputStream is = Resources.getResourceAsStream(resource);
    factory = new SqlSessionFactoryBuilder().build(is);
} catch (IOException e) {
    e.printStackTrace();
}
```

首先读取 mybatis-config.xml，然后通过 SqlSessionFactoryBuilder 的 Builder 方法去创建 SqlSessionFactory。整个过程比较简单，而里面的步骤还是比较烦琐的，只是 MyBatis 采用了 Builder 模式为开发者隐藏了这些细节。这样一个 SqlSessionFactory 就被创建出来了。

采用 XML 创建的形式，信息在配置文件中，有利于我们日后的维护和修改，避免了重新编译代码，因此笔者推荐这种方式。

### 使用代码创建 SqlSessionFactory

虽然笔者不推荐使用这种方式，但是我们还是谈谈如何使用它。通过代码来实现与使用 XML 构建 SqlSessionFactory 一样的功能——创建 SqlSessionFactory，代码如下所示。

```java
// 数据库连接池信息
PooledDataSource dataSource = new PooledDataSource();
dataSource.setDriver("com.mysql.jdbc.Driver");
dataSource.setUsername("root");
dataSource.setPassword ("1128");
dataSource.setUrl("jdbc:mysql://localhost:3306/mybatis");
dataSource.setDefeultAutoCommit(false);
// 采用 MyBatis 的 JDBC 事务方式
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment ("development", transactionFactory, dataSource);
// 创建 Configuration 对象
Configuration configuration = new Configuration(environment);
// 注册一个 MyBatis 上下文别名
configuration.getTypeAliasRegistry().registerAlias("role", Role.class);
// 加入一个映射器
configuration.addMapper(RoleMapper.class);
//使用 SqlSessionFactoryBuilder 构建 SqlSessionFactory
SqlSessionFactory SqlSessionFactory =
new SqlSessionFactoryBuilder().build(configuration);
return SqlSessionFactory;
```

注意代码中的注释，它和 XML 方式实现的功能是一致的，只是方式不太一样而已。但是代码冗长，如果发生系统修改，那么有可能需要重新编译代码才能继续，所以这不是一个很好的方式。

- 除非有特殊的需要，比如在配置文件中，需要配置加密过的数据库用户名和密码，需要我们在生成 SqlSessionFactory 前解密为明文的时候，才会考虑使用这样的方式。

## MyBatis SqlSession简介

在 MyBatis 中，SqlSession 是其核心接口。在 MyBatis 中有两个实现类，DefaultSqlSession 和 SqlSessionManager。

DefaultSqlSession 是单线程使用的，而 SqlSessionManager 在多线程环境下使用。SqlSession 的作用类似于一个 JDBC 中的 Connection 对象，代表着一个连接资源的启用。具体而言，它的作用有 3 个：

- 获取 Mapper 接口。
- 发送 SQL 给数据库。
- 控制数据库事务。


先来掌握它的创建方法，有了 SqlSessionFactory 创建的 SqlSession 就十分简单了，如下所示。

``` JAVA 
SqlSession sqlSession = SqlSessionFactory.openSession();
```

注意，SqlSession 只是一个门面接口，它有很多方法，可以直接发送 SQL。它就好像一家软件公司的商务人员，是一个门面，而实际干活的是软件工程师。在 MyBatis 中，真正干活的是 Executor，我们会在底层看到它。

SqlSession 控制数据库事务的方法，如下所示。

```java
//定义 SqlSession
SqlSession sqlSession = null;
try {
    // 打开 SqlSession 会话
    sqlSession = SqlSessionFactory.openSession();
    // some code...
    sqlSession.commit();    // 提交事务
} catch (IOException e) {
    sqlSession.rollback();  // 回滚事务
}finally{
    // 在 finally 语句中确保资源被顺利关闭
    if(sqlSession != null){
        sqlSession.close();
    }
}
```

这里使用 commit 方法提交事务，或者使用 rollback 方法回滚事务。因为它代表着一个数据库的连接资源，使用后要及时关闭它，如果不关闭，那么数据库的连接资源就会很快被耗费光，整个系统就会陷入瘫痪状态，所以用 finally 语句保证其顺利关闭。

映射器是 MyBatis 中最重要、最复杂的组件，它由一个接口和对应的 XML 文件（或注解）组成。它可以配置以下内容：

- 描述映射规则。
- 提供 SQL 语句，并可以配置 SQL 参数类型、返回类型、缓存刷新等信息。
- 配置缓存。
- 提供动态 SQL。

## MyBatis实现映射器的2种方式：XML文件形式和注解形式

```sql
CREATE TABLE `role` (
    `id` bigint(20) NOT NULL,
    `role_name` varchar(20) DEFAULT NULL,
    `note` varchar(20) DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

并且定义一个 POJO，它十分简单，如下所示。

```JAVA
package com.mybatis.po;
public class Role {
    private Long id;
    private String roleName;
    private String note;
/**setter and getter**/
}
```

映射器的主要作用就是将 SQL 查询到的结果映射为一个 POJO，或者将 POJO 的数据插入到数据库中，并定义一些关于缓存等的重要内容。

注意，开发只是一个接口，而不是一个实现类。初学者可能会产生一个很大的疑问，那就是接口不是不能运行吗？

是的，接口不能直接运行。MyBatis 运用了动态代理技术使得接口能运行起来，入门阶段只要懂得 MyBatis 会为这个接口生成一个代理对象，代理对象会去处理相关的逻辑即可。

### 1-用 XML 实现映射器

用 XML 定义映射器分为两个部分：接口和 XML。先定义一个映射器接口，如下所示。

```
package com.mybatis.mapper;
import com.mybatis.po.Role;
public interface RoleMapper {
    public Role getRole(Long id);
}
```

在用 XML 方式创建 SqlSession 的配置文件中有这样一段代码：

```xml
<mapper resource="com/mybatis/mapper/RoleMapper.xml" />
```

它的作用就是引入一个 XML 文件。用 XML 方式创建映射器，如下所示。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mapper.RoleMapper">
    <select id="getRole" parameterType="long" resultType="role">
        SELECT id,role_name as roleName,note FROM role WHERE id =#{id}
    </select>
</mapper>
```

有了这两个文件，就完成了一个映射器的定义。XML 文件还算比较简单，我们稍微讲解一下：

<mapper> 元素中的属性 namespace 所对应的是一个接口的全限定名，于是 MyBatis 上下文就可以通过它找到对应的接口。

```tex
<select> 元素表明这是一条查询语句，而属性 id 标识了这条 SQL，属性 parameterType="long" 说明传递给 SQL 的是一个 long 型的参数，而 resultType="role" 表示返回的是一个 role 类型的返回值。而 role 是之前配置文件 mybatis-config.xml 配置的别名，指代的是 com.mybatis.po.Role。
```


这条 SQL 中的 #{id} 表示传递进去的参数。

注意，我们并没有配置 SQL 执行后和 role 的对应关系，它是如何映射的呢？

其实这里采用的是一种被称为自动映射的功能，MyBatis 在默认情况下提供自动映射，只要 SQL 返回的列名能和 POJO 对应起来即可。

这里 SQL 返回的列名 id 和 note 是可以和之前定义的 POJO 的属性对应起来的，而表里的列 role_name 通过 SQL 别名的改写，使其成为 roleName，也是和 POJO 对应起来的，所以此时 MyBatis 就可以把 SQL 查询的结果通过自动映射的功能映射成为一个 POJO。

### 2-注解实现映射器

除 XML 方式定义映射器外，还可以采用注解方式定义映射器，它只需要一个接口就可以通过 MyBatis 的注解来注入 SQL，如下所示。

```JAVA
package com.mybatis.mapper;
import org.apache.ibatis.annotations.Select;
import com.mybatis.po.Role;
public interface RoleMapper2 {
    @Select("select id,role_name as roleName,note from t_role where id=#{id}")
    public Role getRole(Long id);
}
```

这完全等同于 XML 方式创建映射器。也许你会觉得使用注解的方式比 XML 方式要简单得多。如果它和 XML 方式同时定义时，XML 方式将覆盖掉注解方式，所以 MyBatis 官方推荐使用的是 XML 方式，因此本教程以 XML 方式为主讨论 MyBatis 的应用。

在工作和学习中，SQL 的复杂度远远超过我们现在看到的 SQL，比如下面这条 SQL。

```sql
select * from t_user u
left join t_user_role ur on u.id = ur.user_id
left join t_role r on ur.role_id = r.id
left join t_user_info ui on u.id = ui.user_id
left join t_female_health fh on u.id = fh.user_id
left join t_male_health mh on u.id = mh.user_id
where u.user_name like concat('%', ${userName},'%')
and r.role_name like concat('%', ${roleName},'%')
and u.sex = 1
and ui.head_image is not null;
```

显然这条 SQL 比较复杂，如果放入 @Select 中会明显增加注解的内容。如果把大量的 SQL 放入 [Java](http://c.biancheng.net/java/) 代码中，显然代码的可读性也会下降。

如果同时还要考虑使用动态 SQL，比如当参数 userName 为空，则不使用 u.user_name like concat（'%',${userName},'%'）作为查询条件；当 roleName 为空，则不使用 r.role_name like concat（'%',${roleName},'%'）作为查询条件，但是还需要加入其他的逻辑，这样就使得这个注解更加复杂了，不利于日后的维护和修改。

此外，XML 可以相互引入，而注解是不可以的，所以在一些比较复杂的场景下，使用 XML 方式会更加灵活和方便。所以大部分的企业都是以 XML 为主，本教程也会保持一致，以 XML 方式来创建映射器。当然在一些简单的表和应用中使用注解方式也会比较简单。

这个接口可以在 XML 中定义，我们仿造在 mybatis-config.xml 中配置 XML 语句：

``` xml 
<mapper resource="com/mybatis/mapper/RoleMapper.xml" />
```

把它修改为下面的形式即可。

```XML
<mapper resource="com/mybatis/mapper/RoleMapper2" />
```

也可以使用 configuration 对象注册这个接口，比如：

```java
configuration.addMapper(RoleMapper2.class);
```





---

## MyBatis执行SQL的两种方式：SqlSession和Mapper接口

本节主要介绍 MyBatis 执行 SQL 语句的两种方式和它们的区别。

### 1-SqlSession 发送 SQL

有了映射器就可以通过 SqlSession 发送 SQL 了。我们以 getRole 这条 SQL 为例看看如何发送 SQL。

Role role = (Role)sqlSession.select("com.mybatis.mapper.RoleMapper.getRole",1L);

selectOne 方法表示使用查询并且只返回一个对象，而参数则是一个 String 对象和一个 Object 对象。这里是一个 long 参数，long 参数是它的主键。

String 对象是由一个命名空间加上 SQL id 组合而成的，它完全定位了一条 SQL，这样 MyBatis 就会找到对应的 SQL。如果在 MyBatis 中只有一个 id 为 getRole 的 SQL，那么也可以简写为：

Role role = (Role)sqlSession.selectOne("getRole",1L);

这是 MyBatis 前身 iBatis 所留下的方式。

### 2-用 Mapper 接口发送 SQL

SqlSession 还可以获取 Mapper 接口，通过 Mapper 接口发送 SQL，如下所示。

RoleMapper roleMapper = sqlSession.getMapper(RoleMapper.class);
Role role = roleMapper.getRole(1L);

通过 SqlSession 的 getMapper 方法来获取一个 Mapper 接口，就可以调用它的方法了。因为 XML 文件或者接口注解定义的 SQL 都可以通过“类的全限定名+方法名”查找，所以 MyBatis 会启用对应的 SQL 进行运行，并返回结果。

### 3-对比两种发送 SQL 方式

上面分别展示了 MyBatis 存在的两种发送 SQL 的方式，一种用 SqlSession 直接发送，另外一种通过 SqlSession 获取 Mapper 接口再发送。笔者建议采用 SqlSession 获取 Mapper 的方式，理由如下：

使用 Mapper 接口编程可以消除 SqlSession 带来的功能性代码，提高可读性，而 SqlSession 发送 SQL，需要一个 SQL id 去匹配 SQL，比较晦涩难懂。使用 Mapper 接口，类似 roleMapper.getRole（1L）则是完全面向对象的语言，更能体现业务的逻辑。

使用 Mapper.getRole（1L）方式，IDE 会提示错误和校验，而使用 sqlSession.selectOne（“getRole”,1L）语法，只有在运行中才能知道是否会产生错误。

目前使用 Mapper 接口编程已成为主流，尤其在 [Spring](http://c.biancheng.net/spring/) 中运用 MyBatis 时，Mapper 接口的使用就更为简单，所以本教程使用 Mapper 接口的方式讨论 MyBatis。

---



## SqlSessionFactoryBuilder、SqlSessionFactory和SqlSession的作用域以及生命周期

我们已经掌握了 MyBatis 组件的创建及其基本应用，但这是远远不够的，还需要讨论其生命周期。

生命周期是组件的重要问题，尤其是在多线程的环境中，比如互联网应用、Socket 请求等，而 MyBatis 也常用于多线程的环境中，错误使用会造成严重的多线程并发问题，为了正确编写 MyBatis 的应用程序，我们需要掌握 MyBatis 组件的生命周期。

所谓生命周期就是每一个对象应该存活的时间，比如一些对象一次用完后就要关闭，使它们被 [Java](http://c.biancheng.net/java/) 虚拟机（JVM）销毁，以避免继续占用资源，所以我们会根据每一个组件的作用去确定其生命周期。

### 1-SqlSessionFactoryBuilder

SqlSessionFactoryBuilder 的作用在于创建 SqlSessionFactory，创建成功后，SqlSessionFactoryBuilder 就失去了作用，所以它只能存在于创建 SqlSessionFactory 的方法中，而不要让其长期存在。因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。

### 2-SqlSessionFactory

SqlSessionFactory 可以被认为是一个数据库连接池，它的作用是创建 SqlSession 接口对象。因为 MyBatis 的本质就是 Java 对数据库的操作，所以 SqlSessionFactory 的生命周期存在于整个 MyBatis 的应用之中，所以一旦创建了 SqlSessionFactory，就要长期保存它，直至不再使用 MyBatis 应用，所以可以认为 SqlSessionFactory 的生命周期就等同于 MyBatis 的应用周期。

由于 SqlSessionFactory 是一个对数据库的连接池，所以它占据着数据库的连接资源。如果创建多个 SqlSessionFactory，那么就存在多个数据库连接池，这样不利于对数据库资源的控制，也会导致数据库连接资源被消耗光，出现系统宕机等情况，所以尽量避免发生这样的情况。

因此在一般的应用中我们往往希望 SqlSessionFactory 作为一个单例，让它在应用中被共享。所以说 SqlSessionFactory 的最佳作用域是应用作用域。

### 3-SqlSession

如果说 SqlSessionFactory 相当于数据库连接池，那么 SqlSession 就相当于一个数据库连接（Connection 对象），你可以在一个事务里面执行多条 SQL，然后通过它的 commit、rollback 等方法，提交或者回滚事务。

所以它应该存活在一个业务请求中，处理完整个请求后，应该关闭这条连接，让它归还给 SqlSessionFactory，否则数据库资源就很快被耗费精光，系统就会瘫痪，所以用 try...catch...finally... 语句来保证其正确关闭。

所以 SqlSession 的最佳的作用域是请求或方法作用域。

### 4-Mapper

Mapper 是一个接口，它由 SqlSession 所创建，所以它的最大生命周期至多和 SqlSession 保持一致，尽管它很好用，但是由于 SqlSession 的关闭，它的数据库连接资源也会消失，所以它的生命周期应该小于等于 SqlSession 的生命周期。Mapper 代表的是一个请求中的业务处理，所以它应该在一个请求中，一旦处理完了相关的业务，就应该废弃它。

以上，我们讨论了 MyBatis 组件的生命周期，如图 1 所示。

![MyBatis组件的生命周期](http://c.biancheng.net/uploads/allimg/190705/5-1ZF5104453328.png)
图 1 MyBatis 组件的生命周期

在创建项目之前，首先在 [MySQL](http://c.biancheng.net/mysql/) 数据库中创建 mybatis 数据库和 user 表，sql 语句如下所示：

```
CREATE DATABASE mybatis;USE mybatis;DROP TABLE IF EXISTS `user`;CREATE TABLE `user` (  `uid` tinyint(2) NOT NULL,  `uname` varchar(20) DEFAULT NULL,  `usex` varchar(10) DEFAULT NULL,  PRIMARY KEY (`uid`)) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```


下面通过一个实例讲解如何使用 MyEclipse 开发 MyBatis 入门程序。

---



## 第一个MyBatis程序



#### 1）创建 Web 应用，并添加相关 JAR 包

在 MyEclipse 中创建一个名为 myBatisDemo01 的 Web 应用，将 MyBatis 的核心 JAR 包、依赖 JAR 包以及 MySQL 数据库的驱动 JAR 包一起复制到 /WEB-INF/lib 目录下。添加后的 lib 目录如图 1 所示。



![MyBatis相关的JAR包](http://c.biancheng.net/uploads/allimg/190704/5-1ZF4133013333.png)
图 1 MyBatis相关的JAR包

#### 2）创建日志文件

MyBatis 默认使用 log4j 输出日志信息，如果开发者需要查看控制台输出的 SQL 语句，那么需要在 classpath 路径下配置其日志文件。在 myBatis应用的 src 目录下创建 log4j.properties 文件，其内容如下：

``` properties
\# Global logging configuration
log4j.rootLogger=ERROR,stdout
\# MyBatis logging configuration...
log4j.logger.com.mybatis=DEBUG
\# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

在日志文件中配置了全局的日志配置、MyBatis 的日志配置和控制台输出，其中 MyBatis 的日志配置用于将 com.mybatis 包下所有类的日志记录级别设置为 DEBUG。该配置文件内容不需要开发者全部手写，可以从 MyBatis 使用手册中的 Logging 小节复制，然后进行简单修改。

#### 3）创建持久化类

在 src 目录下创建一个名为 com.mybatis.po 的包，在该包中创建持久化类 MyUser，注意在类中声明的属性与数据表 user 的字段一致。

MyUser 的代码如下：

``` java
package com.mybatis.po;
/**
* springtest数据库中user表的持久类
*/
public class MyUser {
    private Integer uid; // 主键
    private String uname;
    private String usex;
    // 此处省略setter和getter方法
    @Override
    public String toString() { // 为了方便查看结果，重写了toString方法
        return "User[uid=" + uid + ",uname=" + uname + ",usex=" + usex + "]";
    }
}
```

#### 4）创建映射文件

在 src 目录下创建一个名为 com.mybatis.mapper 的包，在该包中创建映射文件 UserMapper.xml。

UserMapper.xml 文件的内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mapper.UserMapper">
    <!-- 根据uid查询一个用户信息 -->
    <select id="selectUserById" parameterType="Integer"
        resultType="com.mybatis.po.MyUser">
        select * from user where uid = #{uid}
    </select>
    <!-- 查询所有用户信息 -->
    <select id="selectAllUser" resultType="com.mybatis.po.MyUser">
        select * from user
    </select>
    <!-- 添加一个用户，#{uname}为 com.mybatis.po.MyUser 的属性值 -->
    <insert id="addUser" parameterType="com.mybatis.po.MyUser">
        insert into user (uname,usex)
        values(#{uname},#{usex})
    </insert>
    <!--修改一个用户 -->
    <update id="updateUser" parameterType="com.mybatis.po.MyUser">
        update user set uname =
        #{uname},usex = #{usex} where uid = #{uid}
    </update>
    <!-- 删除一个用户 -->
    <delete id="deleteUser" parameterType="Integer">
        delete from user where uid
        = #{uid}
    </delete>
</mapper>
```



在上述映射文件中，<mapper> 元素是配置文件的根元素，它包含了一个 namespace 属性，该属性值通常设置为“包名+SQL映射文件名”，指定了唯一的命名空间。

子元素 <select>、<insert>、<update> 以及 <delete> 中的信息是用于执行查询、添加、修改以及删除操作的配置。在定义的 SQL 语句中，“#{}”表示一个占位符，相当于“?”，而“#{uid}”表示该占位符待接收参数的名称为 uid。

#### 5）创建 MyBatis 的配置文件

在 src 目录下创建 MyBatis 的核心配置文件 mybatis-config.xml，在该文件中配置了数据库环境和映射文件的位置，具体内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?><!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN""http://mybatis.org/dtd/mybatis-3-config.dtd"><configuration>    <settings>        <setting name="logImpl" value="LOG4J" />    </settings>    <!-- 配置mybatis运行环境 -->    <environments default="development">        <environment id="development">            <!-- 使用JDBC的事务管理 -->            <transactionManager type="JDBC" />            <dataSource type="POOLED">                <!-- MySQL数据库驱动 -->                <property name="driver" value="com.mysql.jdbc.Driver" />                <!-- 连接数据库的URL -->                <property name="url"                    value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8" />                <property name="username" value="root" />                <property name="password" value="1128" />            </dataSource>        </environment>    </environments>    <!-- 将mapper文件加入到配置文件中 -->    <mappers>        <mapper resource="com/mybatis/mapper/UserMapper.xml" />    </mappers></configuration>
```

上述映射文件和配置文件都不需要读者完全手动编写，都可以从 MyBatis 使用手册中复制，然后做简单修改。

#### 6）创建测试类

在 src 目录下创建一个名为 com.mybatis.test 的包，在该包中创建 MyBatisTest 测试类。在测试类中首先使用输入流读取配置文件，然后根据配置信息构建 SqlSessionFactory 对象。

接下来通过 SqlSessionFactory 对象创建 SqlSession 对象，并使用 SqlSession 对象的方法执行数据库操作。 MyBatisTest 测试类的代码如下：

```JAVA
package com.mybatis.test;
import java.io.IOException;
import java.io.InputStream;
import java.util.List;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import com.mybatis.po.MyUser;
public class MyBatisTest {
    public static void main(String[] args) {
        try {
            // 读取配置文件 mybatis-config.xml
            InputStream config = Resources
                    .getResourceAsStream("mybatis-config.xml");
            // 根据配置文件构建SqlSessionFactory
            SqlSessionFactory ssf = new SqlSessionFactoryBuilder()
                    .build(config);
            // 通过 SqlSessionFactory 创建 SqlSession
            SqlSession ss = ssf.openSession();
            // SqlSession执行映射文件中定义的SQL，并返回映射结果
            /*
             * com.mybatis.mapper.UserMapper.selectUserById 为 UserMapper.xml
             * 中的命名空间+select 的 id
             */
            // 查询一个用户
            MyUser mu = ss.selectOne(
                    "com.mybatis.mapper.UserMapper.selectUserById", 1);
            System.out.println(mu);
            // 添加一个用户
            MyUser addmu = new MyUser();
            addmu.setUname("陈恒");
            addmu.setUsex("男");
            ss.insert("com.mybatis.mapper.UserMapper.addUser", addmu);
            // 修改一个用户
            MyUser updatemu = new MyUser();
            updatemu.setUid(1);
            updatemu.setUname("张三");
            updatemu.setUsex("女");
            ss.update("com.mybatis.mapper.UserMapper.updateUser", updatemu);
            // 删除一个用户
            ss.delete("com.mybatis.mapper.UserMapper.deleteUser", 3);
            // 查询所有用户
            List<MyUser> listMu = ss
                    .selectList("com.mybatis.mapper.UserMapper.selectAllUser");
            for (MyUser myUser : listMu) {
                System.out.println(myUser);
            }
            // 提交事务
            ss.commit();
            // 关闭 SqlSession
            ss.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```

上述测试类的运行结果如图 2 所示。



![MyBatis入门程序的运行结果](http://c.biancheng.net/uploads/allimg/190704/5-1ZF416014AR.png)
图 2 MyBatis 入门程序的运行结果



## MyBatis配置文件详解

MyBatis 配置文件并不复杂，它所有的元素如下所示。

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration><!-- 配置 -->
    <properties /><!-- 属性 -->
    <settings /><!-- 设置 -->
    <typeAliases /><!-- 类型命名 -->
    <typeHandlers /><!-- 类型处理器 -->
    <objectFactory /><!-- 对象工厂 -->
    <plugins /><!-- 插件 -->
    <environments><!-- 配置环境 -->
        <environment><!-- 环境变量 -->
            <transactionManager /><!-- 事务管理器 -->
            <dataSource /><!-- 数据源 -->
        </environment>
    </environments>
    <databaseIdProvider /><!-- 数据库厂商标识 -->
    <mappers /><!-- 映射器 -->
</configuration>
```

但是需要注意的是，MyBatis 配置项的顺序不能颠倒。如果颠倒了它们的顺序，那么在 MyBatis 启动阶段就会发生异常，导致程序无法运行。

本节的任务是了解 MyBatis 配置项的作用，其中 properties、settings、typeAliases、typeHandler、plugin、environments、mappers 是常用的内容。

本章不讨论 plugin（插件）元素的使用，在进一步学习 MyBatis 的许多底层内容和设计后我们才会学习它。MyBatis 中 objectFactory 和 databaseIdProvider 不常用。

---

### 1.MyBatis核心配置文件properties元素

properties 属性可以给系统配置一些运行参数，可以放在 XML 文件或者 properties 文件中，而不是放在 [Java](http://c.biancheng.net/java/) 编码中，这样的好处在于方便参数修改，而不会引起代码的重新编译。一般而言，MyBatis 提供了 3 种方式让我们使用 properties，它们是：

- property 子元素。
- properties 文件。
- 程序代码传递。

#### 1(property 子元素

以下面代码为基础，使用 property 子元素将数据库连接的相关配置进行改写，如下所示。

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties>
        <property name="driver" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8" />
        <property name="username" value="root" />
        <property name="password" value="1128" />
    </properties>
    <typeAliases>
        <typeAlias alias="role" type="com.mybatis.po.Role"/>
    </typeAliases>
    <!--数据库环境-->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url"    value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8" />
                <property name="username" value="root" />
                <property name="password" value="1128" />
            </dataSource>
        </environment>
    </environments>
    <!-- 映射文件 -->
    <mappers>
        <mapper resource="com/mybatis/mapper/RoleMapper.xml" />
    </mappers>
</configuration>
```

这里使用了元素 <properties> 下的子元素 <property> 定义，用字符串 database.username 定义数据库用户名，然后就可以在数据库定义中引入这个已经定义好的属性参数，如 ${database.username}，这样定义一次就可以到处引用了。但是如果属性参数有成百上千个，显然使用这样的方式不是一个很好的选择，这个时候可以使用 properties 文件。

#### 2(使用 properties 文件

使用 properties 文件是比较普遍的方法，一方面这个文件十分简单，其逻辑就是键值对应，我们可以配置多个键值放在一个 properties 文件中，也可以把多个键值放到多个 properties 文件中，这些都是允许的，它方便日后维护和修改。

我们创建一个文件 jdbc.properties 放到 classpath 的路径下，如下所示。

```prop
database.driver=com.mysql.jdbc.Driver
database.url=jdbc:mysql://localhost:3306/mybatis
database.username=root
database.password=1128
```

在 MyBatis 中通过 <properties> 的属性 resource 来引入 properties 文件。

<properties resource="jdbc.properties"/>

也可以按 ${database.username} 的方法引入 properties 文件的属性参数到 MyBatis 配置文件中。这个时候通过维护 properties 文件就可以维护我们的配置内容了。

#### 3(使用程序传递方式传递参数

在真实的生产环境中，数据库的用户密码是对开发人员和其他人员保密的。运维人员为了保密，一般都需要把用户和密码经过加密成为密文后，配置到 properties 文件中。

对于开发人员及其他人员而言，就不知道其真实的用户密码了，数据库也不可能使用已经加密的字符串去连接，此时往往需要通过解密才能得到真实的用户和密码了。

现在假设系统已经为提供了这样的一个 CodeUtils.decode（str）进行解密，那么我们在创建 SqlSessionFactory 前，就需要把用户名和密码解密，然后把解密后的字符串重置到 properties 属性中，如下所示。

```JAVA
String resource = "mybatis-config.xml";
InputStream inputStream;
Inputstream in = Resources.getResourceAsStream("jdbc.properties");
Properties props = new Properties();
props.load(in);
String username = props.getProperty("database.username");
String password = props.getProperty("database.password");
//解密用户和密码，并在属性中重置
props.put("database.username", CodeUtils.decode(username));
props.put ("database.password", CodeUtils.decode(password)); 
inputstream = Resources.getResourceAsStream(resource);
//使用程序传递的方式覆盖原有的properties属性参数
SqlSessionFactory = new SqlSessionFactoryBuilder().build(inputstream, props);
```

首先使用 Resources 对象读取了一个 jdbc.properties 配置文件，然后获取了它原来配置的用户和密码，进行解密并重置，最后使用 SqlSessionFactoryBuilder 的 build 方法，传递多个 properties 参数来完成。

这将覆盖之前配置的密文，这样就能连接数据库了，同时也满足了运维人员对数据库用户和密码安全的要求。

### 2.MyBatis中settings属性配置详解

在 MyBatis 中 settings 是最复杂的配置，它能深刻影响 MyBatis 底层的运行，但是在大部分情况下使用默认值便可以运行，所以在大部分情况下不需要大量配置它，只需要修改一些常用的规则即可，比如自动映射、驼峰命名映射、级联规则、是否启动缓存、执行器（Executor）类型等。settings 配置项说明，如表 1 所示。



| 配置项                            | 作用                                                         | 配置选项                                                     | 默认值                                                       |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| cacheEnabled                      | 该配置影响所有映射器中配置缓存的全局开关                     | true\|false                                                  | true                                                         |
| lazyLoadingEnabled                | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。在特定关联关系中可通过设置 fetchType 属性来覆盖该项的开关状态 | true\|false                                                  | false                                                        |
| aggressiveLazyLoading             | 当启用时，对任意延迟属性的调用会使带有延迟加载属性的对象完整加载；反之，每种属性将会按需加载 | true\|felse                                                  | 版本3.4.1 （不包含） 之前 true，之后 false                   |
| multipleResultSetsEnabled         | 是否允许单一语句返回多结果集（需要兼容驱动）                 | true\|false                                                  | true                                                         |
| useColumnLabel                    | 使用列标签代替列名。不同的驱动会有不同的表现，具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果 | true\|false                                                  | true                                                         |
| useGeneratedKeys                  | 允许JDBC 支持自动生成主键，需要驱动兼容。如果设置为 true，则这个设置强制使用自动生成主键，尽管一些驱动不能兼容但仍可正常工作（比如 Derby） | true\|false                                                  | false                                                        |
| autoMappingBehavior               | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射。 PARTIAL 表示只会自动映射，没有定义嵌套结果集和映射结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套） | NONE、PARTIAL、FULL                                          | PARTIAL                                                      |
| autoMappingUnkno wnColumnBehavior | 指定自动映射当中未知列（或未知属性类型）时的行为。 默认是不处理，只有当日志级别达到 WARN 级别或者以下，才会显示相关日志，如果处理失败会抛出 SqlSessionException 异常 | NONE、WARNING、FAILING                                       | NONE                                                         |
| defaultExecutorType               | 配置默认的执行器。SIMPLE 是普通的执行器；REUSE 会重用预处理语句（prepared statements）；BATCH 执行器将重用语句并执行批量更新 | SIMPLE、REUSE、BATCH                                         | SIMPLE                                                       |
| defaultStatementTimeout           | 设置超时时间，它决定驱动等待数据库响应的秒数                 | 任何正整数                                                   | Not Set (null)                                               |
| defaultFetchSize                  | 设置数据库驱动程序默认返回的条数限制，此参数可以重新设置     | 任何正整数                                                   | Not Set (null)                                               |
| safeRowBoundsEnabled              | 允许在嵌套语句中使用分页（RowBounds）。如果允许，设置 false  | true\|false                                                  | false                                                        |
| safeResultHandlerEnabled          | 允许在嵌套语句中使用分页（ResultHandler）。如果允许，设置false | true\|false                                                  | true                                                         |
| mapUnderscoreToCamelCase          | 是否开启自动驼峰命名规则映射，即从经典数据库列名 A_COLUMN 到经典 [Java](http://c.biancheng.net/java/) 属性名 aColumn 的类似映射 | true\|false                                                  | false                                                        |
| localCacheScope                   | MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速联复嵌套査询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlScssion 的不同调用将不会共享数据 | SESSION\|STATEMENT                                           | SESSION                                                      |
| jdbcTypeForNull                   | 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER | NULL、VARCHAR、OTHER                                         | OTHER                                                        |
| lazyLoadTriggerMethods            | 指定哪个对象的方法触发一次延迟加载                           | —                                                            | equals、clone、hashCode、toString                            |
| defaultScriptingLanguage          | 指定动态 SQL 生成的默认语言                                  | —                                                            | org.apache.ibatis .script.ing.xmltags .XMLDynamicLanguageDriver |
| callSettersOnNulls                | 指定当结果集中值为 null 时，是否调用映射对象的 setter（map 对象时为 put）方法，这对于 Map.kcySet() 依赖或 null 值初始化时是有用的。注意，基本类型（int、boolean 等）不能设置成 null | true\|false                                                  | false                                                        |
| logPrefix                         | 指定 MyBatis 增加到日志名称的前缀                            | 任何字符串                                                   | Not set                                                      |
| loglmpl                           | 指定 MyBatis 所用日志的具体实现，未指定时将自动査找          | SLF4J\|LOG4J\|LOG4J2\|JDK_LOGGING \|COMMONS_LOGGING \|ST DOUT_LOGGING\|NO_LOGGING | Not set                                                      |
| proxyFactory                      | 指定 MyBatis 创建具有延迟加栽能力的对象所用到的代理工具      | CGLIB\|JAVASSIST                                             | JAVASSIST （MyBatis 版本为 3.3 及以上的）                    |
| vfsImpl                           | 指定 VFS 的实现类                                            | 提供 VFS 类的全限定名，如果存在多个，可以使用逗号分隔        | Not set                                                      |
| useActualParamName                | 允许用方法参数中声明的实际名称引用参数。要使用此功能，项目必须被编译为 Java 8 参数的选择。（从版本 3.4.1 开始可以使用） | true\|false                                                  | true                                                         |

settings 的配置项很多，但是真正用到的不会太多，我们把常用的配置项研究清楚就可以了，比如关于缓存的 cacheEnabled，关于级联的 lazyLoadingEnabled 和 aggressiveLazy Loading，关于自动映射的 autoMappingBehavior 和 mapUnderscoreToCamelCase，关于执行器类型的 defaultExecutorType 等。

这里给出一个全量的配置样例，如下所示。

```xml
<settings>
    <setting name="cacheEnabled" value="true"/>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="multipleResultSetsEnabled" value="true"/>
    <setting name="useColumnLabel" value="true"/>
    <setting name="useGeneratedKeys" value="false"/>
    <setting name="autoMappingBehavior" value="PARTIAL"/>
    <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
    <setting name="defaultExecutorType" value="SIMPLE"/>
    <setting name="defaultStatementTimeout" value="25"/>
    <setting name="defaultFetchSize" value="100"/>
    <setting name="safeRowBoundsEnabled" value="false"/>
    <setting name="mapUnderscoreToCamelCase" value="false"/>
    <setting name="localCacheScope" value="SESSION"/>
    <setting name="jdbcTypeForNull" value="OTHER"/>
    <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

### 3.MyBatis配置typeAliases（别名）详解

由于类的全限定名称很长，需要大量使用的时候，总写那么长的名称不方便。在 MyBatis 中允许定义一个简写来代表这个类，这就是别名，别名分为系统定义别名和自定义别名。

在 MyBatis 中别名由类 TypeAliasRegistry（org.apache.ibatis.type.TypeAliasRegistry）去定义。注意，在 MyBatis 中别名不区分大小写。

#### 1(系统定义别名

在 MyBatis 的初始化过程中，系统自动初始化了一些别名，如下表所示。



| 别名       | [Java](http://c.biancheng.net/java/) 类型 | 是否支持数组 |
| ---------- | ----------------------------------------- | ------------ |
| _byte      | byte                                      | 是           |
| _long      | long                                      | 是           |
| _short     | short                                     | 是           |
| _int       | int                                       | 是           |
| _integer   | int                                       | 是           |
| _double    | double                                    | 是           |
| _float     | float                                     | 是           |
| _boolean   | boolean                                   | 是           |
| string     | String                                    | 是           |
| byte       | Byte                                      | 是           |
| long       | Long                                      | 是           |
| short      | Short                                     | 是           |
| int        | Integer                                   | 是           |
| integer    | Integer                                   | 是           |
| double     | Double                                    | 是           |
| float      | Float                                     | 是           |
| boolean    | Boolean                                   | 是           |
| date       | Date                                      | 是           |
| decimal    | BigDecimal                                | 是           |
| bigdecimal | BigDecimal                                | 是           |
| object     | Object                                    | 是           |
| map        | Map                                       | 否           |
| hashmap    | HashMap                                   | 否           |
| list       | List                                      | 否           |
| arraylist  | ArrayList                                 | 否           |
| collection | Collection                                | 否           |
| iterator   | Iterator                                  | 否           |
| ResultSet  | ResultSet                                 | 否           |

如果需要使用对应类型的数组型，要看其是否能支持数据，如果支持只需要使用别名加`[]`即可，比如 _int 数组的别名就是 _int[]。而类似 list 这样不支持数组的别名，则不能那么写。

有时候要通过代码来实现注册别名，让我们看看 MyBatis 是如何初始化这些别名的，如下所示。

```
public TypeAliasRegistry() {
    registerAlias("string", String.class);
    registerAlias("byte", Byte.class);
    registerAlias("long", Long.class);
    ......
    registerAlias("byte[]",Byte[].class); registerAlias("long[]",Long[].class);
    ......
    registerAlias("map", Map.class);
    registerAlias("hashmap", HashMap.class);
    registerAlias("list", List.class); registerAlias("arraylist", ArrayList.class);
    registerAlias("collection", Collection.class);
    registerAlias("iterator", Iterator.class);
    registerAlias("ResultSet", ResultSet.class);
}
```

所以使用 TypeAliasRegistry 的 registerAlias 方法就可以注册别名了。一般是通过 Configuration 获取 TypeAliasRegistry 类对象，其中有一个 getTypeAliasRegistry 方法可以获得别名，如 configuration.getTypeAliasRegistry()。

然后就可以通过 registerAlias 方法对别名注册了。而事实上 Configuration 对象也对一些常用的配置项配置了别名，如下所示。

```java
//事务方式别名
typeAliasRegistry.registerAlias("JDBC",JdbcTransactionFactory.class);
typeAliasRegistry.registerAlias("MANAGED",ManagedTransactionFactory.class);
//数据源类型别名
typeAliasRegistry.registerAlias("JNDI",JndiDataSourceFactory.class);
typeAliasRegistry.registerAlias("POOLED",
PooledDataSourceFactory.class);
typeAliasRegistry.registerAlias("UNPOOLED",UnpooledDataSourceFactory.class);
//缓存策略别名
typeAliasRegistry.registerAlias("PERPETUAL",PerpetualCache.class);
typeAliasRegistry.registerAlias("FIFO",FifoCache.class);
typeAliasRegistry.registerAlias("LRU",LruCache.class); typeAliasRegistry.registerAlias("SOFT", SoftCache.class); typeAliasRegistry.registerAlias("WEAK", WeakCache.class);
//数据库标识别名
typeAliasRegistry.registerAlias("DB_VENDOR",
VendorDatabaseIdProvider.class);
//语言驱动类别名
typeAliasRegistry.registerAlias("XML",XMLLanguageDriver.class);
typeAliasRegistry.registerAlias("RAW",RawLanguageDriver.class);
//日志类别名
typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
typeAliasRegistry.registerAlias("COMMONS_LOGGTNG",JakartmCommonsLogginglmpl.class);
typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
typeAliasRegistry.registerAlias("NO_LOGGING",NoLoggingImpl.class);
//动态代理别名
typeAliasRegistry.registerAlias("CGLIB",CglibProxyFactory.class);
typeAliasRegistry.registerAlias("JAVASSIST",JavassistProxyFactory.class);
```

这些配置为的是让我们更容易配置 MyBatis 的相关信息。以上就是 MyBatis 系统定义的别名，我们在使用的时候，不要重复命名，导致出现其他问题。

#### 2(自定义别名

由于现实中，特别是大型互联网系统中存在许多对象，比如用户（User）这个对象有时候需要大量重复地使用，因此 MyBatis 也提供了用户自定义别名的规则。我们可以通过 TypeAliasRegistry 类的 registerAlias 方法注册，也可以采用配置文件或者扫描方式来自定义它。

使用配置文件定义很简单：

```XML
<typeAliases><!--别名-->
  <typeAlias alias="role" type="com.mybatis.po.Role"/>
  <typeAlias alias="role" type="com.mybatis.po.User"/>
</typeAliases>
```

这样就可以定义一个别名了。如果有很多类需要定义别名，那么用这样的方式进行配置可就不那么轻松了。MyBatis 还支持扫描别名。比如上面的两个类都在包 com.mybatis.po 之下，那么就可以定义为：

```XML
<typeAliases><!--别名-->
  <package name="com.mybatis.po"/>
</typeAliases>
```

这样 MyBatis 将扫描这个包里面的类，将其第一个字母变为小写作为其别名，比如类 Role 的别名会变为 role，而 User 的别名会变为 user。使用这样的规则，有时候会出现重名。

比如 com.mybatis.po.User 这个类，MyBatis 还增加了对包 com.mybatis.po 的扫描，那么就会出现异常，这个时候可以使用 MyBatis 提供的注解 @Alias（"user3"）进行区分，如下所示。

```
package com.mybatis.po;
@Alias("user3")
public Class User {
    ......
}
```

这样就能够避免因为别名重名导致的扫描失败的问题。

---

## MyBatis TypeHandler类型转换器

在 JDBC 中，需要在 PreparedStatement 对象中设置那些已经预编译过的 SQL 语句的参数。执行 SQL 后，会通过 ResultSet 对象获取得到数据库的数据，而这些 MyBatis 是根据数据的类型通过 typeHandler 来实现的。

在 typeHandler 中，分为 jdbcType 和 javaType，其中 jdbcType 用于定义数据库类型，而 javaType 用于定义 [Java](http://c.biancheng.net/java/) 类型，那么 typeHandler 的作用就是承担 jdbcType 和 javaType 之间的相互转换。

如图 1 所示。在很多情况下我们并不需要去配置 typeHandler、jdbcType、javaType，因为 MyBatis 会探测应该使用什么类型的 typeHandler 进行处理，但是有些场景无法探测到。

对于那些需要使用自定义枚举的场景，或者数据库使用特殊数据类型的场景，可以使用自定义的 typeHandler 去处理类型之间的转换问题。

![typeHandler的作用](http://c.biancheng.net/uploads/allimg/190709/5-1ZFZ92210195.png)
图 1 typeHandler的作用


和别名一样，在 MyBatis 中存在系统定义 typeHandler 和自定义 typeHandler。MyBatis 会根据 javaType 和数据库的 jdbcType 来决定采用哪个 typeHandler 处理这些转换规则。系统提供的 typeHandler 能覆盖大部分场景的要求，但是有些情况下是不够的，比如我们有特殊的转换规则，枚举类就是这样。

---

## MyBatis自定义TypeHandler

在大部分的场景下，MyBatis 的 typeHandler 就能应付一般的场景，但是有时候不够用。比如使用枚举的时候，枚举有特殊的转化规则，这个时候需要自定义 typeHandler 进行处理它。

从[系统定义的 typeHandler](http://c.biancheng.net/view/4339.html) 可以知道，要实现 typeHandler 就需要去实现接口 typeHandler，或者继承 BaseTypeHandler（实际上，BaseTypeHandler 实现了 typeHandler 接口）。

这里我们仿造一个 StringTypeHandler 来实现一个自定义的 typeHandler——MyTypeHandler，它只是用于实现接口 typeHandler，如下所示。

```java
package com.mybatis.test;
import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.ResultSet;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.TypeHandler;
import org.apache.log4j.Logger;
public class MyTypeHandler implements TypeHandler<String> {
    Logger logger = Logger.getLogger(MyTypeHandler.class);
    @Override
    public void setParameter(PreparedStatement ps, int i, String parameter,
            JdbcType jdbcType) throws SQLException {
        logger.info("设置 string 参数【" + parameter + "】");
        ps.setString(i, parameter);
    }
    @Override
    public String getResult(ResultSet rs, String columnName)
            throws SQLException {
        String result = rs.getString(columnName);
        logger.info("读取 string 参数 1 【" + result + "】");
        return result;
    }
    @Override
    public String getResult(ResultSet rs, int columnIndex) throws SQLException {
        String result = rs.getString(columnIndex);
        logger.info("读取string 参数 2【" + result + "】");
        return result;
    }
    @Override
    public String getResult(CallableStatement cs, int columnIndex)
            throws SQLException {
        String result = cs.getString(columnIndex);
        logger.info("读取 string 参数 3 【" + result + "】");
        return result;
    }
}
```

定义的 typeHandler 泛型为 String，显然我们要把数据库的数据类型转化为 String 型，然后实现设置参数和获取结果集的方法。但是这个时候还没有启用 typeHandler，它还需要做如下所示的配置。

```xml
<typeHandlers>
  <typeHandler jdbcType="VARCHAR" javaType="string" handler="com.mybatis.test.MyTypeHandler"/>
</typeHandlers>
```



配置完成后系统才会读取它，这样注册后，当 jdbcType 和 javaType 能与 MyTypeHandler 对应的时候，它就会启动 MyTypeHandler。有时候还可以显式启用 typeHandler，一般而言启用这个 typeHandler 有两种方式，如下所示。

```xml
....
<resultMap id="roleMapper" type="role">
    <result property="id" column="id"/>
    <result property="roleName" column="role_name" jdbcType="VARCHAR" javaType="string"/>
    <result property="note" column="note" typeHandler=" com.mybatis.test.MyTypeHandler"/>
</resultMap>
<select id="getRole" parameterType="long" resultMap="roleMapper">
    select id,role_name,note from t_role where id = #{id}
</select>
<select id="findRoles" parameterType="string" resultMap="roleMapper">
    select id, role_name, note from t_role
    where role_name like concat('%',#{roleName, jdbcType=VARCHAR,
    javaType=string}, '%')
</select>
<select id="findRoles2" parameterType="string" resultMap="roleMapper">
    select id, role_name, note from t_role
    where note like concat ('%', # {note, typeHandler=com.mybatis.test.MyTypeHandler},'%')
</select>
......
```

注意，要么指定了与自定义 typeHandler 一致的 jdbcType 和 javaType，要么直接使用 typeHandler 指定具体的实现类。

在一些因为数据库返回为空导致无法断定采用哪个 typeHandler 来处理，而又没有注册对应的 javaType 的 typeHandler 时，MyBatis 无法知道使用哪个 typeHandler 转换数据，我们可以采用这样的方式来确定采用哪个 typeHandler 处理，这样就不会有异常出现了。

有时候由于枚举类型很多，系统需要的 typeHandler 也会很多，如果采用配置也会很麻烦，这个时候可以考虑使用包扫描的形式，那么就需要按照以下代码配置了。

```xml
<typeHandlertype>
  <package name="com.mybatis.test"/>
</typeHandlertype>
```



只是这样就没法指定 jdbcType 和 javaType 了，不过我们可以使用注解来处理它们。我们把 MyTypeHandler 的声明修改一下，如下所示。

```java
@MappedTypes(String.class)
@MappedjdbcTypes(jdbcType.VARCHAR)
public class MyTypeHandler implements TypeHandler<String>{
    ......
}
```

---

## MyBatis自定义TypeHandler处理枚举

在绝大多数情况下，typeHandler 因为枚举而使用，MyBatis 已经定义了两个类作为枚举类型的支持，这两个类分别是：

- EnumOrdinalTypeHandler。
- EnumTypeHandler。


因为它们的作用不大，所以在大部分情况下，我们都不用它们，不过我们还是要稍微了解一下它们的用法。在此之前，先来建一个性别枚举类——SexEnum，代码如下所示。

```JAVA
package com.mybatis.po;
public enum SexEnum {
    MALE(1, "男"),
    FEMALE(0, "女");
    private int id;
    private String name;
    /** stter and getter **/
    SexEnum(int id, String name) {
        this.id = id;
        this.name = name;
    }
    public SexEnum getSexById(int id) {
        for (SexEnum sex : SexEnum.values()) {
            if (sex.getId() == id) {
                return sex;
            }
        }
        return null;
    }
}
```

为了使用这个关于性别的枚举，可用以下 sql 语句创建 myUser 表。

```SQL
CREATE TABLE `myuser` (
  `id` bigint(20) NOT NULL,
  `user_name` varchar(20) DEFAULT NULL,
  `password` varchar(20) DEFAULT NULL,
  `sex` char(1) DEFAULT NULL,
  `mobile` varchar(20) DEFAULT NULL,
  `tel` varchar(20) DEFAULT NULL,
  `email` varchar(20) DEFAULT NULL,
  `note` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```


再创建一个用户 POJO，如下所示。

```java
public class User {
    private Long id;
    private String userName;
    private String password;
    private SexWnum sex;
    private String moblie;
    private String tel;
    private String email;
    private String note;
    /**setter and getter**/
}
```

### 1-EnumOrdinalTypeHandler

EnumOrdinalTypeHandler 是按 MyBatis 根据枚举数组下标索引的方式进行匹配的，也是枚举类型的默认转换类，它要求数据库返回一个整数作为其下标，它会根据下标找到对应的枚举类型。

根据这条规则，可以创建一个 UserMapper.xml 作为测试的例子，如下所示。

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mapper.UserMapper">
    <resultMap id="userMapper" type="user">
        <result property="id" column="id" />
        <result property="userName" column="user_name" />
        <result property="password" column="passsword" />
        <result property="sex" column="sex"
            typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler" />
        <result property="mobile" column="mobile" />
        <result property="tel" column="tel" />
        <result property="email" column="email" />
        <result property="note" column="note" />
    </resultMap>
    <select id="getUser" resultMap="userMapper" parameterType="long">
        select id,user_name,password,sex,mobile,tel,email,note from myUser
        where id=#{id}
    </select>
</mapper>
```

插入一条数据，执行的 SQL 如下：

``` XML
INSERT INTO `myuser` (`id`,`user_name`,`password`,`sex`,`mobile`,`tel`,`email`,`note`) VALUES(1,'zhangsan','123456','1','13675683675','0755-88888888','zhangsan@163.com','note......');
```

这样，sex 字段就在数据库里被设置为 1，代表女性，使用以下进行测试。

```java
package com.mybatis.test;
import java.io.IOException;
import java.io.InputStream;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.apache.log4j.Logger;
import com.mybatis.mapper.UserMapper;
import com.mybatis.po.User;
public class MyBatisTest {
    public static void main(String[] args) throws IOException {
        Logger log = Logger.getLogger(MyBatisTest.class);
        InputStream config = Resources
                .getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
        SqlSession ss = ssf.openSession();
        UserMapper userMapper = ss.getMapper(UserMapper.class);
        User user = userMapper.getUser(1L);
        log.info(user.getSex().getName());
    }
}
```

运行结果如图 1 所示：

![运行结果](http://c.biancheng.net/uploads/allimg/190709/5-1ZF9154535E0.png)
图 1 运行结果

### 2-EnumTypeHandler

EnumTypeHandler 会把使用的名称转化为对应的枚举，比如它会根据数据库返回的字符串“MALE”，进行 Enum.valueOf（SexEnum.class,"MALE"）；转换，所以为了测试 EnumTypeHandler 的转换，我们把数据库的 sex 字段修改为字符型（varchar（10）），并把 sex=1 的数据修改为 FEMALE，于是可以执行以下 SQL。

```SQL
ALTER TABLE myUser MODIFY sex VARCHAR(10);
UPDATE myUser SET sex='FEMALE' WHERE SEX = 1;
```

然后使用 EnumTypeHandler 修改 UserMaperr.xml，代码如下所示。

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mapper.UserMapper">
    <resultMap id="userMapper" type="user">
        <result property="id" column="id" />
        <result property="userName" column="user_name" />
        <result property="password" column="passsword" />
        <result property="sex" column="sex"
            typeHandler="org.apache.ibatis.type.EnumOrdinalTypeHandler" />
        <result property="mobile" column="mobile" />
        <result property="tel" column="tel" />
        <result property="email" column="email" />
        <result property="note" column="note" />
    </resultMap>
    <select id="getUser" resultMap="userMapper" parameterType="long">
        select id,user_name,password,sex,mobile,tel,email,note from myUser
        where id=#{id}
    </select>
</mapper>
```

执行以上代码，就可以可以看到正确运行的日志。

### 3-自定义枚举 typeHandler

我们已经讨论了 MyBatis 内部提供的两种转换的 typeHandler，但是它们有很大的局限性，更多的时候我们希望使用自定义的 typeHandler。执行下面的 SQL，把数据库的 sex 字段修改为整数型。

``` SQL
UPDATE myUser SET sex='0' WHERE sex = 'FEMALE';
UPDATE myUser SET sex='1' WHERE sex = 'MALE';
ALTER TABLE myUser MODIFY sex INT(10);
```

此时，按 SexEnum 的定义，sex=1 为男性，sex=0 为女性。为了满足这个规则，让我们自定义一个 SexEnumTypeHandler，如下所示。

```java
package com.mybatis.test;
import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedJdbcTypes;
import org.apache.ibatis.type.MappedTypes;
import org.apache.ibatis.type.TypeHandler;
import com.mybatis.po.SexEnum;
@MappedTypes(SexEnum.class)
@MappedJdbcTypes(JdbcType.INTEGER)
public class SexEnumTypeHandler implements TypeHandler<SexEnum> {
    @Override
    public void setParameter(PreparedStatement ps, int i, SexEnum parameter,
            JdbcType jdbcType) throws SQLException {
        ps.setInt(i, parameter.getId());
    }
    @Override
    public SexEnum getResult(ResultSet rs, String columnName)
            throws SQLException {
        int id = rs.getInt(columnName);
        return SexEnum.getSexById(id);
    }
    @Override
    public SexEnum getResult(ResultSet rs, int columnIndex) throws SQLException {
        int id = rs.getInt(columnIndex);
        return SexEnum.getSexById(id);
    }
    @Override
    public SexEnum getResult(CallableStatement rs, int columnIndex)
            throws SQLException {
        int id = rs.getInt(columnIndex);
        return SexEnum.getSexById(id);
    }
}
```

将 UserMapper.xml 的 typeHandler 换成自定义的 SexEnumTypeHandler，运行程序就可以得到我们想要的结果。

---

## MyBatis BlobTypeHandler读取Blob类型字段

MyBatis 对数据库的 Blob 字段也进行了支持，它提供了一个 BlobTypeHandler，为了应付更多的场景，它还提供了 ByteArrayTypeHandler，只是它不太常用，这里为读者展示 BlobTypeHandler 的使用方法。首先建一个表。

```
create table file(
    id int(12) not null auto_increment,
    content blob not null,
    primary key(id)
);
```

加粗的代码，使用了 Blob 字段，用于存入文件。然后创建一个 POJO，用于处理这个表，如下所示。

```JAVa
public class TestFile{
  long id;
  byte[] content;
  /** setter and getter **/
}
```

这里需要把 content 属性和数据库的 Blob 字段转换，这个时候可以使用系统注册的 typeHandler——BlobTypeHandler 来转换，如下所示。

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ssm.chapter5.mapper.FileMapper">
    <resultMap type="com.ssm.chapter5.pojo.TestFile" id="file">
        <id column="id" property="id"/>
        <id column="content" property="content" typeHandler="org.apache.ibatis.type.BlobTypeHandler"/>
    </resultMap>
    <select id="getFile" parameterType="long" resultMap="file"> 
        select id, content from t_file where id = #{id}
    </select>
    <insert id="insertFile" parameterType="com.ssm.chapter5.pojo.TestFile">
        insert into t_file(content) values(#{content})
    </insert>
</mapper>
```

实际上，不加入加粗代码的 typeHandler 属性，MyBatis 也能检测得到，并使用合适的 typeHandler 进行转换。

在现实中，一次性地将大量数据加载到 JVM 中，会给服务器带来很大压力，所以在更多的时候，应该考虑使用文件流的形式。这个时候只要把 POJO 的属性 content 修改为 InputStream 即可。

如果没有 typeHandler 声明，那么系统就会探测并使用 BlobInputStream TypeHandler 为你转换结果，这个时候需要把加粗代码的 typeHandler 修改为 org.apache.ibatis.type.BlobInputStreamTypeHandler。

因为性能不佳，文件的操作在大型互联网的网站上并不常用。更多的时候，大型互联网的网站会采用文件服务器的形式，通过更为高速的文件系统操作。这是搭建高效服务器需要注意的地方。

---

## MyBatis ObjectFactory（对象工厂）

当创建结果集时，MyBatis 会使用一个对象工厂来完成创建这个结果集实例。在默认的情况下，MyBatis 会使用其定义的对象工厂——DefaultObjectFactory（org.apache.ibatis.reflection.factory.DefaultObjectFactory）来完成对应的工作。

MyBatis 允许注册自定义的 ObjectFactory。如果自定义，则需要实现接口 org.apache.ibatis.reflection.factory.ObjectFactory，并给予配置。

在大部分的情况下，我们都不需要自定义返回规则，因为这些比较复杂而且容易出错，在更多的情况下，都会考虑继承系统已经实现好的 DefaultObjectFactory ，通过一定的改写来完成我们所需要的工作，如下所示。

```java
package com.mybatis.test;
import java.util.List;
import java.util.Properties;
import org.apache.ibatis.reflection.factory.DefaultObjectFactory;
import org.apache.log4j.Logger;
public class MyObjectFactory extends DefaultObjectFactory {
    private static final long serialVersionUID = -4293520460481008255L;
    Logger log = Logger.getLogger(MyObjectFactory.class);
    private Object temp = null;
    @Override
    public void setProperties(Properties properties) {
        super.setProperties(properties);
        log.info("初始化参数：【" + properties.toString() + "】");
    }
    // 方法2
    @Override
    public <T> T create(Class<T> type) {
        T result = super.create(type);
        log.info("创建对象：" + result.toString());
        log.info("是否和上次创建的是同一个对象：【" + (temp == result) + "】");
        return result;
    }
    // 方法1
    @Override
    public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes,
            List<Object> constructorArgs) {
        T result = super.create(type, constructorArgTypes, constructorArgs);
        log.info("创建对象：" + result.toString());
        temp = result;
        return result;
    }
    @Override
    public <T> boolean isCollection(Class<T> type) {
        return super.isCollection(type);
    }
}
```


然后对它进行配置，如下所示。

```xml
<objectFactory type="com.mybatis.test.MyObjectFactory">
  <property name="prop1" value="value1" />
</objectFactory>
```



这样 MyBatis 就会采用配置的 MyObjectFactory 来生成结果集对象，采用下面的代码进行测试。

```java
package com.mybatis.test;
import java.io.IOException;
import java.io.InputStream;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.apache.log4j.Logger;
import com.mybatis.mapper.UserMapper;
import com.mybatis.po.User;
public class MyBatisTest {
    public static void main(String[] args) throws IOException {
        Logger log = Logger.getLogger(MyBatisTest.class);
        InputStream config = Resources
                .getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
        SqlSession ss = ssf.openSession();
        UserMapper userMapper = ss.getMapper(UserMapper.class);
        User user = userMapper.getUser(1L);
        System.out.println(user.getUserName());
    }
}
```

当配置了 log4j.properties 文件的时候，就能看到下图中的输出日志。



![运行结果](http://c.biancheng.net/uploads/allimg/190709/5-1ZF91I649621.png)


如果打断点调试一步步跟进，那么你会发现 MyBatis 创建了一个 List 对象和一个 User 对象。它会先调用方法 1，然后调用方法 2，只是最后生成了同一个对象，所以在写入的判断中，始终返回的是 true。因为返回的是一个 User 对象，所以它会最后适配为一个 User 对象，这就是它的工作过程。



## MyBatis配置文件environments和子元素transactionManager、dataSource解析

在 MyBatis 中，运行环境主要的作用是配置数据库信息，它可以配置多个数据库，一般而言只需要配置其中的一个就可以了。

它下面又分为两个可配置的元素：事务管理器（transactionManager）、数据源（dataSource）。

在实际的工作中，大部分情况下会采用 [Spring](http://c.biancheng.net/spring/) 对数据源和数据库的事务进行管理，这些我们教程后面都会进行讲解。本节我们会探讨 MyBatis 自身实现的类。

运行环境配置，代码如下所示。

```JAVA
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC" />
        <dataSource type="POOLED">
            <property name="driver" value="${database.driver}" />
            <property name="url"
                value="${database.url}" />
            <property name="username" value="${database.username}" />
            <property name="password" value="${database.password}" />
        </dataSource>
    </environment>
</environments>
```

这里用到两个元素：transactionManager 和 environment。

---

### transactionManager（事务管理器）

在 MyBatis 中，transactionManager 提供了两个实现类，它需要实现接口 Transaction（org.apache.ibatis.transaction.Transaction），它的定义代码如下所示。

```java
public interface Transaction {
    Connection getConnection() throws SQLException;
    void commit() throws SQLException;
    void rollback() throws SQLException;
    void close() throws SQLException;
    Integer getTimeout() throws SQLException;
}
```

从方法可知，它主要的工作就是提交（commit）、回滚（rollback）和关闭（close）数据库的事务。MyBatis 为 Transaction 提供了两个实现类：JdbcTransaction 和 ManagedTransaction，如图 1 所示。

![Transaction的实现类](http://c.biancheng.net/uploads/allimg/190710/5-1ZG010033A61.png)
图 1 Transaction的实现类


于是它对应着两种工厂：JdbcTransactionFactory 和 ManagedTransactionFactory，这个工厂需要实现 TransactionFactory 接口，通过它们会生成对应的 Transaction 对象。于是可以把事务管理器配置成为以下两种方式：

```xml
<transactionManager type="JDBC"/>
<transactionManager type="MANAGED"/>
```



这里做简要的说明。

JDBC 使用 JdbcTransactionFactory 生成的 JdbcTransaction 对象实现。它是以 JDBC 的方式对数据库的提交和回滚进行操作。

MANAGED 使用 ManagedTransactionFactory 生成的 ManagedTransaction 对象实现。它的提交和回滚方法不用任何操作，而是把事务交给容器处理。在默认情况下，它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。

不想采用 MyBatis 的规则时，我们可以这样配置：

```xml
<transactionManager type="com.mybatis.transaction.MyTransactionFactory"/>
```

实现一个自定义事务工厂，代码如下所示。

```java
public class MyTransactionFactory implements TransactionFactory {
    @Override
    public void setProperties(Properties props) {
    }
    @Override
    public Transaction newTransaction(Connection conn) {
        return new MyTransaction(conn);
    }
    @Override
    public Transaction newTransaction(DataSource dataSource, TransactionlsolationLevel level, boolean autoCommit) {
        return new MyTransaction(dataSource, level, autoCommit);
    }
}
```

这里就实现了 TransactionFactory 所定义的工厂方法，这个时候还需要事务实现类 MyTransaction，它用于实现 Transaction 接口，代码如下所示。

```java
public class MyTransaction extends JdbcTransaction implements Transaction {
    public MyTransaction(DataSource ds, TransactionIsolationLevel desiredLevel,
            boolean desiredAutoCommit) {
        super(ds, desiredLevel, desiredAutoCommit);
    }
    public MyTransaction(Connection connection) {
        super(connection);
    }
    public Connection getConnection() throws SQLException {
        return super.getConnection();
    }
    public void commit() throws SQLException {
        super.commit();
    }
    public void rollback() throws SQLException {
        super.rollback();
    }
    public void close() throws SQLException {
        super.close();
    }
    public Integer getTimeout() throws SQLException {
        return super.getTimeout();
    }
}
```

这样就能够通过自定义事务规则，满足特殊的需要了。

### environment 数据源环境

environment 的主要作用是配置数据库，在 MyBatis 中，数据库通过 PooledDataSource Factory、UnpooledDataSourceFactory 和 JndiDataSourceFactory 三个工厂类来提供，前两者对应产生 PooledDataSource、UnpooledDataSource 类对象，而 JndiDataSourceFactory 则会根据 JNDI 的信息拿到外部容器实现的数据库连接对象。

无论如何这三个工厂类，最后生成的产品都会是一个实现了 DataSource 接口的数据库连接对象。

由于存在三种数据源，所以可以按照下面的形式配置它们。

```xml
<dataSource type="UNPOOLED">
<dataSource type="POOLED">
<dataSource type="JNDI">
```

论述一下这三种数据源及其属性。

#### 1. UNPOOLED

UNPOOLED 采用非数据库池的管理方式，每次请求都会打开一个新的数据库连接，所以创建会比较慢。在一些对性能没有很高要求的场合可以使用它。

对有些数据库而言，使用连接池并不重要，那么它也是一个比较理想的选择。UNPOOLED 类型的数据源可以配置以下几种属性：

- driver 数据库驱动名，比如 [MySQL](http://c.biancheng.net/mysql/) 的 com.mysql.jdbc.Driver。
- url 连接数据库的 URL。
- username 用户名。
- password 密码。
- defaultTransactionIsolationLevel 默认的连接事务隔离级别，关于隔离级别，后面教程中会讨论。


传递属性给数据库驱动也是一个可选项，注意属性的前缀为“driver.”，例如 driver.encoding=UTF8。它会通过 DriverManager.getConnection（url,driverProperties）方法传递值为 UTF8 的 encoding 属性给数据库驱动。

#### 2. POOLED

数据源 POOLED 利用“池”的概念将 JDBC 的 Connection 对象组织起来，它开始会有一些空置，并且已经连接好的数据库连接，所以请求时，无须再建立和验证，省去了创建新的连接实例时所必需的初始化和认证时间。它还控制最大连接数，避免过多的连接导致系统瓶颈。

除了 UNPOOLED 下的属性外，会有更多属性用来配置 POOLED 的数据源，如表 1 所示：



| 名称                          | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| poolMaximumActiveConnections  | 是在任意时间都存在的活动（也就是正在使用）连接数量，默认值为 10 |
| poolMaximumIdleConnections    | 是任意时间可能存在的空闲连接数                               |
| poolMaximumCheckoutTime       | 在被强制返回之前，池中连接被检出（checked out）的时间，默认值为 20 000 毫秒（即 20 秒） |
| poolTimeToWait                | 是一个底层设置，如果获取连接花费相当长的时间，它会给连接池打印状态日志，并重新尝试获取一个连接（避免在误配置的情况下一直失败），默认值为 20 000 毫秒（即 20 秒）。 |
| poolPingQuery                 | 为发送到数据库的侦测查询，用来检验连接是否处在正常工作秩序中，并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动失败时带有一个恰当的错误消息。 |
| poolPingEnabled               | 为是否启用侦测查询。若开启，也必须使用一个可执行的 SQL 语句设置 poolPingQuery 属性（最好是一个非常快的 SQL），默认值为 false。 |
| poolPingConnectionsNotUsedFor | 为配置 poolPingQuery 的使用频度。这可以被设置成匹配具体的数据库连接超时时间，来避免不必要的侦测，默认值为 0（即所有连接每一时刻都被侦测——仅当 poolPingEnabled 为 true 时适用）。 |

#### 3. JNDI

数据源 JNDI 的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。这种数据源配置只需要两个属性：

#### 1）initial_context

用来在 InitialContext 中寻找上下文（即，initialContext.lookup（initial_context））。initial_context 是个可选属性，如果忽略，那么 data_source 属性将会直接从 InitialContext 中寻找。

#### 2）data_source

是引用数据源实例位置上下文的路径。当提供 initial_context 配置时，data_source 会在其返回的上下文中进行查找；当没有提供 initial_context 时，data_source 直接在 InitialContext 中查找。

与其他数据源配置类似，它可以通过添加前缀“env.”直接把属性传递给初始上下文（InitialContext）。比如 env.encoding=UTF8，就会在初始上下文实例化时往它的构造方法传递值为 UTF8 的 encoding 属性。

MyBatis 也支持第三方数据源，例如使用 DBCP 数据源，那么需要提供一个自定义的 DataSourceFactory，代码如下所示。

```
public class DbcpDataSourceFactory implements DataSourceFactory {    private Properties props = null;    public void setProperties(Properties props) {        this.props = props;    }    public DataSource getDataSource() {        DataSource dataSource = null;        dataSource = BasicDataSourceFactory.createDataSource(props);        return dataSource;    }}
```

然后进行如下配置：

```
<dataSource type="com.mybatis.dataSource.DbcpDataSourceFactory">    <property name="driver" value="${database.driver}" />    <property name="url" value="${database.url}" />    <property name="username" value="${database.username}" />    <property name="password" value="${database.password}" /></dataSource>
```

这样 MyBatis 就会采用配置的数据源工厂来生成数据源了。

---



## MyBatis与Spring的整合步骤

从之前的代码中可以看出直接使用 MyBatis 框架的 SqlSession 访问数据库并不简便。MyBatis 框架的重点是 SQL 映射文件，为方便后续学习，本节讲解 MyBatis 与 [Spring](http://c.biancheng.net/spring/) 的整合。教程的后续讲解中将使用整合后的框架进行演示。

### 1.导入相关JAR包

实现 MyBatis 与 Spring 的整合需要导入相关 JAR 包，包括 MyBatis、Spring 以及其他 JAR 包。

#### 1）MyBatis 框架所需的 JAR 包

MyBatis 框架所需的 JAR 包包括它的核心包和依赖包，包的详情可参考《第一个Mybatis程序》。

#### 2）Spring 框架所需的 JAR 包

Spring 框架所需的 JAR 包包括它的核心模块 JAR、AOP 开发使用的 JAR、JDBC 和事务的 JAR 包（其中依赖包不需要再导入，因为 MyBatis 已提供），具体如下：

- aopalliance-1.0.jar
- aspectjweaver-1.6.9.jar
- spring-aop-3.2.13.RELEASE.jar
- spring-aspects-3.2.13.RELEASE.jar
- spring-beans-3.2.13.RELEASE.jar
- spring-context-3.2.13.RELEASE.jar
- spring-core-3.2.13.RELEASE.jar
- spring-expression-3.2.13.RELEASE.jar
- spring-jdbc-3.2.13.RELEASE.jar
- spring-tx-3.2.13.RELEASE.jar

#### 3）MyBatis 与 Spring 整合的中间 JAR 包

该中间 JAR 包的版本为 mybatis-spring-1.3.1.jar，此版本可以从网址“http://mvnrepository.com/artifact/org.mybatis/mybatis-spring/1.3.1”下载。

#### 4）数据库驱动 JAR 包

教程所使用的 [MySQL](http://c.biancheng.net/mysql/) 数据库驱动包为 mysql-connector-java-5.1.25-bin.jar。

#### 5）数据源所需的 JAR 包

在整合时使用的是 DBCP 数据源，需要准备 DBCP 和连接池的 JAR 包。

本教程所用版本的 DBCP 的 JAR 包为 commons-dbcp2-2.2.0.jar，可以从网址“[htttp://commons.apache.org/proper/commons-dbcp/download_dbcp.cgi](http://htttp//commons.apache.org/proper/commons-dbcp/download_dbcp.cgi)”下载。

最新版本的连接池的 JAR 包为 commons-pool2-2.5.0.jar，可以从网址“http://commons.apache.org/proper/commons-pool/download_pool.cgi”下载。

### 2.在Spring中配置MyBatis工厂

通过与 Spring 的整合，MyBatis 的 SessionFactory 交由 Spring 来构建，在构建时需要在 Spring 的配置文件中添加如下代码：

```xml
<!--配置数据源-->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://127.0.0.1:3306/springtest?seUnicode=true&amp;characterEncoding=utf-8" />
    <property name="username" value="root" />
    <property name="password" value="1128" />
    <!-- 最大连接数 -->
    <property name="maxTotal" value="30"/>
    <!-- 最大空闲连接数 -->
    <property name="maxIdle" value="10"/>
    <!-- 初始化连接数 -->
    <property name="initialSize" value="5"/>
</bean>
<!-- 配置SqlSessionFactoryBean -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 引用数据源组件 -->
    <property name="dataSource" ref="dataSource" />
    <!-- 引用MyBatis配置文件中的配置 -->
    <property name="configLocation" value="classpath:mybatis-config.xml" />
</bean>
```

### 3.使用 Spring 管理 MyBatis 的数据操作接口

使用 Spring 管理 MyBatis 数据操作接口的方式有多种，其中最常用、最简洁的一种是基于 MapperScannerConfigurer 的整合。该方式需要在 Spring 的配置文件中加入以下内容：

```xml
<!-- Mapper代理开发，使用Spring自动扫描MyBatis的接口并装配 （Sprinh将指定包中的所有被@Mapper注解标注的接口自动装配为MyBatis的映射接口） -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- mybatis-spring组件的扫描器，com.dao只需要接口（接口方法与SQL映射文件中的相同） -->
    <property name="basePackage" value="com.dao" />
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
</bean>
```

---

## MyBatis与Spring的整合实例详解



下面通过一个实例实现 MyBatis 与 [Spring](http://c.biancheng.net/spring/) 的整合，具体实现过程如下：

#### 1）创建应用并导入相关 JAR 包

创建一个名为 MyBatis-Spring 的 Web 应用，并将《[MyBatis与Spring的整合步骤](http://c.biancheng.net/view/4311.html)》教程的 JAR 导入 /WEB-INF/lib 目录下。

#### 2）创建持久化类

在 src 目录下创建一个名为 com.po 的包，将《[第一个MyBatis程序](http://c.biancheng.net/view/4309.html)》教程的持久化类复制到包中。

#### 3）创建 SQL 映射文件和 MyBatis 核心配置文件

在 src 目录下创建一个名为 com.mybatis 的包，在该包中创建 MyBatis 核心配置文件 mybatis-config.xml 和 SQL 映射文件 UserMapper.xml。

UserMapper.xml 的代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dao.UserDao">
    <!--根据uid查询一个用户信息 -->
    <select id="selectUserById" parameterType="Integer" resultType="com..po.MyUser">
        select * from user where uid = #{uid}
    </select>
    <!-- 查询所有用户信息 -->
    <select id="selectAllUser" resultType="com.po.MyUser">
        select * from user
    </select>
    <!-- 添加一个用户，#{uname}为 com.mybatis.po.MyUser 的属性值 -->
    <insert id="addUser" parameterType="com.po.MyUser">
    insert into user (uname,usex) values(#{uname},#{usex})
    </insert>
    <!--修改一个用户 -->
    <update id="updateUser" parameterType="com..po.MyUser">
        update user set uname = #{uname},usex = #{usex} where uid = #{uid}
    </update>
    <!-- 删除一个用户 -->
    <delete id="deleteUser" parameterType="Integer">
        delete from user where uid = #{uid}
    </delete>
</mapper>

```

mybatis-config.xml 的代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 告诉MyBatis到哪里去找映射文件 -->
    <mappers>
        <mapper resource="com/mybatis/UserMapper.xml" />
    </mappers>
</configuration>
```

#### 4）创建数据访问接口

在 src 目录下创建一个名为 com.dao 的包，在该包中创建 UserDao 接口，并将接口使用 @Mapper 注解为 Mapper，接口中的方法与 SQL 映射文件一致。

UserDao 接口的代码如下：

```java
package com.dao;
import java.util.List;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;
import com.po.MyUser;
@Repository("userDao")
@Mapper
/*
* 使用Spring自动扫描MyBatis的接口并装配 （Spring将指定包中所有被@Mapper注解标注的接口自动装配为MyBatis的映射接口
*/
public interface UserDao {
    /**
     * 接口方法对应的SQL映射文件UserMapper.xml中的id
     */
    public MyUser selectUserById(Integer uid);
    public List<MyUser> selectAllUser();
    public int addUser(MyUser user);
    public int updateUser(MyUser user);
    public int deleteUser(Integer uid);
}
```

#### 5）创建日志文件

在 src 目录下创建日志文件 log4j.properties，文件内容如下：

``` properties
\# Global logging configuration
log4j.rootLogger=ERROR,stdout
\# MyBatis logging configuration...
log4j.logger.com.mybatis=DEBUG
\# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```



#### 6）创建控制层

在 src 目录下创建一个名为 com.controller 的包，在包中创建 UserController 类，在该类中调用数据访问接口中的方法。

UserController 类的代码如下：

```JAVA
package com.controller;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import com.dao.UserDao;
import com.po.MyUser;
@Controller("userController")
public class UserController {
    @Autowired
    private UserDao userDao;
    public void test() {
        // 查询一个用户
        MyUser auser = userDao.selectUserById(1);
        System.out.println(auser);
        System.out.println("============================");
        // 添加一个用户
        MyUser addmu = new MyUser();
        addmu.setUname("陈恒");
        addmu.setUsex("男");
        int add = userDao.addUser(addmu);
        System.out.println("添加了" + add + "条记录");
        System.out.println("============================");
        // 修改一个用户
        MyUser updatemu = new MyUser();
        updatemu.setUid(1);
        updatemu.setUname("张三");
        updatemu.setUsex("女");
        int up = userDao.updateUser(updatemu);
        System.out.println("修改了" + up + "条记录");
        System.out.println("============================");
        // 删除一个用户
        int dl = userDao.deleteUser(9);
        System.out.println("删除了" + dl + "条记录");
        System.out.println("============================");
        // 查询所有用户
        List<MyUser> list = userDao.selectAllUser();
        for (MyUser myUser : list) {
            System.out.println(myUser);
        }
    }
}
```

#### 7）创建 Spring 的配置文件

在 src 目录下创建配置文件 applicationContext.xml，在配置文件中配置数据源、MyBatis 工厂以及 Mapper 代理开发等信息。

applicationContext.xml 的代码如下：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
    <!-- 指定需要扫描的包（包括子包），使注解生效 -->
    <context:component-scan base-package="com.dao" />
    <context:component-scan base-package="com.controller" />
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url"
            value="jdbc:mysql://127.0.0.1:3306/smbms?
                        useUnicode=true&amp;characterEncoding=utf-8" />
        <property name="username" value="root" />
        <property name="password" value="1128" />
        <!-- 最大连接数 -->
        <property name="maxTotal" value="30" />
        <!-- 最大空闲连接数 -->
        <property name="maxIdle" value="10" />
        <!-- 初始化连接数 -->
        <property name="initialSize" value="5" />
    </bean>
    <!-- 添加事务支持 -->
    <bean id="txManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <!-- 注册事务管理驱动 -->
    <tx:annotation-driven transaction-manager="txManager" />
    <!-- 配置SqlSessionFactoryBean -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 引用数据源组件 -->
        <property name="dataSource" ref="dataSource" />
        <!-- 引用MyBatis配置文件中的配置 -->
        <property name="configLocation" value="classpath:com/mybatis/mybatis-config.xml" />
    </bean>
    <!-- Mapper代理开发，使用Spring自动扫描MyBatis的接口并装配 （Sprinh将指定包中的所有被@Mapper注解标注的接口自动装配为MyBatis的映射接口） -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- mybatis-spring组件的扫描器，com.dao只需要接口（接口方法与SQL映射文件中的相同） -->
        <property name="basePackage" value="com.dao" />
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
    </bean>
</beans>
```

#### 8）创建测试类

在 com.controller 包中创建测试类 TestController，代码如下：

```javA
package com.controller;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class TestController {
    public static void main(String[] args) {
        String xmlPath = "applicationContext.xml";
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
                xmlPath);
        UserController uc = (UserController) applicationContext
                .getBean("userController");
        uc.test();
    }
}
```

上述测试类的运行结果如图 1 所示。



![框架整合测试结果](http://c.biancheng.net/uploads/allimg/190710/5-1ZG01G45c45.png)
图 1 框架整合测试结果


从第 6 步中的 UserController 类可以看出，开发者只需要进行业务处理，不需要再写 SqlSession 对象的创建、数据库事务的处理等烦琐代码。因此，MyBatis 整合 Spring 后方便了数据库访问操作，提高了开发效率。

----



## Mybatis select标签以及使用Map接口和Java Bean传递多个参数

在 SQL 映射文件中 <select> 元素用于映射 SQL 的 select 语句，其示例代码如下：

<!--根据uid查询一个用户信息 -->

```XML
<select id="selectUserById" parameterType="Integer" resultType="com.mybatis.po.MyUser">
  select * from user where uid = #{uid}
</select>
```

在上述示例代码中，id 的值是唯一标识符，它接收一个 Integer 类型的参数，返回一个 MyUser 类型的对象，结果集自动映射到 MyUser 属性。

<select> 元素除了有上述示例代码中的几个属性以外，还有一些常用的属性，如表 1 所示。



| 属性名称      | 描 述                                                        |
| ------------- | ------------------------------------------------------------ |
| id            | 它和 Mapper 的命名空间组合起来使用，是唯一标识符，供 MyBatis 调用 |
| parameterType | 表示传入 SQL 语句的参数类型的全限定名或别名。它是一个可选属性，MyBatis 能推断出具体传入语句的参数 |
| resultType    | SQL 语句执行后返回的类型（全限定名或者别名）。如果是集合类型，返回的是集合元素的类型，返回时可以使用 resultType 或 resultMap 之一 |
| resultMap     | 它是映射集的引用，与 <resultMap> 元素一起使用，返回时可以使用 resultType 或 resultMap 之一 |
| flushCache    | 用于设置在调用 SQL 语句后是否要求 MyBatis 清空之前查询的本地缓存和二级缓存，默认值为 false，如果设置为 true，则任何时候只要 SQL 语句被调用都将清空本地缓存和二级缓存 |
| useCache      | 启动二级缓存的开关，默认值为 true，表示将査询结果存入二级缓存中 |
| timeout       | 用于设置超时参数，单位是秒（s），超时将抛出异常              |
| fetchSize     | 获取记录的总条数设定                                         |
| statementType | 告诉 MyBatis 使用哪个 JDBC 的 Statement 工作，取值为 STATEMENT（Statement）、 PREPARED（PreparedStatement）、CALLABLE（CallableStatement） |
| resultSetType | 这是针对 JDBC 的 ResultSet 接口而言，其值可设置为 FORWARD_ONLY（只允许向前访问）、SCROLL_SENSITIVE（双向滚动，但不及时更新）、SCROLLJNSENSITIVE（双向滚动，及时更新） |

### 1.使用 Map 接口传递多个参数

在实际开发中，查询 SQL 语句经常需要多个参数，例如多条件查询。当传递多个参数时，<select> 元素的 parameterType 属性值的类型是什么呢？在 MyBatis 中允许 Map 接口通过键值对传递多个参数。

假设数据操作接口中有个实现查询陈姓男性用户信息功能的方法：

```JAVA
public List<MyUser> selectAllUser(Map<String,Object> param);
```

此时，传递给映射器的是一个 Map 对象，使用它在 SQL 文件中设置对应的参数，对应 SQL 文件的代码如下：

```XML
<!-- 查询陈姓男性用户信息 -->
<select id="selectAllUser" resultType="com.mybatis.po.MyUser">
    select * from user
    where uname like concat('%',#{u_name},'%')
    and usex = #{u_sex}
</select>
```

在上述 SQL 文件中，参数名 u_name 和 u_sex 是 Map 的 key。

为了测试该示例，首先创建一个 Web 应用 mybatisDemo02，将 mybatisDemo01 应用的所有 JAR 包复制到 /WEB-INF/lib 下，同时将 mybatisDemo01 应用的 src 目录下的所有包和文件复制到 mybatisDemo02 应用的 src 目录下。

然后将 com.mybatis 包中的 SQL 映射文件 UserMapper.xml 中的“查询所有用户信息”的代码片段修改为上述“查询陈姓男性用户信息”的代码片段，最后将 com.controller 包中 UserController 的代码简单修改即可运行测试类了。

com.controller 包中 UserController 的代码片段如下：

```java
@Controller("UserController")
public class UserController {
    private UserDao userDao;
    public void test(){
        ...
        //查询多个用户
        Map<String,Object> map = new HashMap<>();
        map.put("u_name","陈");
        map.put("u_sex","男");
        List<MyUser> list = userDao.seleceAllUser(map);
        for(MyUser myUser : list) {
            System.out.println(myUser);
        }
        ...
    }
}
```

Map 是一个键值对应的集合，使用者要通过阅读它的键才能了解其作用。另外，使用 Map 不能限定其传递的数据类型，所以业务性不强，可读性较差。如果 SQL 语句很复杂，参数很多，使用 Map 将很不方便。幸运的是，MyBatis 还提供了使用 [Java](http://c.biancheng.net/java/) Bean 传递多个参数的形式。

### 2.使用 Java Bean 传递多个参数

首先在 myBatisSemo02 应用的 src 目录下创建一个名为 com.pojo 的包，在包中创建一个 POJO 类 SeletUserParam，代码如下：

```JAVA
package com.pojo;
public class SeletUserParam {
    private String u_name;
    private String u_sex;
    // 此处省略setter和getter方法
}
```

接着将 Dao 接口中的 selectAllUser 方法修改为如下：

```JAVA
public List<MyUser> selectAllUser(SelectUserParam param);
```

然后将 com.mybatis 包中的 SQL 映射文件 UserMapper.xml 中的“查询陈姓男性用户信息”的代码修改为如下：

```XML
<select id="selectAllUser" resultType="com.po.MyUser" parameterType="com.pojo.SeletUserParam">
    select * from user
    where uname like concat('%',#{u_name},'%')
    and usex=#{u_sex}
</select>
```

最后将 com.controller 包中 UserController 的“查询多个用户”的代码片段做如下修改：

```JAVA
SeletUserParam su = new SelectUserParam();
su.setU_name("陈");
su.setU_sex("男");
List<MyUser> list = userDao.selectAllUser(su);
for (MyUser myUser : list) {
    System.out.println(myUser);
}
```

在实际应用中是选择 Map 还是选择 Java Bean 传递多个参数应根据实际情况而定，如果参数较少，建议选择 Map；如果参数较多，建议选择 Java Bean。

----





## MyBatis中的insert、update、delete和sql标签

这节我们来讲 MyBatis 中的 <insert>、<update>、<delete> 和 <sql> 元素。

### \<insert>元素

<insert> 元素用于映射插入语句，MyBatis 执行完一条插入语句后将返回一个整数表示其影响的行数。它的属性与 <select> 元素的属性大部分相同，在本节讲解它的几个特有属性。

- keyProperty：该属性的作用是将插入或更新操作时的返回值赋给 PO 类的某个属性，通常会设置为主键对应的属性。如果是联合主键，可以将多个值用逗号隔开。
- keyColumn：该属性用于设置第几列是主键，当主键列不是表中的第 1 列时需要设置。如果是联合主键，可以将多个值用逗号隔开。
- useGeneratedKeys：该属性将使 MyBatis 使用 JDBC 的 getGeneratedKeys（）方法获取由数据库内部产生的主键，例如 [MySQL](http://c.biancheng.net/mysql/)、SQL Server 等自动递增的字段，其默认值为 false。

#### 1）主键（自动递增）回填

MySQL、SQL Server 等数据库的表格可以采用自动递增的字段作为主键，有时可能需要使用这个刚刚产生的主键，用于关联其他业务。

首先为 com.mybatis 包中的 SQL 映射文件 UserMapper.xml 中 id 为 addUser 的 <insert> 元素添加 keyProperty 和 useGeneratedKeys 属性，具体代码如下：

```xml
<!--添加一个用户，成功后将主键值返回填给uid(po的属性)-->
<insert id="addUser" parameterType="com.po.MyUser" keyProperty="uid" useGeneratedKeys="true">
  insert into user (uname,usex) values(#{uname},#{usex})
</insert>
```

然后在 com.controller 包的 UserController 类中进行调用，具体代码如下：

```java
// 添加一个用户
MyUser addmu = new MyUser();
addmu.setUname("陈恒");
addmu.setUsex("男");
int add = userDao.addUser(addmu);
System.out.println("添加了" + add + "条记录");
System.out.println("添加记录的主键是" + addmu.getUid());
```

#### 2）自定义主键

如果在实际工程中使用的数据库不支持主键自动递增（例如 Oracle），或者取消了主键自动递增的规则，可以使用 MyBatis 的 <selectKey> 元素来自定义生成主键。具体配置示例代码如下：

```xml
<!-- 添加一个用户，#{uname}为 com.mybatis.po.MyUser 的属性值 -->
<insert id="insertUser" parameterType="com.po.MyUser">
    <!-- 先使用selectKey元素定义主键，然后再定义SQL语句 -->
    <selectKey keyProperty="uid" resultType="Integer" order="BEFORE">
    select if(max(uid) is null,1,max(uid)+1) as newUid from user)
    </selectKey>
    insert into user (uid,uname,usex) values(#{uid},#{uname},#{usex})
</insert>
```

在执行上述示例代码时，<selectKey> 元素首先被执行，该元素通过自定义的语句设置数据表的主键，然后执行插入语句。

<selectKey> 元素的 keyProperty 属性指定了新生主键值返回给 PO 类（com.po.MyUser）的哪个属性。

- order 属性可以设置为 BEFORE 或 AFTER。
- BEFORE 表示先执行 <selectKey> 元素然后执行插入语句。
- AFTER 表示先执行插入语句再执行 <selectKey> 元素。

### \<update>与\<delete>元素

<update> 和 <delete> 元素比较简单，它们的属性和 <insert> 元素、<select> 元素的属性差不多，执行后也返回一个整数，表示影响了数据库的记录行数。配置示例代码如下：

```xml
<!-- 修改一个用户 -->
<update id="updateUser" parameterType="com.po.MyUser">
    update user set uname = #{uname},usex = #{usex} where uid = #{uid}
</update>
<!-- 删除一个用户 -->
<delete id="deleteUser" parameterType="Integer">
    delete from user where uid = #{uid}
</delete
```

### \<sql> 元素

<sql> 元素的作用在于可以定义 SQL 语句的一部分（代码片段），以方便后面的 SQL 语句引用它，例如反复使用的列名。

在 MyBatis 中只需使用 <sql> 元素编写一次便能在其他元素中引用它。配置示例代码如下：

```xml
<sql id="comColumns">id,uname,usex</sql>
<select id="selectUser" resultType="com.po.MyUser">
    select <include refid="comColumns"> from user
</select>
```

在上述代码中使用 <include> 元素的 refid 属性引用了自定义的代码片段。



---



## MyBatis resultMap元素的结构及使用

<resultMap> 元素表示结果映射集，是 MyBatis 中最重要也是最强大的元素，主要用来定义映射规则、级联的更新以及定义类型转化器等。

### \<resultMap> 元素的结构

\<resultMap> 元素包含了一些子元素，结构如下：

```
<resultMap id="" type="">
    <constructor><!-- 类再实例化时用来注入结果到构造方法 -->
        <idArg/><!-- ID参数，结果为ID -->
        <arg/><!-- 注入到构造方法的一个普通结果 -->  
    </constructor>
    <id/><!-- 用于表示哪个列是主键 -->
    <result/><!-- 注入到字段或JavaBean属性的普通结果 -->
    <association property=""/><!-- 用于一对一关联 -->
    <collection property=""/><!-- 用于一对多、多对多关联 -->
    <discriminator javaType=""><!-- 使用结果值来决定使用哪个结果映射 -->
        <case value=""/><!-- 基于某些值的结果映射 -->
    </discriminator>
</resultMap>
```

- <resultMap> 元素的 type 属性表示需要的 POJO，id 属性是 resultMap 的唯一标识。
- 子元素 <constructor> 用于配置构造方法（当 POJO 未定义无参数的构造方法时使用）。
- 子元素 <id> 用于表示哪个列是主键。
- 子元素 <result> 用于表示POJO和数据表普通列的映射关系。
- 子元素 <association>、<collection> 和 <discriminator> 用在级联的情况下。关于级联的问题比较复杂，后面教程会详细讲解。


一条查询 SQL 语句执行后将返回结果，而结果可以使用 Map 存储，也可以使用 POJO 存储。

### 使用 Map 存储结果集

任何 select 语句都可以使用 Map 存储结果，示例代码如下：

```
<!-- 查询所有用户信息存到Map中 -->
<select id="selectAllUserMap" resultType="map">
    select * from user
</select>
```

测试上述 SQL 配置文件的过程如下：

首先在 com.dao.UserDao 接口中添加以下接口方法。

public List<Map<String,Object>> selectAllUserMap();

然后在 com.controller 包的 UserController 类中调用接口方法，具体代码如下。

```
// 查询所有用户信息存到Map中
List<Map<String, Object>> lmp = userDao.selectAllUserMap();
for (Map<String, Object> map : lmp) {
    System.out.println(map);
}
```

上述 Map 的 key 是 select 语句查询的字段名（必须完全一样），而 Map 的 value 是查询返回结果中字段对应的值，一条记录映射到一个 Map 对象中。Map 用起来很方便，但可读性稍差，有的开发者不太喜欢使用 Map，更多时候喜欢使用 POJO 的方式。

### 使用POJO存储结果集

有的开发者喜欢使用 POJO 的方式存储结果集，一方面可以使用自动映射，例如使用 resultType 属性，但有时候需要更为复杂的映射或级联，这时候就需要使用 <select> 元素的 resultMap 属性配置映射集合。具体步骤如下：

#### 1）创建 POJO 类

在 myBatisDemo02 应用的 com.pojo 包中创建 POJO 类 MapUser。MapUser 类的代码如下：

```
package com.pojo;public class MapUser {    private Integer m_uid;    private String m_uname;    private String m_usex;    // 此处省略setter和getter方法    @Override    public String toString() {        return "User[uid=" + m_uid + ",uname=" + m_uname + ",usex=" + m_usex                + "]";    }}
```

#### 2）配置\<resultMap> 元素

在 SQL 映射文件 UserMapper.xml 中配置 <resultMap> 元素，其属性 type 引用 POJO 类。具体配置如下：

```java
package com.pojo;
public class MapUser {
    private Integer m_uid;
    private String m_uname;
    private String m_usex;
    // 此处省略setter和getter方法
    @Override
    public String toString() {
        return "User[uid=" + m_uid + ",uname=" + m_uname + ",usex=" + m_usex
                + "]";
    }
}
```

#### 3）配置\<select>元素

在 SQL 映射文件 UserMapper.xml 中配置 <select> 元素，其属性 resultMap 引用了 <resultMap> 元素的 id。具体配置如下：

```xml
<!-- 使用自定义结果集类型查询所有用户 -->
<select id="selectResultMap" resultMap="myResult">
    select * from user
</select>
```

#### 4）添加接口方法

在 com.dao.UserDao 接口中添加以下接口方法：

public List<MapUser> selectResultMap();

#### 5）调用接口方法

在 com.controller 包的 UserController 类中调用接口方法，具体代码如下：

```java
// 使用resultMap映射结果集
List<MapUser> listResultMap = userDao.selectResultMap();
for (MapUser myUser : listResultMap) {
    System.out.println(myUser);
}
```

----



## MyBatis关联查询（级联查询）

级联关系是一个数据库实体的概念，有 3 种级联关系，分别是一对一级联、一对多级联以及多对多级联。

级联的优点是获取关联数据十分方便，但是级联过多会增加数据库系统的复杂度，同时降低系统的性能。

在实际开发中要根据实际情况判断是否需要使用级联。更新和删除的级联关系很简单，由数据库内在机制即可完成。本节只讲述级联查询的相关实现。

如果表 A 中有一个外键引用了表 B 的主键，A 表就是子表，B 表就是父表。当查询表 A 的数据时，通过表 A 的外键将表 B 的相关记录返回，这就是级联查询。例如，当查询一个人的信息时，同时根据外键（身份证号）将他的身份证信息返回。

### MyBatis一对一关联查询（级联查询）

一对一级联关系在现实生活中是十分常见的，例如一个大学生只有一张一卡通，一张一卡通只属于一个学生。再如人与身份证的关系也是一对一的级联关系。

MyBatis 如何处理一对一级联查询呢？在 MyBatis 中，通过 <resultMap> 元素的子元素 <association> 处理这种一对一级联关系。

在 <association> 元素中通常使用以下属性。

- property：指定映射到实体类的对象属性。
- column：指定表中对应的字段（即查询返回的列名）。
- javaType：指定映射到实体对象属性的类型。
- select：指定引入嵌套查询的子 SQL 语句，该属性用于关联映射中的嵌套查询。


下面以个人与身份证之间的关系为例讲解一对一级联查询的处理过程，读者只需参考该实例即可学会一对一级联查询的 MyBatis 实现。

#### 1）创建数据表

```sql
CREATE TABLE 'idcard' (
    'id' tinyint(2) NOT NULL AUTO_INCREMENT,
    'code' varchar(18) COLLATE utf8_unicode_ci DEFAULT NULL,
    PRIMARY KEY ('id')
)；
CREATE TABLE 'person'(
    'id' tinyint(2) NOT NULL,
    'name' varchar(20) COLLATE utf8_unicode_ci DEFAULT NULL,
    'age' int(11) DEFAULT NULL,
    'idcard_id' tinyint(2) DEFAULT NULL,
    PRIMARY KEY ('id'),
    KEY 'idcard_id' ('idcard_id'),
    CONSTRAINT 'idcard_id' FOREIGN KEY ('idcard_id') REFERENCES 'idcard'('id')
);
```

#### 2）创建持久化类

在 myBatisDemo02 应用的 com.po 包中创建数据表对应的持久化类 Idcard 和 Person。

Idcard 的代码如下：

```java
package com.mybatis.po;
public class Idcard {
    private Integer id;
    private String code;
    // 省略setter和getter方法
    /**
     * 为方便测试，重写了toString方法
     */
    @Override
    public String toString() {
        return "Idcard [id=" + id + ",code=" + code + "]";
    }
}
```

Person 的代码如下：

```java
package com.mybatis.po;
public class Person {
    private Integer id;
    private String name;
    private Integer age;
    // 个人身份证关联
    private Idcard card;
    // 省略setter和getter方法
    @Override
    public String toString() {
        return "Person[id=" + id + ",name=" + name + ",age=" + age + ",card="
                + card + "]";
    }
}
```

#### 3）创建映射文件

首先，在 MyBatis 的核心配置文件 mybatis-config.xml（com.mybatis）中打开延迟加载开关，代码如下：

```xml
<!--在使用MyBatis嵌套查询方式进行关联查询时，使用MyBatis的延迟加载可以在一定程度上提高查询效率-->
<settings>
    <!--打开延迟加载的开关-->
    <setting name= "lazyLoadingEnabled" value= "true"/>
    <!--将积极加载改为按需加载-->
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

然后，在 myBatisDemo02 应用的 com.mybatis 中创建两张表对应的映射文件 IdCardMapper.xml 和 PersonMapper.xml。在 PersonMapper.xml 文件中以 3 种方式实现“根据 id 查询个人信息”的功能，详情请看代码备注。

IdCardMapper.xml 的代码如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dao.IdCardDao">
    <select id="selectCodeById" parameterType="Integer" resultType= "com.po.Idcard">
        select * from idcard where id=#{id}
    </select>
</mapper>
```

PersonMapper.xml 的代码如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dao.PersonDao">
    <!-- 一对一根据id查询个人信息：级联查询的第一种方法（嵌套查询，执行两个SQL语句）-->
    <resultMap type="com.po.Person" id="cardAndPerson1">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="age" column="age"/>
        <!-- 一对一级联查询-->
        <association property="card" column="idcard_id" javaType="com.po.Idcard"
        select="com.dao.IdCardDao.selectCodeByld"/>
    </resultMap>
    <select id="selectPersonById1" parameterType="Integer" resultMap=
    "cardAndPerson1">
        select * from person where id=#{id}
    </select>
    <!--对一根据id查询个人信息：级联查询的第二种方法（嵌套结果，执行一个SQL语句）-->
    <resultMap type="com.po.Person" id="cardAndPerson2">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="age" column="age"/>
        <!-- 一对一级联查询-->
        <association property="card" javaType="com.po.Idcard">
            <id property="id" column="idcard_id"/>
            <result property="code" column="code"/>
        </association>
    </resultMap>
    <select id="selectPersonById2" parameterType="Integer" resultMap= "cardAndPerson2">
        select p.*,ic.code
        from person p, idcard ic
        where p.idcard_id=ic.id and p.id=#{id}
    </select>
    <!-- 一对一根据id查询个人信息：连接查询（使用POJO存储结果）-->
    <select id="selectPersonById3" parameterType="Integer" resultType= "com.pojo.SelectPersonById">
        select p.*,ic.code
        from person p, idcard ic
        where p.idcard_id = ic.id and p.id=#{id}
    </select>
</mapper>
```

#### 4）创建 POJO 类

在 myBatisDemo02 应用的 com.pojo 包中创建在第 3 步中使用的 POJO 类 com.pojo.SelectPersonById。

SelectPersonById 的代码如下：

```java
package com.pojo;
public class SelectPersonById {
    private Integer id;
    private String name;
    private Integer age;
    private String code;
    //省略setter和getter方法
    @Override
    public String toString() {
        return "Person [id=" +id+",name=" +name+ ",age=" +age+ ",code=" +code+ "]";
    }
}
```

#### 5）创建数据操作接口

在 myBatisDemo02 应用的 com.dao 包中创建第 3 步中映射文件对应的数据操作接口 IdCardDao 和 PersonDao。

IdCardDao 的代码如下：

```java
@Repository("idCardDao")
@Mapper
public interface IdCardDao {
    public Idcard selectCodeById(Integer i);
}
```

PersonDao 的代码如下：

```java
@Repository("PersonDao")
@Mapper
public interface PersonDao {
    public Person selectPersonById1(Integer id);
    public Person selectPersonById2(Integer id);
    public SelectPersonById selectPersonById3(Integer id);
}
```

#### 6）调用接口方法及测试

在 myBatisDemo02 应用的 com.controller 包中创建 OneToOneController 类，在该类中调用第 5 步的接口方法，同时创建测试类 TestOneToOne。

OneToOneController 的代码如下：

```java
@Controller("oneToOneController")
public class OneToOneController {
    @Autowired
    private PersonDao personDao;
    public void test(){
        Person p1 = personDao.selectPersonById1(1);
        System.out.println(p1);
        System.out.println("=============================");
        Person p2 = personDao.selectPersonById2(1);
        System.out.println(p2);
        System.out.println("=============================");
        selectPersonById p3 = personDao.selectPersonById3(1);
        System.out.println(p3);
    }
}
```

TestOneToOne 的代码如下：

```java
public class TestOneToOne {
    public static void main(String[] args) {
        ApplicationContext appcon = new ClassPathXmlApplicationContext("applicationContext.xml");
        OneToOneController oto = (OneToOneController)appcon.getBean("oneToOne-Controller");
        oto.test();
    }
}
```

上述测试类的运行结果如图 1 所示。

![一对一级联查询结果](http://c.biancheng.net/uploads/allimg/190711/5-1ZG1153452391.png)
图 1 一对一级联查询结果

---



### MyBatis一对多关联查询（级联查询）

在《[MyBatis一对一关联查询](http://c.biancheng.net/view/4368.html)》教程中学习了 MyBatis 如何处理一对一级联查询，那么 MyBatis 又是如何处理一对多级联查询的呢？在实际生活中一对多级联关系有许多，例如一个用户可以有多个订单，而一个订单只属于一个用户。

下面以用户和订单之间的关系为例讲解一对多级联查询（实现“根据 uid 查询用户及其关联的订单信息”的功能）的处理过程，读者只需参考该实例即可学会一对多级联查询的 MyBatis 实现。

#### 1）创建数据表

本实例需要两张数据表，一张是用户表 user，一张是订单表 orders，这两张表具有一对多的级联关系。user 表在前面已创建，orders 表的创建代码如下：

```
CREATE TABLE `orders` (
    `id` tinyint(2) NOT NULL AUTO_INCREMENT,
    `ordersn` varchar(10) DEFAULT NULL,
    `user_id` tinyint(2) DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 2）创建持久化类

在 myBatisDemo02 应用的 com.po 包中创建数据表 orders 对应的持久化类 Orders，user 表对应的持久化类 MyUser 在前面已创建，但需要为 MyUser 添加如下属性：

```java
// 一对多级联查询，用户关联的订单
private List<Orders> ordersList;
```



同时，需要为该属性添加 setter 和 getter 方法。

Orders 类的代码如下：

```java
package com.po;
public class Orders {
    private Integer id;
    private String ordersn;
    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getOrdersn() {
        return ordersn;
    }
    public void setOrdersn(String ordersn) {
        this.ordersn = ordersn;
    }
    @Override
    public String toString() {
        return "Orders[id=" + id + ",ordersn=" + ordersn + "]";
    }
}
```

#### 3）创建映射文件

在 myBatisDemo02 应用的 com.mybatis 中创建两张表对应的映射文件 UserMapper.xml 和 OrdersMapper.xml。映射文件 UserMapper.xml 在前面已创建，但需要添加以下配置才能实现一对多级联查询（根据 uid 查询用户及其关联的订单信息）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mapper.UserMapper">
    <!-- 一对多 根据uid查询用户及其关联的订单信息：级联查询的第一种方法（嵌套查询） -->
    <resultMap type="com.po.MyUser" id="userAndOrders1">
        <id property="uid" column="uid" />
        <result property="uname" column="uname" />
        <result property="usex" column="usex" />
        <!-- 一对多级联查询，ofType表示集合中的元素类型，将uid传递给selectOrdersByld -->
        <collection property="ordersList" ofType="com.po.Orders"
            column="uid" select="com.dao.OrdersDao.selectOrdersByld" />
    </resultMap>
    <select id="selectUserOrdersById1" parameterType="Integer"
        resultMap="userAndOrders1">
        select * from user where uid = #{id}
    </select>
    <!--对多根据uid查询用户及其关联的订单信息：级联查询的第二种方法（嵌套结果） -->
    <resultMap type="com.po.MyUser" id="userAndOrders2">
        <id property="uid" column="uid" />
        <result property="uname" column="uname" />
        <result property="usex" column="usex" />
        <!-- 对多级联查询，ofType表示集合中的元素类型 -->
        <collection property="ordersList" ofType="com.po.Orders">
            <id property="id" column="id" />
            <result property="ordersn" column="ordersn" />
        </collection>
    </resultMap>
    <select id="selectUserOrdersById2" parameterType="Integer"
        resultMap="userAndOrders2">
        select u.*,o.id, o.ordersn from user u, orders o where u.uid
        = o.user_id and
        u.uid=#{id}
    </select>
    <!-- 一对多 根据uid查询用户及其关联的订单信息：连接查询（使用POJO存储结果） -->
    <select id="selectUserOrdersById3" parameterType="Integer"
        resultType="com.pojo.SelectUserOrdersById">
        select u.*, o.id, o.ordersn from user u, orders o where
        u.uid = o.user_id
        and u.uid=#{id}
    </select>
</mapper>
```

OrdersMapper.xml 的配置代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dao.OrdersDao">
    <!-- 根据用户uid查询订单信息 -->
    <select id="selectOrdersById" resultType="com.po.Orders"
        parameterType="Integer">
        select * from orders where user_id=#{id}
    </select>
</mapper>
```

#### 4）创建 POJO 类

在 myBatisDemo02 应用的 com.pojo 包中创建在第 3 步中使用的 POJO 类 com.pojo. SelectUserOrdersById。

SelectUserOrdersById 的代码如下：

```java
package com.po;
public class SelectUserOrdersById {
    private Integer uid;
    private String uname;
    private String usex;
    private Integer id;
    private String ordersn;
    // 省略setter和getter方法
    @Override
    public String toString() { // 为了方便查看结果，重写了toString方法
        return "User[uid=" + uid + ",uname=" + uname + ",usex=" + usex
                + ",oid=" + id + ",ordersn=" + ordersn + "]";
    }
}
```

#### 5）创建数据操作接口

在 myBatisDemo02 应用的 com.dao 包中创建第 3 步中映射文件对应的数据操作接口 OrdersDao 和 UserDao。

OrdersDao 的代码如下：

```java
package com.dao;
import java.util.List;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;
import com.po.Orders;
@Repository("ordersDao")
@Mapper
public interface OrdersDao {
    public List<Orders> selectOrdersById(Integer uid);
}
```

UserDao 接口在前面已创建，这里只需添加如下接口方法：

```java
package com.dao;
import java.util.List;
import java.util.Map;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;
import com.po.MyUser;
import com.po.SelectUserOrdersById;
@Repository("userDao")
@Mapper
public interface UserDao {
    public MyUser selectOrdersById1(Integer uid);
    public MyUser selectOrdersById2(Integer uid);
    public List<SelectUserOrdersById> selectOrdersById3(Integer uid);
}
```

#### 6）调用接口方法及测试

在 myBatisDemo02 应用的 com.controller 包中创建 OneToMoreController 类，在该类中调用第 5 步的接口方法，同时创建测试类 TestOneToMore。

OneToMoreController 的代码如下：

```java
@Controller("oneToMoreController")
public class oneToMoreController {
    @Autowired
    private UserDao userDao;
    public void test(){
        //查询一个用户及订单信息
        MyUser auser1 = userDao.selectUserOrderById1(1);
        System.out.println(auser1);
        System.out.println("=============================");
        MyUser auser2 = userDao.selectUserOrderById2(1);
        System.out.println(auser2);
        System.out.println("=============================");
        List<SelectUserOrdersById> auser3 = userDao.selectUserOrdersById3(1);
        System.out.println(auser3);
        System.out.println("=============================");
    }
}
```

TestOneToMore 的代码如下：

```java
public class TestOneToMore {
    public static void main(String[] args) {
        ApplicationContext appcon = new ClassPathXmlApplicationContext("applicationContext.xml");
        OneToMoreController otm = (OneToMoreController)appcon.getBean("oneToMoreController");
        otm.test();
    }
}
```

测试类的运行结果如图 1 所示。



![一对多级联查询结果](http://c.biancheng.net/uploads/allimg/190711/5-1ZG1160402a8.png)
图 1 一对多级联查询结果

---



### MyBatis多对多关联查询（级联查询）

其实，MyBatis 没有实现多对多级联，这是因为多对多级联可以通过两个一对多级联进行替换。

例如，一个订单可以有多种商品，一种商品可以对应多个订单，订单与商品就是多对多的级联关系，使用一个中间表（订单记录表）就可以将多对多级联转换成两个一对多的关系。

下面以订单和商品（实现“查询所有订单以及每个订单对应的商品信息”的功能）为例讲解多对多级联查询。

#### 1）创建数据表

订单表在前面已创建，这里需要创建商品表 product 和订单记录表 orders_detail，创建代码如下：

```sql
CREATE TABLE 'product'(
    'id' tinyint(2) NOT NULL,
    'name' varchar(50) COLLATE utf8_unicode_ci DEFAULT NULL,
    'price' double DEFAULT NULL,
    PRIMARY KEY ('id')
)；
CREATE TABLE 'orders_detail'(
    'id' tinyint(2) NOT NULL AUTO_INCREMENT,
    'orders_id' tinyint(2) DEFAULT NULL,
    'product_id' tinyint(2) DEFAULT NULL,
    PRIMARY KEY ('id'),
    KEY 'orders_id' ('orders_id'),
    KEY 'product_id' ('product_id'),
    CONSTRAINT 'orders_id' FOREIGN KEY ('orders_id') REFERENCES 'orders' ('id'),
    CONSTRAINT 'product_id' FOREIGN KEY ('product_id') REFERENCES 'product' ('id')
);
```

#### 2）创建持久化类

在 myBatisDemo02 应用的 com.po 包中创建数据表 product 对应的持久化类 Product，而中间表 orders_detail 不需要持久化类，但需要在订单表 orders 对应的持久化类 Orders 中添加关联属性。

Product 的代码如下：

```java
package com.po;
import java.util.List;
public class Product {
    private Integer id;
    private String name;
    private Double price;
    // 多对多中的一个一对多
    private List<Orders> orders;
    // 省略setter和getter方法
    @Override
    public String toString() { // 为了方便查看结果，重写了toString方法
        return "Product[id=" + id + ",name=" + name + ",price=" + price + "]";
    }
}
```

Orders 的代码如下：

```java
package com.po;
import java.util.List;
public class Orders {
    private Integer id;
    private String ordersn;
    // 多对多中的另一个一对多
    private List<Product> products;
    // 省略setter和getter方法
    @Override
    public String toString() {
        return "Orders[id=" + id + ",ordersn=" + ordersn + ",products="
                + products + "]";
    }
}
```

#### 3）创建映射文件

本实例只需在 com.mybatis 的 OrdersMapper.xml 文件中追加以下配置即可实现多对多级联查询。

#### 4）创建 POJO 类

该实例不需要创建 POJO 类。

#### 5）添加数据操作接口方法

在 Orders 接口中添加以下接口方法：

```java
public List<Orders> selectallOrdersAndProducts();
```

#### 6）调用接口方法及测试

在 myBatisDemo02 应用的 com.controller 包中创建 MoreToMoreController 类，在该类中调用第 5 步的接口方法，同时创建测试类 TestMoreToMore。

MoreToMoreController 的代码如下：

```java
package com.controller;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import com.dao.OrdersDao;
import com.po.Orders;
@Controller("moreToMoreController")
public class MoreToMoreController {
    @Autowired
    private OrdersDao ordersDao;
    public void test() {
        List<Orders> os = ordersDao.selectallOrdersAndProducts();
        for (Orders orders : os) {
            System.out.println(orders);
        }
    }
}
```

TestMoreToMore 的代码如下：

```java
javbapackage com.controller;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class TestMoreToMore {
    public static void main(String[] args) {
        ApplicationContext appcon = new ClassPathXmlApplicationContext(
                "applicationContext.xml");
        MoreToMoreController otm = (MoreToMoreController) appcon
                .getBean("moreToMoreController");
        otm.test();
    }
}
```

上述测试类的运行结果如图 1 所示。



![多对多级联查询结果](http://c.biancheng.net/uploads/allimg/190711/5-1ZG11625102G.png)
图 1 多对多级联查询结果

## 动态 SQL

### MyBatis动态sql之if标签：条件判断

开发人员通常根据需求手动拼接 SQL 语句，这是一个极其麻烦的工作，而 MyBatis 提供了对 SQL 语句动态组装的功能，恰能解决这一问题。

MyBatis 的动态 SQL 元素与 J[STL](http://c.biancheng.net/stl/) 或 XML 文本处理器相似，常用 <if>、<choose>、<when>、<otherwise>、<trim>、<where>、<set>、<foreach> 和 <bind> 等元素。

创建 myBatisDemo03 应用，并将《[MyBatis与Spring的整合实例详解](http://c.biancheng.net/view/4355.html)》的 MyBatis-[Spring](http://c.biancheng.net/spring/) 应用的所有 JAR 包和 src 中所有 [Java](http://c.biancheng.net/java/) 程序与 XML 文件都复制到 myBatisDemo03 的相应位置。

动态 SQL 通常要做的事情是有条件地包含 where 子句的一部分，所以在 MyBatis 中 <if> 元素是最常用的元素，它类似于 Java 中的 if 语句。在 myBatisDemo03 应用中测试 <if> 元素，具体过程如下：

#### 1）添加 SQL 映射语句

在 com.mybatis 包的 UserMapper.xml 文件中添加如下 SQL 映射语句：

```xml
<!--使用 if 元素根据条件动态查询用户信息-->
<select id="selectUserByIf" resultType="com.po.MyUser" parameterType="com.po.MyUser">
    select * from user where 1=1
    <if test="uname!=null and uname!=''">
        and uname like concat('%',#{uname},'%')
    </if >
    <if test="usex !=null and usex !=''">
        and usex=#{usex}
    </if >
</select>
```

#### 2）添加数据操作接口方法

在 com.dao 包的 UserDao 接口中添加如下数据操作接口方法：

```java
public List<MyUser> selectUserByIf(MyUser user);
```

#### 3）调用数据操作接口方法

在 com.controller 包的 UserController 类中添加如下程序调用数据操作接口方法。

```
// 使用 if 元素查询用户信息
MyUser ifmu=new MyUser();
ifmu.setUname ("张");
ifmu.setUsex ("女");
List<MyUser> listByif=userDao.selectUserByIf(ifmu);
System.out.println ("if元素================");
for (MyUser myUser:listByif) {
    System.out.println(myUser);
}
```

#### 4）测试动态 SQL 语句

运行 com.controller 包中的 TestController 主类，测试动态 SQL 语句。

### MyBatis动态sql之choose、when、otherwise

有些时候不想用到所有的条件语句，而只想从中择取一二，针对这种情况，MyBatis 提供了 <choose> 元素，它有点像 [Java](http://c.biancheng.net/java/) 中的 switch 语句。在 myBatisDemo03 应用中测试 <choose> 元素，具体过程如下：

#### 1）添加 SQL 映射语句

在 com.mybatis 包的 UserMapper.xml 文件中添加如下 SQL 映射语句：

```
<!--使用choose、when、otherwise元素根据条件动态查询用户信息--><select id="selectUserByChoose" resultType="com.po.MyUser" parameterType= "com.po.MyUser">    select * from user where 1=1    <choose>        <when test="uname!=null and uname!=''">            and uname like concat('%',#{uname},'%')        </when>        <when test="usex!=null and usex!=''">            and usex=#{usex}        </when>        <otherwise>            and uid > 10        </otherwise>    </choose></select>
```

#### 2）添加数据操作接口方法

在 com.dao 包的 UserDao 接口中添加如下数据操作接口方法：

public List<MyUser> selectUserByChoose(MyUser user);

#### 3）调用数据操作接口方法

在 com.controller 包的 UserController 类中添加如下程序调用数据操作接口方法。

```
// 使用 choose 元素查询用户信息MyUser choosemu=new MyUser();choosemu.setUname("");choosemu.setUsex("");List<MyUser> listByChoose = UserDao.selectUserEyChoose(choosemu);System.out.println ("choose 元素================");for (MyUser myUser:listByChoose) {    System.out.println(myUser);}
```

#### 4）测试动态 SQL 语句

运行 com.controller 包中的 TestController 主类，测试动态 SQL 语句。

### MyBatis动态sql之trim、where、set标签

本节主要讲解 MyBatis 动态 Sql 的 trim、where、set 标签。

#### \<trim>元素

\<trim> 元素的主要功能是可以在自己包含的内容前加上某些前缀，也可以在其后加上某些后缀，与之对应的属性是 prefix 和 suffix。

可以把包含内容的首部某些内容覆盖，即忽略，也可以把尾部的某些内容覆盖，对应的属性是 prefixOverrides 和 suffixOverrides。正因为 <trim> 元素有这样的功能，所以也可以非常简单地利用 <trim> 来代替 <where> 元素的功能。

在 myBatisDemo03 应用中测试 <trim> 元素，具体过程如下：

##### 1）添加 SQL 映射语句

在 com.mybatis 包的 UserMapper.xml 文件中添加如下 SQL 映射语句：

```xml
<!--使用trim元素根据条件动态查询用户信息-->
<select id="selectUserByTrim" resultType="com.po.MyUser"parameterType="com.po.MyUser">
    select * from user
    <trim prefix="where" prefixOverrides = "and | or">
        <if test="uname!=null and uname!=''">
            and uname like concat('%',#{uname},'%')
        </if>
        <if test="usex!=null and usex!=''">
            and usex=#{usex}
        </if>
    </trim>
</select>
```

##### 2）添加数据操作接口方法

在 com.dao 包的 UserDao 接口中添加如下数据操作接口方法：

```java
public List<MyUser> selectUserByTrim(MyUser user);
```

##### 3）调用数据操作接口方法

在 com.controller 包的 UserController 类中添加如下程序调用数据操作接口方法。

```java
// 使用trim元素查询用户信息
MyUser trimmu=new MyUser();
trimmu.setUname ("张");
trimmu.setUsex("男");
List<MyUser> listByTrim=userDao.selectUserByTrim(trimmu);
System.out.println ("trim 元素=========================");
for (MyUser myUser:listByTrim) {
    System.out.println(myUser);
}
```

##### 4）测试动态 SQL 语句

运行 com.controller 包中的 TestController 主类，测试动态 SQL 语句。

#### \<where> 元素

<where> 元素的作用是会在写入 <where> 元素的地方输出一个 where 语句，另外一个好处是不需要考虑 <where> 元素里面的条件输出是什么样子的，MyBatis 将智能处理。如果所有的条件都不满足，那么 MyBatis 就会查出所有的记录，如果输出后是以 and 开头的，MyBatis 会把第一个 and 忽略。

当然如果是以 or 开头的，MyBatis 也会把它忽略；此外，在 <where> 元素中不需要考虑空格的问题，MyBatis 将智能加上。

在 myBatisDemo03 应用中测试 <where> 元素，具体过程如下：

#### 1）添加 SQL 映射语句

在 com.mybatis 包的 UserMapper.xml 文件中添加如下 SQL 映射语句：

```xml
<!--使用where元素根据条件动态查询用户信息-->
<select id="selectUserByWhere" resultType="com.po.MyUser" parameterType="com.po.MyUser">
    select * from user
    <where>
        <if test="uname != null and uname ! = ''">
            and uname like concat('%',#{uname},'%')
        </if>
        <if test="usex != null and usex != '' ">
            and usex=#{usex}
        </if >
    </where>
</select>
```

#### 2）添加数据操作接口方法

在 com.dao 包的 UserDao 接口中添加如下数据操作接口方法：

```java
public List<MyUser> selectUserByWhere(MyUser user);
```

#### 3）调用数据操作接口方法

在 com.controller 包的 UserController 类中添加如下程序调用数据操作接口方法。

```java
// 使用where元素查询用户信息
MyUser wheremu=new MyUser();
wheremu.setUname ("张");
wheremu.setUsex("男");
List<MyUser> listByWhere=userDao.selectUserByWhere(wheremu);
System.out.println ("where 元素=========================");
for (MyUser myUser:listByWhere) {
    System.out.println(myUser);
}
```

#### 4）测试动态 SQL 语句

运行 com.controller 包中的 TestController 主类，测试动态 SQL 语句。

#### \<set>元素

在动态 update 语句中可以使用 <set> 元素动态更新列。在 myBatisDemo03 应用中测试 <set> 元素，具体过程如下：

##### 1）添加 SQL 映射语句

在 com.mybatis 包的 UserMapper.xml 文件中添加如下 SQL 映射语句：

```
<!--使用set元素动态修改一个用户-->
<update id="updateUserBySet" parameterType="com.po.MyUser">     
    update user
    <set>
        <if test="uname!=null">uname=#{uname}</if>
        <if test="usex!=null">usex=#{usex}</if>
    </set>
    where uid=#{uid}
</update>
```

##### 2）添加数据操作接口方法

在 com.dao 包的 UserDao 接口中添加如下数据操作接口方法：

```java
public int updateUserBySet(MyUser user);
```



##### 3）调用数据操作接口方法

在 com.controller 包的 UserController 类中添加如下程序调用数据操作接口方法。

```java
// 使用set元素查询用户信息
MyUser setmu=new MyUser();
setmu.setUid (1);
setmu.setUname("张九");
int setup=userDao.updateUserBySet(setmu);
System.out.println ("set 元素修改了"+setup+"条记录");
System.out.println ("=========================")
```

##### 4）测试动态 SQL 语句

运行 com.controller 包中的 TestController 主类，测试动态 SQL 语句。

----



### MyBatis动态sql之foreach标签

<foreach> 元素主要用在构建 in 条件中，它可以在 SQL 语句中迭代一个集合。

<foreach> 元素的属性主要有 item、index、collection、open、separator、close。

- item 表示集合中每一个元素进行迭代时的别名。
- index 指定一个名字，用于表示在迭代过程中每次迭代到的位置。
- open 表示该语句以什么开始。
- separator 表示在每次进行迭代之间以什么符号作为分隔符。
- close 表示以什么结束。


在使用 <foreach> 元素时，最关键、最容易出错的是 collection 属性，该属性是必选的，但在不同情况下该属性的值是不一样的，主要有以下 3 种情况：

- 如果传入的是单参数且参数类型是一个 List，collection 属性值为 list。
- 如果传入的是单参数且参数类型是一个 array 数组，collection 的属性值为 array。
- 如果传入的参数是多个，需要把它们封装成一个 Map，当然单参数也可以封装成 Map。Map 的 key 是参数名，collection 属性值是传入的 List 或 array 对象在自己封装的 Map 中的 key。


在 myBatisDemo03 应用中测试 <foreach> 元素，具体过程如下：

#### 1）添加 SQL 映射语句

在 com.mybatis 包的 UserMapper.xml 文件中添加如下 SQL 映射语句：

```
<!--使用foreach元素查询用户信息-->
<select id="selectUserByForeach" resultType="com.po.MyUser" parameterType=
"List">
    select * from user where uid in
    <foreach item="item" index="index" collection="list"
    open="(" separator="," close=")">
        # {item}
    </foreach>
</select>
```

#### 2）添加数据操作接口方法

在 com.dao 包的 UserDao 接口中添加如下数据操作接口方法：

public List<MyUser> selectUserByForeach(List<Integer> listId);

#### 3）调用数据操作接口方法

在 com.controller 包的 UserController 类中添加如下程序调用数据操作接口方法。

```
//使用foreach元素查询用户信息
List<Integer> listId=new ArrayList<Integer>();
listId.add(34);
listId.add(37);
List<MyUser> listByForeach = userDao.selectUserByForeach(listId);
System.out.println ("foreach元素================");
for(MyUser myUser : listByForeach) {
    System.out.println(myUser);
}
```

#### 4）测试动态 SQL 语句

运行 com.controller 包中的 TestController 主类，测试动态 SQL 语句。

----



### MyBatis动态sql之bind标签

在进行模糊查询时，如果使用“${}”拼接字符串，则无法防止 SQL 注入问题。如果使用字符串拼接函数或连接符号，但不同数据库的拼接函数或连接符号不同。

例如 [MySQL](http://c.biancheng.net/mysql/) 的 concat 函数、Oracle 的连接符号“||”，这样 SQL 映射文件就需要根据不同的数据库提供不同的实现，显然比较麻烦，且不利于代码的移植。幸运的是，MyBatis 提供了 <bind> 元素来解决这一问题。

在 myBatisDemo03 应用中测试 <bind> 元素，具体过程如下： 

#### 1）添加 SQL 映射语句

在 com.mybatis 包的 UserMapper.xml 文件中添加如下 SQL 映射语句：

```xml
<!--使用bind元素进行模糊查询-->
<select id="selectUserByBind" resultType="com.po.MyUser" parameterType= "com.po.MyUser">
    <!-- bind 中的 uname 是 com.po.MyUser 的属性名-->
    <bind name="paran_uname" value="'%' + uname + '%'"/>
        select * from user where uname like #{paran_uname}
</select>
```

#### 2）添加数据操作接口方法

在 com.dao 包的 UserDao 接口中添加如下数据操作接口方法：

```java
public List<MyUser> selectUserByBind(MyUser user);
```



#### 3）调用数据操作接口方法

在 com.controller 包的 UserController 类中添加如下程序调用数据操作接口方法。

```java
// 使用bind元素查询用户信息
MyUser bindmu=new MyUser();
bindmu.setUname ("张");
List<MyUser> listByBind=userDao.selectUserByBind(bindmu);
System.out.println ("bind 元素=========================");
for (MyUser myUser:listByBind) {
    System.out.println(myUser);
}
```

#### 4）测试动态 SQL 语句

运行 com.controller 包中的 TestController 主类，测试动态 SQL 语句。

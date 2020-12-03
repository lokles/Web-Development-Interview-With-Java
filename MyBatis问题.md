# MyBatis问题

## 1、什么是MyBatis？

是一个ORM（对象关系映射）框架。

`Mybatis`内部封装了`jdbc`，使得开发者只需要关注`sql`语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。

`mybatis`通过xml或注解的方式将要执行的各种statement配置起来，并通过java对象和statement中`sql`的动态参数进行映射生成最终执行的`sql`语句，最后由`mybatis`框架执行`sql`并将结果映射为java对象并返回。

`MyBatis` 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。`MyBatis` 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJO映射成数据库中的记录。

## 2、什么是SQL注入？如何防止？

是一种**注入攻击**，它通过将任意代码插入数据库查询，使得攻击者完全控制数据库服务器。  攻击者可以使用SQL注入漏洞绕过应用程序安全措施；可以绕过网页或Web应用程序的身份验证和授权，并检索整个SQL数据库的内容；还可以使用SQL注入来添加，修改和删除数据库中的记录。

${}是字符串替换，相当于直接显示数据，**#{}是预编译处理**，相当于对数据加上双引号。即#{}是将传入的值当做字符串的形式，先替换为?号，然后调用`PreparedStatement`的set方法来赋值，而$是将传入的数据直接显示生成`sql`语句。

**使用#{}可以有效的防止SQL注入**，`MyBatis`启用了预编译功能，在SQL执行前，会先将上面的SQL发送给数据库进行编译；执行时，直接使用编译好的SQL，替换占位符“?”就可以了。因为SQL注入只能对编译过程起作用，所以这样的方式就很好地避免了SQL注入的问题。

**原理：**

在框架底层，是JDBC中的`PreparedStatement`类在起作用，`PreparedStatement`是我们很熟悉的Statement的子类，**它的对象包含了编译好的SQL语句**。这种“准备好”的方式不仅能提高安全性，而且在多次执行同一个SQL时，能够提高效率。原因是SQL已编译好，再次执行时无需再编译。

```mysql
--Mybatis在处理#{}时
select id,name,age from student where id =#{id}
当前端把id值1传入到后台的时候，就相当于:
select id,name,age from student where id ='1'

--Mybatis在处理${}时
select id,name,age from student where id =${id}
当前端把id值1传入到后台的时候，就相当于：
select id,name,age from student where id = 1
```



## 2.1什么情况下用${ }、什么情况下用#{ }？ 

Mybatis 中优先使用 #{}。当需要**动态传入表名或列名**时，使用 ${} 。

## 3、Mybatis 中一级缓存与二级缓存的区别？

合理利用缓存可以避免频繁操作数据库，减轻数据库压力，同时提高系统的性能。

一级缓存是`sqlSession`级别的，`Mybatis`对缓存提供支持，但是在没有配置的默认情况下，它只开启一级缓存。一级缓存在操作数据库时需要构造`sqlSession`对象，在对象中有一个数据结构`（HashMap）`用于存储缓存数据。不同的`sqlSession`之间的缓存数据区域是互相不影响的。也就是他只能作用在同一个`sqlSession`中，不同的`sqlSession`中的缓存是互相不能读取的。当在同一个`sqlSession`中执行两次相同的`sql`语句时，第一次执行完毕会将数据库中查询的数据写到缓存（内存）。

二级缓存是mapper级别的缓存，多个`sqlSession` 去操作同一个mapper的`sql`语句，它们可以公用二级缓存，二级缓存是跨`sqlSession`的。

## 4、使用MyBatis的Mapper接口调用时有什么要求？

1. 接口中的方法名应与mapper中的每一个`sql`的id相同。
2. 接口方法的输出参数类型和 mapper.xml 中定义的每个 `sql` 的 `resultType` 的类型相同
3. 接口方法的输入参数应与`mapper.xml`中定义的每一个`sql`的`parameterType`类型相同。

## 5、MyBatis的运行步骤？

1. 创建 `SqlSessionFactory`
2. 通过 `SqlSessionFactory` 创建 `SqlSession`
3. 通过 `sqlsession` 执行数据库操作
4. 调用 session.commit()提交事务
5. 调用 session.close()关闭会话

## 6、MyBatis中接口绑定有几种实现方式？

1. **通过注解绑定**，在接口的方法上通过@Select@Update等注解里面包含SQL语句来绑定。
2. **通过Mapper.XML文件中编写SQL语句来绑定**，SQL语句的id必须与对应接口的方法名一致，输入输出的类型也必须一致。

## 7、MyBatis的工作原理说一下？

MyBatis先封装SQL，接着调用JDBC操作数据库，最后把数据库返回的表结果封装成Java类。

MyBatis也有四大核心对象：

1. `SqlSession`对象，该对象中包含了执行SQL语句的所有方法。类似于JDBC里面的Connection。
2. Executor接口，它将根据`SqlSession`传递的参数动态地生成需要执行的SQL语句，同时负责查询缓存的维护。类似于JDBC里面的`Statement/PrepareStatement`。
3. `MappedStatement`对象，该对象是对映射SQL的封装，用于存储要映射的SQL语句的id、参数等信息。
4. `ResultHandler`对象，用于对返回的结果进行处理，最终得到自己想要的数据格式或类型。可以自定义返回类型。

## 8、MyBatis 详细工作流程？

![](http://www.mybatis.cn/usr/uploads/2019/10/326517643.png)

1. 读取`MyBatis`的配置文件。`mybatis-config.xml`为`MyBatis`的全局配置文件，用于配置数据库连接信息。
2. 加载映射文件。映射文件即SQL映射文件，该文件中配置了操作数据库的SQL语句，需要在`MyBatis`配置文件`mybatis-config.xml`中加载。`mybatis-config.xml` 文件可以加载多个映射文件，每个文件对应数据库中的一张表。
3. 构造会话工厂。通过`MyBatis`的环境配置信息构建会话工厂`SqlSessionFactory`。
4. 创建会话对象。由会话工厂创建`SqlSession`对象，该对象中包含了执行SQL语句的所有方法。
5. Executor执行器。`MyBatis`底层定义了一个Executor接口来操作数据库，它将根据`SqlSession`传递的参数动态地生成需要执行的SQL语句，同时负责查询缓存的维护。
6. `MappedStatement`对象。在Executor接口的执行方法中有一个`MappedStatement`类型的参数，该参数是对映射信息的封装，用于存储要映射的SQL语句的id、参数等信息。
7. 输入参数映射。输入参数类型可以是Map、List等集合类型，也可以是基本数据类型和POJO类型。输入参数映射过程类似于JDBC对`preparedStatement`对象设置参数的过程。
8. 输出结果映射。输出结果类型可以是Map、List等集合类型，也可以是基本数据类型和POJO类型。输出结果映射过程类似于JDBC对结果集的解析过程。
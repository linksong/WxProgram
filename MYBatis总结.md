## MYBatis总结（一）

##### 1.什么是MYBatis

MyBatis 是支持普通 SQL 查询及动态SQL，存储过程和高级映射（比如：嵌套联合映射）的优秀持久层框架。MyBatis 消除了几乎所有的 JDBC 代码和参数的手工设置以及结果集的检索。MyBatis 使用简单的 XML 或注解用于配置和原始映射，将接口和 Java 的POJOs（Plain Old Java Objects，普通的 Java 对象）映射成数据库中的记录。

##### 2.Mybaits特点：

1. 简单易学。没有任何第三方依赖，最简单只需要2个jar包+几个sql映射文件，通过文档和源代码，即可比较完全的掌握它的设计思路和实现
2. 灵活。不会对应用程序或者数据库的现有设计强加任何影响。Sql写在xml里面，便于统一管理和优化。通过sql基本上可以实现我们不使用数据访问框架可以实现的所有功能。
3. 解除sql与程序代码的耦合。通过提供DAL层，将业务逻辑和数据访问逻辑分离，使系统的设计更清晰，更易维护，更易单元测试。
4. 提供映射标签，支持对象与数据库的ORM字段关系映射
5. 提供对象关系映射标签，支持对象关系组建维护
6. 提供xml标签，支持编写动态sql

##### 3.Mybatis,hibernate的共同点

1. 从配置文件(通常是 XML 配置文件中)得到 sessionfactory
2. 由 sessionfactory 产生 session
3. 在 session 中完成对数据的增删改查和事务提交等
4. 在用完之后关闭 session 
5. 在 Java 对象和 数据库之间有做 mapping 的配置文件，也通常是 xml 文件。

##### 4.\#{}和${}的区别是什么？

```
#{}是预编译处理，${}是字符串替换。
Mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值；
Mybatis在处理${}时，就是把${}替换成变量的值。
使用#{}可以有效的防止SQL注入，提高系统安全性。
```

##### 5.通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？

```
Dao接口，就是人们常说的Mapper接口，接口的全限名，就是映射文件中的namespace的值，接口的方法名，就是映射文件中MappedStatement的id值，接口方法内的参数，就是传递给sql的参数。Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement。
举例：com.mybatis3.mappers.StudentDao.findStudentById，可以唯一找到namespace为com.mybatis3.mappers.StudentDao下面id = findStudentById的MappedStatement。在Mybatis中，每一个<select>、<insert>、<update>、<delete>标签，都会被解析为一个MappedStatement对象。

Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。

Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。
```

##### 6.Mybatis是如何进行分页的？分页插件的原理是什么？

```
Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。
123
```

##### 7.Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？

```
答：第一种是使用<resultMap>标签，逐一定义列名和对象属性名之间的映射关系。第二种是使用sql列的别名功能，将列别名书写为对象属性名，比如T_NAME AS NAME，对象属性名一般是name，小写，但是列名不区分大小写，Mybatis会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成T_NAME AS NaMe，Mybatis一样可以正常工作。

有了列名与属性名的映射关系后，Mybatis通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。
123
```

##### 8.如何执行批量插入?

```java
首先,创建一个简单的insert语句: 
    <insert id=”insertname”> 
     insert into names (name) values (#{value}) 
    </insert>
    
然后在java代码中像下面这样执行批处理插入: 
    list<string> names = new arraylist(); 
    names.add(“fred”); 
    names.add(“barney”); 
    names.add(“betty”); 
    names.add(“wilma”); 

    // 注意这里 executortype.batch 
    sqlsession sqlsession = sqlsessionfactory.opensession(executortype.batch); 
    try { 
     	namemapper mapper = sqlsession.getmapper(namemapper.class); 
     	for (string name : names) { 
    		 mapper.insertname(name); 
     		} 
     	sqlsession.commit(); 
    } finally { 
    	 sqlsession.close(); 
   	}
```

##### 9.Mybatis动态sql是做什么的？都有哪些动态sql？能简述一下动态sql的执行原理不？

```
Mybatis动态sql可以让我们在Xml映射文件内，以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能。
Mybatis提供了9种动态sql标签：trim|where|set|foreach|if|choose|when|otherwise|bind。
其执行原理为，使用OGNL从sql参数对象中计算表达式的值，根据表达式的值动态拼接sql，以此来完成动态sql的功能。
```

#####  10.一对一、一对多的关联查询 ？

```xml
<mapper namespace="com.lcb.mapping.userMapper">  
    <!--association  一对一关联查询 -->  
    <select id="getClass" parameterType="int" resultMap="ClassesResultMap">  
        select * from class c,teacher t where c.teacher_id=t.t_id and c.c_id=#{id}  
    </select>  
    <resultMap type="com.lcb.user.Classes" id="ClassesResultMap">  
        <!-- 实体类的字段名和数据表的字段名映射 -->  
        <id property="id" column="c_id"/>  
        <result property="name" column="c_name"/>  
        <association property="teacher" javaType="com.lcb.user.Teacher">  
            <id property="id" column="t_id"/>  
            <result property="name" column="t_name"/>  
        </association>  
    </resultMap>  

    <!--collection  一对多关联查询 -->  
    <select id="getClass2" parameterType="int" resultMap="ClassesResultMap2">  
        select * from class c,teacher t,student s where c.teacher_id=t.t_id and c.c_id=s.class_id and c.c_id=#{id}  
    </select>  
    <resultMap type="com.lcb.user.Classes" id="ClassesResultMap2">  
        <id property="id" column="c_id"/>  
        <result property="name" column="c_name"/>  
        <association property="teacher" javaType="com.lcb.user.Teacher">  
            <id property="id" column="t_id"/>  
            <result property="name" column="t_name"/>  
        </association>  
        <collection property="student" ofType="com.lcb.user.Student">  
            <id property="id" column="s_id"/>  
            <result property="name" column="s_name"/>  
        </collection>  
    </resultMap>  

</mapper>  
```
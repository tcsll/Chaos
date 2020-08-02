# Mybatis缓存

先说缓存，合理使用缓存是优化中最常见的，将从数据库中查询出来的数据放入缓存中，下次使用时不必从数据库查询，而是直接从缓存中读取，避免频繁操作数据库，减轻数据库的压力，同时提高系统性能。

Mybatis的一级、二级缓存概述
1）一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存。

2）二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态),可在它的映射文件中配置<cache> ；

3）对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。

### 一级缓存

一级缓存是SqlSession级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构用于存储缓存数据。不同的sqlSession之间的缓存数据区域是互相不影响的。也就是他只能作用在同一个sqlSession中，不同的sqlSession中的缓存是互相不能读取的

![img](http://img.blog.csdn.net/20170613200330598)

用户发起查询请求，查找某条数据，sqlSession先去缓存中查找，是否有该数据，如果有，读取；

如果没有，从数据库中查询，并将查询到的数据放入一级缓存区域，供下次查找使用。

但sqlSession执行commit，即增删改操作时会清空缓存。这么做的目的是避免脏读。

如果commit不清空缓存，会有以下场景：A查询了某商品库存为10件，并将10件库存的数据存入缓存中，之后被客户买走了10件，数据被delete了，但是下次查询这件商品时，并不从数据库中查询，而是从缓存中查询，就会出现错误。

既然有了一级缓存，那么为什么要提供二级缓存呢？

二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。二级缓存的作用范围更大。

还有一个原因，实际开发中，MyBatis通常和Spring进行整合开发。Spring将事务放到Service中管理，对于每一个service中的sqlsession是不同的，这是通过mybatis-spring中的org.mybatis.spring.mapper.MapperScannerConfigurer创建sqlsession自动注入到service中的。 每次查询之后都要进行关闭sqlSession，关闭之后数据被清空。所以spring整合之后，如果没有事务，一级缓存是没有意义的。

### 二级缓存

![img](http://img.blog.csdn.net/20170613200342848)

二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

UserMapper有一个二级缓存区域（按namespace分），其它mapper也有自己的二级缓存区域（按namespace分）。每一个namespace的mapper都有一个二级缓存区域，两个mapper的namespace如果相同，这两个mapper执行sql查询到数据将存在相同的二级缓存区域中。

开启二级缓存：

1.打开总开关

在MyBatis的配置文件中加入：

```xml
<span style="font-size:18px;"><settings>    
   <!--开启二级缓存-->    
    <setting name="cacheEnabled" value="true"/>    
</settings> </span>  
```

2.在需要开启二级缓存的mapper.xml中加入cache标签

```xml
<span style="font-size:18px;"><cache/></span>  
```

3.让使用二级缓存的POJO类实现Serializable接口

```xml
<span style="font-size:18px;">public class User implements Serializable {}</span>  
```

4.测试结果

```java
public void testCache() throws Exception {  
    SqlSession sqlSession1 = sqlSessionFactory.openSession();  
    SqlSession sqlSession2 = sqlSessionFactory.openSession();  
    UserMapper userMapper1 = sqlSession1.getMapper(UserMapper.class);  
    User user1 = userMapper1.findUserById(1);  
    System.out.println(user1);  
    sqlSession1.close();  
    UserMapper userMapper2 = sqlSession2.getMapper(UserMapper.class);  
    User user2 = userMapper2.findUserById(1);  
    System.out.println(user2);  
    sqlSession2.close();  
}
```

5.输出结果

```
<span style="font-size:18px;">DEBUG [main] - Cache Hit Ratio [com.iot.mybatis.mapper.UserMapper]: 0.0  
DEBUG [main] - Opening JDBC Connection  
DEBUG [main] - Created connection 103887628.  
DEBUG [main] - Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]  
DEBUG [main] - ==>  Preparing: SELECT * FROM user WHERE id=?   
DEBUG [main] - ==> Parameters: 1(Integer)  
DEBUG [main] - <==      Total: 1  
User [id=1, username=张三, sex=1, birthday=null, address=null]  
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]  
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@631330c]  
DEBUG [main] - Returned connection 103887628 to pool.  
DEBUG [main] - Cache Hit Ratio [com.iot.mybatis.mapper.UserMapper]: 0.5  
User [id=1, username=张三, sex=1, birthday=null, address=null]</span>  
```

我们可以从打印的信息看出，两个sqlSession，去查询同一条数据，只发起一次select查询语句，第二次直接从Cache中读取。

前面我们说到，Spring和MyBatis整合时， 每次查询之后都要进行关闭sqlSession，关闭之后数据被清空。所以spring整合之后，如果没有事务，一级缓存是没有意义的。那么如果开启二级缓存，关闭sqlsession后，会把该sqlsession一级缓存中的数据添加到namespace的二级缓存中。这样，缓存在sqlsession关闭之后依然存在。

总结：

对于查询多commit少且用户对查询结果实时性要求不高，此时采用mybatis二级缓存技术降低数据库访问量，提高访问速度。

但不能滥用二级缓存，二级缓存也有很多弊端，从MyBatis默认二级缓存是关闭的就可以看出来。

二级缓存是建立在同一个namespace下的，如果对表的操作查询可能有多个namespace，那么得到的数据就是错误的。

举个简单的例子:

订单和订单详情，orderMapper、orderDetailMapper。在查询订单详情时我们需要把订单信息也查询出来，那么这个订单详情的信息被二级缓存在orderDetailMapper的namespace中，这个时候有人要修改订单的基本信息，那就是在orderMapper的namespace下修改，他是不会影响到orderDetailMapper的缓存的，那么你再次查找订单详情时，拿到的是缓存的数据，这个数据其实已经是过时的。

根据以上，想要使用二级缓存时需要想好两个问题：

1）对该表的操作与查询都在同一个namespace下，其他的namespace如果有操作，就会发生数据的脏读。

2）对关联表的查询，关联的所有表的操作都必须在同一个namespace。



# Mybatis 中$与#的区别

1. \#是将传入的值当做字符串的形式，eg:select id,name,age from student where id =#{id},当前端把id值1，传入到后台的时候，就相当于 select id,name,age from student where id ='1'
2. $是将传入的数据直接显示生成sql语句，eg:select id,name,age from student where id =${id},当前端把id值1，传入到后台的时候，就相当于 select id,name,age from student where id = 1
3. 使用#可以很大程度上防止sql注入(语句的拼接)
4. 但是如果使用在order by 中就需要使用 $
5. 在大多数情况下还是经常使用#，但在不同情况下必须使用$



我觉得#与的区别最大在于：#{} 传入值时，sql解析时，参数是带引号的，而的区别最大在于：#{} 传入值时，sql解析时，参数是带引号的，而{}穿入值，sql解析时，参数是不带引号的。

一 : 理解mybatis中 $与#

  在mybatis中的$与#都是在sql中动态的传入参数。

  eg:select id,name,age from student where name=#{name}  这个name是动态的，可变的。当你传入什么样的值，就会根据你传入的值执行sql语句。

二:使用$与#

  \#{}: 解析为一个 JDBC 预编译语句（prepared statement）的参数标记符，一个 #{ } 被解析为一个参数占位符 。

  ${}: 仅仅为一个纯碎的 string 替换，在动态 SQL 解析阶段将会进行变量替换。

 name-->cy

 eg:  select id,name,age from student where name=#{name}  -- name='cy'

​    select id,name,age from student where name=${name}   -- name=cy



# Mybatis四种分页方式

数组分页

sql分页

拦截器分页

RowBounds分页

# Mybatis底层实现原理

![img](https://img-blog.csdnimg.cn/20190610104238275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JveWFsX2xy,size_16,color_FFFFFF,t_70)
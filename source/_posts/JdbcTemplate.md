---
title: JdbcTemplate
date:  2023-08-31 08:48:38
categories:  [计算机,后端,Java,Spring,SpringAOP]
tags: article
index_img:
---



## JdbcTemplate概念及使用

​ a）Spring 框架对 JDBC 进行封装，使用 JdbcTemplate 方便实现对数据库操作

​ b）引入相关 jar 包

 c）在 spring 配置文件配置**数据库连接池xml**

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
 destroy-method="close">
 <property name="url" value="jdbc:mysql:///test" />
 <property name="username" value="root" />
 <property name="password" value="root" />
 <property name="driverClassName" value="com.mysql.jdbc.Driver" />
</bean>
```

 d）配置 JdbcTemplate 对象，注入 DataSource

```xml
<!-- JdbcTemplate 对象 -->
<bean id="jdbcTemplate" 
class="org.springframework.jdbc.core.JdbcTemplate">
 <!--注入 dataSource-->
 <property name="dataSource" ref="dataSource"></property>
<!--set方式注入-->
</bean>
```

e）创建 service 类，创建 dao 类，在 dao 注入 jdbcTemplate 对象

```xml
<!-- 组件扫描 -->
<context:component-scan base-package="com.atguigu"></context:component-scan>
```

```java
@Service
public class BookService {
 //注入 dao
 @Autowired
 private BookDao bookDao;
}

@Repository
public class BookDaoImpl implements BookDao {
 //注入 JdbcTemplate
 @Autowired
 private JdbcTemplate jdbcTemplate;
}
```

## JdbcTemplate 操作数据库（添加）

​ a）对应数据库创建实体类

​ b）创建service和dao

​ （1）在 dao 进行数据库添加操作

​ （2）调用 JdbcTemplate 对象里面 update 方法实现添加操作

```java
@Repository
public class BookDaoImpl implements BookDao {
 //注入 JdbcTemplate
 @Autowired
 private JdbcTemplate jdbcTemplate;
 //添加的方法
 @Override
 public void add(Book book) {
 //1 创建 sql 语句
 String sql = "insert into t_book values(?,?,?)";
 //2 调用方法实现
 Object[] args = {book.getUserId(), book.getUsername(),book.getUstatus()};
 int update = jdbcTemplate.update(sql,args);
 System.out.println(update);
 }
}
```

## JdbcTemplate 操作数据库（修改和删除）

```java
//1、修改
@Override
public void updateBook(Book book) {
 String sql = "update t_book set username=?,ustatus=? where user_id=?";
 Object[] args = {book.getUsername(), book.getUstatus(),book.getUserId()};
 int update = jdbcTemplate.update(sql, args);
 System.out.println(update);
}
//2、删除
@Override
public void delete(String id) {
 String sql = "delete from t_book where user_id=?";
 int update = jdbcTemplate.update(sql, id);
 System.out.println(update);
}
//使用JdbcTemplate 模板所实现的 “增删改” 都是调用了同一个 “update” 方法
```

## JdbcTemplate 操作数据库（查询返回某个值）

```java
//查询表记录数
@Override
public int selectCount() {
 String sql = "select count(*) from t_book";
//queryForObject方法中：第一个参数代表--sql语句；第二个参数代表--返回类型class  
 Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
 return count;
}
JdbcTemplate 操作数据库（
```

## JdbcTemplate 操作数据库（查询返回对象）

```java
//查询返回对象
@Override
public Book findBookInfo(String id) {
 String sql = "select * from t_book where user_id=?";
 //调用方法
/*
	queryForObject方法中：
		第一个参数：sql语句
		第二个参数：RowMapper 是接口，针对返回不同类型数据，使用这个接口里面 实现类 完成数据封装
		第三个参数：sql 语句值
*/
 Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);
 return book;
}
```

## JdbcTemplate 操作数据库（查询返回集合）

```java
//所用场景：查询图书列表分页、、
//查询返回集合
@Override
public List<Book> findAllBook() {
 String sql = "select * from t_book";
 //调用方法
 List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
 return bookList;
}
```

## JdbcTemplate 操作数据库（批量操作）

```java
//批量添加
@Override
public void batchAddBook(List<Object[]> batchArgs) {
 String sql = "insert into t_book values(?,?,?)";
//batchUpdate方法 第一个参数：sql语句		第二个参数：List集合，添加多条记录数据
 int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
 System.out.println(Arrays.toString(ints));
}

//批量添加测试
List<Object[]> batchArgs = new ArrayList<>();
Object[] o1 = {"3","java","a"};
Object[] o2 = {"4","c++","b"};
Object[] o3 = {"5","MySQL","c"};
batchArgs.add(o1);
batchArgs.add(o2);
batchArgs.add(o3);
//调用批量添加
bookService.batchAdd(batchArgs);
```

## JdbcTemplate 实现批量修改操作

```java
//批量修改(同批量添加一样，调用同一个方法)
@Override
public void batchUpdateBook(List<Object[]> batchArgs) {
 String sql = "update t_book set username=?,ustatus=? where user_id=?";
 int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
 System.out.println(Arrays.toString(ints));
}
```

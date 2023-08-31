---
title: SpringAOP的基本概念和使用
date:  2023-08-31 10:16:10
categories: [计算机,后端,Java,Spring,SpringAOP]
tags: article
index_img: 
---



## IOC容器

### 什么是IOC容器

将对象的创建交给Spring进行管理，这个管理众多的对象的容器就叫做IOC容器，由于在一般代码中，我们容易依赖很多对象，造成耦合度过高，用这种方式进行统一管理可以降低耦合度。

### IOC底层

xml解析，工厂模式，反射

### Spring提供的IOC容器实现的两种方式 

BeanFactory接口：IOC容器基本实现是Spring内部接口的使用接口，不提供给开发人员进行使用（加载配置文件时候不会创建对象，在获取对象时才会创建对象。）

一般我们采用下面这种方式进行加载

ApplicationContext接口：BeanFactory接口的子接口，提供更多更强大的功能，提供给开发人员使用（加载配置文件时候就会把在配置文件对象进行创建）

## IOC容器-Bean管理

### 何谓Bean管理

1. Spring创建对象
2. Spring注入属性

### 怎么使用

1. 需要在xml里面进行对象的创建和属性的注入

```xml
<!--对象的创建-->
<bean id="user" class="com.company.User">
    <!-- 属性的注入 -->
    <property name="bproperty" value="bvalue"></property>
</bean>


<bean id="book" class="com.example.order">
    <constructor-arg name = "oname" value="Hello"></constructor-arg>
    <constructor-arg name = "address" value="china"></constructor-arg>  
</bean>
```

2. 需要设置被注入的类

```java
public class Btest{
    private String bproperty;
    //设置注入的属性需要设置其set方法
    public void setBproperty(String bproperty){
        this.bproperty = bproperty;
    }
}
// 构造函数的注入
public class Order {
    private String oname;
    private String address;
    public Orders(String oname,String address){
        this.oname=oname;
        this.address=address;
    }

}

```

### 进阶

1. 命名空间注入

```xml
<!--1、添加p名称空间在配置文件头部-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"		<!--在这里添加一行p-->

<!--2、在bean标签进行属性注入（算是set方式注入的简化操作）-->
    <bean id="book" class="com.atguigu.spring5.Book" p:bname="very" p:bauthor="good">
    </bean>
<br />

```

2. 注入空值

```xml
<bean id="book" class="com.atguigu.spring5.Book">
    <!--（1）null值-->
    <property name="address">
        <null/><!--属性里边添加一个null标签-->
    </property>
  
    <!--（2）特殊符号赋值-->
     <!--属性值包含特殊符号
       a 把<>进行转义 &lt; &gt;
       b 把带特殊符号内容写到CDATA
      -->
        <property name="address">
            <value><![CDATA[<<南京>>]]></value>
        </property>
</bean>
```

3. 注入类

类中有引用其他的类，需要被注入进来

```java
public class UserService {//service类

    //创建UserDao类型属性，生成set方法
    private UserDao userDao;
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    public void add() {
        System.out.println("service add...............");
        userDao.update();//调用dao方法
    }
}

public class UserDaoImpl implements UserDao {//dao类

    @Override
    public void update() {
        System.out.println("dao update...........");
    }
}
```

配置文件

```xml
<!--1 service和dao对象创建-->
<bean id="userService" class="com.atguigu.spring5.service.UserService">
    <!--注入userDao对象
        name属性：类里面属性名称
        ref属性：创建userDao对象bean标签id值
    -->
    <property name="userDao" ref="userDaoImpl"></property>
</bean>

<!-- 可以理解成在bean的全局命名空间创建了一个类，让其被引用到上面了 -->
<bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>
<!-- 因此上面也可以写成这个样子 作为局部变量-->

<bean id="userService" class="com.atguigu.spring5.service.UserService">
    <property name="userDao">
        <bean id="userService" class="com.atguigu.spring5.service.UserService">
    </property>
</bean>l
```

另外，在配置里面的属性如果是类的话，也还可以实现层级赋值

```xml
 <!--级联赋值-->
<bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="jams"></property>
        <property name="gender" value="男"></property>
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
        <property name="dept.dname" value="技术部门"></property>
</bean>
<bean id="dept" class="com.atguigu.spring5.bean.Dept">
</bean>
```

4. 注入集合

还是老样子，需要先设置 set方法，设置一个需要被注入的类

```java
//（1）创建类，定义数组、list、map、set 类型属性，生成对应 set 方法
public class Stu {
    //1 数组类型属性
    private String[] courses;
    //2 list集合类型属性
    private List<String> list;
    //3 map集合类型属性
    private Map<String,String> maps;
    //4 set集合类型属性
    private Set<String> sets;
  
    public void setSets(Set<String> sets) {
        this.sets = sets;
    }
    public void setCourses(String[] courses) {
        this.courses = courses;
    }
    public void setList(List<String> list) {
        this.list = list;
    }
    public void setMaps(Map<String, String> maps) {
        this.maps = maps;
    }
```

进行配置 关键词： map array list

```xml
<!--（2）在 spring 配置文件进行配置-->
    <bean id="stu" class="com.atguigu.spring5.collectiontype.Stu">
        <!--数组类型属性注入-->
        <property name="courses">
            <array>
                <value>java课程</value>
                <value>数据库课程</value>
            </array>
        </property>
        <!--list类型属性注入-->
        <property name="list">
            <list>
                <value>张三</value>
                <value>小三</value>
            </list>
        </property>
        <!--map类型属性注入-->
        <property name="maps">
            <map>
                <entry key="JAVA" value="java"></entry>
                <entry key="PHP" value="php"></entry>
            </map>
        </property>
        <!--set类型属性注入-->
        <property name="sets">
            <set>
                <value>MySQL</value>
                <value>Redis</value>
            </set>
        </property>
</bean>
```

### 工厂Bean

什么是工厂bean

普通 bean：在配置文件中定义 bean 类型就是返回类型

工厂 bean：在配置文件定义 bean 类型可以和返回类型不一样 第一步 创建类，让这个类作为工厂 bean，实现接口 FactoryBean 第二步 实现接口里面的方法，在实现的方法中定义返回的 bean 类型

基本使用

```xml
<bean id="myBean" class="com.atguigu.spring5.factorybean.MyBean">
</bean>
```

```java
public class MyBean implements FactoryBean<Course> {

    //定义返回bean
    @Override
    public Course getObject() throws Exception {
        Course course = new Course();
        course.setCname("abc");
        return course;
    }
}
```

```java
@Test
public void test3() {
// 使用的时候调用getBean 即可得到工厂类
 ApplicationContext context =
 new ClassPathXmlApplicationContext("bean3.xml");
//返回值类型可以不是定义的bean类型！
 Course course = context.getBean("myBean", Course.class);
 System.out.println(course);
}


```

### bean的生命周期

* 通过构造器创建 bean 实例（无参数构造）

  ```java
  // 这个是创建了 Spring实例
  ApplicationContext context =
  new ClassPathXmlApplicationContext("bean3.xml");
  // 就是这段话，实际上是调用了 其中bean的构造方法创建了实例
  Course course = context.getBean("myBean", Course.class);
  ```

* 为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）

* bean 的初始化的方法（需要进行配置初始化的方法）

* bean 可以使用了（对象获取到了）

* 当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）

* ```java
  Course course = context.getBean("myBean", Course.class);
   public class Orders {
           //无参数构造
           public Orders() {
               System.out.println("第一步 执行无参数构造创建 bean 实例");
           }
           private String oname;
           public void setOname(String oname) {
           this.oname = oname;
           System.out.println("第二步 调用 set 方法设置属性值");
           }
           //创建执行的初始化的方法
           public void initMethod() {
           System.out.println("第三步 执行初始化的方法");
           }
           //创建执行的销毁的方法
           public void destroyMethod() {
           System.out.println("第五步 执行销毁的方法");
           }
  }

  ```

除了以上的方法外，还可以调用一些不常见的生命周期，不过前提是要实现接口

```java
public class MyBeanPost implements BeanPostProcessor {//创建后置处理器实现类
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }
}
```

```xml
<!--配置文件的bean参数配置-->
<bean id="orders" class="com.atguigu.spring5.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">	<!--配置初始化方法和销毁方法-->
    <property name="oname" value="手机"></property><!--这里就是通过set方式（注入属性）赋值-->
</bean>

<!--配置后置处理器-->
<bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>
```

配置了后处理器的生命完整周期

​ （1）通过构造器创建 bean 实例（无参数构造）

​ （2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）

​ （3）把 bean 实例传递 bean 后置处理器的方法 postProcessBeforeInitialization

​ （4）调用 bean 的初始化的方法（需要进行配置初始化的方法）

​ （5）把 bean 实例传递 bean 后置处理器的方法 postProcessAfterInitialization

​ （6）bean 可以使用了（对象获取到了）

 （7）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）

### IOC 操作 Bean 管理(外部属性文件)

必要性

实际项目中极有可能是策划等人员将各类配置在外部，需要开发人员进行读取。

**直接配置数据库信息**

（1）配置Druid（德鲁伊）连接池 

（2）引入Druid（德鲁伊）连接池依赖 jar 包

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
</bean>
```

通过配置文件配置数据库信息


**配置数据库信息**

1. 创建外部属性文件，properties 格式文件，写数据库信息（ **jdbc.properties** ）

```xml
   prop.driverClass=com.mysql.jdbc.Driver
    prop.url=jdbc:mysql://localhost:3306/userDb
    prop.userName=root
    prop.password=root
```

2. 把外部 properties 属性文件引入到 spring 配置文件中 —— 引入 context 名称空间

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd"><!--引入context名称空间-->
  
        <!--引入外部属性文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClass}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.userName}"></property>
        <property name="password" value="${prop.password}"></property>
    </bean>
  
</beans>
```


## IOC容器注解方式管理

### 什么是注解

（1）注解是代码特殊标记，格式：@注解名称(属性名称=属性值, 属性名称=属性值…)

​ （2）使用注解，注解作用在类上面，方法上面，属性上面

​ （3）使用注解目的：简化 xml 配置

### Spring 针对 Bean 管理中创建对象提供注解

 下面四个注解功能是一样的，都可以用来**创建 bean 实例**

 （1）@Component

​ （2）@Service

​ （3）@Controller

​ （4）@Repository

### 基于注解方式实现对象创建

​ 第一步 引入依赖 （引入 **spring-aop jar包** ）

​ 第二步 开启组件扫描

```xml
<!--开启组件扫描
 1 如果扫描多个包，多个包使用逗号隔开
 2 扫描包上层目录
-->
<context:component-scan base-package="com.atguigu"></context:component-scan>
```

 第三步 创建类，在类上面添加创建对象注解

```java
//在注解里面 value 属性值可以省略不写，
//默认值是类名称，首字母小写
//UserService -- userService
//注解等同于XML配置文件：<bean id="userService" class=".."/>
@Component(value = "userService") 
public class UserService {
     public void add() {
     System.out.println("service add.......");
     }
}

```

### 开启组件扫描细节配置

这里配置了有关组件的筛选控制，可以排除哪些内容不进行扫描

```xml
<!--示例 1
 use-default-filters="false" 表示现在不使用默认 filter，自己配置 filter
 context:include-filter ，设置扫描哪些内容
-->
<context:component-scan base-package="com.atguigu" use-defaultfilters="false">
 <context:include-filter type="annotation"

expression="org.springframework.stereotype.Controller"/><!--代表只扫描Controller注解的类-->
</context:component-scan>
<!--示例 2
 下面配置扫描包所有内容
 context:exclude-filter： 设置哪些内容不进行扫描
-->
<context:component-scan base-package="com.atguigu">
 <context:exclude-filter type="annotation"

expression="org.springframework.stereotype.Controller"/><!--表示Controller注解的类之外一切都进行扫描-->
</context:component-scan>

```

### 基于注解方式实现属性注入

自动装配,只要外部bean配置有公共空间的类即可实现自动注入，前提是只有这一个类，无其他实现该类的接口或者子类。

如果这个类有接口，那么可以使用 @Qualifier 和 @Resource 进行注入。 前者根据类型，后者根据类型或者名称。但是后者不太推荐，因为这个是Java本身的包。

```java
//定义需要注入的类，并加上注入注解
@Service
public class UserService{
//定义一个对象属性
//需要加上自动装配的注解
@Autowired
@Qualifier(value = "userDaoImpl1") 
//@Resource(name = "userDaoImpl1")
private UserDao userDao;
}
//普通类型可以采用这种方式进行注入
@Value(value="abc")
private String name;
//定义一个实现上面引用的类

public class UserDaoImpl implements UserDao{
@override
public void add(){}
}
```

### 完全注解开发

（1）创建配置类，替代 xml 配置文件

```java
@Configuration //作为配置类，替代 xml 配置文件
//扫描这个包里面所有的内容
@ComponentScan(basePackages = {"com.atguigu"})
public class SpringConfig {

}
```

 （2）编写测试类

```java
@Test
public void testService2() {
 //加载配置类 ， 这里和以前不一样的是访问了类
 ApplicationContext context
//这一行和以前不一样
 = new AnnotationConfigApplicationContext(SpringConfig.class);
 UserService userService = context.getBean("userService",
UserService.class);
 System.out.println(userService);
 userService.add();
}

```

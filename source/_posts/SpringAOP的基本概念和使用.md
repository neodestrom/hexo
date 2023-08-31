
---
title: SpringAOP的基本概念和使用
date:  2023-08-30 16:21:08
categories:  [计算机,后端,Java,Spring,SpringAOP]
tags: article
index_img:
---

## AOP 基本概念

(1)面向切面编程（方面），利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得 业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
(2)通俗描述：**不通过修改源代码方式，在主干功能里面添加新功能**
​(3)使用登录例子说明 AOP 
![image.png](image-20220221095713-kavhejk.png)

## AOP（底层原理）

a）AOP 底层使用动态代理 ，动态代理有两种情况：

第一种 **有接口情况，使用 JDK 动态代理** ；

创建 **接口实现类代理对象** ，增强类的方法

这样如果想要新增内容

![image.png](image-20220221100038-ugxu7ow.png)

第二种 没有接口情况，使用  **CGLIB 动态代理** ；

创建 **子类的代理对象** ，增强类的方法

![image.png](image-20220221100229-42hgbwv.png)

## AOP（JDK 动态代理）

1）使用 JDK 动态代理，使用 Proxy 类里面的方法创建代理对象

```java
// 第一个是类加载器，增强方法
// 第二个是增强方法所在类的接口们
// 第三个是我们写的增强的类，实现上面的接口
public static Object newProxyInstance(ClassLoader loader,
Class<?>[] interface,InvocationHandler h)
```

 2）编写 JDK 动态代理代码

```java
// 模拟被增强类，被增强类的接口和增强的类
class Duck implements Fly,Yuck{
@override  
fly(){}
@override
yuck(){}
} 
interface Fly{
fly();
}
interface Yuck{
yuck();
}
class SpeakDuck implements InvocationHandler{

private Object obj;
public SpeackDuck(Object obj){
this.obj = obj;
}

@override  
//Method 指的是被增强的方法,后面的args就是它的参数
public Object invoke (Object proxy,Method method,Object[] args) 
throws Throwable{
//被增强的方法执行
method.invoke(obj,args);
//我们自定义的方法可以写在这下面，或者上面

}
@override
yuck(){
speak();
}
speak(){
System.out.println("I can speak")
}
}

//使用的时候
public class JDKproxy{
public static void main(Stringp[ args){
// 创建接口实现类的代理对象
Class[] interfaces = {Fly.class,Yuck.class};
Duck duck= new Duck;
Yuck speakingDuck =(Yuck)Proxy.newProxyInstance(
JDKProxy.class.getClassLoader(),interfaces,new SpeakDuck(duck));
// 下面这个对象就已经被增强了，可以随意调用其中接口的方法
speakingDuck.yuck();

}
}
```

## AOP（术语）

​ a）连接点：类里面哪些方法可以被增强，这些方法称为连接点

​ b）切入点：实际被真正增强的方法称为切入点

​ c）通知（增强）：实际增强的逻辑部分称为通知，且分为以下五种类型：

​ 1）前置通知 2）后置通知 3）环绕通知 4）异常通知 5）最终通知

 d）切面：把通知应用到切入点过程  
<br />

## AOP操作

​ a）Spring 框架一般都是基于 AspectJ 实现 AOP 操作，AspectJ 不是 Spring 组成部分，独立 AOP 框架，一般把 AspectJ 和 Spirng 框架一起使 用，进行 AOP 操作

​ b）基于 AspectJ 实现 AOP 操作：1）基于 xml 配置文件实现 （2）基于注解方式实现（使用）

​ c）引入相关jar包

​ d）切入点表达式，如下：

```java
（1）切入点表达式作用：知道对哪个类里面的哪个方法进行增强 
（2）语法结构： execution([权限修饰符] [返回类型] 
[类全路径] [方法名称]([参数列表]) )
（3）例子如下：
    例 1：对 com.atguigu.dao.BookDao 类里面的 add 进行增强
		execution(* com.atguigu.dao.BookDao.add(..))
 	例 2：对 com.atguigu.dao.BookDao 类里面的所有的方法进行增强
		execution(* com.atguigu.dao.BookDao.* (..))
    例 3：对 com.atguigu.dao 包里面所有类，类里面所有方法进行增强
		execution(* com.atguigu.dao.*.* (..))
```


## AOP 操作（AspectJ 注解）

```java
// 创建一个类
public class User {
 public void add(){
  
 }
//增强类
public class UserProxy{
 public void before(){}
}

}

```

```xml
<!--3、进行通知的配置-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.atguigu.spring5.aopanno"></context:component-scan>

    <!-- 开启Aspect生成代理对象-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

```java
//4、配置不同类型的通知
@Component
@Aspect  //生成代理对象
public class UserProxy {
      //相同切入点抽取
    @Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void pointdemo() {

    }

    //前置通知
    //@Before注解表示作为前置通知
    @Before(value = "pointdemo()")//相同切入点抽取使用！
    public void before() {
        System.out.println("before.........");
    }

    //后置通知（返回通知）
    @AfterReturning(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterReturning() {
        System.out.println("afterReturning.........");
    }

    //最终通知
    @After(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void after() {
        System.out.println("after.........");
    }

    //异常通知
    @AfterThrowing(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterThrowing() {
        System.out.println("afterThrowing.........");
    }

    //环绕通知
    @Around(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前.........");

        //被增强的方法执行
        proceedingJoinPoint.proceed();

        System.out.println("环绕之后.........");
    }
}
```

```java
//（1）在增强类上面添加注解 @Order(数字类型值)，数字类型值越小优先级越高
@Component
@Aspect
@Order(1)
public class PersonProxy{ }

```

## AOP 操作（AspectJ 配置文件）

```java
<!--1、创建两个类，增强类和被增强类，创建方法（同上一样）-->
<!--2、在 spring 配置文件中创建两个类对象-->
<!--创建对象-->
<bean id="book" class="com.atguigu.spring5.aopxml.Book"></bean>
<bean id="bookProxy" class="com.atguigu.spring5.aopxml.BookProxy"></bean>
<!--3、在 spring 配置文件中配置切入点-->
<!--配置 aop 增强-->
<aop:config>
 <!--切入点-->
 <aop:pointcut id="p" expression="execution(* com.atguigu.spring5.aopxml.Book.buy(..))"/>
 <!--配置切面-->
 <aop:aspect ref="bookProxy">
 <!--增强作用在具体的方法上-->
 <aop:before method="before" pointcut-ref="p"/>
 </aop:aspect>
</aop:config>
```

## 总结

1.Aop注解的方式  
AOP注入依赖的这个AspectJ只是一种AOP的实现方式，以后可能会有其他的实现相似的功能。其核心还是开启注解，开启AOP，之后只需标识增强方法和被增强的方法即可。  
2.Aop配置文件方式  
创建增强类和被增强类，之后创建AopConfig。  
配置切入点----被增强类需要增强的方法  
配置切面  
 配置切面引用为增强类  
 配置各类方法，并加上其切入点引用

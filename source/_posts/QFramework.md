
---
title: QFramework
date:  2022-09-07 10:19:57
categories:  [Game,框架分析,QFramework]
tags: article
---

## QFramework
>
> QFramework 是一款轻量级游戏框架，其主要思想 划分系统架构，并通过事件在这几层架构中进行通信。
> 


## 架构

QFramework系统设计架构分为四层及其规则：  
1、表现层：ViewController层。IController接口，负责接收输入和状态变化时的表现，一般情况下，MonoBehaviour均为表现层  
2、系统层：System层。ISystem接口，帮助IController承担一部分逻辑，在多个表现层共享的逻辑，比如计时系统、商城系统、成就系统等  
3、数据层：Model层。IModel接口，负责数据的定义、数据的增删查改方法的提供  
4、工具层：Utility层。IUtility接口，负责提供基础设施，比如存储方法、序列化方法、网络连接方法、蓝牙方法、SDK、框架继承等。啥都干不了，可以集成第三方库，或者封装API

前三层均 实现 **IBelongToArchitecture** 接口，该接口能够获取到 前三层的对象，也可注册前三层的对象。 第四层比较特殊，是留给外部API进行交互的，需要前面三层得实现 GetUtility方法

## 重要概念
前面说了，通过事件可以进行各个层之间的交互。事件的注册是注册在全局的，可以在前三层任意地方进行触发，触发后执行的方法是一个Command ， Command 是一个重要的执行逻辑载体，该类也继承 **IBelongToArchitecture** 这意味着也可以随时访问到全局的各层方法。

## 总结
这个框架轻量执行逻辑很清晰，不过由于所有的层的通信需要通过 Get+层+具体的类 来获取，实际上赋给每一个层级类的权限过大了，层类过多这个调用逻辑就会显得很复杂，但是这也是优点，类和类之间的调用就更为方便代码的利用率更高，如果能够规范好各类调用，其实还是很不错的。
例如做一个排行榜功能，排行版面板需要请求网络数据得到数据列表，随后根据这个列表数据进行渲染。这个过程就用到`GetSystem<NetSystem>` 用于网络请求， 并涉及到 `Model` 进行本地储存，并将raw数据处理成列表能够渲染的数据，随后在页面中就可以使用该数据 利用插件 `GetUtility<SuperScrollView>` 进行渲染。 渲染到子视图的每个排名玩家，可能存在一些触发，这些触发将会变成Command供 父类进行调用。
另一个缺点就是会写大量的用于标识接口的方法，例如Command 和 Event Model等等。一旦进行某些数据的修改，那么就会到很多模块中进行修改。
该框架的轻量级即是优点也是缺点，轻量也就意味着其余的很多模块需要自己完善或者是去下扩展包，例如资源的存储，资源的打包，网络通信，热更等一系列的方案，要实现一套适配于该框架的游戏框架解决方案的成本会显得很高。



















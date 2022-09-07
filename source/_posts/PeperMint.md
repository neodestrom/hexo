---
title: PeperMint
date:  2022-09-07 10:19:35
categories:  [Game,框架分析,PeperMint]
tags: article
---

### 简介
pepermint 是一款Unity的UGUI数据绑定框架，这款插件采用了[[MVVM模式]]，实现了数据和视图的绑定。
这款插件具有以下特点：
1.  简洁的代码
Peppermint 数据绑定基于属性和反射。该类不需要从指定的类或接口继承，任何具有属性的对象都可以用作绑定源或绑定目标。现有代码只需进行最少的更改即可支持数据绑定
要检测源更改，源必须实现 INotifyPropertyChanged 接口，或者继承 Bindable 基类
2.  易于设置
使 UI 支持数据绑定非常容易。您只需要添加三种类型的组件：DataContext、DataContextRegister 和 Binder。大多数组件只有很少的参数需要设置
-   内置的绑定包括所有 UGUI 组件的绑定，ImageBinder、AnimatorBinder、CustomBinder、Selector、Setter、Getter 等。您可以轻松创建自己的绑定类来支持新功能
3.  Model-View-ViewModel 模式
Peppermint 数据绑定旨在简化使用 MVVM 模式构建游戏 UI。应用程序逻辑和 UI 之间的清晰分离将使您的游戏更容易测试、维护和扩展
4.  性能
广泛优化的 C# 代码，例如类型缓存、对象池、自定义事件、快速委托等。
### 使用
跟其他的MVVM框架一样， pepermint 的使用也是将 视图 和 逻辑分开的，因此需要对视图和逻辑分别进行设置。
#### 视图操作
视图需要绑定一个[[数据源]] ，绑定方法是在视图的根组件上添加 `DataRegister`  和 `DataContext` 组件，其中DataRegister中需要填写数据源的名称，该名称将会在Source中进行定义。
绑定了数据源后，即可使用数据源中的数据内容。对于基本的数据内容，pepermint提供了 `TextBinder` 用于绑定文本，`ButtonBinder` 用于绑定按钮的点击事件， `imageBinder` 用于绑定Image Sprite等等。


#### 视图模型操作
pepermint中的VM 又称为 Source，为方便起见后文将用Source指代它。 Source中定义一个响应数据的基本格式如下：
```c#
// 需要定义一个全局的私有变量以及一个公共的变量
private ICommand setClickCommand;
public ICommand SetClickCommand  
{  
    get => setCategoryCountryCommand;  
    // ref 指向私有变量，第三个参数为 UGUI上Binder 绑定这个数据的依据。
    set { SetProperty(ref setClickCommand, value, "SetClickCommand"); }  
}

private String text;
public String Text  
{  
    get => text;  
    set { SetProperty(ref text, value, "Text"); }  
}

```
Source 的初始化一般就包括上面数据的一个绑定，以及下面将其添加到BindingManager中。
```C#
	// 这里第一个为当前source 对象，第二个为视图ContextRegister的依据
	BindingManager.Instance.AddSource(this, "SourceName");
```



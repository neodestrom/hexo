---
title: mvc
date:  2022-07-19 08:54:07
categories:  [杂记]
tags: article
---


### 简述
开发 Unity UI的时候， 发现了一套范式。

1. 普通UI的内容替换
普通UI的替换只需要将UI的组件绑定到一个脚本上去，由脚本进行统一的处理。例如需要更改文字内容
```C#
class UIPage : MonoBehaviour{
	[SerilizeField]private Text text;
	public void SetText(string txt){
		text.text = txt;
	}
}
```
通过这种MonoBehavior的方式控制UI有一个好处就是视图逻辑 和 游戏逻辑可以解耦。 其他脚本可以通过得到该对象从而控制UI。
```c#
class xxSystem : MonoBehavior{
	UIPage page = GetComponent<UIPage>;
	page.SetText("123");
	
}

```
其他脚本有多种方式可以获取到UI的控制权，除了通过脚本挂载同一游戏对象外，也可以通过 单例的模式将 UI组件实例拿到，也可以通过在其他组件上挂载这个UI组件，总之办法很多，结果都是参与到对该UI的控制。

2. 集合UI的内容替换
集合UI指的是大数据内容，例如滚动条一类。有一个superscroll插件可以很好处理大数据的内容。集合UI的分UI交互：
父传子:
+ 通过集合遍历生成单个对象，通过对象对子类进行操作。
子传父:
+ 这类通常是子类事件的触发，需要传递数据到父亲上面，这类通常使用一个 Action 来处理， 在Js中也叫做 callbackFunction ，通过一个方法获取到父类的作用域进而对父类进行特定的操作。 因此对于这类方法需要保留一个 Action变量进行存储

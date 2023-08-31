---
title: Unitask
date:  2023-08-27 13:39:34
categories:  [计算机,Game,框架源码分析,Unitask]
tags: article
index_img:
---


## 概述
Unitask是一个 Unity异步解决方案的代码框架库。
Unity本身是提供了一个协程解决方案。通过创建一个IEnumerator 的生成器方法，并且在其中使用 yield return 来中断程序运行达到异步执行的效果。
但是这个方案无法得到异步的返回值，也无法处理错误信息。并且其还需要限制在Unity的MonoBehavior中执行，编写上也不便，需要写一个IEnumerator方法。很难做取消过程，无法控制多个写协程的并行处理等等。相较而言，Task这类异步解决方案则可以解决上面的问题。
C#提供了一个Task异步解决方案，不过它在创建一个异步方法的时候，会同步开启一个线程，但是由于Unity本身是单线程这就会导致访问堆栈的问题，以及debug等诸多不便。另外Task本身在unity里面执行的时候，会产生很多的性能消耗。
因此Unitask诞生就是结合了Task编写方式和Unity单线程的特性，使得在Unity中能够编写高性能，容易编写的异步代码。
## 如何使用UniTask
## 如何转化原有AsyncOperation，交付给Unitask处理
例子
```c#
// 加载场景可以的异步可以得到进度值，并且输出。
await SceneManager.LoadSceneAsync(address).ToUniTask(
	(Progress.Create<float>(
		(p)=>
		{
			LoadSceneSlider.value=p;
			ProgressText.text = $"progress : {p*100}%"
		}
	))
);
```





---
title: Profiler
date:  2023-08-27 13:39:10
categories:  [计算机,Game,框架源码分析,Profiler]
tags: article
index_img:
---



Profiler是Unity自带的性能分析工具。可以查看CPU，GPU，物理，UI等等详细位置的消耗。
日常使用的时候，最常使用的是CPU分析工具，通过这个可以查看代码的执行速度,执行次数，执行所产生的GC等等。

常见的查看具体方法模式有三个，最常使用的是hierarchy视图，可以查看什么方法执行了，方法的子方法的执行等等，timeline有时也会使用，可以查看job的内容。profiler 还有deepProfiler模式，开启后可以看到详细的方法调用。

具体来说，想要查看性能上最好性能的就是查看playerloop方法下的自己写的代码，按照耗时来排序。耗时的代码分为自己消耗的以及子方法消耗的，但是自己消耗的代码如果过多想要查看具体的代码，那么需要这样。

```c#
// 包裹开始，命名为括号内的
Profiler.BeginSample("test");
for(int i=0;i<10000;i++){
	int y = i;
}
Profiler.EndSample();

```

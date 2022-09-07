---
title: Ilruntime
date:  2022-09-07 10:19:23
categories:  [Game,框架分析,IlRuntime]
tags: article
---

## 简介
Ilruntime 是一款热更框架，用于产品打包上架后，能够通过拉取新的资源包和代码实现无需下载安装包而在程序上直接更新的功能。

## 底层部分原理
ILRuntime 内部集成了一个解释器系统，多亏于一个开源项目 Mono.cecil 项目，ILRuntime 可以读取解释由C#语言编译的IL的汇编指令，通过解释执行这些指令，实现 外部代码的执行。


## 流程
### 下载对应的 dll文件
下载可以用不同的网络工具 甚至可以用本地流进行加载，最终需要将dll转成2进制并且存在
最后 调用ILRuntime 的LoadAssembly 方法即可
``` c#
byte[] dll = res.bytes;
// pdb 文件是调试数据库，可以在日志中显示报错的行号
byte[] pdb = res1.bytes;

MemoryStream fs = new MemoryStream(dll)
MemoryStream p = new MemoryStream(pdb)

appdomain.LoadAssembly(fs, p, new ILRuntime.Mono.Cecil.Pdb.PdbReaderProvider());

```
关键在 从MONO中加载模块这里， 内部采用了寄存器解释代码，此处不在加以深入。
```c#
public void LoadAssembly(System.IO.Stream stream, System.IO.Stream symbol, ISymbolReaderProvider symbolReader){
	var module = ModuleDefinition.ReadModule(stream); //从MONO中加载模块
	...
}
```
#### 常见的下载dll文件方案
通过addressable的热更方案，将dll文件放在热更服务器上，通过addressable 检查更新
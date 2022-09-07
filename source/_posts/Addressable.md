
---
title: Addressable
date:  2022-09-07 10:19:04
categories:  [Game,框架分析,Addressable]
tags: article
---


## Addressable

>
> Addressable 是一个很好用的资源管理框架。
> 比起之前的资源管理，它提供了在哪儿都可以引用到资源的功能，甚至能够引用到网络上的资源进行热更新。
> 

### Addressable的使用
1. 创建资源组
	1. 在addressable 的菜单下打开group，并且创建setting asset文件。
	2. 随后可以在该group中 创建自己的group， 每一个group 相当于一个ab包
2. 资源加入资源组
	1. 资源可以直接拖进某个group组中。
3. 打包资源
	1. 打包资源分为本地的打包和远程的打包，设置在该资源组的 assets 文件中，设置成remote即远程打包，local则为本地打包。
	2. 本地打包的资源在 libary/com.addressable/aa/ 文件中， 远程打包则在项目根目录下的 ServerData目录下
	3. 打包还可以打补丁包，通常为远程打包而准备，这个时候需要设置 addressableSettings.asset 文件勾选上 build remote catalog， 勾选后则会创建一个更新列表文件，用于检查热更的。
4. 加载资源
	1. 加载资源可以调用 LoadAsync 方法, 该方法返回一个AsyncHandler，通常需要使用UniTask进行封装随后等待其完成。路径是 group的名字加上文件名，另外，文件名需要改成简称，否则就需要加很多前缀上去。
	2. 对于预制体文件，还提供了直接生成预制体的方法。
	3. 因为加载资源并不分是网络资源还是本地资源，如果是网络资源则会到服务器进行哈希比对并进行资源更新。

### 代码层面的上述步骤：
1. 创建资源组
```c#
// 如果没有settings且参数为true 文件则进行创建
var settings = AddressableAssetSettingsDefaultObject.GetSettings(true)
// 内部实现：
if (AddressableAssetSettingsDefaultObject.Settings == null)  
{  
	AddressableAssetSettingsDefaultObject.Settings = AddressableAssetSettings.Create(
		AddressableAssetSettingsDefaultObject.kDefaultConfigFolder,
		AddressableAssetSettingsDefaultObject.kDefaultConfigAssetName,
		true,
		true);  
	return AddressableAssetSettingsDefaultObject.Settings;
}
//参数说明 组名，设置默认组，只可读，发送修改事件，用于复制的schema原型 ，类型
// 默认情况下像下面这样即可创建组。
AddressableAssetGroupTemplate groupTemplate = settings.GetGroupTemplateObject(0) as AddressableAssetGroupTemplate;
var cus_group = settings.CreateGroup("groupName",
									 false,
									 false,
									 false,
									 null,
									 groupTemplate.getTypes()
									 )
```

2. 资源加入资源组
```c#
var settings = AddressableAssetSettingsDefaultObject.GetSettings(true);
var cus_group = settings.FindGroup("groupName");
string guid = AssetDatabase.AssetPathToGUID("assetPath");
// 这个entity 代表的就是一个group里面的某个 资源引用
var entity = settings.CreateOrMoveEntry(guid,cus_group);
// 组里面设置路径地址 和 Label
entity.SetAddress("assetPath");
entity.SetLabel("LabelName",true,true);

```

3. 进行打包
```c#
// 根据平台上设置的 playmode进行打包 相当于 build default 那个按钮
AddressableAssetSettings.BuildPlayerContent();
```



## Addressable 的原理
addressable 作为一个资源加载框架，最外层 Adressable 类是一个中介类，提供公共方法给用户使用，包括一些加载资源的方法等。
前期准备核心原理集中在 AddressableAssetSettings 里面，该类负责前期的资源组创建。
使用的核心原理集中在Addressables 的Load方法里面，使用前面创建的信息，合理加载到各类资源。 [[Addressable 的加载资源原理]]











---
title: Addressable 的加载资源原理
date:  2023-08-31 08:55:05
categories:  [计算机,Game,框架源码分析,Addressable,系统]
tags: article
index_img:
---


### 概要
Addressable 的加载主要由Manager类，Provider类和AsynchOperation负责。
Addressable 开放给开发者的LoadAssetsAsync方法执行后，先是执行了Manager类的方法，在里面先定位了资源的位置信息，随后分发创建具体的Provider类，由Provider类根据资源的类型再创建具体的asyncOperation类创建异步操作，随之完成加载。

### 具体流程
Addressable  是一个外观类，负责调用各类接口类实现实际操作，其中 AddressableImpl 负责了资源的调度。
资源的加载过程如下
```csharp

public AsyncOperationHandle<TObject> LoadAssetAsync<TObject>(IResourceLocation location)
{
    return TrackHandle(ResourceManager.ProvideResource<TObject>(location));
}
```

资源加载的核心就在这个 ProvideResource方法中。

```csharp
private AsyncOperationHandle ProvideResource(IResourceLocation location, Type desiredType = null)
{
    ...
    // ### 获得ResourceProvider
    IResourceProvider provider = null;
    if (desiredType == null)
    {
        provider = GetResourceProvider(desiredType, location);
        ...
        desiredType = provider.GetDefaultType(location);
    }

    // ###  获得AsyncOperation
    IAsyncOperation op;
    int hash = location.Hash(desiredType);
    if (m_AssetOperationCache.TryGetValue(hash, out op))
    {
        op.IncrementReferenceCount();
        return new AsyncOperationHandle(op);
    }

    // ###  获得desiredType对应的AsyncOperation类型
    Type provType;
    if (!m_ProviderOperationTypeCache.TryGetValue(desiredType, out provType))
        m_ProviderOperationTypeCache.Add(desiredType, provType = typeof(ProviderOperation<>).MakeGenericType(new Type[] { desiredType }));

    // ### 创建AsyncOperation  这个op的类型是 ProviderOperation 
    op = CreateOperation<IAsyncOperation>(provType, provType.GetHashCode(), hash, m_ReleaseOpCached);

    // ### 创建依赖项的AsyncOperation
    int depHash = location.DependencyHashCode;
    var depOp = location.HasDependencies ? ProvideResourceGroupCached(location.Dependencies, depHash, null, null) : default(AsyncOperationHandle<IList<AsyncOperationHandle>>);
    if (provider == null)
        provider = GetResourceProvider(desiredType, location);

    // ### 初始化AsyncOperation 整合刚才的操作步骤资源。 诸如当前资源的加载，依赖资源的加载。
    ((IGenericProviderOperation)op).Init(this, provider, location, depOp);

    // ### 开始异步加载 最终会执行 的方法是 IResourceProvider 的Provide方法， 并且通过AysncOperation
    // 这个中介来向外暴露拉取情况
    var handle = StartOperation(op, depOp);
    ...
    return handle;
}
```
步骤如下：
1. 根据路径信息获取到路径的类型，并得到相应的Provider，Provider 是专门负责某一类资源的加载，负责直接对接 AssetsBundle 或是其他的资源类。
2. 生成 AsyncOperation 类，该类用于通知当前资源的加载情况。
3. 加载当前路径下依赖的其他的资源
4. 将以上的内容整合进一个 ProviderOperation类中，并由它分发给具体的IResource 进行执行，根据情况调用 asyncOperation 的结果callback等。

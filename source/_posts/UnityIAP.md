---
title: UnityIAP
date:  2022-07-28 14:38:30
categories:  [Game,框架分析,UnityIAP]
tags: article
---

## 简介
Unity IAP系统是一个内购插件，由Unity官方开发，目前支持Google,Amazon,IOS和Unity Distribution Portal 平台。平台的切换无需额外编写代码，只需要在编辑器窗口上选择切换的平台即可。
## 使用

开发者关注的类对象是一个实现`IStoreListener`接口的自定义对象。
该对象能够接受内购按钮的事件，并能够调用内购的方法，主要是购买，restore，查询等。相当于中心管理内购的类。
该对象使用的一般流程如下：
```c#
public class MyIapStoreListener:IStoreListener {
	public MyIapStoreListener(){
		// 必须先初始化
		
		//这个是默认的模块，规定了当前平台等信息
		StandardPurchasingModule module = StandardPurchasingModule.Instance();
		//这个提供了部分供开发者使用的公共配置，并保留上面的模块
		ConfigurationBuilder builder = ConfigurationBuilder.Instance(module);
		// 调用UnityPurchasing 初始化方法进行初始化，传入builder 和 当前的listener
		UnityPurchasing.Initialize(instance, builder);
	}
	// 下面四个方法是覆盖的IstoreListener的方法，主要是初始化的回调。以及购买的回调。
	void OnInitializeFailed(InitializationFailureReason error);
	PurchaseProcessingResult ProcessPurchase(PurchaseEventArgs purchaseEvent){
		//购买行为完毕后，会出现的回调情况
		PurchaseProcessingResult result;  
		// if any receiver consumed this purchase we return the status  
		bool consumePurchase = false;  
		bool resultProcessed = false;  
		foreach (IAPButtonEx button in activeButtons)  
		{  
		    if (button.productId == e.purchasedProduct.definition.id)  
		    {        
			    result = button.ProcessPurchase(e);  
		        if (result == PurchaseProcessingResult.Complete)  
		        {
					consumePurchase = true;  
		        }  
		        resultProcessed = true;  
		    }
		}  
		foreach (IAPListener listener in activeListeners)  
		{  
		    result = listener.ProcessPurchase(e);  
		    if (result == PurchaseProcessingResult.Complete)  
		    {        
			    consumePurchase = true;  
		    }  
		    resultProcessed = true;  
		}  
		return (consumePurchase) ? PurchaseProcessingResult.Complete : PurchaseProcessingResult.Pending;
	}
	void OnPurchaseFailed(Product product, PurchaseFailureReason failureReason);
	void OnInitialized(IStoreController controller, IExtensionProvider extensions);
}
```
如果购买成功或者购买失败了，可以通过该类将事件分发至具体的某个Item上，执行其具体的行为。

## 源码解析

### 流程解析
1. 生成module
	 默认生成的module是标准module，同时也可以生成自己创建的module。 module 可以用来注册商店，生成一些IAP的默认的一些配置等等。
2. 构建ConfigurationBuilder(module)
	该Builder 接收上面生成的module，作为其中的factory实例，该类还负责一些其他的公共配置，例如是否使用本地的商品列表。
3. 进行初始化Purchasing Manager
	初始化会将刚刚生成的builder 实例拿来创建一个 PurchaseManager实例， 创建该实例的时候会将其中的product 等信息传入到Manager中去， PurchaseManager 也是负责直接调用 各类商店（谷歌商店，Appstore等）方法的类，同时也是直接接收这些商店传回回调的类。
```c#
// IStoreListener 中执行将本身和builder 传入
UnityPurchasing.Initialize(instance, builder);

// UnityPurchasing
internal static void Initialize(IStoreListener listener, ConfigurationBuilder builder,  
         ILogger logger, string persistentDatapath, IUnityAnalytics analytics, ICatalogProvider catalog)  
     {  
        var transactionLog = new TransactionLog(logger, persistentDatapath);  
        // 生成 PurchaseingManager
		var manager = new PurchasingManager(transactionLog, logger, builder.factory.service, builder.factory.storeName);  
         var analyticsReporter = new AnalyticsReporter(analytics);  
  
         // Proxy the PurchasingManager's callback interface to forward Transactions to Analytics.  
         var proxy = new StoreListenerProxy(listener, analyticsReporter, builder.factory);  
			FetchAndMergeProducts(builder.useCatalogProvider, builder.products, catalog, response =>  
             {  
                 manager.Initialize(proxy, response);  
             });     }
```
4. 初始化StoreListener的方法
	开发者通过StoreListener 间接控制购买流程，因此需要将PurchasingManager的结果反馈和操作方法都提供给 StoreListener
```c#
// PurchasingManager
// 如果其他的流程都没出问题 则将本身传到StoreListener中去
m_Listener.OnInitialized(this);

// StoreListener

public void OnInitialized(IStoreController controller, IExtensionProvider extensions)  
{  
    initializationComplete = true;  
	//通过该对象即可调用 购买等逻辑
	this.controller = controller;  
}
// 购买
public void InitiatePurchase(string productID){
	controller.InitiatePurchase(productID);
}
//处理购买逻辑
public PurchaseProcessingResult ProcessPurchase(PurchaseEventArgs e)  
{
	
}

```

### 自定义的商店
通过构建 自定义Module 则可以通过module选择自定义的商店。

1. 构建自定义Module
```c#
public class MyCusPurchasingModule : AbstractPurchasingModule  
{  
    private static MyCusPurchasingModule _instance;  
    public static MyCusPurchasingModule Instance  
    {  
        get  
        {  
            if (_instance == null)  
            {                
		        _instance = new MyCusPurchasingModule();  
		        
            }            
            return _instance;  
        }    
    }
    // 需要覆盖该方法，对商店进行注册
    public override void Configure()  
    {        
	    RegisterStore("AppGallery", InstantiateMyCusStore());  
    }  
    private IStore InstantiateMyCusStore()  
    {        
	    if (Application.platform == RuntimePlatform.Android)  
	        {            
		        return new MyCusStore();  
	        }        
	        return null;  
	    }
    }

```
2. 构建自定义的商店
```c#
public class MyStore : IStore{
	private IStoreCallback callback; 
	public void Initialize (IStoreCallback callback) { 
	this.callback = callback; 
	} 
	public void RetrieveProducts (ReadOnlyCollection<ProductDefinition> products) { 
		// 获取商品信息 的元数据 以及 所有权状态
		// 调用PurcasingManager中的 OnRetrieve 方法。而只有调用了这个方法，实现IStoreListener
		// 的类才能初始化成功
		callback.OnProductsRetrieved(); 
		// 如果不想要调用该方法则调用
		callback.OnSetupFailed(InitializationFailureReason)
	} 
	public void Purchase (ProductDefinition product, string developerPayload) { 
	// 启动购买流程并调用 购买成功或者失败的回调
		callback.OnPurchaseSucceeded() 
		callback.OnPurchaseFailed() 
	} 
	public void FinishTransaction (UnityEngine.Purchasing.ProductDefinition product, string transactionId) { 
	// 执行与交易相关的内务处理 
	
	}
}
```
3. 使用方法
 通过创建自定义的Module 和 Istore ，初始化内购系统。
 就可以通过后续的controller 来简介控制自定义的store方法。
```c# 
class MyStoreListener : IStoreListener{

	public MyStoreListener(){
		//这个是默认的模块，规定了当前平台等信息
		StandardPurchasingModule module = StandardPurchasingModule.Instance();
		// 自定义module
		MyCusPurchasingModule cusModule = MyCusPurchasingModule.Instance;
		// 这里注册的时候可以传入多个module，自定义module 放在前面覆盖后面的配置。
		ConfigurationBuilder builder = ConfigurationBuilder.Instance(cusModule,module);
		// 调用UnityPurchasing 初始化方法进行初始化，传入builder 和 当前的listener
		UnityPurchasing.Initialize(instance, builder);	
	}
}



```
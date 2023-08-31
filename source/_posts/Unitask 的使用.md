---
title: Unitask 的使用
date:  2023-08-27 13:39:43
categories:  [计算机,Game,框架源码分析,Unitask]
tags: article
index_img:
---



### Token的使用方法
#### UniTaskCompletionSource
`UniTaskCompletionSource<T>`可被实例化传入泛型，泛型可作为结果通过Task属性返回。如
```c#
public static UniTask<T> LoadAssetsAsync<T>(string path){
	var utcs = new UniTaskCompletionSource<T>();
	LoadAsync<T>(path,(res)=>{
		// 同等情况下还可以执行 utcs.TrySetCanceled(token)   utcs.TrySetException(new SystemException) 手动实现失败或者取消
		utcs.TrySetResult(res) // 这个情况下 utcs.Complete = true 
		return utcs.Task;
	})
	return utcs.Task;
}
```
这种utcs最好的应用场景就是封装原有的 回调式的异步方法，使之能够await，并且使用Unitask生态带来的Unitask操作。这个token是可以复用的
#### CancellationTokenSource
cancellation可以用于任务的取消
```c#
// token的创建
var firstCancelToken = new CancellationTokenSource()
var secondCancelToken = new CancellationTokenSource()
var linkedCancelToken = CancellationTokenSource.CreateLinkedTokenSource(firstCancelToken,secondCancelToken)//其中一个取消了就取消。

//token 加在普通asyncOperation操作上
UnityWebRequest.Get(url).SendWebRequest().WithCancellation(secondCancelToken.Token)

 
// token的取消回调
try{
	await LoadAssetAsync()
}catch(OperationCanceledException e){
	// 这里是取消回调
}
// 第二种方法更常见用这样的 如果后面的await 有返回值，那么则位于元组的第二个参数。否则就直接返回一个布尔值
var (cancelled,_) = await LoadAssetAsync().SuppressCancellationThrow();
if(cancelled){...}

// token取消 取消后必须进行销毁操作，否则就不能再次使用这个token；
firstCancelToken.Cancel();
firstCancelToken.Despose();
firstCancelToken = new CanclelationTokenSource() 
```

`cts.CancelAfterSlim(TimeSpan.FromSeconds(5f))` 5s后触发取消操作

### 延时操作
`Unitask.Delay(TimeSpan.FromSeconds(0.1f))`  等待0.1s
`Unitask.Delay(TimeSpan.FromSeconds(0.1f，true))`  忽略timescale
`Unitask.DelayFrame(5)`  延时5帧
`Unitask.NextFrame()`  下一帧的Update之后执行 yield return null
`Unitask.WaitEndofFrame()` 这一帧结束下一帧开始之前执行 相当于 yield return endofFrame
`Unitask.Yeild(timeloop)` 这个可以自定义playerloop之后执行 这一个效率最高

### UniTask等待条件触发
`UniTask.WaitUntil(()=>somethingOk); `等待值为True
`UniTask.WaitUntilValueChanged();` 等待值改变
### 多个Unitask操作
`WhenAll()` 等待全部完成
`WhenAny()` 等待一个完成
### 同步方法中执行异步方法
SomeAsyncMethod().Forget()
### 使用Unitask执行线程
```c#
// 这两行需要同时调用
await UniTask.RunOnThreadPool(()=>{})
await UniTask.SwitchToMainThread();
// 第二种方法
await UniTask.SwitchToThreadPool();
await File.ReadAllTextAsync(filename)
await UniTask.Yield(PlayerLoopTiming.Update)
```

### Unitask的 IUniTaskAsyncEnumerable 对象的用法

很多UI的对象可以转化为这类对象，例如 var asyncEnumerable = Button.OnClickAsAsyncEnumerable();
这会使得每一次点击都会向 asyncEnumerable中添加一次内容。使用场景可以如下：
```c#
// 实现点击一个物体，对具体的点击次数进行判断
var asyncEnumerable = Button.OnClickAsAsyncEnumerable()
// 这个是ForEachAsync 是同步事件 
await asyncEnumerable.Take(3).ForEachAsync((_,index)=>{
	if(index==0)
	if(index==2)
},token) // 这里有一个取消token

	// 异步可迭代器可以视作对事件的回调，当这里面实现了异步操作的时候，外部的所有事件将不会进行传递。
	// 如果想要继续接收事件，那么需要这样写 asyncEnumerable.Queue().ForEachAwaitAsync 
	// 对事件进行一个排队
await asyncEnumerable.ForEachAwaitAsync(async (_)=>{

	await UniTask.Delay(TimeSpan.FromSeconds(2f),cancellationToken: token)

})

```

实质上异步可迭代器可以将其范围扩大，无论是按钮还是自定义触发的事件都可以使用它。可以很方便的实现诸如上面的对具体触发次数进行判断，或者判断双击，对点击进行限制等等操作。所以除了按钮事件，Unity上还有很多事件例如碰撞等，都可以转化为碰撞可迭代器。

### UniTask的异步流编写方式
```c#
// 创建
private AsyncReactiveProperty<int> currentHp = new AsyncReactiveProperty<int>(100);
public Text ShowHpText;


//第一种使用事件订阅, 每当数值发生变化则会触发 OnHpChange方法
currentHp.Subscribe(OnHpChange)
// WithoutCurrent 指忽略当前值
currentHp.WithoutCurrent().ForEachAsync((_,index)=>{}) // 天生支持转化为异步可迭代器
currentHp.FirstAsync((value)=>{}) //可以进行一次查询，相当于第一次满足的Select
currentHp.BindTo(ShowHpText); // 可以绑定数值到一个Unity组件上

```
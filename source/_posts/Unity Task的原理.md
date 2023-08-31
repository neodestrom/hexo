---
title: Unity Task的原理
date:  2023-08-27 13:39:46
categories:  [计算机,Game,框架源码分析,Unitask]
tags: article
index_img:
---

## 为什么有些原生的c# 异步方法使用await后可以被转化为Unitask而不是Task
UniTask 内部实现了一些扩展方法，能够将原有的C#异步的方法转化为Unitask。这是因为Unitask对于一些特定的继承自AysncOperation的类进行了扩展，然后实现了它们内部的GetAwaiter的方法，返回的是一个自定义的Awaiter struct。而这个Awaiter就可以高效率地使用UniTask的内部任务分配机制。


## UniTask的 0GC原理
原始使用async 关键词所发生的事情：
```c#
Task<object> LoadAssetAsync(string address){...}

// 使用该方法
var cache = await LoadAssetAsync(address)
```
上面使用await关键字的代码会被编译为下面的代码
```c#
var awaiter = LoadAssetAsync(address).GetAwaiter();
if(awaiter.IsCompleted){
	cache = awaiter.GetResult();
}else{
	awaiter.UnsafeOnCompleted(moveNext);
	return promise;
}
return Task.FromResult(cache);
```
实际上await 关键字最终还是调用的实现了await关键字方法的GetAwait方法。
接着就是async编译后
```c#
//
public Task<object> LoadAssetAsync(string address)
{
	// 内部生成一个状态机，不过该状态机是struct本身实例不会产生GC。
	var stateMachine = new __LoadAssetAsync
	{
	    __this = this,
	    address = address,
	    builder = AsyncTaskMethodBuilder<object>.Create(),
	    state = -1
	};
	var builder = stateMachine.builder;
    builder.Start(ref stateMachine);
    return stateMachine.builder.Task;
}
// 实现IAsyncStateMachine这样一个接口这主要就是
struct __LoadAssetAsync : IAsyncStateMachine
{
    //
    public Loader __this;
    public string address;

    // internal state
    public AsyncTaskMethodBuilder<object> builder;
    public int state;

    // internal local variables
    TaskAwaiter<object> loadAssetAsyncAwaiter;
	
    public void MoveNext()
    {
        try
        {
            switch (state)
            {
                // initial(call from builder.Start)
                case -1:
                    if (__this.cache != null)
                    {
                        goto RETURN;
                    }
                    else
                    {
                        // await LoadAssetAsync(address)
                        loadAssetAsyncAwaiter = __this.LoadAssetAsync(address).GetAwaiter();
                        if (loadAssetAsyncAwaiter.IsCompleted)
                        {
                            goto case 0;
                        }
                        else
                        {
                            state = 0;
                            builder.AwaitUnsafeOnCompleted(ref loadAssetAsyncAwaiter, ref this);
                                return; // when call MoveNext again, goto case 0:
                            }
                        }
                    case 0:
                        __this.cache = loadAssetAsyncAwaiter.GetResult();
                        goto RETURN;
                    default:
                        break;
                }
            }
            catch (Exception ex)
            {
                state = -2;
                builder.SetException(ex);
                return;
            }
 
            RETURN:
            state = -2;
            builder.SetResult(__this.cache);
        }
        public void SetStateMachine(IAsyncStateMachine stateMachine)
        {
            builder.SetStateMachine(stateMachine);
        }
    }
```

上面的StateMachine方法，实际上本身是给builder进行调用的，builder决定状态机该如何切换。例如当IsCompelte为 true,则会立即调用GetResult方法得到结果，而如果为false 则会进入到builder等待方法中。
```C#
public struct AsyncTaskMethodBuilder<TResult>
{
    MoveNextRunner runner;
    Task<TResult> task;
 
    public static AsyncTaskMethodBuilder<TResult> Create()
    {
        return default;
    }
 
    public void Start<TStateMachine>(ref TStateMachine stateMachine)
        where TStateMachine : IAsyncStateMachine
    {
        // when start, call stateMachine's MoveNext directly.
        stateMachine.MoveNext();
    }
 
    public Task<TResult> Task
    {
        get
        {
            if (task == null)
            {
                // internal task creation(same as TaskCompletionSource but avoid tcs allocation)
                task = new Task<TResult>();
            }
            return task.Task;
        }
    }
 
    public void AwaitUnsafeOnCompleted<TAwaiter, TStateMachine>(ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : ICriticalNotifyCompletion
        where TStateMachine : IAsyncStateMachine
    {
        // at first await, copy struct state machine to heap(boxed).
        if (runner == null)
        {
            _ = Task; // create TaskCompletionSource
            // create runner
            runner = new MoveNextRunner((IAsyncStateMachine)stateMachine); // 这里有装箱成本
        }
        // set cached moveNext delegate(as continuation).
        awaiter.UnsafeOnCompleted(runner.CachedDelegate);
    }
 
    public void SetResult(TResult result)
    {
        if (task == null)
        {
            _ = Task; // create Task
            task.TrySetResult(result); // same as TaskCompletionSource.TrySetResult.
        }
        else
        {
            task.TrySetResult(result);
        }
    }
}
 
public class MoveNextRunner
{
    public Action CachedDelegate;
 
    IAsyncStateMachine stateMachine;
 
    public MoveNextRunner(IAsyncStateMachine stateMachine)
    {
        this.stateMachine = stateMachine;
        this.CachedDelegate = Run; // Create cached delegate.
    }
    public void Run()
    {
        stateMachine.MoveNext();
    }
}
```

builder 的await方法创建了一个 runner类(这里也是开销),随后执行 awaiter的await 方法传入委托。awaiter的await方法本身基本上就只是执行了这个委托。
再来看这个委托，实际上执行的是MoveNextRunner的Run方法，而这个run方法实际上又指回MoveNext方法。所以说绕了一圈，实际执行过程是这样的。

builder.Start() => stateMachine.MoveNext() => 成功返回/失败执行 => bulider.AwaitUnsafeOnCompleted => stateMachine.MoveNext();

这个过程会持续始终，直到Compelte的时候，将值设置成功后结束。

可以看到原生的Task会产生不少的分配，例如runner类，执行的委托分配等等。
Unitask是如何解决这些分配的呢？ 
1. runner 对象用完后会放进对象池里，这里就避免了任务类回收，状态机相关的分配。
2. 状态机本身使用struct
```c#
// Unitask的bulider
public struct AsyncUniTaskMethodBuilder<T>
{
    IStateMachineRunnerPromise<T> runnerPromise;
    T result;
    public UniTask<T> Task
    {
        get
        {
            // when registered callback
            if (runnerPromise != null)
            {
                return runnerPromise.Task; 
            }
            else
            {
                // sync complete, return struct wrapped result
                return UniTask.FromResult(result);
            }
        }
    }
 
    public void AwaitUnsafeOnCompleted<TAwaiter, TStateMachine>(ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : ICriticalNotifyCompletion
        where TStateMachine : IAsyncStateMachine
    {
        if (runnerPromise == null)
        {
            // get Promise/StateMachineRunner from object pool
            AsyncUniTask<TStateMachine, T>.SetStateMachine(ref stateMachine, ref runnerPromise);
        }
 
        awaiter.UnsafeOnCompleted(runnerPromise.MoveNext);
    }
 
    public void SetResult(T result)
    {
        if (runnerPromise == null)
        {
            this.result = result;
        }
        else
        {
            // SetResult signal Task continuation, it will call task.GetResult and finally return to pool self.
            runnerPromise.SetResult(result);
 
            // AsyncUniTask<TStateMachine, T>.GetResult
            /*
            try
            {
                return core.GetResult(token);
            }
            finally
            {
                TryReturn();
            }
            */
        }
    }
}

```
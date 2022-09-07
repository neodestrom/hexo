---
title: 安卓和Unity3D交互
date:  2022-09-07 10:20:51
categories:  [Game,UnityAPI]
tags: article
---

## 安卓和Unity3D交互

**[参考文章](https://bbs.huaweicloud.com/blogs/detail/285723)**

`交互简单来说就是通信的一种，即unity和安卓可以互相发送信息。由于unity模拟器无法直接调用安卓特有的一些方法或者是基于安卓平台的SDK，那么Unity能够调用安卓里面的方法可以扩展unity的功能，意义十分巨大。同样的，安卓平台里可能通过自己的SDK获取到了某类信息，这类信息获取到后无法直接使用，需要发送给unity，unity根据这条信息做出反应，例如更新视图等。`  
 由此引出安卓和unity3d通信的常用场景。

* 安卓 >>> Unity 更新视图。

* Unity >>> 安卓 调用特有的SDK方法，注册SDK中的回调

### Unity叫安卓

```csharp
//通过反射机制获取到class
AndroidJavaClass jc = new AndroidJavaClass ("com.unity3d.player.UnityPlayer");
//特殊静态属性currentActivity获取到activity上下文，通过这个上下文可以随心所欲调用该activity下的方法
AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject> ("currentActivity");
jo.Call ("login","");
```

以上就是最基础的用法。接下来还有一些用法。

* unity调用android静态方法。

  > 注意：这里的AndroidJavaClass()里面的是自己的包名+类名
  >
  > 这样写的可以不用继承UnityPlayerActivity也可以用
  >

```java
//C#: 这个类本质上就是通过反射去获取到对应的类，所以很方便。
AndroidJavaClass jc = new AndroidJavaClass ("com.example.test.Test");
jc.GetStatic<AndroidJavaObject> ("login","");
//android:
package com.example.test;
public class Test{
public static void login( String str ) {    
   // 写上自己的操作
   }
}
```

* unity调用android非静态方法。

  ```csharp
  AndroidJavaClass jc = new AndroidJavaClass("com.hasee.librarydemo.Test"); //包名加类名
  AndroidJavaObject jo = jc.CallStatic<AndroidJavaObject>("getInstance");
  jo.Call("login","");
  ```

### 安卓叫Unity

* 通过发消息UnitySendMessage的方式调用Unity

```javascript
//把消息发送给Unity场景中iFlytekASRController物体上的OnResult方法，哪儿都可以调用
UnityPlayer.UnitySendMessage("iFlytekASRController", "OnResultWake", resultString);
```

* 通过代理AndroidJavaProxy 的方式

  > 安卓给Unity通讯可以通过这个AndroidJavaProxy 的方式，使用起来比通过发消息要麻烦些，但是能干的事多了，而且使用这个代理相当于给Unity 的回调，比发消息要靠谱点。发消息使用的反射的机制，字符串也容易写错，可能会有发送失败、延迟等可能，但是用这个回调自然就更稳定啦 下面我写一个Demo做通讯测试
  >

​ **AS端：** 1.首先要在AS端写一个接口，接口中可以写一些需要给Unity调用的方法或参数等，等于用于传过去给Unity的回调

```javascript
package com.example.test;

public interface UnityasrEventCallback {
    public void Speechcontent(int a);
    public void Test1(String msg);
}
复制代码
```

2.写一个Activity用于与unity通讯，Unity端就调用这个方法（setCallback(UnityasrEventCallback callback)）将代理传过来，然后通过传过来的代理，将AS接口中定义的方法和参数回调传给Unity端

```javascript
   private UnityasrEventCallback mCallback;
    //获取接口内容
    public void setCallback(UnityasrEventCallback callback){
        Log.d("@@@", "UnityBatteryEventCallback setCallback start ");
        mCallback = callback;
        Log.d("@@@", "UnityBatteryEventCallback setCallback end ");
         mCallback.Test1("连通成功了");
          mCallback.Speechcontent(666);
    }
复制代码
```

**Unity端：** 1.在一个cs脚本中写一个内部类，然后继承AndroidJavaProxy。然后写一个构造方法继承AS的 包名+接口名 然后实现这个接口，方法名一定要与AS中写的一样，再定义一个数值用于接收AS中传过来的数据即可

```csharp
 public class AsrEventCallback : AndroidJavaProxy
    {
        public AsrEventCallback() : base("com.example.test.UnityasrEventCallback") {  }
        public void Speechcontent(int content){int a = content;}
        public void Test1(string msg){string b = msg;}
    }
复制代码
```

base()里的是AS中接口的，包名+接口名。看好不要写错
`public AsrEventCallback() : base("com.example.test.UnityasrEventCallback"){ }`
然后在这个cs脚本的Start中new一个代理，然后通过 jo.Call("setCallback", asrEventCallback);将这个代理传到AS中，然后AS就可以调用这个代理给Unity返回数据了

```csharp
    void Start()
    {
    AndroidJavaClass jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
    AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject>("currentActivity");
    AsrEventCallback asrEventCallback = new AsrEventCallback();
    // 设置语音识别回调函数接口
        jo.Call("setCallback", asrEventCallback);
    }
```

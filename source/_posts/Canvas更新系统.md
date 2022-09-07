
---
title: Canvas更新系统
date:  2022-09-07 10:20:12
categories:  [Game,框架分析,UGUI,UGUI的 Canvas更新系统]
tags: article
---

## 简介
Canvas 更新系统采用了脏标记模式， 该模式能够捕捉变化的UI元素，并进行最小程度的更新。
Canvas 类会在每次渲染之前 执行 CanvasUpdateRegistry 类中的 PerformUpdate() 方法
该方法会分别取出 布局重建队列 和  图像重建队列里面的元素，执行其中的 Rebuild方法对其进行更新。

重建队列是由 具体的UI元素加上去的
```c#
// Graphic
//  标记具体的布局
public virtual void SetVerticesDirty()  
{  
    if (!IsActive())  
        return;  
    m_VertsDirty = true;  
    // 将本身传入到registery 中进行注册
    CanvasUpdateRegistry.RegisterCanvasElementForGraphicRebuild(this);  
    if (m_OnDirtyVertsCallback != null)  
        m_OnDirtyVertsCallback();  
}
// 每类UI元素可能会有不同的更新自身的UI策略
public virtual void Rebuild(CanvasUpdate update)  
{  
    if (canvasRenderer == null || canvasRenderer.cull)  
        return;  
  
    switch (update)  
    {  
        case CanvasUpdate.PreRender:  
            if (m_VertsDirty)  
            {                UpdateGeometry();  
                m_VertsDirty = false;  
            }            if (m_MaterialDirty)  
            {                UpdateMaterial();  
                m_MaterialDirty = false;  
            }            break;  
    }
}
```

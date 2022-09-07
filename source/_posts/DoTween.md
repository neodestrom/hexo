---
title: DoTween
date:  2022-09-07 10:20:29
categories:  [Game,框架分析,Unity DoTween]
tags: article
---

## 简介
Dotween 是一个缓动插件，能对Unity原生的对象的一些值进行缓动， 同时也支持一些自定义数值的缓动

### Dotween 的自定义缓动
`DOTween.To(getter,setter,to, float duration)`
将给定属性从当前值更改为指定属性
```c#

_progressAnim = DOTween.To(() => progressBar.fillAmount, value =>  
{  
    var x = _progressWidth * value;  
    _fishRect.localPosition = new Vector3(x, _fishLocalPos.y, _fishLocalPos.z);  
    progressBar.fillAmount = value;  
},progress , 1f).SetEase(Ease.Linear).SetSpeedBased().SetUpdate(UpdateType.Fixed);
						   
```


---
title: Debug
date:  2022-09-07 10:22:35
categories:  [Unity 代码库]
tags: article
---

在脚本中加入该代码用于按钮触发
``` C#
private void OnGUI()  
{  
    if (GUI.Button(new Rect(20, 40, 80, 20), "测试点这里"))  
    {        
	    Debug.Log("OK");  
    }

}
```
---
title: Debug
date:  2023-08-31 10:17:03
categories:  [计算机,有用的代码库,Unity 代码库]
tags: article
index_img:
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
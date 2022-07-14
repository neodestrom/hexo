
---
title: Debug
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
---
title: Unity 自定义编辑器
date:  2022-09-07 10:21:00
categories:  [Game,UnityAPI,Unity Editor]
tags: article
---


### 自定义编辑器概述
自定义编辑器由两个脚本构成，一个为普通脚本，另一个脚本则是继承Editor类且指向这个普通脚本。第二个脚本可以决定普通脚本在Inspector窗口的显示。

### 自定义脚本基本呈现
自定义脚本需要在类的上面标注属性 `CustomEditor` 指向一个Inspector编辑器
`CanEditMultipleObjects` 表示可以用此编辑器更改所有拥有脚本的Inspector。

```c#
using UnityEditor;  
[CustomEditor(typeof(LookAtPoint))]  
[CanEditMultipleObjects]  
public class LookAtPointEditor : Editor {  
    SerializedProperty lookAtPoint;  
  
    void OnEnable()  
    {        
	    // 可以拿到脚本上标注的值
	    lookAtPoint = serializedObject.FindProperty("lookAtPoint");  
    }  
    public override void OnInspectorGUI()  
    {        
	    // 这里可以编写各种GUI代码
	    serializedObject.Update();  
        EditorGUILayout.PropertyField(lookAtPoint);  
        serializedObject.ApplyModifiedProperties();  
    }}

```

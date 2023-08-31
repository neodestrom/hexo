---
title: Obsidian插件基础原理
date:  2023-04-26 11:05:56
categories:  [杂记]
tags: article
index_img:
---

# 概要
Obsidian的插件主要是继承于Obsidian提供的Plugin接口，随后从该接口调用实现各种功能。
这里Obsidian主要功能可以是添加命令、添加全局提示、添加设置面板。
- 对于添加命令，只需要在软件页面调用ctrl+p则可以显示所有的命令，在插件里只需要调用 `this.addCommand({id:"",name:"",callback:()=>{}})` 即可添加一个命令。
- 对于全局提示，只需要执行`new Notice("")`在构造参数里填写字符即可。
- 对于左边按钮栏内容，需要执行`this.addRibbonIcon("IconName","name",(e)=>{})`即可，需要一个icon名称，按钮名称，随后点击后会执行后面的回调。
- 添加设置面板为`this.addSettingTab(new SettingTab(this.app,this))` 里面传入一个类型为PluginSettingTab的类，在这个类里可以控制其插件的设置样式。
	- 在类里的 display() 生命周期里，可引入一个 `containerEl` 这个类为浏览器HTMLElement类型，可以实现html样式。
	- 需要在最后调用new Setting,从而应用样式。

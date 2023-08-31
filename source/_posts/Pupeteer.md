---
title: Pupeteer
date:  2023-04-26 11:05:21
categories:  [NodeJs代码库,Pupeteer]
tags: article
index_img:
---

## 概要
Pupeteer是一个谷歌出版的类似于selenium库的操作浏览器的一个框架。不同之处为，该框架可以使用用户数据,还可以很方便的链接一个已经打开的浏览器进行爬取操作。
## 安装
`npm i puppeteer`
## 基础使用
```js
import puppeteer from 'puppeteer';
(async () => {
	// 启用一个浏览器对象
	const browser = await puppeteer.launch();
	// 启用一个页面
	const page = await browser.newPage();
	await page.goto('https://developer.chrome.com/');
	// 设置视距
	await page.setViewport({width: 1080, height: 1024});
	// 点击某个class
	await page.type('.search-box__input', 'automate beyond recorder');
	// 另一种点击方式
	const searchResultSelector = '.search-box__link';
	await page.waitForSelector(searchResultSelector);
	await page.click(searchResultSelector);
	// Locate the full title with a unique string
	const textSelector = await page.waitForSelector(
	  'text/Customize and automate'
	);
	const fullTitle = await textSelector.evaluate(el => el.textContent);
	// Print the full title
	console.log('The title of this blog post is "%s".', fullTitle);
	await browser.close();
})();

```

## 实用使用
### 使用本地浏览器数据去访问需要鉴权的网站
首先需要**关闭所有浏览器**包括浏览器的后台，随后访问的时候带上userDataDir。
userDataDir 的获取在谷歌浏览器中访问 chrome://version/ ，即可得到个人资料路径。
```js
const puppeteer = require('puppeteer-core');

(async () => {
	const browser = await puppeteer.launch({
		// 这里必须将本地链接转义
		executablePath: 'C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe',
		headless: false,
		defaultViewport: {
		  width: 1920,
		  height: 1080
		},
		userDataDir: 'C:\\Users\\admin\\AppData\\Local\\Google\\Chrome\\User Data\\Default'
	});
const page = await browser.newPage();
await page.goto('https://www.baidu.com');
await page.screenshot({ path: 'baidu.png' });
await browser.close();
})();
```
### 链接到现在已经打开的浏览器上
需要浏览器运行时执行 `--remote-debugging-port=9222 ` 指令。对于windows 可以在快捷方式那里对于文件目录那里后面添加上该指令，即可每次运行都会默认开启一个谷歌浏览器的服务。
随后访问 http://localhost:9222/json/version ,其返回一个json 。 需要复制webSocketDebuggerUrl，作为后续puppetter连接的依据。
```json
   "Browser": "Chrome/112.0.5615.138",
   "Protocol-Version": "1.3",
   "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36",
   "V8-Version": "11.2.214.14",
   "WebKit-Version": "537.36 (@b160f1d9e90aa6940d17d5cb44d9e815205d2024)",
   "webSocketDebuggerUrl": "ws://localhost:9222/devtools/browser/4c2a97e5-395c-4d61-a96b-b63ed7a01f77"
```
以下是puppeteer 代码
```ts
(async () => {
    const browser = await puppeteer.connect({
	    // 这里不能访问localhost,据测试用 127.0.0.1最好
        browserWSEndpoint: 'ws://127.0.0.1:9222/devtools/browser/4c2a97e5-395c-4d61-a96b-b63ed7a01f77'
    });
    const page = await browser.newPage();
    await page.goto('https://google.com/');
    await page.screenshot({ path: 'google.png' });
    // await browser.close();
})();
```
由于访问localhost可能会存在访问不到的情况，可通过本地控制台 输入 `netstat` 指令检查所有开放的端口，检查是否浏览器已经开放了端口。

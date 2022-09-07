---
title: Mock
date:  2022-09-07 10:22:16
categories:  [NodeJs代码库]
tags: article
---

#### 常用的mock
```js
const Mock = require('mockjs')

const mockDatas = Mock.mock(
        {
            "playerInfo|99999": [
                {
                    "device_id": '@string(20)',
                    "country|1": ['US','CN'],
                    "name": '@name',
                    "avatar_id": '@integer',
                }
            ]
        })
```

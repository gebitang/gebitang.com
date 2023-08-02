+++
title = "a little ai"
description = "a little ai"
tags = [
    "ai"
]
date = "2023-02-06"
topics = [
    "ai",
    "dev"
]
draft = false
toc=true
+++


## chatGPT注册

- 全局代理（美国或新加坡）
- 可接收美国短信的手机号码(KR上线后如果无法接收到短信，在sim应用程序中switch 一下国别到所在国)

>Q: Not available OpenAI's services are not available in your country.  
>A: 浏览器调试删除js： `window.localStorage.removeItem(Object.keys(window.localStorage).find(i=>i.startsWith('@@auth0spajs')))`

这种情况是由于代理没有全局，或者位置不对，需要切换代理到其它地区，比如韩国。

本人实测，在其中一个Mac上正常完成注册并可以chat；但另外一台Mac折腾的可以注册完成，但始终无法开启chat，一直是类似上面的提示。

全局代理设置完成之后，登陆访问 https://www.ipip.net/ip.html 查询本机的ip，确保已经是正确的代理ip之后，再登录使用chatGPT——另外一台设备上也登录OK了

可以正常开启聊天之后，代理似乎可以重新切换为autoswitch模式了

## next

- [护照过期线上申请](http://banshi.beijing.gov.cn/pubtask/task/1/110000000000/bea13e6c-b013-457b-aa6b-7feef2a68476.html)
- [100 欧元:爱沙尼亚电子公民身份](https://zhuanlan.zhihu.com/p/212249614)
- [become an e-⁠resident](https://www.e-resident.gov.ee/become-an-e-resident/)
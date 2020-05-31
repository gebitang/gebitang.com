
+++
title = "Authorization"
description = "Basic Auth, OAuth 1.0 and OAuth 2.0"
tags = [
    "oath",
    "tech"
]
date = "2020-05-21"
topics = [
    "auth"
]
toc = true
+++

## Basic Authorization

[basic-authentication-in-postman/](https://www.toolsqa.com/postman/basic-authentication-in-postman/)

>用postMan调用我们部署的sonarqube的webapi的/issues/buld_change这个接口的时候，一直提示401的权限验证问题，我们也在Authorization这里选中了Basic Auth类型 并设置了用户名和密码

复习一下code含义——

- 400 bad request，请求报文存在语法错误
- 401 unauthorized，表示发送的请求需要有通过 HTTP 认证的认证信息
- 403 forbidden，表示对请求资源的访问被服务器拒绝
- 404 not found，表示在服务器上没有找到请求的资源

所以是认证未通过。看完上面的文章，猜测应该是没有进行Base64编码。Basic Authorization默认需要对`username`和`password`两个字段先以`:`连接，然后整体做Base64编码转换。使用token方式时，密码可以为空。

java端的演示代码类似：
```
String credential = Credentials.basic("token-value-generated", "");

#okhttp的实现
/** Returns an auth credential for the Basic scheme. */
  public static String basic(String username, String password) {
    return basic(username, password, ISO_8859_1);
  }

  public static String basic(String username, String password, Charset charset) {
    String usernameAndPassword = username + ":" + password;
    String encoded = ByteString.encodeString(usernameAndPassword, charset).base64();
    return "Basic " + encoded;
  }
```

## 微博分享

- 注册开发者
- 申请一个[网站应用](https://open.weibo.com/connect)
- 获取到测试的token

[开发文档首页](https://open.weibo.com/wiki/API%E6%96%87%E6%A1%A3_V2)
[微博API](https://open.weibo.com/wiki/%E5%BE%AE%E5%8D%9AAPI)

[第三方分享到微博](https://open.weibo.com/wiki/2/statuses/share)

[授权机制说明](https://open.weibo.com/wiki/%E6%8E%88%E6%9D%83%E6%9C%BA%E5%88%B6%E8%AF%B4%E6%98%8E)

>为了方便开发者编程、调试接口，针对开发者自身，可以使用开发者授权，这样开发者自己就不用通过频繁授权自己的应用，来获取授权token才能调接口。而且开发者的授权token有效期为**`5`**年。
>
>开发者自身授权有一定的限制要求，即只有应用的所有者账号授权自己名下的应用时，才享有这个特性。
>
>开发者自身授权可以通过接口测试工具快速获得，前提是需要用开发者的账号登录到微博开放平台，接口测试工具下才会列出这个账号下的应用，进而也只能获取到开发者名下所列出应用的授权token。

[获取测试token](https://open.weibo.com/tools/console)

- 上传图片时需要采用`multipart/form-data`编码方式，
- 没有上传图片则采用正常编码方式`application/x-www-form-urlencoded`

[https://open.weibo.com/wiki/FAQ](https://open.weibo.com/wiki/FAQ)

## XAUTH fanfou

官方示例代码为[python实现](https://github.com/FanfouAPI/FanFouAPIDoc/wiki/Xauth)，本地调试时需要做一点调整`oauth`调整为`from oauth import oauth`

提供了对应的参数后，可以获取到用户数据。

搜索Java的实现，[一款简洁的饭否Android客户端](https://github.com/betroy/xifan)这个工程下对XAuth的[实现XAuthUtils](https://github.com/betroy/xifan/blob/33a61f3e963378ba39404d143dc556d5ca629660/app/src/main/java/com/troy/xifan/util/XAuthUtils.java)可以借鉴。

下列代码相对于官方示例的Java版本：

- 利用XAuth方式获取Access_Token
- 使用Token的方式[验证用户信息](https://github.com/FanfouAPI/FanFouAPIDoc/wiki/account.verify-credentials)

```java
String url = "http://fanfou.com/oauth/access_token";
Request request = new Request.Builder().url(url).build();
String code = getAuthorization(request);

request = new Request.Builder().url(url).addHeader("Authorization", code).build();

Response response = new OkHttpClient().newCall(request).execute();
System.out.println( response.code() + "------------" + response.body().string());
//oauth_token=xxxxxxxx&oauth_token_secret=yyyyyyyyyyyy

//获取到上述两个值之后，后续的签名需要用到
url = "http://api.fanfou.com/account/verify_credentials.xml";
request = new Request.Builder().url(url).build();
code = getAuthorization(request);

//
request = new Request.Builder().url(url).addHeader("Authorization", code).build();
response = new OkHttpClient().newCall(request).execute();
System.out.println(response.code());
System.out.println(response.body().string());
```

[访问饭否API](https://github.com/FanfouAPI/FanFouAPIDoc/wiki/Oauth#%E4%BD%BF%E7%94%A8access-token%E8%AE%BF%E9%97%AE%E9%A5%AD%E5%90%A6api)

每次API请求需要包含如下参数——

| 参数 | 意义 |
| ---|--- |
| oauth_consumer_key | 饭否应用的API Key |
| oauth_token | **Access Token** |
| oauth_signature_method | 签名方法，目前只支持HMAC-SHA1 |
| oauth_signature | 签名值，签名方法见[OAuthSignature](https://github.com/FanfouAPI/FanFouAPIDoc/wiki/Oauthsignature) |
| oauth_timestamp | 时间戳，取当前时间 |
| oauth_nonce | 单次值，随机的字符串，防止重复请求 |


[OAuthSignature](https://github.com/FanfouAPI/FanFouAPIDoc/wiki/Oauthsignature) 需要注意——

- 如果请求存在Authorization的Header并且以OAuth开头，则其随后的参数, 所有的名字以oauth_　和xauth_(用于XAuth)开头的参数都参与签名
- 如果请求是GET, 则QueryString中所有的参数都参与签名
- 如果请求是POST并且，Content-Type是application/x-www-form-urlencoded, 则所有的POST参数需要参加签名
- 其他的POST请求，参数都不参加签名，包括multipart/form-data

也别担心，如果签名签错了，返回值会告诉你期望的签名内容是怎样的。`Invalid signature. Expected basestring is xxx`，通常都是encode出现了问题。可以利用这个[OAuthEncoder](https://github.com/betroy/xifan/blob/33a61f3e963378ba39404d143dc556d5ca629660/app/src/main/java/com/troy/xifan/util/OAuthEncoder.java)编码一下。
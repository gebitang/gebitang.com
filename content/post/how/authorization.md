
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

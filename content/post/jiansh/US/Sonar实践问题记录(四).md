+++
title = "Sonar实践问题记录(四)"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "US"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/477561fbfb18)

### 配置

默认配置包括以下几个方面——
- DATABASE 
- WEB SERVER
- SSO AUTHENTICATION
- LDAP CONFIGURATION 
- ELASTICSEARCH
- UPDATE CENTER
- LOGGING
- OTHERS

默认配置文件`sonar.properties`中对每一项有详细的说明。

--- 

与默认配置不同的地方——

1. DATABASE：提供用户名、密码以及链接地址
```
sonar.jdbc.username=usexxx
sonar.jdbc.password=xxpxxx
sonar.jdbc.url=jdbc:mysql://ip.or.domain:3306/db_name?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
```

2. 更新默认的web服务端口：
```
sonar.web.port=8080
```

3. 提供LDAP服务 LDAP CONFIGURATION

```
sonar.security.realm=LDAP
sonar.security.savePassword=false
sonar.forceAuthentication=true
ldap.url=ldap://server.address.name.com
ldap.bindDn=userName
ldap.bindPassword=password
ldap.authentication=simple

ldap.user.baseDn=OU=User_Accounts,DC=domain_corp,DC=com
ldap.user.realNameAttribute=cn
ldap.user.emailAttribute=mail
ldap.user.request=(&(objectClass=user)(sAMAccountName={login}))
```

4. CE（提供分析数据进程的配置）COMPUTE ENGINE

``` 
sonar.ce.javaOpts=-Xmx2048m -Xms512m -XX:+HeapDumpOnOutOfMemoryError
```

5. ES端口号： Elasticsearch port
```
sonar.search.port=9901 
```

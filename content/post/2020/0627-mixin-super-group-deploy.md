+++
title = "Mixin 大群部署完全教程 实操总结"
description = "how to deploy group service for Mixin"
tags = [
    "go"
]
date = "2020-06-27"
topics = [
    "go"
]
toc = true
+++

# 步骤概述

{{< fluid_imgs
  "pure-u-1-1|https://cdn.dbarobin.com/loplv1n.png|架构示意图|https://dbarobin.com/2019/05/19/mixin-super-group/"
>}}


前提条件—— 

- 基于安全考虑，需要使用SSL方式：Certbot客户端
- 后端使用Go语言，前端使用VUE：需要Go语言环境+Node.js环境
- 数据库使用PostgreSQL：安装Postgres
- 实际上是一个Web服务站点：使用nginx

以上都在linux环境下进行安装部署。

---

[Mixin 大群部署完全教程](https://dbarobin.com/2019/05/19/mixin-super-group/)   
[Mixin 大群部署完全教程 实操(一)](https://www.jianshu.com/p/941c7721b392)  
[Mixin 大群部署完全教程 实操(二)](https://www.jianshu.com/p/476a89ee7ec0)  

跟着教程实际操作了一遍，目前大群可以正常使用了。总结一下遇到的问题和处理方案。

教程已经写得很详细，不过新手如我总会遇到一些问题。

- 域名 + SSL 问题
- Nginx 配置学习
- PostgreSQL基础用法
- V2ray + COW实现go build编译 
- yaml文件配置
- 日志问题

## SSL 问题

[Let's Encrypt ](https://letsencrypt.org/getting-started/)介绍，有Shell能力的，使用[Certbot](https://certbot.eff.org/ "Certbot")客户端，实现了 [ACME protocol](https://tools.ietf.org/html/rfc8555) Automatic Certificate Management Environment (ACME)。

Ubuntu 18.04 + Nignx 环境 [ubuntu bionic nginx](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx) 

- 环境准备

将certbot仓库添加到安装仓库中，顺序执行下列命令——

```
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
```

- 安装certbot

```
sudo apt-get install certbot python3-certbot-nginx
```

- 安装证书 

如果有运行中的Nginx，可以执行`sudo certbot --nginx`自动进行配置。

前提是nginx中`server_name`有对应的真实有效域名，并且域名配置了对应的公网IP（域名解析），因为证书安装过程会进行实际的域名访问。

以下内容会被自动增加到nginx的对应配置文件中
```
listen 443 ssl; # managed by Certbot
# 证书，二级域名可以使用相同的证书 例如 group.buxiangzhuce.com也可以使用这个配置
ssl_certificate /etc/letsencrypt/live/buxiangzhuce.com/fullchain.pem; # managed by Certbot
# 私钥
ssl_certificate_key /etc/letsencrypt/live/buxiangzhuce.com/privkey.pem; # managed by Certbot
include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
```

**注：** 正常到这一步就完成了SSL证书的配置。服务器开发443端口后可以进行https访问，可以到 `https://www.ssllabs.com/ssltest/`站点进行测试验证

如果只是生产证书，不进行配置，可以执行`sudo certbot certonly --nginx`命令

- 配置自动更新

因为Let's Encrypt证书的有效期只有3个月，所以需要配置自动更新。

```
sudo certbot renew --dry-run

Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/buxiangzhuce.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator nginx, Installer nginx
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for api.buxiangzhuce.com
http-01 challenge for buxiangzhuce.com
http-01 challenge for group.buxiangzhuce.com
Using default address 80 for authentication.
Using default address 80 for authentication.
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of nginx server; fullchain is
/etc/letsencrypt/live/buxiangzhuce.com/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/buxiangzhuce.com/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.

```

执行完之后，可以在`/etc/cron.d`目录下看的自动配置的`certbot`任务，也可以执行`systemctl list-timers` 查看

##  Nginx 配置学习

[install Nginx on Ubuntu](http://nginx.org/en/linux_packages.html#Ubuntu)

- 安装依赖 `sudo apt install curl gnupg2 ca-certificates lsb-release`
- 添加稳定版镜像 `echo "deb http://nginx.org/packages/ubuntu `lsb_release -cs` nginx"   | sudo tee /etc/apt/sources.list.d/nginx.list`
- 添加签名key `curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -` 验证版本 `sudo apt-key fingerprint ABF5BD827BD9BF62`
- 执行安装命令 `sudo apt update; sudo apt install nginx`
 
默认占有80端口（本来就是做转发，当然要靠它）`nginx`就正常启动了，默认配置文件`nginx.conf` 位于 `/usr/local/nginx/conf`, `/etc/nginx`或` /usr/local/etc/nginx`，ubuntu环境下位置为`/etc/nginx`

[Nginx Beginners Guide](http://nginx.org/en/docs/beginners_guide.html)  使用`nginx -s signal` signal 可以为以下参数：

- *stop* — fast shutdown
- *quit* — graceful shutdown
- *reload* — reloading the configuration file
- *reopen* — reopening the log files

参考默认的配置`/etc/nginx/conf.d/default.conf`和文章[Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html)学习基础的配置选项

## PostgreSQL基础用法

- 安装数据库

```
$ apt -y update
$ apt -y install postgresql postgresql-contrib
...
..
.
$ /usr/lib/postgresql/10/bin/postgres -V
postgres (PostgreSQL) 10.12 (Ubuntu 10.12-0ubuntu0.18.04.1)
```

- 数据库配置

`vim /etc/postgresql/10/main/postgresql.conf`
重启数据库 `systemctl restart postgresql`

- 基础用法

[How to List Databases and Tables in PostgreSQL Using psql](https://chartio.com/resources/tutorials/how-to-list-databases-and-tables-in-postgresql-using-psql/)

1. 切换到默认数据库管理员用户 postgres ` sudo -i -u postgres`；
2. `psql`命令连接进入数据库
3. 默认提示符为 `username=#`；
4. 使用`createdb dbname`创建数据库
5. `\conninfo`查看连接信息
6. `\i paht\to\schema.sql` 执行sql语句
7. `\dt`查看当前数据库下的表
8. `\c dbname`连接到数据库
9. `\dt`显示当前数据的表

```
➜  ~ sudo -i -u postgres
postgres@iZ2zebob74margeal4esovZ:~$
postgres@iZ2zebob74margeal4esovZ:~$ psql
psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1))
Type "help" for help.


postgres=# help
You are using psql, the command-line interface to PostgreSQL.
Type:  \copyright for distribution terms
       \h for help with SQL commands
       \? for help with psql commands
       \g or terminate with semicolon to execute query
       \q to quit
postgres=#

```

## 镜像编译

国内服务上编译时，有可能超时失败

```
go: cloud.google.com/go@v0.41.0: Get "https://proxy.golang.org/cloud.google.com/go/@v/v0.41.0.mod": dial tcp 34.64.4.113:443: i/o timeout
```

再次推荐[AgentNEO](https://agneo.co/?rc=8d50vj3e)。[一枝红杏出墙来](../../jiansh/fgw/一枝红杏出墙来/)

V2ray + COW实现go build编译。前者提供Sock5代理，后者提供http代理，并使用Sock5做二级代理


- 安装V2ray


` bash < (curl -L -s https://install.direct/go.sh)` 搭建了一个本地的Socks5代理，

新建文件 `config.json`，复制下文本并粘贴。修改其中的**节点地址**和**V2Ray UUID**。

`v2ray -config /home/root/config.json`即可启动。 该客户端仅提供了一个 Socks5 服务在本地的 1080 端口，还需要针对不同的软件进行配置才能使用。

```
{
  "inbounds": [
    {
      "port": 1080,
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "settings": {
        "auth": "noauth"
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "**节点地址**",
            "port": 443,
            "users": [
              {
                "id": "**V2Ray UUID**",
                "alterId": 1
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/v2"
        }
      }
    }
  ]
}
```

- 安装[COW](https://github.com/cyfdecyf/cow/)

`curl -L git.io/cow | bash` 下载不下来，可以将内容本地下载，复制到`install-cow.sh`中，赋值可执行权限后进行安装。

编辑 `~/.cow/rc`配置文件，然后使用`cow `命令即可启动。

```
listen = http://127.0.0.1:7777

# SOCKS5 二级代理
proxy = socks5://127.0.0.1:1080
```

然后export代理之后，就可以继续使用 `go build`进行编译


## yaml文件配置

- 类似下面的`appearance`项，要么配置完整，要么全部删除，否则前端展示时可能出现转圈现象（解析加载不完整？）

其中的icon为图片链接即可。url的值可以为网页链接，也可以为Mixin上的机器人链接，机器人链接的格式为`mixin://users/user-id`，例如在Mixin消息里`mixin://users/e49eeb58-64cd-4065-b4f8-bac86647448d`会被识别为一个链接，点击后会拉起一个机器人页面。

```
# 主页外观
appearance:
  home_shortcut_groups:
    - label_en: "3-Party Services"
      label_zh: "第三方提供的服务"
      shortcuts:
        # icon URL
        - icon: "..."
          label_en: "Service Name"
          label_zh: "服务名称"
          # 服务链接
          url: "https://exinone.com"
```

- 前端VUE配置中的链接要与后端保持一致，都使用https链接的方式

```
# 前端web目录 cp -v env.example .env.local
$ vim .env.local

NODE_ENV="production"
VUE_APP_WEB_ROOT="https://group.buxiangzhuce.com"
VUE_APP_API_ROOT="https://api.buxiangzhuce.com"
VUE_APP_CLIENT_ID="xxxxxxxx"
VUE_APP_ROUTER_MODE="history"

# 后端
service:
  name: "mixin.lol"
  enviroment: "production"
  port: 7001
  host: "https://group.buxiangzhuce.com"
mixin:
  client_id        : "" 
  client_secret    : ""
  session_asset_pin : ""
  pin_token        : ""
  session_id       : ""
  session_key: |
    -----BEGIN RSA PRIVATE KEY-----
    -----END RSA PRIVATE KEY-----

```

机器人的client_id的值不会变，后端的`client_secret`从app主页的`APP SECRET` generate a new secret获取，其他的值从app主页的 `APP SESSION` generate a new session获取。

## 日志查看

将前后端分别部署为两个服务之后，使用 [`journalctl -u service.name`](https://unix.stackexchange.com/a/225407)查看对应服务的日志信息。
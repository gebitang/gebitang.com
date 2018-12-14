+++
title = "Hugo, Github & Wercker"
description = "How to update your personal website automatically"
tags = [
    "hugo"
]
date = "2018-01-27"
topics = [
    "hugo",
    "wercker"
]
toc = true
+++

“自留地里写东西。”

1. 使用Hugo生成静态网站
2. 使用Github pages部署网站
3. 利用Wercker完成自动更新部署

写完，commit到仓库，同步本地仓库到github即完成了网站的自动更新。[Official Doc](https://gohugo.io/hosting-and-deployment/deployment-with-wercker/)

最少必要知识：

- Git安装和基础操作
- Github注册和使用：github page服务
- Hugo安装和基础操作：完成站点创建
- Wrecker配置

## Git安装、使用

[官网下载](https://git-scm.com/downloads)安装对应平台的应用程序。

### Git基本使用

[史上最浅显易懂的Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000) 

## Github账号、仓库

创建账号：略。

登录后访问[创建仓库](https://github.com/new)，新建一个自己命名的，如geb.im的空仓库，不包括readme、不添加license、不添加gitignore文件。

场景成功后，系统会自动提示你如何进行下一步的操作。

### 本地创建仓库，使用这个新建的仓库作为远程仓库
```
# 命令行操作

# 添加一个README.md文件并写入 # geb.im 作为内容
echo "# geb.im" >> README.md

# 初始化本地仓库
git init

# 将 README.md文件添加入仓库
git add README.md

# 提交commit信息，
git commit -m "first commit"

# 将 git@github.com:gebitang/geb.im.git作为本地仓库的远程仓库
git remote add origin git@github.com:gebitang/geb.im.git

# 推送本地仓库到远程仓库
git push -u origin master
```

## Hugo安装、生成站点 

[官网](https://gohugo.io/)
[下载安装](https://github.com/gohugoio/hugo/releases)。选择Hugo是因为安装完立即即可使用，三种主流平台全支持，傻瓜式默认安装即可。windows用户实际上只有一个.exe文件，将其所在的目录添加到系统的环境变量Path中即可使用。

验证 
```
# 命令行执行，目前最新的为v0.34版本
 ~ hugo version
Hugo Static Site Generator v0.26 darwin/amd64 BuildDate: 2017-08-17T21:50:17+08:00
```

### 新建站点

hugo是命令行可识别的命令，可使用`hugo --help`查看帮助；支持二级命令帮助，如`hugo new --help`表示new这个二级命令如何使用。`hugo new sit your.site.name` 就是新建一个hugo站点。

```
# 新建一个名称叫 geb.im的站点
➜  github hugo new site geb.im
Congratulations! Your new Hugo site is created in /Users/gebitang/github/geb.im.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
➜
➜  github cd geb.im
➜  geb.im ls
archetypes  config.toml content     data        layouts     static      themes
➜  geb.im
```

### 为新站点添加主题
到[主题区](https://themes.gohugo.io/)选择自己喜欢的主题进行下载安装。随时可以修改主题内容或自己创作主题。 

下载[purehugo主题](https://themes.gohugo.io/purehugo/)，Download按钮对应的是下载地址，类似 https://github.com/dplesca/purehugo.git 
```
#进入到 themes目录执行
➜  geb.im cd themes
➜  themes ls
➜  themes git clone https://github.com/dplesca/purehugo.git
Cloning into 'purehugo'...
remote: Counting objects: 430, done.
remote: Total 430 (delta 0), reused 0 (delta 0), pack-reused 430
Receiving objects: 100% (430/430), 457.17 KiB | 67.00 KiB/s, done.
Resolving deltas: 100% (219/219), done.
➜  themes ls
purehugo

```

### 添加内容
删除不需要的文件 .git ，这个文件夹保存了git版本管理的所有相关内容

使用 `hugo new about.md`添加内容
```
➜  themes rm -rf purehugo/.git/
## 回到主目录，添加 about.md文件
➜  geb.im hugo new about.md
/Users/gebitang/github/geb.im/content/about.md created
➜  geb.im
```

文件默认状态是draft即草稿形式，不会被显示出来，将对于的值设置为false即可
```
---
title: "About"
date: 2018-01-28T14:47:49+08:00
draft: false
---

上面的内容为hugo支持的markdown格式的meta信息。draft值默认为true，修改为false。否则hugo server启动时无法生产页面。
this is a test file.
```

### 本地启动server

```
#执行 hugo server --watch 命令可以实时监控内容更新；--theme参数知道使用什么主题
hugo server --watch --theme=purehugo
```

启动完成后可以在浏览器访问 localhost:1313，就可以看到自己的博客了。

### 配置化

实际上hugo启动的配置都在工作根目录下的config.toml文件中，默认如下面前三行内容；如启动使用的主题可以在这个文件中配置，添加第四行的内容即可。此时启动时就不需要再直接指定所使用的主题了
```
baseURL = "http://example.org/"
languageCode = "en-us"
title = "My New Hugo Site"
theme = "purehugo"
```

另外，hugo支持多个server同时使用，会自动分配端口，如第一个使用了默认的端口1313，第二个会使用类似63044
```
➜  geb.im hugo server --watch
ERROR 2018/01/28 15:40:31 port 1313 already in use, attempting to use an available port
Started building sites ...
Built site for language en:
0 draft content
0 future content
0 expired content
1 regular pages created
6 other pages created
0 non-page files copied
1 paginator pages created
0 tags created
0 categories created
total in 5 ms
Watching for changes in /Users/gebitang/github/geb.im/{data,content,layouts,static,themes}
Serving pages from memory
Web Server is available at http://localhost:63044/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

可以使用Ctrl+C随时结束server。

到此为止，本地的静态博客系统就可以随意使用了

## Wercker配置使用

### 准备工作
为了使用Wercker，需要对本地站点目录进行一些配置操作。

- 初始化本地目录
- 添加.gitignore文件
- 添加静态文件robots.txt

`git init`初始化本地目录；在根目录下添加.gitignore文件并写入`/public`内容，表示public这个目录下的文件不进行版本维护；在static目录下添加robots.txt文件，并写入以下内容，表示运行所有的蜘蛛进行抓取页面内容。
```
User-agent: *
Disallow:
```

```
➜  geb.im git init
Initialized empty Git repository in /Users/gebitang/github/geb.im/.git/
➜  geb.im git:(master) ✗ echo "/public" >> .gitignore
➜  geb.im git:(master) ✗ echo "User-agent: *\nDisallow:" > static/robots.txt
➜  geb.im git:(master) ✗ ls
archetypes  config.toml content     layouts     static      themes
➜  geb.im git:(master) ✗ ls static
robots.txt
➜  geb.im git:(master) ✗ ls -a
.           ..          .git        .gitignore  archetypes  config.toml content     layouts     static      themes

# 将当前目录所以文件添加到仓库工作区
➜  geb.im git:(master) ✗ git add .
```

提交内容到本地仓库并为本地仓库添加远程仓库连接
```
➜  geb.im git:(master) ✗ git commit -a -m 'init commit'
[master (root-commit) c582ca6] init commit
 31 files changed, 794 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 archetypes/default.md
 create mode 100644 config.toml
 create mode 100644 content/about.md
 create mode 100644 static/robots.txt
 create mode 100644 themes/purehugo/.gitignore
 create mode 100644 themes/purehugo/LICENSE.md
 create mode 100644 themes/purehugo/README.md
 create mode 100644 themes/purehugo/archetypes/default.md
 create mode 100644 themes/purehugo/assets/css/blog.css
 create mode 100644 themes/purehugo/assets/css/syntax-highlighter.css
 create mode 100644 themes/purehugo/assets/js/jquery.min.js
 create mode 100644 themes/purehugo/assets/js/jquery.prettysocial.min.js
 create mode 100644 themes/purehugo/assets/js/rainbow-custom.min.js
 create mode 100644 themes/purehugo/assets/js/scripts.js
 create mode 100644 themes/purehugo/gulpfile.js
 create mode 100644 themes/purehugo/images/screenshot.png
 create mode 100644 themes/purehugo/images/tn.png
 create mode 100644 themes/purehugo/layouts/_default/list.html
 create mode 100644 themes/purehugo/layouts/_default/single.html
 create mode 100644 themes/purehugo/layouts/index.html
 create mode 100644 themes/purehugo/layouts/partials/analytics.html
 create mode 100644 themes/purehugo/layouts/partials/footer.html
 create mode 100644 themes/purehugo/layouts/partials/header.html
 create mode 100644 themes/purehugo/layouts/partials/pagination.html
 create mode 100644 themes/purehugo/layouts/partials/sidebar.html
 create mode 100644 themes/purehugo/layouts/shortcodes/img-responsive.html
 create mode 100644 themes/purehugo/package.json
 create mode 100644 themes/purehugo/static/css/all.min.css
 create mode 100644 themes/purehugo/static/js/all.min.js
 create mode 100644 themes/purehugo/theme.toml
➜  geb.im git:(master) git status
On branch master
nothing to commit, working tree clean
➜  geb.im git:(master)
```

为本地仓库添加远程连接并推送
```
➜  geb.im git:(master) git remote add origin git@github.com:gebitang/geb.im.git
➜  geb.im git:(master) git push origin master
Counting objects: 50, done.
Writing objects: 100% (50/50), 278.52 KiB | 1.49 MiB/s, done.
Total 50 (delta 0), reused 0 (delta 0)
To github.com:gebitang/geb.im.git
 * [new branch]      master -> master
➜  geb.im git:(master)

```

出现上面的类似内容，则可以在github上看到推送的内容了。


### 配置github仓库


使用master分支维护代码，使用 gh-pages分支进行发布，所以需要先新建一个gh-pages分支并推送到远端。

在对应仓库的setting中选择启用github pages服务并使用gh-pages分支

```
git branch gh-pages
➜  geb.im git:(master) git branch -a
  gh-pages
* master
  remotes/origin/master
➜  geb.im git:(master) git checkout gh-pages
Switched to branch 'gh-pages'
➜  geb.im git:(gh-pages) git push origin gh-pages
Total 0 (delta 0), reused 0 (delta 0)
To github.com:gebitang/geb.im.git
 * [new branch]      gh-pages -> gh-pages
```

在github对应仓库的settings下设置CNAME，或直接手动添加一个CNAME文件，内容为注册的域名
```
➜  geb.im git:(gh-pages) echo "geb.im">CNAME
➜  geb.im git:(gh-pages) ✗ git add CNAME
➜  geb.im git:(gh-pages) ✗ git commit -a
[gh-pages d1c0ae0] add CNAME file.
 1 file changed, 1 insertion(+)
 create mode 100644 CNAME
➜  geb.im git:(gh-pages) git push origin gh-pages
Counting objects: 3, done.
Writing objects: 100% (3/3), 442 bytes | 442.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:gebitang/geb.im.git
   c582ca6..d1c0ae0  gh-pages -> gh-pages
➜  geb.im git:(gh-pages)
```

### 更新远程分支
当gh-pages分支已经在远端存切与本地有关联时。

- 删除远程分支
- 重置本地的跟踪
- 重新推送到远端

```
# 删除远端分支

➜  geb.im git:(master) git push origin :gh-pages
To github.com:gebitang/geb.im.git
 - [deleted]         gh-pages

# unset跟踪关系
 geb.im git:(master) git checkout gh-pages
Switched to branch 'gh-pages'
Your branch is based on 'origin/gh-pages', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)
➜  geb.im git:(gh-pages) git branch --unset-upstream
➜  geb.im git:(gh-pages) gi

# 重新上传分支
➜  geb.im git:(gh-pages) git push origin gh-pages
Counting objects: 33, done.
Writing objects: 100% (33/33), 14.36 KiB | 2.05 MiB/s, done.
Total 33 (delta 0), reused 0 (delta 0)
To github.com:gebitang/geb.im.git
 * [new branch]      gh-pages -> gh-pages
➜  geb.im git:(gh-pages)
```


### 配置wercker
- 使用github账户登录wercter站点: Login with github

- [新建Application](https://app.wercker.com/applications/create)

配置一共分四步：

1. Select SCM，使用github登录后，会自动识别出使用的是github
2. Select a Repository
3. Configure Access
4. Review

将生成的wrecker.yml文件内容修改如下并保存在根目录：


- 可以在Step Repository中搜索`hugo build`

[hugo-build](https://app.wercker.com/steps/arjen/hugo-build)，升级到最新版本为0.40.3，测试可用。~~0.17的旧版本不可用~~

```
box: wercker/default
build:
  steps:
    - arjen/hugo-build:
        version: "0.40.3"
        theme: purehugo
```

- 搜索`gh-pages`可以得到部署相关的step

```
deploy:
  steps:
    - gh-pages:
        token: $GIT_TOKEN
        domain: gem.im
        basedir: public
```

其中使用的变量GIT_TOKEN需要在wercker上对应的Application的[Environment](https://app.wercker.com/gebitang/geb.im/environment)选项下进行设置。

- 指定pipeline内容，在


```
➜  geb.im git:(gh-pages) vi wercker.yml
➜  geb.im git:(gh-pages) ✗ git add wercker.yml
➜  geb.im git:(gh-pages) ✗ git commit -a -m 'add wercker file.'
[gh-pages 2d720f1] add wercker file.
 1 file changed, 6 insertions(+)
 create mode 100644 wercker.yml
➜  geb.im git:(gh-pages)
```

### 配置域名

找域名服务提供上给自定义域名添加A记录执行github的ip
```
192.30.252.153
192.30.252.154

# 利用dig查看设置的A记录是否起效
geb.im git:(master) dig geb.im  +nostats +nocomments +nocmd

; <<>> DiG 9.9.7-P3 <<>> geb.im +nostats +nocomments +nocmd
;; global options: +cmd
;geb.im.				IN	A
geb.im.			60	IN	A	192.30.252.153
geb.im.			82175	IN	NS	f1g1ns2.dnspod.net.
geb.im.			82175	IN	NS	f1g1ns1.dnspod.net.
f1g1ns1.dnspod.net.	64252	IN	A	180.163.19.15
f1g1ns1.dnspod.net.	64252	IN	A	182.140.167.166
...
```

### 为wercker配置token

- [生成tokens](https://github.com/settings/tokens)
为了在wercer.yaml文件中使用变量而不是直接暴露自己的token

### 重置远程分支

删除远程分支；`git push original :gh-pages`

如果出错，先清理远程分支，再重新同步远程分支

### 配置pipeline

在对应的workflow tab页下配置需要的pipeline，然后编辑整个的workflow过程

https://help.github.com/articles/setting-up-an-apex-domain/

搭建——应该交配置——好这个网站后一直想总结一下流程，一直拖延到今天。还是跟LP立了军令状之后才动手的:( 这样交差还行吧:)
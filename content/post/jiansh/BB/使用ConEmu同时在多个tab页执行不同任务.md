+++
title = "使用ConEmu同时在多个tab页执行不同任务"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "BB"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/d1077ecb2d38)

喜欢ConEmu的一点是终于可以在一个window里管理多个tab了。

在新tab页面批量执行不同的任务。[official doc](https://conemu.github.io/en/LaunchNewTab.html),  [solution]( https://superuser.com/a/593648)

ConEmu默契启动后在 ` C:\Users\userName`目录，可以将不同的执行脚本保存在当前目录下。例如：启动hugo、启动jekyll等

- 启动hugo脚本 `startHugoSite.bat`
```
#启动hugo站点
D:
cd  D:/openSources/site/hugositefolder
hugo server --watch --verbose --buildDrafts
```

- 启动jekyll脚本 `startJekyllSite.bat`
```
# 启动jekyll站点
D:
cd  D:/openSources/site/jekyllsitefolder
bundle exec jekyll serve
```

- 启动桌面应用脚本`startDesktop.bat`
```
#启动桌面应用
D:
cd D:\openSources\site\desktop-app
yarn electron:serve
```

- 启动上述脚本的脚本 `startAll.bat`
```
@echo off 
echo start trio
# https://superuser.com/a/593648
start "trio" "D:\tools\conEmu\ConEmu64.exe" /cmdlist ^> cmd /k startHugoSite.bat ^|^|^| cmd /k startJekyllSite.bat ^|^|^| cmd /k startDesktop.bat
echo finish jobs.
```

上述命令会新启动一个ConEmu窗口并在其中打开三个tab页面，分别执行三个启动脚本。强迫症如我，就主动退出当前窗口。使用新创建的window了。

bat脚本内容：`start "trio" cmd /k call b.bat`中：
- “trio”是一段字符串，代表新打开的cmd窗口的名字，可以随便起名。
- /k是表示新打开的cmd窗口在执行完命令后保存打开状态，如果希望执行完就关闭窗口就使用/c

如果在当前脚本环境中执行，可直接使用`call`目录：
- `call b.bat`表示call命令，即调用b.bat文件；该命令可以用""括起来，即："call b.bat"


+++
title = "Jekyll-on-Github-Pages"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Mixin-lol"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/6c771fc29b7b)

在github上使用github page：[jekyll github pages](https://jekyllrb.com/docs/github-pages/) 实际上，GitHub Pages are powered by Jekyll behind the scenes, so they’re a great way to host your Jekyll-powered website for free.

所有的相关文档可以在这里找到：[Setting up a GitHub Pages site with Jekyll](https://help.github.com/en/github/working-with-github-pages/setting-up-a-github-pages-site-with-jekyll)

新建站点之后，最新版本的jekyll 4.0.0似乎还没上到github上。Github page使用的官方依赖：  [https://pages.github.com/versions/](https://pages.github.com/versions/)看起来是3.8.5版本。

编辑Gemfile文件：
- `gem "github-pages", group: :jekyll_plugins` 替换为 `gem "github-pages", "~> 203", group: :jekyll_plugins`
- `gem "jekyll", "~> 4.0.0"` 替换为 `gem "jekyll", "~> 3.8.5"`版本。否则会提示 `Could not find gem github-pages` 

然后重新执行:
```
# https://github.com/prose/starter/issues/44#issuecomment-562822698
bundle update jekyll
# followed by :
bundle install
# test site by launching it locally :
bundle exec jekyll serve
```

--- 

使用jekyll果然就比较简单了。
- 直接部署master分支
- 直接指出定制域名（ [配置对应的dns解析即可](https://help.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site)）


[github page官方文档](https://help.github.com/en/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll)、[jekyll站点结构](https://jekyllrb.com/docs/structure/)，[jekyll永久链接](http://jekyllcn.com/docs/permalinks/) [jekyll permalinks](https://jekyllrb.com/docs/permalinks/)


手动生成站点——

```
# jekyll --help
jekyll build
# 使用环境配置（不使用当前项目的配置）
bundle exec jekyll  build
```
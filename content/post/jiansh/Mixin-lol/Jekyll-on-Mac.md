+++
title = "Jekyll-on-Mac"
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



[link on JianShu](https://www.jianshu.com/p/9a18ea358b14)

标准操作只需要三部：

1. 安装gem：`gem install bundler jekyll`
2. 生成站点：`jekyll new my-awesome-site`
3.  启动站点：`bundle exec jekyll serve`

`bundle exec jekyll serve --drafts`
**VS.** 
`hugo server --watch --verbose --buildDrafts`

以为会简单许多，还是可能会遇到不同的问题。但Mac上会给出清楚的提示，按照提示执行即可。

- Ruby 版本过低`brew install ruby`
- 更新变化变量 `echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.zshrc; source ~/.zshrc`
- 安装兼容版本 `bundle update`

--- 
Mac上默认的ruby版本似乎有点低。安装`gem install bundle jekyll`时提示`jekyll-sass-converter requires Ruby version >= 2.4.0.`，需要先升级一下ruby版本：`brew install ruby` 

安装完成后，需要更新下PATH
```
echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
#检查ruby 版本

➜ ruby -v
ruby 2.3.7p456 (2018-03-28 revision 63024) [universal.x86_64-darwin18]
➜ source ~/.zshrc
➜  ruby -v
ruby 2.6.5p114 (2019-10-01 revision 67812) [x86_64-darwin18]

```
然后再安装jekyll `gem install jekyll`，会花费一些时间。

启动站点 `bundle exec jekyll serve`提示不兼容。

```
bundle exec jekyll serve
Bundler could not find compatible versions for gem "nokogiri":
  In snapshot (Gemfile.lock):
    nokogiri (= 1.10.7)

  In Gemfile:
    github-pages (~> 203) was resolved to 203, which depends on
      nokogiri (>= 1.10.4, < 2.0)

Running `bundle update` will rebuild your snapshot from scratch, using only
the gems in your Gemfile, which may resolve the conflict.
```

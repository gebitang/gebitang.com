+++
title = "github"
description = "Why is it so successful?"
tags = [
    "github"
]
date = "2020-08-26"
topics = [
    "github"
]
toc=true
+++


[Migrate From Travis CI to GitHub Actions](https://developer.okta.com/blog/2020/05/18/travis-ci-to-github-actions)  
[Travis CI to GitHub Actions](https://medium.com/@nwillc/travis-ci-to-github-actions-925d574f3f2b)
Travis CI还没怎么用上呢，现在都开始迁移到GA([github actions](https://docs.github.com/en/actions))了？

发现这个宝藏库[lab.github.com](https://lab.github.com/)，跟着学习一下基础的[Hello World Action](https://lab.github.com/githubtraining/github-actions:-hello-world)。

交互非常惊喜，大纲进行说明，开始第一步后，自动在用户目录下创建[hello-github-actions](https://github.com/gebitang/hello-github-actions)项目。

**[github-learning-lab](https://github.com/marketplace/github-learning-lab)**机器人会自动新建一个[issues #1](https://github.com/gebitang/hello-github-actions/issues/1)简单描述Actions的背景知识、第一步如何操作——直接以comments的方式进行交互。

按照流程操作后，提交一次PR，然后会在这个[pull #2](https://github.com/gebitang/hello-github-actions/pull/2)请求里依次完成接下来的6个步骤——

- Add a Dockerfile
- Add an entrypoint script
- Add an action.yml file
- Start your workflow file
- Run an action from your workflow file
- Trigger the workflow
- Incorporate the workflow

按照要求完成一步进行push操作，机器人会根据提交的内容自动判断是否符合步骤要求。如果不符合，会自动进行相应的提示，自动列出“可能出错的场景”（例如我第一次将文件夹名称写错了）

完成后，自动关闭[issues #1]并新开一个[issues #3](https://github.com/gebitang/hello-github-actions/issues/3)祝贺完成。

此时回到[Hello World Action]课程，可以看到这哥课程已经完成。交互体验完美。

--- 

这样依赖[wercker](https://app.wercker.com/)生成hugo静态代码站的CI流程可以迁移到GA了，搜索一下，已经很多人提供写好的[hugo actions](https://github.com/marketplace?query=hugo&type=actions)
了。

事实上，[github Actions](https://github.com/actions)本身也是一个开源组织，要用到的具体Action，例如[checkout@v2](https://github.com/actions/checkout)就是这个组织下的具体项目。

写好的actions可以提交到[商店marketplace](https://github.com/marketplace)
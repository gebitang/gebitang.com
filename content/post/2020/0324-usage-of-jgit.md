+++
title = "JGit 支持推送"
description = "Usage of JGit"
tags = [
]
date = "2020-03-24"
topics = [
    "git"
]
+++



>[JGit](https://www.eclipse.org/jgit) is a pure Java library implementing the Git version control system. It is an Eclipse project and started out as the Git library for EGit, which provides a Git integration into the Eclipse IDE.

[A Guide to JGit](https://www.baeldung.com/jgit)  
[JGit - Tutorial](https://www.vogella.com/tutorials/JGit/article.html#opening-a-local-repository-with-jgit)  
[JGit User Guide](https://wiki.eclipse.org/JGit/User_Guide)  

基础的流程包括：clone、更新内容、提交commit、推送。教程通常缺少最后一步：推送更新到远端。

使用[用户名、密码方式](https://stackoverflow.com/a/47458653/1087122)——

```
   Git git = Git.open(localPath); 

    // add remote repo:
    RemoteAddCommand remoteAddCommand = git.remoteAdd();
    remoteAddCommand.setName("origin");
    remoteAddCommand.setUri(new URIish(httpUrl));
    // you can add more settings here if needed
    remoteAddCommand.call();

    // push to remote:
    PushCommand pushCommand = git.push();
    pushCommand.setCredentialsProvider(new UsernamePasswordCredentialsProvider("username", "password"));
    // you can add more settings here if needed
    pushCommand.call();    
```

针对**github**可优化未token方式，从[github tokens](https://github.com/settings/tokens)这里新建一个自己的token，然后上面的Credential Provider可以直接使用`new UsernamePasswordCredentialsProvider("token", ""`即可。

上面的内容亲测可用。

>GitHub uses password authentication for https URLs and public-key authentication for SSH URLs

--- 

使用public key时需要注意：Windows环境下生成的私钥默认无法识别（因为head行不兼容，[参考这里](https://stackoverflow.com/a/55740276/1087122)）使用`ssh-keygen -p -f file -m pem -P passphrase -N passphrase`进行重置（如果创建时passphrase没有设置，使用""代替即可）

[JGit Authentication Explained](https://www.codeaffine.com/2014/12/09/jgit-authentication/)这里提到了各个不同协议使用的区别。使用SSH的方式暂未验证。

[org.eclipse.jgit.test](https://github.com/eclipse/jgit/tree/master/org.eclipse.jgit.test)官方测试库里有对应的代码。

通常使用的两种方式——

- `https://github.com/username/project.git`需要使用用户名+密码登录
- `git@github.com:username/project.git`表示SSH方式登录

[这里](https://gist.github.com/grawity/4392747)对各个协议类型做了详细的说明。备份如下——

---

Primary differences between SSH and HTTPS. This post is specifically about accessing Git repositories on GitHub.

## Protocols to choose from when cloning:

**plain Git**, aka `git://github.com/`

  * **Does not add security** beyond what Git itself provides. The server is not verified.

    If you clone a repository over git://, you should check if the latest commit's hash is correct.

  * You **cannot push** over it. (But see "Mixing protocols" below.)

**HTTPS**, aka `https://github.com/`

  * HTTPS **will always verify the server** automatically, using certificate authorities.

  * (On the other hand, in the past years several certificate authorities have been broken into, and many people consider them not secure enough. Also, some important HTTPS security enhancements are only available in web browsers, but not in Git.)

  * Uses **password** authentication for pushing, and still allows anonymous pull.

  * Downside: You have to enter your GitHub password every time you push. [Git can remember passwords][2] for a few minutes, but you need to be careful when storing the password permanently – since it can be used to change anything in your GitHub account.

  * If you have two-factor authentication enabled, you will have to use a [personal access token][3] instead of your regular password.

  * HTTPS **works practically everywhere**, even in places which block SSH and plain-Git protocols. In some cases, it can even be **a little faster** than SSH, especially over high-latency connections.

**HTTP**, aka `http://github.com/`

  * Doesn't work with GitHub anymore, but is offered by some other Git hosts.

  * Works practically everywhere, like HTTPS.

  * But does not provide any security – the connection is plain-text.

**SSH**, aka `git@github.com:` or `ssh://git@github.com/`

  * Uses **public-key** authentication. You have to [generate a **keypair**][1] (or "public key"), then add it to your GitHub account.

  * Using keys is **more secure than passwords**, since you can add many to the same account (for example, a key for every computer you use GitHub from). The private keys on your computer can be protected with passphrases.

  * On the other hand, since you do not use the password, GitHub does not require two-factor auth codes either – so whoever obtains your private key can push to your repositories without needing the code generator device.

  * However, the keys only allow pushing/pulling, but _not_ editing account details. If you lose the private key (or if it gets stolen), you can just remove it from your GitHub account.

  * A minor downside is that authentication is needed for all connections, so you always **need a GitHub account** – even to pull or clone.

  * You also need to **carefully verify the server's fingerprint** when connecting for the first time. Many people skip that and just type "yes", which is insecure.

  * (Note: This description is about GitHub. On personal servers, SSH can use passwords, anonymous access, or various other mechanisms.)

## Mixing protocols

### Globally

You can clone everything over `git://`, but tell Git to push over HTTPS.

    [url "https://github.com/"]
        pushInsteadOf = git://github.com/

Likewise, if you want to clone over `git://` or HTTPS, but push over SSH:

    [url "git@github.com:"]
        pushInsteadOf = git://github.com/
        pushInsteadOf = https://github.com/

These go to your *git* config file – sometimes `~/.config/git/config`, or `~/.gitconfig`, or just run `git config --edit --global`.

### Per-repository

You can also set different pull and push URLs for every remote separately, by changing <code>remote.<i>name</i>.pushUrl</code> in the repository's own `.git/config`:

    [remote "origin"]
        url = git://nullroute.eu.org/~grawity/rwho.git
        pushUrl = ssh://sine/pub/git/rwho.git

[1]: https://help.github.com/articles/generating-ssh-keys
[2]: https://help.github.com/articles/set-up-git#password-caching
[3]: https://help.github.com/articles/creating-an-access-token-for-command-line-use

### JGit 拉取代码逻辑

默认clone方法，将默认的远端分支clone到本地。假设远端默认分支为master。则本地跟踪的最新分支为`HEAD -> master, origin/master`

这几个概念要理解清楚——

- `master` is a local branch
- `origin/master` is a remote branch (which is a local copy of the branch named "master" on the remote named "origin")
One remote:
- `origin` is a remote

>`HEAD`: the current commit your repo is on. Most of the time HEAD points to the latest commit in your current branch, but that doesn't have to be the case. HEAD really just means "what is my repo currently pointing at".

```
git = Git.cloneRepository()
                        .setURI(urlPath)
                        .setBranch(branch)
                        .setDirectory(localFile)
                        .setCredentialsProvider(credentialsProvider)
                        .call();
```
相当于本地clone远端分支`origin/master`到本地`master`分支，当前最新`HEAD`执行`master`分支

拉取代码是使用类似如下代码——

```
PullResult result = git.pull()
    .setCredentialsProvider(user)
    .setRemote("origin")
    .setRemoteBranchName("dev")
    .call();
```

这样操作的结果比较“诡异”——`HEAD -> master, origin/dev`。相当于本地master分支重新跟踪了远端的`origin/dev`分支。

命令行操作可以使用`git checkout -B master origin/master`重置。主要`-b|-B`的区别。

> -B <new_branch> is created if it doesn’t exist; otherwise, it is reset.

拉取前配合stash操作——`git.stashCreate().call();` 倒是可以满足“拉取指定分支最新代码”的需求。


[JGit使用示例](https://github.com/centic9/jgit-cookbook), Please make sure to take a look at the nicely written [introduction](http://www.codeaffine.com/2015/12/15/getting-started-with-jgit/) and also use the existing [JavaDoc](http://download.eclipse.org/jgit/site/5.5.0.201909110433-r/apidocs/) and the [User Guide](http://wiki.eclipse.org/JGit/User_Guide) as well.


[What are the git concepts of HEAD, master, origin?](https://stackoverflow.com/questions/8196544/what-are-the-git-concepts-of-head-master-origin)

>I highly recommend the book ["Pro Git" by Scott Chacon](http://git-scm.com/book). Take time and really read it, while exploring an actual git repo as you do.

>**HEAD**: the current commit your repo is on. Most of the time `HEAD` points to the latest commit in your current branch, but that doesn't have to be the case. `HEAD` really just means "what is my repo currently pointing at".

>In the event that the commit `HEAD` refers to is not the tip of any branch, this is called a "detached head".

>**master**: the name of the default branch that git creates for you when first creating a repo. In most cases, "master" means "the main branch". Most shops have everyone pushing to master, and master is considered the definitive view of the repo. But it's also common for release branches to be made off of master for releasing. Your local repo has its own master branch, that almost always follows the master of a remote repo.

>**origin**: the default name that git gives to your main remote repo. Your box has its own repo, and you most likely push out to some remote repo that you and all your coworkers push to. That remote repo is almost always called origin, but it doesn't have to be.

>`HEAD` is an official notion in git. `HEAD` always has a well-defined meaning. `master` and `origin` are common names usually used in git, but they don't have to be.

查看`.git/config`文件可以看到具体对应的分支，类似——

```
[remote "origin"]
        url = git@github.com:gebitang/gebitang.com.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

“refs/heads/master” and “refs/remotes/origin/master”  are two different symbolic names that can point to different things. `refs/heads/master` is a branch in your working copy named `master`. Frequently that is a tracking branch of `refs/remotes/origin/master` because `origin` is the default name for the remote created by git clone and its primary branch is usually also named `master`.

You can see the difference between them with `git rev-list refs/heads/master..refs/remotes/origin/master` which will be empty if they are the same and will otherwise list the commits between them.


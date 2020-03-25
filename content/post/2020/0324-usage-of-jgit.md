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



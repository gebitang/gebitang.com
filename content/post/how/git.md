+++
title = "Git"
description = "Everything about Git"
tags = [
    "git"
]
date = "2021-01-07"
topics = [
    "git"
]
toc = true
+++

## github 

### 查看repo的first commit

[参考](https://stackoverflow.com/a/49647826/1087122)，执行——

- 在repo的主页查看commit的次数，例如1567
- 进入commit页面，点击older按钮将获取类似`commits/main?after=69d0500a1b4eeaa5c3b8e2df05d7a285b1231a3b+34&branch=main`
- 这里的`+34`表示commit向前走34次。根据第一步的commits次数就可以从UI上获取到最古老的commit

### 查看repo的创建时间 

[参考](https://stackoverflow.com/a/23612723/1087122)，直接通过get api即可，例如`https://api.github.com/repos/{:owner}/{:repository}`

使用代理可能ip会被限制，提示：

```
{
    message: "API rate limit exceeded for xx.xx.xx.90. (But here's the good news: Authenticated requests get a higher rate limit. Check out the documentation for more details.)",
    documentation_url: "https://docs.github.com/rest/overview/resources-in-the-rest-api#rate-limiting"
}
```

## Git Pro

### 推送tag

`git push origin <tag_name>` [推送单一tag到remote](https://stackoverflow.com/a/5195913/1087122)

### 获取commit的代码变动

[How to count the lines changed between two commits in gitlab?](https://stackoverflow.com/questions/42249174/how-to-count-the-lines-changed-between-two-commits-in-gitlab/42254042)

[gitlab commit api](https://docs.gitlab.com/ee/api/commits.html#get-a-single-commit)默认的`stats`参数为true，会返回类似如下数据。使用sdk时需要注意传递指定参数，否则这个字段不会返回

```
stats: {
    additions: 2,
    deletions: 3,
    total: 5
},
```

commit涉及到的每个具体文件的变更，需要调用[commit diff api](https://docs.gitlab.com/ee/api/commits.html#get-the-diff-of-a-commit)，再重新解析diff字段进行判断。

### 复现commit id

[How does Git create unique commit hashes, mainly the first few characters?](https://stackoverflow.com/questions/34764195/how-does-git-create-unique-commit-hashes-mainly-the-first-few-characters)

Git uses the following information to generate the sha-1:

- The source tree of the commit (which unravels to all the subtrees and blobs)
- The parent commit sha1
- The author info (with timestamp)
- The committer info (right, those are different!, also with timestamp)
- The commit message

[完整说明](https://gist.github.com/masak/2415865)，另外可参考[The anatomy of a Git commit](https://blog.thoughtram.io/git/2014/11/18/the-anatomy-of-a-git-commit.html)，和 [10.1 Git 内部原理 - 底层命令与上层命令](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E5%BA%95%E5%B1%82%E5%91%BD%E4%BB%A4%E4%B8%8E%E4%B8%8A%E5%B1%82%E5%91%BD%E4%BB%A4)对应的英文版本——[10.1 Git Internals - Plumbing and Porcelain](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)


执行以下步骤可以复现commit id——

- `git show` 获得当前最新的commit信息
- `git cat-file commit HEAD` 
- `printf "commit %s\0" $(git cat-file commit HEAD | wc -c)` 在开头增加`commit len\0` 文本，长度，NUL-terminated header
- `(printf "commit %s\0" $(git cat-file commit HEAD | wc -c); git cat-file commit HEAD) | sha1sum`组合上面两步的内容作为SHA1算法的输入，计算获得的40个字符长度文本就是commitid信息——复现成功


第二步的结构化信息——
```
sha1(
    tree            => 9c435a86e664be00db0d973e981425e4a3ef3f8d （对所有tree对象的递归计算到root节点）
    parents         => [0d973e9c4353ef3f8ddb98a86e664be001425e4a]
    author(with timestamp)          => Christoph Burgdorf <christoph.burgdorf@gmail.com> Sat Nov 8 11:13:49 2014 +0100
    committer(with timestamp)        => Christoph Burgdorf <christoph.burgdorf@gmail.com> Sat Nov 8 11:13:49 2014 +0100
    commit message  => "second commit"
)
```

理解Git中的对象概念：Tree Object（对目录、或目录和文件的SHA-1描述。如何获取的？）

```
.
├── .git (contents left out)
├── assets
|   ├── logo.png
|   └── app.css
└── app.js
```

{{< fluid_imgs
  "pure-u-1-|https://upload-images.jianshu.io/upload_images/3296949-8bdd5ebafaecf4c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240|demo|https://upload-images.jianshu.io/upload_images/3296949-8bdd5ebafaecf4c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240"
>}}


### git tree object

- [The Git Object Model](http://shafiul.github.io/gitbook/1_the_git_object_model.html)  
- [tree object格式](https://stackoverflow.com/questions/14790681/what-is-the-internal-format-of-a-git-tree-object)  
- [Git Internals - Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects) 
- [Git 内部原理 - Git 对象](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1)

键值数据库文件系统；使用zlib进行压缩存储对象；SHA1摘要算法

Every object consists of three things - a `type`, a `size` and `content`. The size is simply the size of the contents, the contents depend on what type of object it is, and there are four different types of objects: "blob", "tree", "commit", and "tag".

`git write-tree` Create a tree object from the current index. returns the hash of the top level tree. 

`git ls-tree` List the contents of a tree object. 可以使用 `git cat-file -p treehash`起到同样的作用

`git cat-file` Provide content or type and size information for repository objects


```
joechin@Gebitang MINGW64 /e/0daywork/gitinit (master)
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   h.txt
        new file:   lib/lealone.png


joechin@Gebitang MINGW64 /e/0daywork/gitinit (master)
$ git write-tree
f6da736647be31514378f51704936fcf40171a07

joechin@Gebitang MINGW64 /e/0daywork/gitinit (master)
$ git ls-tree f6da73
100644 blob ce013625030ba8dba906f756967f9e9ca394464a    h.txt
040000 tree 5de66da52a71d943871ac3753fbed006ee0688f4    lib

joechin@Gebitang MINGW64 /e/0daywork/gitinit (master)
$ git cat-file tree f6da736647be31514378f51704936fcf40171a07
100644 h.txt▒6%
               ▒۩▒V▒▒▒▒▒FJ40000 lib]▒m▒*q▒C▒▒u?▒▒▒▒▒

```


## Git命令使用

可配合GUI工具和命令行工具参考。

### 编译安装git 

环境：Oracle Linux 7相当于CentOS 7，但没有gcc编译器，无网络环境，需要手动离线安装gcc先 

[Linux离线安装gcc等依赖包](https://blog.csdn.net/li1325169021/article/details/115385274)，从[这里下载](http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/)

```shell
# 包含7个文件
rpm -ivh mpfr-3.1.1-4.el7.x86_64.rpm
rpm -ivh libmpc-1.0.1-3.el7.x86_64.rpm
rpm -ivh kernel-headers-3.10.0-862.el7.x86_64.rpm
rpm -ivh glibc-headers-2.17-222.el7.x86_64.rpm
rpm -ivh glibc-devel-2.17-222.el7.x86_64.rpm
rpm -ivh cpp-4.8.5-28.el7.x86_64.rpm
rpm -ivh gcc-4.8.5-28.el7.x86_64.rpm

# 单独安装glibc-headers时提示依赖glibc，glibc又依赖glibc-common互相依赖 
rpm -ivh glibc-common-2.17-317.el7.x86_64.rpm
rpm -ivh glibc-2.17-317.el7.x86_64.rpm

# 都下载后，强制无依赖安装 
rpm  -ivh  *.rpm --nodeps --force

# make 时提示 zlib.h no such file or directory 需要下载额外两个rpm包
zlib-1.2.7-18.el7.x86_64.rpm
zlib-devel-1.2.7-18.el7.x86_64.rpm

```

下载[git源码](https://mirrors.edge.kernel.org/pub/software/scm/git/)，解压到文件目录，进入目录。 

```shell
# 执行 
./configue 
# make 
make prefix=/usr/local/git all 
make prefix=/usr/local/git install
# update PATH
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
source /etc/bashrc
# verify 
git --version

```


### Filename too long in Git for Windows

[Filename too long in Git for Windows](https://stackoverflow.com/questions/22575662/filename-too-long-in-git-for-windows)

使用管理员权限执行`git config --system core.longpaths true`修复此问题。

error: could not lock config file C:/Program Files/Git/mingw64/etc/gitconfig: Permission denied


git clone sonarqube源码时，提示

>Resolving deltas: 100% (498459/498459), done.
fatal: cannot create directory at 'server/sonar-db-migration/src/test/resources/org/sonar/server/platform/db/migration/version/v84/permissiontemplates/fk/permtplcharacteristics/AddUniqueIndexOnTemplateUuidAndPermissionKeyColumnsOfPermTplCharacteristicsTableTest': Filename too long
warning: Clone succeeded, but checkout failed.
You can inspect what was checked out with 'git status'
and retry the checkout with 'git checkout -f HEAD'

然后再执行`git checkout -f HEAD`。今天我守着不完整的代码找半天webhook相关的内容

### git http clone方式报错：

[error: RPC failed; curl transfer closed with outstanding read data remaining](https://stackoverflow.com/questions/38618885/error-rpc-failed-curl-transfer-closed-with-outstanding-read-data-remaining)

- `$ git clone http://github.com/large-repository --depth 1`
- `cd large-repository`
- `git fetch --unshallow`

### git clone with token in command line 

work for gitlab at list: 

`git clone https://oauth2:ACCESS_TOKEN@somegitlab.com/vendor/package.git`

### change remote url

```
git remote -v
# View existing remotes

git remote set-url origin https://github.com/user/repo2.git
# Change the 'origin' remote's URL

git remote -v
# Verify new remote URL
```

### use token from command line

[use token from command line](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)

```
$ git clone https://github.com/username/repo.git
Username: your_username
Password: your_token
```

### fatal: Unable to create '/path/my_project/.git/index.lock': File exists

[sto](https://stackoverflow.com/a/7860765/1087122): `rm -f ./.git/index.lock`

### 修改commit工具 vim

提交说明信息的时候, linux默认是 nano 编辑器  
nano 这个编辑器使用 ctrl + x 来退出


```
# global
git config --global core.editor vim

# project update: .git/config add--
editor=vim
```

### 查找rep中特定用户的提交记录

[official doc](https://git-scm.com/docs/git-log), [from Stackoverflow](https://stackoverflow.com/a/2954501 )
```
git log --since=2018-01-01  --pretty=oneline --author=uername --author=gebitang |wc -l
```
### ssh variant 'simple' does not support setting port
update setting for git： `git config --global ssh.variant ssh`

### Git Bash: Could not open a connection to your authentication agent
生成公钥添加到对应的网站后，直接clone时，可能提示无法验证。
```
# need to start ssh-agent first.
eval `ssh-agent -s`
```

### Permission denied (publickey).
添加公钥之后，测试验证时会提示“Permission denied (publickey).”。这是因为还没有将生成的对应的key添加到ssh管理中（默认生成的会自动添加，后续生成的多个rsa文件需要手动添加）
```
# step 1 start agent first.  "ssh-agent bash" or " eval `ssh-agent` "

joechin@Gebitang MINGW64 ~/.ssh
$ ssh-agent bash


joechin@Gebitang MINGW64 ~/.ssh
$ ssh-add xxx_id_rsa
Identity added: xxx_id_rsa (aaaa@xxx.com)


# add the_abs_path_of_the_new_RSA_file, such as ~/.ssh/coding_id_rsa
ssh-add coding_id_rsa

# " Could not open a connection to your authentication agent" show, go to step 1

# then it could be cloned normaly.
```


### Fork and Sync Repo

```
1. git remote add upstream  https://github.com/url/project.git
2. git fetch upstream //download objects and refs from another repository 
3. git merge upstream/master //merge upstream/master to current branch

#meaning of merge
git merge origin master //将origin merge 到 master 上
git merge origin/master //将origin上的master分支 merge 到当前 branch 上 
```
- 从其他人的Repo中fork分支到自己的Repo下
- clone当前工程到本地
- [添加远程跟踪源(需要以http的方式添加)](https://help.github.com/articles/fork-a-repo/)

```
$ git remote add upstream  https://github.com/b3log/pipe.git

#查看添加结果
$ git remote -v
origin  git@github.com:gebitang/pipe.git (fetch)
origin  git@github.com:gebitang/pipe.git (push)
upstream        https://github.com/b3log/pipe.git (fetch)
upstream        https://github.com/b3log/pipe.git (push)

```

- [同步远程的跟踪源并合并到本地分支下](https://help.github.com/articles/syncing-a-fork/)

```
$ git fetch upstream

$ git merge upstream/master
```

### 无效的 index-pack

提示 `fatal: 过早的文件结束符 (EOF)`， `fatal: fetch-pack: invalid index-pack output`

无效的index-pack 输出。[参考这里的](https://stackoverflow.com/questions/21277806/fatal-early-eof-fatal-index-pack-failed)，有效的——

```
[core] 
packedGitLimit = 512m 
packedGitWindowSize = 512m 
compression = 9
[pack] 
deltaCacheSize = 2047m 
packSizeLimit = 2047m 
windowMemory = 2047m
```
都设置之后，执行  `git clone --depth 1 --branch $BRANCH $URL` 可以clone出来指定的branch


### 指定branch clone
` git clone -b dawn-2.x git@github.com:EOSIO/eos.git`

### git checkout 

```
test@test~$git checkout -b upcoming origin/upcoming
正在检出文件: 100% (1718/1718), 完成.
分支 'upcoming' 设置为跟踪来自 'origin' 的远程分支 'upcoming'。
切换到一个新分支 'upcoming'
test@test~$
```

### 0x0. 命令行单独对不同的文件进行commit操作
```
#理解暂存区的概念，多个文件即可分别处理
git add new.file/modified.file
#然后进行commit，即完成分别进行commit的动作
git commit 

#-a参数对所有工作区的文件都有效，未添加到跟踪区的文件除外
git commit -a 
```

### 0x1. 配置github账号
注意事项：[source](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

1. 生成rsa文件时可以指定不同的名称，以保证可以生成使用多个
2. 需要输入passphrase，否则系统认定不够安全，不允许添加到ssh-agent中
3. 配置config 文件，[支持多个git账号](https://my.oschina.net/csensix/blog/184434)
4. 如果不进行config的配置，在添加ssh key之后，将统一使用global的设置


### ssh config 示例 多个rsa文件时

ssh登录远程机器：

- server端不允许密码登录， `/etc/ssh/sshd_config`配置中`PasswordAuthentication no`
- client端生成ssh-key `ssh-keygen -t rsa`保持可区分的不同秘钥名称
- 将对应的public key的内容追加到server端对应用户的`/home/user/.ssh/authorized_keys`文件中
- client端配置`.ssh/config`内容，指定登录server时使用对应的秘钥

配置好之后，可以声明对应的alias，例如`alias geb="ssh ubuntu@xx.xxx.xx.xx"`

[SSH CONFIG FILE](https://www.ssh.com/ssh/config/)</br>
[OpenSSH Config File Examples](https://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/)

配置了多个公钥后，需要针对不同的站点进行配置，否则会在git clone等交互操作时依然提示需要输入秘密。

配置了config文件后，idea中会根据这个配置文件使用对应的key[Using Git integration](https://www.jetbrains.com/help/idea/using-git-integration.html#SSH-access)

提交commit时，依然需要至少有**`user.name`**和**`user.email`**信息
```
# 对于ssh的配置
# Defines for which host or hosts the configuration section applies. 
# The section ends with a new Host section or the end of the file. 
# A single * as a pattern can be used to provide global defaults for all hosts.
Host github.com
    # Specifies the real host name to log into. Numeric IP addresses are also permitted.
    HostName github.com
    # Defines the username for the SSH connection.
    User gebitang
    # Specifies a file from which the user’s DSA, ECDSA or DSA authentication identity is read. 
    IdentityFile ~/.ssh/id_rsa
Host prj.testin.cn
    HostName prj.testin.cn
    # Specifies the port number to connect on the remote host. only if the port for ssh connection is not 22.
    Port 8201
    # The default is ~/.ssh/identity for protocol version 1, and ~/.ssh/id_dsa, ~/.ssh/id_ecdsa and ~/.ssh/id_rsa for protocol version 2.
    IdentityFile ~/.ssh/gitlab_id_rsa
Host git.coding.net
    HostName git.coding.net
    IdentityFile ~/.ssh/coding_id_rsa
Host bitbucket
    HostName bitbucket.org
    IdentityFile ~/.ssh/bitbucket_id_rsa

```

[The SSH Protocol](https://git-scm.com/book/en/v2/Git-on-the-Server-The-Protocols)

To clone a Git repository over SSH, you can specify an `ssh://` URL like this:

`$ git clone ssh://[user@]server/project.git`

Or you can use the shorter scp-like syntax for the SSH protocol:

`$ git clone [user@]server:project.git`

- ssh配置

生成ssh-key
```
ssh-keygen -t rsa -b 4096 -C "email@address.com"

# create new key pair and add it to agent
ssh-keygen -t ed25519 -C "email@address.com"
ssh-agent bash
ssh-add id_ed25519
```
>添加ssh key（id_rsa_xxx、id_rsa_xxx.pub）到对应的repository，如github、gitlab，即可保证可以从对应的设备上就可以访问对应的repository。</br>

- git配置

>git config 配置完全局的 `--global user.name` 和 `--global  user.email`之后，可以在对应的仓库下，配置当前仓库使用的user.name 和user.email

### 0x2. commit amend修改commit
使用 git commit --amend 参数可以对最近的提交进行修改。</br>
作用相当于新提交了一个commit。
如果修改前的commit已经被同步过，需要先进行拉取 git pull的操作
否则在推送amend之后的commit时会提示:
```
$ git push origin master
To git@github.com:gebitang/gebitang.com.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:gebitang/gebitang.com.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
### 0x3. Permission denied (publickey)问题
[原因：SSH的配置文件ssh_config中的“IdentityFile“ 与实际情况不相符](http://blog.itpub.net/25851087/viewspace-1262468/)</br>
生成的一对秘钥，公钥上传到对应的站点后，使用-vt参数进行调试验证
```
#测试连接情况
# v for verbose mode, V for version
ssh -vT git@git.coding.net 
```

>如果clone时使用了https格式，则再次推送时总是会提示输入用户名密码，可以通过`git config --list`查看，然后修改`remote.origin.url`的值

### 0x4. 合并commits
[参考](http://www.jianshu.com/p/964de879904a)
```
# -i for interactive, 而不是指 其中，-i 的参数是不需要合并的 commit 的 hash 值 
# commitid 可理解为在此commit上进行rebase。即此commit之后的commit都可以进行rebase:)
git rebase -i commitid
```
原理：选择要rebase的基础commit，进入交互模式，根据提示对不同的commit处理为pick或squash

### 0x5. 清理、合并操作

>Your local changes to the following files would be overwritten by merge

提示上述错误信息时，可以使用下面的命令清理
```
git clean -d -fx
```
注意：会导致工程的所有配置都会丢失
```
git merge branch.name
```

### 0x6. 重新添加已经忽略的文件
```
#添加已经忽略的文件 
git add -f ignored/file/name
#忽略已经添加的文件 
git rm --cached file/want/to/be/ignore
```
[参考](http://blog.csdn.net/cyxlzzs/article/details/61422214)

### git rm 

[删除文件](https://www.liaoxuefeng.com/wiki/896043488029600/900002180232448)  
```
# 先本地删除
rm test.txt
# 在从git删除
git rm test.txt 
# 然后commit提交
git commit "delete test.txt file."
# 本地需要的话，可以更新到gitignore文件中
```

### 更新mac后默认自带的git无法使用

[StackOverFlow](https://stackoverflow.com/questions/32661484/os-x-cant-start-git-usr-bin-git-probably-the-path-to-git-executable-is-not)
```
➜  gebitang.com git status
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
➜  gebitang.com sudo xcodebuild -license
Password:
xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance
➜  gebitang.com sudo /usr/bin/git
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
➜  gebitang.com xcode-select --install
xcode-select: note: install requested for command line developer tools
➜  gebitang.com
➜  gebitang.com git:(master) ✗
➜  gebitang.com git:(master) ✗
```

### Clang issue 

执行`go run main.go`提示如下错误，找不到clang环境。
```
 go run main.go
# runtime/cgo
sh: line 1: 52743 Bus error: 10           /Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild -sdk /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk -find clang 2> /dev/null
clang: error: unable to find utility "clang", not a developer tool or in PATH
```

需要先安装`CommandLineTools`，可以先检查`xcode-select --print-path`安装路径，如果不是`/Library/Developer/CommandLineTools`，则需要先安装`xcode-select --install`，然后修改路径`sudo xcode-select --switch /Library/Developer/CommandLineTools`。正常情况下，此时环境恢复正常。

通过命令行更新xcode版本：检查配套版本：`softwareupdate --list`。参考[How to update Xcode from command line](https://stackoverflow.com/questions/34617452/how-to-update-xcode-from-command-line)老版本可参考此操作。

### 升级到Big Sur之后git无法使用

到git仓库下执行`git status`检查时提示找不到git，not a developer tool or in PATH。 搜索了一圈，只有这个[比较靠谱](https://timemachine.icu/posts/9894dc13/)

检查了一下`/Applications/Xcode.app/Contents/Developer/usr/bin`这个目录下有git，暂时将这个目录手动添加到环境变量后，问题解决。

```
sh: line 1:  3799 Bus error: 10           /Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild -sdk /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk -find git 2> /dev/null
git: error: unable to find utility "git", not a developer tool or in PATH
```

### git log历史

```
git log --oneline # show all logs line by line
git show commit # show the commit detail 

git log commit 　　# 查询commit之前的记录，包含commit
git log commit1 commit2  # 查询commit1与commit2之间的记录，包括commit1和commit2
git log commit1..commit2 # 同上，但是不包括commit1
```

### 项目初始化

- create empty project

```
# from 0 to 1
mkdir gotourgo
cd gotourgo
git init
echo "# gotourgo" >> README.md
git add README.md
git commit -m "first commit"
git remote add origin git@git.coding.net:gebitang/gotourgo.git
git push -u origin master

```

- add existed one 

```
git remote add origin git@git.coding.net:gebitang/gotourgo.git
git push -u origin master
```

### 指定branch复制
```
git clone git@192.168.1.93:Automation/project.git -b next next
```

### 修改、忽略、撤销、删除、更新url

```
# 修改已提交的commit
git commit --amend
#忽略已跟踪的文件
git update-index --assume-unchanged filename
#撤销用：
git update-index --no-assume-unchanged filename
#删除已入仓库的文件夹
git rm -r --cached .idea/
git rm -r --cached 要忽略的文件

#git修改文件名大小写的方法。
#首先，在git命令行里面运行：
git config core.ignorecase false

#更新url
git remote set-url origin new-url

#rebase
git rebase -i base-commit-hash
```
### Basic Command line instructions

```
Git global setup
git config --global user.name "gebitang"
git config --global user.email "gebitang@hotmail.com"

Create a new repository
git clone git@gitlab.com:gebitang/simpleweb.git
cd simpleweb
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master

Existing folder
cd existing_folder
git init
git remote add origin git@gitlab.com:gebitang/simpleweb.git
git add .
git commit -m "Initial commit"
git push -u origin master

Existing Git repository
cd existing_repo
git remote rename origin old-origin
git remote add origin git@gitlab.com:gebitang/simpleweb.git
git push -u origin --all
git push -u origin --tags
```
### 删除本地、远程分支

```
# 删除本地
git branch -d local_branch

# 删除远程 
git push --delete remote_branch_name
```

### 显示分支branch的关系
[STO](https://stackoverflow.com/questions/5298972/relationship-between-n-git-branches)  
```
# branch A B C
git log --graph --decorate --oneline --simplify-by-decoration A B C
# all 
git log --graph --decorate --oneline --simplify-by-decoration --all
```

### git tag 

tag默认是按照字母顺序排列的。[tag操作](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001376951758572072ce1dc172b4178b910d31bc7521ee4000)  
```
# 查看所以tag  -l 注意是字母"L",以列表形式列出所有tag的版本号. -n 显示出每个版本号对应的附加说明.
git tag -l -n
# 显示时间
git log --tags --simplify-by-decoration --pretty="format:%ci %d "
# 一行显示
git log --tags --simplify-by-decoration --pretty="format:%ci %d "
```

### git更新历史commit中的author信息
[https://help.github.com/en/github/using-git/changing-author-info](https://help.github.com/en/github/using-git/changing-author-info)

[https://www.git-tower.com/learn/git/faq/change-author-name-email](https://www.git-tower.com/learn/git/faq/change-author-name-email)

更新最近一次commit的committer：  

`git commit --amend --author="John Doe <john@doe.org>"`


[https://blog.tinned-software.net/rewrite-author-of-entire-git-repository/](https://blog.tinned-software.net/rewrite-author-of-entire-git-repository/)

Rewriting the history is done with “[git filter-branch](https://git-scm.com/docs/git-filter-branch)” by walking through the complete history. For each commit, filters are applied after which the changes are re-committed. The different filters allow modifying different parts of the commit.

The following uses “[git filter-branch](https://git-scm.com/docs/git-filter-branch)” to filter the history. Instead of manipulating the files to be recommitted like explained in [Remove files from git history](https://blog.tinned-software.net/remove-files-from-git-history/), this command uses the “–env-filter” to alter the environment in which the re-committing statement takes place.


单独clone出来当前repo
```
git clone --bare https://github.com/user/repo.git
cd repo.git
```

执行修改动作
```
#!/bin/sh
# -f to force rewrite, such as git filter-branch -f --env-filter
git filter-branch --env-filter '

OLD_EMAIL="your-old-email@example.com"
CORRECT_NAME="Your Correct Name"
CORRECT_EMAIL="your-correct-email@example.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

这样修改后，会导致远端与本地所有替换的内容都认为不同，需要重新先 'git pull' 获取一次，当出现'fatal: refusing to merge unrelated histories'的提示时，需要执行'git pull origin master --allow-unrelated-histories'命令，然后修改可能的冲突即可。

更新修改后的repo
```
git push --force --tags origin 'refs/heads/*'
```

最后可以删除临时的repo

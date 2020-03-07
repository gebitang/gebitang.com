+++
title = "git更新历史commit中的author信息"
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



[link on JianShu](https://www.jianshu.com/p/2ae4edc97f9f)

[https://help.github.com/en/github/using-git/changing-author-info](https://help.github.com/en/github/using-git/changing-author-info)

[https://www.git-tower.com/learn/git/faq/change-author-name-email](https://www.git-tower.com/learn/git/faq/change-author-name-email)


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

更新修改后的repo
```
git push --force --tags origin 'refs/heads/*'
```

最后可以删除临时的repo

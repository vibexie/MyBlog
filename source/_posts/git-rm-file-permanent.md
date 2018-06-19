---
title: Git删除文件及其所有历史记录
date: 2016-12-24 2:50:39
tags: git
---
在使用Git的时候，可能会不小心上传了一些敏感文件(例如密码), 或者不想上传的文件(没及时或忘了加到.gitignore里的)。

我们必须在不丢失其它文件提交历史的前提下，把这个敏感文件“赶尽杀绝”, 因此, 我们需要一个方法,永久的删除这个文件(包括该文件的历史记录).

First, 你可以参考Github官方给出的 [方案文档](https://help.github.com/articles/remove-sensitive-data/)

当然，如果不想去看英文文档，或者说文档说的太多，下面给出总结的步骤：
<!-- more -->
* 从你的资料库中清除文件
切换到git管理的项目的根目录,执行下面命令,注意path-to-your-remove-file是相对于项目根目录的路径，千万不要弄错了。
``` shell
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch path-to-your-remove-file' --prune-empty --tag-name-filter cat -- --all
```

执行后你会看到以下输出就说明成功了
``` shell
......

Rewrite a36d6cf0ca29e3edbd91d9764853c75f9086d8f3 (56/61) (5 seconds passed, remaining 0 predicted)    rm 'source/_posts/file.md'
Rewrite 9d69424c20077d5732f81c6e818658643a28324f (56/61) (5 seconds passed, remaining 0 predicted)    rm 'source/_posts/file.md'
Rewrite 7680c23365e2348ca3a4b62936f2cb869110e29e (56/61) (5 seconds passed, remaining 0 predicted)    rm 'source/_posts/file.md'

Ref 'refs/heads/master' was rewritten
Ref 'refs/remotes/origin/master' was rewritten
```

* 推送到远程仓库
这一步得强制执行
``` shell
git push origin master --force
```

* 清理和回收空间
虽然上面我们已经删除了文件,但是我们的仓库里面仍然保留了这些objects, 等待垃圾回收(GC), 所以我们要用命令彻底清除它, 并收回空间.
依然到项目根目录，依次执行下面命令
``` shell
rm -rf .git/refs/original/
```

``` shell
git reflog expire --expire=now --all
```

``` shell
git gc --prune=now
```
你将看到以下输出：
``` shell
Counting objects: 4516, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2651/2651), done.
Writing objects: 100% (4516/4516), done.
Total 4516 (delta 1698), reused 0 (delta 0)
```

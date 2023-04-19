---
title: git操作
date: 2020.07.13 17:31:28
tag: git
categories: Java
---
<meta name="referrer" content="no-referrer" />
基本操作就不说了，这里详细介绍其他中高级操作
##### 1、查看日志
```
git log
```
 
这个命令对于提交少的项目比较友好，但是对于提交很多的项目，会输出一大长串，很不方便，于是有一个比较简便的命令：
```
git log --pretty=oneline
```
输出：
![git log](https://upload-images.jianshu.io/upload_images/15200008-45ddb710fbaa4feb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 2、撤销修改
**背景**
在一个文件中添加了一点代码，现在我`git add .`了但是没提交，现在我想撤销我修改的东西

![想要撤销修改](https://upload-images.jianshu.io/upload_images/15200008-ef3f18e06870f830.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**解决方案**
两个命令：
```
git reset HEAD  src/main/java/com/tinner/demo03/constant/RedisContant.java
git checkout --  src/main/java/com/tinner/demo03/constant/RedisContant.java

```
**效果**
![效果](https://upload-images.jianshu.io/upload_images/15200008-4f6a48f749025f40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 3、创建分支
两个命令：
```
git checkout -b dev
git switch -c dev
```
##### 4、删除分支
```
git branch -d <name>
```
如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除。
##### 5、合并代码
```
git merge --no-ff -m "注释。。。。" <分支名>
```
##### 6、stash
**背景**
在一个新的需求开发过程中，我在dev分支开发了一些东西，但是并没有提交我也不想提交，这个时候来了一个bug，我必须将bug改完才能进行dev的开发，那么这个时候就需要用到`git stash`命令
**步骤**
- 首先，我看看git仓库中的状态
![git状态](https://upload-images.jianshu.io/upload_images/15200008-a972ff6bedf30c71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，在dev分支，我创建了一个文件，修改了一个文件。
- 然后，使用`git stash`命令，之后再来查看状态：
![储藏代码](https://upload-images.jianshu.io/upload_images/15200008-8e9e83fcd28a6c6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此时工作区是干净的。然后我切换到master分支，创建一个issue分支，解决bug，合并到master，发版，之后再切换到dev分支，可以看到我的工作区还是干净的，那么之前的代码跑哪去了？
- 使用`git stash list`可以查看储藏的代码列表
![git stash list](https://upload-images.jianshu.io/upload_images/15200008-3b3f6b8f63f04ea9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 现在我们需要恢复之前的代码，有两个命令可以参考：
```
git stash apply
git stash pop
```
`git stash apply`命令恢复后，stash内容并不删除，你需要用`git stash drop`来删除；
![git stash apply](https://upload-images.jianshu.io/upload_images/15200008-b8880068ad06e4c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

另一种方式是用`git stash pop`，恢复的同时把stash内容也删了

![git stash pop](https://upload-images.jianshu.io/upload_images/15200008-41d742ccc7ace756.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令:
```
git stash apply stash@{0}
```
**扩展**
我们知道，master的bug修复之后，在现在的dev代码相关的bug并没有修复，所以，这个bug其实在当前dev分支上也存在。
*如何快速修复？*
同样的bug，要在dev上修复，我们只需要把`修复issuebug`这个提交所做的修改“复制”到dev分支。注意：我们只想复制`修复issuebug`这个提交所做的修改，并不是把整个master分支`merge`过来。

为了方便操作，Git专门提供了一个`cherry-pick`命令，让我们能复制一个特定的提交到当前分支：
```
git cherry-pick <版本号>
```
![修复bug](https://upload-images.jianshu.io/upload_images/15200008-30ceb22e3d0b5e90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/15200008-81c651ad31eed27d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用`git cherry-pick`，我们就不需要在dev分支上手动再把修bug的过程重复一遍。

### 几个命令的区别
##### merge和rebase的区别
merge和rebase都是用来合并分支的。
- 采用merge和rebase后，git log的区别，**merge命令不会保留merge的分支的commit**：

![merge和rebase](//upload-images.jianshu.io/upload_images/6081878-f7c90a58674a5584.png?imageMogr2/auto-orient/strip|imageView2/2/w/895)

- 处理冲突的方式：
`（一股脑）`使用merge命令合并分支，解决完冲突，执行git add .和git commit -m 'fix conflict'。这个时候会产生一个commit。
`（交互式）`使用rebase命令合并分支，解决完冲突，执行git add .和git rebase --continue，不会产生额外的commit。这样的好处是"干净"，分支上不会有无意义的解决分支的commit；坏处：如果合并的分支中存在多个commit，需要重复处理多次冲突。
##### git pull和git pull --rebase
`git pull`做了两个操作分别是`获取`和`合并`。所以加了`rebase`就是以`rebase`的方式进行合并分支，默认为`merge`。
##### git merge 和 git merge --no-ff的区别
`git merge –no-ff` 可以保存你之前的分支历史。能够更好的查看 merge历史，以及branch 状态。
`git merge` 则不会显示 feature，只保留单条分支记录。
![git merge 和 git merge --no-ff的区别](https://upload-images.jianshu.io/upload_images/15200008-f7ea47cfb59ac349.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### tag相关
在当前分支的最新commit上打tag
```
git tag v1.0
```
在f52c633版本的提交上打tag
```
git tag v0.9 f52c633
```
查看所有的tag
```
git tag
```
*注意，标签不是按时间顺序列出，而是按字母排序的*

还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：
```
git tag -a v0.1 -m "文字说明" 1094adb
```
用命令`git show <tagname>`可以看到说明文字
提交v1.0标签
```
git push origin v1.0
```
提交所有的标签
```
git push origin --tags
```
![标签](https://upload-images.jianshu.io/upload_images/15200008-57c8edc0e40ec346.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![tag](https://upload-images.jianshu.io/upload_images/15200008-5e47e69158aed78a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从本地删除tag：
```
git tag -d v0.9
```
删除远程tag：
```
git push origin :refs/tags/v0.9
```
![tag删除](https://upload-images.jianshu.io/upload_images/15200008-45ac44d2f7a193ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### Git HEAD^与HEAD~的关系
一张图：
![Git HEAD^与HEAD~的关系](https://upload-images.jianshu.io/upload_images/15200008-57f1b8f25ab5d35a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### Git reset后面跟的参数
比如我在我的本地目录下面新建了一个文件c.txt，add到了我的暂存区（缓冲区），同时也commit提交到了本地仓库
默认：`git reset` = `git reset --mixed`，这个命令会跳转到指定版本、**缓存区的文件还原、但是工作区（本地目录）下的文件不会还原**
`git reset --hard`这个命令会跳转到指定版本、**缓存区的文件还原、工作区（本地目录）下的文件也会被还原**
`git reset --soft`这个命令会跳转到指定版本、**缓存区的文件不会被还原、而且工作区（本地目录）下的文件也不会还原**
### git revert
`git revert`会产生一个commit，撤销某次操作，此次操作之前和之后的commit和history都会被保留，并且会把这次撤销作为一次最新的提交。







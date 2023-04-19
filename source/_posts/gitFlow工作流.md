---
title: gitFlow工作流
date: 2020.07.01 17:01:03
tag: git
categories: Java
---
<meta name="referrer" content="no-referrer" />

![git工作流](https://upload-images.jianshu.io/upload_images/15200008-742c050e94084a3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Gitflow工作流通过为功能开发、发布准备和维护（bug修复）分配独立的分支，让发布迭代过程更流畅。严格的分支模型也为大型项目提供了一些非常必要的结构。
![git工作流](https://upload-images.jianshu.io/upload_images/15200008-5dcea875b5e3537f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
虽然有这么优秀的版本管理工具，但是我们面对版本管理的时候，依然有非常大得挑战，我们都知道大家工作在同一个仓库上，那么彼此的代码协作必然带来很多问题和挑战，如下：
 
- 如何开始一个Feature的开发，而不影响别的Feature？
- 由于很容易创建新分支，分支多了如何管理，时间久了，如何知道每个分支是干什么的？
哪些分支已经合并回了主干？
- 如何进行Release的管理？开始一个Release的时候如何冻结Feature, 如何在Prepare Release的时候，开发人员可以继续开发新的功能？
- 线上代码出Bug了，如何快速修复？而且修复的代码要包含到开发人员的分支以及下一个Release?
- 大部分开发人员现在使用Git就只是用三个甚至两个分支，一个是Master, 一个是Develop, 还有一个是基于Develop打得各种分支。这个在小项目规模的时候还勉强可以支撑，因为很多人做项目就只有一个Release, 但是人员一多，而且项目周期一长就会出现各种问题。

# 一、Git Flow常用的分支

**Master 分支**
这个分支最近发布到生产环境的代码，最近发布的Release， 这个分支只能从其他分支合并，不能在这个分支直接修改
**Dev 分支**
这个分支是我们是我们的主开发分支，包含所有要发布到下一个Release的代码，这个主要合并与其他分支，比如Feature分支
**Feature 分支**
这个分支主要是用来开发一个新的功能，一旦开发完成，我们合并回Develop分支进入下一个Release
**Release分支**
当你需要一个发布一个新Release的时候，我们基于Develop分支创建一个Release分支，完成Release后，我们合并到Master和Develop分支
**Hotfix分支**
当我们在Master发现新的Bug时候，我们需要创建一个Hotfix, 完成Hotfix后，我们合并回Master和Dev分支，所以Hotfix的改动会进入下一个Release

# 二、Git Flow如何工作

## 1. Master分支

所有在Master分支上的Commit应该Tag
![master分支](https://upload-images.jianshu.io/upload_images/15200008-58d45fc3dd13f764.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2. Feature 分支

Feature分支做完后，必须合并回Develop分支, 合并完分支后一般会删点这个Feature分支，但是我们也可以保留

![Feature分支](https://upload-images.jianshu.io/upload_images/15200008-80e1a93d016f969a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每个新功能位于一个自己的分支，这样可以push到中央仓库以备份和协作。 但功能分支不是从master分支上拉出新分支，而是使用develop分支作为父分支。当新功能完成时，合并回develop分支。 新功能提交应该从不直接与master分支交互。

## 3. Release分支

Release分支基于Develop分支创建，打完Release分之后，我们可以在这个Release分支上测试，修改Bug等。同时，其它开发人员可以基于开发新的Feature (记住：一旦打了Release分支之后不要从Develop分支上合并新的改动到Release分支)

发布Release分支时，合并Release到Master和Develop， 同时在Master分支上打个Tag记住Release版本号，然后可以删除Release分支了。

一旦develop分支上有了做一次发布（或者说快到了既定的发布日）的足够功能，就从develop分支上checkout一个发布分支。 新建的分支用于开始发布循环，所以从这个时间点开始之后新的功能不能再加到这个分支上—— 这个分支只应该做Bug修复、文档生成和其它面向发布任务。 一旦对外发布的工作都完成了，发布分支合并到master分支并分配一个版本号打好Tag。 另外，这些从新建发布分支以来的做的修改要合并回develop分支。

使用一个用于发布准备的专门分支，使得一个团队可以在完善当前的发布版本的同时，另一个团队可以继续开发下个版本的功能。 这也打造定义良好的开发阶段（比如，可以很轻松地说，『这周我们要做准备发布版本4.0』，并且在仓库的目录结构中可以实际看到）。
![release分支](https://upload-images.jianshu.io/upload_images/15200008-15c5b67abe8ead6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4. Hotfix分支

hotfix分支基于Master分支创建，开发完后需要合并回Master和Develop分支，同时在Master上打一个tag
![Hotfix分支](https://upload-images.jianshu.io/upload_images/15200008-c4a6e09f61baf2dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
该分支用于生成快速给产品发布版本（production releases）打补丁，这是唯一可以直接从master分支fork出来的分支。 修复完成，修改应该马上合并回master分支和develop分支（当前的发布分支），master分支应该用新的版本号打好Tag。

为Bug修复使用专门分支，让团队可以处理掉问题而不用打断其它工作或是等待下一个发布循环。 你可以把维护分支想成是一个直接在master分支上处理的临时发布。

# 三、如何进行测试

**Testing On Feature Branch**
开发人员在功能分支开发完成后，提测给对应测试人员，标明其对应的功能分支，测试人员发现bug，开发人员仍在该分支修改bug，修改完交给测试人员测试，当测试通过后，开发人员发起一个pull request合并会develop分支，然后从develop检出release，测试人员在预发环境部署并快速回归测试，发现bug，开发人员直接在release分支进行修正，测试通过后，将release合回develop并合并到master准备上线。





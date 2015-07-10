---
layout: post
title: "git使用总结"
date: 2015-03-21 11:19:25 +0800
comments: true
keywords: git 总结
categories: iOS开发
---
##本文只是一些个人使用上的总结,命令操作平台为Mac的终端.

<!-- more -->


##1.初始化
通常有两种方式来进行初始化：

1）`git clone`是一种较为简单的初始化方式，当你已经有一个远程的git版本库时，只需要在本地克隆一份即可。
例如在终端中输入:

```
git clone git://github.com/someone/some_project.git   some_project
```
上面的命令就是将远程版本库克隆到本地电脑的`some_project`目录下。

2）`git init`和`git remote`。这种方式稍微复杂一些。当你本地创建了一个工作目录，你可以进入这个目录，使用`git init`命令创建一个git版本管理库。这时候如果需要将工程放到远程服务器上，可以在远程服务器上创建一个目录，并把可访问的URL记录下来，此时就可以使用`git remote add`命令来增加一个远程服务器版本了。例如：

```
git remote add origin git://github.com/someone/another_project.git
```
上面的命令就会增加该URL地址且名称为`origin`的远程服务器。以后提交的代码只需要使用`origin`这个别名即可。


##2.1分支操作
查看本地分支:`git branch`

查看远程分支:`git branch -r`

查看所有的分支:`git branch -a` 

创建本地分支:`git branch name` 注意新分支创建后不会自动切换为当前分支

切换分支:`git checkout name`

这里需要注意的一点就是，如果是要同步远程的分支（比如同事建立了新的分支，以保证针对某个版本的修改在该分支下），请不要在本地新建一个跟远端同名的分支，也就是说不要使用`git branch name`这个命令，而是直接使用`git checkout name`命令把远端的分支拉下来即可。以避免把新的代码合并到旧的分支里。


创建新分支并立即切换到新分支:`git checkout -b name`

删除分支:`git branch -d name`,`-d`选项只能删除已经参与了合并的分支，对于未有合并的分支是无法删除的。如果想强制删除一个分支，可以使用`-D`选项

合并分支:`git merge name`将名称为`name`的分支与当前分支合并，是合并到当前分支，而不是合并到`name`分支。要注意的合并前先使用`git branch`查看下当前所处的分支。

可以在后面加参数，比如我要把`issue340`分支合并到`dev`分支，可以先`checkout`到`dev`分支并使用以下命令：

```
git merge issue340 -n --ff

```

后面的参数`-n --ff`据说是为了分支合并时更快速直接？具体还有待研究。

重命名分支：

`git branch –m oldname newname`

`-m`不会覆盖已有分支名称，即如果名为newname的分支已经存在，则会提示已经存在了。

如果改成`-M`就可以覆盖已有分支名称了，即会强制覆盖名为newname的分支，这种操作要谨慎。

创建远程分支(本地分支push到远程):`git push origin name`

删除远程分支:`git push origin :heads/name` 或 `git push origin : name`。

####这里需要注意的是，在开发新功能、测试或者重构部分代码时，最好是新建一条分支来操作，测试新功能OK后再合并回主分支，以避免干扰到主分支。

##2.2合并分支

注意没参数的情况下merge是fast-forward的，即Git将master分支的指针直接移到dev的最前方。
换句话说，如果顺着一个分支走下去可以到达另一个分支的话，那么Git在合并两者时，只会简单移动指针，所以这种合并成为快进式(Fast-forward)。

介绍两个合并分支时常用的指令

1.`--squash`

将一条分支上的若干个提交条目压合成一个提交条目，提交到另一条分支的末梢。此时，dev上的所有提交已经合并到当前工作区并暂存，但还没有作为一个提交，可以像其他提交一样，把这个改动提交到版本库中：

`git commit –m “something from dev”`


`--squash`指令含义是：本地文件内容与不使用该选项的合并结果相同，但是不提交、不移动HEAD，因此需要一条额外的commit命令。其效果相当于将another分支上的多个commit合并成一个，放在当前分支上，原来的commit历史则没有拿过来。


判断是否使用`--squash`选项最根本的标准是，待合并分支上的历史是否有意义。

如果在开发分支上提交非常随意，甚至写成微博体，那么一定要使用`--squash`选项。版本历史记录的应该是代码的发展，而不是开发者在编码时的活动。


2.`—no-ff`

`--no-ff`指的是强行关闭fast-forward方式。

fast-forward方式就是当条件允许的时候，git直接把HEAD指针指向合并分支的头，完成合并。属于“快进方式”，不过这种情况如果删除分支，则会丢失分支信息。因为在这个过程中没有创建commit.

`git merge --squash branch` 是用来把一些不必要commit进行压缩，比如说，你的feature在开发的时候写的commit很乱，那么我们合并的时候不希望把这些历史commit带过来，于是使用--squash进行合并，此时文件已经同合并后一样了，但不移动HEAD，不提交。需要进行一次额外的commit来“总结”一下，然后完成最终的合并。


总结：

`--no-ff`：不使用fast-forward方式合并，保留分支的commit历史
`--squash`：使用squash方式合并，把多次分支commit历史压缩为一次


合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。示意图如下：


![](/images/2015/other/branch_1.png)   

![](/images/2015/other/branch_2.png)






##3.版本操作
查看版本:`git tag`

创建版本:`git tag name`

删除版本:`git tag -d name`



##4.撤销文件更改
`git checkout`有两个作用，其中一个功能是在不同的`branch`之间进行切换，例如`git checkout another_branch`就会切换到`another_branch`的分支上去。

另一个功能是还原代码的作用，例如`git checkout app/model/user.rb`就会将`user.rb`文件从上一个已提交的版本中更新回来，未提交的内容全部会回滚到改动前的状态。



##5.查看git配置
`git config --list`该指令可以查看所有关于git的配置，用户和邮箱，远程库的URL等。



##6.版本回滚
git版本管理的好处就在于有无限的反悔权。`git reset HEAD^`就會回到前一版本(一個^表示是前一版)，並把其中的 changes 繼續留在 working tree 中。適合發現前一次 commit 有問題或是想要修改 commit log，可以修改後再重新 commit。

`git reset`如果加上`–soft`參數則會把 changes 直接加到 staging area（暂存区）。加上`–hard`參數表示不留 staging area 也不留 working tree(完全刪除任何修改記錄)。例如`git reset --hard`指令会清楚所有与最近一次commit不同的修改，也就是说放弃当前所有的更改。

如果在合并（merge）过程中发生冲突了，想放弃这次合并，也可以使用`git reset --hard`来取消。

`git reset --hard ORIG_HEAD`指令会取消最近一次成功的合并以及所有你在这次合并后所做的修改。


##7.修改上一次提交的信息

有时候手抖，在还没有输入完提交信息的时候按了回车，可以用如下指令修改提交的信息：

```
git commit -m"your message" --amend
```

##8.单个文件回滚
有时候为了改了一个功能，没完全测试OK就推到服务器了，发现改的功能还不如之前的好用怎么办？
有两种情况，第一种是，在你提交错误的代码到服务器后，队友还有没有提交过，可以使用该指令：

```
git reset --hard commit_id
```
其中`commit_id`是提交记录的代号，可以使用指令:

```
git log
```

来查看所有的提交记录，找到你需要回滚的版本即可（键盘方向键可滚动查看所有记录，鼠标目测不行）。

这里要注意的是，git会回滚到你最近一次提交的`commit_id`之后的版本，而不是`commit_id`之前的版本。在本次操作中，由于队友都没有更新，只需要找到当前提交的上一次的`commit_id`即可。比如今天是3月21日，你提交了错误的代码到服务器，服务端在之前3月20日有一次更新，那你找到3月20日的`commit_id`即可。

第二种情况是，如果你提交了错误代码后队友也有新的提交，直接版本回退会把队友新提交的代码也搞没了，这时候如果你改动的文件不多的话，可以使用单个文件回滚的方式：

1)执行`git log test.h`查看该文件的提交记录，找到需要回滚的commit版本，复制版本号。

2)执行`git reset “commit版本号” test.h`。

3)继续执行`git checkout test.h`，文件回滚成功。

需要注意的是，如果文件在比较深的目录下，上述操作室需要输入目录结构的，比如`git log aaa/bbb/test.h`。

##其他

1.提交代码的通常步骤`git status`查看当前状态，有改动则使用`git add.`把所有改动添加到本地仓库or暂存区(有待研究)？然后`git commit -m"your commit message"`后，先`git pull`拉下服务器的最新代码，确认没有问题后可以`git push`提交代码。

2.`git pull`之后，或者进入其他界面，都可使用`:wq`退回到git的操作界面

3.如果在`git pull`或者`git push`时，提示没有设置pull或者push的默认分支，按照提示的指令设置默认分支即可。

###再次强调，在开发新功能、测试或者重构部分代码时，最好是新建一条分支来操作！

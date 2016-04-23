---
layout: post
title: "将Octopress博客从Github迁移到Gitcafe或coding"
date: 2015-04-27 21:31:02 +0800
comments: true
categories: iOS开发
keywords: Octopress github gitcafe coding
---

本文记录了将基于octopress 和github pages搭建的博客迁移到国内代码托管服务商gitcafe的过程。
搭建博客的过程在[前文](http://www.zoomfeng.com/blog/ji-yuOctopressda-jian-bo-ke.html)有介绍。

将博客放在github上虽然很方便，也很有Geek的格调，但是很不爽的就是，github这个网站访问很慢，导致我们的博客访问也很慢，我测试的时候发现开机后第一次加载的话花个30秒是正常的，第二次以后访问会好一些，但相比国内的网站还是很慢。故有了这次博客迁移之旅。

本博客迁移的过程并不轻松，经过跟gitcafe工程师十几封邮件后的沟通后才算完全搞掂。

补充：由于gitcafe即将关闭服务，所以又把博客迁移到了coding.net。具体迁移方式跟迁移到gitcafe大致一致，下文有标注。

<!--more-->

##迁移原理
我们并非完全放弃github，只是把博客多备份一个副本到gitcafe上而已，然后在自己域名的管理网站上，把自己的域名指向gitcafe备份的那一份博客，从而加快访问速度。

另由于octopress有两个分支，soure和master，通常的写博客在source分支下，master分支的内容是用来存放网页供他人访问的。我们在备份到gitcafe后一切操作跟备份之前一样。

在本次迁移之前，我去[GoDaddy](www.godaddy.com)上面买了个域名，三年不到￥200，感觉还是蛮便宜的。

备注：GoDaddy支持支付宝付款，买域名很方便，买的时候，那些附加的东西可以统统不要勾选。

##详细步骤

1.注册[gitcafe](www.gitcafe.com)账号，这个必须的。

2.在刚才的账号下，添加SSH公钥。这一步跟github上的添加公钥几乎一致。

3.创建一个与你的gitcafe用户名```相同名称```的项目。记住，如果你需要通过gitcafe的pages功能，请保证这个项目的名称与你的用户名完全一致，否则gitcafe识别不了！本人就犯过这个错误，导致长时间博客无法访问！

4.打开octopress下的```_deploy```目录，添加一个gitcafe的远程仓库：

```
git remote add gitcafe git@gitcafe.com:username/username.git
```

将上述的username用你的gitcafe用户名替换掉。

补充：迁移到coding也是类似的方式，不过之前的remote指向gitcafe，删掉再重新添加remote到coding即可，具体coding的pages功能页有教程。添加公钥什么的也是类似。

5.修改Rakefile，以实现发布到github的时候，也顺便备份一次到gitcafe的服务器。
只需要在第269行增加如下代码：

```
system "git remote add gitcafe git@gitcafe.com:username/username.git >> /dev/null 2>&1"
system "git push -u gitcafe master:gitcafe-pages"
```
将上述的username用你的用户名替换掉。

补充：如果是博客迁移到coding的，就按照下面的方式来更改，其实是一样的。需要注意的一点是，按照coding上的教程，新建了一个coding-pages 的分支用来做为博客的备份分支。注意该分支也是在_deploy目录下的git仓库建立的啊，不要傻傻的跑去octopress目录下的那个仓库。

```
system "git remote add git@git.coding.net:username/username.git >> /dev/nul$
system "git push -u origin coding-pages"
```



具体添加的位置请看[这里](http://blog.devtang.com/blog/2014/06/02/use-gitcafe-to-host-blog/);

另外如果你想加些提示信息，也可以在上述代码插入的前一行或后一行，添加一些说明。
添加的说明的格式在Rakefile里可以看到，大概是这样子：

```
puts "\n## add your message"
```

到这里，配置博客部分就完成了，`rake gen_deploy`操作可以推送博客内容到gitcafe和github两个地方了。

补充：以上改好以后，`rake gen_deploy` 就可以推送博客内容到coding和github两个地方了。

6.修改域名指向。

补充：coding的pages服务，也是要先去pages服务的栏目添加域名绑定，然后去godaddy将域名指向为`pages.coding.me` 即可。

如果你之前只是使用`username.github.io`这样的域名来访问博客的话，这一步可以跳过，直接去测试`username.gitcafe.io`即可。

如果使用了自己的域名来配置过github的博客，请继续往下。

第一步，找到你在gitcafe的项目，点击项目设置-》pages服务，添加你的域名。

第二步去修改域名配置。由于之前把博客配置在github时，使用了CNAME来作为域名跳转，而gitcafe的方式跟github不大一样，故需要做如下改动：

①删掉octopress目录下的CNAME文件

②去GoDaddy网站（或者你的其他域名管理网站）修改域名指向。
如下图所示：

![](/images/2015/04/27/01.png)

添加一个A记录指向gitcafe的服务器即可，没必要把所有的A记录都删掉只剩下gitcafe的，我的A记录就有两个，尽管另一个我也不知道是干嘛用的。

备注：本来按照gitcafe的文档提示，只需要添加CNAME或者A记录其中一项即可，但我在迁移过程中，发现需要两个一起配置才行，gitcafe工程师解释的原因如图：

![](/images/2015/04/27/03.png)


![](/images/2015/04/27/02.png)

我的博客是有`www`这个前缀的，所以我乖乖的添加了一条CNAME。

这两次搭建和迁移博客给了我比较大的体会就是，遇到有问题时，是可以找网站技术人员寻求支持的。我之前在搭建博客时，也遇到过问题，通过给github发邮件，问题得到了解决，这次也一样，跟gitcafe邮件沟通后问题也得到了解决。

祝大家顺利。


参考链接：

1.[将 Octopress 博客从 GitHub 迁移到 GitCafe](http://blog.leichunfeng.com/blog/2014/11/15/migrate-octopress-blog-from-github-to-gitcafe/).

2.[将博客从GitHub迁移到GitCafe](http://git.devzeng.com/blog/change-blog-host-to-gitcafe.html).

3.[将博客从GitHub迁移到GitCafe](http://blog.devtang.com/blog/2014/06/02/use-gitcafe-to-host-blog/).

4.[gitcafe官方帮助文档](https://gitcafe.com/GitCafe/Help/wiki/Pages-%E7%9B%B8%E5%85%B3%E5%B8%AE%E5%8A%A9#wiki).
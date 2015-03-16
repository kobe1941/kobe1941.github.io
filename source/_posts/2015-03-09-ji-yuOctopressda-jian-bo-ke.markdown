---
layout: post
title: "基于Octopress搭建博客"
date: 2015-03-09 14:52:24 +0800
comments: true
keywords: Octopress github 搭建博客
categories: 
---
##本文介绍在Mac下利用Octopress和Github Pages搭建个人博客的流程

<!-- more -->

##1.安装一些环境依赖
先安装git(如果你的电脑没有git)，打开终端输入
```
git version
```
可以查看当前git的版本号，一般说来确定本机安装了git就可以了。

然后安装ruby，因为Octopress是用ruby语言写的，所该该框架是必须的。
具体过程为：

```
ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
```
这一步是为了安装HomeBrew，这是一个软件包管理系统，安装它是为了安装ruby作准备。
安装好HomeBrew后先执行以下命令检查看是否安装妥当：

```
brew doctor
```
如果顺利终端会显示：

```
Your system is ready to brew.
```
好了，软件管理包装好了，接下来可以正式安装ruby，安装ruby有两种方式：rvm和rbenv。本文采用rvm方式，rvm其实是ruby version manager的简称。
执行以下命令：

```
curl -L https://get.rvm.io | bash -s stable --ruby
```
然后执行以下命令：

```
rvm install 2.0.0
rvm 2.0.0
rvm rubygems latest
```
2.0.0表示安装的ruby的版本，Octopress需要1.9.3以上，你也可以试着用其他的版本，只要大于等于1.9.3即可。安装好之后可以执行：

```
ruby --version
```
查看当前ruby的版本号。

##2.本地安装Octopress
这一步非常简单，一般不会有什么大的问题。
打开终端，执行命令：

```
git clone git://github.com/imathis/octopress.git octopress
```
以上命令的作用是把Octopress克隆到本地磁盘，并生成一个名为octopress的文件夹
然后进入该文件夹，并安装一些依赖项：

```
cd octopress
gem install bundler
bundle install
```
其中`gem install bundler`这条指令在根目录安装也是可以的，但是`bundle install`这条指令请在octopress目录下安装，否则可能会出错。
然后安装默认的博客主题：

```
rake install
```
rake就是ruby make的缩写，到此Octopress在本地就安装完毕啦。

可以执行：

```
rake preview
```
然后在浏览器输入`http://localhost:4000/`可以预览刚刚搭建完成的博客，没有任何博文，但是可以看到主题风格了。

Tips:以上步骤如果遇到权限问题，比如提示permission not enough之类，请在指令前面加上sudo和空格，重新执行一遍，sudo是指电脑的最高权限，相当于windows的管理员权限。

##3.配置Github Pages
这一步比较简单，主要是你必须要有一个github账号，并建立一个仓库用来存放你的博客。具体步骤请看[这里](http://shengmingzhiqing.com/blog/setup-octopress-with-github-pages.html/)，这里需要注意的是，使用密钥来对github的日常操作进行加密是一个好习惯，通常又称之为ssh。
该步骤为打开终端输入：

```
ssh-keygen
```
然后在隐藏的.ssh/目录下会生成两个文件：`id_rsa和id_rsa.pub`。

.pub的文件是公钥，另一个是私钥，加密用的。用文本编辑打开公钥，可复制里边的内容，然后去github上的设置栏/SSH添加一个公钥(请把.pub文件里的内容复制到内容的栏目，切勿复制到title栏目上)，以后你的同步和上传博客操作就不需要输入github的密码啦。

另Mac下显示隐藏文件的命令为：

```
defaults write com.apple.finder AppleShowAllFiles -bool true
```

##4.将本地的博客发布到Github Pages

打卡终端，执行以下命令：

```
cd octopress
rake setup_github_pages
```
然后要输入github上存放博客的仓库的位置，类似:`git@github.com:[your_username]/[your_username].github.io.git`这种。
以上步骤成功后，接着输入：

```
rake generate
rake deploy
```
第一个命令用来生成静态页面，第二个命令用来把写好的博客部署到github上。上传完成后，浏览器打开：`http://[your_username].github.io/`
你将看到跟刚才在本地预览时相同的页面。

最后不要忘了把源文件备份到github的source分支下：

```
git add .
git commit -m"你的提交内容"
git push origin source
```
请注意，搭建该博客系统的过程中，github的仓库会有两个分支，一个是master，一个是source，我们写博客发博文的操作都在source分支下即可。source分支是存放博客源文件用的，master是存放部署好的静态页面给其他用户用浏览器访问用的。

##5.发布新的博文
打开终端，输入：

```
cd octopress
rake new_post["your title"]
```
这条指令可以新建一条博文。为了使用顺利，新手请暂时不要用中文命名，等流程全部跑通后再解决中文编码的问题(不过我倒是没遇到中文编码的问题，也许是ruby版本比较高的原因罢)。
以上命令执行后，会再`octopress/source/_posts`目录下生成一个`yyyy-mm-dd-your title.markdown`的文件，用文件编辑器可以编辑，但是不能预览效果。可以使用专用的markdown编辑器来编辑，比如[Mou](http://25.io/mou/)。
编辑完之后保存，使用指令：

```
rake gen_deploy
```
可以把该博文部署到github上。这条指令其实是

```
rake generate
rake deploy
```
这两条指令的合体。

最后请记得，以上指令只是把博文的内容发布到github上，也就是说博文内容在master分支下，还需要做一个操作：

```
git add .
git commit -m"your message"
git push origin source
```
以备份博文到github的source分支下。

再次强调，source分支用来管理日常编辑和备份，master分支的内容是用来支撑其他网友访问博客用的。

以上步骤一般可以成功搭建一个普通博客了，本人在实践中遇到了一个问题，就是在输入指令：

```
rake deploy
```
时发现分支同步上出现了问题，最后也得到了解决，具体解决方案是：

```
cd _deploy
git pull origin master
```
该命令其实是同步用来发布的内容，注意哦，这里同步的是master分支，因为这是发布到github上给其他用户访问的。更多详细请见我在V2EX上的[帖子](http://v2ex.com/t/175120#reply13)。

######总结：搭建这个博客，需要对git版本管理比较熟悉（markdown语法不会没关系，不影响搭博客，当然以后要更黑客般的写博客肯定还是要学习markdown的），然后其他部分Google别人教程基本都可以解决。新手需要注意的是，搭博客有很多种方式，比如wordpress、octopress、hexo等等，在搜索资料的时候，请先确定使用哪一种，以避免混淆具体操作的步骤。



##6.在多台电脑上同步写博客
需要注意的是，另一台电脑也要安装git和ruby等环境，但Octorepress就不用了，因为会从github上克隆下来。具体请参考[这里](http://boboshone.com/blog/2013/06/05/write-octopress-blog-on-multiple-machines/)。

##7.配置博客

####7.1 添加Google统计

①去[这里](http://www.google.com/analytics/)注册Google Analytics账号，如果已有Gmail可以直接使用Gmail账号登陆。

②按照Google的提示，可以获取一个google_analytics_tracking_id，复制此id，然后打开本地的_config.yml文件(终端下nano命令可直接打开)，找到google_analytics_tracking_id，在冒号后填入刚刚复制的内容(冒号后记得隔一个空格再填写)。

③按照Google的提示，对该博客网站进行验证。只需要将Google提供的用于验证的代码添加到`source/_includes/head.html`文件的一对`<head></head>`之间即可。部署之后一段时间就能通过验证。
之后在[这里](https://www.google.com/analytics/web/?hl=zh-CN&pli=1#report/visitors-overview/a60680038w95181548p99221362/%3Foverview-graphOptions.primaryConcept%3Danalytics.pageviews/)即可看到你博客的访问信息了，统计还蛮全面的，除了访问量这个数字其他也暂时不知道有啥用。


####7.2 侧边栏添加微博秀
首先从[新浪微博秀](http://app.weibo.com/tool/weiboshow)获取自定义的微博秀代码，调整好微博秀的颜色和样式后复制代码，然后在`source/_includes/asides`目录下新建`weibo.html`文件（用终端touch指令即可完成新建）。按照如下格式编辑该文件(nano指令):

```
<section>
    <h1>新浪微博</h1>
    <ul id="weibo">
    <li>

   -- 在此插入获得的微博秀代码 --

      </li>
    </ul>
</section>
```

把刚刚复制的微博秀代码粘贴上去。
接着用终端打开 `_config.yml` 文件，找到 `default_asides` 项，添加`asides/weibo.html`到中括号里即可。
微博秀不一定能预览，但部署到github后稍等一下一般都可正常显示。
另需要注意的是关于微博秀的配置信息是可以直接在复制微博秀代码的时候，进行改动的，读一下代码，可以选择粉丝显示或不显示，显示的行数等等。


####7.3 导航栏添加栏目并制定跳转链接
默认的导航栏只有Blog和Archives两个选项，本博客就添加了一个我的豆瓣栏目，具体可以编辑文件`source/_includes/custom/navigation.html`，仿照加一段代码即可:

```
  <li><a href="http://www.douban.com/people/你的豆瓣id/">我的豆瓣</a></li>
```


####7.4 添加多说评论系统
去[多说](http://duoshuo.com/create-site/)注册，获取站点的short_name（其实就是多说域名中间的那一段）。
然后在`_config.yml`文件的最后添加如下内容:

```
duoshuo_comments: true
duoshuo_short_name: yourname
```
用你的short_name替换上述的`yournamw`。

接着在`source/_layouts/post.html`中添加多说评论模块:

```
｛% if site.duoshuo_short_name and site.duoshuo_comments == true and page.comments == true %｝
  <section>
    <h1>Comments</h1>
    <div id="comments" aria-live="polite">｛% include post/duoshuo.html %｝</div>
  </section>
｛% endif %｝
```
最后新建`source/_includes/post/duoshuo.html`文件,并填入如下内容:

	<!-- Duoshuo Comment BEGIN -->
	<div class="ds-thread"></div>
	<script type="text/javascript">
	  var duoshuoQuery = {short_name:"your_name"};
	  (function() {
	    var ds = document.createElement('script');
	    ds.type = 'text/javascript';ds.async = true;
	    ds.src = 'http://static.duoshuo.com/embed.js';
	    ds.charset = 'UTF-8';
	    (document.getElementsByTagName('head')[0]
	    || document.getElementsByTagName('body')[0]).appendChild(ds);
	  })();
	</script>
	<!-- Duoshuo Comment END -->

将上述内容中的`your_name`用你的short_name替换。



####7.5 添加社会化分享 
本博客采用bshare,在`_config.yml`中增加

```
bshare: true
```
然后在`source/_includes/post`下的sharing.html中添加如下代码:

	
	<div class="bshare-custom"><a title="分享到QQ空间" class="bshare-qzone"></a><a title="分享到新浪微博" class="bshare-sinaminiblog"></a><a title="分享到人人网" class="bshare-renren"></a><a title="分享到腾讯微博" class="bshare-qqmb"></a><a title="分享到网易微博" class="bshare-neteasemb"></a><a title="更多平台" class="bshare-more bshare-more-icon more-style-addthis"></a><span class="BSHARE_COUNT bshare-share-count">0</span></div><script type="text/javascript" charset="utf-8" src="http://static.bshare.cn/b/buttonLite.js#style=-1&amp;uuid=&amp;pophcol=2&amp;lang=zh"></script><script type="text/javascript" charset="utf-8" src="http://static.bshare.cn/b/bshareC0.js"></script>
	
而且社交网站的分享位置是可以调整的，比如把微博分享的符号放在qq空间前面，直接调整上述代码顺序即可。简直爽哭了有没有！

###Good Luck!


####参考文章：
1.[Octopress 搭建流程 – Github Pages](http://shengmingzhiqing.com/blog/setup-octopress-with-github-pages.html/)

2.[Octopress 精益修改 (1)](http://shengmingzhiqing.com/blog/octopress-lean-modification-1.html/)

3.[你好！github页面](http://beyondvincent.com/2013/07/27/2013-07-27-107-hello-page-of-github/)

4.[像hackers一样写博客(二)：Octopress设置与增加微博的侧边栏](http://caok1231.iteye.com/blog/1553939)

5.[Octopress添加社会化分享和添加多说评论](http://evoupsight.com/blog/2013/10/09/octopress-add-share-discuss/)
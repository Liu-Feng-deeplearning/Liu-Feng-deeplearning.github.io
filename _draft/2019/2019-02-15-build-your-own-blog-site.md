---
layout: post
title: 使用Github搭建自己的个人博客
categories: Others
description: 搭建个人博客
keywords: github pages, github, personal blog
---

现在，越来越多的技术人员都有写博客的习惯，来记录学习笔记、分享个人想法、与同行交流等等。这导致了大量博客、社交网站的兴起，比如csdn、简书、博客园、知乎等等。但是这些都是借助别人的平台，有诸多不便，比如你的博客样式是平台定的，你的博客可能有大量的广告，等等。如果你想拥有个性化的、纯净的博客，你可以自己搭建一个博客平台，可能需要一下这些步骤：
- 购买域名
- 部署网站
- 设计网站样式

其中第一步是需要花钱的，具体价格可以自行查阅，大概一年几百到几千不等；而第二步和第三步可能需要大量的网站方面的技术支持，对于不是网站方面的技术人员较为繁琐。如果你使用Github Pages，那么以上三步都可以省去，而且步骤非常简单，分分钟搞定。
关于github pages的官方介绍请参考[这里](https://pages.github.com/)。简单的说github pages是github平台的一项服务（大多数人可能只知道github有git的服务，另外还有github pages、github jobs服务），用于你部署你的个人网站，步骤非常简单。

# 教程

首先你得拥有一个github账号，如果没有请先注册一个账号。

>最终个人博客的网址与注册的账号名有关，比如你注册的账户名为:`bob`，那么最终你的博客网址为:`https://bob.github.io`或者`https://bob.github.io/<projectname>`。下文中假设你的账户名为`feedliu`，实际上这是我的账户名，请将一下所有出现feedliu的地方改为你的github账号名。

这里的教程分为两部分：
- 自己设计博客样式
- 借用别人的博客样式

作者对于网站方面是个小白，所以第一部分只是个简单的介绍，第二部分会挑一个特定的博客样式详细讲解。如果你对网站方面的技术很懂，比如html、css等，那么你完全可以设计个性化的博客页面。实际上，不管你网站方面的专家，还是小白，都建议你看完这两部分，你会对其中的原理有一定的了解，而不是直接借用别人的样式，连最起码的原理和每个文件的大概作用都不知道。

## 自己设计博客样式

![build_blog](https://github.com/feedliu/feedliu.github.io/blob/master/images/blog/build_blog.gif?raw=true)

### 新建一个项目

新建项目时，还有个点，就是用户网站和项目网站的区别，官方介绍请参考[这里](https://help.github.com/articles/user-organization-and-project-pages/)。实际上，我感觉差别不大。
如果你想新建用户网站，请将项目命名为`feedliu.github.io`，最终你的个人博客地址为`https://feedliu.github.io`;如果你想新建项目网站，可以随意命名，假设你的项目名为`blog`，那么最终你的个人博客地址为`https://feedliu.github.io/blog`。建议你新建项目网站，跟教程同步。

### 新建index.html文件

在项目中新建一个文件`index.html`，文件中加入一行：
```
Hello, world!
```

### 启用github pages服务

点击`settings->github pages`，将`master`作为网站发布的分支。

经过这简单的三步，就搭建起了一个简单的网站，当然你可以丰富自己的`index.html`文件。

### 使用markdown文件作为页面

![](https://github.com/feedliu/feedliu.github.io/blob/master/images/blog/apply_markdown.gif?raw=true)

现在，markdown文件非常盛行，因为它只需要通过简单的format就能得到漂亮的界面和排版，markdown文件的后缀为`.md`，如果你想使用markdown文件做页面，可以这么做：
- 新建配置文件`_config.yml`，内容为：
    ```
    include:
        README.md
    ```
- 删除`index.html`文件
- 等待几分钟后，登录网址`https://feedliu.github.io/blog`查看

>这里的_config.yml非常重要，是整个网站的配置文件，控制着网站的页头、页脚、样式、包含文件等等。不管你是自己设计还是借用别人的，都会有这个文件。

### 测试

![](https://github.com/feedliu/feedliu.github.io/blob/master/images/blog/test.gif?raw=true)

现在我们可以使用markdown文件作为页面了，我们可以新建几个页面，测试一下。

- 新建`blog1.md`，内容为
    ```
    # blog1
    ```
- 新建`blog2.md`，内容为
    ```
    # blog2
    ```
- 新建`blog3.md`，内容为
    ```
    # blog3
    ```
- 修改`README.md`，内容为

    ```
    # My Blogs
    - [blog1](./blog1.md)
    - [blog2](./blog2.md)
    - [blog3](./blog3.md)
    ```
- 等待几分钟后，登录网址`https://feedliu.github.io/blog`查看

细心的你可能发现，每个网页上面都有`blog`的字样，这个可以通过配置文件`_config.yml`取消，具体的请参考[这里](https://jekyllrb.com/docs/configuration/)。

## 借用别人的博客样式

能自己设计博客样式固然好，但是也非常繁琐，其实已经有了许多别人的模板，我们直接套用就好，如果觉得不好，可以在别人的模板上修改。
官方的样式模板，请参考[这里](https://pages.github.com/themes/)，github上的样式模板，请参考[这里](https://github.com/topics/jekyll-theme)。

关于每个模板如何运用，点进每个模板的github主页，都会有介绍。这里先简单介绍一下，随后会有一个详细的例子。

### 简单引用

引用别人的模板，其实很简单，只需要在`_config.yml`后加一行：

假设是官方模板[Cayman](https://github.com/pages-themes/cayman)，加上
```
theme: jekyll-theme-cayman
```
假设是github上的非官方模板[daattali/beautiful-jekyll](https://github.com/daattali/beautiful-jekyll)，加上
```
remote_theme: daattali/beautiful-jekyll
```
但简单引用的缺点是，你无法修改样式。

### fork引用

这里的例子使用的是非官方模板[mzlogin/mzlogin.github.io](https://github.com/mzlogin/mzlogin.github.io)（实际上，这也是我自己的博客采用的模板），你可以点进该模板的github主页，README中有教程叫你如何使用，以及修改成自己的样式。你也可以选择跟着我的教程走，如果有任何问题，欢迎讨论。

#### fork模板项目

点进你想使用的模板github主页，点击fork。

#### 修改fork的项目名称

修改你刚刚fork的项目名称，建议改为`feedliu.github.io`（注意将feedliu改成你的github账号）

#### 修改 _config.yml 配置文件

修改配置文件，根据自己的信息修改，比如url、标题等等。

#### 添加你的markdown博客文件

这里就拿本篇博客举例。

- 文件开头，加上参数
    ```
    ---
    layout: post
    title: 使用Github搭建自己的个人博客
    categories: Others
    description: 搭建个人博客
    keywords: github pages, github, personal blog
    ---
    ```
    title表示标题显示在主页上的，categories表示文章类别，这个模板还有分类系统，你写上这些后，会自动为你归类。
- 修改文件名为`2019-02-15-build-your-own-blog-site.md`，前面三个数字表示发博日期，后面是文件名。
- 将本博客markdown文件加到`_posts`文件夹或者子文件夹中。

最终主页该博客显示效果：

![](https://github.com/feedliu/feedliu.github.io/blob/master/images/blog/blog-example.png?raw=true)

#### 清理数据

- 清理blogs、drafts、wiki、images文件夹中别人的数据
- 修改`pages/`中404页面和about页面

至此，已经搭建了一个功能完整的个人博客。此外，还有一些额外功能。


#### 添加评论功能

你可以添加第三方评论,支持qq和微信登录评论等，这里采用的是gittalk，简单使用issue作为评论，也就是你使用github账号评论博客。具体参考[这里](https://github.com/gitalk/gitalk#install)。

- 注册一个[OAuth App](https://github.com/settings/developers)，注册时`Authorization callback URL`这一栏填你的博客地址`https://feedliu.github.io`
- github里新建一个项目，取名为`blog-comments`，名字无所谓
- 修改`_config.yml`中的gittalk参数，其中clientID和clientSecret改成自己的，从settings -> Develop Settings 点进查看。

#### 添加google analysis来跟踪自己的博客访问量

先去[这里](https://analytics.google.com/analytics/web/)注册一个track id，可能需要你填写网站标题和url，google会为你分配一个track id，然后修改`_config.yml`配置参数：

```yml
google:
    analytics_id: # Your Google Analytics ID
```
然后就可以在google analysis的控制台查看自己的网站访问情况：

![](https://github.com/feedliu/feedliu.github.io/blob/master/images/blog/google_analysis.png?raw=true)

比如这里显示有一个用户访问，还有一些其他的统计，请自行查看。

恭喜你，到这里你就拥有了一个还算完整的个人博客了，如果想定制自己的风格，请自行去该模板的github主页学习 ^_^。

最后，你可以查看我的[个人博客](https://feedliu.github.io/)，看看效果，也可以查看[github项目](https://github.com/feedliu/feedliu.github.io)，看我具体是如果修改的。

谢谢你耐心的看完这篇博客，如果觉得对你有用，还请去我的github点个star，谢谢，也别忘了给模板作者点个star ^_^。

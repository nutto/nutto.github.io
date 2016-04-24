---
title: "使用GitHub Page搭建博客手记"
tags: [jekyll, github]
---

经过一些摸索,利用GitHub Page搭建了这个博客,期间记录了一些笔记,现在整理一下希望以后有个借鉴.


## 搭建静态的GitHub Page

> 参考: [https://pages.github.com/](https://pages.github.com/)

GitHub为了能让他人快速了解你个人,组织或项目,允许你对他们建立对应的Page.我们可以利用这个Page,建立自己的博客.

流程十分简单:

1, **在自己仓库中开一个以*用户名.github.io*命名的仓库**

比如我的用户名是*nutto*,存放我个人Page的仓库就叫*nutto.github.io*

2, **使用clone命令将新建的仓库clone到本地**

{% highlight ruby %}
git clone https://github.com/username/username.github.io
{% endhighlight %}

3, **在项目内创建index.html文件**

{% highlight ruby %}
cd username.github.io
echo "Hello World" > index.html
{% endhighlight %}

4, **将项目内的修改推到GitHub中**

{% highlight ruby %}
git add --all
git commit -m "Initial commit"
git push -u origin master
{% endhighlight %}

5, **然后我们就可以通过访问*http://username.github.io*来访问我们刚新建的index页面了**

> 我的主页是: http://nutto.github.io


## 使用Jekyll搭建更加灵活的博客

GitHub中的Page只允许静态的页面展示,如果每次写博客都要写HTML,更糟糕的还有可能要写CSS样式.这样肯定要疯掉的.作为blogger我当然希望只关心内容的撰写,最好可以使用Markdown,代码和排版不要太难看其实都已经心满意足了.

幸好我们可以使用静态页面生成工具[Jekyll][jekyll_home].开始时我们可以使用它设置好页面模版,从此以后专心撰写博客就可以了,更加好的是他能支持Markdown渲染和代码高亮等的功能,有很多地方做的比我想的更加好,简直就是天赐福音.


### 安装Jekyll

> 来源: [http://jekyll-windows.juthilo.com/](http://jekyll-windows.juthilo.com/)

Jekyll是使用Ruby写的,这就是说我们要使用它就要先安装Ruby和它配套的工具.而且可惜的是Jekyll并不官方支持Win(在开源的世界Linux总是会比Win方便得多),但是也是有工具能让Jekyll在Win中运行的.Linux的安装方法在[Jekyll主页][jekyll_installation]中都有详细的描述,而且本人的机子用Win,所以还是说说Win下的安装流程吧.


#### 安装运行Jekyll的环境

1, **到http://rubyinstaller.org/downloads/中寻找适合的安装包安装Ruby**

注意安装的时候要勾选*Add Ruby executables to your PATH*

![Add Ruby executables to your PATH](http://7xtbqv.com2.z0.glb.clouddn.com/ruby-path.png)

2,**到http://rubyinstaller.org/downloads/中下载适合的RubyDevKit压缩包**

这个RubyDevKit为Win下的Ruby提供了额外的一些工具,我们下载压缩包后将它解压,然后进入到目录下面运行

{% highlight ruby %}
ruby dk.rb init
{% endhighlight %}

然后安装

{% highlight ruby %}
ruby dk.rb install
{% endhighlight %}

就完成了


#### 安装Jekyll

话说环境安装好之后安装Jekyll是很简单的事情,运行

{% highlight ruby %}
gem install jekyll
{% endhighlight %}

理论上就能顺利安装了

但是由于国家的特殊国情我们总是会读取不到rubygems.org的资源文件,这样的话我们有几个选择

1, **使用国内的镜像**

国内的比较有名的镜像

[https://ruby.taobao.org/](https://ruby.taobao.org/)   ---貌似已经停止维护了

[http://gems.ruby-china.org/](http://gems.ruby-china.org/)

2, **使用VPN**

这个就比较简单了,但是总是觉得VPN不太优雅,一旦开启整个网络环境都改变了,访问个国内网站还要绕个圈.

3, **使用SSH**

对于应付特殊国情我觉得这个方法是比较优雅的了, 因为控制的粒度可以达到每个访问

回到正题,开启SSH后我们可以使用命令

{% highlight ruby %}
set http_proxy=http://user:password@proxy_ip:port

gem install jekyll
{% endhighlight %}

来安装Jekyll

> 参考: [https://github.com/rubygems/rubygems/issues/1068#issuecomment-63629560](https://github.com/rubygems/rubygems/issues/1068#issuecomment-63629560)


#### 快速使用Jekyll

新建一个Jekyll项目

{% highlight ruby %}
jekyll new myblog
{% endhighlight %}

进入项目目录并启动渲染

{% highlight ruby %}
cd myblog

jekyll serve
{% endhighlight %}

```jekyll serve```会对项目进行监听和渲染,如果我们想单纯地渲染项目可以使用```jekyll build```

Jekyll使用的是[Liquid](https://github.com/Shopify/liquid/wiki)模版引擎,灵活使用可以做出很多好看的博客模版
Jekyll的具体用法和配置还有很多不是三言两语就可以说得完的,具体可以参照[https://jekyllrb.com/docs/home/](https://jekyllrb.com/docs/home/)进行实践


## 博客的进阶


### 自定义博客的URL

> 参考: https://help.github.com/articles/quick-start-setting-up-a-custom-domain/

按照上述流程搭建起GitHub Page博客的URL都是固定的:**username.github.io**,这样对于希望个性化的人来说这样肯定不太好,所以GitHub也允许其他域名的别名指向,我们在自己的域名管理商中做一个别名指向到GitHub Page博客中,然后在GitHub Page的项目根目录中添加一个```CNAME```文件,内容为自定义的博客域名.

比如我在域名管理商中做了 ```blog.nuttopan.cn CNAME to nutto.github.io```的指向

然后我在CNAME文件中填写```blog.nuttopan.cn```

我就可以在http://blog.nuttopan.cn访问我的博客了

**注意: CNAME文件中如果填写多个域名,GitHub只会解释到第一个域名**


### 使用更加漂亮的Jekyll模版

如果有时间打造自己的Jekyll模版那当然是一件好事,但是绝大多数的人都没有时间去自己打造自己的Jekyll模版(特别是要做得漂亮),那么我们可以善用别人的轮子,在[http://jekyllthemes.org/](http://jekyllthemes.org/)中有很多漂亮的Jekyll模版,我们可以直接使用,最好还是要注明一下模版的作者,支持一下模版作者的开源工作.


### 使用Google的工具对博客进行监控

对于网站的监控,我还是比较喜欢使用Google的工具,文档齐全还可以有课程和资料参考,我通常使用

[Google Search Console](https://www.google.com/webmasters/)

[Google Analytics](analytics.google.com)

都是可以按照网站步骤注册使用,然后在网页提示中不断学习.


### 为博客加入评论功能

GitHub Page的评论解决方案有很多,我的网站使用的是[disqus](https://disqus.com/)

基本的操作都是注册使用,然后贴控件代码就可以了.

[jekyll_home]:  https://jekyllrb.com/docs/quickstart/
[jekyll_installation]:  https://jekyllrb.com/docs/installation/



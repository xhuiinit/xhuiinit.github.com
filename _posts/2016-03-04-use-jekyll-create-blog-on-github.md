---
layout: post
title: Jekyll本地搭建开发环境以及Github部署流程
subtitle: "使用Jekyll本地写博客，并部署到Github，绑定个人域名"
author: pizida
date: 2016-03-04 07:48:51 +0800
categories: technology
tag: 
   - 技术
---

* TOC
{:toc}


## 前言
  博客从wordpres迁移到Jekyll上来了，整个过程还是很顺利的。Jekyll是什么？它是一个简单静态博客生成工具，为什么是静态，因为它是不需要数据库的，通过<a href="http://wowubuntu.com/markdown/">markdown</a>编写静态文件，生成Html页面，它的优点是提升了页面的响应速度，并且让博主可以只专注于写文章，不用再去考虑如何排版（markdown语法很好的解决了排版问题）。那么为什么要迁移到Jekyll呢？不是因为跟风，也不是为了提高自我，完全就是瞎折腾！

## 本地搭建Jekyll
本次安装可以MacOS或Linux下进行。

### 使用gem安装Jekyll
```ruby
gem install jekyll
```
这样就把jekyll安装完成了。
如果本地没有gem，那么需要先安装Ruby和gem，这里就不介绍了。

### 使用Jekyll创建你的博客站点
```ruby
jekyll new blog  #创建你的站点
```
控制台可以看见类似 `New jekyll site installed in /home/user/blog.`的提示

### 开启Jekyll服务
```ruby
cd blog    	 #进入blog目录,记得一定要进入创建的目录，否则服务无法开启
jekyll serve 	 #启动你的http服务 
```

本地服务开启后，Jekyll服务默认端口是4000，所以我打开浏览器，输入：<a href="http://localhost:4000">http://localhost:4000</a> 即可访问

### 使用Jekyll写博文
我们进入blog目录后，会发现Jekyl的结构如下：

```ruby
.
├ about.md
├ _config.yml
├ css
│   └ main.scss
├ feed.xml
├ _includes
│   ├ footer.html
│   ├ header.html
│   ├ head.html
│   ├ icon-github.html
│   ├ icon-github.svg
│   ├ icon-twitter.html
│   └ icon-twitter.svg
├ index.html
├ _layouts
│   ├ default.html
│   ├ page.html
│   └ post.html
├ _posts
│   └ 2016-03-04-welcome-to-jekyll.markdown
└ _sass
    
    ├ _layout.scss
    └ _syntax-highlighting.scss

```       
我们进入_post目录，撰写的markdown语法的博文都放在这里。默认会有一篇测试文章：2016-03-04-welcome-to-jekyll.markdown.
拷贝测试博文

```ruby
cp 2016-03-04-welcome-to-jekyll.markdown 2016-03-04-test.markdown
```

然后刷新一下浏览器、发现页面并没有变化.因为我们还没有通过Jekyll build去生成。

```ruby
jekyll build
```
默认情况下，服务会以前台的方式挂起，如果希望用后台进程运行服务，我们可以使用 `--detach`参数，缩写参数为`-B`(应该是Background的首字母)

```ruby
jekyll serve build --detach
```
或者

```ruby
jekyll serve build -B
```
**值得注意的是：如果用vagrant虚拟机去安装jekyll，那么启动服务时还需要加上-H参数，指定访问主机号为0.0.0.0，即`jekyll serve build -B -H 0.0.0.0`,否则vagrant下可能启动失败**

再查看首页，发现已经有两篇文章了！因为没有更改标题，所以都是一样的。
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_D8F5893E-CBEB-4F56-A2A8-125929F64B3A.png"/>
好的，打开复制的 2016-03-04-test.markdown-test.markdown,可以看见开头有如下几个东东。

```ruby
---
layout: post
title:  "Welcome to Jekyll!"
date:   2016-03-04 10:52:19 +0100
categories: jekyll update
---
```

`layout`表示使用的是post布局，`title`是文章标题，`date`是自动生成的日期，`categories`是该文章生成html文件后存放的目录，可以去_site/jekyll/update下找到对应日期下面的html文档。当然你也可以只设置jekyll单一的目录，甚至是更多级别的目录，用空格分开即可。头信息设置完成后就可以书写正文了。

如果每次都输入这些头信息，还要去整理格式，那么一定很烦躁，这种重复性的东西我们就把它自动化，通过Rakefile去解决，它类似shell这样的脚本，可以使用交互模式。以下是我的Rakefile,可以复制后命名为Rakefile，放在站点根目录直接使用，也可以修改为适合自己的Rakefile：

```ruby
task :default => :new

require 'fileutils'

desc "创建新 post"
task :new do
  puts "请输入要创建的 post URL："
	@url = STDIN.gets.chomp
	puts "请输入 post 标题："
	@name = STDIN.gets.chomp
	puts "请输入 post 子标题："
	@subtitle = STDIN.gets.chomp
	puts "请输入 post 分类，以空格分隔："
	@categories = STDIN.gets.chomp
	puts "请输入 post 标签："
	@tag = STDIN.gets.chomp
	@slug = "#{@url}"
	@slug = @slug.downcase.strip.gsub(' ', '-')
	@date = Time.now.strftime("%F")
	@post_name = "_posts/#{@date}-#{@slug}.md"
	if File.exist?(@post_name)
			abort("文件名已经存在！创建失败")
	end
	FileUtils.touch(@post_name)
	open(@post_name, 'a') do |file|
			file.puts "---"
			file.puts "layout: post"
			file.puts "title: #{@name}"
			file.puts "subtitle: #{@subtitle}"
			file.puts "author: pizida"
			file.puts "date: #{Time.now}"
			file.puts "categories: #{@categories}"
			file.puts "tag: #{@tag}"
			file.puts "---"
	end
	exec "vi #{@post_name}"
end
```
如何使用Rake，输入一下命令：

```ruby
rake new
```
rake会启动交互模式，让你依次输入title，subtitle，author，categories，tag等信息，并为你创建好具有头信息的markdown文件。如下一样:

```shell
请输入要创建的 post URL：
testurl
请输入 post 标题：
testpost
请输入 post 子标题：
subtitle    
请输入 post 分类，以空格分隔：
jekyll
请输入 post 标签：
技术
```
我们查看`_post`目录，发现已经有一篇**2016-03-05-testurl.md**文章，打开看下

```ruby
---
layout: post
title: testpost
subtitle: subtitle
author: pizida
date: 2016-03-05 07:31:27 +0800
categories: jekyll
tag: 技术
---
```
已经为我们创建好头信息了。

本地搭建jekyll和写博文大致就是如此，有更多疑问可到官网<a href="https://jekyllrb.com/">https://jekyllrb.com/</a>或中文镜像站<a href="http://jekyllcn.com/">http://jekyllcn.com/</a>查阅资料。

***

## 使用Github pages服务生成个人博客

### 创建我们的仓库
我们repository name设置为username.github.com(这里的username是指你在Github上的用户名，我的用户名是engine-go,后面的**username**均指的是个人账户，不再做说明)，选择**Public**仓库类型:
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_C4D37F64-A0AD-4EEB-88A8-36BCB6DED2EA.png" alt="创建git仓库"/>

### 为仓库创建Github Pages
创建仓库后，选择**setting**
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_123.png" alt="创建Github Pages"/>

点击**Launch automatic page generator**
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_3F514A8A-3464-47B5-89AE-61FF6BC665B8.png" alt="生成pages"/>

然后编辑下标题和描述，任意选择一个模板，点击**Publish**
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_1.png"/>

如此，我们已经可以通过浏览器输入 http://username.github.io访问博客主页。那么我就访问engine-go.github.com。
如下图所示，就是你的博客首页。
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_FABD47CE-F2EC-4ECA-93C7-18517303F939.png"/>

### 将本地jekyll代码部署到Github上的仓库
前面我们已经详细地说明如何搭建Jekyll，我们可以在本地开发测试，推送代码到仓库，发布到线上。

### 克隆仓库到本地
请确保本地安装了git客户端，克隆你的username.github.com仓库到本地。

```ruby
git clone https://github.com/username/username.github.com.git
```

此时你会看见当前存在username.github.com这个目录，我们启动jekyll服务（启动前确保其他目录下没有jekyll服务，可以`ps aux|grep jekyll`查看进程,有的话,用`kill -9 进程号`杀掉）:

```ruby
cd username.github.com
jekyll serve -B
```

现在我们打开<a href="localhost:4000">http://localhost:4000</a>,即可看见我们在Github上创建的主页，理论上和http://username.github.com  访问的应该是一模一样的。

### 拷贝本地的jekyll目录到版本库
删除username.github.com下面的示例文件(README.md,不要删除，绑定域名会用到):

```ruby
rm -rf _site index.html params.json  stylesheets
``` 

拷贝本地blog(这个是前面本地搭建的blog，后续等同，不再说明)下的所有目录及文件到username.github.com

```ruby
cp -r blog/* username.github.com
```
重启服务，就可以在本地看见jekyll的站点页面了。

### 本地Jekyll站点部署到Github Pages上（相当于线上环境）

```ruby
git add --all			   #添加到暂存区	
git commit -m "提交jekyll默认页面" #提交到本地仓库
git push origin master    	   #线上的站点是部署在master下面的
```
稍等10分钟左右，Github Pages有一定时间缓存,我们刷新username.github.io看看,已经ok了！
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_D9ADB28B-F0C7-4ADE-957C-90F9682956CF.png"/>
现在已经可以在本地blog下写文章，然后通过git推送到远程仓库，大家可以通过http://username.github.io/访问你的站点了

### 为username.github.io绑定个性域名
写博客的朋友一般都有自己的域名，请根据以下方法设置：

1. 我们要绑定的话需要在username.github.com目录下增加一个CNAME文件。
在里面添加你的域名，假设为example.com，然后推送CNAME文件到远程仓库:

```ruby
git add CNAME
git push origin master
```
2. 到域名服务商增加你的CNAME记录。
添加两条记录，@和www的主机记录，记录类型为CNAME类型，CNAME表示别名记录，该记录可以将多个名字映射到同一台计算机。
记录值请写**username.github.io.**,值得注意的是`io`后面还有一个圆点，切记。
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_E1A2BE7C-E031-4379-AC65-57A1570518CF.png"/>

3. 过个几分钟，刷新博客，我的博客域名是pizida.com，就能直接访问了，不截图了。

***

## 总结
本文介绍了Jekyll本地搭建过程和远程部署Jekyll的流程，我们梳理一下思路，以后可这样开发以及发布到线上：

### 本地开发：
1. 通过`jekyll serve -B`启动服务，使用Rakefile创建文章，然后用自己喜欢的工具进行写作。
2. 创作完成，通过`jekyll build`生成页面，本地localhost:4000查看文章。

### 发布线上
1. 本地确认文章无误，可以通过`git add`,`git commit`,`git push`等git命令推送文章到Github Pages服务器。
2. 通过绑定的域名，查看线上文章，这个时候大家都能访问了。

有说得不对的地方，请多多指教。

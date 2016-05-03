---
layout: post
title: Access-Control-Allow-Origin跨域现象分析及解决方案
subtitle: "XMLHttpRequest cannot load的跨域问题以及解决方案"
author: pizida
date: 2016-03-15 10:40:19 +0100
categories: technology 
tag: 技术
---

* TOC
{:toc}


## 什么是跨域
谈跨域概念之前，我们应该讲讲`同源策略`这个东西。同源策略是现代浏览器为了安全性问题而制定的一种安全策略,该策略限制了一个源（origin）中加载文本或脚本与来自其它源（origin）中资源的交互方式，请记住是交互方式。WIKI上有详细说明：<a href="https://en.wikipedia.org/wiki/Same-origin_policy">Same-origin policy</a> 

那么我们用人话来说，什么是同源策略？那就是如果有两个页面或请求来源，它们具有相同的`协议`，`端口`,`主机`，那么这两个页面就是同源，符合同源策略。这样子，跨域就十分好理解了，只要违背了同源策略的标准，有一个因素出现不同，那么这两个页面就跨域。


我们用表格列出跨域例子：

|页面1|页面2|说明|是否允许通信|
|:----:|:----:|:---:|:-----------------:|
|http://www.t.com/a.js|http://www.t.com/b.js|不同的js文件|允许通信|
|http://www.t.com/a/a.js|http://www.t.com/b/b.js|不同文件夹下的js文件|允许通信|
|http://www.t.com`:80`/a.js|http://www.t.com`:8080`/b.js|**端口不同**|不允许通信|
|`http:`//www.t.com/a.js|`https:`//www.t.comb.js|**协议不同**|不允许通信|
|http://`www.t.com`/a.js|http://`www.tt.com`/b.js|**主机不同**|不允许通信|
|http://`www.t.com`/a.js|http://`192.168.204.69`/b.js|**主机和主机对应的ip**|不允许通信|
|http://`www.t.com`/a.js|http://`t.com`/b.js|**子域名和主域名**|不允许通信|

## 跨域现象剖析
了解概念后，我们看看跨域到底会带来怎么样的现象

### 实验一：协议、域名相同，端口号不同
准备条件：本地创建两个web服务，一个是80端口，一个是8080端口。
本地新建`jsonp.html`,`cross80.php`,`cross8080.php`。
其中`jsonp.html`,`cross80.php`放在80端口web目录下，`cross8080.php`放在8080端口web目录下。

说明：实验一将显示全部代码，后面实验只显示核心代码。

1. 首先我们访问通过AJAX访问cross80.php.

 jsonp.html代码如下 (为了集中描述跨域问题，我们这里使用jQuery)：

{% highlight javascript %}
		$(function(){
				
				var request_url = "http://localhost:80/cross80.php";
				$.ajax({
					type:'get',
					url:request_url,
					dataType:"json",
					success:function(data){
						alert(data.msg);
					}
				});
		});
{% endhighlight %}

cross.php 代码如下：

{% highlight php %}
<?php
	$return_data = array('msg'=>'success!');
	echo json_encode($return_data);
{% endhighlight %}

我们打开浏览器，访问http://localhost:80/cross80.php，浏览器会弹窗。输出**success！**。

2. 然后我们访问cross8080.php

`cross8080.php`和`cros80.php`保持一模一样

jsonp.html中的访问地址修改为http://localhost:8080/cross8080.php

```js
                $(function(){

                                var request_url = "http://localhost:8080/cross8080.php";
                                $.ajax({
                                        type:'get',
                                        url:request_url,
                                        dataType:"json",
                                        success:function(data){
                                                alert(data.msg);
                                        }
                                });
                });
```
我们发现，浏览器没有弹窗，查看chrome控制台，显示一下结果：
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_QQ%E5%9B%BE%E7%89%8720160315185609.png"/>
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_QQ%E5%9B%BE%E7%89%8720160315185631.png"/>

图片截得太小，我把提示打出来:

<blockquote>XMLHttpRequest cannot load http://localhost:8080/cross8080.php. 
No 'Access-Control-Allow-Origin' header is present on the requested resource. 
Origin 'http://localhost' is therefore not allowed access.</blockquote>

提示已经说得很清楚，我翻译一下:XMLHttpRequest无法加载cross8080.php这个文件，因为端口号不同，这逼跨域了,根据浏览器同源策略，不给予访问。

### 实验二：协议、端口相同，域名不同
准备域名：

* `www.test1.com`
* `www.test2.com` 

准备页面：

jsonp.html 代码：

```js
var request_url = "http://www.test1.com/test/test1.php";
``` 
* http://www.test1.com/jsonp.html
* http://www.test1.com/`test`/test1.php

* jsonp.html,test1.php(放在www.test1.com上)
* test2.php(放在www.test2.com上)

1.访问test1.php

jsonp.html 代码：

```js
var request_url = "http://www.test1.com/test1.php";
```
test1.php 代码：
{% highlight php %}
<?php
        $return_data = array('msg'=>'success!');
        echo json_encode($return_data);
{% endhighlight %}

结果：浏览器正常弹框，显示**success!**


2.请求test2.php

jsonp.html 代码：

```js
var request_url = "http://www.test2.com/test2.php";
```
test1.php 代码：
{% highlight php %}
<?php
        $return_data = array('msg'=>'success!');
        echo json_encode($return_data);
{% endhighlight %}

我们先直接打开test2.php,界面显示如下：
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_2016-03-16_114223.png"/>
说明test2.php是可以正常访问的。

我们通过jsonp.html去访问看看,显示如下：
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_2016-03-16_114702.png"/>
图中提示和实验一的是一模一样的，
<blockquote>XMLHttpRequest cannot load http://www.test2.com/test2.php. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://www.test1.com' is therefore not allowed access.</blockquote>
因为跨域问题，所以不能访问到test2.php脚本。
结果：不弹窗。端口（都是80）相同，协议相同，域名不同也是违反了同源策略。

### 实验三：协议、端口相同，一个是子域名，一个是主域名
准备页面：

* http://www.test1.com/jsonp.html
* http://test1.com/test1.php

显示提示如下：
<blockquote>XMLHttpRequest cannot load http://test1.com/test1.php. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://www.test1.com' is therefore not allowed access.</blockquote>

结果：不弹窗。说明子域名和主域名也不能通过XMLHttpRequest进行通信。

### 实验四：协议、端口、域名相同，目录不同
准备页面：

* http://www.test1.com/jsonp.html
* http://www.test1.com/`test`/test1.php

jsonp.html 代码：                                                                                                

```js                                                  
var request_url = "http://www.test1.com/test/test1.php";    
```                                                    

结果：弹窗，显示**succes!**。说明不同目录不受同源策略限制。

### 实验五：协议、端口相同,一个是域名，另一个是域名对应的ip

准备页面：

* http://**IP**/jsonp.html
* http://www.test1.com/test1.php

提示如下：
<blockquote>XMLHttpRequest cannot load http://www.test1.com/test1.php. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://IP' is therefore not allowed access.</blockquote>

结果：主机对应的ip和主机名也违法了同源策略。

### 实验六：XMLHttpRequest请求同源接口，后端控制跨域访问

准备页面：

* http://www.test1.com/jsonp.html
* http://www.test1.com/test1.php

jsonp.html

```js
var request_url = "http://www.test1.com/test/test1.php";
```
test1.php
{% highlight php %}
<?php
	$url = "http://www.baidu.com";
	header("Location:".$url);
        $return_data = array('msg'=>'success!');
        echo json_encode($return_data);
{% endhighlight %}

提示如下：
<blockquote>XMLHttpRequest cannot load http://www.baidu.com. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://www.test1.com' is therefore not allowed access.</blockquote>

因为后端脚本更改了header的Locaiton,更改了当前访问源头。
我们尝试把修改一下`$url`,
{% highlight php %}
<?php
        $url = "http://www.test1.com/test2.php";
        header("Location:".$url);
        $return_data = array('msg'=>'success!');
        echo json_encode($return_data);
{% endhighlight %}

并且，我们创建一个test2.php文件
然后访问jsonp.html页面，此时正常跳转到test2.php页面。显示如下：

```js
{
msg: "success!"
}
```
结果：后端修改header的Location字段时，当跳转的url非同源时，将会跨域。

## XMLHttpRequest跨域的解决方案

### 解决方案一：修改访问服务器的header，跳过同源策略限制。

增加HTTP响应头的**Access-Control-Allow-Origin**,如果需要跨域请求，那么请求的服务器响应头可以返回如下：

```php
Access-Control-Allow-Origin: <origin> | *
```

举个例子，如果我们允许来自http://baidu.com的请求，可以这样设定：
<blockquote>Access-Control-Allow-Origin: http://baidu.com</blockquote>

如果允许任何域的请求，可以这样设定：
<blockquote>Access-Control-Allow-Origin: *</blockquote>

各个语言的实现：

PHP中实现：

允许任何域请求：

```php
header('Access-Control-Allow-Origin: *');  
```

允许特定域请求

```php
header('Access-Control-Allow-Origin: http://pizida.com');
```

Java中实现：

允许任何域请求：

```java
response.addHeader("Access-Control-Allow-Origin", "*");
```

允许特定域请求

```java
response.addHeader("Access-Control-Allow-Origin", "http://pizida.com");
```

ASP.NET中实现

允许任何域请求：

```java
 filterContext.RequestContext.HttpContext.Response.AddHeader("Access-Control-Allow-Origin", "*");
        base.OnActionExecuting(filterContext);
```

允许特定域请求

```java
 filterContext.RequestContext.HttpContext.Response.AddHeader("Access-Control-Allow-Origin", "http://pizida.com");
        base.OnActionExecuting(filterContext);
```

结论：

* 优点：方便快捷。
* 缺点：缺乏安全性；需要有服务器的控制权，如果是第三方提供的接口不一定可以使用。

### 解决方案二：动态创建script（俗称JSONP）

要使用动态创建script来解决问题，首先了解一下几点：

* 我们通过前面的实验已经了解到在js中通过XMLHttpRequest进行跨域请求不允许
* 但是浏览器允许引用js文件，还有更多的标签允许跨域内嵌：
  1. `<script src="..." ></script>`标签嵌入跨域脚本。语法错误信息只能在同源脚本中捕捉到。
  2. `<link rel="stylesheet" href="...">`标签嵌入CSS
  3. `<img>`嵌入图片
  4. `<video>` 和 `<audio>`嵌入多媒体资源
  5. `@font-face`引入的字体
  6. `<frame>` 和 `<iframe>`载入的任何资源
* JSON是一种用来描述数据结构的简单方式，和XML一样，却又更简洁。

这样，我们是否可以用一种方法让服务端返回具有JSON格式的数据，JSON中包含script将会使用的函数或方法，然后在页面嵌入这些·<script>·，达到调用目标函数的目的，从而进行跨域通信。
说了这些，我们还是实践一下为妙！

#### 实例1：跨域读取js文件

现在localhost:`80`上有一个jsonp.html页面，localhost:`8080`上有一个test.js页面
jsonp.html代码如下：

```js
<script src="http://localhost:8080/test.js"></script>
```

test.js代码如下：

```js
alert('Hello World!');
```

现在刷新浏览器，显示如下：
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_2016-03-17_115854.jpg"/>

你们知道肯定会弹窗的了，对不对？上面说了`script`标签嵌入浏览器是支持的，并且这是一种很流行的做法。我们的确通过这种做法引用了其他域的资源，获得了数据。那么我们是否可以同样地获取更多的后端数据呢？


#### 实例2：跨域获取服务器的简单数据

同样，在localhost:`80`上有一个jsonp.html页面，这次localhost:`8080`上换成一个test.php文件。

jsonp.html代码如下：

```js
<script>	
	var my_alert = function (data){
		alert('跨域获得数据，服务器传来的提示是：'+ data.msg);
	}
</script>

<script src="http://localhost:8080/test.php"></script>
```
实际上这里有两段js，第一段是声明客户端的函数，第二段是需要获取的服务端返回的数据（精妙之处就在这里！）
我们就拿端口号不同从而造成跨域问题来解决：

```php
echo "my_alert({'msg':'80端口，你好！我是8080端口'})";
```
我们因为知道了客户端定义的是`my_alert`函数，所以这里加上了改函数名，并且构建了一个JSON，里面有一个msg元素。

刷新页面，显示如下：
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_2016-03-17_121825.jpg"/>

貌似已经可以成功跨域通信啦！但是我是事先知道`my_alert`函数名，如果不知道呢？那服务端如何处理，假设我还需要传递参数呢，服务端通过参数进行一系列逻辑呢？要如何实现？

#### 实例3：传递calback参数，服务器动态返回回调函数名

同样，在localhost:`80`上有一个jsonp.html页面，localhost:`8080`上是一个test.php文件。

jsonp.html代码如下：

```js
<script>
		var my_alert = function (data){
			alert('查询到的商品是:'+data.product+"，价格为："+data.price);
		}
   </script>

   <script src="http://localhost:8080/test.php?product=苹果&callback=my_alert"></script>
```

test.php代码：

```php
$myFunction= $_GET['callback'];
$product= $_GET['product'];

//可以点业务逻辑什么的,获得了price

$price=6888;

echo "{$myFunction}({'product':'$product','price':'$price'})";
exit;
```

刷新页面，显示如下：
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_2016-03-17_132901.jpg"/>

现在我们把调用的函数通过calback传递过去，那么服务端就不用关心客户端到底是什么函数名了。同理，还能传递其他参数。

这样子实际上就完成了嵌入script解决跨域问题。但是我们这样子写是不是太松散了，可否模块化处理呢？那就使用jQuery吧！

#### 实例4：通过jQuery封装的JSONP进行跨域通信（业界常用）

JSONP并没有什么高大上的地方，只是将我们前面几种实例封装起来，放进jQuery中供客户端调用。现在我们来实践一下吧。

jsonp.html

```js
<script>
	$(function(){
				
			var request_url = "http://localhost:8080/test.php";

			$.ajax({
				type:'get',		
				url:request_url,
				data:'product=香蕉',
				dataType:"jsonp",	  //返回的数据类型，定义为jsonp
				jsonp:"callback",	  //传给服务端的，为获得回调名称，默认为calllback
				jsonpCallback:"my_alert", //回调函数名称，可以不写
				success:function(data){
				    console.log(data);
				}
			});
		});

</script>
```
test.php我们不做任何修改。
查看页面，如下：
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_2.jpg"/>
确定弹窗已经返回了香蕉，是我们代码中设置的。

我们再看看请求过去的参数以及响应地址和端口，证明跨域访问成功：
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_QQ%E5%9B%BE%E7%89%8720160317142025.jpg"/>

最后看看console.log的内容是否正确：
<img src="http://7xqfiw.com1.z0.glb.clouddn.com/blog_11.jpg"/>


## 总结 
* 其实跨域问题解决起来十分简单，我们可以更改header响应实现，也可以通过动态嵌入script（也就是封装好的JSONP）。

* JSONP是大家发挥聪明才智想出来的办法，也是业界最常用最方便的跨域解决方法。

* 现在相信大家看见`Access-Control-Allow-Origin`等字眼就再也不会陌生了。当然除了这两个方法，还有一些其他方案，例如`document.domain+iframe`的设置,`window.name`实现的跨域数据传输,使用`HTML5 postMessage`,利用`flash`,因为JSONP是最常用的，也是最实用的，其他方案就不展开了，大家可以自行研究。

* 本文后端语言完全可以换成其他，例如Java,Python,Ruby等都是可以的。



title: 为NexT主题添加文章阅读量统计功能
isupdate: false
date: 2015-10-21 21:57:00
updatetime:
tags: Hexo
categories: Hexo
---



### 前言

---

由于最近在折腾Android项目，需要用到一些与服务器交互、以及数据存储的相关功能，然后发现了[LeanCloud](https://leancloud.cn)这家服务提供商,使用下来还感觉还挺靠谱的(请给我广告费)。正好发现他们服务提供了[JavaScript SDK](https://leancloud.cn/docs/js_guide.html)，于是就想着尝试着实现Hexo博客文章的浏览数统计功能，之前虽然在使用不蒜子，但是不蒜子不能够在主页展示文章阅读量啊！对于博主这种有强迫症又想装X的人来说果断不能忍啊！



### ~~修改NexT主题模版~~

~~本方法理论上对Hexo博客通用，由于博主使用的是NexT主题，所以当然针对NexT来说咯。~~**<font color=red>NexT主题目前已经合并这个Feature，因此如果你使用的是NexT主题，可以直接使用不用修改主题模版而直接在`_config.yml`中配置即可，请直接跳转查看[配置LeanCloud](#%E9%85%8D%E7%BD%AELeanCloud)</font>**

<!-- more -->

####  ~~修改`_config.yml`文件~~

~~打开NexT主题的根目录下的`_config.yml`文件，在任意位置添加以下内容：~~

```javascript
leancloud_visitors:
  enable: true
  app_id: #<AppID>
  app_key: #<AppKEY>
```

#### ~~添加`lean-analytics.swig`文件~~

~~在主题的`layout\_scripts`路径下，新建一个`lean-analytics.swig`文件，并向里面添加以下内容~~


```javascript
<!-- custom analytics part create by xiamo -->
<script src="https://cdn1.lncld.net/static/js/av-core-mini-0.6.1.js"></script>
<script>AV.initialize("{{theme.leancloud_visitors.app_id}}", "{{theme.leancloud_visitors.app_key}}");</script>
<script>
function showTime(Counter) {
	var query = new AV.Query(Counter);
	$(".leancloud_visitors").each(function() {
		var url = $(this).attr("id").trim();
		query.equalTo("url", url);
		query.find({
			success: function(results) {
				if (results.length == 0) {
					var content = $(document.getElementById(url)).text() + ': 0';
					$(document.getElementById(url)).text(content);
					return;
				}
				for (var i = 0; i < results.length; i++) {
					var object = results[i];
					var content = $(document.getElementById(url)).text() + ': ' + object.get('time');
					$(document.getElementById(url)).text(content);
				}
			},
			error: function(object, error) {
				console.log("Error: " + error.code + " " + error.message);
			}
		});

	});
}

function addCount(Counter) {
	var Counter = AV.Object.extend("Counter");
	url = $(".leancloud_visitors").attr('id').trim();
	title = $(".leancloud_visitors").attr('data-flag-title').trim();
	var query = new AV.Query(Counter);
	query.equalTo("url", url);
	query.find({
		success: function(results) {
			if (results.length > 0) {
				var counter = results[0];
				counter.fetchWhenSave(true);
				counter.increment("time");
				counter.save(null, {
					success: function(counter) {
						var content = $(document.getElementById(url)).text() + ': ' + counter.get('time');
						$(document.getElementById(url)).text(content);
					},
					error: function(counter, error) {
						console.log('Failed to save Visitor num, with error message: ' + error.message);
					}
				});
			} else {
				var newcounter = new Counter();
				newcounter.set("title", title);
				newcounter.set("url", url);
				newcounter.set("time", 1);
				newcounter.save(null, {
					success: function(newcounter) {
					    console.log("newcounter.get('time')="+newcounter.get('time'));
						var content = $(document.getElementById(url)).text() + ': ' + newcounter.get('time');
						$(document.getElementById(url)).text(content);
					},
					error: function(newcounter, error) {
						console.log('Failed to create');
					}
				});
			}
		},
		error: function(error) {
			console.log('Error:' + error.code + " " + error.message);
		}
	});
}
$(function() {
	var Counter = AV.Object.extend("Counter");
	if ($('.leancloud_visitors').length == 1) {
		addCount(Counter);
	} else if ($('.post-title-link').length > 1) {
		showTime(Counter);
	}
});
</script>
```

#### ~~修改`post.swig`文件~~

~~在主题的`layout\_macro`路径下，打开`post.swig`文件，找到以下内容（大概88行）：~~

```html
          {% if not is_index and theme.facebook_sdk.enable and theme.facebook_sdk.like_button %}
            &nbsp; | &nbsp;
            <div class="fb-like" data-layout="button_count" data-share="true"></div>
          {% endif %}
```

~~在其后面添加如下内容：~~

```html
{% if theme.leancloud_visitors.enable %}
		  	 <span id="{{ url_for(post.path) }}"class="leancloud_visitors"  data-flag-title="{{ post.title }}">
            &nbsp; | &nbsp; {{__('post.visitors')}}
            </span>
		  {% endif %}
```

~~添加完毕之后，文件内容像这个样子：~~

```html
          {% if not is_index and theme.facebook_sdk.enable and theme.facebook_sdk.like_button %}
            &nbsp; | &nbsp;
            <div class="fb-like" data-layout="button_count" data-share="true"></div>
          {% endif %}
			 {% if theme.leancloud_visitors.enable %}
		  	 <span id="{{ url_for(post.path) }}"class="leancloud_visitors"  data-flag-title="{{ post.title }}">
            &nbsp; | &nbsp; {{__('post.visitors')}}
            </span>
		  {% endif %}
        </div>
      </header>
    {% endif %}
```

#### ~~修改`layout.swig`文件~~

~~在NexT根目录的`layout`路径下，打开 `_layout.swig`文件，在`</body>`上方添加如下内容：~~
```html
{% if theme.leancloud_visitors.enable %}
  	 {% include '_scripts/lean-analytics.swig' %}
  {%  endif %}
```

~~添加完成之后，文件内容像这个样子：~~

```html
{# LazyLoad #}
  <script type="text/javascript" src="{{ url_for(theme.js) }}/lazyload.js"></script>
  <script type="text/javascript">
    $(function () {
      $("#posts").find('img').lazyload({
        placeholder: "{{ url_for(theme.images) }}/loading.gif",
        effect: "fadeIn"
      });
    });
  </script>
  {% if theme.leancloud_visitors.enable %}
  	 {% include '_scripts/lean-analytics.swig' %}
  {%  endif %}
</body>
</html>
```

#### ~~修改`zh-Hans.yml`文件~~

~~在NexT目录的`languages`路径下的`zh-Hans.yml`文件，在`post:`结点下添加`visitors: 阅读次数`，像这个样子：~~
```html
post:
  posted: 发表于
  visitors: 阅读次数
  updated: 更新于
  in: 分类于
  read_more: 阅读全文
  untitled: 未命名
  toc_empty: 此文章未包含目录
```

~~** 如果你使用的是其它NexT的语言，请相应的添加该字段即可。**~~

~~至此NexT的修改工作就完成了，但是现在还是不能够使用文章阅读量这个统计功能的。这个功能依赖于[LeanCloud](https://leancloud.cn/)提供后端数据存取，因此我们需要注册一个[LeanCloud帐号](https://leancloud.cn/login.html#/signup)才能继续使用这个功能，**[点我快速注册](https://leancloud.cn/login.html#/signup).**~~



-----



### 配置[LeanCloud](https://leancloud.cn)

在注册完成LeanCloud帐号并验证邮箱之后，我们就可以登录我们的LeanCloud帐号，进行一番配置之后拿到`AppID`以及`AppKey`这两个参数即可正常使用文章阅读量统计的功能了。


#### 创建应用

- 我们新建一个应用来专门进行博客的访问统计的数据操作。首先，打开控制台，如下图所示：


![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/open_consoloe.png "打开控制台")

- 在出现的界面点击`创建应用`：


![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/create_app.png "创建应用")

- 在接下来的页面，新建的应用名称我们可以随意输入，即便是输入的不满意我们后续也是可以更改的:


![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/creating_app.png "创建的新应用名称")

- 这里为了演示的方便，我新创建一个取名为test的应用。创建完成之后我们点击新创建的应用的名字来进行该应用的参数配置：


![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/create_class.png "打开应用参数配置界面")

- 在应用的数据配置界面，左侧下划线开头的都是系统预定义好的表，为了便于区分我们新建一张表来保存我们的数据。点击左侧右上角的齿轮图标，新建Class：
   在弹出的选项中选择`创建Class`来新建Class用来专门保存我们博客的文章访问量等数据:
   点击`创建Class`之后，~~理论上来说名字可以随意取名，只要你交互代码做相应的更改即可~~，但是为了保证我们前面对NexT主题的修改兼容，此处的**<font color=green>新建Class名字必须为<font color=red >`Counter`**</font></font>:

![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/creating_class.png "权限配置")

- 由于LeanCloud升级了默认的ACL权限，如果你想避免后续因为权限的问题导致次数统计显示不正常，建议在此处选择`无限制`。



![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/open_app_key.png "打开应用设置")

创建完成之后，左侧数据栏应该会多出一栏名为`Counter`的栏目，这个时候我们点击顶部的设置，切换到test应用的操作界面:
在弹出的界面中，选择左侧的`应用Key`选项，即可发现我们创建应用的`AppID`以及`AppKey`，有了它，我们就有权限能够通过主题中配置好的Javascript代码与这个应用的Counter表进行数据存取操作了:


![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/opened_app_key.png "获取Appid、Appkey")

复制`AppID`以及`AppKey`并在NexT主题的`_config.yml`文件中我们相应的位置填入即可，正确配置之后文件内容像这个样子:

```html
leancloud_visitors:
  enable: true
  app_id: joaeuuc4hsqudUUwx4gIvGF6-gzGzoHsz
  app_key: E9UJsJpw1omCHuS22PdSpKoh
```

这个时候重新生成部署Hexo博客，应该就可以正常使用文章阅读量统计的功能了。需要特别说明的是：记录文章访问量的唯一标识符是文章的`发布日期`以及`文章的标题`，因此请确保这两个数值组合的唯一性，如果你更改了这两个数值，会造成文章阅读数值的清零重计。

### 后台管理

当你配置部分完成之后，初始的文章统计量显示为0，但是这个时候我们LeanCloud对应的应用的`Counter`表中并没有相应的记录，只是单纯的显示为0而已，当博客文章在配置好阅读量统计服务之后第一次打开时，便会自动向服务器发送数据来创建一条数据，该数据会被记录在对应的应用的`Counter`表中。



![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/background.png "后台管理")

我们可以修改其中的`time`字段的数值来达到修改某一篇文章的访问量的目的（博客文章访问量快递提升人气的装逼利器）。双击具体的数值，修改之后回车即可保存。

-  `url`字段被当作唯一`ID`来使用，因此如果你不知道带来的后果的话请不要修改。
-  `title`字段显示的是博客文章的标题，用于后台管理的时候区分文章之用，没有什么实际作用。
-  其他字段皆为自动生成，具体作用请查阅LeanCloud官方文档，如果你不知道有什么作用请不要随意修改。

### Web安全

因为AppID以及AppKey是暴露在外的，因此如果一些别用用心之人知道了之后用于其它目的是得不偿失的，为了确保只用于我们自己的博客，建议开启Web安全选项，这样就只能通过我们自己的域名才有权访问后台的数据了，可以进一步提升安全性。

选择应用的设置的`安全中心`选项卡:



![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/open_safe_center.png "进入安全中心")

在`Web 安全域名`中填入我们自己的博客域名，来确保数据调用的安全:

![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/bind_domain.png "锁定域名")


如果你不知道怎么填写安全域名而或者填写完成之后发现博客文章访问量显示不正常，打开浏览器调试模式，发现如下图的输出:


![](http://7xkj6q.com1.z0.glb.clouddn.com/static/images/leancloud-page-anlysis/broswer_403.png "Web安全域名填写错误")

这说明你的安全域名填写错误，导致服务器拒绝了数据交互的请求，你可以更改为正确的安全域名或者你不知道如何修改请在本博文中留言或者放弃设置Web安全域名。


Enjoy it！

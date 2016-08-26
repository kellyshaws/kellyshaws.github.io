---
layout: post
title:  "From Apprentice To Artisan --服务提供者"
date:   2016-04-21 14:00:15
description: "From Apprentice To Artisan --Service Providers"
permalink: post/service-providers
disqus:
  id: service-providers
categories:
- blog
- php
- laravel
---

一个Laravel服务提供者就是一个用来进行IoC绑定的类。事实上，Laravel有好几十个服务提供者，用于管理框架核心组件的容器绑定。几乎框架里每一个组件的IoC绑定都是靠服务提供者来做的。你可以在app/config/app.php这个文件里查看目前有哪些服务提供者。<br>

一个服务提供者必须有一个register方法。你可以在这个方法里写IoC绑定。当一个请求发过来，程序框架刚启动时，所有在你配置文件里的服务提供者的register方法就会被调用。这在程序周期的很早的地方就会执行，所以在你自己的引导代码里所有的服务已经准备好了。<br>

<blockquote>
	<p>
		注册Vs引导代码<br>

		永远不要在register方法里面使用任何服务。该方法只是用来进行IoC绑定的地方。所有关于绑定类后续的判断、交互都要在boot方法里进行。<br>
	</p>
</blockquote>

你用Composer安装的一些第三方包也会有服务提供者。在第三方包的安装说明里一般都会告诉你要在providers数组里加上一行。一旦你加上了，那这个服务就算安装好了。<br>

<blockquote>
	<p>
		包提供者<br>

		不是所有的第三方包都需要服务提供者。事实上一个包并不需要服务提供者。因为服务提供者只是一个用来自动初始化服务组件的地方，一个方便管理引导代码和容器绑定的地方。<br>
	</p>
</blockquote>

##延迟加载的服务提供者##

并非在你配置文件中的providers数组里的所有提供者在每次请求都会被实例化。否则会对性能不利，尤其是这个服务的功能用不到的情况下。比如，QueueServiceProvider服务就不是每次都用得到。<br>

为了达到只实例化需要的服务的提供者，Laravel生成了服务清单并且储存在了app/storage/meta目录下。这份清单列出了应用里所有的服务提供者，包括容器绑定的名字也记录了。这样，当应用想让容器取出一个名为queue的绑定时，Laravel知道需要先实例化并运行QueueServiceProvider因为在服务清单里记录着该服务提供者能提供queue的绑定。如此这般框架就能够延迟加载每个请求需要的服务了，性能大大提高。<br>

<blockquote>
	<p>
		如何生成服务清单<br>

		当你在providers数组里新增一条，Laravel在下一次请求时就会自动重新生成服务清单。<br>
	</p>
</blockquote>

如果你有时间，去看看服务清单文件里面的内容。理解这个文件的结构有助于你对服务进行排错。<br>
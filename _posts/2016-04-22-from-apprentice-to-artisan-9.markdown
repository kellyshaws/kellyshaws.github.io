---
layout: post
title:  "From Apprentice To Artisan --服务提供者"
date:   2016-04-21 14:20:15
description: "From Apprentice To Artisan --Service Providers"
permalink: post/service-providers-3
disqus:
  id: service-providers-3
categories:
- blog
- php
- laravel
---

在所有服务提供者都注册以后，他们就进入了“启动”过程。该过程会触发每个服务提供者的boot方法。这里会发生一种常见的错误用法：在register方法里面调用其他的服务。由于在register方法里我们不能保证所有其他服务都已经被加载，所以在该方法里调用别的服务有可能会出错。所以如果你想在服务提供者里调用别的服务，请在boot方法里做这种事儿。register方法只能进行容器注册。<br>

在启动方法里面，你想做什么都可以：注册事件监听，引入路由文件，注册过滤器，或者其他你能想象到的事儿。再强调一下，要发挥服务提供者的管理功能。可能你想将相关的多个事件监听归为一组？将他们放到一个服务提供者的boot方法里，这会很管用的！或者你也可以引入单独的events、routesPHP文件：<br>

```php

	public function boot()
	{
	    require_once __DIR__.'/events.php';
	    require_once __DIR__.'/routes.php';
	}

```

我们已经学习了依赖注入以及如何使用服务提供者来组织管理我们的项目。这样我们的Laravel应用就有了一个很好的基础，它结构优美并且易于维护和测试。接下来，我们将探索Laravel框架本身是如何使用服务提供者的，并且深究其原理！<br>

<blockquote>
	<p>
		不要让条条框框限制你自己<br>

		记住，服务提供者不仅仅是专业的软件包才能使用。请大胆的使用它来组织管理你的应用服务吧。<br>
	</p>
</blockquote>

##核心也是服务提供者的模式##

你可能已经注意到，在app配置文件里面已经有了很多服务提供者。每一个都负责启动框架核心的一部分。比如MigrationServiceProvider负责启动数据库迁移的类，包括Artisan里面的命令。EventServiceProvide负责启动和注册事件调度机制。不同的服务提供者有着不同的复杂度，但他们都负责启动核心的一部分。<br>

<blockquote>
	<p>
		和服务提供者们见见面<br>

		理解Laravel核心的最好方法是去读它的核心服务源码。如果你对这些服务的源码、容器注册等都很熟悉，那么你对Laravel是如何工作的将会有十分深刻的理解。<br>
	</p>
</blockquote>

大部分的服务提供者是延迟加载的，意味着并非所有请求都会调用到他们；然而有一些很基础的服务是每一次请求都会被加载的，比如FilesystemServiceProvide和ExceptionServiceProvider。有人会说核心服务提供者和应用程序容器就是Laravel。Laravel其实是将这么多不同部分联系起来，形成一个单一的、内聚的整体的这么一个机制。拿建筑来比喻，那些服务提供者就是框架的预制模块。<br>

正如之前提到的那样，如果你想更深的了解框架是如何运行的，请读Lravel的核心服务的源码吧。读过之后，你会对框架如何将各部分组合在一起、每一个服务是如何为你所用这些机制有更坚实的理解。此外，有了这些进一步的理解，你也可以为Laravel添砖加瓦！<br>
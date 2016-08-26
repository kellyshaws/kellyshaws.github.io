---
layout: post
title:  "From Apprentice To Artisan --扩展框架"
date:   2016-04-21 15:10:15
description: "From Apprentice To Artisan -- Extending The Framework"
permalink: post/extending-the-framework
disqus:
  id: extending-the-framework
categories:
- blog
- php
- laravel
---

为了方便你自定义框架核心组件，Laravel提供了大量可以扩展的地方。你甚至可以完全替换掉旧组件。例如：哈希器遵守了HasherInterface接口，你可以按照你自己应用的需求来重新实现。你也可以扩展Request对象，添加你自己用的顺手的helper方法。你甚至可以添加全新的身份认证、缓存和会话机制！<br>

Laravel组件通常有两种扩展方式：在IoC容器里面绑定新实现，或者用Manager类注册一个扩展，该扩展采用了工厂模式实现。在本章中我们将探索不同的扩展方式并检查我们都需要些什么代码。<br>

<blockquote>
	<p>
		扩展方式<br>

		要记住Laravel通常有以下两种扩展方式：通过IoC绑定和通过Manager类。其中管理类实现了工厂设计模式，负责组件的实例化。比如缓存和会话机制。<br>
	</p>
</blockquote>

##管理者和工厂##

Laravel有好多Manager类用来管理基于驱动的组件的生成过程。基于驱动的组件包括：缓存、会话、身份认证、队列组件等。管理类负责根据应用程序的配置，来生成特定的驱动实例。比如：CacheManager可以创建APC、Memcached、Native、还有其他不同的缓存驱动的实现。<br>

每个管理类都包含名为extend的方法，该方法可用于将新功能注入到管理类中。下面我们将逐个介绍管理类，为你展示如何注入自定义的驱动。<br>

<blockquote>
	<p>
		如何了解你的管理类<br>

		请花点时间看看Laravel中各个Manager类的代码，比如CacheManager和SessionManager。通过阅读这些代码能让你对Laravel的管理类机制更加清楚透彻。所有的管理类都继承自Illuminate\Support\Manager基类，该基类为每一个管理类提供了一些有效且通用的功能。<br>
	</p>
</blockquote>

##缓存##

要扩展Laravel的缓存机制，我们将使用CacheManager里的extend方法来绑定我们自定义的缓存驱动。扩展其他的管理类也是类似的。比如，我们想注册一个新的缓存驱动，名叫"mongo"，代码可以这样写：<br>

```php

	Cache::extend('mongo', function($app)
	{
	    // Return Illuminate\Cache\Repository instance...
	});

```

extend方法的第一个参数是你要定义的驱动的名字。该名字对应着app/config/cache.php配置文件中的driver项。第二个参数是一个匿名函数（闭包），该匿名函数有一个$app参数是Illuminate\Foundation\Application的实例也是一个IoC容器，该匿名函数要返回一个Illuminate\Cache\Repository的实例。<br>

要创建我们自己的缓存驱动，首先要实现Illuminate\Cache\StoreInterface接口。所以我们用MongoDB来实现的缓存驱动就可能看上去是这样：<br>

```php

	class MongoStore implements Illuminate\Cache\StoreInterface 
	{
	    public function get($key) {}
	    public function put($key, $value, $minutes) {}
	    public function increment($key, $value = 1) {}
	    public function decrement($key, $value = 1) {}
	    public function forever($key, $value) {}
	    public function forget($key) {}
	    public function flush() {}
	}

```

我们只需使用MongoDB链接来实现上面的每一个方法即可。一旦实现完毕，就可以照下面这样完成该驱动的注册：<br>

```php

	use Illuminate\Cache\Repository;

	Cache::extend('mongo', function($app)
	{
	    return new Repository(new MongoStore);
	}

```

你可以像上面的例子那样来创建Illuminate\Cache\Repository的实例。也就是说通常你不需要创建你自己的仓库类。<br>

如果你不知道要把自定义的缓存驱动代码放到哪儿，可以考虑放到Packagist里！或者你也可以在你应用的主目录下创建一个Extensions目录。比如，你的应用叫做Snappy，你可以将缓存扩展代码放到app/Snappy/Extensions/MongoStore.php。不过请记住Laravel没有对应用程序的结构做硬性规定，所以你可以按任意你喜欢的方式组织你的代码。<br>

<blockquote>
	<p>
		在哪儿调用Extend方法？<br>

		如果你还发愁在哪儿放注册代码，先考虑放到服务提供者里吧。我们之前就讲过，使用服务提供者是一种非常棒的管理你应用代码的途径。<br>
	</p>
</blockquote>	

##会话##

扩展Laravel的会话机制和上文的缓存机制一样简单。和刚才一样，我们使用extend方法来注册自定义的代码：<br>

```php

	Session::extend('mongo', function($app)
	{
	    // Return implementation of SessionHandlerInterface
	});

```

注意我们自定义的会话驱动实现的是SessionHandlerInterface接口。这个接口在PHP5.4以上版本才有。但如果你用的是PHP 5.3也别担心，Laravel会自动帮你定义这个接口的。该接口要实现的方法不多也不难。我们用MongoDB来实现就像下面这样：<br>

```php

	class MongoHandler implements SessionHandlerInterface 
	{
	    public function open($savePath, $sessionName) {}
	    public function close() {}
	    public function read($sessionId) {}
	    public function write($sessionId, $data) {}
	    public function destroy($sessionId) {}
	    public function gc($lifetime) {}
	}

```

这些方法不像刚才的StoreInterface接口定义的那么容易理解。我们来挨个简单讲讲这些方法都是干啥的：<br>

- open方法一般在基于文件的会话系统中才会用到。Laravel已经自带了一个native的会话驱动，使用的就是PHP自带的基于文件的会话系统，你可能永远也不需要在这个方法里写东西。所以留空就好。另外这也是一个接口设计的反面教材。

- close方法和open方法通常都不是必需的。对大部分驱动来说都不必要实现。

- read方法应该根据$sessionId参数来返回对应的会话数据的字符串形式。在你的会话驱动里，不论读写都不需要做任何数据序列化工作。因为Laravel会负责数据序列化的。

- write方法应该将$sessionId对应的$data字符串放置在一个持久化存储系统中。比如MongoDB，Dynamo等等。

- destroy方法应该将$sessionId对应的数据从持久化存储系统中删除。

- gc方法应该将所有时间超过参数$lifetime的数据全都删除，该参数是一个UNIX时间戳。如果你使用的是类似Memcached或Redis这种有自主到期功能的存储系统，那该方法可以留空。

一旦SessionHandlerInterface实现完毕，我们就可以将其注册进会话管理器：<br>

```php

	Session::extend('mongo', function($app)
	{
	    return new MongoHandler;
	});

```	

注册完毕后，我们就可以在app/config/session.php配置文件里使用mongo驱动了。<br>
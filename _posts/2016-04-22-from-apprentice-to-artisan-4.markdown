---
layout: post
title:  "From Apprentice To Artisan --反转控制容器"
date:   2016-04-21 13:30:15
description: "From Apprentice To Artisan --The IoC Container"
permalink: post/the-ioc-container-2
disqus:
  id: the-ioc-container-2
categories:
- blog
- php
- laravel
---

用反射来自动处理依赖是Laravel容器的一个最强大的特性。反射是一种运行时探测类和方法的能力。比如，PHP的ReflectionClass可以探测一个类的方法。method_exists某种意义上说也是一种反射。我们来把玩一下PHP的反射类，试试下面的代码吧：<br>

```php

	$reflection = new ReflectionClass('StripeBiller');
	var_dump($reflection->getMethods());
	var_dump($reflection->getConstants());

```

依靠这个强大的PHP特性，Laravel的IoC容器可以实现很有趣的功能！考虑接下来这个类：<br>

```php

	class UserController extends BaseController
	{
	    public function __construct(StripBiller $biller)
	    {
	        $this->biller = $biller;
	    }
	}

```

注意这个控制器的构造函数暗示着有一个StripBiller类型的参数。使用反射就可以检测到这种类型暗示。当Laravel的容器无法解决一个类型的明显绑定时，容器会试着使用反射来解决。程序流程类似于这样的：<br>

1. 已经有一个StripBiller的绑定了么？

2. 没绑定？那用反射来探测一下StripBiller吧。看看他都需要什么依赖。

3. 解决StripBiller需要的所有依赖。

4. 使用ReflectionClass->newInstanceArgs()来实例化StripBiller。

如你所见，容器替我们做了好多重活，这能帮你省去写大量绑定的麻烦。这就是Laravel容器最强大也是最独特的特性。熟练掌握这种能力对构建大型Laravel应用是十分有益的。<br>

下面我们修改一下控制器，改成这样会发生什么事儿呢？<br>

```php

	class UserController extends BaseController
	{
	    public function __construct(BillerInterface $biller)
	    {
	        $this->biller = $biller;
	    }
	}

```

假设我们没有为BillerInterface做任何绑定，容器该怎么知道要注入什么类呢？要知道，interface不能被实例化，因为它只是个约定。如果我们不提供更多信息的话，容器是无法实例化这个依赖的。我们需要明确指出哪个类要实现这个接口，这就需要用到bind方法：<br>

```php

	App::bind('BillerInterface','StripBiller');

```

这里我们只传了一个字符串进去，而不是一个匿名函数。这个字符串告诉容器总是使用StripBiller来作为BillerInterface的实现类。此外我们也获得了只改一行代码即可轻松改变实现的能力。比如，假设我们需要切换到BalancedPayments作为我们的支付提供商，我们只需要新写一个BalancedBiller来实现BillerInterface接口，然后这样修改容器代码：<br>

```php

	App::bind('BillerInterface', 'BalancedBiller');

```

我们的应用程序就装载上了的新支付实现代码了！<br>

你也可以使用singleton方法来实现单例模式。<br>

```php

	App::singleton('BillerInterface', 'StripBiller');

```
<blockquote>
	<p>
		掌握容器<br>

		想了解更多关于容器的知识？去读源码！容器只有一个类Illuminate\Container\Container. 读完了你就对容器有更深的认识了。<br>
	</p>
</blockquote>
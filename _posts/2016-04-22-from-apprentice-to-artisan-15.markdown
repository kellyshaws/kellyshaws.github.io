---
layout: post
title:  "From Apprentice To Artisan --扩展框架"
date:   2016-04-21 15:20:15
description: "From Apprentice To Artisan -- Extending The Framework"
permalink: post/extending-the-framework-2
disqus:
  id: extending-the-framework-2
categories:
- blog
- php
- laravel
---

身份认证模块的扩展方式和缓存与会话的扩展方式一样：使用我们熟悉的extend方法就可以进行扩展：<br>

```php

	Auth::extend('riak', function($app)
	{
	    // Return implementation of Illuminate\Auth\UserProviderInterface
	});

```

接口UserProviderInterface负责从各种持久化存储系统——如MySQL，Riak等——中获取数据，然后得到接口UserInterface的实现对象。有了这两个接口，Laravel的身份认证机制就可以不用管用户数据是如何储存的、究竟哪个类来代表用户对象这种事儿，从而继续专注于身份认证本身的实现。<br>

咱们来看一看UserProviderInterface接口的代码：<br>

```php 

	interface UserProviderInterface 
	{
	    public function retrieveById($identifier);
	    public function retrieveByCredentials(array $credentials);
	    public function validateCredentials(UserInterface $user, array $credentials);
	}

```

方法retrieveById通常接受一个数字参数用来表示一个用户，比如MySQL数据库的自增ID。该方法要找到匹配该ID的UserInterface的实现对象，并且将该对象返回。<br>

retrieveByCredentials方法接受一个参数作为登录帐号。该参数是在尝试登录系统时从Auth::attempt方法传来的。那么该方法应该查询”底层的持久化存储系统，来找到那些匹配到该帐号的用户。通常该方法会执行一个带有where条件的查询来匹配参数里的$credentials['username']。该方法不应该做任何密码验证。<br>

validateCredentials方法会通过比较$user参数和$credentials参数来检测用户是否通过认证。比如，该方法会调用$user->getAuthPassword();方法，将得到的字符串与$credentials['password']经过Hash::make处理后的结果进行比对。<br>

现在我们探索了UserProviderInterface接口的每一个方法，接下来咱们看一看UserInterface接口。别忘了UserInterface的实例应当是retrieveById和retrieveByCredentials方法的返回值：<br>

```php 

	interface UserInterface 
	{
	    public function getAuthIdentifier();
	    public function getAuthPassword();
	}

```

这个接口很简单。 getAuthIdentifier方法应当返回用户的主键。就像刚才提到的，在MySQL中可能就是自增主键了。getAuthPassword方法应当返回经过散列处理的用户密码。有了这个接口，身份认证系统就可以不用关心用户类到底使用了什么ORM或者什么存储方式。Laravel已经在app/models目录下，包含了一个默认的User类且实现了该接口。所以你可以参考这个类当例子。<br>

当我们最后实现了UserProviderInterface接口后，我们可以将该扩展注册进Auth里面：<br>

```php

	Auth::extend('riak', function($app)
	{
	    return new RiakUserProvider($app['riak.connection']);
	});

```

使用extend方法注册好驱动以后，你就可以在app/config/auth.php配置文件里面切换到新的驱动了。<br>

##使用容器进行扩展##

Laravel框架内几乎所有的服务提供者都会绑定一些对象到IoC容器里。你可以在app/config/app.php文件里找到服务提供者列表。如果你有时间的话，你应该大致过一遍每个服务提供者的源码。这么做你便可以对每个服务提供者有更深的理解，明白他们都往框架里加了什么东西，对应的什么键。那些键就用来联系着各种各样的服务。<br>

举个例子，PaginationServiceProvider向容器内绑定了一个paginator键，对应着一个Illuminate\Pagination\Environment的实例。你可以很容易的通过覆盖容器绑定来扩展重写该类。比如，你可以创建一个扩展自Environment类的子类：<br>

```php

	namespace Snappy\Extensions\Pagination;

	class Environment extends \Illuminate\Pagination\Environment 
	{
	    //
	}

```

子类写好以后，你可以再创建个新的SnappyPaginationProvider服务提供者来扩展其boot方法，在里面覆盖paginator：<br>

```php

	class SnappyPaginationProvider extends PaginationServiceProvider
	{
	    public function boot()
	    {
	        App::bind('paginator', function()
	        {
	            return new Snappy\Extensions\Pagination\Environment;
	        }

	        parent::boot();
	    }
	}

```

注意这里我们继承了PaginationServiceProvider，而非默认的基类ServiceProvider。扩展的服务提供者编写完毕后，就可以在app/config/app.php文件里将PaginationServiceProvider替换为你刚扩展的那个类了。<br>

这就是扩展绑定进容器的核心类的一般方法。基本上每一个核心类都以这种方式绑定进了容器，都可以被重写。还是那一句话，读一遍框架内的服务提供者源码吧。这有助于你熟悉各种类是怎么绑定进容器的，都绑定的是哪些键。这是学习Laravel框架到底如何运转的好方法。<br>

##请求的扩展##

由于他是框架里面非常基础的部分，并且在请求流程中很早就被实例化，所以要扩展Request类的方法与之前相比是有些许不同的。<br>

首先还是要写个子类：<br>

```php 

	namespace QuickBill\Extensions;

	class Request extends \Illuminate\Http\Request 
	{
	    // Custom, helpful methods here...
	}

```

子类写好后，打开bootstrap/start.php文件。该文件是应用的请求流程中最早被载入的几个文件之一。要注意被执行的第一个动作是创建Laravel的$app实例：<br>

```php

	$app = new \Illuminate\Foundation\Application;

```

当新的应用实例创建后，它将会创建一个Illuminate\Http\Request的实例并且将其绑定到IoC容器里，键名为request。所以我们需要找个方法来将一个自定义的类指定为默认的请求类，对不对？而且幸运的是，应用实例有一个名为requestClass的方法就是用来干这事儿的！所以我们只需要在bootstrap/start.php文件最上面加一行：<br>

```php

	use Illuminate\Foundation\Application;

	Application::requestClass('QuickBill\Extensions\Request');

```

一旦你指定了自定义的请求类，Laravel将在任何时候都可以使用这个Request类的实例。并使你很方便的能随时访问到它，甚至单元测试也不例外！<br>

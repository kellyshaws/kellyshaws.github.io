---
layout: post
title:  "From Apprentice To Artisan --扩展框架"
date:   2016-04-21 15:10:15
description: "From Apprentice To Artisan -- Extending The Framework"
permalink: post/extending-the-framework
disqus:
  id: extending-the-framework
categories:
- laravel
---

##展框架

为了方便你自定义框架核心组件，Laravel提供了大量可以扩展的地方。你甚至可以完全替换掉旧组件。例如：哈希器遵守了HasherInterface接口，你可以按照你自己应用的需求来重新实现。你也可以扩展Request对象，添加你自己用的顺手的helper方法。你甚至可以添加全新的身份认证、缓存和会话机制！<br>

Laravel组件通常有两种扩展方式：在IoC容器里面绑定新实现，或者用Manager类注册一个扩展，该扩展采用了工厂模式实现。在本章中我们将探索不同的扩展方式并检查我们都需要些什么代码。<br>

<blockquote>
<p>
    扩展方式<br>

    要记住Laravel通常有以下两种扩展方式：通过IoC绑定和通过Manager类。其中管理类实现了工厂设计模式，负责组件的实例化。比如缓存和会话机制。<br>
</p>
</blockquote>

##管理者和工厂

Laravel有好多Manager类用来管理基于驱动的组件的生成过程。基于驱动的组件包括：缓存、会话、身份认证、队列组件等。管理类负责根据应用程序的配置，来生成特定的驱动实例。比如：CacheManager可以创建APC、Memcached、Native、还有其他不同的缓存驱动的实现。<br>

每个管理类都包含名为extend的方法，该方法可用于将新功能注入到管理类中。下面我们将逐个介绍管理类，为你展示如何注入自定义的驱动。<br>

<blockquote>
<p>
    如何了解你的管理类<br>

    请花点时间看看Laravel中各个Manager类的代码，比如CacheManager和SessionManager。通过阅读这些代码能让你对Laravel的管理类机制更加清楚透彻。所有的管理类都继承自Illuminate\Support\Manager基类，该基类为每一个管理类提供了一些有效且通用的功能。<br>
</p>
</blockquote>

##缓存

要扩展Laravel的缓存机制，我们将使用CacheManager里的extend方法来绑定我们自定义的缓存驱动。扩展其他的管理类也是类似的。比如，我们想注册一个新的缓存驱动，名叫"mongo"，代码可以这样写：<br>

```
Cache::extend('mongo', function($app)
{
    // Return Illuminate\Cache\Repository instance...
});
```

extend方法的第一个参数是你要定义的驱动的名字。该名字对应着app/config/cache.php配置文件中的driver项。第二个参数是一个匿名函数（闭包），该匿名函数有一个$app参数是Illuminate\Foundation\Application的实例也是一个IoC容器，该匿名函数要返回一个Illuminate\Cache\Repository的实例。<br>

要创建我们自己的缓存驱动，首先要实现Illuminate\Cache\StoreInterface接口。所以我们用MongoDB来实现的缓存驱动就可能看上去是这样：<br>

```
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

```
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

##会话

扩展Laravel的会话机制和上文的缓存机制一样简单。和刚才一样，我们使用extend方法来注册自定义的代码：<br>

```
Session::extend('mongo', function($app)
{
    // Return implementation of SessionHandlerInterface
});
```

注意我们自定义的会话驱动实现的是SessionHandlerInterface接口。这个接口在PHP5.4以上版本才有。但如果你用的是PHP 5.3也别担心，Laravel会自动帮你定义这个接口的。该接口要实现的方法不多也不难。我们用MongoDB来实现就像下面这样：<br>

```
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

```
Session::extend('mongo', function($app)
{
    return new MongoHandler;
});
```	

注册完毕后，我们就可以在app/config/session.php配置文件里使用mongo驱动了。<br>

##身份认证

身份认证模块的扩展方式和缓存与会话的扩展方式一样：使用我们熟悉的extend方法就可以进行扩展：<br>

```
Auth::extend('riak', function($app)
{
    // Return implementation of Illuminate\Auth\UserProviderInterface
});
```

接口UserProviderInterface负责从各种持久化存储系统——如MySQL，Riak等——中获取数据，然后得到接口UserInterface的实现对象。有了这两个接口，Laravel的身份认证机制就可以不用管用户数据是如何储存的、究竟哪个类来代表用户对象这种事儿，从而继续专注于身份认证本身的实现。<br>

咱们来看一看UserProviderInterface接口的代码：<br>

```
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

``` 
interface UserInterface 
{
    public function getAuthIdentifier();
    public function getAuthPassword();
}
```

这个接口很简单。 getAuthIdentifier方法应当返回用户的主键。就像刚才提到的，在MySQL中可能就是自增主键了。getAuthPassword方法应当返回经过散列处理的用户密码。有了这个接口，身份认证系统就可以不用关心用户类到底使用了什么ORM或者什么存储方式。Laravel已经在app/models目录下，包含了一个默认的User类且实现了该接口。所以你可以参考这个类当例子。<br>

当我们最后实现了UserProviderInterface接口后，我们可以将该扩展注册进Auth里面：<br>

```
Auth::extend('riak', function($app)
{
    return new RiakUserProvider($app['riak.connection']);
});
```

使用extend方法注册好驱动以后，你就可以在app/config/auth.php配置文件里面切换到新的驱动了。<br>

##使用容器进行扩展

Laravel框架内几乎所有的服务提供者都会绑定一些对象到IoC容器里。你可以在app/config/app.php文件里找到服务提供者列表。如果你有时间的话，你应该大致过一遍每个服务提供者的源码。这么做你便可以对每个服务提供者有更深的理解，明白他们都往框架里加了什么东西，对应的什么键。那些键就用来联系着各种各样的服务。<br>

举个例子，PaginationServiceProvider向容器内绑定了一个paginator键，对应着一个Illuminate\Pagination\Environment的实例。你可以很容易的通过覆盖容器绑定来扩展重写该类。比如，你可以创建一个扩展自Environment类的子类：<br>

```
namespace Snappy\Extensions\Pagination;

class Environment extends \Illuminate\Pagination\Environment 
{
    //
}
```

子类写好以后，你可以再创建个新的SnappyPaginationProvider服务提供者来扩展其boot方法，在里面覆盖paginator：<br>

```
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

##请求的扩展

由于他是框架里面非常基础的部分，并且在请求流程中很早就被实例化，所以要扩展Request类的方法与之前相比是有些许不同的。<br>

首先还是要写个子类：<br>

``` 
namespace QuickBill\Extensions;

class Request extends \Illuminate\Http\Request 
{
    // Custom, helpful methods here...
}
```

子类写好后，打开bootstrap/start.php文件。该文件是应用的请求流程中最早被载入的几个文件之一。要注意被执行的第一个动作是创建Laravel的$app实例：<br>

```
$app = new \Illuminate\Foundation\Application;
```

当新的应用实例创建后，它将会创建一个Illuminate\Http\Request的实例并且将其绑定到IoC容器里，键名为request。所以我们需要找个方法来将一个自定义的类指定为默认的请求类，对不对？而且幸运的是，应用实例有一个名为requestClass的方法就是用来干这事儿的！所以我们只需要在bootstrap/start.php文件最上面加一行：<br>

```
use Illuminate\Foundation\Application;

Application::requestClass('QuickBill\Extensions\Request');
```

一旦你指定了自定义的请求类，Laravel将在任何时候都可以使用这个Request类的实例。并使你很方便的能随时访问到它，甚至单元测试也不例外！<br>

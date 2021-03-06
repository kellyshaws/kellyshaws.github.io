---
layout: post
title:  "From Apprentice To Artisan --服务提供者"
date:   2016-04-21 14:00:15
description: "From Apprentice To Artisan --Service Providers"
permalink: post/service-providers
disqus:
  id: service-providers
categories:
- laravel
---

服务提供者
=========

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

延迟加载的服务提供者
=================

并非在你配置文件中的providers数组里的所有提供者在每次请求都会被实例化。否则会对性能不利，尤其是这个服务的功能用不到的情况下。比如，QueueServiceProvider服务就不是每次都用得到。<br>

为了达到只实例化需要的服务的提供者，Laravel生成了服务清单并且储存在了app/storage/meta目录下。这份清单列出了应用里所有的服务提供者，包括容器绑定的名字也记录了。这样，当应用想让容器取出一个名为queue的绑定时，Laravel知道需要先实例化并运行QueueServiceProvider因为在服务清单里记录着该服务提供者能提供queue的绑定。如此这般框架就能够延迟加载每个请求需要的服务了，性能大大提高。<br>

<blockquote>
<p>
    如何生成服务清单<br>

    当你在providers数组里新增一条，Laravel在下一次请求时就会自动重新生成服务清单。<br>
</p>
</blockquote>

如果你有时间，去看看服务清单文件里面的内容。理解这个文件的结构有助于你对服务进行排错。<br>

作为管理工具
==========

想制作一个结构优美的Laravel应用的话，就要去学习如何用服务提供者来管理代码。当你在注册IoC绑定的时候，所有代码都杂乱的塞进了app/start路径下的文件里。别再这样做了，使用服务提供者来注册这些吧。<br>

<blockquote>
<p>
    万物之初<br>

    你应用的"启动"文件都储存在app/start目录下。根据不同的请求入口，系统会载入不同的启动文件。在全局的start.php文件加载后，系统会根据执行环境的不同来加载不同的启动文件。此外，在执行命令行程序时，artisan.php文件会被载入。<br>
</p>
</blockquote>

咱们来考虑这个例子。也许我们的应用正在使用Pusher来为客户推送消息。为了将我们的应用和Pusher解耦，我们要定义EventPusherInterface接口和对应的实现类PusherEventPusher。这样在需求变化或应用改进时，我们就可以随时轻松的改变推送服务提供商。<br>

```
interface EventPusherInterface
{
    public function push($message, array $data = array());
}

class PusherEventPusher implements EventPusherInterface
{
    public function __construct(PusherSdk $pusher)
    {
        $this->pusher = $pusher;
    }

    public function push($message, array $data = array())
    {
        // Push message via the Pusher SDK...
    }
}
```

接下来我们创建一个EventPusherServiceProvider：<br>

```
use Illuminate\Support\ServiceProvider;

class EventPusherServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton('PusherSdk', function()
        {
            return new PusherSdk('app-key', 'secret-key');
        }

        $this->app->singleton('EventPusherInterface', 'PusherEventPusher');
    }
}
```

很好！我们对事件推送进行了清晰的抽象，同时我们也有了一个很不错的地方进行注册、绑定其他相关的东西到容器里。最后一步只需要将EventPusherServiceProvider写入app/config/app.php文件内的providers数组里就可以了。现在这个应用里的EventPusherInterface已经被绑定到了正确的实现类上。<br>

<blockquote>
<p>
    要使用单例么？<br>

    用不用单例可以这样来考虑：如果再一次请求周期中该类只需要有一个实例，就使用singleton；否则就使用bind。<br>
</p>
</blockquote>

```
App::singleton('EventPusherInterface', 'PusherEventPusher');
```

当然服务提供者的功能不仅仅局限于消息推送。像是云存储、数据库访问、自定义的视图引擎比如Twig等等都可以用这种模式来设置。服务提供者就是你的应用里的启动代码和管理工具，没什么神奇的。<br>

所以大胆的去创建你自己的服务提供者。并不是你非要发布个什么软件包才需要服务提供者，他们只是非常好的管理代码的工具。使用它们的力量去管理好应用中的各个组件吧。<br>

服务提供者的启动过程
================

在所有服务提供者都注册以后，他们就进入了“启动”过程。该过程会触发每个服务提供者的boot方法。这里会发生一种常见的错误用法：在register方法里面调用其他的服务。由于在register方法里我们不能保证所有其他服务都已经被加载，所以在该方法里调用别的服务有可能会出错。所以如果你想在服务提供者里调用别的服务，请在boot方法里做这种事儿。register方法只能进行容器注册。<br>

在启动方法里面，你想做什么都可以：注册事件监听，引入路由文件，注册过滤器，或者其他你能想象到的事儿。再强调一下，要发挥服务提供者的管理功能。可能你想将相关的多个事件监听归为一组？将他们放到一个服务提供者的boot方法里，这会很管用的！或者你也可以引入单独的events、routesPHP文件：<br>

```
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

核心也是服务提供者的模式
===================

你可能已经注意到，在app配置文件里面已经有了很多服务提供者。每一个都负责启动框架核心的一部分。比如MigrationServiceProvider负责启动数据库迁移的类，包括Artisan里面的命令。EventServiceProvide负责启动和注册事件调度机制。不同的服务提供者有着不同的复杂度，但他们都负责启动核心的一部分。<br>

<blockquote>
<p>
    和服务提供者们见见面<br>

    理解Laravel核心的最好方法是去读它的核心服务源码。如果你对这些服务的源码、容器注册等都很熟悉，那么你对Laravel是如何工作的将会有十分深刻的理解。<br>
</p>
</blockquote>

大部分的服务提供者是延迟加载的，意味着并非所有请求都会调用到他们；然而有一些很基础的服务是每一次请求都会被加载的，比如FilesystemServiceProvide和ExceptionServiceProvider。有人会说核心服务提供者和应用程序容器就是Laravel。Laravel其实是将这么多不同部分联系起来，形成一个单一的、内聚的整体的这么一个机制。拿建筑来比喻，那些服务提供者就是框架的预制模块。<br>

正如之前提到的那样，如果你想更深的了解框架是如何运行的，请读Lravel的核心服务的源码吧。读过之后，你会对框架如何将各部分组合在一起、每一个服务是如何为你所用这些机制有更坚实的理解。此外，有了这些进一步的理解，你也可以为Laravel添砖加瓦！<br>

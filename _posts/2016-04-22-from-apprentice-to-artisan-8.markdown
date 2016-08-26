---
layout: post
title:  "From Apprentice To Artisan --服务提供者"
date:   2016-04-21 14:10:15
description: "From Apprentice To Artisan --Service Providers"
permalink: post/service-providers-2
disqus:
  id: service-providers-2
categories:
- blog
- php
- laravel
---

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
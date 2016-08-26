---
layout: post
title:  "From Apprentice To Artisan --应用程序结构"
date:   2016-04-21 14:50:15
description: "From Apprentice To Artisan --Application Structure"
permalink: post/application-structure-2
disqus:
  id: application-structure-2
categories:
- blog
- php
- laravel
---

当用Laravel开发应用时，你可能迷惑于应该把各种东西都放在哪儿。比如，辅助函数要放在哪里？事件监听器要放在哪里？视图组件要放在哪里？答案可能出乎你的意料——想放哪儿都行！Laravel并没有很多在文件系统上的约定。不过这个答案的确不能让人满意，所以下面我们就这个问题展开探索，探索这些东西究竟可以放在哪儿。<br>

##辅助函数##

Laravel有一个文件里面都是辅助函数。你或许希望创建一个类似的文件来存储你自己的辅助函数。"start"文件是个不错的入口，该文件会在应用的每一次请求时被访问。在start/global.php里，你可以引入你自己写的helpers.php文件，就像这样：<br>

```
//Within app/start/global.php

require_once __DIR__.'/../helpers.php';
```

##事件监听器##

事件监听器当然不该放到routes.php文件里面，若直接放到"start"目录下的文件里会比较乱，所以我们要找另外的地方来存放。服务提供者是个好地方。我们之前了解到，服务提供者可不仅仅是用来做依赖注入绑定，还可以干其他事儿。可以将事件监听器用服务提供者来管理起来，让代码更整洁，不至于影响到你应用的主要逻辑代码。视图组件其实和事件差不多，也可以类似的放到服务提供者里面。<br>

```
namespace QuickBill\Providers;

use Illuminate\Support\ServiceProvider;

class BillingEventsProvider extends ServiceProvider
{
    public function boot()
    {
        Event::listen('billing.failed', function($bill)
        {
            // Handle failed billing event...
        });
    }
}
```

创建好服务提供者后，就可以将它加入到app/config/app.php配置文件的providers数组里。<br>

<blockquote>
	<p>
		注意启动流程<br>

		记住在上面的例子里面，我们在boot方法里进行编写是有原因的。register方法只能用来进行依赖注入绑定。<br>
	</p>
</blockquote>    

##错误处理##

如果你的应用里面有很多自定义的错误处理方法，那你的“启动”文件可能会很臃肿。和刚才的事件监听器一样，错误处理方法也最好放到服务提供者里面。这种服务提供者可以命名为像QuickBillErrorProvider这种。然后你在boot方法里想注册多少错误处理方法都可以了。重申一下精神：让呆板的代码离你应用的业务逻辑越远越好。下方展示了这种服务提供者的一种可能的书写方法：<br>

```
namespace QuickBill\Providers;

use App, Illuminate\Support\ServiceProvider;

class QuickBillErrorProvider extends ServiceProvider {
    public function register()
    {    
        //
    }

    public function boot()
    {
        App::error(function(BillingFailedException $e)
        {
            // Handle failed billing exceptions ...
        });
    }
}
```

<blockquote>
	<p>
		简便做法<br>

		当然如果你只有一两条简单的错误处理方法，那么都写在“启动”文件里面也是一种又快又好的简便做法。<br>
	</p>
</blockquote>

##其他##

通常只要遵循PSR-4就可以保持类的整洁。命令式的代码比如事件监听器、错误处理器还有其他注册性质的操作都可以放在服务提供者里面。对于什么代码要放在什么地方这个问题，结合你目前为止学到的知识，应当可以给出一个有理有据的答案了。但永远不要害怕试验。Laravel最美妙之处就是你可以做出最适合你自己的风格。去探索和发现最适合你自己应用的结构吧，别忘了和他人分享你的见解！<br>

例如你可能注意到我们上面的例子，你可以创建个Providers的命名空间来存放你自己写的服务提供者，目录就类似于这样：<br>

```
// app
    // QuickBill
        // Billing
        // Extensions
            //Pagination
                -> Environment.php
        // Providers
            -> EventPusherServiceProvider.php
        // Repositories
        User.php
        Payment.php
```

看上面的例子我们有Providers和Extensions两个命名空间。你自己写的服务提供者可以放到Providers命名空间下。那个Extensions命名空间可以用来存放你对框架核心进行扩展的类。<br>
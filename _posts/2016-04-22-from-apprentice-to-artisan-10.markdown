---
layout: post
title:  "From Apprentice To Artisan --应用程序结构"
date:   2016-04-21 14:30:15
description: "From Apprentice To Artisan --Application Structure"
permalink: post/application-structure
disqus:
  id: application-structure
categories:
- laravel
---

##应用结构

这个类要写到哪儿？这是一个在用框架写应用程序时十分常见的问题。大量的开发人员都有这个疑问。他们被灌输Model就是Database，在控制器里面处理HTTP请求，在模型里操作数据库，视图里包含了要显示的HTML。不过，发送电子邮件的类要写到哪儿？数据验证的类要写到哪儿？调用外部API的类要写到哪儿？在这一章节，我们将学习如何写结构优美的Laravel应用，打破长久以来掣肘开发人员的普遍思维惯性这个拦路虎，最终做出好的设计。<br>

##MVC是慢性谋杀

为了做出好的程序设计，最大的拦路虎就是一个简单的缩写词：
M-V-C。模型、视图、控制器主宰了Web框架的思想已经好多年了。这种思想的流行某种程度上是托了Ruby on Rails愈加流行的福。然而，如果你问一个开发人员模型的定义是什么。通常你会听到他嘟哝着什么数据库之类的东西。这么说，模型就是数据库了。不管这意味着什么，模型里包含了关于数据库的一切。但是，你很快就会知道，你的应用程序需要的不仅仅是一个简单的数据库访问类。他需要更多的逻辑如：数据验证、调用外部服务、发送电子邮件，等等更多。<br>

<blockquote>
<p>
    模型是啥？<br>

    单词"model"的含义太模糊了，很难说行具体的含义。更具体来讲，模型是用来将我们的应用划分成更小、更清晰的类，使得各代码部分有着明确的权责。<br>
</p>
</blockquote>

所以怎么解决这个问题呢？很多开发者开始将业务逻辑包装到控制器里面。当控制器庞大到一定规模，他们将会需要重用业务逻辑。大部分开发人员没有将这些业务逻辑提取到别的类里面，而是错误的臆想他们需要在控制器里面调用别的控制器。这种模式通常被称为"HMVC"。不幸的是，这种模式通常也预示着糟糕的程序设计，并且控制器已经太复杂了。<br>

<blockquote>
<p>
    HMVC预示着糟糕的设计<br>

    你觉得需要在控制器里面调用其他的控制器？这通常预示着糟糕的程序设计并且你的控制器里面业务逻辑太多了。把业务逻辑抽出来放到一个新的类里面，这样你就可以在其他任何控制器里面调用了。<br>
</p>
</blockquote>

有一种更好的程序结构。但首先我们要忘掉以往我们被灌输的关于model的一切。干脆点，让我们直接删掉model目录，重新开始吧！<br>

##再见，模型

删掉你的models目录了么？还没删就赶紧删了！我们将要在app目录下创建个新的目录，目录名就以我们这个应用的名字来命名，这次我们就叫QuickBill吧。在后续的讨论中，我们在前面写的那些接口和类都会出现。<br>

<blockquote>
<p>
    注意使用场景<br>

    记住，如果你在写一个很小的Laravel应用，那在models目录下写几个Eloquent模型其实挺合适的。但在本章节，我们主要关注如何开发更有合适层次架构的大型复杂项目。<br>
</p>
</blockquote>

这样我们现在有了个app/QuickBill目录，它和应用目录下的其他目录如controllers还有views都是平级的。在QuickBill目录下我们还可以创建几个其他的目录。我们来在里面创建个Repositories和Billing目录。目录都创建好以后，别忘了在composer.json文件里加入PSR-4的自动载入机制：<br>

```
"psr-4":  {
    "QuickBill\\":    "app/QuickBill"
}
```    

现在我们把继承自Eloquent的模型类都放到QuickBill目录下面。这样我们就能很方便的以QuickBill\User,QuickBill\Payment的方式来使用它们。Repositories目录属于PaymentRepository 和UserRepository这种类，里面包含了所有对数据的访问功能比如getRecentPayments和getRichestUser。Billing目录应当包含调用第三方支付服务的类。整个目录结构应该类似这样：<br>

```
// app
    // QuickBill
        // Repositories
            -> UserRepository.php
            -> PaymentRepository.php
        // Billing
            -> BillerInterface.php
            -> StripeBiller.php
        // Notifications
            -> BillingNotifierInterface.php
            -> SmsBillingNotifier.php
        User.php
        Payment.php
```     

<blockquote>
<p>
    数据验证怎么办？<br>

    在哪儿进行数据验证常常困扰着开发人员。可以考虑将数据验证方法写进你的"实体"类里面。方法名可以设为validForCreation或hasValidDomain。或者你也可以专门创建个验证器类UserValidator，放到Validation命名空间下，然后将这个验证器类注入到你的repository类里面。两种方式你都可以试试，看哪个你更喜欢！<br>
</p>
</blockquote>   

摆脱了models目录后，你通常就能克服心理障碍，实现好的设计。使得你能创建一个更合适的目录结构来为你的应用服务。当然，你建立的每一个应用程序都会有一定的相似之处，因为每个复杂的应用程序都需要一个数据访问层，一些外部服务层等等。<br>

<blockquote>
<p>
    别害怕目录<br>

    不要惧怕建立目录来管理应用。要常常将你的应用切割成小组件，每一个组件都要有十分专注的职责。跳出模型的框框来思考。比如我们之前就说过，你可以创建个Repositories目录来存放你所有的数据访问类。<br>
</p>
</blockquote>

##核心思想就是分层

你可能注意到，优化应用的设计结构的关键就是责任划分，或者说是创建不同的责任层次。控制器只负责接收和响应HTTP请求然后调用合适的业务逻辑层的类。你的业务逻辑/领域逻辑层才是你真正的程序。你的程序包含了读取数据，验证数据，执行支付，发送电子邮件，还有你程序里任何其他的功能。事实上你的领域逻辑层不需要知道任何关于"网络"的事情！网络仅仅是个访问你程序的传输机制，关于网络和HTTP请求的一切不应该超出路由和控制器层。做出好的设计的确很有挑战性，但好的设计也会带来可持续发展的清晰的好代码。<br>

举个例子。与其在你业务逻辑类里面直接获取网络请求，不如你直接把网络请求从控制器传给你的业务逻辑类。这个简单的改动将你的业务逻辑类和"网络"分离开了，并且不必担心怎么去模拟网络请求，你的业务逻辑类就可以简单的测试了：<br>

```
class BillingController extends BaseController
{
    public function __construct(BillerInterface $biller)
    {
        $this->biller = $biller;
    }
    
    public function postCharge()
    {
        $this->biller->chargeAccount(Auth::user(), Input::get('amount'));
        return View::make('charge.success');
    }
}
```

现在chargeAccount方法更容易测试了。我们把Request和Input从BillingInterface里提出来，然后在控制器里把方法需要的支付金额直接传过去。<br>

编写拥有高可维护性应用程序的关键之一，就是责任分割。要时常检查一个类是否管得太宽。你要常常问自己"这个类需不需要关心XXX呢？"如果答案是否定的，那么把这块逻辑抽出来放到另一个类里面，然后用依赖注入的方式进行处理。<br>

<blockquote>
<p>
    Single Reason To Change <br>

    如何判断一个类是否管得太宽，有一个有用的方法就是检查你为什么要改这块儿代码。举个例子：当我们想调整通知逻辑的时候，我们需要修改Biller的实现代码么？当然不需要，Biller的实现仅仅需要考虑支付，它与通知逻辑应当仅通过约定来进行交互。使用这种思路过一遍代码，会让你很快找出应用中需要改进的地方。<br>
</p>
</blockquote>

##东西都放哪儿？

当用Laravel开发应用时，你可能迷惑于应该把各种东西都放在哪儿。比如，辅助函数要放在哪里？事件监听器要放在哪里？视图组件要放在哪里？答案可能出乎你的意料——想放哪儿都行！Laravel并没有很多在文件系统上的约定。不过这个答案的确不能让人满意，所以下面我们就这个问题展开探索，探索这些东西究竟可以放在哪儿。<br>

##辅助函数

Laravel有一个文件里面都是辅助函数。你或许希望创建一个类似的文件来存储你自己的辅助函数。"start"文件是个不错的入口，该文件会在应用的每一次请求时被访问。在start/global.php里，你可以引入你自己写的helpers.php文件，就像这样：<br>

```
//Within app/start/global.php

require_once __DIR__.'/../helpers.php';
```

##事件监听器

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

class QuickBillErrorProvider extends ServiceProvider 
{
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

##其他

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

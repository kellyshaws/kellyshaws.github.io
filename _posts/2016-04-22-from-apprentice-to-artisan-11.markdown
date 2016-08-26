---
layout: post
title:  "From Apprentice To Artisan --应用程序结构"
date:   2016-04-21 14:40:15
description: "From Apprentice To Artisan --Application Structure"
permalink: post/application-structure
disqus:
  id: application-structure
categories:
- blog
- php
- laravel
---

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

##核心思想就是分层##

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
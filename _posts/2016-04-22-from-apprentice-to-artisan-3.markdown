---
layout: post
title:  "From Apprentice To Artisan --反转控制容器"
date:   2016-04-21 13:20:15
description: "From Apprentice To Artisan --The IoC Container"
permalink: post/the-ioc-container
disqus:
  id: the-ioc-container
categories:
- laravel
---

##基础绑定

我们已经学习了依赖注入，接下来咱们一起来探索IoC Container。IoC容器可以使你更容易管理依赖注入，Laravel框架拥有一个很强大的IoC容器。Laravel的核心就是这个IoC容器，这个IoC容器使得框架各个组件能很好的在一起工作。事实上Laravel的Application类就是继承自Container类！<br>


在Laravel应用里，你可以通过App来访问控制反转容器。容器有很多方法，不过我们从最基础的开始。让我们继续使用上一章写的BillerInterface和BillingNotifierInterface，且假设我们使用了Stripe来进行支付操作。我们可以将Stripe的支付实现绑定到容器里，就像这样：<br>

```
App::bind('BillerInterface', function()
{
    return new StripeBiller(App::make('BillingNotifierInterface'));
});
```

注意在我们处理BillingInterface时，我们额外需要一个BillingNotifierInterface的实现，也就是再来一个bind：<br>

```
App::bind('BillingNotifierInterface', function()
{
    return new EmailBillingNotifier;
});
```

如你所见，这个容器就是个用来存储各种绑定的地方。一旦一个类在容器里绑定了以后，我们可以很容易的在应用的任何位置调用它。我们甚至可以在bind函数内写另外的bind。<br>

<blockquote>
<p>
    Have Acne?<br>

    Laravel框架的Illuminate容器和另一个名为Pimple的IoC容器是可替换的。所以如果你之前用的是Pimple，你尽可以大胆的升级为Illuminate Container，后者还有更多新功能！<br>
</p>
</blockquote>

一旦我们使用了容器，切换接口的实现就是一行代码的事儿。<br>

比如考虑以下代码：<br>

```
class UserController extends BaseController
{
    public function __construct(BillerInterface $biller)
    {
        $this->biller = $biller;
    }
}
```

当这个控制器通被容器实例化后，包含着EmailBillingNotifier的StripeBiller会被注入到这个控制器中。如果我们现在想要换一种提示方式，我们可以简单的将代码改为这样：<br>


```
App::bind('BillingNotifierInterface', function()
{
    return new SmsBillingNotifier;
});
```

现在不管在应用的哪里需要一个提示器，我们总会得到SmsBillingNotifier的对象。利用这种结构，我们的应用可以在不同的实现方式之间快速切换。<br>

只改一行就能切换代码实现，这可是很厉害的能力。比如我们想把短信服务从原来的提供商替换为Twilio。我们可以开发一个新的Twilio的提示器类然后修改绑定语句。如果Twilio有任何闪失，我们只需修改一行代码就可以快速的切换回原来的短信提供商。看到了吧，依赖注入的好处多得很呢。你能再想出几个使用依赖注入和控制反转容器的好处么？<br>

想在应用中只实例化某类一次？没问题，使用singleton方法吧：<br>

```
App::singleton('BillingNotifierInterface', function()
{
    return new SmsBillingNotifier;
});
```

这样只要这个容器生成了这个提示器对象一次，在接下来的生成请求中容器都只会提供这同样的一个对象。<br>

容器的instance方法和singleton方法很类似，区别是instance可以绑定一个已经存在的对象。然后容器每次返回的都是这个对象了。<br>

```
$notifier = new SmsBillingNotifier;
App::instance('BillingNotifierInterface', $notifier);
```

现在我们熟悉了容器的基础用法，让我们深入发掘它更强大的功能：依靠反射来处理类和接口。<br>

##反射解决方案

用反射来自动处理依赖是Laravel容器的一个最强大的特性。反射是一种运行时探测类和方法的能力。比如，PHP的ReflectionClass可以探测一个类的方法。method_exists某种意义上说也是一种反射。我们来把玩一下PHP的反射类，试试下面的代码吧：<br>

```
$reflection = new ReflectionClass('StripeBiller');
var_dump($reflection->getMethods());
var_dump($reflection->getConstants());
```

依靠这个强大的PHP特性，Laravel的IoC容器可以实现很有趣的功能！考虑接下来这个类：<br>

```
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

```
class UserController extends BaseController
{
    public function __construct(BillerInterface $biller)
    {
        $this->biller = $biller;
    }
}
```

假设我们没有为BillerInterface做任何绑定，容器该怎么知道要注入什么类呢？要知道，interface不能被实例化，因为它只是个约定。如果我们不提供更多信息的话，容器是无法实例化这个依赖的。我们需要明确指出哪个类要实现这个接口，这就需要用到bind方法：<br>

```
App::bind('BillerInterface','StripBiller');
```

这里我们只传了一个字符串进去，而不是一个匿名函数。这个字符串告诉容器总是使用StripBiller来作为BillerInterface的实现类。此外我们也获得了只改一行代码即可轻松改变实现的能力。比如，假设我们需要切换到BalancedPayments作为我们的支付提供商，我们只需要新写一个BalancedBiller来实BillerInterface接口，然后这样修改容器代码：<br>

```
App::bind('BillerInterface', 'BalancedBiller');
```

我们的应用程序就装载上了的新支付实现代码了！<br>

你也可以使用singleton方法来实现单例模式。<br>

```
App::singleton('BillerInterface', 'StripBiller');
```

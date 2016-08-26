---
layout: post
title:  "From Apprentice To Artisan --依赖注入"
date:   2016-04-21 13:10:15
description: "From Apprentice To Artisan --Dependency Injection"
permalink: post/dependency-injection-2
disqus:
  id: dependency-injection-2
categories:
- blog
- php
- laravel
---

让我们考虑另一个例子来巩固理解。可能我们想要去提醒用户该交钱了。我们会定义两个接口，或者约定。这些约定使我们在更改实际实现时更加灵活。<br>

```
interface BillerInterface 
{
    public function bill(array $user, $amount);
}

interface BillingNotifierInterface 
{
    public function notify(array $user, $amount);
}
```

接下来我们要写一个BillerInterface的实现：<br>

```
class StripeBiller implements BillerInterface
{
    public function __construct(BillingNotifierInterface $notifier)
    {
        $this->notifier = $notifier;
    }
    
    public function bill(array $user, $amount)
    {
        // Bill the user via Stripe...
        $this->notifier->notify($user, $amount);
    }
}
```

只要遵守了每个类的责任划分，我们很容易将不同的提示器注入到账单类里面。比如，我们可以注入一个SmsNotifier或者EmailNotifier。账单类只要遵守了约定，就不用再考虑如何实现提示功能。只要是遵守约定的类，账单类都能用。这不仅仅是方便了我们的开发，而且我们还可以通过模拟BillingNotifierInterface来进行无痛测试。<br>

<blockquote>
	<p>
		使用接口<br>

		写接口可能看上去挺麻烦，但实际上能加速你的开发。你不用实现任何接口，就能使用模拟库来模拟你的接口，进而测试整个后台逻辑！<br>
	</p>
</blockquote>

那我们如何做依赖注入呢？很简单：<br>

``` 
$biller = new StripeBiller(new SmsNotifier);
```

这就是依赖注入。biller不需再考虑提醒用户的事儿，我们直接传给他一个提示器。这种微小的改动能使你的应用焕然一新。你的代码马上就变得更容易维护，因为明确指定了类的职责边界。并且更容易测试，你只需使用模拟依赖即可。<br>


那IoC容器呢？难道依赖注入不需要IoC容器么？当然不需要！在接下来的章节里面你会了解到，容器使得依赖注入更易于管理，但是容器不是依赖注入所必须的。只要遵循本章提出的原则，你可以在你任何的项目里面实施依赖注入，而不必管该项目是否使用了容器。<br>

##太像Java了？##

有人会说使用接口让PHP代码看上去太像Java了——即代码太罗嗦了——你必须定义接口然后实现它，要多按好多下键盘。<br>

对于小而简单的应用来说，以上说法也对。接口通常是不必要的。将代码耦合到那些你认为不会改变的地方也是可以的。在你确信不会改变的地方就没有必要使用接口了。架构师说"不会改变的地方是不存在的"。不过话说回来，有时候的确不会改。<br>

在大型应用中接口是很有帮助的。和提升的代码灵活性、可测试性比起来，多敲键盘费的功夫就微不足道了。当你迅速的切换了代码实现的时候，你的经理一定会被你的神速吓一跳的。你也可以写出更适应变化的代码。<br>

总而言之，记住本书提倡"简单"架构。如果你在写小程序的时候无法遵守接口原则，别觉得不好意思。要记住我们写代码是要快乐的写。如果你不喜欢写接口，那就先简单的写代码吧。日后再精进即可。<br>
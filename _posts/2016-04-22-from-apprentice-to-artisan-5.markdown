---
layout: post
title:  "From Apprentice To Artisan --面向接口编程"
date:   2016-04-21 13:40:15
description: "From Apprentice To Artisan --Interface As Contract"
permalink: post/interface-as-contract
disqus:
  id: interface-as-contract
categories:
- blog
- php
- laravel
---

在之前的章节里，涵盖了依赖注入的基础知识：什么是依赖注入；如何实现依赖注入；依赖注入有什么好处。之前章节里面的例子也模拟了将interface注入到classes里面的过程。在我们继续学习之前，有必要深入讲解一下接口，而这正是很多PHP开发者所不熟悉的。<br>

在我成为PHP程序员之前，我是写.NET的。你觉得我是M么？在.NET里可到处都是接口。事实上很多接口是定义在.NET框架核心中了，一个好的理由是：很多.NET语言比如C#和VB.NET都是强类型的。也就是说，你在给一个函数传值，要么传原生类型对象，要么就必须给这个对象一个明确的类型定义。比如考虑以下C#方法：<br>

```
public int BillUser(User user)
{
    this.biller.bill(user.GetId(), this.amount)
}
```

注意在这里，我们不仅要定义传进去的参数是什么类型的，还要定义这个方法返回值是什么类型的。C#鼓励类型安全。除了指定的User对象，它不允许我们传递其他类型的对象到BillUser方法中。<br>

然而PHP是一种鸭子类型的语言。所谓鸭子类型的语言，一个对象可用的方法取决于使用方式，而非这个方法从哪儿继承或实现。来看个例子：<br>

```
public function billUser($user)
{
    $this->biller->bill($user->getId(), $this->amount);
}
```

在PHP里面，我们不必告诉一个方法需要什么类型的参数。实际上我们传递任何类型的对象都可以，只要这个对象能响应getId的调用。这里有个关于鸭子类型的解释：如果一个东西看起来像个鸭子，叫声也像鸭子叫，那他就是个鸭子。换言之在程序里，一个对象看上去是个User，方法响应也像个User，那他就是个User。<br>

不过PHP到底有没有任何强类型功能呢？当然有！PHP混合了强类型和弱类型的结构。为了说明这点，咱们来重写一下billUser方法：<br>

```
public function billUser(User $user)
{
    $this->biller->bill($user->getId(), $amount);
}
```

给方法加上了加上了User类型提示后，我们可以确信的说所有传入billUser方法的参数，都是User类或是继承自User类的一个实例。<br>

强类型和弱类型各有优劣。在强类型语言中，编译器通常能提供编译时错误检查的功能，这功能可是非常有用的。方法的输入和输出也更加明确。<br>

与此同时，强类型的特性也使得程序僵化。比如Eloquent ORM中，类似whereEmailOrName的动态方法就不可能在C#这样的强类型语言里实现。我们不讨论强类型弱类型哪种更好，而是要记住他们分别的优劣之处。在PHP里面使用强类型标记不是错误，使用弱类型特性也不是错误。但是不加思索，不管实际情况去使用一种模式，这么固执的使用就是错的。<br>

##约定的范例##

接口就是约定。接口不包含任何代码实现，只是定义了一个对象应该实现的一系列方法。如果一个对象实现了一个接口，那么我们就能确信这个接口所定义的一系列方法都能在这个对象上使用。因为有约定保证了特定方法的实现标准，通过多态也能使类型安全的语言变得更灵活。<br>

<blockquote>
	<p>
		多什么肽？<br>

		多态含义很广，其本质上是说一个实体拥有多种形式。在本书中，我们讲多态是一个接口有着多种实现。比如UserRepositoryInterface可以有MySQL和Redis两种实现，每一种实现都是UserRepositoryInterface的一个实例。<br>
	</p>
</blockquote>

为了说明在强类型语言中接口的灵活性，咱们来写一个酒店客房预订的代码。考虑以下接口：<br>

```
interface ProviderInterface
{
    public function getLowestPrice($location);
    public function book($location);
}
```

当用户订房间时，我们需要将此事记录在系统里。所以在User类里面写点方法：<br>

```
class User
{
    public function bookLocation(ProviderInterface $provider, $location)
    {
        $amountCharged = $provider->book($location);
        $this->logBookedLocation($location, $amountCharged);
    }
}
```

因为我们写出了ProviderInterface的类型提示，该User类的就可以放心大胆的认为book方法是可以调用的。这使得bookLocation方法有了重用性。当用户想要换一家酒店提供商时也就更灵活。最后咱们来写点代码来强化他的灵活性。<br>

```
$location = 'Hilton, Dallas';

$cheapestProvider = $this->findCheapest($location, array(
    new PricelineProvider,
    new OrbitzProvider,
));

$user->bookLocation($cheapestProvider, $location);
```

太棒了！不管哪家是最便宜的，我们都能够将他传入User对象来预订房间了。由于User对象只需要要有一个符合ProviderInterface约定的实例就可以预订房间，所以未来有更多的酒店供应商我们的代码也可以很好的工作。<br>

<blockquote>
	<p>
		忘掉细节<br>

		记住，接口实际上不真正做任何事情。它只是简单的定义了类们必须实现的一系列方法。<br>
	</p>
</blockquote>
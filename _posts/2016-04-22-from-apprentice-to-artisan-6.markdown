---
layout: post
title:  "From Apprentice To Artisan --面向接口编程"
date:   2016-04-21 13:50:15
description: "From Apprentice To Artisan --Interface As Contract"
permalink: post/interface-as-contract-2
disqus:
  id: interface-as-contract-2
categories:
- blog
- php
- laravel
---

当你的团队在开发大型应用时，不同的部分有着不同的开发速度。比如一个开发人员在制作数据层，另一个开发人员在做前端和网站控制器层。前端开发者想测试他的控制器，不过后端开发较慢没法同步测试。那如果两个开发者能以接口的方式达成协议，后台开发的各种类都遵循这种协议，就像这样：<br>

```
interface OrderRepositoryInterface 
{
    public function getMostRecent(User $user);
}
```

一旦建立了约定，就算约定还没实现，前端开发者也可以测试他的控制器了！这样应用中的不同组件就可以按不同的速度开发，并且单元测试也可以做。而且这种处理方法还可以使组件内部的改动不会影响到其他不相关组件。要记着无知是福。我们写的那些类们不用知道别的类如何实现的，只要知道它们能实现什么。这下咱们有了定义好的约定，再来写控制器：<br>

```
class OrderController 
{
    public function __construct(OrderRepositoryInterface $orders)
    {
        $this->orders = $orders;
    }
    public function getRecent()
    {
        $recent = $this->orders->getMostRecent(Auth::user());
        return View::make('orders.recent', compact('recent'));
    }
}
```

前端开发者甚至可以为这接口写个"假"实现，然后这个应用的视图就可以用假数据填充了：<br>

```
class DummyOrderRepository implements OrderRepositoryInterface 
{
    public function getMostRecent(User $user)
    {
        return array('Order 1', 'Order 2', 'Order 3');
    }
}
```

一旦假实现写好了，就可以被绑定到IoC容器里，然后整个程序都可以调用他了：<br>

```
App::bind('OrderRepositoryInterface', 'DummyOrderRepository');
```

接下来一旦后台开发者写完了真正的实现代码，比如叫RedisOrderRepository。那么IoC容器就可以轻易的切换到真正的实现上。整个应用就会使用从Redis读出来的数据。<br>

<blockquote>
	<p>
		接口就是大纲<br>

		接口在开发程序的“骨架”时非常有用。在设计组件时，使用接口进行设计和讨论都是对你的团队有益处的。比如定义一个BillingNotifierInterface然后讨论他有什么方法。在写任何实现代码前先用接口讨论好一套好的API！<br>
	</p>
</blockquote>
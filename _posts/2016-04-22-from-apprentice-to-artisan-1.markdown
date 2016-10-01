---
layout: post
title:  "From Apprentice To Artisan --依赖注入"
date:   2016-04-21 13:00:15
description: "From Apprentice To Artisan --Dependency Injection"
permalink: post/dependency-injection-the-problem
disqus:
  id: dependency-injection-the-problem
categories:
- laravel
---

Laravel框架的基础是一个功能强大的IoC container。为了真正理解本框架，需要好好掌握该容器。然而我们需要了解，控制反转容器只是一种用于方便实现"依赖注入"的工具。但要实现依赖注入并不一定需要控制反转容器只是用容器会更方便和容易。<br>

##遇到的问题

首先来看看我们为何要使用依赖注入，它能带来什么好处。考虑下列代码：<br>

```
class UserController extends BaseController
{
    public function getIndex()
    {
    	$users= User::all();
    	return View::make('users.index', compact('users'));
    }
}
```

这段代码很简短，但我们要想测试这段代码的话就一定会和实际的数据库发生联系。也就是说，Eloquent和该控制器有着紧耦合。如果不使用Eloquent，不连接到实际数据库，我们就没办法运行或者测试这段代码。这段代码同时也违背了"关注分离"这个软件设计原则。简单讲：这个控制器知道的太多了。控制器不需要去了解数据是从哪儿来的，只要知道如何访问就行。控制器也不需要知道这数据是从MySQL或哪儿来的，只需要知道这数据目前是可用的。<br>

<blockquote>
<p>
    关注分离<br>

    每一个类都应该有单独的职责，并且该职责应完全被这个类封装。<br>
</p>
</blockquote>

关注分离的好处就是能让Web控制器和数据访问解耦。这会使得实现存储迁移更容易，测试也会更容易。"Web"就仅仅是为你真正的应用做数据的传输了。<br>

想象一下你有一个类似于监视器的程序，有着很多线缆接口。你可以通过不同的接口访问不同的监视器。把Internet想象成另一个插进你程序线缆接口。大部分显示器的功能是与线缆接口互相独立的。线缆接口只是一种传输机制就像HTTP是你程序的一种传输机制一样。所以我们不想把传输机制和业务逻辑混在一起。这样的好处是很多其他的传输机制比如API调用、移动应用等都可以访问我们的业务逻辑。<br>

那么我们就别再将控制器和Eloquent耦合在一起了。咱们注入一个资料库类。<br>

##建立约定

首先我们定义一个接口，然后实现该接口。<br>

```
interface UserRepositoryInterface
{
    public function all();
}

class DbUserRepository implements UserRepositoryInterface
{
    public function all()
    {
    	return User::all()->toArray();
    }
}
```

然后我们将该接口的实现注入我们的控制器。<br>

```
class UserController extends BaseController
{
    public function __construct(UserRepositoryInterface $users)
    {
        $this->users = $users;
    }
   
    public function getIndex()
    {
        $users=$this->users->all();
        return View::make('users.index', compact('users'));
    }
}
```	

现在我们的控制器就完全和数据层面无关了。在这里无知是福！我们的数据可能来自MySQL，MongoDB或者Redis。我们的控制器不知道也不需要知道他们的区别。仅仅做出了这么小小的改变，我们就可以独立于数据层来测试Web层了，将来切换存储实现也会很容易。<br>

<blockquote>
<p>
    严守边界<br>

    记得要保持清晰的责任边界。控制器和路由是作为HTTP和你的应用程序之间的中间件来用的。当编写大型应用程序时，不要将你的领域逻辑混杂在其中。<br>
</p>
</blockquote>

为了巩固学到的知识，咱们来写一个测试案例。首先，我们要模拟一个资料库然后绑定到应用的IoC容器里。然后，我们要保证控制器正确的调用了这个资料库：<br>

```
public function testIndexActionBindsUsersFromRepository()
{    
    // Arrange...
    $repository = Mockery::mock('UserRepositoryInterface');
    $repository->shouldReceive('all')->once()->andReturn(['foo']);
    App::instance('UserRepositoryInterface', $repository);
    // Act...
    $response  = $this->action('GET', 'UserController@getIndex');
     
    // Assert...
    $this->assertResponseOk();
    $this->assertViewHas('users', array('foo'));
}
```

<blockquote>
<p>
    你在模仿我么？<br>

    在上面的例子里，我们使用了名为Mockery的模仿库。这个库提供了一套整洁且富有表达力的方法，用来模仿你写的类。Mockery可以通过Composer安装。<br>
</p>
</blockquote>

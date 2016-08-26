---
layout: post
title:  "From Apprentice To Artisan --设计模式"
date:   2016-04-21 15:50:15
description: "From Apprentice To Artisan -- Design pattern"
permalink: post/design-pattern-liskov-substitution-principle
disqus:
  id: design-pattern-liskov-substitution-principle
categories:
- blog
- php
- laravel
---

别担心，里氏替换原则读起来吓人学起来简单。该原则要求：一个抽象的任意一个实现，可以被用在任何需要该抽象的地方。读起来绕口，用普通人的话来解释一下。该原则规定：如果一个类使用了一个接口的一个实现类，那么该接口的任何其他实现类也可以被这里直接使用，不用做出任何修改。<br>

<blockquote>
	<p>
		里氏替换原则<br>

		该原则规定对象应该可以被该对象子类的实例所替换，并且不会影响到程序的正确性。<br>
	</p>
</blockquote>

为了说明该原则，我们继续编写上一章节的OrderProcessor。看下面的方法：<br>

```php

	public function process(Order $order)
	{
	    // Validate order...
	    $this->orders->logOrder($order);
	}

```

注意当我们的Order通过了验证，就被OrderRepositoryInterface的实现对象存储起来了。假设当我们的业务刚起步时，我们将订单存储在CSV格式的文件系统中。我们的OrderRepositoryInterface的实现类是CsvOrderRepository。现在，随着我们订单增多，我们想用一个关系数据库来存储订单。那么我们来看看新的订单资料库类该怎么编写吧：<br>

```php

	class DatabaseOrderRepository implements OrderRepositoryInterface 
	{
	    protected $connection;
	    public function connect($username, $password)
	    {
	        $this->connection = new DatabaseConnection($username, $password);
	    }

	    public function logOrder(Order $order)
	    {
	        $this->connection->run('insert into orders values (?, ?)', array(
	            $order->id, $order->amount
	        ));
	    }
	}

```

现在我们来研究如何使用这个实现类：<br>

```php

	public function process(Order $order)
	{
	    // Validate order...

	    if($this->repository instanceof DatabaseOrderRepository)
	    {
	        $this->repository->connect('root', 'password');
	    }
	    $this->repository->logOrder($order);
	}

```

注意在这段代码中，我们必须在资料库外部检查OrderRepositoryInterface的实例对象是不是用数据库实现的。如果是的话，则必须先连接数据库。在很小的应用中这可能不算什么问题，但如果OrderRepositoryInterface被几十个类调用呢？我们可能就要把这段“启动”代码在每一个调用的地方复制一遍又一遍。这让人非常头疼难以维护，非常容易出错误。一旦我们忘了将所有调用的地方进行同步修改，那程序恐怕就会出问题。<br>

很明显，上面的例子没有遵循里氏替换原则。如果不附加启动代码来调用connect方法，则这段代码就没法用。好了，我们已经找到问题所在，咱们修好他。下面就是新的DatabaseOrderRepository：<br>

```php

	class DatabaseOrderRepository implements OrderRepositoryInterface 
	{
	    protected $connector;
	    public function __construct(DatabaseConnector $connector)
	    {
	        $this->connector = $connector;
	    }
	    public function connect()
	    {
	        return $this->connector->bootConnection();
	    }
	    public function logOrder(Order $order)
	    {
	        $connection = $this->connect();
	        $connection->run('insert into orders values (?, ?)', array(
	            $order->id, $order->amount
	        ));
	    }
	}

```

现在DatabaseOrderRepository掌管了数据库连接，我们可以把启动代码从OrderProcessor移除了：<br>

```php

	public function process(Order $order)
	{
	    // Validate order...

	    $this->repository->logOrder($order);
	}

```

这样一改，我们就可以想用CsvOrderRepository也行，想用DatabaseOrderRepository也行，不用改OrderProcessor一行代码。我们的代码终于实现了里氏替换原则！要注意，我们讨论过的许多架构概念都和知识相关。具体讲，知识就是一个类和它所具有的周边领域，比如用来帮助类完成任务的外围代码和依赖。当你要制作一个容错性强大的应用架构时，限制类的知识是一种常用且重要的手段。<br>

还要注意如果不遵守里氏替换原则，那后果可能会影响到我们之前已经讨论过的其他原则。不遵守里氏替换原则，那么开放封闭原则一定也会被打破。因为，如果调用者必须检查实例属于哪个子类的，那一旦有个新的子类，调用者就得做出改变。<br>

<blockquote>
	<p>
		小心遗漏<br>

		你可能注意到这个原则和上一章节提到的抽象的漏洞密切相关。我们的数据库资料库的抽象漏洞就是没有遵守里氏替换原则的第一迹象。要留意那些漏洞！<br>
	</p>
</blockquote>
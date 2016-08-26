---
layout: post
title:  "From Apprentice To Artisan --设计模式"
date:   2016-04-21 16:10:15
description: "From Apprentice To Artisan -- Design pattern"
permalink: post/design-pattern-dependency-inversion-principle 
disqus:
  id: design-pattern-dependency-inversion-principle 
categories:
- blog
- php
- laravel
---

在整个坚实原则概述的旅途中，我们到达最后一站了！最后的原则是依赖反转原则，它规定高等级的代码不应该依赖低等级的代码。首先，高等级的代码应该依赖着抽象层，抽象层就像是中间人一样，负责连接着高等级和低等级的代码。其次，抽象定义不应该依赖着具体实现，但具体实现应该依赖着抽象定义。如果这些东西让你极端困惑，别担心。接下来我们会将这两方面统统介绍给你。<br>

<blockquote>
	<p>
		依赖反转原则<br>

		该原则要求高等级代码不应该依赖低等级代码，抽象定义不应该依赖具体实现。<br>
	</p>
</blockquote>

如果你已经读过了本书前面几个章节，你就已经很好掌握了依赖反转原则！为了说明本原则，让我们考虑下面这个类：<br>

```
class Authenticator 
{
    public function __construct(DatabaseConnection $db)
    {
        $this->db = $db;
    }

    public function findUser($id)
    {
        return $this->db->exec('select * from users where id = ?', array($id));
    }

    public function authenticate($credentials)
    {
        // Authenticate the user...
    }
}
```

你可能猜到了，Authenticator就是用来查找和验证用户的。继续研究它的构造函数。我们发现它使用了类型提示，要求传入一个DatabaseConnection对象，所以该验证类和数据库被紧密的联系在一起。而且基本上讲，这个数据库还只能是关系数据库。从而可知，我们的高级代码（Authenticator）直接的依赖着低级代码（DatabaseConnection）。<br>

首先我们来谈谈高级代码和低级代码。低级代码用于实现基本的操作，比如从磁盘读文件，操作数据库等。高级代码用于封装复杂的逻辑，它们依靠低级代码来达到功能目的，但不能直接和低级代码耦合在一起。取而代之的是高级代码应该依赖着低级代码的顶层抽象，比如接口。不仅如此，低级代码也应当依赖着抽象。所以我们来写个Authenticator可以用的接口：<br>

```
interface UserProviderInterface 
{
    public function find($id);

    public function findByUsername($username);
}
```

接下来我们将该接口注入到Authenticator里面：<br>

```
class Authenticator 
{
    public function __construct(UserProviderInterface $users, HasherInterface $hash)
    {
        $this->hash = $hash;
        $this->users = $users;
    }

    public function findUser($id)
    {
        return $this->users->find($id);
    }

    public function authenticate($credentials)
    {
        $user = $this->users->findByUsername($credentials['username']);
        return $this->hash->make($credentials['password']) == $user->password;
    }
}
```

做了这些小改动后，Authenticator现在依赖于两个高级抽象：UserProviderInterface和HasherInterface。我们可以向Authenticator自由的注入这俩接口的任何实现类。比如，如果我们的用户存储在Redis里面，我们只需写一个RedisUserProvider来实现UserProviderInterface接口即可。Authenticator不再依赖着具体的低级别的存储操作了。<br>

此外，由于我们的低级别代码实现了UserProviderInterface接口，则我们说该低级代码依赖着这个接口。<br>

```
class RedisUserProvider implements UserProviderInterface 
{
    public function __construct(RedisConnection $redis)
    {
        $this->redis = $redis;
    }

    public function find($id)
    {
        $this->redis->get('users:'.$id);
    }
    
    public function findByUsername($username)
    {
        $id = $this->redis->get('user:id:'.$username);
        return $this->find($id);
    }
}
```

<blockquote>
	<p>
		反转的思维<br>

		贯彻这一原则会反转好多开发者设计应用的方式。不再将高级代码直接和低级代码以“自上而下”的方式耦合在一起，这个原则提出无论高级还是低级代码都要依赖于一个高层次的抽象。<br>
	</p>
</blockquote>

在我们没有反转Authenticator的依赖之前，它除了使用数据库存储系统别无选择。如果我们改变了存储系统，Authenticator也需要被修改，这就违背了开放封闭原则。我们又一次看到，这些设计原则通常一荣俱荣一损俱损。<br>

通过强制让Authenticator依赖着一个存储抽象层，我们就可以使用任何实现了UserProviderInterface接口的存储系统，且不用对Authenticator本身做任何修改。传统的依赖关系链已经被反转了，代码变得更灵活，更加无惧变化！<br>
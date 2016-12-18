---
layout: post
title:  "php依赖注入原理的代码实现"
date:   2016-04-21 10:42:15
description: "php依赖注入原理的代码实现"
permalink: post/php-ioc-achieve
disqus:
  id: php-ioc-achieve
categories:
- blog
- php
---

依赖注入就是根据类的反射来完成。

```
namespace Database;

use ReflectionMethod;

class Database
{

    protected $adapter;

    public function __construct ()
    {

    }

    public function test (MysqlAdapter $adapter)
    {
         $adapter->test();
    }
}

class MysqlAdapter
{

    public function test ()
    {
    	echo 'i am MysqlAdapter test';
    }
}

class App
{
    public static function run ($instance, $method)
    {
    	if (! method_exists($instance, $method)) {
        	return null;
      }

      $reflector = new ReflectionMethod($instance, $method);

      $parameters = [1];

      foreach ($reflector->getParameters() as $key => $parameter)
      {
          $class = $parameter->getClass();

          if ($class) {
              array_splice($parameters, $key, 0, [new $class->name()]);
          }
      }

      call_user_func_array([$instance,$method], $parameters);
    }
}

App::run(new Database(), 'test');
```

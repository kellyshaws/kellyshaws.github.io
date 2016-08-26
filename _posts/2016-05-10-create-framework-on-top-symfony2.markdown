---
layout: post
title:  "使用Symfony2的组件创建PHP框架"
date:   2016-05-10 17:42:15
description: "使用Symfony2的组件创建PHP框架"
permalink: post/create-framework-on-top-of-the-symfony2-components
disqus:
  id: create-framework-on-top-of-the-symfony2-components
categories:
- blog
- php
- symfony
---

当创建一个框架的时候，只是为遵循MVC模式并不是一个正确的目的。最主要的目的是要做到“分离关注点”（其实就是我经常说的框架的目的是为了分工，MVC就是一种分工的方式）。事实上这是我认为需要真正关心的，唯一的设计模式。Symfony组件都是围绕实现HTTP协议这个基本原则而创建的。同样，我们将要创建的框架应更准确的被描述为HTTP框架或者说是“请求/响应框架”。<br>

许多现代框架都称呼自己为“MVC框架”。在这里我们不讨论MVC，因为Symfony组件可以用来创建任何类型的框架，而不仅仅是MVC架构的框架。总之，如果你看过MVC的定义，在此只讨论如何创建控制器部分。<br>

与其按自己想象瞎写一个框架，我们不如不断的改进我们的“应用程序”，每次改进都引入一个“概念”。让我们从最简单的一个程序开始：<br>

```
//index.php 

$input = isset($_GET['name']) ? $_GET['name'] : 'World';

header('Content-Type: text/html; charset=utf-8');

printf('Hello %s', htmlspecialchars($input, ENT_QUOTES, 'UTF-8'));
```

正如你所见到的那样，为了不出现PHP警告以及加强安全性，代码已经不那样简单了。除了安全性，这段代码也很难测试。<br>

写web程序都是有关HTTP交互的，所以我们写框架的基本原则都是围绕着HTTP协议展开的。HTTP协议描述了客户端和服务器端的交互方式。客户端和服务器端的对话是通过良好定义的“消息”——即请求和响应的传递来实现的。客户端向服务器发起一个请求，服务器根据这个请求再向客户端返回一个响应。在PHP的世界里，请求由全局变量（$_GET,$_POST,$_FILE等）来表示，响应又由一些函数来生成。<br>

执行下面的命令<br>

``` 
composer require symfony/http-foundation
composer require symfony/routing
composer require symfony/http-kernel
composer require symfony/event-dispatcher
```

``` 
//index.php
require 'vendor/autoload.php';

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing;
use Symfony\Component\HttpKernel;

$request = Request::createFromGlobals();
$routes = new Routing\RouteCollection();
//$routes->add('hello', new Routing\Route('/hello/{name}', 
//	['name' => 'World', '_controller' => 'render_template',])
//);
//$routes->add('say', new Routing\Route('/say/{name}', [
//    'name' => 'hi',
//    '_controller' => function ($request) {
	        // $foo将在模板里可见
    //		$request->attributes->set('foo', 'bar');
    //		$response = render_template($request);
    		// 改变一些头信息
    //		$response->headers->set('Content-Type', 'text/plain');
    //		return $response;
//    }
//]));

//$routes->add('leap_year', new Routing\Route('/is_leap_year/{year}', [
//    'year' => null,
//    '_controller' => array(new LeapYearController(), 'indexAction'),
//]));
//$routes->add('bye', new Routing\Route('/bye'));
$routes->add('leap_year', new Routing\Route('/is_leap_year/{year}', [
	'year' => null,
	'_controller' => 'LeapYearController::indexAction',
]));
	 
$context = new Routing\RequestContext();
$context->fromRequest($request);
$matcher = new Routing\Matcher\UrlMatcher($routes, $context);

function render_template($request)
{
		extract($request->attributes->all(), EXTR_SKIP);
		ob_start();
		include sprintf(__DIR__.'/../src/pages/%s.php', $_route);

		return new Response(ob_get_clean());
}

class LeapYearController
{
	public function indexAction($request)
	{
	 	if (is_leap_year($request->attributes->get('year'))) {
	            return new Response('Yep, this is a leap year!');
	        }
	 
	        return new Response('Nope, this is not a leap year.');
	}
}
	 
try {
	// extract($matcher->match($request->getPathInfo()), EXTR_SKIP);
	// ob_start();
	// include sprintf(__DIR__.'/../src/pages/%s.php', $_route);
	// $response = new Response(ob_get_clean());
	//$request->attributes->add($matcher->match($request->getPathInfo()));
		//$response = call_user_func($request->attributes->get('_controller'), $request);

		$resolver = new HttpKernel\Controller\ControllerResolver();
	$controller = $resolver->getController($request);
	$arguments = $resolver->getArguments($request, $controller);
	$response = call_user_func_array($controller, $arguments);
} catch (Routing\Exception\ResourceNotFoundException $e) {
	$response = new Response('Not Found', 404);
} catch (Exception $e) {
	$response = new Response('An error occurred', 500);
}

$response->send();
```

只暴露一个脚本给终端用户这种方式，是一种叫“前端控制器(front controller)”的设计模式。 <br>

```
namespace Simplex;
	 
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Matcher\UrlMatcher;
use Symfony\Component\Routing\Exception\ResourceNotFoundException;
use Symfony\Component\HttpKernel\Controller\ControllerResolver;
use Symfony\Component\Routing\Matcher\UrlMatcherInterface;
use Symfony\Component\HttpKernel\Controller\ControllerResolverInterface;
	 
class Framework
{
	protected $matcher;
	protected $resolver;
	 
	public function __construct(UrlMatcherInterface $matcher, 
		ControllerResolverInterface $resolver)
	{
		$this->matcher = $matcher;
	        $this->resolver = $resolver;
	}
	 
	public function handle(Request $request)
	{
	        try {
	            $request->attributes->add($this->matcher->match($request->getPathInfo()));
	 
	            $controller = $this->resolver->getController($request);
	            $arguments = $this->resolver->getArguments($request, $controller);
	 
	            return call_user_func_array($controller, $arguments);
	        } catch (ResourceNotFoundException $e) {
	            return new Response('Not Found', 404);
	        } catch (\Exception $e) {
	            return new Response('An error occurred', 500);
	        }
	 }
}
```

```
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing;

$request = Request::createFromGlobals();
$routes->add('leap_year', new Routing\Route('/is_leap_year/{year}', [
	'year' => null,
	'_controller' => 'Calendar\\Controller\\LeapYearController::indexAction',
]));
	 
$context = new Routing\RequestContext();
$context->fromRequest($request);
$matcher = new Routing\Matcher\UrlMatcher($routes, $context);
$resolver = new HttpKernel\Controller\ControllerResolver();
	 
$framework = new Simplex\Framework($matcher, $resolver);
$response = $framework->handle($request);
	 
$response->send();
```


``` 
namespace Calendar\Controller;
	 
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Calendar\Model\LeapYear;
	 
class LeapYearController
{
	public function indexAction(Request $request, $year)
	{
		$leapyear = new LeapYear();
	        if ($leapyear->isLeapYear($year)) {
	            return new Response('Yep, this is a leap year!');
	        }
	 
	        return new Response('Nope, this is not a leap year.');
	}
}
```

```
namespace Calendar\Model;

class LeapYear
{
    public function isLeapYear($year = null)
    {
        if (null === $year) {
            $year = date('Y');
        }
 
        return 0 == $year % 400 || (0 == $year % 4 && 0 != $year % 100);
    }
}
```

我们的框架依然缺少作为好框架必备的一个特点：扩展性。拥有扩展性意味着，开发者可以很方便的通过拦截的方式，修改请求被处理的过程。我们说的拦截是什么东西呢？验证或者缓存就是两个例子（译者注：其实请求的处理就像通过一层层的筛子。比如访问一个需要做登陆验证的url，那么我们可以设计这么一个筛子，它将判断请求来源是否已登陆，如果没有登录请求将被拦截住，转向登录页面。如果一个url设计为可以被缓存，一个请求过来以后，缓存筛子判断这个url是否已经被缓存，如果是，这个筛子直接拦截此次请求不继续往下处理，而是直接把缓存的内容发送出去）。为了灵活性，拦截程序必须是即插即用型的。<br>

因为在PHP里面没有相关的标准，我们将使用著名的设计模式“观察者模式”，来将各种拦截模块连接到我们的框架中。<br>

Symfony的事件调度(EventDispatcher)组件为此模式做了一个轻量级的实现。作为此组件的核心的调度器，将对每一个连接过它的监听器（listener）做出事件提醒。<br>

```
namespace Simplex;
 
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Matcher\UrlMatcherInterface;
use Symfony\Component\Routing\Exception\ResourceNotFoundException;
use Symfony\Component\HttpKernel\Controller\ControllerResolverInterface;
use Symfony\Component\EventDispatcher\EventDispatcher;
 
class Framework
{
	protected $matcher;
	protected $resolver;
	protected $dispatcher;
	 
	public function __construct(
	        EventDispatcher $dispatcher, 
	        UrlMatcherInterface $matcher, 
	        ControllerResolverInterface $resolver
	) 
	{
	        $this->matcher = $matcher;
	        $this->resolver = $resolver;
	        $this->dispatcher = $dispatcher;
	}
	 
	public function handle(Request $request)
	{
	        try {
	            $request->attributes->add($this->matcher->match($request->getPathInfo()));
	 
	            $controller = $this->resolver->getController($request);
	            $arguments = $this->resolver->getArguments($request, $controller);
	 
	            $response = call_user_func_array($controller, $arguments);
	        } catch (ResourceNotFoundException $e) {
	            $response = new Response('Not Found', 404);
	        } catch (\Exception $e) {
	            $response = new Response('An error occurred', 500);
	        }
	 
	        // dispatch a response event
	        $this->dispatcher->dispatch('response', new ResponseEvent($response, $request));
	 
	        return $response;
	}
}
```

``` 
namespace Simplex;
	 
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\EventDispatcher\Event;
	 
class ResponseEvent extends Event
{
	private $request;
	private $response;
	 
	public function __construct(Response $response, Request $request)
	{
		$this->response = $response;
	        $this->request = $request;
	}
	 
	public function getResponse()
	{
	        return $this->response;
	}
	 
	public function getRequest()
	{
	        return $this->request;
	}
}
```

``` 
use Symfony\Component\EventDispatcher\EventDispatcher;
	 
$dispatcher = new EventDispatcher();
$dispatcher->addListener('response', function (Simplex\ResponseEvent $event) {
	$response = $event->getResponse();
	 
	if ($response->isRedirection()
		|| ($response->headers->has('Content-Type') 
		&& false === strpos($response->headers->get('Content-Type'), 'html'))
		|| 'html' !== $event->getRequest()->getRequestFormat()
	) 
	{
		return;
	}
	 
	$response->setContent($response->getContent().'GA CODE');
});
	 
$framework = new Simplex\Framework($dispatcher, $matcher, $resolver);
$response = $framework->handle($request);
	 
$response->send();
```

```
$dispatcher->addListener('response', function (Simplex\ResponseEvent $event) {
	$response = $event->getResponse();
	$headers = $response->headers;
	 
	if (!$headers->has('Content-Length') && !$headers->has('Transfer-Encoding')) {
		$headers->set('Content-Length', strlen($response->getContent()));
	}
});
```

让我们将代码重构一下，把GA监听控件放在属于他自己的类里：<br>

```	 
namespace Simplex;
	 
class GoogleListener
{
	public function onResponse(ResponseEvent $event)
	{
		$response = $event->getResponse();
	 
	        if ($response->isRedirection()
	            || ($response->headers->has('Content-Type') 
	            && false === strpos($response->headers->get('Content-Type'), 'html'))
	            || 'html' !== $event->getRequest()->getRequestFormat()
	        ) {
	            return;
	        }
	 
	        $response->setContent($response->getContent().'GA CODE');
	}
}
```

```
namespace Simplex;
	 
class ContentLengthListener
{
	public function onResponse(ResponseEvent $event)
	{
		$response = $event->getResponse();
	        $headers = $response->headers;
	 
	        if (!$headers->has('Content-Length') 
	            && !$headers->has('Transfer-Encoding')
	        ) {
	            $headers->set('Content-Length', strlen($response->getContent()));
	        }
	}
}
```

```
$dispatcher = new EventDispatcher();
$dispatcher->addListener('response', array(
	new Simplex\ContentLengthListener(), 
	'onResponse'
), -255);
$dispatcher->addListener('response', array(new Simplex\GoogleListener(), 'onResponse'));
```

```
$dispatcher = new EventDispatcher();
$dispatcher->addSubscriber(new Simplex\ContentLengthListener());
$dispatcher->addSubscriber(new Simplex\GoogleListener());
```

```
namespace Simplex;
		 
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
	 
class GoogleListener implements EventSubscriberInterface
{
	// ...
	 
	public static function getSubscribedEvents()
	{
		return array('response' => 'onResponse');
	}
}
```

```
namespace Simplex;
	 
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
	 
class ContentLengthListener implements EventSubscriberInterface
{
	// ...
	 
	public static function getSubscribedEvents()
	{
		return array('response' => array('onResponse', -255));
	}
}
```
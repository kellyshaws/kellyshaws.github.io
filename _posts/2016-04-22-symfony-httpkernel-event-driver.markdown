---
layout: post
title:  "Symfony2 HttpKernel事件驱动解析"
date:   2016-04-21 11:42:15
description: "Symfony2 HttpKernel事件驱动解析"
permalink: post/symfony2-httpkernel-event-driver
disqus:
  id: symfony2-httpkernel-event-driver
categories:
- blog
- php
- symfony
---

Symfony是一个基于MVC模式的面向对象的PHP5框架。Symfony允许在一个web应用中分离事务控制，服务逻辑和表示层。

Symfony2框架层和应用层的工作都是在 HttpKernel::handle()中完成，HttpKernel::handle() 的内部的实现其实是通过调度事件（HttpKernel内的事件监听器）来完成的，相当于把所有组件都整合成完整的应用。<br>
使用HttpKernel很简单，只需要创建一个EventDispatcher和ControllerResolver，可以实现更多的事件监听器丰富应用的功能：

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/1.png)

##kernel.request Event##
实现kernel.request事件目的是为了添加更多信息到Request对象，或者得到返回的Response对象。<br>
kernel.request事件是HttpKernel::handle()调度的第一个事件，那么监听该事件的多个监听器就会被执行。

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/2.png)

其它的监听器实现一些初始化或者添加更多的信息到Request对象。<br>
例如路由监听器，路由监听器从Request对象中获取路由相关的信息，经过进一步加工，确定处理请求的Controller，并且把相应的信息保存到Request对象里的'attribute'里，这些信息都会被Controller解析器使用的，所以可以实现监听器之间的解耦。<br>
总得来说，实现kernel.request事件的目的要么就是直接创建和返回Response对象，要么就是添加更多的信息到Request对象。

<blockquote>
    <p>
    RouterListener是Symfony框架中实现kernel.request事件的最重要监听器，RouterListener 在路由层中执行，返回一个包含符合当前请求的路由信息的数组，例如路由匹配模式里面的_controller和请求的参数name。这些信息都会 存放在Request里的attributes数组里面，目前只是会添加路由信息到Request对象中还没有做其它的动作，但是解析 Controller的时候会被用到。
    </p>
</blockquote>

##Resolve the Controller##

假设实现kernel.request事 件的时候没有创建和返回Response对象，那么下一步就是确定、解析controller和controller需要的参数。controller部 分是应用层的最后一个堡垒，负责创建和返回包含一个特定页面的Response对象。如何确定被请求的controller完全取决于应用程序，这个工作有controller解析器来完成——一个是实现ControllerResolverInterface的类，同时也是HttpKernel构造函数的一个参数。

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/3.png)

实现ControllerResolverInterface的两个方法getController和getArguments：

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/4.png)

HttpKernel::handle()首先调用ControllerResolver的getController()方法，并向该方法传入Request对象，controller resolver根据Request包含的信息确定并返回controller。<br>
第二个方法，getArguments()会在kernel.controller事件被调度的时候执行。

<blockquote>
    <p>
    解析Controller<br>
	Symfony框架使用内置的ControllerResolver，该解析器利用了RouterListener保存到Request对象的attributes属性里信息来确定controller。
	getController<br>
	ControllerResolver在Request对象的attributes数组中查找_controller键：<br>
	a) 如果_controller键对应AcmeDemoBundle:Default:index 这个格式的值，那么该值就包含了类名和方法名，可以被Symfony框架解析成为，例如：Acme\DemoBundle\Controller\DefaultController::indexAction，这个转换是由Symfony框架的特定的ControllerResolver的子类完成的。<br>
	b) 你的controller类会被实例化，而且该controller类必须包含一个无参的构造函数。<br>
	c) 如果你的controller还实现了ContainerAwareInterface，那么setContainer方法就会被调用，container就会被注入到controller中，这个实现也是由Symfony框架的特定的ControllerResolver的子类完成的。<br>
	上面也有一些其他变化过程，例如你把你的controller配置成为service。
	</p>
</blockquote>

##The kernel.controller Event##

kernel.controller事件是在controller被执行前初始化一些信息或者改变controller对象。<br>
被调用的controller确定之后，HttpKernel::handle()就会调度kernel.controller事件。在系统的某部分被确定后（例如：controller、路由信息等）但是这些部分被执行前，监听kernel.controller事件的监听器就会运行了。

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/5.png)

<blockquote>
    <p>
    kernel.controller<br>
	Symfony框架有几个kernel.controller事件的监听器，多数都是处理分析数据的。<br>
	SensioFrameworkExtraBundle中一个监听器ParamConverter允许我们把一个对象作为参数传入到controller中，而不是字符串或者数值参数。
	</p>
</blockquote>

##获得controller的参数##

getAttributes()方法是返回一个参数数组，这个参数数组会被传递给controller，我们也可以自定义该方法，也可以使用Symfony框架内置的。

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/6.png)

<blockquote>
    <p>
    到了这一步，kernel已经获得了controller和controller运行时需要的参数了。<br>
 	Symfony框架获得controller的参数<br>
	ControllerResolver使用放射机制获得被调用的controller的方法的参数列表。遍历该列表，使用下面的步骤来确定参数列表中一一对应的值：<br>
	a) 使用参数作为键查找Request对象中的attributes数组，如果找到，那么相应的值会传入到controller的方法中，例如：controller方法的第一个参数是$name，那么在Request的attributes数组中包含$attributes['name']的值，那么$attributes['name']就会被使用。<br>                     
	b) 如果该该参数在Symfony配置routing的时候被指定，那么就会跳过对该参数的查找。<br>
	</p>
</blockquote>

##调用Controller##

这一步，controller就会被执行。<br>
controller会创建包含特定页面或者json的Response对象，这也是应用层的最后一个步骤。
 
![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/7.png)

 如果controller返回的是Response对象，那么下一步kernel.response事件就会触发，否者kernel.view事件就会被触发。

<blockquote>
    <p>
    controller必须要有返回值，如果返回null，程序会报错。
	</p>
</blockquote>

##kernel.view事件##

controller返回值不是Response对象的时候被触发。

如果controller返回的不是Response对象，kernel.view事件会被触发，kernel.view事件的目的是把controller返回的值创建一个Response对象。

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/8.png)

监听实现kernel.view事件也是很有好处的，如果controller返回了html页面或者json字符串，我们可以通过监听 kernel.view事件，把html页面或者json字符串封装到Response对象，那么Symfony框架依旧可以正常运行。<br>
但是如果没有监听器没有设置Response对象到事件中，那么程序就会报错。either the controller or one of the view listeners must always return a Response.

<blockquote>
    <p>
    Symfony框架中实现kernel.view事件<br>
	Symfony框架中没有缺省的监听器实现kernel.view事件，可是，有一个核心Bundle——SensioFrameworkExtraBundle里有个监听改事件的监听器。 如果你的controller返回一个数组，并且在controller类的顶部有@Template的注解，那么该监听器就会渲染一个模板，把 controller返回的数组传入到模板中，最后利用模板返回的内容创建一个Response对象，并返回该Response对象。<br>
	除此之 外，FOSRestBundle也实现了监听该事件的监听器，a listener on this event which aims to give you a robust view layer capable of using a single controller to return many different content-type responses (e.g. HTML, JSON, XML, etc).
	</p>
</blockquote>

##kernel.response 事件##

在发送Response对象到客户端前修改它。<br>
kernel的目的是把Request对象转换成为Response对象。Response对象可能是在kernel.request事件中创建，可能是由controller返回，又或者是由监听kernel.view事件的监听器返回。<br>
不管是在哪一个环节创建Response对象，最后kernel.response事件都会被触发。监听kernel.response事件的监听器都会 以某种方式修改Response对象，例如：修改Response的header部分，修改cookie，或者甚至会修改Response对象返回的内容。<br>
kernel.response事件完成后，HttpKernel::handle()返回最终的Response对象，调用Response::send()箱客户端发送headers头部和Response实体。

<blockquote>
    <p>
    Symfony框架实现kernel.response事件<br>
	Symfony框架内置几个监听器监 听kernel.response事件，更多的可以通过开发者社区获得。例如：在dev开发环境下WebDebugToolbarListener向页面 的底部注入javascript代码，debug工具条就会显示出来。还有另一个监听器，ContextListener序列化当前用户的信息保存到 session中，下一次请求的时候直接在session中重载用户信息。
	</p>
</blockquote>

##kernel.terminate事件##

监听该事件的监听器通常都是处理一些耗时的后台程序。<br>
HttpKernel进程的最后一个事件是kernel.terminate事件，而且该事件的触发是在HttpKernel::handle()方法之后，并且响应的内容已经发送给用户。

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/9.png)

正如上面的截图所示，在向客户端发送响应内容后才调用terminate方法，terminate方法内触发kernel.terminate事件，监听该事件的监听器就会运行，这些监听器都是在发送响应信息到客户端后执行的。

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/10.png)

##Symfony框架一个完整的工作流程##

使用HttpKernel组件的时候，我们不需要实现任何监听器添加到内核事件中，也不需要实现controller resolver。HTTp组件自带的监听器和controller resolver就能够正常工作了：

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/11.png)

##子请求##
除了把“main request”传入到HttpKernel::handle之外，还可以把所谓的“sub request”传入HttpKernel::handle中。子请求看起来和其它的请求差不多，不同的是，一般的请求是渲染完整的一个页面，而子请求渲染的是一个页面的一部分。通常我们都是在controller里面创建一个子请求（或者在模板里面创建）。

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/12.png)

HttpKernel::handle方法运行子请求的时候，需要修改第二个参数的值：

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/13.png)

子请求也是创建一个完整的请求——响应周期。唯一不同的是，有些监听器可能只会在“main request”中运行（security）。KernelEvent的子类传递给监听器，监听器通过 KernelEvent::getRequestType()判断当前请求是“main request”还是“sub request”。<br>
例如一个监听器只会在“main request”的请求下才会执行：

![Symfony 截图]({{ site.baseurl }}/uploads/2016/0422/14.png)
---
layout: post
title:  "From Apprentice To Artisan --应用架构"
date:   2016-04-21 15:00:15
description: "From Apprentice To Artisan --Application Architecture"
permalink: post/application-architecture
disqus:
  id: application-architecture
categories:
- blog
- php
- laravel
---

我们已经讨论了用Laravel4制作优美的程序架构的各个方面，让我们再深入一些细节。在本章，我们将讨论如何解耦各种处理函数：队列处理函数、事件处理函数，甚至其他"事件型"的结构如路由过滤器。<br>

<blockquote>
	<p>
		不要堵塞传输层<br>

		大部分的"处理函数"可以被当作传输层组件。也就是说，队列触发器、被触发的事件、或者外部发来的请求等都可能调用处理函数。你大可将处理函数当作控制器，避免在里面堆积太多具体业务逻辑实现。<br>
	</p>
</blockquote>

##解耦处理函数##

接下来我们看一个例子。考虑有一个队列处理函数用来给用户发送手机短信。信息发送后，处理函数还要记录消息日志来保存给用户发送的消息历史。代码应该看起来是这样：<br>

```
class SendSMS
{
    public function fire($job, $data)
    {
        $twilio = new Twilio_SMS($apiKey);
        $twilio->sendTextMessage(array(
            'to'=> $data['user']['phone_number'],
            'message'=> $data['message'],
        ));
        $user = User::find($data['user']['id']);
        $user->messages()->create(array(
            'to'=> $data['user']['phone_number'],
            'message'=> $data['message'],
        ));
        $job->delete();
    }
}
```

简单审查下这个类，你可能会发现一些问题。首先，它难以测试。在fire方法里直接使用了Twilio_SMS类，意味着我们没法注入一个模拟的服务。第二，我们直接使用了Eloquent，导致在测试时肯定会对数据库造成影响。第三，我们没法在队列外面发送短信，想在队列外面发还要重写一遍代码。也就是说我们的短信发送逻辑和Laravel的队列耦合太多了。<br>

将里面的逻辑抽出成为一个单独的"服务"类，我们即可将短信发送逻辑和Laravel的队列解耦。这样我们就可以在应用的任何位置发送短信了。我们将其解耦的过程，也令其变得更易于测试。<br>

那么我们来稍微改一改：<br>

```
class User extends Eloquent 
{
    /**
     * Send the User an SMS message
     *
     * @param SmsCourierInterface $courier
     * @param string $message
     * @return SmsMessage
     */
    public function sendSmsMessage(SmsCourierInterface $courier, $message)
    {
        $courier->sendMessage($this->phone_number, $message);
        return $this->sms()->create(array(
            'to'=> $this->phone_number,
            'message'=> $message,
        ));
    }
}
```

在本重构的例子中，我们将短信发送逻辑抽出到User模型里。同时我们将SmsCourierInterface的实现注入到该方法里，这样我们可以更容易对该方法进行测试。现在我们已经重构了短信发送逻辑，让我们再重写队列处理函数：<br>

```
class SendSMS 
{
    public function __construct(UserRepository $users, SmsCourierInterface $courier)
    {
        $this->users = $users;
        $this->courier = $courier;
    }
    public function fire($job, $data)
    {
        $user = $this->users->find($data['user']['id']);
        $user->sendSmsMessage($this->courier, $data['message']);
        $job->delete();
    }
}
```

你可以看到我们重构了代码，使得队列处理函数更轻量化了。它本质上变成了队列系统和你真正的业务逻辑之间的转换层。这可是很了不起！这意味着我们可以很轻松的脱离队列系统来发送短信息。最后，让我们为短信发送逻辑写一些测试代码：<br>

```
class SmsTest extends PHPUnit_Framework_TestCase 
{
    public function testUserCanBeSentSmsMessages()
    {
        /**
         * Arrage ...
         */
        $user = Mockery::mock('User[sms]');
        $relation = Mockery::mock('StdClass');
        $courier = Mockery::mock('SmsCourierInterface');

        $user->shouldReceive('sms')->once()->andReturn($relation);

        $relation->shouldReceive('create')->once()->with(array(
            'to' => '555-555-5555',
            'message' => 'Test',
        ));

        $courier->shouldReceive('sendMessage')->once()->with(
            '555-555-5555', 'Test'
        );

        /**
         * Act ...
         */
        $user->sms_number = '555-555-5555'; //译者注： 应当为 phone_number
        $user->sendMessage($courier, 'Test');
    }
}
```

##其他处理函数##

使用类似的方式，我们可以改进和解耦很多其他类型的"处理函数"。将这些处理函数限制在转换层的状态，你可以将你庞大的业务逻辑和框架解耦，并保持整洁的代码结构。为了巩固这种思想，我们来看看一个路由过滤器。该过滤器用来验证当前用户是否是交过钱的高级用户套餐。<br>

```
Route::filter('premium', function()
{
    return Auth::user() && Auth::user()->plan == 'premium';
});
```

猛一看这路由过滤器没什么问题啊。这么简单的过滤器能有什么错误？然而就是是这么小的过滤器，我们却将我们应用实现的细节暴露了出来。要注意我们在该过滤器里是写明了要检查plan变量。这使得将套餐方案在我们应用中的代表值暴露在了路由/传输层里面。现在我们若想调整高级套餐在数据库或用户模型的代表值，我们竟然就需要改这个路由过滤器！<br>

让我们简单改一点儿：<br>

```
Route::filter('premium', function()
{
    return Auth::user() && Auth::user()->isPremium();
});
```

小小的改变就带来巨大的效果，并且代价也很小。我们将判断用户是否使用高级套餐的逻辑放在了用户模型里，这样就从路由过滤器里去掉了对套餐判断的实现细节。我们的过滤器不再需要知道具体怎么判断用户是不是高级套餐了，它只要简单的把这个问题交给用户模型。现在如果我们想调整高级套餐在数据库里的细节，也不必再去改动路由过滤器了！<br>

<blockquote>
	<p>
		谁负责？<br>

		在这里我们又一次讨论了责任的概念。记住，始终保持一个类应该有什么样的责任，应该知道什么。避免在处理函数这种传输层直接编写太多你应用的业务逻辑。<br>
	</p>
</blockquote>
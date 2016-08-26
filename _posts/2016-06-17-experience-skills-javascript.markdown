---
layout: post
title:  "JavaScript经验技巧"
date:   2016-06-17 20:00:00
description: "experience skills for JavaScript"
permalink: post/experience-skills
disqus:
  id: experience-skills
categories:
- blog
- javascript
---

JavaScript看似简单，实际上很复杂。下面列出一些简单的问题，看似简单，但是每一点都涉及到javascript的一些知识领域。<br>

###Object prototype###

定义一个函数spacify ,将一个字符串作为参数传入，然后返回一个字符串，不过该字符串相对原有传入参数的变化是字母与字母之间多了一个空格。<br>

```javascript
spacify('hello world') // => 'h e l l o  w o r l d'  
```

正确的答案如下，大多少会选择for循环，当然这并没有错。<br>

```javascript
function spacify(str) {  
  return str.split('').join(' ');
}
```

接下来，如何将这个函数直接作用在一个字符串对象上.<br>

```javascript
'hello world'.spacify();
```
```javascript
String.prototype.spacify = function(){  
  return this.split('').join(' ');
};
```

###Arguments###

定义一个log函数。<br>

```javascript
log('hello world');
```

函数类似实现一个简单的控制台输出，在控制台输出传入的字符串。

```javascript
function log(msg){  
  console.log(msg);
}
```

如果传入多个参数依旧输出一个字符串。

```javascript
log('hello', 'world');
```

不过将console上下文传入也是非常重要的.<br>

```
function log(){  
  console.log.apply(console, arguments);
};
```

如果那个输出的字符串前统一加上(app) 这样的字符串，类似于这样:<br>

```
'(app) hello world'  
```

这个问题明显会复杂很多，应该知道arguments是一个伪数组，需要先将它转换成正常的数组，可以使用Array.prototype.slice,代码如下:<br>

```
function log(){  
  var args = Array.prototype.slice.call(arguments);
  args.unshift('(app)');

  console.log.apply(console, args);
};
```

###Context###

者对于上下文以及this的理解，给出下边的代码，如何去解释count的值。<br>

```
var User = {  
  count: 1,

  getCount: function() {
    return this.count;
  }
};

console.log(User.getCount()); //1
var func = User.getCount;  //undefined
console.log(func());
```

func的上下文是window，因此已经失去了count属性。如何确保func的上下文始终都和User关联，这样可以使输出的答案是1。<br>

正确答案是使用Function.prototype.bind，代码如下:<br>

```
var func = User.getCount.bind(User);  
console.log(func());  
```

老的浏览器并不支持该方法，应该怎样去兼容。

```
Function.prototype.bind = Function.prototype.bind || function(context) {  
    var self = this;

    return function(){
        return self.apply(context,   arguments);
  };
}
```

###apply, call and bind###

在Javascript中，Function是一种对象。Function对象中的this指向决定于函数被调用的方式。<br>

使用apply，call 与 bind 均可以改变函数对象中this的指向。<br>

```
function add(c, d){
    return this.a + this.b + c + d;
}
var o = {a:1, b:3};
add.call(o, 5, 7); // 1 + 3 + 5 + 7 = 16
add.apply(o, [10, 20]); // 1 + 3 + 10 + 20 = 34
```

注意，如果传入的第一个参数不是对象类型的，如1，那么这个参数会被自动转化为对象类型。<br>

```
function bar() {     
    console.log(
        Object.prototype.toString.call(this)
    );
}

bar.call(7); // [object Number]
```

ECMAScript 5引入了Function.prototype.bind。调用f.bind(someObject)会产生一个新的函数对象。在这个新的函数对象中，this被永久绑定到了bind的第一个参数上面，无论后期这个新的函数被如何使用。<br>

```
function f(){
    return this.a;
}

var g = f.bind({a:"azerty"});
console.log(g()); // azerty

var o = {a:37, f:f, g:g};
console.log(o.f(), o.g()); // 37, azerty
```

另外，如果第一个参数为null或者undefined的话，那么，实际上绑定到的是全局对象，即global。这一点对apple,call,bind都适用。<br>

```
function bar() {     
    console.log(
        Object.prototype.toString.call(this)
    );
}

bar.bind()(); // [object global]
bar.apply();  // [object global]
bar.call();   // [object global]
```

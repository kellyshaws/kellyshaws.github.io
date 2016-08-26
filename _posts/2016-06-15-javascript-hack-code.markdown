---
layout: post
title:  "JavaScript技巧"
date:   2016-06-15 23:00:00
description: "12 extremely useful hacks for JavaScript"
permalink: post/javascript-hack-code
disqus:
  id: javascript-hack-code
categories:
- blog
- javascript
---

在这篇文章中，我将分享12个关JavaScript非常有用的技巧。这些技巧将帮助你减少代码运行并优化代码。<br>

##Converting to boolean using !! operator##

```javascript

function Account(cash) {
    this.cash = cash;
    this.hasMoney = !!cash;
}
var account = new Account(100.50);
console.log(account.cash); // 100.50
console.log(account.hasMoney); // true

var emptyAccount = new Account(0);
console.log(emptyAccount.cash); // 0
console.log(emptyAccount.hasMoney); // false

```

当值为:0, null, "", undefined or NaN 返回false,其余返回为true. <br>

##Converting to number using + operator##

```javascript

function toNumber(strNumber) {
    return +strNumber;
}
console.log(toNumber("1234")); // 1234
console.log(toNumber("ACB")); // NaN

console.log(+new Date()) // 1461288164385

```
只能对数字字符串进行操作，不然返回NaN (Not a Number).

##Short-circuits conditionals##

```javascript

if (conected) {
    login();
}

```

上面的代码可以写成这样：<br>

```javascript

conected && login();
 
 //or
 
user && user.login();

```

##Default values using || operator##

```javascript

function User(name, age) {
    this.name = name || "Oliver Queen";
    this.age = age || 27;
}
var user1 = new User();
console.log(user1.name); // Oliver Queen
console.log(user1.age); // 27

var user2 = new User("Barry Allen", 25);
console.log(user2.name); // Barry Allen
console.log(user2.age); // 25

```

虽然ES6已经有了默认参数，但是为了兼容旧的代码，可以这样给变量赋默认值。一种骤死。<br>

##Caching the array.length in the loop##

```javascript

for(var i = 0; i < array.length; i++) {
    console.log(array[i]);
}

```

上面的代码中，每次都会去计算一次数组的长度，很浪费，所以我们常常使用下面一种方法：<br>

```javascript

var length = array.length;
for(var i = 0; i < length; i++) {
    console.log(array[i]);
}

```

更精简的可以写成这样: <br>

```javascript

for(var i = 0, length = array.length; i < length; i++) {
    console.log(array[i]);
}

```

##Detecting properties in an object##

```javascript

if ('querySelector' in document) {
    document.querySelector("#id");
} else {
    document.getElementById("id");
}

```

这个技巧很简单，很有意思。<br>

##Getting the last item in the array##

```javascript

var array = [1,2,3,4,5,6];
console.log(array.slice(-1)); // [6]
console.log(array.slice(-2)); // [5,6]
console.log(array.slice(-3)); // [4,5,6]

```

##Array truncation##

```javascript

var array = [1,2,3,4,5,6];
console.log(array.length); // 6
array.length = 3;
console.log(array.length); // 3
console.log(array); // [1,2,3]

```

##Replace all##

```javascript

var string = "john john";
console.log(string.replace(/hn/, "ana")); // "joana john"
console.log(string.replace(/hn/g, "ana")); // "joana joana"

```

##Merging arrays##

```javascript

var array1 = [1,2,3];
var array2 = [4,5,6];
console.log(array1.concat(array2)); // [1,2,3,4,5,6];

```

如果数组太大，将会占用很多内存。<br>

```javascript

var array1 = [1,2,3];
var array2 = [4,5,6];
console.log(array1.push.apply(array1, array2)); // [1,2,3,4,5,6];

```

##Converting NodeList to Arrays##

```javascript

var elements = document.querySelectorAll("p"); // NodeList
var arrayElements = [].slice.call(elements); // Now the NodeList is an array
var arrayElements = Array.from(elements); // This is another way of converting NodeList to Array

```

##Shuffling array’s elements##

```javascript

var list = [1,2,3];
console.log(list.sort(function() { Math.random() - 0.5 })); // [2,1,3]

```

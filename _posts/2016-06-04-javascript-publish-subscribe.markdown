---
layout: post
title:  "JavaScript Publish Subscribe"
date:   2016-06-04 12:42:15
description: "JavaScript"
permalink: post/javascript-publish-subscribe
disqus:
  id: javascript-publish-subscribe
categories:
- blog
- javascript
---

Publish-Subscribe属于观察着模式，在很多框架上都用来解耦。<br>

下面，我用JavaScript来实现一个简单的Demo。<br>

```
PubSub = {handlers: {}};

PubSub.on = function(eventType, handler) {
	if (!(eventType in this.handlers)) {
		this.handlers[eventType] = [];
	}
	this.handlers[eventType].push(handler);
	return this;
}

PubSub.emit = function(eventType) {
	var handlerArgs = Array.prototype.slice.call(arguments, 1);
	for (var i = 0; i < this.handlers[eventType].length; i++) {
		this.handlers[eventType][i].apply(this, handlerArgs);
	}
	return this; 
}

PubSub.on('click',function(e){
	console.log('1',e);
}).on('click',function(e){
	console.log('2',e);
}).on('click',function(e){
	console.log('3',e);
});

PubSub.emit('click','hello');
```
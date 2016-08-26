---
layout: post
title:  "Eloquent save 发生的事件"
date:   2016-05-19 11:40:15
description: "Eloquent save 发生的事件"
permalink: post/eloquent-save-event
disqus:
  id: eloquent-save-event
categories:
- blog
- php
- eloquent
---

我们来看看laravel Eloquent save的时候到底触发了哪些事件 <br>

```php 

Illuminate/Database/Eloquent/Model.php

public function save(array $options = [])
{
	$query = $this->newQueryWithoutScopes();

	if ($this->fireModelEvent('saving') === false) {
	
	     return false;
	     
	}

	if ($this->exists) {
	
	    $saved = $this->performUpdate($query, $options);
	         
	} else {
	
	    $saved = $this->performInsert($query, $options);
	         
	}

	if ($saved) {
	
	    $this->finishSave($options);
	    
	}

	return $saved;
}

 ```

首先触发的当然是saving，如果saving返回的是false，那么save就失败了，返回false <br>

接着如果$this->exists，就是这个model不是新创建的，那么就需要进行更新操作 <br>


```php 

protected function performUpdate(Builder $query, 
	array $options = [])
{
	$dirty = $this->getDirty();

	if (count($dirty) > 0) {

	     if ($this->fireModelEvent('updating') === false) {
	         
	         return false;
	            
	     }

	     if ($this->timestamps && 
	         	Arr::get($options, 'timestamps', true)) {
	         	
	         $this->updateTimestamps();
	             
	     }

	     $dirty = $this->getDirty();

	     if (count($dirty) > 0) {
	         
	          $numRows = $this->setKeysForSaveQuery($query)->update($dirty);

	          $this->fireModelEvent('updated', false);
	     }
	}

	return true;
}


 ```

- 首先如果这个模型dirty了，也就是脏了，也就是有属性改变了,那么才需要更新
- 先触发updating，更新失败了就false
- 更新时间戳
- 然后执行update，也就是写入了数据库
- 最后触发updated
- 那么如果这个对象是新建的，也就是需要执行插入操作，performInsert也做了差不多的事
- 触发creating
- 执行insert，插入数据库
- 最后created

最后回到save方法，如果更新或者插入操作成功了，那么就finishSave来结束save <br>

finishSave 触发了saved事件, 最后syncOriginal <br>

- 新创建的对象，save依次触发 saving->creating->created->saved
- 已存在的对象，save依次触发 saving->updating->updated->saved

打印过Eloquent 对象的应该都会发现，其实模型会有两个数组，一个original，一个attributes。<br>

这下应该就明白了，初始化一个模型的时候这两个数组是一样的，赋值操作只是改变attributes，所以isDirty也就是根据两个数据来判断模型是否dirty了，模型触发完saved事件后才会执行syncOriginal，syncOriginal也就是将attributes赋值给original。所以上面触发的所有事件，我们都是能拿到原来的值和变化的值得。 <br>

当模型触发updated事件的时候，我们根据isDirty(['status'])知道订单状态改变了，然后拿到订单之前是个什么样子，改变后是个什么样子，该发什么消息也就很明朗了。 <br>

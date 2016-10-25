---
layout: post
title:  "php简单的无限极分类实现"
date:   2016-10-5 10:42:15
description: "无限极分类"
permalink: post/php-unlimited-classification
disqus:
  id: php-unlimited-classification
categories:
- blog
- php
---

```
function tree($array, $id = 0, $primary = 'id', $parent = 'parent_parent')
{
	$subArray = [];
	foreach ($array as $key => $value) {
		if ($id == $value[$parent]) {
			$subArray[$key] = $value;
			$subArray[$key]['childerns'] = tree($array,$value[$primary]);
		}
	}
	return $subArray;
}
```

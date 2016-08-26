---
layout: post
title:  "一致性Hash"
date:   2016-07-02 13:00:15
description: "consistence hash algorithm"
permalink: post/consistence-hash-algorithm
disqus:
  id: consistence-hash-algorithm
categories:
- blog
- php
---

随着memcache、redis以及其它一些内存K/V数据库的流行，一致性哈希也越来越被开发者所了解。因为这些内存K/V数据库大多不提供分布式支持，所以如果要提供多台server来提供服务的话，就需要解决如何将数据分散到server，并且在增减server时如何最大化的不令数据重新分布。<br>

###Hash###

一种将任意长度的消息压缩到某一固定长度的消息摘要的函数。<br>

目前普遍采用的哈希算法是time33，又称DJBX33A (Daniel J. Bernstein, Times 33 with Addition)。这个算法被广泛运用于多个软件项目，Apache、Perl和Berkeley DB等。对于字符串而言这是目前所知道的最好的哈希算法，原因在于该算法的速度非常快，而且分类非常好(冲突小，分布均匀)。<br>

PHP内核就采用了time33算法来实现HashTable，来看下time33的定义：<br>

```
hash(i) = hash(i-1) * 33 + str[i]
```

```
<?php
function hash($str) {
    // hash(i) = hash(i-1) * 33 + str[i]
    $hash = 0;
    $s    = md5($str);
    $seed = 5;
    $len  = 32;
    for ($i = 0; $i < $len; $i++) {
        $hash = ($hash << $seed) + $hash + ord($s{$i});
    }

    return $hash & 0x7FFFFFFF;
}

echo "key1: " . (hash("key1") % 2) . "\n";
echo "key2: " . (hash("key2") % 2) . "\n";
```

对于key1和key2来说，同时存储到一台服务器上，这似乎没什么问题，但正因为key1和key2是始终存储到这台服务器上，一旦这台服务器下线了，则这台服务器上的数据全部要重新定位到另一台服务器。对于增加服务器也是类似的情况。而且重新hash(之前跟2进行hash，现在是跟3进行hash)之后，结果就变掉了，导致大多数数据需要重新定位到server。<br>

在服务器数量不变的时候，这种方式也是能很好的工作的。<br>

###一致性哈希###

由于hash算法结果一般为unsigned int型，因此对于hash函数的结果应该均匀分布在[0,2^32-1]区间，如果我们把一个圆环用2^32 个点来进行均匀切割，首先按照hash(key)函数算出服务器(节点)的哈希值， 并将其分布到0～2^32的圆环上。<br>

用同样的hash(key)函数求出需要存储数据的键的哈希值，并映射到圆环上。然后从数据映射到的位置开始顺时针查找，将数据保存到找到的第一个服务器(节点)上。<br>

![Hash 截图]({{ site.baseurl }}/uploads/2016/0702/1.jpg)

key1、key2、key3和server1、server2通过hash都能在这个圆环上找到自己的位置，并且通过顺时针的方式来将key定位到server。按上图来说，key1和key2存储到server1，而key3存储到server2。如果新增一台server，hash后在key1和key2之间，则只会影响key1(key1将会存储在新增的server上)，其它不变。<br>

上图这个圆环相当于是一个排好序的数组，我们先通过代码来看下key1、key2、key3、server1、server2的hash值，然后再作分析：<br>

```
class ConsistentHash {
    // server列表
    private $serverList = [];
    // 延迟排序，因为可能会执行多次addServer
    private $layzeSorted = false;

    public function addServer($server) {
        $hash = hash($server);
        $this->layzeSorted = false;

        if (!isset($this->serverList[$hash])) {
            $this->serverList[$hash] = $server;
        }

        return $this;
    }

    public function find($key) {
        // 排序
        if (!$this->layzeSorted) {
            asort($this->serverList);
            $this->layzeSorted = true;
        }

        $hash = hash($key);
        $len  = count($this->serverList);
        if ($len == 0) {
            return false
        }

        $keys   = array_keys($this->serverList);
        $values = array_values($this->serverList);

        // 如果不在区间内，则返回最后一个server
        if ($hash <= $keys[0] || $hash >= $keys[$len - 1]) {
            return $values[$len - 1];
        }

        foreach ($keys as $key=>$pos) {
            $nextPos = null;
            if (isset($keys[$key + 1]))
            {
                $nextPos = $keys[$key + 1];
            }
            
            if (is_null($nextPos)) {
                return $values[$key];
            }

            // 区间判断
            if ($hash >= $pos && $hash <= $nextPos) {
                return $values[$key];
            }
        }
    }
}

$consisHash = new ConsistentHash();
$consisHash->addServer("server1")->addServer("server2")->addServer("server3");
echo "key1 at " . $consisHash->find("key1") . ".\n";
echo "key2 at " . $consisHash->find("key2") . ".\n";
echo "key3 at " . $consisHash->find("key3") . ".\n";
```

即使新增或下线服务器，也不会影响全部，只要根据hash顺时针定位就可以了。<br>

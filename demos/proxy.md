# 代理模式

## Overview

* [什么是代理模式](#什么是代理模式)
* [实现](#实现)
* [代理模式的意义](#代理模式的意义)
* [总结](#总结)

## 什么是代理模式

代理模式是为一个对象提供一个代用品或占位符，以便控制对它的访问。

举一个例子, 小明在追一个 MM 想给她送一束花，但是因为小明性格比较腼腆，所以他托付了 MM 的一个好朋友来送。

再举个例子，假如我每天都得写工作日报 ( 其实没有这么惨 ). 我的日报最后会让总监审阅. 如果我们都直接把日报发给 总监 , 那可能 总监 就没法工作了. 所以通常的做法是把日报发给我的组长 ， 组长把所有组员一周的日报都汇总后再发给总监。

## 实现

接下来用一些代码来说明代理模式是怎么玩儿的。

举一个明星买鞋子的例子。

1.明星自己去买鞋。

```js
// 定义一个鞋子类
var Shoes = function(name) {
    this.name = name;
};

Shoes.prototype.getName = function() {
    return this.name;
};

// 定义一个明星对象
var star = {
    buyShoes: function(shoes) {
        console.log('买到了一双' + shoes.getName());
    }
}
star.buyShoes(new Shoes('皮鞋')); // "买到了一双皮鞋"
```

当然了，想买鞋这种事，一般都会交给助理去做。
2.明星让助理代自己去买鞋。

```js
// 定义一个鞋子类
var Shoes = function(name){
    this.name = name;
};

Shoes.prototype.getName = function() {
    return this.name;
};

// 定义一个明星对象
var star = {
    buyShoes: function(name) {
        console.log('买到了一双' + name);
    }
};

// 定义一个助理对象
var assistant = {
    buyShoes: function(shoes) {
        star.buyShoes(shoes.getName());
    }
};

assistant.buyShoes(new Shoes('高跟鞋'));  // 助理"买到了一双高跟鞋"
```

可能到这里有的同学会比较疑惑，代理的实现结果不是和不使用一样吗？是的，一样的实现结果是必须的，但是，值用代理并不是我们看到的那样将简单的事情复杂化了，代理的使用场景当然不是这种简单的场景，而是针对一些比较复杂或特殊的情况使用，这里只是为了举例说明代理的实现。下面就介绍一些使用场景。

### 小明送花的故事

假设当 A 在心情好的时候收到花，小明表白成功的几率有 60%，而当 A 在心情差的时候收到花，小明表白的成功率无限趋近于 0。
小明跟 A 刚刚认识两天，还无法辨别 A 什么时候心情好。如果不合时宜地把花送给 A，花 被直接扔掉的可能性很大，这束花可是小明吃了 7 天泡面换来的。
但是 A 的朋友 B 却很了解 A，所以小明只管把花交给 B，B 会监听 A 的心情变化，然后选 择 A 心情好的时候把花转交给 A

直接上终极代码：

```js
var Flower = function() {};
var xiaoming = {
    sendFlower(target){
        var flower = new Flower();
        target.receiveFlower(flower);
    }
};
var B = {
    receiveFlower(flower) {
        A.listenGoodMood(function() {
            var flower = new Flower();
            A.receiveFlower(flower);
        });
    }
};
var A = {
    receiveFlower(flower) {
        // 监听 A 的好心情
        console.log('收到花 ' + flower);
    },
    listenGoodMood(fn){
        setTimeout(function() { // 假设 10 秒之后 A 的心情变好
            fn();
        }, 10000);
    }
};
xiaoming.sendFlower(B);
```

虽然这只是一个虚拟的例子，但是从中可以找到两种代理模式的身影。代理B可以帮助A过滤掉一些请求，比如送花的人可以直接拒绝掉注入年龄太大或者没有宝马的。这种代理叫做*保护代理*。A和B一个充当白脸一个充当黑脸。

关于B代码的解释： 假设new Flower是一个非常昂贵的操作，所以不去让xaioming直接执行new Flower，取而代之的是让B来执行。这也是另外一种代理模式叫*虚拟代理*

虚拟代理就是把一些开销很大的对象延迟到需要它的时候去创建。
保护代理用于控制不同权限的对象对目标对象的访问，但是在js并不容易实现保护代理，因为我们无法判断谁访问了某个对象。

### 虚拟代理实现图片预加载

1、首先创建一个普通的本体对象，这个对象负责页面创建一个img标签，并且提供一个对外的setSrc接口

```js
var myImage = (function() {
    var imgNode = document.createElement( 'img' );
    document.body.appendChild(imgNode);
    return {
        setSrc(src) {
            imgNode.src = src;
        }
    };
})();
```

我们把网速调到5Kb/s，然后通过MyImage.setSrc给该img节点设置src,可以看到在图片被加载好之前，页面中有一段长长的空白时间。

2、接下来引入proxyImage，在图片被真正加载好之前页面中将出现一张占位菊花图loading.gif来提示用户图片正在加载。

```js
var proxyImage = (function() {
    var img = new Image;
    img.onload = function() {
        myImage.setSrc(this.src);
    }
    return {
        setSrc(src) {
            myImage.setSrc('file:// /C:/Users/svenzeng/Desktop/loading.gif');
            img.src = src;  
        }
    };
})();
proxyImage.setSrc('http://imgcache.qq.com/music/photo/k/000GGDys0yA0Nk.jpg');
```

现在呢 就会通过proxyImage间接的访问MyImage。proxyImage控制了客户对MyImage的访问，并且在此过程中加入了一些额外的操作，比如在真正的图片加载好之前预先设置一张本地的loadin图片。

### 缓存代理

缓存代理可可以为一些开销很大的运算结果提供暂时的存储，如果下次运算时，如果传递进来的参数跟之前一致，则可以直接返回前面存储的运算结果。

看个计算乘积的例子。

```js
let mult = function() {
    let args = Array.prototype.slice.call(arguments, 0);
    let res = 1;
    for(let i = 0, len = args.length; i < len; i++) {
        res = res * args[i];
    }
    return res;
}
```

接下来加入代理

```js
let ProxyMult = (function() {
    let cache = {};
    return function() {
        let cacheKey = Array.prototype.join.call(arguments, '_');
        if(cacheKey in cache) {
            return cache[args];
        }
        return cache[cacheKey] = mult.apply(this, arguments);
    };
})();
```

mult函数可以专注自身职责-计算乘积，缓存的功能由代理对象实现。

试想如果我们需要再做一个加法或者二其他运算，上面的代理对象可能不太实用了。这时候其实可以用高阶函数动态创建代理。

### 高阶函数动态创建代理

```js
let createProxyFactory = function(fn) {
    let cache = {};
    return function() {
        let cacheKey = Array.prototype.join.call(arguments, '_');
        if(cacheKey in cache) {
            return cache[cacheKey];
        }
        return cache[cacheKey] = fn.apply(this, arguments);
    };
}
```

这个时候需要来一个加法

```js
let plus = function() {
    let args = Array.prototype.slice.call(arguments, 0);
    let res = 0;
    for(let i = 0, len = args.length; i < len; i++) {
        res = res + args[i];
    }
    return res;
}
```

```js
// 调用
let proxyMult = createProxyFactory(mult);
let proxyPlus = createProxyFactory(plus);
proxyMult(1, 2); // 2
proxyMult(1, 2); // 2
proxyPlus(1, 2, 3); // 6
proxyPlus(1, 2, 3); // 6
```

## 代理模式的意义

单一职责原则，就一个类（通常也包括对象和函数）而言，应该仅有一个引起它变化的原因。如果一个对象承担了多项职责，就意味着这个对象将变得巨大，引起它变化的原因可能会有多个。面向对象设计鼓励将行为分布到细粒度的对象之中，如果一个对象承担的职责过多，等于吧这些职责耦合到一起，这种耦合会导致脆弱和低内聚的设计。当变化发生时，设计可能会遭到意外的破坏。

职责被定义为引起变化的原因。上面的代码中MyImage对象除了负责给img节点设置src外，还要负责预加载图片。在处理一个职责时，可能因为其强耦合影响另外一个恶职责的实现。预加载其实是一个锦上添花的功能。把这个操作放在另外一个对象里面是非常好的办法，这也是代理的作用。不久之后网速快到不需要预加载了，只需要改请求本体，不需要改动请求代理对象MyImage。这同时符合开放-封闭原则。

## 代理和本体接口一致性

代理请求的过程对于用户来说是透明的，用户并不清楚代理和本体的区别，这样的好处是：

1、用户可与放心的请求代理，只关心是否能得到想要的结果；
2、在任何使用本体的地方都可以替换成使用代理

## 总结

代理模式包括许多小分类，在js开发中最常用的是虚拟代理和缓存代理。当然了，虽然代理模式非常有用，我们不必刻意使用代理模式，当真正发现不方便直接访问某个对象时再编写代理也不迟。

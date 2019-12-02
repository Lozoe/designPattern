# 单例模式

## Overview

* [什么是单例模式](#什么是单例模式)
* [单例模式的应用场景](#单例模式的应用场景)
* [有什么优点？](#有什么优点？)
* [实现-简单实现](#实现-简单实现)
* [实现-透明单例实现](#实现-透明单例实现)
* [实现-代理实现](#实现-代理实现)
* [实现-惰性单例](#实现-惰性单例)
* [总结](#总结)

## 什么是单例模式

模式的定义：保证一个类仅有一个实例，并提供一个访问它的全局访问点。
模式的核心：确保只有一个实例，并且提供全局访问。

## 单例模式的应用场景

单例模式是一种常用的模式，有一些对象我们往往只需要一个，比如线程池、全局缓存、浏览器中的window对象。

## 有什么优缺点

优点：
    1. 在单例模式中，活动的单例只有一个实例，对单例类的所有实例化得到的都是相同的一个实例。这样就防止其它对象对自己的实例化，确保所有的对象都访问一个实例
    2. 单例模式具有一定的伸缩性，类自己来控制实例化进程，类就在改变实例化进程上有相应的伸缩性。
    3. 提供了对唯一实例的受控访问。
    4. 由于在系统内存中只存在一个对象，因此可以节约系统资源，当需要频繁创建和销毁的对象时单例模式无疑可以提高系统的性能。
    5. 允许可变数目的实例。
    6. 避免对共享资源的多重占用。
缺点：
    1. 不适用于变化的对象，如果同一类型的对象总是要在不同的用例场景发生变化，单例就会引起数据的错误，不能保存彼此的状态。
    2. 由于单利模式中没有抽象层，因此单例类的扩展有很大的困难。
    3. 单例类的职责过重，在一定程度上违背了“单一职责原则”。
    4. 滥用单例将带来一些负面问题，如为了节省资源将数据库连接池对象设计为的单例类，可能会导致共享连接池对象的程序过多而出现连接池溢出；如果实例化的对象长时间不被利用，系统会认为是垃圾而被回收，这将导致对象状态的丢失。

## 实现-简单实现

要实现一个标准的单例模式并不复杂，无非就是用一个变量来标志当前是否已经为某个类创建过对象，如果是，则在下一次获取该类的实例时，直接返回之前创建的对象。

```js
var Singleton = function( name ){
    this.name = name;
};
Singleton.instance = null;
Singleton.prototype.getName = function() {
    return this.name;
};
Singleton.getInstance = function(name) {
    if(!this.instance) {
        this.instance = new Singleton(name);
    }
    return this.instance;
};

var a = Singleton.getInstance( 'sven1' );
var b = Singleton.getInstance( 'sven2' );
alert ( a === b ); // true

// 或者：
var Singleton = function(name) {
    this.name = name;
};
Singleton.prototype.getName = function() {
    return this.name;
};
Singleton.getInstance = (function() {
    var instance = null;
    return function(name) {
        if(!instance) {
            instance = new Singleton(name);
        }
        return instance;
    }
})();
```

这里通过Singleton.getInstance来获取Singleton类的唯一对象，这种方式相对简单。不过有一个问题增加了这个类的不透明性Singleton的使用者必须知道这个是一个单例类，以往new XXX的方式来获取对象不同。

## 实现-透明单例实现

利用原型的特性实现一个观察者模式，代码如下：

```js
var CreateDiv = (function(){
    var instance;
    var CreateDiv = function( html ){
        if ( instance ){
            return instance;
        }
        this.html = html;
        this.init();
        return instance = this;
    };
    CreateDiv.prototype.init = function(){
        var div = document.createElement( 'div' );
        div.innerHTML = this.html;
        document.body.appendChild( div );
    };
    return CreateDiv;
})();
```

CreateDiv增加程序的复杂度，负责了两件事情
1、创建对象和执行初始化init
2、保证只有一个对象

## 实现-代理实现

```js
var CreateDiv = function( html ){
    this.html = html;

    this.init();
};
CreateDiv.prototype.init = function(){
    var div = document.createElement( 'div' );
    div.innerHTML = this.html;
    document.body.appendChild( div );
};

var ProxySingletonCreateDiv = (function(){
    var instance;
    return function( html ){
        if ( !instance ){
            instance = new CreateDiv( html );
        }
        return instance;
    }
})();

var a = new ProxySingletonCreateDiv( 'sven1' );
var b = new ProxySingletonCreateDiv( 'sven2' );
alert ( a === b );
```

## 实现-惰性单例

需要的时候才创建实例对象

```js
var getSingle = function( fn ){
    var result;
    return function(){
        return result || ( result = fn .apply(this, arguments ) );
    }
};
var createLoginLayer = function(){
    var div = document.createElement( 'div' );
    div.innerHTML = '我是登录浮窗';
    div.style.display = 'none';
    document.body.appendChild( div );
    return div;
};
var createSingleLoginLayer = getSingle( createLoginLayer );
document.getElementById( 'loginBtn' ).onclick = function(){
    var loginLayer = createSingleLoginLayer();
    loginLayer.style.display = 'block';
};
```

将创建实例的职责和管理单例的职责分别放置在两个方法里，这两个方法可以独立变化而互不影响。当他们连接一起的时候就完成了创建唯一实例对象的功能

## 总结

在getSingle函数中，实际上也提到闭包和高阶函数的概念。单例模式是一种非常简单但是非常实用的模式，特别是惰性单例技术，在合适的时候才创建对象，并且只创建唯一的一个。更奇妙的是，创建对象和管理单例的职责被分布在两个不同的方法中。组合起来更具单例模式的威力。

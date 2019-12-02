# 装饰者模式

## Overview

* [什么是装饰者模式](#什么是装饰者模式)
* [实现](#实现)
* [总结](#总结)

## 什么是装饰者模式

装饰者(Decorator)模式能够在不改变对象自身的基础上，在程序运行期间给对像动态的添加职责而不会影响从这个类中派生的其它对象。与继承相比，装饰者是一种更轻便灵活的做法。

## 实现

### 模拟传统面向对象语言实现

假设我们在编写一个飞机大战的游戏，随着经验值的增加，我们操作的飞机对象可以升级成更厉害的飞机，一开始这些飞机只能发射普通的子弹，升到第二级时可以发射导弹，升到第三级时可以发射原子弹。

```js
// 原始飞机类
var Plane = function(){};

Plane.prototype.fire = function() {
    console.log('发射普通子弹');
}

// 修饰类 - 导弹
var MissileDecorator = function(plane) {
    this.plane = plane;
}
MissileDecorator.prototype.fire = function() {
    this.plane.fire();
    console.log('发射导弹');
}

// 修饰类 - 原子弹
var AtomDecorator = function(plane) {
    this.plane = plane;
}
AtomDecorator.prototype.fire = function() {
    this.plane.fire();
    console.log('发射原子弹');
}

var plane = new Plane();
plane = new MissileDecorator(plane);
plane = new AtomDecorator(plane);
plane.fire();  // 发射普通子弹 发射导弹 发射原子弹
```

这种给对象添加职责的方式并没有真正的改动对象自身，而是将对象当如另一个对象中，这些对象以一条链的方式进行引用，形成一个聚合对象。这些对象拥有相同的接口，当请求到达链中的某个对象时，这个对象会执行自身的操作，随后把请求转发给链中的下一个对象。

### Javascript语言装饰者模式实现

```js
var plane = {
    fire: function() {
        console.log('发射普通子弹');
    }
}
var missileDecorator = function() {
    console.log('发射导弹');
}
var atomDecorator = function() {
    console.log('发射原子弹');
}
var fire1 = plane.fire;
plane.fire = function() {
    fire1();
    missileDecorator();
}
var fire2 = plane.fire;
plane.fire = function() {
    fire2();
    atomDecorator();
}
plane.fire();
// 分别输出： 发射普通子弹、发射导弹、发射原子弹
```

### AOP装饰函数

特点： 松耦合高复用

```js
Function.prototype.before = function(beforefn) {
    var __self = this; // 保存原函数的引用
    return function () { // 返回包含了原函数和新函数的"代理"函数
        beforefn.apply(this, arguments); // 执行新函数，且保证this 不被劫持，新函数接受的参数
        // 也会被原封不动地传入原函数，新函数在原函数之前执行
        return __self.apply(this, arguments); // 执行原函数并返回原函数的执行结果，
        // 并且保证this 不被劫持
    }
}

Function.prototype.after = function(afterfn) {
    var __self = this;
    return function() {
        var ret = __self.apply(this, arguments);
        afterfn.apply(this, arguments);
        return ret;
    }
};

// test 1
document.getElementById = document.getElementById.before(function(){
    alert (1);
});
var button = document.getElementById( 'button' );
console.log(button);   // 先弹出1 再打印button

// test 2
window.onload = function () {
    alert(1);
}
window.onload = (window.onload || function () { }).after(function () {
    alert(2);
}).after(function () {
    alert(3);
}).after(function () {
    alert(4);
});

// 避免原型污染
var before = function(fn, beforefn) {
    return function() {
        beforefn.apply(this, arguments);
        return fn.apply(this, arguments);
    }
};
var a = before(
    function() { alert (3) },
    function() { alert (2) }
);
a = before(a, function() { alert (1); } );
a();  // 分别输出 1 2 3
```

#### 常见应用1 - 数据统计上报

```js
Function.prototype.after = function (afterfn) {
    var __self = this;
    return function() {
        var ret = __self.apply(this, arguments);
        afterfn.apply(this, arguments);
        return ret;
    }
};
var showLogin = function() {
    console.log('打开登录浮层');
}
var log = function() {
    console.log('上报标签为: ' + this.getAttribute('tag'));
}

// 点击登录button后进行数据上报
showLogin = showLogin.after(log); // 打开登录浮层之后上报数据
document.getElementById('button').onclick = showLogin;
```

#### 常见应用2 - 动态改变函数参数

#### 常见应用3 - 插件式表单验证

```html
<body>
    用户名：<input id="username" type="text" />
    密码： <input id="password" type="password" />
    <input id="submitBtn" type="button" value="提交"></button>
</body>
<script>
    // 原始写法 未使用AOP
    var username = document.getElementById('username'),
        password = document.getElementById('password'),
        submitBtn = document.getElementById('submitBtn');
    var formSubmit = function () {
        if (username.value === '') {
            return alert('用户名不能为空');
        }
        if (password.value === '') {
            return alert('密码不能为空');
        }
        var param = {
            username: username.value,
            password: password.value
        }
        ajax('http:// xxx.com/login', param); // ajax 具体实现略
    }
    submitBtn.onclick = function () {
        formSubmit();
    }

    // 改进 - 第一步
    var validata = function () {
        if (username.value === '') {
            alert('用户名不能为空');
            return false;
        }
        if (password.value === '') {
            alert('密码不能为空');
            return false;
        }
    }
    var formSubmit = function () {
        if (validata() === false) { // 校验未通过
            return;
        }
        var param = {
            username: username.value,
            password: password.value
        }
        ajax('http:// xxx.com/login', param);
    }
    submitBtn.onclick = function () {
        formSubmit();
    }

    // 改进 - 第二步 使用AOP
    Function.prototype.before = function (beforefn) {
        var __self = this;
        return function () {
            if (beforefn.apply(this, arguments) === false) {
                // beforefn 返回false 的情况直接return，不再执行后面的原函数
                return;
            }
            return __self.apply(this, arguments);
        }
    }
    var validata = function () {
        if (username.value === '') {
            alert('用户名不能为空');
            return false;
        }
        if (password.value === '') {
            alert('密码不能为空');
            return false;
        }
    }
    var formSubmit = function () {
        var param = {
            username: username.value,
            password: password.value
        }
        ajax('http:// xxx.com/login', param);
    }
    formSubmit = formSubmit.before(validata);
    submitBtn.onclick = function () {
        formSubmit();
    }
</script>
```

## 总结

装饰者模式和代理模式的结构看起来非常相像，这两种模式都描述了怎样为对象提供 一定程度上的间接引用，它们的实现部分都保留了对另外一个对象的引用，并且向那个对象发送 请求。
代理模式和装饰者模式最重要的区别在于它们的意图和设计目的。代理模式的目的是，当直接访问本体不方便或者不符合需要时，为这个本体提供一个替代者。本体定义了关键功能，而代理提供或拒绝对它的访问，或者在访问本体之前做一些额外的事情。装饰者模式的作用就是为对 象动态加入行为。

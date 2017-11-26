---
layout: post
title: "聊聊设计模式之 发布－订阅模式"
keywords: ["mode", "publish", "subscription"]
description: "基于commanderjs打造自己的命令行工具"
category: "js"
tags: ["mode"]
---
{% include JB/setup %}

> 最近在进行设计模式的复习，聊聊设计模式 系列文章只是对复习的一些纪录，也方便以后需要时进行查阅。

#### 现实世界里的发布－订阅模式

发布－订阅模式也叫做观察者模式，从字面上就很好理解。生活中有很多例子都是该模式的应用。比如购物网站上有时会有抢购商品，没到指定时间时，你可以选择订阅抢购提醒，抢购时间到了，购物软件就会给你推送抢购消息，也就是发布消息了，然后你就可以去抢购。再比如我们平时都会定好闹钟，在早上的某个点把你叫醒。这些都是发布订阅模式的很好的例子。

#### DOM事件
在我们开发中，发布订阅模式也随处可见，那就是我们的DOM事件
``` javascript
document.body.addEventListener('click', function () {
    console.log('clicked')
})
setTimeout(function () {
  document.body.click()
}, 2000)
```
在上面的代码中，我们给`body`绑定了一个点击事件，也就是`body`订阅了点击事件，然后延时2秒触发`body`的点击事件，也就是发布了。


#### 自定义事件
在平时的开发中，难免会遇到需要用到发布订阅模式的时候。比如用户登陆，登陆成功后，我们需要刷新用户信息的展示，或许还有页面上其它模块的视图在登陆前跟登陆后不一样。通常的做法会在异步登陆后的回调里来处理这些逻辑，比如登陆成功后调用成功的方法，登陆失败调用登陆失败的方法。如果登陆后的逻辑不是很多，这样也勉强说的过去，一旦逻辑比较复杂，代码的耦合性就比较高了。这里其实就可以引入我们的发布－订阅模式了，登陆成功就发布一个登陆成功的消息，订阅了登陆成功消息的模块就会做对应的处理了，这样代码的耦合行也大大降低了，登陆失败亦然。

如何实现我们的发布订阅呢？在`javascript`中，我们很容易想到的就是事件模型，所以，我们可以打造一个自定义事件的模型。按照一贯的做法，我们先理清思路，然后再撸码。

我们可以定义一个对象`Event`，然后对象有一个属性来存放我们的订阅队列`list`，有一个订阅的方法`on`，一个发布的方法`trigger`，当然，我们也可以添加一个取消订阅的方法`off`。订阅的时候，我们往`list`里push一个任务，发布的时候，取出那个任务就OK了，当然，任务一般都是方法，我们也可以给任务取个名字，同一个任务也可以订阅多次。取消订阅就是找到对应的任务，删除之即可。思路大概清楚了，我们看下具体的实现。

``` javascript
var Event = {

  list: {},

  on: function (key, fn) {
    if (!this.list[key]) {
      this.list[key] = []
    }
    this.list[key].push(fn)
  },

  trigger: function () {
    var key = Array.prototype.shift.call(arguments) // 订阅的事件名
    var fns = this.list[key] // 订阅事件的回调，可能订阅了多次
    if (!fns || fns.length === 0) {
      return false
    }
    for (var i = 0; i < fns.length; i++) {
      fns[i].apply(this, arguments)
    }
  },

  off: function (key, fn) {
    var fns = this.list[key]
    if (!fns) {
      return false
    }
    if (!fn) {
      fns && (fns.length = 0) // 订阅的事件全部取消
    } else {
      for (var i = 0; i < fns.length; i++) {
        if (fns[i] === fn) {
          fns.splice(i, 1)
        }
      }
    }
  }
}
```

``` javascript
Event.on('a', function () { // 订阅a
  console.log('a')
})

Event.on('b', b1) // 订阅b，推送b1

Event.on('b', b2) //订阅b, 推送b2

Event.trigger('a') // 发布a

Event.off('b', b1) // 取消订阅b上的推送b1

Event.trigger('b') // 发布b

function b1 () {
  console.log('b1')
}

function b2 () {
  console.log('b2')
}

### output
a
b2
```
尝试运行下上面的代码，发现输出跟预想的一样，一个简单的发布－订阅模式就这样完成了，😄

上面的代码在实际的工作中也是可以使用的，为了使用方便，我们可以将它挂在到全局，在浏览器里，也要放在其它脚本之前引入

``` javascript
window.$ = Event
$.on('a', function () {})
$.off('a')
$.trigger('a')
```

突然想起，`jquery`有一个`once`方法，也就是事件只能绑定一次，发布后就不能再次发布了，我们也可以简单的实现下，第一次发布后就取消订阅就可以了

``` javascript
var Event = {
  ...
  once: function (key, fn) {
    this.on(key, fn)
    this.off(key, fn)
  }
}
```

#### 总结
发布－订阅模式确实能为我们的开发提供很大的便利性，解耦我们的代码，但是当工程很大，模块很多时，到处充斥着发布和订阅，反而不利于我们对代码的维护，每次订阅，都会增加`list`占用的内存，如果订阅的事件一直没触发，那就是浪费内存了，所以，合理使用就好。


> 参考：[JavaScript设计模式与开发实践](https://item.jd.com/11686375.html)

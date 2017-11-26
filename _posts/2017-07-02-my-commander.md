---
layout: post
title: "打造一款实用命令行工具－上篇"
keywords: ["commanderjs", "react", "simr"]
description: "基于commanderjs打造自己的命令行工具"
category: "nodejs"
tags: ["commanderjs", "react", "simr"]
---
{% include JB/setup %}

> 背景：在使用react进行组件化开发的过程中，不知道你有没有遇到这样的问题，当我需要添加一个组件的时候，我需要去创建组件对应的`js`脚本文件和样式脚本文件，如`sass`，当然，也可能还有其他的文件。并且创建文件后，开始也会写一些通用代码，如果你足够懒，可能会在硬盘上存上一份这样的模板文件，需要创建组件时直接拷贝过来改改，然后开始开发。哈哈，懒才能导致进步，最近就思考了这个问题，能不能自己打造一款工具，能够像开源社区很多的cli工具一样，一条简单的命令就创建好我的组件需要的脚本文件和内容，如`simr c App --sass`，这样的话不就可以大大提升我们的开发效率？

基于以上背景，开始了我的第一次命令行工具探索之路～

#### [commander.js](https://github.com/tj/commander)

tj大神出品的`nodejs`的`cli`工具，基于此工具可以开发自己的cli工具。关于`commanderjs`的内容，这里不作过多介绍，详解`commanderjs`也不是本文的目的，读者自己看看API或[网上教程](https://aotu.io/notes/2016/08/09/command-line-development/)就好，内容不多。具体需要用到的方法会在后面讲解。

#### 工具基础搭建
- 创建工具目录，如`my-commander`
```
mkdir my-commander
cd my-commander
npm init // 按默认提示配置好`package.json`就好
mkdir bin
cd bin && touch simr
```
以上工作也可以在IDE里完成，现在编辑下我们的`package.json`文件，添加如下配置，然后保存
```
"bin": {
    "simr": "./bin/simr"
},
```
添加后的`package.json`看起来大概这个样子
```
{
    "name": "my-commander",
    "bin": {
        "simr": "./bin/simr"
    },
    "version": "1.0.0",
    "description": "命令行工具",
    "main": "index.js",
    ...
}
```

切换到我们刚才创建的`bin`目录下的`simr`文件，文件的第一行输入如下内容，代表这是一个可执行文件
```
#! /usr/bin/env node
```

- 编写创建组件的命令（可以参照[simr](https://github.com/dancdx/my-commander/blob/master/bin/simr)阅读本节内容）
    > 简单说下`commanderjs`的几个API(copy from [凹凸](https://aotu.io/notes/2016/08/09/command-line-development/))
    - command – 定义命令行指令，后面可跟上一个name，用空格隔开，如 .command( ‘app [name] ‘)
    - alias – 定义一个更短的命令行指令 ，如执行命令$ app m 与之是等价的
    - description – 描述，它会在help里面展示
    - option – 定义参数。它接受四个参数，在第一个参数中，它可输入短名字 `-a`和长名字`–app` ,使用` | `或者,分隔，在命令行里使用时，这两个是等价的，区别是后者可以在程序里通过回调获取到；第二个为描述, 会在 help 信息里展示出来；第三个参数为回调函数，他接收的参数为一个string，有时候我们需要一个命令行创建多个模块，就需要一个回调来处理；第四个参数为默认值
    - action – 注册一个callback函数,这里需注意目前回调不支持let声明变量
    - parse – 解析命令行

继续编写`simr`文件
``` javascript
const program = require('commander')

program
  .command('component [componentName]')
  .alias('c')
  .description('创建一个组件')
  .option('-s, --sass', '使用sass')
  .option('-l, --less', '使用less')
  .action(function (componentName, option) {
    console.log(componentName, option)
  })

program.parse(process.argv)
```

上面我们创建了一个命令`component`，别名为`c`，该命令接受2个参数`sass`和`less`，现在就来试试该命令能不能顺利跑起来
执行`npm link`，将我们的`simr`挂到全局node，
``` 
# bash
simr component App
simr c App
simr c App --sass
simr c App --less
```
尝试上面的命令，控制台没有报错（如有报错，请核对[simr](https://github.com/dancdx/my-commander/blob/master/bin/simr)），并成功打印对应参数信息，恭喜你，第一步已经完成，`simr`命令已经生效，打造一款命令行工具是不是很简单，哈哈～

目前我们的工具只是简单的打印，现在回到我们打造这款工具的初衷，提升开发效率，将重复的劳动工程化。在该系列文章的下篇，我将为你揭开神秘的面纱～

> 仓库地址：[https://github.com/dancdx/my-commander](https://github.com/dancdx/my-commander), 欢迎`start`

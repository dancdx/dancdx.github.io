---
layout: post
title: "打造一款实用命令行工具－下篇"
keywords: ["commanderjs", "react", "simr"]
description: "基于commanderjs打造自己的命令行工具"
category: "nodejs"
tags: ["commanderjs", "react", "simr"]
---
{% include JB/setup %}

> 该系列文章的上篇，我们已经打造好了工具的模型了，能够在机器的任何目录下执行我们的 `simr`命令，并成功打印参数信息，本篇文章讲继续后面的内容，也是该系列文章的重点，通过工程化提升我们的开发效率。

`simr c App`当执行该命令的时候，我们已经拿到了我们需要的参数，组件名`App`，命令参数`sass: true`，下面将根据参数来展开。

- 在`my-commander`目录下创建如下结构和对应文件
```
├── bin
│   └── simr
├── lib
│   ├── create
│   │   ├── Base.js
│   │   ├── task
│   │   │   └── component.js
│   │   └── templates
│   │       └── default
│   │           └── component
│   │               ├── index.css
│   │               ├── index.js
│   │               ├── index.less
│   │               └── index.scss
│   └── util
│       ├── index.js
│       └── wrench.js
└── package.json
```
简单梳理下我们接下来编码的逻辑：
执行`simr c App`命令后，我们在`action`回调里拿到了我们输入的组件名称`App`，然后需要创建文件夹`App`，并在`App`目录创建`index.js`和`index.css`文件，当然，我们这里还得考虑样式文件是用`css`还是`sass`或者`less`。

回到我们刚才的创建文件，这里有多种方式，直接创建，或者开始就创建好模板文件，直接拷贝过去。其实使用拷贝模板文件是比较方便的，使用模板文件那我们就可以写模板代码了，然后将命令里对应的参数和模板进行编译，就可以拿到正确的文件内容了，下面切入正题

- 编写工具方法
参考 [util](https://github.com/dancdx/my-commander/tree/master/lib/util) 编写我们的`util/indexjs` 和 `util/wrench.js`，

`util/indexjs`里有两个方法，获取`homedir`和`simr`和用户机器上将要存放模板的`.simr`目录，该目录开始是没有的，第一次使用时创建
``` javascript
const path = require('path')
const fs = require('fs')
const os = require('os')
const util = require('util')

class Util {

  getSimrPath () {
    let simrPath = path.join(this.homedir(), '.simr')
    if (!fs.existsSync(simrPath)) {
      fs.mkdirSync(simrPath)
    }
    return simrPath
  }

  homedir () {
    if (typeof os.homedir === 'function') {
      return os.homedir()
    } else {
      const env = process.env
      const home = env.HOME
      const user = env.LOGNAME || env.USER || env.LNAME || env.USERNAME

      if (process.platform === 'win32') {
        return env.USERPROFILE || env.HOMEDRIVE + env.HOMEPATH || home || null
      }

      if (process.platform === 'darwin') {
        return home || (user ? '/Users/' + user : null);
      }

      if (process.platform === 'linux') {
        return home || (process.getuid() === 0 ? '/root' : (user ? '/home/' + user : null));
      }

      return home || null
    }
  }
}

module.exports = new Util()
```

[util/wrench.js](https://github.com/dancdx/my-commander/tree/master/lib/util/wrench.js)是目录和文件操作的一些，后面会用到（感谢开源社区作出的贡献）

- 编写create逻辑
`create`目录下就主要是我们创建文件，编译文件，拷贝文件的一些逻辑了。在`Base.js`里，我们写一个`Base`类，里面有获取资源地址，目标地址，模板地址，编译拷贝模板等实现方法，尽管目前我们只有创建组件这一条命了，以后开发其他命了，如创建页面，创建模块等，甚至一键生成整个项目，都可以继承这个`Base`类来开发的。

``` javascript
const fs = require('fs')
const path = require('path')
const pathExists = require('path-exists')
const memFs = require('mem-fs')
const FileEditor = require('mem-fs-editor')

const wrench = require('../util/wrench')
const Util = require('../util')

class Base {
  constructor () {
    this.sharedFs = memFs.create()
    this.fs = FileEditor.create(this.sharedFs)
    this.sourceRoot(path.join(Util.getSimrPath()))
  }

  mkdir () {
    fs.mkdirSync.apply(fs, arguments)
  }

  // 资源根路径
  sourceRoot (rootPath) {
    if (typeof rootPath === 'string') {
      this._sourceRoot = path.resolve(rootPath)
    }
    if (!fs.existsSync(this._sourceRoot)) {
      this.mkdir(this._sourceRoot)
    }
    return this._sourceRoot
  }

  templatePath () {
    let filepath = path.join.apply(path, arguments)
    if (!path.isAbsolute(filepath)) {
      filepath = path.join(this.sourceRoot(), '', filepath)
    }
    return filepath
  }

  destinationRoot (rootPath) {
    if (typeof rootPath === 'string') {
      this._destinationRoot = path.resolve(rootPath)
      if (!pathExists.sync(rootPath)) {
        this.mkdir(rootPath)
      }
      process.chdir(rootPath)
    }
    return this._destinationRoot || process.cwd()
  }

  destinationPath () {
    let filepath = path.join.apply(path, arguments)
    if (!path.isAbsolute(filepath)) {
      filepath = path.join(this.destinationRoot(), filepath)
    }
    return filepath
  }

  template (type, source, dest, data) {
    if (typeof dest !== 'string') {
      options = data
      data = dest
      dest = source
    }
    this.fs.copyTpl(
      this.templatePath('', type, source),
      this.destinationPath(dest),
      data || this
    )
    return this
  }

  copy (type='component', source, dest) {
    dest = dest || source
    this.template(type, source, dest)
    return this
  }

  getTemplate (name='component', cb) {
    const filepath = path.join(__dirname, `./templates/default/${name}`)
    const tmpPath = path.join(this._sourceRoot, name)
    if (!pathExists.sync(tmpPath)) {
      wrench.copyDirSyncRecursive(filepath, tmpPath)
    }
    cb()
  }
}

module.exports = Base
```

完成了`Base`类的编写，我们接下来编写我们的`Component`类，也就是我们创建组件的具体任务类

首先，我们的`Component`类是继承于`Base`类的，先看代码
``` javascript
const fs = require('fs')
const path = require('path')
const inquirer = require('inquirer')
const chalk = require('chalk')

const Base = require('../Base')
const Util = require('../../util')

class Component extends Base {

  constructor (options) {
    super(...arguments)
    this.conf = Object.assign({
      description: '',
      componentName: null,
      sass: false,
      less: false,
      type: 'css'
    }, options)
    this.init()
  }

  init () {
    console.log(chalk.magenta(`开始创建组件～`))
  }

  talk (cb) {
    const prompts = []
    const conf = this.conf
    if (typeof conf.componentName !== 'string') {
      prompts.push({
        type: 'input',
        name: 'componentName',
        message: '输入组件名称',
        validate: (input) => {
          if (!input) return '组件名称不能为空，请再次输入'
          if (fs.existsSync(this.destinationPath(input))) {
            return '组件已存在，换个名字'
          }
          return true
        }
      })
    } else if (fs.existsSync(this.destinationPath(conf.componentName))) {
      prompts.push({
        type: 'input',
        name: 'componentName',
        message: '组件已存在，换个名字',
        validate: (input) => {
          if (!input) return '组件名称不能为空，请再次输入'
          if (fs.existsSync(this.destinationPath(input))) {
            return '组件已存在，换个名字'
          }
          return true
        }
      })
    }
    if (conf.sass === undefined && conf.less === undefined) {
      prompts.push({
        type: 'list',
        name: 'type',
        message: '选择css预处理器',
        choices: [{
          name: '不需要',
          value: 'css'
        }, {
          name: 'Sass/Compass',
          value: 'scss'
        }, {
          name: 'Less',
          value: 'less'
        }]
      })
    }

    inquirer.prompt(prompts).then((answers) => {
      if (conf.sass) answers.type = 'scss'
      if (conf.less) answers.type = 'less'
      Object.assign(this.conf, answers)
      this.write(cb)
    })
  }

  write (cb) {
    const conf = this.conf
    this.mkdir(conf.componentName)
    this.copy('component/index.js', '', this.destinationPath(conf.componentName) + '/index.js')
    if (conf.type === 'scss') {
      this.copy('component/index.scss', '', this.destinationPath(conf.componentName) + '/index.scss')
    } else if (conf.type === 'less') {
      this.copy('component/index.less', '', this.destinationPath(conf.componentName) + '/index.less')
    } else {
      this.copy('component/index.css', '', this.destinationPath(conf.componentName) + '/index.css')
    }
    this.fs.commit(() => {
      console.log(chalk.green('创建组件' + conf.componentName + '成功'))
    })
  }

  create (cb) {
    this.getTemplate('component', () => {
      this.talk(cb)
    })
  }
}

module.exports = Component
```
在`Component`类中，主要有3个方法，`create`方法提供给`simr`命了执行后的`action`回调调用的，在`create`方法里，我们调用了`Base`的获取模板方法，该方法会去读机器上的模板文件，如果读不到，就创建模板到机器上的模板目录，机器上有了模板后，后面就调用`talk`方法，开始命令行交互。

在开始的时候我们提到了，我们创建组件时的样式文件是可以选择的，我们支持`simr c App`，同样也需要支持`simr c App --sass`，在这里，我们使用[inquirer](https://www.npmjs.com/package/inquirer)这个包来进行交互，具体使用可以查看`talk`方法或者[官方文档](https://www.npmjs.com/package/inquirer)，当输入是`simr c App`时，我们会出现下拉选择，让用户选择使用哪种预处理器，如果输入的命了已经带了预处理器，如`simr c App --less`，直接使用就好，输入参数都确认好了，我们就可以调用`write`方法将选择好的模板写到当前目录了

在`write`方法里，先`mkdir`创建组件目录，然后编译拷贝`index.js`，然后根据选择的预处理器拷贝对应文件。可以注意下`this.copy`和`this.fs`方法，这2个方法都是在`Base`里定义的，使用的包是[mem-fs-editor](https://www.npmjs.com/package/mem-fs-editor)，这个包提供了`copyTpl`方法，使用[ejs](https://www.npmjs.com/package/ejs)作为模板语法进行编译，并将编译好的文件拷贝到目标目录，该包提供的文件操作方法开始都是缓存起来的，所以最后需要执行`this.fs.commit`把之前的文件操作全部提交，这样才能在你的机器上生成你所期望的文件了。

抱歉，刚才花了点时间去介绍一个`mem-fs-editor`包...回到我们的`simr`文件，在我们命令的`action`回调里可以调用我们`Component`类的`create`方法了
``` javascript
program
  .command('component [componentName]')
  .alias('c')
  .description('创建一个组件')
  .option('-s, --sass', '使用sass')
  .option('-l, --less', '使用less')
  .action(function (componentName, option) {
    const component = new Component({
      componentName: componentName,
      description: option.description,
      sass: option.sass,
      less: option.less
    })
    component.create(() => {
      console.log(chalk.green('组件创建成功～'))
    })
  })
```

至此，我们创建组件的工具就完成了，是不是很激动，赶紧体验一把吧～

> 仓库地址：[https://github.com/dancdx/my-commander](https://github.com/dancdx/my-commander), 欢迎`start`

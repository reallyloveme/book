# cli开发

## 1. 基本知识

### 1.1 什么是npm

npm由网站，命令行界面(cli)，注册表组成。

npm 是Node.js默认的，用JavaScript编写的软件包管理系统。npm会随着Node.js自动安装。

通过“registry”的查询服务，用户可以通过本地的npm命令下载安装指定的模块，用户也可以将自己设计的模块

分发到registry上面。

registry上面的模块通常采用CommonJS格式，而且都包含一个JSON格式的元文件。

npm的模块以“先到先得”的原则注册，各模块作者不会发生混乱。

缺点：

一旦有人撤回自己发布的模块，会使依赖那个模块的项目出现问题，还会带来安全风险。

registry没有审核机制，会存在一些低质量，不安全甚至有害的模块。

```shell
npm config ls -l // 查找npm包配置
```

### 1.2 npm安装使用

本地项目中安装 npm install <包名> （--save-dev或--save）

devependencies 和 dependencies 只有在特定环境才会生效，例如在下载npm插件的时候，不需要引用插件的依赖包

![](https://cdn.nlark.com/yuque/0/2020/png/735885/1606184534155-8d41bfb4-e31d-409c-bb5b-acd05eeb51b3.png)

```
// save-dev会被记录在package.json的devependencies下，当进行代码打包时，不会将这里的插件源码打包入我们的项目代码中

// save 它就会被记录在package.json的dependencies。当进行代码打包时，会将插件打包入我们的项目代码中

// webpack项目打包跟devependencies和dependencies无太大关系，webpack打包是依赖于node_modules文件夹的，
// 只要这个文件夹里有相应的模块，就可以打包
```

全局安装 npm install <包名> -g 或者npm install <包名> -g

```javascript
// 全局安装默认在/usr/local
// 如果是win32 则安装在/usr/local/node_modules,否则/usr/local/lib/node_modules
// cli/lib/install.js

// cli/lib/utils/flat-options.js
this.globalPrefix = '/usr/local'
// cli/lib/npm.js
get globalDir () {
    return process.platform !== 'win32'
      ? resolve(this.globalPrefix, 'lib', 'node_modules')
      : resolve(this.globalPrefix, 'node_modules')
  }
```

插件安装后可以通过require(ES5)或者import

### 1.3 *#!/usr/bin/env node* 含义

\#!usr/bin/env node主要是帮助脚本找到 node 的脚本解释器

  \#！含义在Linux或者Unix中叫做：[shebang](https://links.jianshu.com/go?to=%5Bhttps%3A%2F%2Fen.wikipedia.org%2Fwiki%2FShebang_(Unix)%5D(https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FShebang_(Unix))%2C)，unix类操作系统中一个普通文件带有`#!`开头的，就会当做一个执行文件来运行，#就是注释的含义

usr/bin/env node 是指通过node来执行文件，usr/bin/env 指在当前路径的env环境变量中去查找。

由于Windows不支持shebang行，因此`npm`创建了一个*包装* `*.cmd`（批处理）文件，该文件使用脚本文件shebang行中指定的任何可执行文件显式调用“二进制”文件（在`bin`键中`package.json`指定的脚本文件）

### 1.4 npm link

npm link 的作用相当于是将我们的工程进行了全局安装,通过npm prefix -g可以查看安装路径

卸载模块

```shell
npm uninstall -g <name>
```



## 2. 什么是CLI

cli是用户通过键盘输入指令，计算机接收到指令后，予以执行。相对于图形用户界面，操作速度更快，但是需要记更多的操作命令

## 3. 如何开发CLI

### 3.1 命令行参数处理 commander

node 中有自带的process对象提供有关当前node.js 进程的信息和控制的全局对象,我们一般用第三方模块进行处理 commander

源码地址：https://github.com/tj/commander.js

commander特征：

自动记录代码，自动生成帮助，合并短参数，默认选项，强制选项，命令解析，提示符

#### version 方法：定义命令程序版本 

```javascript
const program = require('commander')

// 定义命令程序版本
program
   .version('0.0.1')
// 自定义flag,--version不能省略
program
   .version('0.0.1', '-m, --version')
 program.parse(process.argv)
```

#### option方法：定义命令选项

自定义的flag<必须>，一长一短的flag，可以用逗号，竖线或空格隔开，flag后面可以跟参数

<>代表必填，[]定义可选参数

选项描述（可省略），通过-h或--help可查看

选项默认值（可省略）

```javascript
program
    // .option('-a --add', 'add something')
    .option('--more-word', 'moreWord') // 多单词形式
    .option('-a --add <name>[file]', 'not word', 'ss') // 选项默认值
```

#### command方法：自定义命令

写法：

自定义命令名称，名称必填，命令参数通过<>或[]定义

命令描述（可省略）

配置选项

```javascript
program
    .command('my <name> [otherDirs...]')
    .description('message')
    .action((name, otherDirs, cmd) => {
        console.log('name', name)
        console.log('otherDirs', otherDirs)
        // console.log('cmd', cmd)
    })  
```

#### description 方法： 命令的描述性语句

#### action() 方法：定义命令的回调函数

#### parse方法：解析process.argv，设置options以及触发commands

参数： 

- - - process.argv 属性会返回一个数组，其中包含当 Node.js 进程被启动时传入的命令行参数

```javascript
// 打印 process.argv。
process.argv.forEach((val, index) => {
  console.log(`${index}: ${val}`);
});
```

### 3.2 交互式命令 inquirer

| 参数        | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| type        | 提示问题的类型：input，confirm,list,rawlist,expand,checkbox,editor |
| name        | 当前问题回答的变量                                           |
| message     | 问题的描述                                                   |
| default     | 默认值                                                       |
| choices     | 列表选项，包含一个分隔符（separator）                        |
| validate    | 对用户的回答进行有效验证                                     |
| filter      | 对用户回答进行过滤处理，返回处理的值                         |
| transformer | 对用户回答的显示效果处理                                     |
| when        | 根据前面问题回答，判断当前问题是否需要回答                   |
| prefix      | 修改message默认前缀                                          |
| suffix      | 修改message默认后缀                                          |

## 3.3 文件下载 download-git-repo

参数：

第一个参数是下载路径，第二个参数下载文件存放位置，第三个是回调函数，可以对错误进行处理



## 3.4 cli 发布

1. 1. 1. 官网注册账号 https://www.npmjs.com/
      2. 通过命令行登陆npm（npm login）

1. 1. 1. 当前待发布包下输入npm publish
      2. 发布成功

## 3.5 更新版本

1. 1. 1. 修改package.json 中version字段
      2.  npm version patch-->1.0.1：属于小修改，不更改功能使用

npm version minor-->1.1.0:可能添加了一些功能，不影响以前的使用。

npm version major-->2.0.0:可能改了API，输入大范围的修改

## 3.6 包的使用

通过npm安装

## 3.7 发布后取消或删除npm 包

```shell
npm unpublish --force // 强制取消
npx force-unpublish package-name '原因描述' // 删除
```

## 4 实战

https://github.com/reallyloveme/vue-cli-auto

https://github.com/reallyloveme/sj-vite

### 4.1 hellow world

```shell
// 创建文件夹
mkdir <文件名>
// npm 初始化
npm init -y

// 创建index.js文件
#!/usr/bin/env node

process.argv.forEach((val, index) => {
  if (val === 'hello') {
    console.log('hello world!')
 	}
})

// package.json里面创建
"bin":{
  "cli": "./index.js"
}
// 执行npm link
// 输入cli hello，就可以看到 hello world！
```

### 4.2 下载初始化模版项目

- - - 创建cli项目文件
    - npm初始化 npm init -y

- - - 创建文件目录和文件 bin/index.js
    - 修改package.json配置

- - - 执行npm link
    - 输入 cli命令

```javascript
// 安装插件
npm i commander --save-dev
npm i download-git-repo --save-dev
npm i ora --save-dev
npm i clear --save-dev

// bin/index.js
#!/usr/bin/env node

const program = require('commander') // commander库
const ora = require('ora')
const { promisify } = require('util')
const clear = require('clear') // 清除
program.version(require('./package.json').version)


program
.command('init <name>')
.description('init project ')
.action((name) => {
  console.log(name)
  const spinner = ora(`下载…… ${name}`)
  clear()
  const download = promisify(require('download-git-repo'))
  spinner.start()
  download('github:reallyloveme/vue-template', name)
  spinner.succeed()
})
program.parse(process.argv)

// package.json
"bin": {
   "cli": "./index.js"
 }
```
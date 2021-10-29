

# webpack

1. webpack是什么？
2. webpack能做什么？
3. 如何使用webpack？

## webpack是什么？

webpack本质就是一个模块打包工具，通过**万物皆模块**的设计思想，实现了整个前端项目的模块化。

具有两个特性：

1. Loader机制
2. 插件机制

## webpack能做什么？

1. 编译功能--能够将开发阶段编写的包含新特性的代码转换为能够兼容大多数环境的代码，解决环境兼容问题
2. 打包能力--能够将散落的模块打包在一起，解决浏览器频繁请求模块文件的问题
3. 打包不能文件类型的模块--能够将样式css，图片，字体等资源文件作为模块使用，通过统一模块化方案，所有资源文件加载都通过代码控制

## 如何使用webpack？

### 不使用webpack

![image-20211023145704473](https://gitee.com/myreally/pic/raw/master/image-20211023145704473.png)

```javascript
// modules.js
export  default () => {
  const element = document.createElement('h1')
  element.textContent = 'webpack demon'
  element.addEventListener('click', () => {
    alert('点击')
  })
  return element
}

// index.js
import createElement from './modules.js'
const elementHtml = createElement()

document.body.append(elementHtml)

// index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>webpack使用</title>
</head>
<body>
  <script type="module" src="./src/index.js"></script>
</body>
</html>
```

运行结果：

![image-20211023150052999](https://gitee.com/myreally/pic/raw/master/image-20211023150052999.png)

### 使用webpack

```shell
// 安装webpack
npm init -y
npm i webpack webpack-cli --save-dev

// 通过webpack打包
npx webapck
```

![image-20211023151044940](https://gitee.com/myreally/pic/raw/master/image-20211023151044940.png)

### 自定义webpack打包配置

```javascript
/**
 * @type {import('webpack').Configuration}
 */
const config = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js'
  }
}

module.exports = config
```

打包后文件

![image-20211023155259415](https://gitee.com/myreally/pic/raw/master/image-20211023155259415.png)

### ES6转换为ES5

es6转es5需要借助于babel

```shell
// 关联 webpack 和 babel
npm i babel-loader @babel/core -D
// 编译js
npm i @babel/preset-env -D 

```

编译为ES5前

![image-20211023160045758](https://gitee.com/myreally/pic/raw/master/image-20211023160045758.png)

编译后：将const 转成 var ，箭头函数转换为function，此处Promise并未转换

![image-20211023160834828](https://gitee.com/myreally/pic/raw/master/image-20211023160834828.png)

编译转换Promise

```shell
// 安装@babel/polyfil
npm install --save @babel/polyfill
// .babelrc
{
  "presets": [
    [
      "@babel/preset-env",
     {
      corejs: 2, //新版本需要指定核⼼库版本
      useBuiltIns: "usage"//按需注⼊
     }
    ]
    
  ]
}
```

### 编译其他类型文件

增加css文件

```
// 修改webpack.config配置
entry: './src/index.css'
```

打包报错![image-20211023165307731](https://gitee.com/myreally/pic/raw/master/image-20211023165307731.png)

**Tip: webpack 内部默认loader只能处理js模块，其他模块需要配置不同的loader**

```shell
// 安装css loader
npm i css-loader --save-dev
```

将css-loader中所加载的样式，通过创建style标签方式添加到页面

```shell
// 安装style loader
npm i style-loader --save-dev
```

## 编写loader插件

```javascript
// loader-md.js
module.exports = source => {
  // 加载的模块内容
  console.log('source', source)
  return 'loader md' // 返回文本字符串
}
```

![image-20211023172307584](https://gitee.com/myreally/pic/raw/master/image-20211023172307584.png)

报错提示需要其他loader加载器

```javascript
// 返回值修改为一段js代码字符串
module.exports = source => {
  // 加载的模块内容
  console.log('source', source)
  // return 'loader md'
  return 'console.log("md 转换")'
}
```

![image-20211023172913520](https://gitee.com/myreally/pic/raw/master/image-20211023172913520.png)

**Tip: loader 返回值必须是一段js代码字符串**

# 插件机制

比较常用的插件：clean-webpack-plugin（清除dist），html-webpack-plugin（生成html模版文件），copy-webpack-plugin（赋值文件）

### 开发插件

在 Webpack 整个工作过程会有很多环节，为了便于插件的扩展，Webpack 几乎在每一个环节都埋下了一个钩子。这样我们在开发插件的时候，通过往这些不同节点上挂载不同的任务，就可以轻松扩展 Webpack 的能力。

Webpack hooks https://webpack.js.org/api/compiler-hooks/

# webpack运行机制总结：

1. webpack cli 启动打包流程
2. 载入webpack核心模块，创建Compiler对象
3. 使用compiler对象开始编译整个项目
4. 从入口文件开始，解析模块以来，形成依赖树
5. 递归依赖树，将每个模块给对应的Loader处理
6. 合并Loader处理完的结果，将打包结果输出到dist目录

![image-20211029090205826](https://gitee.com/myreally/pic/raw/master/image-20211029090205826.png)

## 源码解析

执行**webpack/bin/webpack.js** 中**runCli(cli)**方法，启动 webpack cli的打包流程

```javascript
// bin/webpack.js
runCli(cli);

const runCli = cli => {
	const path = require("path");
	const pkgPath = require.resolve(`${cli.package}/package.json`);
	// eslint-disable-next-line node/no-missing-require
	const pkg = require(pkgPath);
	// eslint-disable-next-line node/no-missing-require
  // 引入webpack-cli文件
	require(path.resolve(path.dirname(pkgPath), pkg.bin[cli.binName]));
};
```

webpack-cli中载入webpack核心模块

```javascript
// 开始调用webpack核心模块
      console.log('开始调用webpack核心模块')
      compiler = this.webpack(
        config.options,
        callback
          ? (error, stats) => {
              if (error && this.isValidationError(error)) {
                this.logger.error(error.message);
                process.exit(2);
              }

              callback(error, stats);
            }
          : callback,
      );
```

创建compiler对象

```javascript
const { compiler, watch, watchOptions } = create();

const create = () => {
      // 校验options是否符合要求
			if (!webpackOptionsSchemaCheck(options)) {
				getValidateSchema()(webpackOptionsSchema, options);
			}
			/** @type {MultiCompiler|Compiler} */
      // 创建compiler对象
			let compiler;
			let watch = false;
			/** @type {WatchOptions|WatchOptions[]} */
			let watchOptions;
			if (Array.isArray(options)) { // 判断options是否数组，说明webpack支持多路打包
				/** @type {MultiCompiler} */
				compiler = createMultiCompiler(
					options,
					/** @type {MultiCompilerOptions} */ (options)
				);
				watch = options.some(options => options.watch);
				watchOptions = options.map(options => options.watchOptions || {});
			} else { // 单线程打包
				const webpackOptions = /** @type {WebpackOptions} */ (options);
				/** @type {Compiler} */
				compiler = createCompiler(webpackOptions);
				watch = webpackOptions.watch;
				watchOptions = webpackOptions.watchOptions || {};
			}
			return { compiler, watch, watchOptions };
		};

// 开始构建
compiler.run((err, stats) => {
  compiler.close(err2 => {
    callback(err || err2, stats);
  });
});
```

编译整个项目，创建compilation对象

```javascript
const run = () => {
  	// webpack 的 hooks
			this.hooks.beforeRun.callAsync(this, err => {
				if (err) return finalCallback(err);

				this.hooks.run.callAsync(this, err => {
					if (err) return finalCallback(err);

					this.readRecords(err => {
						if (err) return finalCallback(err);
            // 开始编译整个项目
            console.log('开始编译整个项目')
						this.compile(onCompiled);
					});
				});
			});
		};
```

compilation对象创建完成后，触发make钩子

```javascript
			logger.time("make hook");
			// 触发make钩子
			this.hooks.make.callAsync(compilation, err => {
				logger.timeEnd("make hook");
				if (err) return callback(err);

				logger.time("finish make hook");
				this.hooks.finishMake.callAsync(compilation, err => {
					logger.timeEnd("finish make hook");
					if (err) return callback(err);

					process.nextTick(() => {
						logger.time("finish compilation");
						compilation.finish(err => {
							logger.timeEnd("finish compilation");
							if (err) return callback(err);

							logger.time("seal compilation");
							compilation.seal(err => {
								logger.timeEnd("seal compilation");
								if (err) return callback(err);

								logger.time("afterCompile hook");
								this.hooks.afterCompile.callAsync(compilation, err => {
									logger.timeEnd("afterCompile hook");
									if (err) return callback(err);

									return callback(null, compilation);
								});
							});
						});
					});
				});
			});
```

make 阶段：根据 entry 配置找到入口模块，开始依次递归出所有依赖，形成依赖关系树，然后将递归到的每个模块交给不同的 Loader 处理。

make 阶段具体流程：

1. EntryPlugin 中调用了 **Compilation** 对象的 **addEntry** 方法，开始解析入口；
2. addEntry 方法中又调用了 **_addEntryItem** 方法，将入口模块添加到模块依赖列表中；
3. 紧接着通过 Compilation 对象的 buildModule 方法进行模块构建；
4. buildModule 方法中执行具体的 Loader，处理特殊资源加载；
5. build 完成过后，通过 acorn 库生成模块代码的 AST 语法树；
6. 根据语法树分析这个模块是否还有依赖的模块，如果有则继续循环 build 每个依赖；
7. 所有依赖解析完成，build 阶段结束；
8. 最后合并生成需要输出的 bundle.js 写入 dist 目录。

```js
// EntryPlugin.js
// 解析入口
compiler.hooks.make.tapAsync("EntryPlugin", (compilation, callback) => {
			compilation.addEntry(context, dep, options, err => {
				callback(err);
			});
		});
// 
	addEntry(context, entry, optionsOrName, callback) {
		// TODO webpack 6 remove
		const options =
			typeof optionsOrName === "object"
				? optionsOrName
				: { name: optionsOrName };
// 调用_addEntryItem 将入口文件添加到模块依赖列表
		this._addEntryItem(context, entry, "dependencies", options, callback);
	}
```

![image-20211026212411747](https://gitee.com/myreally/pic/raw/master/image-20211026212411747.png)


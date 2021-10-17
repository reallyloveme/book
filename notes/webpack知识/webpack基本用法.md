# webpack

一：初始化项目，安装webpack

```shell
npm init -y
npm install webpack webpack-cli --save-dev
```

二：hello webapck

![img](https://cdn.nlark.com/yuque/0/2020/png/735885/1591754426812-ba5e809a-bec1-45be-ac2e-81d113a0cef4.png)

```javascript
安装lodash
npm install --save lodash
// index.js
import _ from 'lodash'

function component() {
  var element = document.createElement('div')

  // Lodash（目前通过一个 script 脚本引入）对于执行这一行是必需的
  // Lodash, now imported by this script
  element.innerHTML = _.join(['Hello', 'webpack'], ' ')

  return element
}

document.body.appendChild(component())

// package.json
{
  "name": "webpack-demon",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "css-loader": "^3.5.3",
    "style-loader": "^1.2.1",
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.11"
  },
  "dependencies": {
    "lodash": "^4.17.15"
  }
}

// webpack.config.js
const path = require('path')
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
// index.html
<!doctype html>
  <html>
    <head>
      <title>asset management</title>
    </head>
    <body>
      <script src="bundle.js"></script>
    </body>
  </html>
```

三：加载css

```javascript
......安装插件
npm install --save-dev style-loader css-loader
......新增style.css文件
// style.css
.hello {
  color: red;
}
//index.js
import _ from 'lodash'
import './style.css'

function component() {
  var element = document.createElement('div')

  // Lodash（目前通过一个 script 脚本引入）对于执行这一行是必需的
  // Lodash, now imported by this script
  element.innerHTML = _.join(['Hello', 'webpack'], ' ')
  element.classList.add('hello')
  return element
}

document.body.appendChild(component())
// webpack.config.js
const path = require('path')
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      }
    ]
  }
}
```

打包后文件

![img](https://cdn.nlark.com/yuque/0/2020/png/735885/1591755078660-9c11c283-fb0d-468d-892c-0f167cb620aa.png)

四：加载图片

```javascript
......安装插件
npm install --save-dev file-loader
// webpack.config.js
const path = require('path')
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: ['file-loader']
      }
    ]
  }
}
// index.js
import _ from 'lodash'
import './style.css'
import icon from './icon.jpg'

function component() {
  var element = document.createElement('div')

  // Lodash（目前通过一个 script 脚本引入）对于执行这一行是必需的
  // Lodash, now imported by this script
  element.innerHTML = _.join(['Hello', 'webpack'], ' ')
  element.classList.add('hello')
  // 图像嵌入div
  const myIcon = new Image()
  myIcon.src =  icon
  element.appendChild(myIcon)
  return element
}

document.body.appendChild(component())
```

五：加载数据

```shell
......安装插件
npm install --save-dev csv-loader xml-loader
```

六：管理输出

```shell
// 设置HtmlWebpackPlugin
......安装HtmlWebpackPlugin
npm install --save-dev html-webpack-plugin
// 清理dist文件
......安装插件clean-webpack-plugin
npm install clean-webpack-plugin --save-dev
```



![img](https://cdn.nlark.com/yuque/0/2020/png/735885/1608446004466-45bb7d42-bfe4-42e8-b80c-103c6fc2a69d.png)
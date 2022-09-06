# webpack基础

node.js环境

`__dirname`、`__filename`、`process.cwd()`或者 `./` 和`../`, 前三者都是绝对路径, 为了便于比较 `./` 和 `../` 我们使用 `path.resolve('./')` 来转换为绝对路径



## 基本demo

简单demo

项目结构

```bash
|-demo_test_01
	|- public
	|	|- index.html
	|- src
	|	|- main.js
	|	|- js
	|	|	|- count.js
	|	|	|- sum.js
	|- package.json
```

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script></script>
</head>
<body>
    <h1>aaaaaaaaaa</h1>
    <script src="../dist/main.js"></script>
</body>
</html>
```

main.js

```js
import { count } from "./js/count";
import { sum } from "./js/sum";

console.log(count(1,4));

console.log(sum(1,2,3,45));
```

count.js

```js
let count  = function(x,y){
    return x + y;
}
export {count}
```

sum.js

```js
let sum = function(...args) {
    return args.reduce((p,c)=>p+c,0)
}
export { sum }
```

执行命令

```bash
pnpm init 
```

生成`package.json`

```json
{
  "name": "demo_test_01",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^5.74.0",
    "webpack-cli": "^4.10.0"
  }
}
```

```bash
## 安装weback
pnpm add -D webpack webpack-cli

#"devDependencies": {
#   "webpack": "^5.74.0",
#   "webpack-cli": "^4.10.0"
#}
```

```bash
## 执行打包
## webpack 使用./src/main.js 作为打包入口
## --mode选项 production|none|development
npx webpack ./src/main.js --mode=production
```



## 基本配置

### 核心概念

#### 1.`entry`（入口）

​	指示webpack从哪个文件开始执行打包

#### 2.`output`( 输出)

​	指示weback打包完的文件输出到哪里去，如何命名等

#### 3.`loader`(加载器)

​	webpack(本身只能处理js、json等资源，其他资源需要loader，webpack才能解析)

#### 4.`plugin`(插件)

​	扩展webpack的功能

#### 5.`mode`(模式)

​	主要有两种模式

​		开发者模式`development`

​		生产者模式 `production`



### webpack配置

`webpack.config.js`

```js
const path = require("path");

module.exports = {
    //入口
    entry: "./src/main.js",
    //输出
    output:{
        //所有文件输出路径
        path:path.resolve(__dirname, "dist"),
        //文件名
        filename:'main.js',
        clean:true //在打包前清空path路劲
    },
    //加载器
    module:{
        rules:[
            //loader的配置

        ]
    },
    //插件
    plugins:[
        //plugins的配件

    ],
    // 模式
    mode:"production",

}
```

打包命令

```bash
npx webpack 
## 或者
pnpm webpack
```



#### 使用entry

```js
//入口
entry: "./src/main.js",
```



#### 使用output

```js
//输出
output:{
    //所有文件输出路径
    path:path.resolve(__dirname, "dist"),
    //文件名
    filename:'static/js/main.js', //js打包输出路径
},
```



#### 使用loader

##### 处理`css`资源

###### `public/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>webpack5</title>
  </head>
  <body>
    <h1>Hello Webpack5</h1>
    <div class="box1"></div>
    <div class="box2"></div>
    <div class="box3"></div>
    <div class="box4"></div>
    <script src="../dist/main.js"></script>
  </body>
</html>
```



###### `css-loader`

```bash
pnpm -D add css-loader style-loader
```

`index.css`

```css
.box1{
    width: 100px;
    height: 100px;
    background-color: blue;
    background-image: url("../images/1.jpg");
    background-size: cover;
}
```

添加规则`webpack.config.js`

```js
{
    test: /\.css$/, 
    use:[
        { loader: 'style-loader' }, //将js中引入的css生成style标签在html中
        { loader: 'css-loader',}, // 将引入的css编译打包成commonjs的模块js中
     ]
},
```



###### `less-loader`

```bash
pnpm add -D less-loader less
```

`index.css`

```css
.box1{
    width: 100px;
    height: 100px;
    background-color: blue;
    background-image: url("../images/1.jpg");
    background-size: cover;
}
```

添加规则

```js
{
     test: /\.less$/,
     use: ["style-loader", "css-loader", "less-loader"],
}
```



###### `sass-loader`

```bash
pnpm add -D sass-loader sass
```

`index.sass`

```sass
.box3
  width: 100px
  height: 100px
  background-color: hotpink

```

`index.scss`

```scss
.box4{
    width: 100px;
    height: 100px;
    background-color: rgb(148, 148, 248);
}
```

添加规则

```js
{
    test: /\.s[ac]ss$/,
	use: ["style-loader", "css-loader", "sass-loader"],
}
```



###### `main.js`

```js
import "./css/index.css"
import "./css/index.less"
import "./css/index.sass"
import "./css/index.scss"

import { count } from "./js/count";
import { sum } from "./js/sum";
```



###### 总配置

```js
    //加载器
    module:{
        rules:[
            //loader的配置
            {
                test: /\.css$/, 
                use:[
                        { loader: 'style-loader' }, //将js中引入的css生成style标签在html中
                        { loader: 'css-loader',}, // 将引入的css编译打包成commonjs的模块js中
                ]
            },{
        		test: /\.less$/,
        		use: ["style-loader", "css-loader", "less-loader"],
      		},{
        		test: /\.s[ac]ss$/,
		        use: ["style-loader", "css-loader", "sass-loader"],
    		},{
        		test: /\.styl$/,
        		use: ["style-loader", "css-loader", "stylus-loader"],
      			},   
        ]
    },
```

##### 处理图片资源

不需要额外依赖，已经内置在`webpack`中

添加loader规则即可

```js
{
    test:/\.png|jpe?g|gif|webp|svg$/,
    type:"asset",
}
```

处理其他资源(图片、字体等)

```js
//处理图片资源
{
    test:/\.(png|jpe?g|gif|webp|svg)$/,
    type:"asset",
    parser: {
         dataUrlCondition: {
           maxSize: 4 * 1024 // 4kb 小于4kb转换bash64 可以自定义修改
         }
     }
     generator: {
         filename: 'static/images/[hash][ext][query]' //修改输出路径以及命名方式
     }
}
// 处理其他资源字体 视频等多媒体资源
{
    test:/\.(ttl|woff2?|mp(3|4)|avi|rmvb)$/,
    type:"asset/resource", //asset/resource不会压缩  /asset会被压缩, 例如上面的图片处理低于4kb会被压缩成bash64
     generator: {
         filename: 'static/media/[hash][ext][query]' //修改输出路径以及命名方式
     }
}
```

#### 使用plugins

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack'); //to access built-in plugins

module.exports = {
  module: {
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
  plugins: [new HtmlWebpackPlugin({ template: './src/index.html' })],
};
```



## 检查处理js资源

（兼容性或者语法检查）

#### 使用`babel`解决代码兼容性问题

##### - 配置文件

`babel.config(.js|.json）`或者`balelrc（.js|json）`

以 `babel.config.js` 配置文件为例：

```js
module.exports = {
  // 预设
  presets: [],
};
```

presets 预设

简单理解：就是一组 Babel 插件, 扩展 Babel 功能

- `@babel/preset-env`: 一个智能预设，允许您使用最新的 JavaScript。
- `@babel/preset-react`：一个用来编译 React jsx 语法的预设
- `@babel/preset-typescript`：一个用来编译 TypeScript 语法的预设



##### - 在 `Webpack` 中使用

安装

```bash
pnpm add -D babel-loader @babel/core @babel/preset-env
```

配置

`babel.config.js`

```js
module.exports = {
  presets: ["@babel/preset-env"],
};
```



基本配置

`webpack.config.js`添加`bablel`的加载器`loader`

```js
const path = require("path");

module.exports = {
  module: {
    rules: [
      { //bable加载器规则
        test: /\.js$/,
        exclude: /node_modules/, // 排除node_modules代码不编译
        loader: "babel-loader",
      },
    ],
  },
};
```

这是语法会被转换基本的语法如`箭头函数`会被转换成`fuction`形式, 将`es6`语法转换成通用语法



#### 使用`eslint`解决语法问题

##### 配置文件

配置文件`.eslintrc|.eslintrc.js|.eslintrc.json`任意一个即可，不要写多个配置文件

也可以写在`package.json`中的`eslintConfig`中，不需要用新的文件

以 `.eslintrc.js` 配置文件为例：

```js
module.exports = {
  // 解析选项
  parserOptions: {},
  // 具体检查规则
  rules: {},
  // 继承其他规则
  extends: [],
  // ...
  // 其他规则详见：https://eslint.bootcss.com/docs/user-guide/configuring
};
```

1.parserOptions 解析选项

```javascript
parserOptions: {
  ecmaVersion: 6, // ES 语法版本
  sourceType: "module", // ES 模块化
  ecmaFeatures: { // ES 其他特性
    jsx: true // 如果是 React 项目，就需要开启 jsx 语法
  }
}
```

2.rules 具体规则

- `"off"` 或 `0` - 关闭规则
- `"warn"` 或 `1` - 开启规则，使用警告级别的错误：`warn` (不会导致程序退出)
- `"error"` 或 `2` - 开启规则，使用错误级别的错误：`error` (当被触发的时候，程序会退出)

```javascript
rules: {
  semi: "error", // 禁止使用分号
  'array-callback-return': 'warn', // 强制数组方法的回调函数中有 return 语句，否则警告
  'default-case': [
    'warn', // 要求 switch 语句中有 default 分支，否则警告
    { commentPattern: '^no default$' } // 允许在最后注释 no default, 就不会有警告了
  ],
  eqeqeq: [
    'warn', // 强制使用 === 和 !==，否则警告
    'smart' // https://eslint.bootcss.com/docs/rules/eqeqeq#smart 除了少数情况下不会有警告
  ],
}
```

更多规则详见：[规则文档open in new window](https://eslint.bootcss.com/docs/rules/)

3.extends 继承

开发中一点点写 rules 规则太费劲了，所以有更好的办法，继承现有的规则。

现有以下较为有名的规则：

- [Eslint 官方的规则open in new window](https://eslint.bootcss.com/docs/rules/)：`eslint:recommended`
- [Vue Cli 官方的规则open in new window](https://github.com/vuejs/vue-cli/tree/dev/packages/@vue/cli-plugin-eslint)：`plugin:vue/essential`
- [React Cli 官方的规则open in new window](https://github.com/facebook/create-react-app/tree/main/packages/eslint-config-react-app)：`react-app`

```javascript
// 例如在React项目中，我们可以这样写配置
module.exports = {
  extends: ["react-app"],
  rules: {
    // 我们的规则会覆盖掉react-app的规则
    // 所以想要修改规则直接改就是了
    eqeqeq: ["warn", "smart"],
  },
};
```



#####  [在 Webpack 中使用](https://yk2012.github.io/sgg_webpack5/base/javascript.html#_3-在-webpack-中使用) 

###### 基本配置

1.下载包

```bash
pnpm add -D eslint-webpack-plugin eslint
```

2.定义 `Eslint` 配置文件

`.eslintrc.js`

```javascript
module.exports = {
  // 继承 Eslint 规则
  extends: ["eslint:recommended"],
  env: {
    node: true, // 启用node中全局变量
    browser: true, // 启用浏览器中全局变量
  },
  parserOptions: {
    ecmaVersion: 6,
    sourceType: "module",
  },
  rules: {
    "no-var": 2, // 不能使用 var 定义变量
  },
};
```

3.修改 js 文件代码

`main.js`

```javascript
import count from "./js/count";
import sum from "./js/sum";
// 引入资源，Webpack才会对其打包
import "./css/iconfont.css";
import "./css/index.css";
import "./less/index.less";
import "./sass/index.sass";
import "./sass/index.scss";
import "./styl/index.styl";

var result1 = count(2, 1);
console.log(result1);
var result2 = sum(1, 2, 3, 4);
console.log(result2);
```

###### 插件配置

```js
const ESLintWebpackPlugin = require("eslint-webpack-plugin");

module.exports = {
  module: {
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
  plugins: [
      new HtmlWebpackPlugin(
          { 
              template: './src/index.html' 
          }
      ),
      // 指定检查文件的根目录
      new ESLintWebpackPlugin(
          {
              context: path.resolve(__dirname, "src"),
          }
      ),
  ],
};

```

添加`eslintrc.js`配置文件

```js
module.exports = {
  // 继承 Eslint 规则
  extends: ["eslint:recommended"],
  env: {
    node: true, // 启用node中全局变量
    browser: true, // 启用浏览器中全局变量
  },
  parserOptions: {
    ecmaVersion: 6,
    sourceType: "module",
  },
  rules: {
    "no-var": 2, // 不能使用 var 定义变量
  },
};
```



##### `VSCode Eslint` 插件

打开 `VSCode`，下载 `Eslint` 插件，即可不用编译就能看到错误，可以提前解决

但是此时就会对项目所有文件默认进行 `Eslint` 检查了，我们 `dist` 目录下的打包后文件就会报错。但是我们只需要检查 `src` 下面的文件，不需要检查 `dist` 下面的文件。

所以可以使用 `Eslint` 忽略文件解决。在项目根目录新建下面文件:

- `.eslintignore`

```text
# 忽略dist目录下所有文件
/dist
```







## 处理html资源



安装插件

```bash
pnpm add -D html-webpack-plugin
```

配置文件添加插件

`webpack.config.js`

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "public/index.html"),
    }),
  ],
};
```



## 开发服务器自动化（热部署）

开发环境下不会产生`打包文件(dist)`，开发服务器运行在内存中

下载依赖

```bash
pnpm add -D webpack-dev-server
```

添加配置

`webpack.config.js`

```js
module.exports = {
  // 开发服务器
  devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
  },
};
```



## 环境切分

目录树

```sh
|-demo_test_01
	|-config
		|- webpack.dev.js
		|- webpack.prod.js
	|- public
	|	|- index.html
	|- src
	|	|- js
	|	|	|- count.js
	|	|	|- sum.js
	|	|- main.js
	|- package.json
```



config配置文件

### 开发环境配置

#### 配置文件

`webpack.dev.js`

因为文件目录变了，所以所有绝对路径需要回退一层目录才能找到对应的文件

```js
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: undefined, // 开发模式没有输出，不需要指定输出目录
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    // clean: true, // 开发模式没有输出，不需要清空输出结果
  },
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.less$/,
        use: ["style-loader", "css-loader", "less-loader"],
      },
      {
        test: /\.s[ac]ss$/,
        use: ["style-loader", "css-loader", "sass-loader"],
      },
      {
        test: /\.styl$/,
        use: ["style-loader", "css-loader", "stylus-loader"],
      },
      {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          },
        },
        generator: {
          // 将图片文件输出到 static/imgs 目录中
          // 将图片文件命名 [hash:8][ext][query]
          // [hash:8]: hash值取8位
          // [ext]: 使用之前的文件扩展名
          // [query]: 添加之前的query参数
          filename: "static/imgs/[hash:8][ext][query]",
        },
      },
      {
        test: /\.(ttf|woff2?)$/,
        type: "asset/resource",
        generator: {
          filename: "static/media/[hash:8][ext][query]",
        },
      },
      {
        test: /\.js$/,
        exclude: /node_modules/, // 排除node_modules代码不编译
        loader: "babel-loader",
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
  ],
  // 其他省略
  devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
  },
  mode: "development",
};
```



#### 指令

运行开发模式的指令：

```bash
npx webpack serve --config ./config/webpack.dev.js
```

```bash
pnpm webpack serve --config ./config/webpack.dev.js
```



### 生产环境配置

#### 配置文件

`webpack.prod.js`

```js
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "../dist"), // 生产模式需要输出
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    clean: true,
  },
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.less$/,
        use: ["style-loader", "css-loader", "less-loader"],
      },
      {
        test: /\.s[ac]ss$/,
        use: ["style-loader", "css-loader", "sass-loader"],
      },
      {
        test: /\.styl$/,
        use: ["style-loader", "css-loader", "stylus-loader"],
      },
      {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          },
        },
        generator: {
          // 将图片文件输出到 static/imgs 目录中
          // 将图片文件命名 [hash:8][ext][query]
          // [hash:8]: hash值取8位
          // [ext]: 使用之前的文件扩展名
          // [query]: 添加之前的query参数
          filename: "static/imgs/[hash:8][ext][query]",
        },
      },
      {
        test: /\.(ttf|woff2?)$/,
        type: "asset/resource",
        generator: {
          filename: "static/media/[hash:8][ext][query]",
        },
      },
      {
        test: /\.js$/,
        exclude: /node_modules/, // 排除node_modules代码不编译
        loader: "babel-loader",
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
  ],
  // devServer: {
  //   host: "localhost", // 启动服务器域名
  //   port: "3000", // 启动服务器端口号
  //   open: true, // 是否自动打开浏览器
  // },
  mode: "production",
};
```



#### 指令

```bash
npx webpack --config ./config/webpack.prod.js
```

```bash
pnpm webpack --config ./config/webpack.prod.js
```



### 在`package.json`添加配置`scripts`配置切换环境



```json
// package.json
{
  // 其他省略
  "scripts": {
    "start": "npm run dev",
    "dev": "npx webpack serve --config ./config/webpack.dev.js",
    "build": "npx webpack --config ./config/webpack.prod.js"
  }
}
```

或者

```json
// package.json
{
  // 其他省略
  "scripts": {
    "start": "pnpm run dev",
    "dev": "pnpm webpack serve --config ./config/webpack.dev.js",
    "build": "pnpm webpack --config ./config/webpack.prod.js"
  }
}
```



以后启动指令：

- 开发模式：`npm start` 或 `npm run dev`
- 生产模式：`npm run build`

```bash
pnpm run dev
pnpm run build
```





## `CSS`处理（生产环境）

### 打包处理



`CSS` 文件目前被打包到 `js` 文件中，当 js 文件加载时，会创建一个 `style` 标签来生成样式

这样对于网站来说，会出现闪屏现象，用户体验不好

我们应该是单独的 `CSS` 文件，通过 `link` 标签加载性能才好



#### 安装插件

```bash
pnpm add -D mini-css-extract-plugin
```



####  配置

`webpack.prod.js`

##### 引入插件

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
```



##### 添加插件配置

```js
new MiniCssExtractPlugin({
  // 定义输出文件名和目录
  filename: "static/css/main.css",
}),
```


##### 修改`loader`规则

将style-loader替换成`MiniCssExtractPlugin.loader`

`use: [MiniCssExtractPlugin.loader, "css-loader"],`

```js
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "../dist"), // 生产模式需要输出
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    clean: true,
  },
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: [MiniCssExtractPlugin.loader, "css-loader"],
      },
      {
        test: /\.less$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "less-loader"],
      },
      {
        test: /\.s[ac]ss$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "sass-loader"],
      },
      {
        test: /\.styl$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "stylus-loader"],
      },
      {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          },
        },
        generator: {
          // 将图片文件输出到 static/imgs 目录中
          // 将图片文件命名 [hash:8][ext][query]
          // [hash:8]: hash值取8位
          // [ext]: 使用之前的文件扩展名
          // [query]: 添加之前的query参数
          filename: "static/imgs/[hash:8][ext][query]",
        },
      },
      {
        test: /\.(ttf|woff2?)$/,
        type: "asset/resource",
        generator: {
          filename: "static/media/[hash:8][ext][query]",
        },
      },
      {
        test: /\.js$/,
        exclude: /node_modules/, // 排除node_modules代码不编译
        loader: "babel-loader",
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
    // 提取css成单独文件
    new MiniCssExtractPlugin({
      // 定义输出文件名和目录
      filename: "static/css/main.css",
    }),
  ],
  // devServer: {
  //   host: "localhost", // 启动服务器域名
  //   port: "3000", // 启动服务器端口号
  //   open: true, // 是否自动打开浏览器
  // },
  mode: "production",
};
```





### 兼容性处理

`webpack.prod.js`

```js
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "../dist"), // 生产模式需要输出
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    clean: true,
  },
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          {
            loader: "postcss-loader",
            options: {
              postcssOptions: {
                plugins: [
                  "postcss-preset-env", // 能解决大多数样式兼容性问题
                ],
              },
            },
          },
        ],
      },
      {
        test: /\.less$/,
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          {
            loader: "postcss-loader",
            options: {
              postcssOptions: {
                plugins: [
                  "postcss-preset-env", // 能解决大多数样式兼容性问题
                ],
              },
            },
          },
          "less-loader",
        ],
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          {
            loader: "postcss-loader",
            options: {
              postcssOptions: {
                plugins: [
                  "postcss-preset-env", // 能解决大多数样式兼容性问题
                ],
              },
            },
          },
          "sass-loader",
        ],
      },
      {
        test: /\.styl$/,
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          {
            loader: "postcss-loader",
            options: {
              postcssOptions: {
                plugins: [
                  "postcss-preset-env", // 能解决大多数样式兼容性问题
                ],
              },
            },
          },
          "stylus-loader",
        ],
      },
      {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          },
        },
        generator: {
          // 将图片文件输出到 static/imgs 目录中
          // 将图片文件命名 [hash:8][ext][query]
          // [hash:8]: hash值取8位
          // [ext]: 使用之前的文件扩展名
          // [query]: 添加之前的query参数
          filename: "static/imgs/[hash:8][ext][query]",
        },
      },
      {
        test: /\.(ttf|woff2?)$/,
        type: "asset/resource",
        generator: {
          filename: "static/media/[hash:8][ext][query]",
        },
      },
      {
        test: /\.js$/,
        exclude: /node_modules/, // 排除node_modules代码不编译
        loader: "babel-loader",
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
    // 提取css成单独文件
    new MiniCssExtractPlugin({
      // 定义输出文件名和目录
      filename: "static/css/main.css",
    }),
  ],
  // devServer: {
  //   host: "localhost", // 启动服务器域名
  //   port: "3000", // 启动服务器端口号
  //   open: true, // 是否自动打开浏览器
  // },
  mode: "production",
};
```





#### 安装插件

```bash
pnpm add -D postcss-loader postcss postcss-preset-env 
```

####  配置文件



不需要添加插件项

##### 添加加载器规则

`postcss-loader`要配在`css-loader`后面

`use` 数组里面 Loader 执行顺序是`从右到左`

```js
        // use 数组里面 Loader 执行顺序是从右到左
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          {
            loader: "postcss-loader",
            options: {
              postcssOptions: {
                plugins: [
                  "postcss-preset-env", // 能解决大多数样式兼容性问题
                ],
              },
            },
          },
        ],
```

##### 修改`package.json`

```json
{
  "name": "demo_test_01",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "pnpm run dev",
    "dev": "pnpm webpack serve --config ./config/webpack.dev.js",
    "build": "pnpm webpack --config ./config/webpack.prod.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "css-loader": "^6.7.1",
    "html-webpack-plugin": "^5.5.0",
    "mini-css-extract-plugin": "^2.6.1",
    "postcss": "^8.4.16",
    "postcss-loader": "^7.0.1",
    "postcss-preset-env": "^7.8.0",
    "style-loader": "^3.3.1",
    "webpack": "^5.74.0",
    "webpack-cli": "^4.10.0",
    "webpack-dev-server": "^4.10.1"
  },
  "browserslist": ["last 2 version", "> 1%", "not dead"]
}
```



添加配置

```json
{
  // 其他省略
  "browserslist": ["ie >= 8"]
}
```

想要知道更多的 `browserslist` 配置，查看[browserslist 文档open in new window](https://github.com/browserslist/browserslist)

以上为了测试兼容性所以设置兼容浏览器 ie8 以上。

实际开发中我们一般不考虑旧版本浏览器了，所以我们可以这样设置：

```json
{
  // 其他省略
  "browserslist": ["last 2 version", "> 1%", "not dead"]
}
```





### 合并配置

`webpack.prod.js`

```js
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

// 获取处理样式的Loaders
const getStyleLoaders = (preProcessor) => {
  return [
    MiniCssExtractPlugin.loader,
    "css-loader",
    {
      loader: "postcss-loader",
      options: {
        postcssOptions: {
          plugins: [
            "postcss-preset-env", // 能解决大多数样式兼容性问题
          ],
        },
      },
    },
    preProcessor,
  ].filter(Boolean);
};

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "../dist"), // 生产模式需要输出
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    clean: true,
  },
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: getStyleLoaders(),
      },
      {
        test: /\.less$/,
        use: getStyleLoaders("less-loader"),
      },
      {
        test: /\.s[ac]ss$/,
        use: getStyleLoaders("sass-loader"),
      },
      {
        test: /\.styl$/,
        use: getStyleLoaders("stylus-loader"),
      },
      {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          },
        },
        generator: {
          // 将图片文件输出到 static/imgs 目录中
          // 将图片文件命名 [hash:8][ext][query]
          // [hash:8]: hash值取8位
          // [ext]: 使用之前的文件扩展名
          // [query]: 添加之前的query参数
          filename: "static/imgs/[hash:8][ext][query]",
        },
      },
      {
        test: /\.(ttf|woff2?)$/,
        type: "asset/resource",
        generator: {
          filename: "static/media/[hash:8][ext][query]",
        },
      },
      {
        test: /\.js$/,
        exclude: /node_modules/, // 排除node_modules代码不编译
        loader: "babel-loader",
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
    // 提取css成单独文件
    new MiniCssExtractPlugin({
      // 定义输出文件名和目录
      filename: "static/css/main.css",
    }),
  ],
  // devServer: {
  //   host: "localhost", // 启动服务器域名
  //   port: "3000", // 启动服务器端口号
  //   open: true, // 是否自动打开浏览器
  // },
  mode: "production",
};
```





### 压缩处理

在生产环境中已经默认压缩`html`和`js`文件不需要额外操作，现在对`css`进行压缩处理

#### 安装插件

```bash
pnpm add -D css-minimizer-webpack-plugin
```

#### 配置 

`webpack.prod.js`

```js
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");

// 获取处理样式的Loaders
const getStyleLoaders = (preProcessor) => {
  return [
    MiniCssExtractPlugin.loader,
    "css-loader",
    {
      loader: "postcss-loader",
      options: {
        postcssOptions: {
          plugins: [
            "postcss-preset-env", // 能解决大多数样式兼容性问题
          ],
        },
      },
    },
    preProcessor,
  ].filter(Boolean);
};

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "../dist"), // 生产模式需要输出
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    clean: true,
  },
  module: {
    rules: [
      {
        // 用来匹配 .css 结尾的文件
        test: /\.css$/,
        // use 数组里面 Loader 执行顺序是从右到左
        use: getStyleLoaders(),
      },
      {
        test: /\.less$/,
        use: getStyleLoaders("less-loader"),
      },
      {
        test: /\.s[ac]ss$/,
        use: getStyleLoaders("sass-loader"),
      },
      {
        test: /\.styl$/,
        use: getStyleLoaders("stylus-loader"),
      },
      {
        test: /\.(png|jpe?g|gif|webp)$/,
        type: "asset",
        parser: {
          dataUrlCondition: {
            maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
          },
        },
        generator: {
          // 将图片文件输出到 static/imgs 目录中
          // 将图片文件命名 [hash:8][ext][query]
          // [hash:8]: hash值取8位
          // [ext]: 使用之前的文件扩展名
          // [query]: 添加之前的query参数
          filename: "static/imgs/[hash:8][ext][query]",
        },
      },
      {
        test: /\.(ttf|woff2?)$/,
        type: "asset/resource",
        generator: {
          filename: "static/media/[hash:8][ext][query]",
        },
      },
      {
        test: /\.js$/,
        exclude: /node_modules/, // 排除node_modules代码不编译
        loader: "babel-loader",
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
    // 提取css成单独文件
    new MiniCssExtractPlugin({
      // 定义输出文件名和目录
      filename: "static/css/main.css",
    }),
    // css压缩
    new CssMinimizerPlugin(),
  ],
  // devServer: {
  //   host: "localhost", // 启动服务器域名
  //   port: "3000", // 启动服务器端口号
  //   open: true, // 是否自动打开浏览器
  // },
  mode: "production",
};
```



##### 引入插件

```js
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");
```



##### 添加插件配置

```js
  plugins: [
    // css压缩
    new CssMinimizerPlugin(),
  ],
```

或者

```js
  optimization: {
    minimize: true,
    minimizer: [
      // css压缩也可以写到optimization.minimizer里面，效果一样的
      new CssMinimizerPlugin(),
    ],
  },
```





## `SourceMap`处理报错问题



SourceMap（源代码映射）是一个用来生成源代码与构建后代码一一映射的文件的方案。

它会生成一个 xxx.map 文件，里面包含源代码和构建后代码每一行、每一列的映射关系。当构建后代码出错了，会通过 xxx.map 文件，从构建后代码出错位置找到映射后源代码出错位置，从而让浏览器提示源代码文件出错位置，帮助我们更快的找到错误根源。

### [怎么用 ](https://yk2012.github.io/sgg_webpack5/senior/enhanceExperience.html#怎么用)

通过查看[Webpack DevTool 文档open in new window](https://webpack.docschina.org/configuration/devtool/)可知，SourceMap 的值有很多种情况.

但实际开发时我们只需要关注两种情况即可：

开发模式：`cheap-module-source-map`

- 优点：打包编译速度快，只包含行映射
- 缺点：没有列映射

```javascript
module.exports = {
  // 其他省略
  mode: "development",
  devtool: "cheap-module-source-map",
};
```

生产模式：

```
source-map
```

- 优点：包含行/列映射
- 缺点：打包编译速度更慢



```javascript
module.exports = {
  // 其他省略
  mode: "production",
  devtool: "source-map",
};
```



## 提升打包构建速度(开发环境)

### `HotModuleReplacement`

热模块替换

配置

```js
module.exports = {
  // 其他省略
  devServer: {
    host: "localhost", // 启动服务器域名
    port: "3000", // 启动服务器端口号
    open: true, // 是否自动打开浏览器
    hot: true, // 开启HMR功能（只能用于开发环境，生产环境不需要了）
  },
};
```

此时 `css` 样式经过 style-loader 处理，已经具备 HMR 功能了。 但是 js 还不行。

JS 配置

```javascript
// main.js
import count from "./js/count";
import sum from "./js/sum";
// 引入资源，Webpack才会对其打包
import "./css/iconfont.css";
import "./css/index.css";
import "./less/index.less";
import "./sass/index.sass";
import "./sass/index.scss";
import "./styl/index.styl";

const result1 = count(2, 1);
console.log(result1);
const result2 = sum(1, 2, 3, 4);
console.log(result2);

// 判断是否支持HMR功能
if (module.hot) {
  module.hot.accept("./js/count.js", function (count) {
    const result1 = count(2, 1);
    console.log(result1);
  });

  module.hot.accept("./js/sum.js", function (sum) {
    const result2 = sum(1, 2, 3, 4);
    console.log(result2);
  });
}
```



上面这样写会很麻烦，所以实际开发我们会使用其他 loader 来解决。

比如：[vue-loader](https://github.com/vuejs/vue-loader), [react-hot-loader](https://github.com/gaearon/react-hot-loader)。



### `Oneof`

使用场景: 开发环境和生产环境 `webpack.dev.js` | `webpack.prod.js`

每个文件指挥被其中一个`loader`处理

打包时每个文件都会经过所有 `loader` 处理，虽然因为 `test` 正则原因实际没有处理上，但是都要过一遍。比较慢。

配置方式

将所有规则放到`oneof`数组里面

```js
    rules: [
      {
        oneOf: [
          {
            // 用来匹配 .css 结尾的文件
            test: /\.css$/,
            // use 数组里面 Loader 执行顺序是从右到左
            use: ["style-loader", "css-loader"],
          },
          {
            test: /\.less$/,
            use: ["style-loader", "css-loader", "less-loader"],
          },
          {
            test: /\.s[ac]ss$/,
            use: ["style-loader", "css-loader", "sass-loader"],
          },
          {
            test: /\.styl$/,
            use: ["style-loader", "css-loader", "stylus-loader"],
          },
          {
            test: /\.(png|jpe?g|gif|webp)$/,
            type: "asset",
            parser: {
              dataUrlCondition: {
                maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
              },
            },
            generator: {
              // 将图片文件输出到 static/imgs 目录中
              // 将图片文件命名 [hash:8][ext][query]
              // [hash:8]: hash值取8位
              // [ext]: 使用之前的文件扩展名
              // [query]: 添加之前的query参数
              filename: "static/imgs/[hash:8][ext][query]",
            },
          },
          {
            test: /\.(ttf|woff2?)$/,
            type: "asset/resource",
            generator: {
              filename: "static/media/[hash:8][ext][query]",
            },
          },
          {
            test: /\.js$/,
            exclude: /node_modules/, // 排除node_modules代码不编译
            loader: "babel-loader",
          },
        ],
      },
    ],
```



### `include/exclude`

为什么

开发时我们需要使用第三方的库或插件，所有文件都下载到 node_modules 中了。而这些文件是不需要编译可以直接使用的。

所以我们在对 js 文件处理时，要排除 node_modules 下面的文件。

[是什么](https://yk2012.github.io/sgg_webpack5/senior/liftingSpeed.html#是什么-2)

`include` 包含，只处理 xxx 文件

`exclude` 排除，除了 xxx 文件以外其他文件都处理

配置

```js
{
    test: /\.js$/,
    // exclude: /node_modules/, // 排除node_modules代码不编译
    include: path.resolve(__dirname, "../src"), // 也可以用包含
    loader: "babel-loader",
},
```

```js
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
      exclude: "node_modules", // 默认值
    }),
  ],
```



### `Cache`

每次打包时 js 文件都要经过 Eslint 检查 和 Babel 编译，速度比较慢。

我们可以缓存之前的 Eslint 检查 和 Babel 编译结果，这样第二次打包时速度就会更快了。

[是什么](https://yk2012.github.io/sgg_webpack5/senior/liftingSpeed.html#是什么-3)

对 `Eslint` 检查 和 `Babel` 编译结果进行缓存。

`babel`加载器配置

```js
          {
            test: /\.js$/,
            // exclude: /node_modules/, // 排除node_modules代码不编译
            include: path.resolve(__dirname, "../src"), // 也可以用包含
            loader: "babel-loader",
            options: {
              cacheDirectory: true, // 开启babel编译缓存
              cacheCompression: false, // 缓存文件不要压缩
            },
          },
```

`eslint`插件配置

```js
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
      exclude: "node_modules", // 默认值
      cache: true, // 开启缓存
      // 缓存目录
      cacheLocation: path.resolve(
        __dirname,
        "../node_modules/.cache/.eslintcache"
      ),
    }),
```



### `Thead`

当项目越来越庞大时，打包速度越来越慢，甚至于需要一个下午才能打包出来代码。这个速度是比较慢的。

我们想要继续提升打包速度，其实就是要提升 js 的打包速度，因为其他文件都比较少。

而对 js 文件处理主要就是 `eslint` 、`babel`、`Terser` 三个工具，所以我们要提升它们的运行速度。

我们可以开启多进程同时处理 js 文件，这样速度就比之前的单进程打包更快了。

[是什么](https://yk2012.github.io/sgg_webpack5/senior/liftingSpeed.html#是什么-4)

多进程打包：开启电脑的多个进程同时干一件事，速度更快。

**需要注意：请仅在特别耗时的操作中使用，因为每个进程启动就有大约为 600ms 左右开销。**

[怎么用](https://yk2012.github.io/sgg_webpack5/senior/liftingSpeed.html#怎么用-4)

我们启动进程的数量就是我们 CPU 的核数。

1.如何获取 CPU 的核数，因为每个电脑都不一样。

```javascript
// nodejs核心模块，直接使用
const os = require("os");
// cpu核数
const threads = os.cpus().length;
```

2.下载包

```bash
pnpm add -D thread-loader
```

3.使用

引入配置

开发环境，生产环境使用同样配置

```js
const os = require("os");
const TerserPlugin = require("terser-webpack-plugin");

// cpu核数
const threads = os.cpus().length;
```

```js
plugins:[
	new ESLintWebpackPlugin({
	    // 指定检查文件的根目录
    	context: path.resolve(__dirname, "../src"),
	    exclude: "node_modules", // 默认值
    	cache: true, // 开启缓存
	    // 缓存目录
    	cacheLocation: path.resolve(
	      	__dirname,
	      	"../node_modules/.cache/.eslintcache"
	    ),
	    threads,
	}),
]
```



```js
//babel配置
{
    test: /\.js$/,
    //exclude: /node_modules/, // 排除node_modules代码不编译
    include: path.resolve(__dirname, "../src"), // 也可以用包含
    use:[
      {
        loader: "thread-loader", // 开启多进程
        options: {
          workers: threads, // 数量
        },
      },{
        loader: "babel-loader",
        options: {
            cacheDirectory: true, // 开启babel编译缓存
            cacheCompression: false, // 缓存文件不要压缩
        },
      }
    ],
},
```



```js
  optimization: {
    minimize: true,
    minimizer: [
      // css压缩也可以写到optimization.minimizer里面，效果一样的
      new CssMinimizerPlugin(),
      // 当生产模式会默认开启TerserPlugin，但是我们需要进行其他配置，就要重新写了
      new TerserPlugin({
        parallel: threads // 开启多进程
      })
    ],
  },

```





```js
const os = require("os");
const path = require("path");
const ESLintWebpackPlugin = require("eslint-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");
const TerserPlugin = require("terser-webpack-plugin");

// cpu核数
const threads = os.cpus().length;

// 获取处理样式的Loaders
const getStyleLoaders = (preProcessor) => {
  return [
    MiniCssExtractPlugin.loader,
    "css-loader",
    {
      loader: "postcss-loader",
      options: {
        postcssOptions: {
          plugins: [
            "postcss-preset-env", // 能解决大多数样式兼容性问题
          ],
        },
      },
    },
    preProcessor,
  ].filter(Boolean);
};

module.exports = {
  entry: "./src/main.js",
  output: {
    path: path.resolve(__dirname, "../dist"), // 生产模式需要输出
    filename: "static/js/main.js", // 将 js 文件输出到 static/js 目录中
    clean: true,
  },
  module: {
    rules: [
      {
        oneOf: [
          {
            // 用来匹配 .css 结尾的文件
            test: /\.css$/,
            // use 数组里面 Loader 执行顺序是从右到左
            use: getStyleLoaders(),
          },
          {
            test: /\.less$/,
            use: getStyleLoaders("less-loader"),
          },
          {
            test: /\.s[ac]ss$/,
            use: getStyleLoaders("sass-loader"),
          },
          {
            test: /\.styl$/,
            use: getStyleLoaders("stylus-loader"),
          },
          {
            test: /\.(png|jpe?g|gif|webp)$/,
            type: "asset",
            parser: {
              dataUrlCondition: {
                maxSize: 10 * 1024, // 小于10kb的图片会被base64处理
              },
            },
            generator: {
              // 将图片文件输出到 static/imgs 目录中
              // 将图片文件命名 [hash:8][ext][query]
              // [hash:8]: hash值取8位
              // [ext]: 使用之前的文件扩展名
              // [query]: 添加之前的query参数
              filename: "static/imgs/[hash:8][ext][query]",
            },
          },
          {
            test: /\.(ttf|woff2?)$/,
            type: "asset/resource",
            generator: {
              filename: "static/media/[hash:8][ext][query]",
            },
          },
          {
            test: /\.js$/,
            // exclude: /node_modules/, // 排除node_modules代码不编译
            include: path.resolve(__dirname, "../src"), // 也可以用包含
            use: [
              {
                loader: "thread-loader", // 开启多进程
                options: {
                  workers: threads, // 数量
                },
              },
              {
                loader: "babel-loader",
                options: {
                  cacheDirectory: true, // 开启babel编译缓存
                },
              },
            ],
          },
        ],
      },
    ],
  },
  plugins: [
    new ESLintWebpackPlugin({
      // 指定检查文件的根目录
      context: path.resolve(__dirname, "../src"),
      exclude: "node_modules", // 默认值
      cache: true, // 开启缓存
      // 缓存目录
      cacheLocation: path.resolve(
        __dirname,
        "../node_modules/.cache/.eslintcache"
      ),
      threads, // 开启多进程
    }),
    new HtmlWebpackPlugin({
      // 以 public/index.html 为模板创建文件
      // 新的html文件有两个特点：1. 内容和源文件一致 2. 自动引入打包生成的js等资源
      template: path.resolve(__dirname, "../public/index.html"),
    }),
    // 提取css成单独文件
    new MiniCssExtractPlugin({
      // 定义输出文件名和目录
      filename: "static/css/main.css",
    }),
    // css压缩
    // new CssMinimizerPlugin(),
  ],
  optimization: {
    minimize: true,
    minimizer: [
      // css压缩也可以写到optimization.minimizer里面，效果一样的
      new CssMinimizerPlugin(),
      // 当生产模式会默认开启TerserPlugin，但是我们需要进行其他配置，就要重新写了
      new TerserPlugin({
        parallel: threads // 开启多进程
      })
    ],
  },
  // devServer: {
  //   host: "localhost", // 启动服务器域名
  //   port: "3000", // 启动服务器端口号
  //   open: true, // 是否自动打开浏览器
  // },
  mode: "production",
  devtool: "source-map",
};
```





## 减小代码体积

### `Tree Shaking`

#### 为什么

开发时我们定义了一些工具函数库，或者引用第三方工具函数库或组件库。

如果没有特殊处理的话我们打包时会引入整个库，但是实际上可能我们可能只用上极小部分的功能。

这样将整个库都打包进来，体积就太大了。

#### 是什么

`Tree Shaking` 是一个术语，通常用于描述移除 JavaScript 中的没有使用上的代码。

#### 怎么用

`Webpack` 已经默认开启了这个功能，无需其他配置。



### `Babel`

#### 为什么

Babel 为编译的每个文件都插入了辅助代码，使代码体积过大！

Babel 对一些公共方法使用了非常小的辅助代码，比如 `_extend`。默认情况下会被添加到每一个需要它的文件中。

你可以将这些辅助代码作为一个独立模块，来避免重复引入

#### 是什么

`@babel/plugin-transform-runtime`: 禁用了 Babel 自动对每个文件的 runtime 注入，而是引入 `@babel/plugin-transform-runtime` 并且使所有辅助代码从这里引用

#### 怎么用

##### 安装

```bash
pnpm add -D @babel/plugin-transform-runtime
```

##### 配置

修改`babel-loader`配置添加插件

```js
{
      loader: "babel-loader",
       options: {
       cacheDirectory: true, // 开启babel编译缓存
       cacheCompression: false, // 缓存文件不要压缩
       plugins: ["@babel/plugin-transform-runtime"], // 减少代码体积
	},
},
```



### `Image Minimizer`



#### 为什么

开发如果项目中引用了较多图片，那么图片体积会比较大，将来请求速度比较慢。

我们可以对图片进行压缩，减少图片体积。

**注意：如果项目中图片都是在线链接，那么就不需要了。本地项目静态图片才需要进行压缩。**

#### [是什么](https://yk2012.github.io/sgg_webpack5/senior/reduceVolume.html#是什么-2)

`image-minimizer-webpack-plugin`: 用来压缩图片的插件

#### [怎么用](https://yk2012.github.io/sgg_webpack5/senior/reduceVolume.html#怎么用-2)

##### 安装

```bash
pnpm add -D image-minimizer-webpack-plugin imagemin
```

还有剩下包需要下载，有两种模式：

无损压缩

```bash
pnpm add -D imagemin-gifsicle imagemin-jpegtran imagemin-optipng imagemin-svgo
```

有损压缩

```bash
pnpm add -D imagemin-gifsicle imagemin-mozjpeg imagemin-pngquant imagemin-svgo
```

##### 配置

以无损压缩为例

###### 引入包

```js
const ImageMinimizerPlugin = require("image-minimizer-webpack-plugin");
```

###### 修改配置

```js
  optimization: {
    minimizer: [
      // css压缩也可以写到optimization.minimizer里面，效果一样的
      new CssMinimizerPlugin(),
      // 当生产模式会默认开启TerserPlugin，但是我们需要进行其他配置，就要重新写了
      new TerserPlugin({
        parallel: threads, // 开启多进程
      }),
      // 压缩图片
      new ImageMinimizerPlugin({
        minimizer: {
          implementation: ImageMinimizerPlugin.imageminGenerate,
          options: {
            plugins: [
              ["gifsicle", { interlaced: true }],
              ["jpegtran", { progressive: true }],
              ["optipng", { optimizationLevel: 5 }],
              [
                "svgo",
                {
                  plugins: [
                    "preset-default",
                    "prefixIds",
                    {
                      name: "sortAttrs",
                      params: {
                        xmlnsOrder: "alphabetical",
                      },
                    },
                  ],
                },
              ],
            ],
          },
        },
      }),
    ],
  },
```



打包时会出现报错：

```bash
Error: Error with 'src\images\1.jpeg': '"C:\Users\86176\Desktop\webpack\webpack_code\node_modules\jpegtran-bin\vendor\jpegtran.exe"'
Error with 'src\images\3.gif': spawn C:\Users\86176\Desktop\webpack\webpack_code\node_modules\optipng-bin\vendor\optipng.exe ENOENT
```

我们需要安装两个文件到 node_modules 中才能解决, 文件可以从课件中找到：

- jpegtran.exe

需要复制到 `node_modules\jpegtran-bin\vendor` 下面

> [jpegtran 官网地址open in new window](http://jpegclub.org/jpegtran/)

- optipng.exe

需要复制到 `node_modules\optipng-bin\vendor` 下面

> [OptiPNG 官网地址](http://optipng.sourceforge.net/)




















































































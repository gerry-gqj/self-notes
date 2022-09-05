# webpack基础



## 基本demo



简单demo



项目结构

```bash
demo_test_01
	|+ public
		|| -index.html
	|+ src
		|| -main.js
		||+ js
			||| -count.js
			||| -sum.js
	| -package.jon
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

生成package.son

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



#### - 使用entry

```js
//入口
entry: "./src/main.js",
```



#### - 使用output

```js
//输出
output:{
    //所有文件输出路径
    path:path.resolve(__dirname, "dist"),
    //文件名
    filename:'static/js/main.js', //js打包输出路径
},
```

#### - 使用loader

安装`css-loader`

```bash
pnpm add --save-dev css-loader
pnpm add --save-dev style-loader
```

添加配置

处理css资源

```js
    //加载器
    module:{
        rules:[
            //loader的配置
            {
                test: /\.css$/, 
                use:[
                        // [style-loader](/loaders/style-loader)
                        { loader: 'style-loader' }, //将js中引入的css生成style标签在html中
                        // [css-loader](/loaders/css-loader)
                        {
                            loader: 'css-loader', // 将引入的css编译打包成commonjs的模块js中
                            //options: {
                            //    modules: true
                            //}
                        },
                ]
            }
            
        ]
    },
```

处理图片资源(不需要额外依赖，已经内置在webpack中)

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

#### - 使用plugins

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





## 检查处理js资源（兼容性或者语法检查）

#### 使用bable解决代码兼容性问题

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



##### - 在 Webpack 中使用

```bash
pnpm add  -D babel-loader @babel/core @babel/preset-env
```

基本配置

`webpack.config.js`添加`bablel`的加载器l`loader`

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





#### 使用eslint解决语法问题

##### - 配置文件

配置文件`eslintrc|eslintrc.js|eslintrc.json`任意一个即可，不要写多个配置文件

也可以卸载package.json中的eslintConfig中，不需要用新的文件

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



##### - [在 Webpack 中使用](https://yk2012.github.io/sgg_webpack5/base/javascript.html#_3-在-webpack-中使用) 

###### 基本配置

1.下载包

```text
pnpm add eslint-webpack-plugin eslint -D
```

2.定义 Eslint 配置文件

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


























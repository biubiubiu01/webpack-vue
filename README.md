# webpack-vue

## 总览

> 前言： 第一次学习webpack,为了深入了解webpack运行机制和原理,就自己搭建了一个，感觉个人搭建的看看就好,还需要多看看大佬是如何搭建脚手架的

> webpack-vue是为了熟悉学习webpack运行机制和原理搭建的项目，从0开始一步步搭建,并记录搭建中使用的插件和实际的运用

使用webpack 4.43.0

## 包含webpack插件和loader
```
* html:    
  html-webpack-plugin clean-webpack-plugin

* css:    
  style-loader css-loader sass-loader node-sass postcss-loader autoprefixer

* js:      
  babel-loader @babel/preset-env @babel/core @babel/polyfill

* vue:     
  vue-loader vue-template-compiler vue-style-loader vue vue-router axios vuex

* webpack: 

  file-loader url-loader webpack-dev-server webpack-merge copy-webpack-plugin happypack HardSourceWebpackPlugin 
  webpack-bundle-analyzer

* 其他:
  Echarts,ElementUI,driver.js,vue-count-to等
```

## webpack功能

-- 生成hmtl模板

-- 删除上一次的dist文件

-- 自动添加浏览器前缀

-- 使用sass预编译器

-- 转换ES6,7,8,9语法为ES5

-- 大于10k文件打包到dist，小于10k转换为base64

-- 兼容vue

-- 热更新

-- 区分当前环境

-- 多线程打包

-- 缓存未改变模块

-- 打包大小分析

## 实现步骤

### 1.初体验
```
1. 新建一个文件 取名为webpack-vue

2. cd webpack-vue  npm init -y  npm i -D webpack webpack-cli

3. 新建 src/main.js ,里面随便写点 console.log('hello,webpack')

4. 修改 package.json - >scripts ,添加 "build":"webpack src/main.js"

5. 然后 npm run build 如果多了一个dist文件,那么初次打包就成功了
```

### 2.配置

1. 新建一个 build 文件夹,新建一个 webpack.config.js

2. 写入以下内容
```
const path=require('path')

module.exports = {
    mode:'development',
    entry:path.resolve(__dirname,'../src/main.js'),  //入口文件
    output:{
        filename:'[name].[hash:8].js',   //打包后的名字  生成8位数的hash
        path.resolve(__dirname,'../dist')   //打包的路径
     }
 }
```
`然后修改 package.json ->scripts,修改为： "build":"webpack --config build/webpack.config.js"`

 然后npm run build

3.我们生成了main.js之后,不可能每次都手动在index.html里面引入，所以我们需要这个插件来帮我们自动引入

先安装插件:

`npm i -D html-webpack-plugin`

根目录新建一个 public/index.html

修改webpack.config.js

```
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin')   //这里引入插件
  module.exports = {
      mode:'development', // 开发模式
      entry: path.resolve(__dirname,'../src/main.js'),    // 入口文件
      output: {
        filename: '[name].[hash].js',      // 打包后的文件名称
        path: path.resolve(__dirname,'../dist')  // 打包后的目录
      },
      //插件注入
      plugins:[
        new HtmlWebpackPlugin({
          template:path.resolve(__dirname,'../public/index.html')
        })
      ]
  }
```

然后npm run build 就会发现dist里面多了index.html，并且已经自动帮我们引入了main.js

4.由于hash每次生成的不同，导致每次打包都会将新的main.js打包到dist文件夹，所以我们需要一个插件来打包前删除dist文件

`npm i -D clean-webpack-plugin`
```
webpack.config.js

const {CleanWebpackPlugin} = require('clean-webpack-plugin')   //引入

 plugins:[
    new HtmlWebpackPlugin({
        template:path.resolve(__dirname,'../public/index.html')
    }),
    new CleanWebpackPlugin()
 ]
```

5.为了让webpack识别css,我们需要安装loader，并将解析后的css插入到index.html里面的style

先安装:

`npm i -D style-loader css-loader`

```
  webpack.config.js

module.exports = {
    module:{
        rules:[{
          test:/\.css$/,
          use:['style-loader','css-loader'] // 从右向左解析原则
        }]
    }
} 
```

6.我们这里可以使用预编译器更好的处理css，我这里使用的是sass

先安装:

`npm install -D  sass-loader  node-sass`

```
 webpack.config.js

module:{
    rules: [{
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader'] // 从右向左解析原则
    }]
}
```

7.自动添加浏览器前缀

先安装：

` npm i -D postcss-loader autoprefixer  `



## 项目功能

-- 登录/注销

-- 动态路由

-- 页面权限

-- 开发和生产环境

-- menu导航菜单

-- table表

    -- 固定表头
    -- 动态表头

-- echarts图表

    -- 地图(配合高德api获取行政区边界，点击下钻)
    -- 饼图
    -- 柱状图
    -- 折线图
    -- 水球图

-- vue-count-to 数字滚动

-- dirver.js 引导页

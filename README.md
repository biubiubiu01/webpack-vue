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
  babel-loader @babel/preset-env @babel/core @babel/polyfill core-js@2 @babel/plugin-transform-runtime
  @babel/runtime  @babel/runtime-corejs2

* vue:     
  vue-loader vue-template-compiler vue-style-loader vue vue-router axios vuex

* webpack: 

  file-loader url-loader webpack-dev-server webpack-merge copy-webpack-plugin happypack HardSourceWebpackPlugin 
  webpack-bundle-analyzer optimize-css-assets-webpack-plugin  portfinder  FriendlyErrorsPlugin

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

-- 拷贝静态文件

-- 热更新

-- 区分当前环境

-- 多线程打包

-- 缓存未改变模块

-- g-zip压缩

-- 获取本机ip

-- 打包大小分析

-- 压缩css

-- 检查端口是否存在


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

5.我们一般把不需要打包的静态文件放在public里面,这个地方的文件不会被打包的dist里面，所以我们需要一个插件来把这些文件复制过去

npm i -D  copy-webpack-plugin

webpack.config.js

const CopyWebpackPlugin = require('copy-webpack-plugin')      // 复制文件

plugins: [
   new CopyWebpackPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, '../public'),
          to: path.resolve(__dirname, '../dist')
        }
      ]
    })
]


```

6.为了让webpack识别css,我们需要安装loader，并将解析后的css插入到index.html里面的style

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

7.我们这里可以使用预编译器更好的处理css，我这里使用的是sass

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

8.自动添加浏览器前缀

先安装：

` npm i -D postcss-loader autoprefixer  `

```
  webpakc.config.js

  module: {
      rules: [
        {
          test: /\.scss$/,
          use: ['style-loader', 'css-loader', 'postcss-loader', 'sass-loader'] // 从右向左解析原则   sass-loader必须写后面
      }]
  }

 在根目录新建  .browserslistrc

 这个里面配置兼容的浏览器

 这里贴下我的默认配置，可以根据实际情况自己去配置

 > 1%
last 2 versions
not ie <= 8

再新建一个postcss.config.js

module.exports = {
  plugins: [require('autoprefixer')]  // 引用该插件即可了
}

这样打包后就自动帮你添加浏览器前缀了

```

9.之前的style-loader只是把css打包到index.html里面的style-loader里面,如果css特别多这种办法肯定不行，所以我们需要单独抽离提取css

先安装:

`npm i -D mini-css-extract-plugin`

```
  webpack.config.js

const MiniCssExtractPlugin = require("mini-css-extract-plugin")   //提取css

module: {
    rules: [{
          test: /\.scss$/,
          use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader'] // 从右向左解析原则
    }]
}

plugins: [
    new MiniCssExtractPlugin({
        filename: "css/[name].[hash:8].css",
        chunkFilename: "[id].css",
    })
]

这样打包后就将css打包到css文件下里面了

```

10.我们为了减少图片字体等打包的大小，我们可以使用url-loader将少于指定大小的文件转换为base64,使用file-loader将大于指定大小的文件
  移动到指定的位置

  先安装:
  
`npm i -D file-loader url-loader`

```
  webpack.config.js

 module:{
    rules:[
      {
        test:/\.(jpe?g|png|gif|svg)(\?.*)?$/            //图片
        use:[{
          loader:'url-loader',
          options:{
            limit:10240,
            fallback:{
              loader:'file-loader',
              options:{
                name:'img/[name].[hash:8].[ext]',
                publicPath: '../'       //为了防止图片路径错误
              }
            }
          }
        }]
      },{
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        use: [{
          loader: 'url-loader',
          options: {
            limit: 10240,
            fallback: {
              loader: 'file-loader',
              options: {
                name: 'media/[name].[hash:8].[ext]',
                publicPath: '../'
              }
            }
          }
        }]
      },{
        test:/\.(woff2?|eot|ttf|otf)(\?.*)$/,
        loader:'url-loader',
        options:{
          limit:10240,
          fallback:{
           loader:'file-loader,
           options:{
             name:'fonts/[name].[hash:8].[ext]',
             publicPath: '../'
           }
        }
        }
      }
    ]
  }

webpack只会转换和移动项目中使用了的图片

打包后发现图片字体等也打包进去了

```

11.为了兼容浏览器，我们需要将ES6,7,8,9转换为Es5

先安装：

 npm install -D  babel-loader @babel/preset-env @babel/core

```
 webpack.config.js

module: {
    rules: [{
        test: /\.js$/,
        use: ['babel-loader']
    }]
}

根目录新建  .babelrc

{
  "presets": [
    "@babel/preset-env"
  ]
}


一些新的api如Promise,set,Maps等还不支持转换，所以我们需要另一个插件babel/polyfill,但是这个插件会将所有的poly都打包到
mian.js里面，所以我们需要另一个插件 core-js@2  来按需加载

`npm i -s @babel/polyfill core-js@2`

修改.babelrc

{
  "presets": [
    [
      "@babel/preset-env",
      {
        "modules": false,
        "useBuiltIns": "usage",
        "corejs": 2,
        "targets": {
          "browsers": [
            "last 2 versions",
            "ie >= 10"
          ]
        }
      }
    ]
  ]
}

还有一个问题，会影响全局依赖,所以我们需要另一个插件来解决这个问题

 npm i @babel/plugin-transform-runtime -D
 npm i --save @babel/runtime
 npm i --save @babel/runtime-corejs2

修改.babelrc

{
  "presets": [
    "@babel/preset-env"
  ],
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "helpers": true,
        "regenerator": true,
        "useESModules": false,
        "corejs": 2
      }
    ]
  ]
}

这样就完美解决ES6新api的兼容问题了

```

12.兼容vue

```
  先安装：

  npm i -D vue-loader vue-template-compiler vue-style-loader

  npm i -S vue 

  webpack.config.js

const vueLoaderPlugin=require('vue-loader/lib/plugin') 
function resolve(dir) {
  return path.join(__dirname, '..', dir)
}

 module:{
    rules:[{
        test:/\.vue$/,
        use:[vue-loader]
    }]
 },
resolve:{
    alias:{
        'vue$':'vue/dist/vue.runtime.esm.js',
        '@':path.resolve(__dirname,'../src')
    },
    extensions: ['.js', '.vue', '.json'],
},
plugins:[
   new vueLoaderPlugin()
]

这样配置完成之后就可以webpack就识别了vue文件

```

13.热更新

先安装：

```
  npm i -D webpack-dev-server

  wepack.config.js

const webpack = require('webpack')

devServer: {
    compress: true,
    port: 8989,
    hot: true,
    inline: true,
    hotOnly: true,  //当编译失败时，不刷新页面
    overlay: true,  //用来在编译出错的时候，在浏览器页面上显示错误
    publicPath: '/',  //一定要加
    open: true,
    watchOptions: {
      // 不监听的文件或文件夹，支持正则匹配
      ignored: /node_modules/,
      // 监听到变化后等1s再去执行动作
      aggregateTimeout: 1000,
      // 默认每秒询问1000次
      poll: 1000
    }
  },
plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NamedModulesPlugin(),
    new webpack.NoEmitOnErrorsPlugin()
]

在package.json里面配置： "dev":"webpack-dev-server --config build/webpack.config.js"

在main.js里面

import Vue from "vue"
import App from "./App"

new Vue({
    render:h=>h(App)
}).$mount('#app')

src文件夹内新建一个APP.vue，内容自定义

然后npm run dev  页面就运行起来了

```

14.区分开发环境和生产环境

```
build文件夹内新建 webpack.dev.js  webpack.prod.js

开发环境：

1.不需要压缩代码
2.热更新
3.完整的soureMap
...

生产环境：

1.压缩代码
2.提取css文件
3.去除soureMap (根据个人需要)
...

我们这里需要安装 webpack-merge   合并配置项

npm i -D webpack-merge   

webpack.dev.js

const webpackConfig = require('./webpack.config')
const merge = require('webpack-merge')
const webpack = require('webpack')

module.exports = merge(webpackConfig, {
  mode: 'development',
  devtool: 'cheap-module-eval-source-map',
  devServer: {
    compress: true,
    port: 8989,
    hot: true,
    inline: true,
    hotOnly: true,  //当编译失败时，不刷新页面
    overlay: true,  //用来在编译出错的时候，在浏览器页面上显示错误
    publicPath: '/',  //一定要加
    open: true,
    watchOptions: {
      // 不监听的文件或文件夹，支持正则匹配
      ignored: /node_modules/,
      // 监听到变化后等1s再去执行动作
      aggregateTimeout: 1000,
      // 默认每秒询问1000次
      poll: 1000
    }
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ['babel-loader']
      },
      {
        test: /\.css$/,
        use: ['vue-style-loader', 'css-loader', 'postcss-loader'],
      },
      {
        test: /\.scss$/,
        use: ['vue-style-loader', 'css-loader', 'postcss-loader', 'sass-loader'],
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NamedModulesPlugin(),
    new webpack.NoEmitOnErrorsPlugin()
  ]
})


   webpack.prod.js

const webpackConfig = require('./webpack.config')
const merge = require('webpack-merge')
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')     //清除dist
const MiniCssExtractPlugin = require("mini-css-extract-plugin")   //提取css

function resolve(dir) {
  return path.join(__dirname, '..', dir)
}

module.exports = merge(webpackConfig, {
  mode: "production",
  devtool: 'none',
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendors: {
          name: 'vendors',
          test: /[\\\/]node_modules[\\\/]/,
          priority: -10,
          chunks: 'initial'
        },
        common: {
          name: 'chunk-common',
          minChunks: 2,
          priority: -20,
          chunks: 'initial',
          reuseExistingChunk: true
        }
      }
    }
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ['babel-loader']
        exclude: /node_modules/,
        include: [resolve('src'), resolve('node_modules/webpack-dev-server/client')]
      },
      {
        test: /\.css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: '../',
            }
          }, 'css-loader', 'postcss-loader'],
      },
      {
        test: /\.scss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: '../',
            }
          }, 'css-loader', 'postcss-loader', 'sass-loader'],
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new MiniCssExtractPlugin({
      filename: 'css/[name].[hash].css',
      chunkFilename: 'css/[name].[hash].css',
    })
  ]
})


  webpack.config.js

const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')   //这里引入插件
const vueLoaderPlugin = require('vue-loader/lib/plugin')
const CopyWebpackPlugin = require('copy-webpack-plugin')      // 复制文件

function resolve(dir) {
  return path.join(__dirname, '..', dir)
}

module.exports = {
  mode: 'development',
  entry: path.resolve(__dirname, '../src/main.js'),
  output: {
    filename: 'js/[name].[hash:8].js',
    path: path.resolve(__dirname, '../dist'),
    chunkFilename: 'js/[name].[hash:8].js',  //异步加载模块
    publicPath: './'
  },
  externals: {},
  module: {
    noParse: /jquery/,
    rules: [
      {
        test: /\.vue$/,
        use: [{
          loader: 'vue-loader',
          options: {
            compilerOptions: {
              preserveWhitespace: false
            }
          }
        }]
      },
      {
        test: /\.(jpe?g|png|gif)$/i, //图片文件
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10240,
              fallback: {
                loader: 'file-loader',
                options: {
                  name: 'img/[name].[hash:8].[ext]',
                  publicPath: '../'
                }
              }
            }
          }
        ]
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/, //媒体文件
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10240,
              fallback: {
                loader: 'file-loader',
                options: {
                  name: 'media/[name].[hash:8].[ext]',
                  publicPath: '../'
                }
              }
            }
          }
        ]
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/i, // 字体
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10240,
              fallback: {
                loader: 'file-loader',
                options: {
                  name: 'font/[name].[hash:8].[ext]',
                  publicPath: '../'
                }
              }
            }
          }
        ]
      }
    ]
  },
  resolve: {
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
    },
    extensions: ['.js', '.vue', '.json'],
  },
  //插件注入
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, '../public/index.html')
    }),
    new vueLoaderPlugin(),
    new CopyWebpackPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, '../public'),
          to: path.resolve(__dirname, '../dist')
        }
      ]
    })
  ]
}


```


### 3.webpack配置优化

1.设置mode

默认为production，webpack4.x默认会压缩代码和去除无用的代码  

可选参数：production, development

ps：之前我认为只需要设置mode为production,就不用使用压缩css和js的插件，但结果发现我错了，仔细比较了下，还是要安装的

先安装打包css的：

npm i -D  optimize-css-assets-webpack-plugin  

```
webpack.prod.js

const optimizeCss = require('optimize-css-assets-webpack-plugin');

plugins:[
  new optimizeCss({
      cssProcessor: require('cssnano'), //引入cssnano配置压缩选项
      cssProcessorOptions: {
        discardComments: { removeAll: true }
      },
      canPrint: true //是否将插件信息打印到控制台
  })
]
```

压缩js和js打包多线程的暂时没有添加，在网上搜有的说不用添加，有的说还是要安装插件，等实际项目中我用完之后再来添加


2.缩小搜索范围

```
 alias 可以告诉webpack去指定文件夹去寻找，尽量多使用

 include exclude    包括和过滤

 noParse   当我们代码中使用到import jq from 'jquery'时，webpack会去解析jq这个库是否有依赖其他的包。这个可以告诉webpack不必解析

 extensions    使用频率高的写在前面  

```
3.单线程转多线程

webpack处理文本，图片,css的时候，由于js单线程的特性，只能一个一个文件的处理,HappyPack可以将这个任务
分解到多个子线程里面，子线程完毕后会把结果发送到主线程，从而加快打包速度

先安装:

`npm i -D happypack`

```
webpack.prod.js

const HappyPack = require('happypack')     //单进程转多进程
const os = require('os')
const happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length })

module: {
    rules: [{
        test: /\.js$/,
        use: ['happypack/loader?id=happyBabel'],
        exclude: /node_modules/,
        include: [resolve('src'), resolve('node_modules/webpack-dev-server/client')]
    }]
}

plugins:[
   new HappyPack({
      id: 'happyBabel',
      loaders: ['babel-loader?cacheDirectory'],
      threadPool: happyThreadPool
    })
]

```

4.第三方模块优化

将不怎么改变的第三方依赖，我们可以用DllPlugin DllReferencePlugin将它从依赖中抽离出来，这样每一次打包就不用打包这些文件，加快了打包的速度;

但是webpack4.x的性能已经很好了，参考vue-cli也没有使用dll抽离，所以我们这里也不使用了，这里我们使用
另一个插件：hard-source-webpack-plugin ，这个插件会去对比修改了哪些配置，只去打包修改过了的配置
第一次打包速度正常，第二次打包速度能提升 50%+

```
npm i -D hard-source-webpack-plugin

webpack.prod.js

const HardSourceWebpackPlugin = require('hard-source-webpack-plugin')    //缓存第三方模块
plugins: [
   new HardSourceWebpackPlugin()
]

```

5.externals

通过cdn加载的依赖，可以在这里设置，就不会通过webpack编译

6.g-zip压缩

g-zip压缩可以将已经压缩过的js，css再次压缩一遍，减少了打包大小，需要nginx配置

```
npm i -D compression-webpack-plugin

webpack.prod.js

const CompressionWebpackPlugin = require('compression-webpack-plugin')
const productionGzipExtensions = ["js", "css"];

plugins:[
   new CompressionWebpackPlugin({
      filename: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp("\\.(" + productionGzipExtensions.join("|") + ")$"),
      threshold: 10240, // 只有大小大于10k的资源会被处理
      minRatio: 0.6 // 压缩比例，值为0 ~ 1
    })
]

```

7.自动获取本地Ip，通过本地ip地址启动项目

```
webpack.dev.js

const os = require('os')
devServer:{
  host:()=>{
      var netWork = os.networkInterfaces()
      var ip = ''
      for (var dev in netWork) {
          netWork[dev].forEach(function (details) {
              if (ip === '' && details.family === 'IPv4' && !details.internal) {
                 ip = details.address
                 return;
              }
          })
     }
        return ip || 'localhost'
   }
}

```

8.抽离一些公共配置

根目录新建vue.config.js  里面配置一些公共的配置如：

```
const os = require('os')

module.exports = {
  dev: {
    host: getNetworkIp(),   //端口号
    port: 8999,
    autoOpen: true,  //自动打开
  },
  build: {
    productionGzipExtensions: ["js", "css"],  //需要开启g-zip的文件后缀
    productionGzip: false     //是否开启g-zip压缩
  }
}

//获取本地ip地址
function getNetworkIp() {
  var netWork = os.networkInterfaces()
  var ip = ''
  for (var dev in netWork) {
    netWork[dev].forEach(function (details) {
      if (ip === '' && details.family === 'IPv4' && !details.internal) {
        ip = details.address
        return;
      }
    })
  }
  return ip || 'localhost'
}

然后webpack.dev.js  webpack.prod.js引入这个文件，获取其中的配置就好了

```

9.打包大小分析

先安装:

`npm i -D webpack-bundle-analyzer`

```
webpack.prod.js

if (process.env.npm_config_report) {
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  prodConfig.plugins.push(
    new BundleAnalyzerPlugin()
  )
}

然后 npm run build --report就会弹出一个页面，里面就是打包大小分析

```

10.完整的vue项目（vue-router axios vuex等）

先安装:

`npm i -S vue-router axios vuex `

然后在src里面新建 -> router文件夹 ->新建index.js

```
index.js

import Vue from "vue"
import Router from "vue-router"

Vue.use(Router)

export default new Router({
  mode: 'hash',
  routes: [
    {
      path: '/',
      name: 'home',
      component: () => import(/* webpackChunkName: "home" */ "@/views/home"),
    },
  ]
})

main.js

import router from "./router"
new Vue({
  router,
  render: h => h(App)
}).$mount('#app')

新建views -> Home.vue

随便写点东西，然后npm run dev  这样就完成了一个路由了

```


到这里webpack搭建vue项目就搭建完成了，我们接下来就用这个webpack-vue去写项目

如果觉得有帮助，那就点个 Star 吧~

未完待续...

npm install   安装依赖

npm run dev 运行

npm run build  打包

npm run build --report  打包大小分析

npm run test   运行测试





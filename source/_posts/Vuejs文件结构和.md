---
title: webpack和vue配合使用
date: 2019-11-18 12:33:48
tags:
- vuejs
- webpack
categories:
- frontend
preview: 100
---



## 目录结构

```
├── dist  					# 打包后的资源
│   ├── bundle.js  	# 打包后的js文件
│   └── img					
├── index.html			# 最外层文件
├── node_modules		# 模块
├── src							# 源代码，在此编写代码
│   ├── css
│   ├── img
│   ├── js
│   ├── main.js			# 
│   └── vue					# 存放vue组件文件
└── webpack.config.js	# 配置
```

<!-- more -->

## 代码示例



```javascript
import Vue from 'vue'
import App from './app.vue'

new Vue({
    el: '#app',
    template: '<App/>',
    components: {
        App,
    }
})
```

- vue实例中不要写太多代码，把代码都写到`*.vue`文件中，以模块的方式导入
- 实例化一个vue对象时，vue会将el选择的节点替换为template的内容
- [组件注册]([https://cn.vuejs.org/v2/guide/components-registration.html#%E5%B1%80%E9%83%A8%E6%B3%A8%E5%86%8C](https://cn.vuejs.org/v2/guide/components-registration.html#局部注册))

```javascript
<template>
    <div>
        <h2>{{ message }}</h2>
        <button class="btn" @click="btnClick">按钮</button>
    </div>
</template>

<script>
    export default {
        data: function() {
            return {
                message: 'hello webpack',
            }
        },
        methods: {
            btnClick: function() {
                this.message = "changed";
            }
        }
    }
</script>  
<style>
    body {
        background: url('../img/timg.jpeg');
    }
    .btn {
        width: 50px;
        height: 30px;
        background: red;
    }
</style>
```

- script标签里的`export default`是es6的语法，导出默认对象，这样再其他文件中import的时候就不需要知道原模块定义的变量名

- [ES6模块](https://www.runoob.com/w3cnote/es6-module.html)

  

## 插件的使用
1. npm 下载
2. webpack.config.js 配置
### 示例1
打包代码到dist文件夹后，index.html不在文件夹内，那么打包的js文件也没有意义了，现在用`HtmlWebpackPlugin`插件解决这个问题
1. `npm install html-webpack-plugin --save-dev `
2. 修改`webpack.config.js`中plugin部分如下
```javascript
var HtmlWebpackPlugin = require('html-webpack-plugin');
...
plugins: [
  new HtmlWebpackPlugin({
    template: 'index.html'
  }),
]
```

- template指明根据什么模板来生成index.html

[HtmlWebpackPlugin](https://webpack.docschina.org/plugins/html-webpack-plugin/)

### 示例2

使用 `UglifyjsWebpackPlugin`将打包后的代码进行压缩

1. `$ npm install uglifyjs-webpack-plugin --save-dev`
2. 修改`webpack.config.js`中plugin部分如下

```javascript
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

module.exports = {
    plugins: [
        new UglifyJsPlugin(),
    ],
};
```

[UglifyjsWebpackPlugin](https://webpack.docschina.org/plugins/uglifyjs-webpack-plugin/)



## 开启本地服务器

不用每次写完代码都npm run build，让页面实时刷新

1. `npm install --save-dev webpack-dev-server`

```javascript
// webpack.config.js

module.exports = {
  ... 
  devServer: {
      contentBase: './dist',		// 把这个文件夹作为
      inline: true,							// 实时更新
    }
}
```

2. 输入`./node_modules/.bin/webpack-dev-server`即可开启，但是这样太麻烦，可以在`package.json `里配置scripts

   ```json
     "scripts": {
   		
       "dev": "webpack-dev-server --open"
     },
   ```

   - --open 表示用浏览器打开

## 配置文件的分离

上一个例子中为了实现本地开发服务器配置了一些东西，但是在实际编译的时候不需要，压缩代码的插件UglifyJsPlugin在编译的时候才用到，所以要分离配置文件

1. `npm install --save-dev webpack-merge`

2. 根目录新建一个build目录, 新建三个文件 base.config.js dev.config.js prod.config.js 分别存放公共部分配置，开发配置，生产配置

   ```javascript
   // base.config.js
   const path = require('path');
   const VueLoaderPlugin = require('vue-loader/lib/plugin');
   var HtmlWebpackPlugin = require('html-webpack-plugin');
   
   module.exports = {
       entry: './src/main.js',
       output: {
           path: path.resolve(__dirname, '../dist'),
           filename: 'bundle.js',
       },
       module: {
           rules: [
             {
               test: /\.css$/,
               use: ['style-loader', 'css-loader' ]
             },
             {
               test: /\.(png|jpg|gif|jpeg)$/,
               use: [
                 {
                   loader: 'url-loader',
                   options: {
                       limit: 8192,
                       name: 'img/[name].[hash:8].[ext]',
                   }
                 }
               ]
             },
             {
               test: /\.m?js$/,
               exclude: /(node_modules|bower_components)/,
               use: {
                 loader: 'babel-loader',
                 options: {
                   presets: ['@babel/preset-env']
                 }
               }
             },
             {
               test: /\.vue$/,
               loader: 'vue-loader'
             },
           ]
         },
       plugins: [
           new VueLoaderPlugin(),
           new HtmlWebpackPlugin({
             template: 'index.html'
           }),
       ],
       resolve: {
           alias: {
               'vue$': 'vue/dist/vue.esm.js'
           }
       },
   
   };
   ```

   ```javascript
   // dev.config.js
   const webpackMerge = require('webpack-merge')
   const baseConfig = require('./base.config')
   
   module.exports = webpackMerge(baseConfig, {
       devServer: {
           contentBase: './dist',
           inline: true,
       }
   })
   
   ```

   ```javascript
   // prod.config.js
   const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
   const baseConfig = require('./base.config')
   const webpackMerge = require('webpack-merge')
   
   
   module.exports = webpackMerge(baseConfig, {
       plugins: [
           new UglifyJsPlugin(),
       ]
   })
   ```

   - 记得修改base.config.js力path中的dist为../dist

## 代码

[github](https://github.com/baichen99/webpack_test)




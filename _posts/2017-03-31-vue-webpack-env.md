---
layout: post
title:  构造vue2.0 + webpack2环境
date:   2017-03-31 21:38:00 +0800
categories: 前端工具
tag: 前端工具
---

* content
{:toc}

前面vue2.0和webpack都已经有接触了些，现在开始学习如何构造自己的vue2.0 + vuex + vue-router + webpack环境。

### 1. 开始

这就不写了，直接放[地址](https://github.com/peiyanhuang/MyMusicPlayer)

npm install 安装 'package.json' 中的依赖项。

### 2. 构建 webpack.config.js 

上代码：

```
var webpack = require("webpack"); 
var path = require("path");
var HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
	devtool: 'cheap-module-eval-source-map',
	entry: {
		index: path.join(__dirname + "/app/index.js"),
		vendor: ['vue', 'vue-router', 'vuex']
	},

	output: {
		path: path.join(__dirname + "/output"),
		publicPath: '/',
		filename: '[name].[hash].js',
		chunkFilename: '[id].chunk.js',
	},
	resolve: {
		extensions: ['.js','.es6','.vue','.css'],
		alias: {
			'vue$': 'vue/dist/vue.common.js',
			'vue-router$': 'vue-router/dist/vue-router.common.js',
		}
	},
	module: {
	    rules: [
	      	{
	      		test: /\.css$/, 
	      		use: ['css-loader', 'style.loader'],
	      	},
	      	{
	      		test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
	      		use: 'url-loader?limit=10240'
	      	},
	      	{
	        	test: /\.js$/,
	        	exclude: /node_modules/,
	        	use: [{
	        		loader: 'babel-loader',
	        		options: {
	          			presets: ['es2015']
	        		}
	        	}]
	      	},
	      	{
	      		test: /\.vue?/,
	      		use: 'vue-loader',
	      	}
	    ]
	},
	plugins: [
		//压缩插件
	    new webpack.optimize.UglifyJsPlugin({
	    	sourceMap: true,
	      	compress: {
	        	warnings: false
	      	}
	    }),
	    //提取公共脚本
	    new webpack.optimize.CommonsChunkPlugin({
	    	name: 'vendor', 
	    	filename: 'bound.js'
	    }),
	    //自动生成html
	    new HtmlWebpackPlugin({
	    	filename: './index.html',
            template: path.join(__dirname + '/app/index.html'),
            inject: true
	    })
  	],
};
```

简单的 webpack.config.js 配置就完成了，接下来开始添加其他的内容。

### 3. Hot Module Replacement

每次都需要运行构建命令才能查看改变后的代码效果，这是很没有效率，于是还需要安装  webpack-dev-middleware 中间件和 webpack-hot-middleware 中间件。

先来添加 webpack-dev-middleware 中间件。

在build目录中创建一个dev-server.js文件，并写入内容：

```
// 引入必要的模块
const express = require('express');
const webpack = require('webpack');
const webpackDevMiddleware = require('webpack-dev-middleware');

const config = require('./webpack.config');

// 创建一个express实例
var app = express();

// 调用webpack并把配置传递过去
var compiler = webpack(config);

// 使用 webpack-dev-middleware 中间件
var devMiddleware = webpackDevMiddleware(compiler, {
    publicPath: config.output.publicPath,
    stats: {
        colors: true,
        chunks: false
    }
});
app.use(devMiddleware);

app.use(express.static(__dirname));

// 监听 8888端口，开启服务器
app.listen(8888, function (err) {
    if (err) {
        console.log(err);
        return;
    }
    console.log('Listening at http://localhost:8888');
});
```

此时修改文件刷新就可以看到修改了。

每次都要手动刷新很烦，webpack-hot-middleware 中间件就来了。

第1步: 在build目录下新建一个webpack.dev.conf.js文件，意思是开发模式下要读取的配置文件，并写入一下内容：

```
/*
 *  配置 热重载
 */

var webpack = require('webpack');

// 引入基本配置
var config = require('../webpack.config');

config.plugins = config.plugins.concat([
	//new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin(),
]);

// 动态向入口配置中注入 webpack-hot-middleware/client

var devClient = 'webpack-hot-middleware/client';

Object.keys(config.entry).forEach(function (name, i) {
    var extras = [devClient];
    config.entry[name] = extras.concat(config.entry[name]);
});

module.exports = config;
```

第2步: 修改 dev-server.js 文件：

```
const config = require('./webpack.config');
替换为：
const config = require('./webpack.dev.conf');

添加：
const webpackHotMiddleware = require('webpack-hot-middleware');

//使用 webpack-hot-middleware 
var hotMiddleware = webpackHotMiddleware(compiler);
app.use(hotMiddleware);
```

到这里还没结束，这里只是监听到Favlist.vue文件的改动，为了能监听到index.html文件的改动，我们还需要做一些工作。

第1步: 在dev-server.js文件中监听html文件改变事件，修改 dev-server.js 文件：

```
添加：
// webpack插件，监听html文件改变事件
compiler.plugin('compilation', function (compilation) {
    compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
        // 发布事件
        hotMiddleware.publish({ action: 'reload' });
        cb();
    });
});
```

第2步: 修改webpack.dev.conf.js文件

```
var devClient = 'webpack-hot-middleware/client';
修改为：
var devClient = './build/dev-client';
```

第3步: 新建build/dev-client.js文件，并编辑如下内容：

```
/*
 *	配置html的热重载
 */

var hotClient = require('webpack-hot-middleware/client');

// 订阅事件，当 event.action === 'reload' 时执行页面刷新
hotClient.subscribe(function (event) {
    if (event.action === 'reload') {
        window.location.reload();
    }
});
```

至此，开发环境终于搞定了。哦忘了，还差个 CSS 单独打包

在 webpack.config.js 中添加：

```
var ExtractTextPlugin = require('extract-text-webpack-plugin');

const extractCSS = new ExtractTextPlugin('[name]-css.css');

//打包css、 less文件
module: {
    rules: [
      	{
      		test: /\.css$/, 
      		use: extractCSS.extract({
      			fallback: 'style-loader',
      			use: ['css-loader'],
      		})
      	}
    ]
}

//plugins 中加上 extractCSS,
```

再把 autoprefixer 也装上吧，开启 sourceMap，修改下

```
module: {
	rules: [
		{
			test: /\.css$/, 
			exclude: /node_modules/,
			use: extractCSS.extract({
				fallback: 'style-loader',
				use: [
					{
						loader: 'css-loader', 
						options: {
							sourceMap: true,
							importLoaders: 1
						}
					},
					{
						loader: 'postcss-loader',
						options: {
							sourceMap: 'inline'
						}
					}
				],
			})
		}
	]
}
```

然后创建 postcss.config.js:

```
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}
```

在 package.json 中添加：

```
"browserslist": [
    "> 1%",
    "last 2 versions",
    "not ie <= 8"
  ]
```

可以随你设置。

现在OK！

### ELSE

现在，来讲讲配置时的各种 坑 Ｏ(≧口≦)Ｏ ...

- Webpack has been initialised using a configuration object that does not match the API schema.

这是 webpack 的版本问题。我安装的 webpack 版本是 "webpack": "^2.3.2" 。

解决方法 1 ：按照 webpack2 的格式写 

```
modules: [
	{
		test: /\.js$/,
		exclude: /node_modules/,
		use: [{
			loader: 'babel-loader',
			options: {
				presets: ['es2015']
			}
		}]
	}
]
```

具体 1.x 和 2.x  的区别看这
[Migrating from v1 to v2](https://webpack.js.org/guides/migrating/)

[webpack2.2的中文文档看这](http://www.css88.com/doc/webpack2/)

解决方法 2 ：

安装着个版本的 webpack ："webpack": "^2.1.0-beta.22"，支持 1 的写法。

-  热重载所需的 new webpack.optimize.OccurenceOrderPlugin() 插件默认已经有了，可以不用引入了；new webpack.NoErrorsPlugin() 需替换为 new webpack.NoEmitOnErrorsPlugin()。

- 引入文件时 require() 和 module.exports 配对使用， import x from './tag' 和 export default 配对使用，不能混合。
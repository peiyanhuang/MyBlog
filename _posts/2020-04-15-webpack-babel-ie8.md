---
layout: post
title:  Webpack4+Babel7+ES6兼容IE8
date:   2020-04-15 19:00:00 +0800
categories: 前端工具
tag: 前端工具
---

* content
{:toc}

原文[Webpack4+Babel7+ES6兼容IE8]（https://juejin.im/post/5cabf7b0e51d456e8b07dd04?utm_medium=hao.caibaojian.com&utm_source=hao.caibaojian.com）

IE8 很老了大多数时候已经不需要考虑了，但有时候还是有兼容的要求。这边记录下如何实现兼容的。（之前没记录，差点忘了…………）

### 语法支持

IE 对 es6 的支持很差，要在 IE 下运行，需要把 es6 转换为 es5，这个直接用 `babel-loader` 就行了，也没什么坑，就不细说了。

### ES3保留关键字

如果在 IE8 下通过 `object.propertyName` 的方式使用ES3中的保留关键字（比如 `default`、`class`、`catch`），就会报错

```
SCRIPT1048: 缺少标识符
```

webpack 有一款 loader 插件 `es3ify-loader` 专门用来处理 ES3 的关键字兼容问题。这个插件的作用就是把这些保留字给你加上引号，使用字符串的形式引用。

```js
// 编译前
function(t) { return t.default; }

// 编译后
function(t) { return t["default"]; }
```

但我亲身实践后发现，`UglifyJS` 本来就已经提供了对IE浏览器的支持，不需要额外引入 `es3ify-loader`。webpack 默认的 UglifyJS 配置不支持ie8，需要手动配下。

```js
{
  mode: 'production',
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        uglifyOptions: {
          ie8: true
        }
      })
    ]
  }
}
```

### 执行环境

解决了前面两个问题只能保证语法上不报错，但使用ES6中的API（比如`Promise`）还是会报错。另外，IE8对ES5的API支持也很差，只支持了少量的API，有些API还只是支持部分功能（比如`Object.defineProperty`）。所以，要在IE8中完美运行ES6的代码，不仅需要填充ES6的API，还要填充ES5的API。

babel为此提供了两种解决方案：`@babel/polyfill`、`@babel/runtime`。具体使用方法官方文档已经写的很详细了，笔者就不赘述了。想了解两者之间的差别的同学可以看下大搜车墨白同学的文章，[babel-polyfill VS babel-runtime](https://juejin.im/post/5a96859a6fb9a063523e2591)

这里纠正墨白同学文中的一个错误，就是 `@babel/polyfill` 现在已经支持按需加载，准确的说也不能算是错误，因为墨白同学在写这篇文章的时候还不支持按需加载。具体方法我就不细说了，文档里都有，配置下 `browserlist` 和 `@babel/preset-env的useBuiltsIns` 属性就可以了。

我只说下我在实际开发过程中碰到的坑。

虽然 `@babel/polyfill`、`@babel/runtime` 都支持按需加载，但都只能识别出业务代码中使用到的缺失的API，如果第三方库有用到这些缺失的API，babel不能识别出来，自然也就不能填充进来。比如 `regenerator-runtime` 中用到的 `Object.create` 和 `Array.prototype.forEach`。解决办法是，在入口文件处手动引入缺失的API。

### 模块化加载

笔者原来是想用ES6的模块化加载方案，因为这样可以利用webpack的 tree shaking，移除冗余代码，使打包出来的文件体积更小。但在IE8下测试发现 `Object.defineProperty` 会报错'Accessors not supported!'。报错代码如下

```js
if ('get' in Attributes || 'set' in Attributes) throw TypeError('Accessors not supported!');
```

我用 `@babel/plugin-transform-modules-commonjs` 转成 commonjs 加载就可以把这个坑绕过去，但同时也意味着放弃了tree shaking。

### 总结

package.json

```json
{
  "devDependencies": {
    "@babel/core": "^7.2.2",
    "@babel/plugin-transform-runtime": "^7.2.0",
    "@babel/preset-env": "^7.1.0",
    "@babel/runtime": "^7.3.4",
    "babel-loader": "^8.0.4",
    "core-js": "^3.0.1",
    "uglifyjs-webpack-plugin": "^2.0.1",
    "webpack": "^4.20.2",
    "webpack-cli": "^3.1.2",
    "webpack-dev-server": "^3.1.9",
    "webpack-merge": "^4.1.4"
  }
}
```

webpack配置

```js
{
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-env'
            ],
            plugins: [
              [
                '@babel/plugin-transform-runtime'
              ],
              [
                // 为了兼容IE8才用了这个插件，代价是不能tree shaking
                // 没有IE8兼容需求的可以把这个插件去掉
                '@babel/plugin-transform-modules-commonjs'
              ]
            ]
          }
        }
      }
    ]
  },
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        sourceMap: true,
        uglifyOptions: {
          ie8: true,
        }
      })
    ]
  }
}
```

入口文件按需引入缺失的API

```js
require('core-js/features/object/define-property')
require('core-js/features/object/create')
require('core-js/features/object/assign')
require('core-js/features/array/for-each')
require('core-js/features/array/index-of')
require('core-js/features/function/bind')
require('core-js/features/promise')
```

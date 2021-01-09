![browserify-logo](https://zcd520.gitee.io/pics/browserify/logo.png)

【导语】Browserify 是一个 JavaScript 工具，它可以让你在浏览器中使用 `require('module')` 来加载模块。

## 简介

Browserify 是一个 JavaScript 工具，它可以让你像在 node 中那样，在浏览器中也可以使用 `require('module')` 来加载模块。换句话说，它可以让服务端的 CommonJs 的模块运行在浏览器端。

## 安装

如果你想在命令行中使用，可以直接全局安装
```
npm install -g browserify
```

## 基础使用

### 简单上手

我们先来看一个简单的例子。假设我们有一个文件 index.js，引入了一个外部模块 `uniq`：

首先我们安装这个模块：
```
npm install uniq
```

接着在 index.js 使用这个模块：
```js
const uniq  = require("uniq");

console.info(uniq([1,2,3,1,2,3]))
```

我们知道，这个 index.js 文件是可以直接在 node 环境下运行的，但是在浏览器上运行会有什么效果呢？

我们来试一下效果，在 index.html 中手动引入这个文件：

```html
<script src="./index.js"></script>
```

打开网页，我们可以看到控制台报错了：

![no-require-error](https://zcd520.gitee.io/pics/browserify/01-no-require-error.png)

其实很容易理解，我们根本没有定义这个 `require`，浏览器当然也不会认识它了。这时候我们的主角 Browserify 就派上用场了，我们在当前路径下执行如下命令：

```
browserify index.js > app.js
```

然后修改我们 index.html 引入的脚本为 app.js：

```html
<script src="./app.js"></script>
```

接着，我们刷新一下页面，可以看到正常显示了去重后的数组：
![hello browserify](https://zcd520.gitee.io/pics/browserify/02-hello-browserify.png)


### 外部依赖

如果你想在 script 标签中直接使用第三方模块，或者我们自己定义的模块，你可以使用 `--require` 选项参数，它的简写为 `-r`。

承接上例，假设我们的 index.js 导出了一个对象：

```js
module.exports = {
  name: "Jerry",
  techs: ["Vue", "React", "Webpack", "Vue"]
}
```

并且我们想在 index.html 中直接使用 uniq 模块和 index.js 导出的对象。此时我们需要修改我们的命令：

```
browserify -r uniq -r index.js:myModule > app.js
```

上述命令的意思是，将 uniq 打包成一个可 require 的模块，将 index.js 也打包成一个可 require 的模块，并且这个模块的名字叫 myModule。

接着在 index.html 中修改 script 标签：

```html
<script src="./app.js"></script>
<script>
  const uniq  = require("uniq");
  const myModule = require("myModule");
  console.info(`Got unique techs [${uniq(myModule.techs)}] from ${myModule.name}`)

</script>
```

![externals](https://zcd520.gitee.io/pics/browserify/03-externals.png)

可以看到，输出内容跟我们预期的一致。

### 生成 sourcemap

Browserify 也可以生成 sourcemap 方便我们开发中定位问题，它的使用很简单，只需要加上 `--debug` 选项即可（简写为 `-d`）。我们修改之前的命令：

```
browserify -d -r uniq -r index.js:myModule > app.js 
```

然后看一下生成的 app.js 文件：

![sourcemap](https://zcd520.gitee.io/pics/browserify/04-sourcemap.png)

可以看到在最后生成了 sourcemap 内容，在开发中可以方便的借助于它来进行调试了。

当然，如果你想将生成的 sourcemap 与我们的 js 文件分离开，可以借助于 `exorcist` 来实现（exorcist 原意为“驱魔人”，这里指代的是，将文件中的 sourcemap 部分的内容“驱赶”到另一个单独文件中去，更多用法请参考：[exorcist](https://github.com/thlorenz/exorcist)）：

修改我们的命令为：

```
browserify -d -r uniq -r index.js:myModule | npx exorcist ./app.js.map > ./app.js 
```

执行后你就会发现，你的 app.js 和 它对应的 sourcemap 文件已经分离了：

![external sourcemap](https://zcd520.gitee.io/pics/browserify/05-external-sourcemap.png)

### 多文件打包

当多个页面同时用到某一个相同的文件时，我们希望单独抽离出这个文件，再结合缓存机制，可以实现高效的加载。在 browserify 中我们可以通过 `--external`（简写 -x） 和 `--require`（简写 -r） 来实现。

比如我们此时有如下文件：

utils.js（公用的工具库）：

```js
function add(a, b) {
  return a + b;
}

function minus(a, b) {
  return a - b;
}

function upper(str) {
  return String(str).toLocaleUpperCase()
}

module.exports = {
  add,
  minus,
  upper
}
```

sum.js：

```js
const { add } = require("./utils")

console.info(add(1,2))
```

upper.js：

```js
const { upper } = require("./utils")

console.info(upper("hello"))
```

我们想将 utils.js 单独打包成一个文件，然后 sum.js 和 upper.js 的打包文件中不用包含这个 utils.js 的代码，而且直接 require 引入，我们需要如下三个命令：

```
// 单独打包 utils.js 文件为 common.js 文件
browserify -r utils.js -o ./lib/common.js

// 打包 sum.js，并且指定其使用的 utils.js 为外部模块（即不需要将这个 utils.js 打包进来）
browserify -x utils.js sum.js -o ./lib/sum.js

// 打包 upper.js，并且指定其使用的 utils.js 为外部模块（即不需要将这个 utils.js 打包进来）
browserify -x utils.js upper.js -o ./lib/upper.js
```

依次执行后会在当目录下生成 lib 文件夹，里面有三个打包后的文件：

lib/common.js：

```js
require=(function(){function r(e,n,t){function o(i,f){if(!n[i]){if(!e[i]){var c="function"==typeof require&&require;if(!f&&c)return c(i,!0);if(u)return u(i,!0);var a=new Error("Cannot find module '"+i+"'");throw a.code="MODULE_NOT_FOUND",a}var p=n[i]={exports:{}};e[i][0].call(p.exports,function(r){var n=e[i][1][r];return o(n||r)},p,p.exports,r,e,n,t)}return n[i].exports}for(var u="function"==typeof require&&require,i=0;i<t.length;i++)o(t[i]);return o}return r})()({"/src/03-multiple/utils.js":[function(require,module,exports){
function add(a, b) {
  return a + b;
}

function minus(a, b) {
  return a - b;
}

function upper(str) {
  return String(str).toLocaleUpperCase()
}

module.exports = {
  add,
  minus,
  upper
}
},{}]},{},[]);
```

lib/sum.js

```js
(function(){function r(e,n,t){function o(i,f){if(!n[i]){if(!e[i]){var c="function"==typeof require&&require;if(!f&&c)return c(i,!0);if(u)return u(i,!0);var a=new Error("Cannot find module '"+i+"'");throw a.code="MODULE_NOT_FOUND",a}var p=n[i]={exports:{}};e[i][0].call(p.exports,function(r){var n=e[i][1][r];return o(n||r)},p,p.exports,r,e,n,t)}return n[i].exports}for(var u="function"==typeof require&&require,i=0;i<t.length;i++)o(t[i]);return o}return r})()({1:[function(require,module,exports){
const { add } = require("./utils")

console.info(add(1,2))
},{"./utils":"/src/03-multiple/utils.js"}]},{},[1]);

```


lib/upper.js

```js
(function(){function r(e,n,t){function o(i,f){if(!n[i]){if(!e[i]){var c="function"==typeof require&&require;if(!f&&c)return c(i,!0);if(u)return u(i,!0);var a=new Error("Cannot find module '"+i+"'");throw a.code="MODULE_NOT_FOUND",a}var p=n[i]={exports:{}};e[i][0].call(p.exports,function(r){var n=e[i][1][r];return o(n||r)},p,p.exports,r,e,n,t)}return n[i].exports}for(var u="function"==typeof require&&require,i=0;i<t.length;i++)o(t[i]);return o}return r})()({1:[function(require,module,exports){
const { upper } = require("./utils")

console.info(upper("hello"))
},{"./utils":"/src/03-multiple/utils.js"}]},{},[1]);

```

可以看到，common.js 中包含着公有的方法，sum.js 和 upper.js 中只包含着自己的业务逻辑，而外部模块的使用是通过 require 来引入的。我们来验证一下是否可以运行：

新建一个 sum.html 文件，引入：

```html
<script src="./lib/common.js"></script>
<script src="./lib/sum.js"></script>
```

运行结果如下图，可以看到正确的输出了结果。

![multiple bundles](https://zcd520.gitee.io/pics/browserify/06-multiple-bundles.png)

### 封装 npm scripts

在开发过程中，我们通常会将一些常用的构建命令封装成 npm 脚本去执行，这样可以减小我们输错脚本内容的几率，我们在 package.json 文件中增加 scripts

```json
{
  "scripts": {
    "build": "browserify index.js > app.js"
  }
}
```

接着我们执行 `npm run build` 就可以完成同样的任务了，这也是最佳实践的一种。



## 进阶用法

### 使用 ES Module

如果你使用的是 import 或者 export 这样的 ES6 的模块化语言来引入其他模块的话，browserify 也提供了插件支持这样的语法。我们需要安装如下依赖：

```
npm install --save-dev @babel/core @babel/preset-env babelify
```

然后添加 babel.config.json 配置文件来配置 babel：

```js
{
  "presets": ["@babel/preset-env"]
}
```

假设此时我们的 index.js 文件如下：

```js
import uniq from "uniq";

console.info(uniq([1,2,3,1,2,3]))
```

那么要像让 browserify 正确的打包，需要如下命令：

```
browserify index.js > app.es.js -t babelify
```

### 代码压缩

假设我们的 index.js 中出现了没有使用的变量或者方法，这些代码永远不会被执行，那么打包进最终的生成文件无疑是一种浪费。browserify 也提供了这样的插件来优化我们的生成代码，比如：删除未使用的变量，删除无效的代码，压缩代码等。

我们需要安装这个插件

```
npm install --save-dev tinyify
```

假设我们的 index.js 是这样的：

```js
const name = 'Hello';
const version = 1.1;

function add(a, b) {
  return a + b;
}

function upper(str) {
  return str.toUpperCase()
}

console.info(add(name, version))
```

首先我们看一下不使用插件的打包输出：

```
browserify index.js > app.big.js
```

生成的 app.big.js 文件内容：

```js
(function(){function r(e,n,t){function o(i,f){if(!n[i]){if(!e[i]){var c="function"==typeof require&&require;if(!f&&c)return c(i,!0);if(u)return u(i,!0);var a=new Error("Cannot find module '"+i+"'");throw a.code="MODULE_NOT_FOUND",a}var p=n[i]={exports:{}};e[i][0].call(p.exports,function(r){var n=e[i][1][r];return o(n||r)},p,p.exports,r,e,n,t)}return n[i].exports}for(var u="function"==typeof require&&require,i=0;i<t.length;i++)o(t[i]);return o}return r})()({1:[function(require,module,exports){
const name = 'Hello';
const version = 1.1;

function add(a, b) {
  return a + b;
}

function upper(str) {
  return str.toUpperCase()
}

console.info(add(name, version))


},{}]},{},[1]);
```

我们可以看到，index.js 中的代码原样的被包裹在匿名函数中。并且我们的源代码中没有使用其他模块，但是打包后的文件还是模块引入之类的代码打包进来了。
我们使用 tinyify 插件再看看：

```
browserify index.js > app.big.js -p tinyfiy
```

生成的 app.small.js 文件内容如下：

```js
console.info("Hello1.1");
```

对，你没有看错，就是这么一行代码！简单粗暴！其实读者回过头来看一下 index.js 的代码，无非也就是这么一行输出代码而已。所以，借助于 tinyify 插件，我们可以有效的减少打包文件的体积，提升程序加载速度与运行效率。

### 监听文件修改

在开发过程中，我们不希望每次修改文件都要去手动的执行构建脚本，所以需要这样一个工具来监听文件的修改，一有变动立即执行构建逻辑。我们可以使用 watchify 来实现这样的效果。首先安装这个模块：

```
npm install watchify
```

接着只要运行如下的命令即可：

```
watchify index.js -o app.js
```

当我们的 index.js 文件修改时，app.js 就会被重新生成，刷新浏览器后就可以看到最新的结果了。

我们可以将上述三个脚本封装成 npm scripts 来使用：

```json
{
  "scripts": {
    "dev": "watchify index.js -o app.js",
    "build": "browserify index.js -o app.js -p tinyify",
    "build:es": "browserify index.js > app.es.js -t babelify -p tinify",
  }
}
```

好了，关于 browserify 的使用就介绍到这里，如果想了解更多，可以去翻阅它的[官网](http://browserify.org/)掌握更多用法。
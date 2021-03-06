# WEBPACK的模块热替换
模块热替换(HMR) 任然是一个实验特性。  

## 目录
[TOC]

##翻译说明
原文：<https://webpack.github.io/docs/hot-module-replacement-with-webpack.html>  
如果有人看到翻译中的错误希望有人告诉我。  
希望有人能帮我校对。  

## 引言
模块热替换(HMR) 在一个应用运行的时候更改、添加或者移除模块__无需__重新加载页面。

### 你需要了解
- 使用插件: <http://webpack.github.io/docs/using-plugins.html>
- 代码分割: <http://webpack.github.io/docs/code-splitting.html>
- webpack-dev-server: <http://webpack.github.io/docs/webpack-dev-server.html>

### 它是怎么工作的？
在编译过程中，Webpacks 添加了一个小的模块热替换 (HMR) 运行环境在你的应用内部运行。Webpack 在编译完成时不会退出，它将继续监视源文件的改动。如果 Webpack 检测到文件改动，它仅仅会编译有更新的模块。 根据配置，Webpack 将会发送一个信号到 HMR 运行环境，或者 HMR 运行环境轮询 Webpack 获取变更。 无论哪种方式，变更的模块被传到 HMR 运行环境，然后尝试热更新应用。首先 HMR 运行环境会检擦更新的模块是否可以调用`.accept()`接受更新。如果不能，继续检查哪些模块`require`了更新的模块。如果那些模块也不能通过`.accept()`更新，那么继续冒泡向上检查`require`了那些模块（`require`了更新过的模块的模块）的模块。HMR 运行环境会向上冒泡检查直到到模块可以调用`.accept()`更新。或者冒泡到应用的入口，热更新失败。

#### 你的应用要做什么？
你的应用要通知 HMR 运行环境检查更新。HMR 运行环境下载更新（异步），并告诉应用有可用更新。应用通知 HMR 运行环境进行更新。 HMR 运行环境执行更新（同步）。 这个过程，你可以决定是否需要通过你的应用界面交互来执行。  


#### webpack 编译器做了什么
除了一般资源，编译器需要发出“更新”让之前的版本更新到当前版本。“更新”包含两个部分：  
>1. 更新清单 （json）  
2. 一个或多个更新的块 （js）    

清单里包含了新的编译哈希值和一个所有更新块的列表（2.）。  
更新块包含所有更新过的模块的代码（或者是一个模块移除标识）。  

编译器能够确保模块和编译的块的ids之间保持对应。编译器使用一个 “records” json 文件来保存他们之间的对应关系（或者保存在内存里）。

_注： chunks 暂翻译为“块”_

#### 模块做了什么  
HMR 是一个可选特性，因此它只会影响包含了 HMR 代码的模块。[这篇文档](https://webpack.github.io/docs/)描述了模块可以使用的 API 。一般情况下，模块开发者应当编写一个当模块的依赖更新时被调用的处理器。或者也可以写一个模块自身被更新时调用的处理器。  

大多数情况下，并不强制要为每个模块添加 HMR 代码。如果一个模块没有 HRM 处理器，更新会向上冒泡。这意味着单个处理器能处理一个完整的模块树的更新，如果模块树里的一个模块更新了，整个模块树将重新加载（不会重新编译）。  

#### HMR 运行环境做了什么
运行环境为模块系统附加代码来跟踪模块的`parents`和`children`。  

运行环境在管理端支持两个方法：`check`和`apply`。

`check`方法发送一个 HTTP 请求获取更新清单。如果求失败时，就没有可用更新。否则会对更新的块的列表和当前加载的块的列表进行对比，为每个块下载好相应的更新块。所有的模块的更新存放进运行环境当中。运行环境切换到`ready`状态，这意味着一个更新已经被下载，准备好被应用。

在`ready`状态每个新添加的块也已经被下载。

`apply`方法标记所有不能更新的模块。不能更新的无效模块必须更新他们的处理器或者他们的父模块的处理器。无效模块被打包，他们的父模块也被标记为不能更新的无效模块。这个过程持续“冒泡”直到模块可以更新。如果冒泡到入口，那么这个过程失败。  

如果现在所有的无效模块已经部署了处理器或无效模块移除了。那么当前的哈希值更新，所有的 "accept" 处理器被调用。运行环境切换回`idle`状态，一切都照常运行。  

#### 块文件生成

左边标表示初始编译。右边表示模块4 和 9 更新.

![Generated files](http://webpack.github.io/assets/HMR.svg)

### 我能用它做什么？
你可以在开发中用它替代 LiveReload 。 webpack-dev-server 支持一个热加载模式，它会在整个页面重新加载之前尝试通过 HMR 更新。你只需要添加`webpack/hot/dev-server`到入口以及使用`--hot`参数运行 dev-server 。 

如果 HRM 更新失败`webpack/hot/dev/dev-server`会重新加载页面。如果你想[自己重新加载页面](https://github.com/webpack/webpack/issues/418)，你可以用`webpack/hot/only-dev-server`替换添加的入口。

你也可以把它当作一个更新机制在生产中使用。你必须将 HMR 整合到你的应用代码里。

一些加载器已经生成能够热更新的模块（例如：`style-loader`能够改变样式）。这种情况下你不需要做什么特别的事情。

### 使用它需要什么？
如果你 “accept” 一个模块，仅仅这个模块会被更新。所以你必须在这个模块的的父模块或父模块的父模块调用`module.hot.accept`。比如，在一个路由或者一个子视图会是一个好地方。

如果你只想通过 webpack-dev-server 使用它，仅需添加`webpack/hot/dev-server`入口。否者你需要添加 HMR 管理代码，调用`check`和`apply`。

你需要在编译器开启 records ，来在多个进程中跟踪模块 id 。（监视模式和 webpack-dev-server 保存记录在内存中，所以你开发时不需要开启）

你需要在编译器里启用 HMR 来允许编译器添加一个 HMR 运行环境。

### 好在哪里？
- 它像 LiveReload 但是它是每一个模块的 LiveReload 。
- 你能在生产中使用它。
- 根据你的代码分割，只更新修改过的部分。
- 在你的应用里的一部分使用它，不会影响其他模块。
- 如果 HMR 禁用，所有的 HMR 代码在编译的时候会被移除（抱在`if(module.hot)`里面的）。


### 注意事项
- 它是实验性质的，没有通过严格的测试。
- 预计有一些 bug 。
- 理论上在生产中是可用的，但是使用它在正经事上太早了。
- 因为需要跟踪每一次编译模块的 ids ，所以必须保存它们(`records`)。
- 在第一次编译后，优化器不能优化模块的 ids 了。因此打包的大小受到了一点影响。
- HMR 运行环境的代码会增加打包的大小。
- 需要为生产中使用添加 HMR 处理器的测试。

## 教程

使用 webpack 代码热替换 你需要4样东西：  

- records （`--records-path`，`recrodsPath: ...`）  
- 全局启用代码热替换（`HotModuleReplacementPlugin`）
- 在你的代码里使用`module.hot.accept`来热替换代码
- 在你的代码里使用`module.hot.check`和`module.hot.apply`来管理代码热替换

一个简单的实例：
```css
/* style */
body {
  background: red;
}
```
```javascript
/* entry.js */
require("./style.css");
document.write("<input type='text' />");
```
这样就可以通过 dev-server 使用代码热替换了。  
```shell
npm install webpack webpack-dev-server -g
npm install webpack css-loader style-loader
webpack-dev-server ./entry --hot --inline --module-bind "css=style\!css"
```
开发服务器在内存中提供了records，原文说开发的时候棒棒哒。  
`--hot`参数开启代码热替换。
>它添加了 HotModuleReplacementPlugin 。务必使用`--hot`参数，或者在`webpack.config.js`中使用 HotModuleReplacementPlugin ，但是不要两者同时使用，同时使用的话 HMR 插件会被添加两次，扰乱了
系统。

`webpack/hot/devserver`内有专门针对 dev-server 的管理代码，通过参数`--inline`自动添加。（你不必添加到你的 `webpack.config,js`）  
`style-loader`模块已经包含热替换代码。  
如果访问 <http://localhost:8080/bundle> 你将会看到一个红色的页面和一个输入框。输入一些文本到输入框，编辑`style.css`修改背景颜色。

哇... 背景颜色更新了，但是没有刷新整个页面。  

更多关于怎么写热替换（管理）代码： [hot module replacement](https://webpack.github.io/docs/hot-module-replacement.html)  

无需代码的示例 [example-app](http://webpack.github.io/example-app/)。 (注意: 有点老了, 别看源码, 因为 HMR 的 API 有变动)
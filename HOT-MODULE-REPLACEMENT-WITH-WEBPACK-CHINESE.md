# WEBPACK的模块热替换
模块热替换(HMR) 任然是一个实验特性。  

##翻译说明
原文：<https://webpack.github.io/docs/hot-module-replacement.html>

## 引言
模块热替换(HMR) 在一个应用运行的时候更改、添加或者移除模块__无需__重新加载页面。

### 你需要了解
- 使用插件: [http://webpack.github.io/docs/using-plugins.html](http://webpack.github.io/docs/using-plugins.html)
- Code Splitting: [http://webpack.github.io/docs/code-splitting.html](http://webpack.github.io/docs/code-splitting.html)
- webpack-dev-server: [http://webpack.github.io/docs/webpack-dev-server.html](http://webpack.github.io/docs/webpack-dev-server.html)

### 如何工作
在编译过程中，Webpacks 添加了一个小的模块热替换(HMR) 运行时环境包在你的 App 内部运行。Webpack 在编译完成时，它将继续监视源文件的改动。如果 Webpack 检测到文件改动，它仅仅会编译有更新的模块。 根据配置，Webpack 将会发送一个信号到 HMR 运行时环境，或者 HMR 运行时环境 将轮询 Webpack 的变更。 无论哪种方式，变更的模块被传到 HMR 运行时环境，然后尝试热更新应用。首先 HMR 运行时环境会检擦更新的模块是否可以调用`.accept()`接受更新。如果不能，继续检查哪些模块`require`了更新的模块。如果那些模块也不能通过`.accept()`更新，那么继续冒泡向上检查`require`了那些模块（`require`了更新过的模块的模块）的模块。HMR 运行时环境会向上冒泡检查直到到模块可以调用`.accept()`更新。或者冒泡到应用的入口，热更新失败。

### 通过你的应用界面
你的应用通知 HMR 运行时环境检查更新。HMR 运行时环境下载更新（异步），并告诉应用有可用更新。应用通知 HMR 运行时环境进行更新。 HMR 运行时环境执行更新（同步）。 这个过程，你可以决定是否需要通过你的应用界面交互来执行。  
_注：The app code暂翻译为你的应用_

### 通过 webpack 编译程序界面
In addition to the normal assets, the compiler needs to emit the “Update” to allow updating the previous version to the current version. The “Update” contains two parts:

the update manifest (json)
one or multiple update chunks (js)
The manifest contains the new compilation hash and a list of all update chunks (2.).

The update chunks contains code for all updated modules in this chunk (or a flag if a module was removed).

The compiler also makes sure that module and chunk ids are consistent between these builds. It uses a “records” json file to store them between builds (or it stores them in memory).
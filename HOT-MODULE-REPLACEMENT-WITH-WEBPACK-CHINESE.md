# WEBPACK的模块热替换
模块热替换(HMR) 任然是一个实验特性。

## 引言
模块热替换(HMR) 在一个应用运行的时候更改、添加或者移除模块__无需__重新加载页面。

### 你需要了解
- 使用插件: [http://webpack.github.io/docs/using-plugins.html](http://webpack.github.io/docs/using-plugins.html)
- Code Splitting: [http://webpack.github.io/docs/code-splitting.html](http://webpack.github.io/docs/code-splitting.html)
- webpack-dev-server: [http://webpack.github.io/docs/webpack-dev-server.html](http://webpack.github.io/docs/webpack-dev-server.html)

### 如何工作
在编译过程中，Webpacks 添加了一个小的模块热替换(HMR) 运行时环境包在你的 App 内部运行。Webpack 在编译完成时，它将继续监视源文件的改动。如果 Webpack 检测到文件改动，它仅仅会编译有更新的模块。 根据配置，Webpack 将会发送一个信号到 HMR 运行时环境，或者 HMR 运行时环境 将轮询 Webpack 的变更。 无论哪种方式，变更的模块被传到 HMR 运行时环境，然后尝试热更新应用。首先 HMR 运行时环境会检擦更新的模块是否可以调用`.accept()`接受更新。如果不能，继续检查哪些模块`require`了更新的模块。如果那些模块也不能通过`.accept()`更新，那么继续冒泡向上检查`require`了那些模块（`require`了更新过的模块的模块）的模块。HMR 运行时环境会向上冒泡检查直到到模块可以调用`.accept()`更新。或者冒泡到应用的入口，热更新失败。

### From the app view
应用的会向 HMR 运行时环境检查更新。HMR 运行时环境下载更新（异步），并告诉应用有可用更新。The app code asks the HMR runtime to apply updates. The HMR runtime applies the update (sync). The app code may or may not require user interaction in this process (you decide).
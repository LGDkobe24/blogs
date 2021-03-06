Title: 微信小程序原理
Date: 2016-11-05 07:52
Modified: 2016-11-05 07:52
Tags: weapp
Slug: weapp-principle
Authors: Joey Huang
Summary: 微信小程序使用了前端技术栈 JavaScript/WXML/WXSS。它背后的原理是怎么样的呢？

## 写在前面

微信小程序使用了前端技术栈 JavaScript/WXML/WXSS。但和常规的前端开发又有一些区别：

* JavaScript: 微信小程序的 JavaScript 运行环境即不是 Browser 也不是 Node.js。它运行在微信 App 的上下文中，不能操作 Browser context 下的 DOM，也不能通过 Node.js 相关接口访问操作系统 API。所以，严格意义来讲，微信小程序并不是 Html5，虽然开发过程和用到的技术栈和 Html5 是相通的。
* WXML: 作为微信小程序的展示层，并不是使用 Html，而是自己发明的基于 XML 语法的描述。
* WXSS: 用来修饰展示层的样式。官方的描述是 “ WXSS (WeiXin Style Sheets) 是一套样式语言，用于描述 WXML 的组件样式。WXSS 用来决定 WXML 的组件应该怎么显示。” “我们的 WXSS 具有 CSS 大部分特性...我们对 CSS 进行了扩充以及修改。”基于 CSS2 还是 CSS3？大部分是哪些部分？是否支持 CSS3 里的动画？不得而知。

在微信小程序官方文档上，有下面这段话：

> 微信小程序运行在三端：iOS、Android 和 用于调试的开发者工具
>
> * 在 iOS 上，小程序的 javascript 代码是运行在 JavaScriptCore 中
> * 在 Android 上，小程序的 javascript 代码是通过 X5 内核来解析
> * 在 开发工具上， 小程序的 javascript 代码是运行在 nwjs（chrome内核） 中

我们先从开发工具谈起。

## 开发工具

小程序的 javascript 代码运行在 nwjs 中。nwjs 是什么鬼呢？官方介绍是这样写的：

> NW.js (previously known as node-webkit) lets you call all Node.js modules directly from DOM and enables a new way of writing applications with all Web technologies.

[nwjs](http://nwjs.io) 合并 Browser 和 Node.js 的运行时，可以使用前端开发技术来开发跨平台的应用程序。借助 Node.js 访问操作系统原生 API 的能力，可以开发中跨平台的应用程序。微信小程序开发工具就是使用 nwjs 开发的。如果你是 Mac 用户，进入目录 `/Applications/wechatwebdevtools.app/Contents/Resources/app.nw/app` 可以看到开发工具的实现代码，当然代码是经过混淆的。网上流行的破解版本开发工具原理上就是修改这里面的代码。

与此类似的，一个更火的项目是 [Electron](http://electron.atom.io)，由 GitHub 推出的，它也是把 Browser 和 Node.js 结合，用来开发跨平台的应用程序。程序员们应该听说过 [Atom](https://atom.io) 这个编辑器界的后起之秀。包括微软拥抱开源社区的编辑器 [vscode](http://code.visualstudio.com) 也是使用 Electron 开发的。

### Electron vs nwjs

这两个平台有什么区别？为什么微信选择 nwjs 呢？我们不妨猜一猜。

从技术角度来讲：

* 应用程序入口不同：Electron 入口是一个 javascript 脚本，脚本里要自己负责创建浏览器窗口，加载 html 页面。而 nwjs 的入口就是一个 html 页面，框架自己会创建浏览器窗口来显示这个 html 页面。
* Node.js 集成方式不同：Electron 直接使用 Node.js 的共享库，不需要修改 Chromium 代码。而 nwjs 为了集成 Node.js ，需要修改 Chromium 代码，以便在浏览器里能通过 Node.js 访问系统原生 API。
* Multi-Context: nwjs 有多个上下文，一个是浏览器的上下文，用来访问 Browser 相关 API，比如操作 DOM ，另外一个是 Node 上下文，用来访问操作系统 API。Electron 没有使用多个上下文，对开发者更友好。

从应用角度来讲：

* 打包后的文件大小：Electron 打包后文件会比 nwjs 小不少。一个 18M 的程序，使用 Electron 打包后是 117M，而使用 nwjs 打包后的程序是 220M。微信小程序开发工具打包后是 219M (v0.10.102800)。没有亲测，评价来源参考文档。
* 代码保护：Electron 只支持代码混淆来保护，而 nwjs 把核心代码放在 V8 引擎里，不但可以保护代码，还可以提高执行效率。
* 开源社区活跃度：Electron 应该是完胜的。看看[使用 Electron 构建的应用程序](http://electron.atom.io/apps/)就知道了。而据说 nwjs 的开发文档有些都没有及时更新。
* 应用程序启动时间：Electron 会稍微快一点。没有亲测，评价来源参考文档。

从这个分析猜测，微信选择 nwjs 的原因可能是出于代码保护。毕竟开发工具可以上传小程序，有些接口和数据需要比较严密的保护。哪位大牛可以挖挖看哪些代码被保护起来了。

## 真机运行环境

下面内容完全是猜测的，如有言中，实属运气。

微信小程序的运行环境应该更类似 ReactNative 之类，而不是纯 Html5。两者最大的不同在于，ReactNative 的界面是由原生控件渲染出来的，而 Html5 的界面是由浏览器内核渲染出来的。两者在性能上有较大的差异，感兴趣的可以参阅我的另外一篇文章[《跨平台 App 开发技术方案汇总》](http://www.jianshu.com/p/2b4926fa45df)。

原理上，小程序是如何在微信 App 里运行的呢？

* 微信 App 里包含 javascript 运行引擎。
* 微信 App 里包含了 WXML/WXSS 处理引擎，最终会把界面翻译成系统原生的控件，并展示出来。这样做的目的是为了提供和原生 App 性能相当的用户体验。

我们来意淫一下小程序加载运行的过程：

* 用户点击打开一个小程序
* 微信 App 从微信服务器下载这个小程序
* 分析 `app.json` 得到应用程序的配置信息（导航栏，窗口样式，包含的页面列表等）
* 加载并运行 `app.js`
* 加载并显示在 `app.json` 里配置的第一个页面

这个只是从开发者眼中看到的一个简化版的过程，实际过程应该比这要复杂得多，涉及到浏览器线程（就是运行我们的逻辑层代码 app.js 等的线程）和 AppService 线程之间的交互。从官方网站上的一个图片可以看出端倪：

![生命周期](https://mp.weixin.qq.com/debug/wxadoc/dev/image/mina-lifecycle.png)

至于微信 App 是如何与小程序的逻辑层 javascript 交互的呢？可以简单地归纳如下：

JavaScript 是脚本语言，可以在运行时解释并执行。微信 App 里包含了一个 JavaScript 引擎，由它来负责执行逻辑层的 JavaScript 代码。那么 JavaScript 调用的小程序相关 API 怎么实现的呢？答案是最终会被翻译成实现在微信 App 里的原生接口。比如开发者调用 `wx.getLocation(OBJECT)` 获取当前地理位置，微信 App 里的 JavaScript 引擎在执行这个代码时，会去调用微信 App 里实现的原生接口来获取地理位置坐标。

感兴趣的朋友可以阅读我之前推荐过的一篇文章[《React Native 从入门到原理》](http://www.jianshu.com/p/978c4bd3a759)。文章分析的虽然是 ReactNative，但实际上原理是相通的。

## 总结

微信小程序最大的好处是不需要做设备适配，只要微信能运行，小程序就能运行。小程序虽然是一个封闭形态下的前端开发技术，但借助微信的巨大影响力，几乎所有人都在往里面冲。微信小程序太火了，内测火，公测更火。内测刚出来，就有人用微信小程序实现了商城，并开源。感叹一下：你的热情，就像一把火，燃烧了整个沙漠。

作为开发者，提几个不足：

1. 不支持从 node_modules 中加载模块。这样无形中就把 npm 排除在外了。从开发生态角度，这个应该是微信小程序下一步要重点解决的问题吧。
2. 开发工具自带的代码编辑器还是太简陋了。不知道为什么微信要重复发明轮子。理论上，给流行的代码编辑器 (sublime/atom/vscode etc.) 开发个插件。然后用户直接到小程序后台上传提交审核就好了。程序员是挑剔到近乎偏执的物种，代码编辑器又是程序员时刻打交道的工具，要做好实属不易。

## 参考文档

1. https://github.com/electron/electron/blob/master/docs/development/atom-shell-vs-node-webkit.md
2. https://www.akawebdesign.com/2015/05/06/electron-vs-nwjs/
3. https://www.akawebdesign.com/2015/11/02/electron-vs-nwjs-part-2/


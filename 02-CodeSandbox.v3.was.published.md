**V3** 版本是 **CodeSandbox** 诞生以来最大的更新。其中包括 VSCode 扩展，以及许多设计上的调整和全新的开发工具面板。在 GitHub 上，CodeSandbox 的贡献者已经有114人。 

### VSCode 集成 

在 CodeSandbox 已经集成了 VSCode，现在 VSCode 是CodeSandbox 中唯一的编辑器。可以在 CodeSandbox 中使用 VSCode 的键绑定、代码模板、命令、多视图编辑等诸多功能。新版本充分利用了浏览器本身的能力，将更多的工作放在浏览器中直接完成，而不需要后台服务器的支持，这样就大大减少了用户需要等待的时间，带来了很好的体验。相比于之前的版本，CodeSandbox 的界面经过了重新设计，并且支持了很多新的特性。 

在最开始的时候，CodeSandbox 只支持很少的扩展，现在正在逐渐开放这方面的能力，到以后，用户就可以随意的安装扩展了。 

这个版本的 CodeSandbox 已经有了很多功能强大的扩展： 

**本地 TypeScript 类型检测**： 

在 CodeSandbox 中使用 TypeScript 扩展已经和本地 VSCode 中使用有一样的效果了。可以通过 `tsconfig.json` 来配置，可以运行最新版本的 TypeScript。 

![](https://cdn-images-1.medium.com/max/2400/1*M-lQroL98k2SvFwhVMezfQ.gif) 

**自动导入**： 

可以从你自己的文件或者依赖中导入变量。 

![](https://cdn-images-1.medium.com/max/2400/1*dardCLKUrGIMg6bKpxLCtg.gif) 

**重构**： 

可以使用 VSCode 中一样的代码重构功能。 

![](https://cdn-images-1.medium.com/max/2400/1*3AfE1Lrsv5uQ71f4qTzxLA.gif) 

**Vim**： 

CodeSandbox 支持 [VSCodeVim](https://github.com/VSCodeVim/Vim) 插件，直接在设置中就可以打开。 

**支持 Vue**： 

CodeSandbox 中也直接集成了 **Vetur**，对 Vue 的支持与 VSCode 中是一样的，也包括了对新添加的模板的补全。 

![](https://cdn-images-1.medium.com/max/2400/1*w3dF9Ixcq63zQJIwbv5ROw.gif) 

另外也有一起其他的支持的改进，比如支持 graphql，styled-components 和 yarn.lock 的语法高亮等，现在也支持 UI 的配置和图片的预览。 

![](https://cdn-images-1.medium.com/max/2400/1*4maOvmdu7HQpiOP5N58-fQ.png) 



### 新的设计 

在完全支持 VSCode 之后，页面也重新进行了设计。在新版的设计中，去掉了那些不太友好的设计，比如在查看别人的 Codesandbox 时会隐藏左侧的菜单栏，同时增加了很多新的设计. 

![](https://cdn-images-1.medium.com/max/2400/1*cK6OKF0-bQSx1qZzaG7ryQ.png) 

**开发工具面板**： 

浏览器预览已经变成了多功能的。除了可以进行默认的页面预览之外，还增加了新的 tab 页来支持命令行，这两个 tab 切换很容易，如果你想把命令行作为默认的 tab，只需要拖拽一下就可以了。 

![](https://cdn-images-1.medium.com/max/2400/1*_B27FW6q0VKUN1FLP66PGA.gif) 

新的开发工具面板对于测试和命令行输出更加友好。以后命令行面板中会增加更多的开发工具。 

**菜单栏和状态栏**： 

在这次的更新中，顶部也有了很大的更新，直接从 VSCode 中将菜单栏和状态栏的功能拿过来了。之前的版本中顶部有太多的按钮，很难找到需要的功能。在这个版本中，将这些按钮按照功能的分类，放入到菜单底下，这个操作就和使用 VSCode 的体验差不多了。 

**颜色和主题**： 

UI 上有了新的图标和新的颜色主题。后续 CodeSandbox 颜色主题会在 VSCode 的 marketplace 中上架。 

\> 原文：https://hackernoon.com/announcing-codesandbox-v3-4febbaba1963 

(完) 
# 在线代码编辑器

在线代码编辑器或是在线IDE是个很有意思的产品，这种在线的代码编辑器可能并不适合正式的项目开发，但是其还有一定的优势的，这些优势如下：

1. 轻量级的在线代码编辑器，不需要本机环境，只要有网络和浏览器，就可以任意位置用任意设备进行编程。
2. 方便代码分享，尤其对于一些用于文章内的代码示例。
3. 方便于初学者或是想尝试一段试验型的代码的场景。
4. 方便教学，面试，结对编程，练习……
5. 方便快速地检查代码，或是进行一些比较小的代码修改。

所以，这种在线代码IDE不需要运行环境，打开就用，还是有很多用处的。这篇文章，将会介绍几个不错的在引编辑器。

## CodeSandbox

[CodeSandbox](https://codesandbox.io/) 是一个在线的前端代码项目IDE，支持 React.js，Vue.js，Angular.js，Preact.js，Vanilla，Svelte，Dojo…… 前端框架，直接写直接预览结果，不需要使用什么如npm install的命令行，就可以直接把项目跑起来了，还要吧在线的从你的Github把代码同步过来，以及把修改的代码再同步回去。在打开编辑器的时候，还有一个实时显示的预览窗，只要你的代码做了修改，结果就会立马显示在预览窗里了，这种反馈让人很爽。另外，还可以如大多数在线IDE一样，可以做结构编程（多人同时修改）。当然，它也能像 [CodePen](https://codepen.io/)、[Jsfiddle](https://jsfiddle.net/) 或 [Jsbin](https://jsbin.com/) 一样可以把代码share出去，成为一个playgrand。

不过，这个前端的在线编辑最强大的还不是这些，去年十月，CodeSandbox 把自己的在线编辑变成了 VSCode，设计者把CodeSandbox整得跟VSCode一模一样，完全就是一个VSCode的在线的版本。让用户可以在 CodeSandbox 中使用 VSCode 的键绑定、代码模板、命令、多视图编辑等诸多功能。但是对于VSCode的扩展，CodeSandbox 那时还只支持很少的扩展。 

这两天（2019年3月），其发布了 **V3** 版本，这是 CodeSandbox 诞生以来最大的更新。其中包括可以直接使用 VSCode 扩展，比如：

- [VSCodeVim](https://github.com/VSCodeVim/Vim) 扩展，直接在设置中就可以打开。 
- [Vetur](https://github.com/vuejs/vetur)扩展，对 Vue 的支持与 VSCode 中是一样的，也包括了对新添加的模板的补全。

随着更多VSCode的扩展的支持，未来可以直接随意安装VSCode的扩展，这样一样，就和VSCode没什么两样了。

除此之外，在最新的[发布公告](https://hackernoon.com/announcing-codesandbox-v3-4febbaba1963)中，我们可以看到CodeSandbox还整了好多有趣强大的功能，这些都受益于VSCode的插件。比如：

- **原生的 TypeScript 类型检测**。在 CodeSandbox 中使用 TypeScript 扩展已经和本地 VSCode 中使用有一样的效果了。可以通过 `tsconfig.json` 来配置，可以运行最新版本的 TypeScript。 

![](https://cdn-images-1.medium.com/max/2400/1*M-lQroL98k2SvFwhVMezfQ.gif) 

- **自动导入**。可以从你自己的文件或者依赖中导入变量 —— 当你使用了相关的外部变量时，会自动帮你添加相关的 `import` 语句。 

![](https://cdn-images-1.medium.com/max/2400/1*dardCLKUrGIMg6bKpxLCtg.gif) 

- **重构**。可以使用 VSCode 中一样的代码重构功能。 比如，把 promise 版重构成 async 版，或是把一个函数移到另外一个文件中。

![](https://cdn-images-1.medium.com/max/2400/1*3AfE1Lrsv5uQ71f4qTzxLA.gif) 

另外也有一起其他的支持的改进，比如支持 graphql，styled-components 和 yarn.lock 的语法高亮等，现在也支持 UI 的配置和图片的预览。 

![](https://cdn-images-1.medium.com/max/2400/1*4maOvmdu7HQpiOP5N58-fQ.png) 


（未完）

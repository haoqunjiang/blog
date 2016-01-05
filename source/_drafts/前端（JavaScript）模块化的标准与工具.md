title: 前端（JavaScript）模块化的标准与工具
tags:
---

本文所讲的模块化主要指 JavaScript 模块化。尽管本文所述的有些工具支持引入和打包 CSS 文件，但是 CSS 的模块化（[或者称之为「组件化」可能更合适点][hax's blog]）跟 JavaScript 相比，在需求和实现方式上都有很大的不同，并不适合放在一起讨论，而且篇幅所限，本文也无法再详述这么一个复杂的问题。

<!-- more -->

## 模块化解决的问题

- 职责分离。可以提高代码的可维护性。
- 代码复用。
- 命名冲突。
- 依赖管理、资源定位。
- 多版本共存。好的模块化方案应当能通过别名等配置方法，实现模块的版本管理和多版本共存。


## JavaScript 模块化方案/标准

### 直接暴露到全局空间

### [CommonJS][] (Modules/1.0)

得益于 [Node.js][] 的流行，这是最广泛使用的 JavaScript 模块化标准。

### [AMD][] (Modules/Async)

### CMD (Modules/2.0)

### [UMD]

### ES2015 Module Standard (Modules/1.x)


## 前端模块加载器/打包工具

### [LABjs][]

### YUI3 Loader

### RequireJS

### SeaJS

### [MT]

### ESL

### Browserify

### Webpack

### System.js

### Rollup.js


## 题外话：前端模块加载器的实现原理

<http://www.cnblogs.com/hustskyking/p/how-to-achieve-loading-module.html>
<http://requirejs.org/docs/why.html](http://requirejs.org/docs/why.html>


## 参考资料

除了文中的链接以外，本文撰写过程中还参考了以下一些文章/网站：

- [已有 JS 模块化和打包方案收集 by 题叶](https://github.com/island205/bodule/wiki/%E5%B7%B2%E6%9C%89-JS-%E6%A8%A1%E5%9D%97%E5%8C%96%E5%92%8C%E6%89%93%E5%8C%85%E6%96%B9%E6%A1%88%E6%94%B6%E9%9B%86)
- [JavaScript 依赖管理 by fantasyni](http://bearcatjs.org/%E5%8D%9A%E5%AE%A2/index.html)
- [Simplicity and JavaScript modules by James Burke](http://tagneto.blogspot.com/2012/01/simplicity-and-javascript-modules.html)
- [AMD is Not the Answer by Tom Dale](http://tomdale.net/2012/01/amd-is-not-the-answer/)
- [再谈 SeaJS 与 RequireJS 的差异 by 糖饼](http://div.io/topic/430)
- [前端工程与模块化框架 by 前端农民工](http://div.io/topic/439)
- [WHY WEB MODULES? by RequireJS contributors](http://requirejs.org/docs/why.html)
- [WHY AMD? by RequireJS contributors](http://requirejs.org/docs/whyamd.html)
- [JavaScript Module Loader by 黄玄](http://huangxuan.me/2015/05/25/js-module-loader/)
- [JavaScript 模块化七日谈](http://huangxuan.me/2015/07/09/js-module-7day/)
- [前端模块化开发那点历史 by 玉伯](https://github.com/seajs/seajs/issues/588)


[hax's blog]: https://github.com/hax/hax.github.com/issues/21
[CommonJS]: http://www.commonjs.org/
[Node.js]: https://nodejs.org/
[AMD]: https://github.com/amdjs/amdjs-api/blob/master/AMD.md
[LABjs]: http://labjs.com/
[MT]: https://mtjs.github.io/


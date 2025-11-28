# Vue相关知识
## 一、Vue文件调用顺序
### 1、npm run dev
Vue的本质是js的语法糖，node.js是运行JS的直接面向操作系统的底层引擎，Node.js = V8引擎（负责把 JS 编译成机器码） + libuv 库（负责提供“非阻塞 I/O”和事件循环） + 一堆 C/C++ 绑定（fs、net、crypto、zlib …）  

Vite是一个“前端开发服务器+热更新+即使构建”工具，在本地启动一个HTTP服务，把浏览器的ESM请求实时按需编译，通过WebSocket推送“热模块替换HMR”，页面局部实时更新

### 2、扫描入口文件src/main.js
Vue需要一个根组件来生成虚拟DOM树，App.vue充当这个根节点，在入口文件中，一般只做三件事：
1. 全局布局（header/footer/sidebar）
2. 放一级路由出口 （router-view/）
3. 注入全局状态（Pinia、Router、i18n…）  

对于这个项目，在main.js做的事情：
1. 加载/写入/本地存储数据的函数
2. 创建全局状态，实时响应显示数据，保存数据，清除数据，清除特定数据
3. 创建应用实例，挂载实例
4. 应用一级路由
5. 监听用户“刷新”行为，需要保证当用户点击刷新按钮时，页面没有清除数据，而是将重定位到总览界面（实际上这里需要并没有成功实现，当用户点击刷新按钮时，旧进程已经kill）
唯一能跨刷新保留数据的api是cookie/url本身的query、path，保存数据需要靠其他的持久化手段

挂载app实例，把vue内部的虚拟DOM树和真实DOM树对接，需要在HTML页面里留下一个接口插座
```
// main.js
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)   // 1. 创建“应用实例”
app.mount('#app')            // 2. 挂载 → 接电线
```
```
//HTML
<div id="app"></div>
```
### 3、DOM树的生成与修改
所有的前端框架/库（React、Vue等等）最终在浏览器里都会翻译为HTML、CSS和JavaScript，对这三种组件的操作抽象为了不同的语法糖、运行时和编译器  
| 框架          | 写代码时                             | 编译 / 运行时后                                                          | 真正进浏览器的                                           |
| ----------- | -------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------- |
| **React**   | JSX + Hooks                      | `React.createElement(...)` → 虚拟 DOM → Diff → `element.appendChild` | HTML 元素 + 内联样式/class + JS 事件回调                    |
| **Vue**     | `<template>{{ msg }}</template>` | 模板编译成 `render` → 虚拟 DOM → Patch                                    | 同上                                                |
| **Angular** | 模板语法 + TypeScript                | NgCompiler → 组件工厂 → 动态创建 DOM                                       | 同上                                                |
| **Svelte**  | `.svelte` 单文件                    | 编译期直接生成 **命令式 DOM 操作代码**（无虚拟 DOM）                                  | 还是 `document.createElement` + `style.textContent` |
| **Solid**   | JSX                              | 编译成 **细粒度响应式** → 直接 `el.textContent = ...`                         | 同上                                                |  

DOM（Document Object Model）树是浏览器把一段HTML文件翻译为内存中一棵倒挂的树，方便js对对象进行增删改查页面的结构和内容  
DOM树的生成：  
1. 浏览器下载HTML字节流
2. 分词 生成token 组装为节点（Node）
3. 按嵌套关系生成树形结构

HTML与DOM树的对应关系：
| HTML 源码   | DOM 树          |
| --------- | -------------- |
| 只是**字符串** | 是**内存对象**      |
| 写错了会容错补全  | 浏览器自动补全后生成正确结构 |
| 无法直接编程    | 可被 JS 任意增删改查   |

```
<html>
  <head>
    <title>Demo</title>
  </head>
  <body>
    <h1>Hello</h1>
    <p>World</p>
  </body>
</html>
```
```
document
└─ html
   ├─ head
   │  └─ title
   │     └─ Text: "Demo"
   └─ body
      ├─ h1
      │  └─ Text: "Hello"
      └─ p
         └─ Text: "World"
```
DOM树中有哪些节点：
1. 元素节点（Element Node）：html、body、h1
2. 文本节点（Text Node）："Hello"、"World"
3. 属性节点（Attribute）：保存在元素节点上，不单独挂树
4. 文档节点（Document）：整棵树的根
Javascript如何修改DOM树：
```
const h1 = document.querySelector('h1') // 找到 h1 节点
h1.textContent = 'Hi!'                 // 改文本
const img = document.createElement('img')
img.src = 'cat.jpg'
document.body.appendChild(img)         // 挂新枝
```
DOM树与渲染树：  
1. DOM树是纯逻辑结构，与样式无关
2. 渲染树是DOM树里的可见节点+CSSOM的样式  
### 对Vue文件的处理
在Vue项目中，需要处理的只有“.vue”文件，  对于一个SFC(single file component),由三部分组成，分别是template、script、style
```
<template>…</template>   <!-- 1. 视图模板 -->
<script>…</script>       <!-- 2. 逻辑（JS/TS） -->
<style>…</style>         <!-- 3. 样式（CSS/SCSS/Less …） -->
```
对于每个单独vue文件，会在vite中注册为一个transform钩子（vite插件），然后将整份SFC切开，各自编译，最终拼成一块纯ES模块返回给浏览器（ES模块，包含js/css/静态资源）  
1. Template块 ——> @vue/compiler-sfc编译为Render()函数，结果为一段js代码，render 函数在浏览器里执行才会生成虚拟 DOM 树
2. Script块 ——>  编译器把顶层的变量、import 全部自动 return 出去，生成真正的 setup() 函数,结果为一段符合 ESModule 的 JS
3. Style 块 ——>  最终把 CSS 字符串扔给 vite 的 createHotContext，以 JS 模块形式注入 <style> 标签，并接入 HMR
4. 把上述三步的产物合并为单份ES模块  
.vue 文件被 Vite/vue-loader 切成三块：模板→render 函数，脚本→ES 模块，样式→CSS 模块；开发时各走各的热更新通道，生产时统一由 Rollup 打包成浏览器认识的纯 JS/CSS/HTML，再也没有 .vue 的影子。

### 关于Vite
Vite只是前端构建工具，dev阶段：按需编译+热更新+代理，build阶段：调用Rollup做生产打包、代码分割、压缩、哈希
Vite的编译器：@vue/compiler-sfc（单文件组件）和 @vue/compiler-core（跨平台核心）负责把 .vue 里的 <template> 编译成render 函数，render 函数在浏览器里执行才会生成虚拟 DOM 树。

### 虚拟DOM树和真实DOM树
虚拟 DOM 和真实 DOM 的对齐是“逐步”进行的，发生在每次渲染时，通过 diff + patch 的方式，把变化应用到真实 DOM 上。

| 阶段                     | 描述                                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------------------- |
| **首次渲染（mount）**        | 虚拟 DOM 树被创建，Vue 会将其完整地 **patch** 到真实的挂载点上（如 `#app`），此时虚拟 DOM 和真实 DOM **首次对齐**。                    |
| **数据更新（update）**       | 响应式数据变化 → 重新执行 `render()` → 生成新的虚拟 DOM 树 → **与上一次的虚拟 DOM 树进行 diff** → 只将**差异部分** patch 到真实 DOM 上。 |
| **强制更新（force update）** | 手动调用 `vm.$forceUpdate()` 或某些极端情况下，Vue 会跳过优化，重新渲染并 patch。                                          |

![Vue文件调用](image/vue.png)

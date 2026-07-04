---
title: Node_VS_Deno
date: 2022-08-08 17:38:00
tags: ["后端"]
---

### 前言

我应该算个node的忠实拥护者与积极实践者，能用node的基本都上node，Express/koa基本换着用一下，node的源码也通过很多的博客与前辈的指导下不断地在阅读。在公司工作时，接触了企业级的node应用，在koa/Express/egg等上都有一些企业级项目经验。最近，社区有在问到我对deno的看法，遂把这篇自己的博客拿了过来，仅供参考。  

最近开始使用Deno，github上找到了一个类似koa的中间件框架oak【A middleware framework for Deno’s net server 🐿️ 🦕】。不得不说这些人真会玩🙄

### [](https://qcblog.hmbstudio.cn/2021/02/09/Deno%20VS%20Node/#%E4%B8%A4%E8%80%85%E5%8C%BA%E5%88%AB "两者区别")两者区别

2021算是deno元年吧，不知道“元年”两字是否适当，在deno发布后，狂热追求者们铺天盖地，自己在阅读相关文章后，deno与node的区别如下：

| OP         | Node                             | Deno            |
| ---------- | -------------------------------- | --------------- |
| API 引用方式   | 模块导入                             | 全局对象            |
| 模块系统       | CommonJS & 新版 node 实验性 ES Module | ES Module 浏览器实现 |
| 安全         | 无安全限制                            | 默认安全            |
| Typescript | 第三方，如通过 ts-node 支持               | 原生支持            |
| 包管理        | npm + node_modules               | 原生支持            |
| 异步操作       | 回调                               | Promise         |
| 包分发        | 中心化 npmjs.com                    | 去中心化 import url |
| 入口         | package.json 配置                  | import url 直接引入 |
| 打包、测试、格式化  | 第三方如 eslint、gulp、webpack、babel 等 | 原生支持            |

### [](https://qcblog.hmbstudio.cn/2021/02/09/Deno%20VS%20Node/#Deno%E6%98%AF%E4%B8%80%E4%B8%AA%E9%94%99%E8%AF%AF%E7%9A%84js-runtime%EF%BC%9F "Deno是一个错误的js runtime？")Deno是一个错误的js runtime？

为什么会突然写这篇总结尼？今天逛社区，看大了这样一篇文章，[为什么我认为 Deno 是一个迈向错误方向的 JavaScript 运行时？](https://www.freecodecamp.org/news/why-deno-is-a-wrong-step-in-the-future/),觉得还是写的中规中矩又不失道理。

原作者 Mehul 在原文以及两个视频中到底想要说 Deno 为什么没那么好？对deno的优点进行了对应的挑战。其观点大致梳理如下：

- Deno 和 Node 确实有竞争关系，因为你必须在你的下个项目中作出选择  
- Deno 现在所做的成果并不是很多，大多特性都可以在 Node 生态中较好地解决掉。
- URL import 还是一场灾难。NPM 中已经有很多明星项目“竟然只有一行代码”、“暗中偷窃用户数据”、“注入挖矿代码”、“兼容性出现问题导致很多上游库受影响”等问题，URL import 本身并不能解决这些问题，更没有一个像 Node 一样强壮的社区来保证受人信任的依赖库，也就不会有更多的开发者愿意加入到 Deno 生态中。
- 由于 TypeScript 是 JavaScript 的超集，完全可以选择跳过类型验证，此时推荐新手在 Deno 上直接使用 TypeScript 编程坑会很大，很可能会出现一堆 any 类型。在经常性的调试报错下，TypeScript 的学习成本也较高，很容易写出低质量代码。
- TypeScript 并不是直接在 Deno 上跑的，其实还是变成了 JavaScript 来跑，何必一定要集成到 Deno 中呢？
- 安全是一个很难的事情，Deno 宣传自己的“安全沙箱”注定要承担很大的责任。Deno 安全沙箱也没有必要，完全可以用 Docker 等容器或虚拟化技术来支持。同时，真正想搞破坏的脚本也会找到自己的方式来规避安全问题。
- 以当时版本下的 deno –allow-run 运行主进程从而开启的子进程能轻松突破安全沙箱的验证来获得更多权限为例，发现 Deno 的“安全沙箱”并没有所说的那么安全。
- Deno 没有必要集成太多工具链（代码格式化、测试工具、TypeScript 等等）于一体，让各种第三方工具链来一起共建生态的同时，保证 Deno 本身的专注性并提供更友好的插件支持会很好。
- Node 的异步模型并没有被淘汰，promise 和事件侦听器模型也没有被淘汰，因此不确定 Deno 将如何解决这些没有被淘汰的问题。
- 未来并不确定会有多少开发者愿意将 npm 中的成熟库逐渐迁移到 Deno 中。

### [](https://qcblog.hmbstudio.cn/2021/02/09/Deno%20VS%20Node/#%E6%89%93%E4%B8%AA%E5%B0%8F%E6%80%BB%E7%BB%93 "打个小总结")打个小总结

都说天下苦npm久矣，npm=“你怕吗”？ 😄，ESM比CJS优秀许多，但ESM真就完全解决CJS的问题了么？

新技术的成熟需要数以年计得社区维护，需要数以万计的开发者加入，node已经发展数年，社区成熟的包，成熟的方案能解决很大部分的开发问题，deno的发展还得看社区。
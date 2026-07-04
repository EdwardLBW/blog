---
title: 网络请求与IO杂谈
date: 2021-05-16 13:30:40
tags: ["后端"]
---

前言
我始终觉得博客不是想什么就写什么的，一定是有某一个契机，使得我们对某一个点或者方向有深度探索继续学习或者加深印象的过程，并有体系的将这个过程记录下来，昨天其实就在写博客了，写的点是关于XMLHttpRequest的，是因为之前跟一个同学谈及到了它，以及相关的网络请求标准AJAX、Fetch、axios等，最近，因为在等待论文盲审的过程中，有了写书的想法，于是便匆匆开始了写书之旅，一本关于Node的原理及实践应用的书，暂取名《Node杂谈》，目前已经写了61页了，但我是个善变的人，说不定过几天我就不想写了😄，但希望自己能坚持用半年到一年的时间完成这本可能没啥用的书。接下来开始自己的博客，先谈谈自己对XHR及相关标准，然后谈谈操作系统的I/0及Node的异步I/0。

网络请求
XHR背景
网络请求的这几个标准和库，其实没有必要开一片博客来记录下自己的想法和理解的，偶然和和同学说道这些网络交互的标准或者是库，有些歧义，我又去网上看看了一些人的博客想着是否可以纠正自己可能会是错误的理解，然后我就在网上不断翻阅，按照我自己对自己写的博客的标准来定义，我查阅到的这些博客都是一些不能入眼的内容，只是互联网没有自动垃圾回收的机制，无可厚非，我的理解或者我的博客一样也会被别人理解为www的设计缺陷，但至少我愿意把一件事讲清楚，给自己讲清楚。最近开始筹划并着手写一本自己对于Node的理解及相关实战项目的书，谈及ajax时候，我写了这样一段话：

![image](/images/1.png)



我在这段话中大肆吐槽了所谓的模块或者框架用的很六的使用者。如果你没有看过fetch的源码实现，或者说你并没有看过axios的源码实现，你连拦截器是怎么定义的都不知道，你用的再六又能怎么样尼？希望这段话在勉励我自己的同时，也能触及到看见我博客的志同道合的朋友。接下来我谈一谈我自己对于ajax、fetch、axios的个人理解。

ajax
ajax全称是啥，看到这你能在脑海中反应出来么？异步的javascript和xml，英文：Asynchronous JavaScript and XML，是利用浏览器已有标准重新组合后的新技术标准，应该可以说ajax是web1.0到web2.0跨越的重大功臣之一。为什么这么说，ajax最直观的特点就是通过请求与响应来局部刷新页面，而不是说之前的必须通过表单进行一次整体的请求与响应，刷新整个页面，浪费网络资源，用户体验也极差。ajax有缺点么？或者说有哪些在以前全局刷新情况下能够很容易实现的功能但是使用ajax后就变得比较繁杂尼？以下4点是我在网上寻到并深有同感的几点，如下：
a. AJAX干掉了Back和History功能，即对浏览器机制的破坏
b. AJAX的安全问题
c. 对搜索引擎支持较弱
d. 违背URL和资源定位的初衷

现代浏览器的实现主要依靠XMLHttpRequest对象，因为浏览器的版本不同，带来不对于XHR的兼容问题，那，看到这里，你可以立马动手实现一个最原始的AJAX么？

function loadXMLDoc()
{
 var xmlhttp;
 if (window.XMLHttpRequest)
 {
  //  IE7+, Firefox, Chrome, Opera, Safari 浏览器执行代码
  xmlhttp=new XMLHttpRequest();
 }
 else
 {
  // IE6, IE5 浏览器执行代码
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
 }
 xmlhttp.onreadystatechange=function()
 {
  if (xmlhttp.readyState==4 && xmlhttp.status==200)
  {
   // 返回数据处理
   document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
  }
 }

 // 请求发送 true 异步，肯定异步啊，你要同步？？？
 xmlhttp.open("GET","/try/ajax/ajax_info.txt",true);
 xmlhttp.send();
}
代码我肯定不会一行一行讲解的，这没啥，就很简单，但是onreadystatechange中的readyState以及status是值得关注的，其记录着ajax的运行状态以及响应状态。onreadystatechange：每当 readyState 属性改变时，就会调用该函数。readyState：存有XMLHttpRequest的状态。从 0 到 4 发生变化，值对应的状态如下：
0: 请求未初始化
1: 服务器连接已建立
2: 请求已接收
3: 请求处理中
4: 请求已完成，且响应已就绪

status：记录着响应的状态 200: "OK"、404: 未找到页面等等

promise
接下来谈谈promise这个对象，用来干啥的，简单说就是对JS异步编程的一种能力加持，让异步代码的编写及阅读不再那么的恶心人。我写一段恶心人的简单实例：

setTimeout(()=>{
 setTimeout(()=>{
  setTimroiut(()=>{
   ......
  },1000)
    },1000)
},1000)
你可能会说，啊这还好呀，这有啥。这只是我的一个简单举例，当在真实场景下无论前后端，不断的异步嵌套的时候，你定会抓狂。那么，Promise是如何改善这种问题的尼？
将Promise理解为状态扭转机，类似于我们在学习编译原理里面的有限状态自动机，会由一个状态过渡到另一个状态，并且状态是会结束的，有终点的。promise具有三种状态，pending、resolve、reject。状态的扭转跟随着两过程：
pending ----> resolve
pending ----> reject
也就是从处理开始等待到成功处理或者从处理开始等待待处理失败的过程。来看我直接markdown手撸把上面的嵌套promise化，让代码看起来更加直观，流程更加清晰，让其顺序执行

console.time();
new Promise((resolve,reject)=>{
 setTimeout(()=>{
  if(Math.random() > 0.5){
   resolve("timer1");
  }else{
   reject("timer1 error");
  }
  
 },1000)
}).then((data)=>{
 console.log(data);
 return new Promise((resolve, reject)=>{
  setTimeout(()=>{
   if(Math.random() > 0.5){
    resolve("timer2");
   }else{
    reject("timer2 error");
   }
  },2000)
 })
 
}).then((data)=>{
 console.log(data);
 return new Promise((resolve, reject)=>{
  setTimeout(()=>{
   if(Math.random() > 0.5){
    resolve("timer3");
   }else{
    reject("timer3 error");
   }
  },2000)
 })
}).catch((err)=>{
 console.log(err);
})
console.timeEnd()
如果你的运气够好，每次都能random在0.5以下，那你就能看到 timer1，timer2，运气不好直接就timer1 error了。为什么我在每个then里面都new了promise？？？感到奇怪？那就去看看mdn，then需要返回一个peomise对象。
![image](/images/2.png)


promise还提供了race、all等静态方法，用于串行或者并行执行，需要的自己下来看看，你也可以试试手撸一个promise，那你就很牛了。

fetch
讲到了fetch，我说fetch其实就是ajax加一层promise的封装，你赞同么？如果你赞同并且说这是一种polyfill，那我觉得你是完全懂fetch的使用方法并且明了其实现过程的，与其说fetch是promise对ajax进行了封装，说的更准确一些的就是fetch是和ajax一样同样使用XMLHTTPRequest对象实现的各种http请求，然后使用promise进行了包装使用.then.catch实现调用，但是其又与ajax稍有不同，为什么尼？因为其有属于自己的实现方案，fetch的源码，一共500多行，重点在其实现的几个对象：Headers, Body, Request, Response，根据这几个对象的命名就可以知道是对http结构及请求响应过程中的数据及方法的封装，最后进行fetch方法的封装。网上看到一个博主放了一段简化的代码，如下：

(function (self) {
  'use strict';
  if (self.fetch) {
     return
  }
 
  // 封装的 Headers，支持的方法参考https://developer.mozilla.org/en-US/docs/Web/API/Headers
  function Headers(headers) {
    ......
  }  
 
  //方法参考：https://developer.mozilla.org/en-US/docs/Web/API/Body
  function Body() {
    ......
  }
 
  // 请求的Request对象 ，https://developer.mozilla.org/en-US/docs/Web/API/Request
  // cache,context,integrity,redirect,referrerPolicy 在MDN定义中是存在的
  function Request(input, options) {
     ......
  }
 
  Body.call(Request.prototype)  //把Body方法属性绑到 Reques.prototype
 
  function Response(bodyInit, options) {
  }
 
  Body.call(Response.prototype) //把Body方法属性绑到 Reques.prototype
 
  self.Headers = Headers  //暴露Headers
  self.Request = Request //暴露Request
  self.Response = Response //暴露Response
 
  self.fetch = function (input, init) {
    return new Promise(function (resolve, reject) {
      var request = new Request(input, init)  //初始化request对象
      var xhr = new XMLHttpRequest()  // 初始化 xhr
 
      xhr.onload = function () { //请求成功，构建Response，并resolve进入下一阶段
        var options = {
          status: xhr.status,
          statusText: xhr.statusText,
          headers: parseHeaders(xhr.getAllResponseHeaders() || '')
        }
        options.url = 'responseURL' in xhr ? xhr.responseURL : options.headers.get('X-Request-URL')
        var body = 'response' in xhr ? xhr.response : xhr.responseText
        resolve(new Response(body, options))
      }
 
      //请求失败，构建Error，并reject进入下一阶段
      xhr.onerror = function () {
        reject(new TypeError('Network request failed'))
      }
 
      //请求超时，构建Error，并reject进入下一阶段
      xhr.ontimeout = function () {
        reject(new TypeError('Network request failed'))
      }
 
      // 设置xhr参数
      xhr.open(request.method, request.url, true)
 
      // 设置 credentials
      if (request.credentials === 'include') {
        xhr.withCredentials = true
      } else if (request.credentials === 'omit') {
        xhr.withCredentials = false
      }
 
      // 设置 responseType
      if ('responseType' in xhr && support.blob) {
        xhr.responseType = 'blob'
      }
 
      // 设置Header
      request.headers.forEach(function (value, name) {
        xhr.setRequestHeader(name, value)
      })
      // 发送请求
      xhr.send(typeof request._bodyInit === 'undefined' ? null : request._bodyInit)
    })
  }
  //标记是fetch是polyfill的，而不是原生的
  self.fetch.polyfill = true
})(typeof self !== 'undefined' ? self : this); // IIFE函数的参数，不用window，web worker, service worker里面也可以使用
fetch本质意义上还是对XMLHttpRequest的请求封装。

axios
谈到axios时，这是我们经常会使用的一个网络请求工具，易用且强大，提供了多种框架的版本，真是好，我还是贴一下axios的特性
1.从浏览器中创建XMLHttpRequests
2.从node.js创建http请求
3.支持PromiseAPI
4.拦截请求和响应
5.转换请求数据和响应数据
6.取消请求
7.自动转换JSON数据
8.客户端支持防御XSRF
axios的源码我还没看完，没有资格评论源码级别的实现，但其就是promise对ajax的封装，利用promise的能力来实现并行与串行的请求等，拦截器是一个很好的特性，允许我们在请求开始之前或者响应开始之前做一些我们想做的操作，这给业务层面的实现又多了其他的实现方案，说实话，我一开始一直觉得axios拦截器的实现是通过捕捉ajax的几种状态值并做响应的方法处理实现的，但是在阅读部分源码后我发现不是，原理是axios维护了一个拦截器方法数组，其实就是自定义方法，并使用use方法依次将拦截器数组里面的自定方法放在请求之前，利用promise形成链式调用，也就是顺序调用来实现的。

I/O
I/O 背景
为啥又要讲I/O了，事情是这样的，上午时候跟一个自己尊敬的老师好友对一款桌面软件的实现方案进行了讨论，大致就是一款图形处理兼AI推理的windows桌面应用，业务方要求必须使用c++完成所有的前后端开发工作，并且算子服务必须函数化在本地，于是我就开始了思考，给出了两个实现方案，如下：
![image](/images/3.png)


方案的思考过程中，我比较关注的两个点是实现成本及软件性能，之前写过一篇对于端侧性能的思考，尤其是这类需要做图像处理，AI推理的应用，性能要求是极高的，当然，毕竟跑在windows上，而不是一些手机等终端上，GPU的调用十分的方便，性能也不会差到哪儿去，然后又基于实现方案的成本及可维护性做了思考，才给出了以上的两种方案，谈及到了I/O，这激起了我对异步I/O的思考。

异步I/O
说到异步编程，我相信这对于除开javascript以外的众多语言都是比较陌生的，而且在绝大多数高级编程语言中，异步貌似不太被采纳，大多数都喜欢利用从头到脚同步阻塞的方式来执行，例如：PHP（哈哈哈，天下最好的语言），当然这样利于逻辑的理解与实现，但是在复杂的应用中，阻塞那是不能想象的，可能你会说，我多线程啊，再进行线程通信不就好了，继续看，看到这篇博客的人，一定是用过nginx的，为什么nginx会有如此高的性能表现，这得益于事件驱动，异步I/O的架构设计，在进行I/O的过程中，不用等待着上一个I/O执行完成才进行下一个步骤的执行。而Node的设计就与Nginx的事件驱动、异步I/O架构不能说一模一样，但也90%一样了。

前面我在吐槽PHP没有异步I/O不能承载高并发应用时，PHPer会说，我为什么不能多线程并行执行来提高并发的能力，为什么非要异步I/O尼，无可厚非，如果创建多线程的开销小于并行执行的开销，那肯定多线程是首选，但是多线程就意味着你需要创建多个线程并且在多个线程之间来回切换会带来巨大的开销，通常来说，遇见比较复杂的应用，你还需要做到锁的应用，以及状态同步等问题，那么，异步I/O就能很好的解决这样的问题，利用单线程，远离多线层死锁、状态同步等问题，远离阻塞，更好的利用CPU等。

异步I/O的执行过程
画了张图如下：
![image](/images/4.jpg)

如上图，异步I/O的精髓在于，单线程进行I/O处理时，并不会阻塞其它指令的执行，在完成处理后，通过callback完成数据返回。虽然上文一直在说异步与同步，但是对于操作系统内核来说，处理I/O只有阻塞与非阻塞之分，阻塞I/O就是调用I/O处理之后一定要等到系统内核层面完成操作后，调用才结束，才能往下执行，这就会造成CPU等待I/O执行完成，造成CPU使用空余，效率低下，而操作系统利用轮询实现的非阻塞I/O能够通过一系列机制来改善阻塞I/O，提升CPU使用效率及性能，这里我画图总结一下。应该来的更明显直观：

![image](/images/4.png)
![image](/images/5.png)



操作系统通过轮询实现的非阻塞调用，能够在貌似异步的情况下取的完整的I/O数据，但是对于应用程序而言，依然是同步的，因为CPU还是需要等待I/O的完全返回。理想的非阻塞异步I/O应该是应用程序发起非阻塞调用，而不是通过一些轮询，遍历机制来实现一种伪异步，现实中，实现真正的应用程序调用异步，使用了线层池方案，通过让部分线程进行阻塞I/O或者非阻塞I/O加轮询完成数据的获取，让一个线程来进行计算处理，通过线程间通信将I/O线程获取的数据进行传递，实现异步I/O。这也是Node实现的异步I/O的底层方案。
![image](/images/6.png)


Node 异步I/O
当然，Node实现了跨平台的异步I/O，开发者在*nix及windwos异步I/O之上实现了Libuv来实现跨平台，linux平台自己实现了异步I/O，windows则采用了iocp实现，所以，我总说Node是机制创新，底层还是得益于C++的线程池异步I/O带来的高性能

![image](/images/7.png)
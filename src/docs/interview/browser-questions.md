---
layout: doc
---

## 从输入 URL 到页面加载的全过程

1. 首先在浏览器中输入 URL
2. 查找缓存：浏览器先查看浏览器缓存-系统缓存-路由缓存中是否有该地址页面，如果有则显示页面内容。如果没有则进行下一步。

   - 浏览器缓存：浏览器会记录 DNS 一段时间，因此，只是第一个地方解析 DNS 请求；
   - 操作系统缓存:如果在浏览器缓存中不包含这个记录，则会使系统调用操作系统， 获取操作系统的记录(保存最近的 DNS 查询缓存)；
   - 路由器缓存：如果上述两个步骤均不能成功获取 DNS 记录，继续搜索路由器缓存；
   - ISP 缓存：若上述均失败，继续向 ISP 搜索。

3. DNS 域名解析：浏览器向 DNS 服务器发起请求，解析该 URL 中的域名对应的 IP 地址。DNS 服务器是基于 UDP 的，因此会用到 UDP 协议。
4. 建立 TCP 连接：解析出 IP 地址后，根据 IP 地址和默认 80 端口，和服务器建立 TCP 连接
5. 发起 HTTP 请求：浏览器发起读取文件的 HTTP 请求，，该请求报文作为 TCP 三次握手的第三次数据发送给服务器
6. 服务器响应请求并返回结果：服务器对浏览器请求做出响应，并把对应的 html 文件发送给浏览器
7. 关闭 TCP 连接：通过四次挥手释放 TCP 连接
8. 浏览器渲染：客户端（浏览器）解析 HTML 内容并渲染出来，浏览器接收到数据包后的解析流程为：

   - 构建 DOM 树：词法分析然后解析成 DOM 树（dom tree），是由 dom 元素及属性节点组成，树的根是 document 对象
   - 构建 CSS 规则树：生成 CSS 规则树（CSS Rule Tree）
   - 构建 render 树：Web 浏览器将 DOM 和 CSSOM 结合，并构建出渲染树（render tree）
   - 布局（Layout）：计算出每个节点在屏幕中的位置
   - 绘制（Painting）：即遍历 render 树，并使用 UI 后端层绘制每个节点。

![dom渲染](/interview/browser/dom渲染.png)

9. JS 引擎解析过程：调用 JS 引擎执行 JS 代码（JS 的解释阶段，预处理阶段，执行阶段生成执行上下文，VO，作用域链、回收机制等等）

   - 创建 `window` `对象：window` 对象也叫全局执行环境，当页面产生时就被创建，所有的全局变量和函数都属于 window 的属性和方法，而 DOM Tree 也会映射在 window 的 doucment 对象上。当关闭网页或者关闭浏览器时，全局执行环境会被销毁。

   - 加载文件：完成 js 引擎分析它的语法与词法是否合法，如果合法进入预编译
   - 预编译：在预编译的过程中，浏览器会寻找全局变量声明，把它作为 `window` 的属性加入到 `window` 对象中，并给变量赋值为'undefined'；寻找全局函数声明，把它作为 `window` 的方法加入到 `window` 对象中，并将函数体赋值给他（匿名函数是不参与预编译的，因为它是变量）。而变量提升作为不合理的地方在 ES6 中已经解决了，函数提升还存在。
   - 解释执行：执行到变量就赋值，如果变量没有被定义，也就没有被预编译直接赋值，在 ES5 非严格模式下这个变量会成为 `window` 的一个属性，也就是成为全局变量。string、int 这样的值就是直接把值放在变量的存储空间里，object 对象就是把指针指向变量的存储空间。函数执行，就将函数的环境推入一个环境的栈中，执行完成后再弹出，控制权交还给之前的环境。JS 作用域其实就是这样的执行流机制实现的。

## 重绘与重排的区别

**重排/回流（Reflow）**：当 DOM 的变化影响了元素的几何信息，浏览器需要重新计算元素的几何属性，将其安放在界面中的正确位置，这个过程叫做重排。表现为重新生成布局，重新排列元素。

**重绘(Repaint)**: 当一个元素的外观发生改变，但没有改变布局,重新把元素外观绘制出来的过程，叫做重绘。表现为某些元素的外观被改变

『重绘』不一定会出现『重排』，『重排』必然会出现『重绘』。

## 如何触发重排和重绘

**触发重排** :one: 页面的首次渲染 :two: 浏览器的窗口大小发生变化 :three: 元素的内容发生变化 :four: 元素的尺寸或者位置发生变化 :five: 元素的字体大小发生变化 激活 CSS 伪类 查询某些属性或者调用某些方法 :six: 添加或者删除可见的 DOM 元素

**触发重绘**

:one: `color`、`background` 相关属性：`background-color`、`background-image` 等

:two: `outline` 相关属性 ： `outline-color` 、 `outline-width` 、 `text-decoration` `border-radius`、`visibility`、`box-shadow`

## 如何避免重绘或者重排

减少回流与重绘的措施：

1. 操作 DOM 时，尽量在低层级的 DOM 节点进行操作 不要使用 table 布局， 一个小的改动可能会使整个 table 进行重新
2. 布局使用 CSS 的表达式不要频繁操作元素的样式，对于静态页面，可以修改类名，而不是样式
3. 避免频繁操作 DOM，可以创建一个文档片段 documentFragment，在它 上面应用所有 DOM 操作，最后再把它添加到文档中
4. 将元素先设置 display: none，操作结束后再把它显示出来。因为在 display 属性为 none 的元素上进行的 DOM 操作不会引发回流和重绘。 将 DOM 的多个读操作（或者写操作）放在一起，而不是读写操作穿插 着写。这得益于浏览器的渲染队列机制。

## 如何优化动画

对于如何优化动画，我们知道，一般情况下，动画需要频繁的操作 DOM，就就会导致页面的性能问题，我们可以将动画的 position 属性 设置为 absolute 或者 fixed，将动画脱离文档流，这样他的回流就 不会影响到页面了。

## 对节流与防抖的理解

**函数防抖**是指在事件被触发 n 秒后再执行回调，如果在这 n 秒内事 件又被触发，则重新计时。这可以使用在一些点击请求的事件上，避 免因为用户的多次点击向后端发送多次请求。 比如：搜索框搜索输入，并在输入完以后自动搜索、手机号，邮箱验证输入检测、窗口大小 resize 变化后，再重新渲染。

```javascript
/**
 * 防抖函数  一个需要频繁触发的函数，在规定时间内，只让最后一次生效，前面的不生效
 * @param fn要被节流的函数
 * @param delay规定的时间
 */
function debounce(fn, delay) {
  //记录上一次的延时器
  var timer = null;
  return function () {
    //清除上一次的演示器
    clearTimeout(timer);
    //重新设置新的延时器
    timer = setTimeout(() => {
      //修正this指向问题
      fn.apply(this);
    }, delay);
  };
}
document.getElementById("btn").onclick = debounce(function () {
  console.log("按钮被点击了" + Date.now());
}, 1000);
```

**函数节流**是指规定一个单位时间，在这个单位时间内，只能有一次触 发事件的回调函数执行，如果在同一个单位时间内某事件被触发多次， 只有一次能生效。节流可以使用在 scroll 函数的事件监听上，通过 事件节流来降低事件调用的频率。 比如：滚动加载更多、搜索框搜的索联想功能、高频点击、表单重复提交……

```javascript
/**
 * 节流函数 一个函数执行一次后，只有大于设定的执行周期才会执行第二次。有个需要频繁触发的函数，出于优化性能的角度，在规定时间内，只让函数触发的第一次生效，后面的不生效。
 * @param fn要被节流的函数
 * @param delay规定的时间
 */
function throttle(fn, delay) {
  //记录上一次函数触发的时间
  var lastTime = 0;
  return function () {
    //记录当前函数触发的时间
    var nowTime = Date.now();
    if (nowTime - lastTime > delay) {
      //修正this指向问题
      fn.call(this);
      //同步执行结束时间
      lastTime = nowTime;
    }
  };
}
document.onscroll = throttle(function () {
  console.log("scllor事件被触发了" + Date.now());
}, 200);
```

## 介绍下 304 过程

- 浏览器请求资源时首先命中资源的 Expires 和 Cache-Control，Expires 受限于本地时间，如果修改了本地时间，可能会造成缓存失效，可以通过 Cache-control: max-age 指定最大生命周期，状态仍然返回 200，但不会请求数据，在浏览器中能明显看到 from cache 字样。
- 强缓存失效，进入协商缓存阶段，首先验证 ETagETag 可以保证每一个资源是唯一的，资源变化都会导致 ETag 变化。服务器根据客户端上送的 If-None-Match 值来判断是否命中缓存。
- 协商缓存 Last-Modify/If-Modify-Since 阶段，客户端第一次请求资源时，服务服返回的 header 中会加上 Last-Modify，Last-modify 是一个时间标识该资源的最后修改时间。再次请求该资源时，request 的请求头中会包含 If-Modify-Since，该值为缓存之前返回的 Last-Modify。服务器收到 If-Modify-Since 后，根据资源的最后修改时间判断是否命中缓存。

## 浏览器的缓存机制 强制缓存 && 协商缓存

浏览器与服务器通信的方式为应答模式，即是：浏览器发起 HTTP 请求 – 服务器响应该请求。那么浏览器第一次向服务器发起该请求后拿到请求结果，会根据响应报文中 HTTP 头的缓存标识，决定是否缓存结果，是则将请求结果和缓存标识存入浏览器缓存中，简单的过程如下图：

![cache](/interview/browser/cache.png)

由上图我们可以知道：

浏览器每次发起请求，都会先在浏览器缓存中查找该请求的结果以及缓存标识

浏览器每次拿到返回的请求结果都会将该结果和缓存标识存入浏览器缓存中

以上两点结论就是浏览器缓存机制的关键，他确保了每个请求的缓存存入与读取，只要我们再理解浏览器缓存的使用规则，那么所有的问题就迎刃而解了。为了方便理解，这里根据是否需要向服务器重新发起 HTTP 请求将缓存过程分为两个部分，分别是强制缓存和协商缓存。

:one: 强制缓存

强制缓存就是向浏览器缓存查找该请求结果，并根据该结果的缓存规则来决定是否使用该缓存结果的过程。当浏览器向服务器发起请求时，服务器会将缓存规则放入 HTTP 响应报文的 HTTP 头中和请求结果一起返回给浏览器，控制强制缓存的字段分别是 `Expires` 和 `Cache-Control`，其中 `Cache-Control` 优先级比 `Expires` 高。
强制缓存的情况主要有三种(暂不分析协商缓存过程)，如下：

1. 不存在该缓存结果和缓存标识，强制缓存失效，则直接向服务器发起请求（跟第一次发起请求一致）。
2. 存在该缓存结果和缓存标识，但该结果已失效，强制缓存失效，则使用协商缓存。
3. 存在该缓存结果和缓存标识，且该结果尚未失效，强制缓存生效，直接返回该结果

:two: 协商缓存

协商缓存就是强制缓存失效后，浏览器携带缓存标识向服务器发起请求，由服务器根据缓存标识决定是否使用缓存的过程，同样，协商缓存的标识也是在响应报文的 HTTP 头中和请求结果一起返回给浏览器的，控制协商缓存的字段分别有：`Last-Modified / If-Modified-Since` 和 `Etag / If-None-Match`，其中 `Etag / If-None-Match` 的优先级比 `Last-Modified / If-Modified-Since` 高。协商缓存主要有以下两种情况：

1. 协商缓存生效，返回 304
2. 协商缓存失效，返回 200 和请求结果结果

## 点击刷新按钮或者按 F5、按 Ctrl+F5 （强制刷新）、地址 栏回车有什么区别

- 点击刷新按钮或者按 F5：浏览器直接对本地的缓存文件过期，但是 会带上 If-Modifed-Since，If-None-Match，这就意味着服务器会对 文件检查新鲜度，返回结果可能是 304，也有可能是 200。
- 用户按 Ctrl+F5（强制刷新）：浏览器不仅会对本地文件过期，而且 不会带上 If-Modifed-Since，If-None-Match，相当于之前从来没有 请求过，返回结果是 200。
- 地址栏回车： 浏览器发起请求，按照正常流程，本地检查是否过期， 然后服务器检查新鲜度，最后返回内容。

## 渲染过程中遇到 JS 文件如何处理

JavaScript 的加载、解析与执行会阻塞文档的解析，也就是说，在构建 DOM 时，HTML 解析器若遇到了 JavaScript，那么它会暂停文档的解析，将控制权移交给 JavaScript 引擎，等 JavaScript 引擎运行完毕，浏览器再从中断的地方恢复继续解析文档。也就是说，如果想要首屏渲染的越快，就越不应该在首屏就加载 JS 文件，这也是都建议将 script 标签放在 body 标签底部的原因。当然在当下，并不是说 script 标签必须放在底部，因为你可以给 script 标签添加 defer 或者 async 属性。

## script 标签中 defer 和 async 的区别

如果没有 defer 或 async 属性，浏览器会立即加载并执行相应的脚本。 它不会等待后续加载的文档元素，读取到就会开始加载和执行，这样 就阻塞了后续文档的加载。

![scriptDefer](/interview/browser/scriptDefer.jpeg)

其中蓝色代表 js 脚本网络加载时间，红色代表 js 脚本执行时间，绿 色代表 html 解析。

defer 和 async 属性都是去异步加载外部的 JS 脚本文件，它们都不 会阻塞页面的解析，其区别如下：

- 执行顺序：多个带 async 属性的标签，不能保证加载的顺序；多个带 defer 属性的标签，按照加载顺序执行；
- 脚本是否并行执行：async 属性，表示后续文档的加载和执行与 js 脚本的加载和执行是并行进行的，即异步执行；defer 属性，加载后 续文档的过程和 js 脚本的加载(此时仅加载不执行)是并行进行的(异步)，js 脚本需要等到文档所有元素解析完成之后才执行， DOMContentLoaded 事件触发执行之前。

## 介绍一下事件模型

- DOM0 级事件模型，这种模型不会传播，所以没有事件流的概念，但是现在有的浏览器支持以冒泡的方式实现，它可以在网页中直接定义监听函数，也可以通过 js 属性来指定监听函数。所有浏览器都兼容这种方式。直接在 dom 对象上注册事件名称，就是 DOM0 写法。
- IE 事件模型，在该事件模型中，一次事件共有两个过程，事件处理阶段和事件冒泡阶段。事件处理阶段会首先执行目标元素绑定的监听事件。然后是事件冒泡阶段，冒泡指的是事件从目标元素冒泡到 document，依次检查经过的节点是否绑定了事件监听函数，如果有则执行。这种模型通过 attachEvent 来添加监听函数，可以添加多个监听函数，会按顺序依次执行。
- DOM2 级事件模型，在该事件模型中，一次事件共有三个过程，第一个过程是事件捕获阶段。捕获指的是事件从 document 一直向下传播到目标元素，依次检查经过的节点是否绑定了事件监听函数，如果有则执行。后面两个阶段和 IE 事件模型的两个阶段相同。这种事件模型，事件绑定的函数是 addEventListener，其中第三个参数可以指定事件是否在捕获阶段执行。（默认 false 为冒泡模式）

```javascript
node.addEventListener(
  "click",
  event => {
    // 阻止冒泡
    event.stopPropagation();
    // 阻止触发默认操作 比如 a 标签跳转
    event.preventDefault();
    console.log("冒泡");
  },
  false
);
```

::: tip

1. 捕获阶段：事件从 `document` 一直向下传播到目标元素，依次检查经过的节点是否绑定了事件监听函数，如果有则执行。

2. 目标阶段：会首先执行目标元素绑定的监听事件。

3. 冒泡阶段：从目标元素冒泡到 `document`，依次检查经过的节点是否绑定了事件监听函数，如果有则执行

:::

## 对事件委托的理解

事件委托本质上是利用了浏览器事件冒泡的机制。因为事件在冒泡过程中会上传到父节点，父节点可以通过事件对象获取到目标节点，因此可以把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件，这种方式称为事件委托（事件代理）。

## 事件触发的过程是怎样的

事件触发有三个阶段：

- window 往事件触发处传播，遇到注册的捕获事件会触发
- 传播到事件触发处时触发注册的事件
- 从事件触发处往 window 传播，遇到注册的冒泡事件会触发

事件触发一般来说会按照上面的顺序进行，**但是也有特例，如果给一个 body 中的子节点同时注册冒泡和捕获事件，事件触发会按照注册的顺序执行**。

## 当一个按钮被点击时，发生了什么

1. 移动到按钮上时，会依次触发 `mouseover`，`mouseenter` 事件，前者冒泡，后者不冒泡
2. 鼠标按下时，广播 `mousedown` 事件
3. 如果此时其它元素有焦点，那么该元素会先失去焦点，并广播 `blur` 事件
4. 如果失去焦点的元素是 `<input>`，`<textarea>`，且它们的内容有变化，则它们会广播 `change` 事件
5. 按钮获得焦点，广播 `focus` 事件
6. 如果因此影响到 `DOM`，那么会等待 `DOM` 变更
7. 如果鼠标没有离开按钮，按钮广播 `mouseup` 事件
8. 最后广播 `click` 事件

需要注意的是，因为` （3）``（4）``（5） `和 `Event loop` 的存在，`DOM` 可能在这个时间点出现变化，导致一些操作没有办法按照预期完成，比如：

1. 一个搜索框
2. 输入的时候自动搜索，结果以 `dropdown` 形式展示在下面
3. 点击 `dropdown` 里的条目可以跳转
4. 搜索框 `blur` 时 `dropdown` 移除
5. 此时，`dropdown` 可能无法点击

## 对事件循环的理解

因为 js 是单线程运行的，在代码执行时，通过将不同函数的执行上下文压入执行栈中来保证代码的有序执行。在执行同步代码时，如果遇到异步事件，js 引擎并不会一直等待其返回结果，而是会将这个事件挂起，继续执行执行栈中的其他任务。当异步事件执行完毕后，再将异步事件对应的回调加入到一个任务队列中等待执行。任务队列可以分为宏任务队列和微任务队列，当当前执行栈中的事件执行完毕后，js 引擎首先会判断微任务队列中是否有任务可以执行，如果有就将微任务队首的事件压入栈中执行。当微任务队列中的任务都执行完成后再去执行宏任务队列中的任务。

![eventLoop](/interview/browser/eventLoop.jpeg)

Event Loop 执行顺序如下所示：

- 首先执行同步代码，这属于宏任务
- 当执行完所有同步代码后，执行栈为空，查询是否有异步代码需要执行
- 执行所有微任务
- 当执行完所有微任务后，如有必要会渲染页面
- 然后开始下一轮 Event Loop，执行宏任务中的异步代码

## 宏任务和微任务分别有哪些

- 微任务包括： promise 的回调、node 中的 process.nextTick 、对 Dom 变化监听的 MutationObserver。
- 宏任务包括： script 脚本的执行、setTimeout ，setInterval ，setImmediate 一类的定时事件，还有如 I/O 操作、UI 渲染等。

## 前端渲染和后端渲染的区别

**前端渲染**： 指的是后端返回 `JSON` 数据，前端利用预先写的 `html` 模板，循环读取 `JSON` 数据，拼接字符串，并插入页面。

- **好处**：网络传输数据量小。不占用服务端运算资源（解析模板），模板在前端（很有可能仅部分在前端），改结构变交互都前端自己来了，改完自己调就行。
- **坏处**：前端耗时较多。前端代码较多，因为部分以前在后台处理的交互逻辑交给了前端处理。占用少部分客户端运算资源用于解析模板。

**后端渲染**： 前端请求，后端用后台模板引擎直接生成 `html`，前端接受到数据之后，直接插入页面。

- **好处**：前端耗时少，即减少了首屏时间，模板统一在后端。前端（相对）省事，不占用客户端运算资源（解析模板）,`seo` 比较友好
- **坏处**：占用服务器资源。

## 什么是执行栈

可以把执行栈认为是一个存储函数调用的栈结构，遵循先进后出的原则。

![bashStack](/interview/browser/bashStack.gif)

当开始执行 JS 代码时，根据先进后出的原则，后执行的函数会先弹出栈，可以看到，foo 函数后执行，当执行完毕后就从栈中弹出了。

## Cookie、LocalStorage、SessionStorage 区别

- cookie： 其实最开始是服务器端用于记录用户状态的一种方式，由服务器设置，在客户端存储，然后每次发起同源请求时，发送给服务器端。cookie 最多能存储 4 k 数据，它的生存时间由 expires 属性指定，并且 cookie 只能被同源的页面访问共享。

- sessionStorage： html5 提供的一种浏览器本地存储的方法，它借鉴了服务器端 session 的概念，代表的是一次会话中所保存的数据。它一般能够存储 5M 或者更大的数据，它在当前窗口关闭后就失效了，并且 sessionStorage 只能被同一个窗口的同源页面所访问共享。

- localStorage： html5 提供的一种浏览器本地存储的方法，它一般也能够存储 5M 或者更大的数据。它和 sessionStorage 不同的是，除非手动删除它，否则它不会失效，并且 localStorage 也只能被同源页面所访问共享。

上面几种方式都是存储少量数据的时候的存储方式，当需要在本地存储大量数据的时候，我们可以使用浏览器的 indexDB 这是浏览器提供的一种本地的数据库存储机制。它不是关系型数据库，它内部采用对象仓库的形式存储数据，它更接近 NoSQL 数据库。

## 懒加载的实现原理

图片的加载是由 `src` 引起的，当对 `src` 赋值时，浏览器就会请求图片 资源。根据这个原理，我们使用 `HTML5` 的 `data-xxx` 属性来储存图片 的路径，在需要加载图片的时候，将 `data-xxx` 中图片的路径赋值给 `src`，这样就实现了图片的按需加载，即懒加载。 注意：`data-xxx` 中的 `xxx` 可以自定义，这里我们使用 data-src 来定 义。懒加载的实现重点在于确定用户需要加载哪张图片，在浏览器中，可 视区域内的资源就是用户需要的资源。所以当图片出现在可视区域时， 获取图片的真实地址并赋值给图片即可

```html
<div>
  <img src="loading.gif" data-src="pic.png" />
  <img src="loading.gif" data-src="pic.png" />
  <img src="loading.gif" data-src="pic.png" />
  <img src="loading.gif" data-src="pic.png" />
  <img src="loading.gif" data-src="pic.png" />
</div>
<script>
  var imgs = document.querySelectAll(img);
  function lazyLoad() {
    var scrollTop = document.body.scrollTop; // 滚动过的距离高度
    var innerHeight = window.innerHeight; // 可视区域高度
    for (let i = 0; i < imgs.length; i++) {
      // 元素顶部距离文档顶部的高度 - （滚动过的距离高度 + 可视区域高度）
      if (imgs[i].offsetTop < scrollTop + innerHeight) {
        imgs[i].src = imgs[i].getAttribute("data-src");
      }
    }
  }
  window.onscroll = lazyLoad();
</script>
```

## 什么是 XSS 攻击

### 概念

XSS 攻击指的是跨站脚本攻击，是一种代码注入攻击。攻击者通过在网站注入恶意脚本，使之在用户的浏览器上运行，从而盗取用户的信息如 cookie 等。

XSS 的本质是因为网站没有对恶意代码进行过滤，与正常的代码混合在一起了，浏览器没有办法分辨哪些脚本是可信的，从而导致了恶意代码的执行。

攻击者可以通过这种攻击方式可以进行以下操作：

- 获取页面的数据，如 DOM、cookie、localStorage；
- DOS 攻击，发送合理请求，占用服务器资源，从而使用户无法访问服务器；
- 破坏页面结构；
- 流量劫持（将链接指向某网站）；

### 攻击类型

XSS 可以分为存储型、反射型和 DOM 型：

- 存储型指的是恶意脚本会存储在目标服务器上，当浏览器请求数据时，脚本从服务器传回并执行。
- 反射型指的是攻击者诱导用户访问一个带有恶意代码的 URL 后，服务器端接收数据后处理，然后把带有恶意代码的数据发送到浏览器端，浏览器端解析这段带有 XSS 代码的数据后当做脚本执行，最终完成 XSS 攻击。
- DOM 型指的通过修改页面的 DOM 节点形成的 XSS。

#### 存储型 XSS 的攻击步骤：

1. 攻击者将恶意代码提交到⽬标⽹站的数据库中。
2. ⽤户打开⽬标⽹站时，⽹站服务端将恶意代码从数据库取出，拼接在 HTML 中返回给浏览器。
3. ⽤户浏览器接收到响应后解析执⾏，混在其中的恶意代码也被执⾏。
4. 恶意代码窃取⽤户数据并发送到攻击者的⽹站，或者冒充⽤户的⾏为，调⽤⽬标⽹站接⼝执⾏攻击者指定的操作。

这种攻击常⻅于带有⽤户保存数据的⽹站功能，如论坛发帖、商品评论、⽤户私信等。

#### 反射型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. ⽤户打开带有恶意代码的 URL 时，⽹站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器。
3. ⽤户浏览器接收到响应后解析执⾏，混在其中的恶意代码也被执⾏。
4. 恶意代码窃取⽤户数据并发送到攻击者的⽹站，或者冒充⽤户的⾏为，调⽤⽬标⽹站接⼝执⾏攻击者指定的操作。

反射型 XSS 跟存储型 XSS 的区别是：存储型 XSS 的恶意代码存在数据库⾥，反射型 XSS 的恶意代码存在 URL ⾥。

反射型 XSS 漏洞常⻅于通过 URL 传递参数的功能，如⽹站搜索、跳转等。 由于需要⽤户主动打开恶意的 URL 才能⽣效，攻击者往往会结合多种⼿段诱导⽤户点击。

DOM 型 XSS 的攻击步骤：

#### DOM 型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. ⽤户打开带有恶意代码的 URL。
3. ⽤户浏览器接收到响应后解析执⾏，前端 JavaScript 取出 URL 中的恶意代码并执⾏。
4. 恶意代码窃取⽤户数据并发送到攻击者的⽹站，或者冒充⽤户的⾏为，调⽤⽬标⽹站接⼝执⾏攻击者指定的操作。

DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执⾏恶意代码由浏览器端完成，属于前端 JavaScript ⾃身的安全漏洞，⽽其他两种 XSS 都属于服务端的安全漏洞。

## 如何防御 XSS 攻击

- 可以从浏览器的执行来进行预防，一种是使用纯前端的方式，不用服务器端拼接后返回（不使用服务端渲染）。另一种是对需要插入到 HTML 中的代码做好充分的转义。对于 DOM 型的攻击，主要是前端脚本的不可靠而造成的，对于数据获取渲染和字符串拼接的时候应该对可能出现的恶意代码情况进行判断。
- 使用 CSP ，CSP 的本质是建立一个白名单，告诉浏览器哪些外部资源可以加载和执行，从而防止恶意代码的注入攻击。
- 对一些敏感信息进行保护，比如 cookie 使用 http-only，使得脚本无法获取。也可以使用验证码，避免脚本伪装成用户执行一些操作。

## 什么是 CSRF 攻击

### 概念

CSRF 攻击指的是跨站请求伪造攻击，攻击者诱导用户进入一个第三方网站，然后该网站向被攻击网站发送跨站请求。如果用户在被攻击网站中保存了登录状态，那么攻击者就可以利用这个登录状态，绕过后台的用户验证，冒充用户向服务器执行一些操作。

CSRF 攻击的本质是利用 cookie 会在同源请求中携带发送给服务器的特点，以此来实现用户的冒充。

### 攻击类型

- GET 类型的 CSRF 攻击，比如在网站中的一个 img 标签里构建一个请求，当用户打开这个网站的时候就会自动发起提交。
- POST 类型的 CSRF 攻击，比如构建一个表单，然后隐藏它，当用户进入页面时，自动提交这个表单。
- 链接类型的 CSRF 攻击，比如在 a 标签的 href 属性里构建一个请求，然后诱导用户去点击。

## 如何防御 CSRF 攻击

- 进行同源检测，服务器根据 http 请求头中 origin 或者 referer 信息来判断请求是否为允许访问的站点，从而对请求进行过滤。当 origin 或者 referer 信息都不存在的时候，直接阻止请求。这种方式的缺点是有些情况下 referer 可以被伪造，同时还会把搜索引擎的链接也给屏蔽了。所以一般网站会允许搜索引擎的页面请求，但是相应的页面请求这种请求方式也可能被攻击者给利用。（Referer 字段会告诉服务器该网页是从哪个页面链接过来的）
- 使用 CSRF Token 进行验证，服务器向用户返回一个随机数 Token ，当网站再次发起请求时，在请求参数中加入服务器端返回的 token ，然后服务器对这个 token 进行验证。这种方法解决了使用 cookie 单一验证方式时，可能会被冒用的问题，但是这种方法存在一个缺点就是，我们需要给网站中的所有请求都添加上这个 token，操作比较繁琐。还有一个问题是一般不会只有一台网站服务器，如果请求经过负载平衡转移到了其他的服务器，但是这个服务器的 session 中没有保留这个 token 的话，就没有办法验证了。这种情况可以通过改变 token 的构建方式来解决。
- 对 Cookie 进行双重验证，服务器在用户访问网站页面时，向请求域名注入一个 Cookie，内容为随机字符串，然后当用户再次向服务器发送请求的时候，从 cookie 中取出这个字符串，添加到 URL 参数中，然后服务器通过对 cookie 中的数据和参数中的数据进行比较，来进行验证。使用这种方式是利用了攻击者只能利用 cookie，但是不能访问获取 cookie 的特点。并且这种方法比 CSRF Token 的方法更加方便，并且不涉及到分布式访问的问题。这种方法的缺点是如果网站存在 XSS 漏洞的，那么这种方式会失效。同时这种方式不能做到子域名的隔离。
- 在设置 cookie 属性的时候设置 Samesite ，限制 cookie 不能作为被第三方使用，从而可以避免被攻击者利用。Samesite 一共有两种模式，一种是严格模式，在严格模式下 cookie 在任何情况下都不可能作为第三方 Cookie 使用，在宽松模式下，cookie 可以被请求是 GET 请求，且会发生页面跳转的请求所使用。
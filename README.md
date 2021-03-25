# browser-learn

## 浏览器之进程与线程

### 进程的基本概念

程序的一次执行, 它占有一片独有的内存空间.是操作系统执行的**基本单元**。启动一个程序的时候，操作系统会为该程序创建一块内存，用来存放代码、运行中的数据和一个执行任务的主线程，我们把这样的一个运行环境叫进程。

### 进程的特点

- 一个进程中至少有一个运行的线程: 主线程, 进程启动后自动创建
- 一个进程中也可以同时运行多个线程, 我们会说程序是多线程运行的
- 一个进程内的数据可以供其中的多个线程直接共享，多个进程之间的数据是不能直接共享的
- 进程之间相互隔离（不同的进程是通过 IPC 通信）

### 浏览器进程的体现

- Browser 浏览器主进程
  浏览器的主进程,负责浏览器界面的显示,和各个页面的管理,浏览器中所有其他类型进程的祖先,负责其他进程的的创建和销毁 **它有且只有一个!!!!!**
- Renderer 浏览器渲染进程
  - 网页渲染进程,负责页面的渲染,可以有多个当然渲染进程的数量不一定等于你开打网页的个数
  - 运行在沙箱、不能读写硬盘、不能获取操作系统权限
  - 解析 渲染 JS 执行
- 各种插件进程
- GPU 进程
  移动设备的浏览器可能不太一样:
  - Android 不支持插件,所以就没有插件进程
  - GPU 演化成了 Browser 进程的一个线程
  - Renderer 进程演化成了操作系统的一个服务进程,它仍然是独立的

### 线程

**是进程内的一个独立执行单元,是 CPU 调度的最小单元。程序运行的基本单元
线程池(thread pool): 保存多个线程对象的容器, 实现线程对象的反复利用**

## Chrom 最新浏览器架构

- 1 个浏览器主进程
  - 界面的显示
  - 用户的交互
  - 子进程管理
  - 提供存储
- 1 个 GPU 进程
  - 3D CSS 渲染
- 1 个网路进程
  - 网络资源的加载
  - 面向渲染进程和浏览器进程 等提供网络下载
- 多个渲染进程
  - HTML+CSS+JS 转换为可交互的网页
  - 排版引擎 Blink
  - JS V8
- 多个插件进程
  - 主要负责插件的运行

## 浏览器渲染原理

![20200407221502](https://raw.githubusercontent.com/yayxs/Pics/master/img/20200407221502.png)

### 1. 浏览器功能

- 网络
  - 浏览器通过网络模块来下载各式各样的资源，例如 html 文本；javascript 代码；样式表；图片；音视频文件等。
  - 网络部分本质上十分重要，因为它耗时长，而且需要安全访问互联网上的资源。
- 资源管理

  - 从网络下载，或者本地获取到的资源需要有高效的机制来管理它们。
  - 例如如何避免重复下载，资源如何缓存等

- 网页浏览

  - 资源转为可视化

- 等等……

## 浏览器渲染流程

- 浏览器采用流式布局模型（`Flow Based Layout`）
- 浏览器会把`HTML`解析成`DOM`，把`CSS`解析成`CSSOM`，`DOM`和`CSSOM`合并就产生了渲染树（`Render Tree`）。
- 有了`RenderTree`，我们就知道了所有节点的样式，然后计算他们在页面上的大小和位置，最后把节点绘制到页面上。
- 由于浏览器使用流式布局，对`Render Tree`的计算通常只需要遍历一次就可以完成，**但`table`及其内部元素除外，他们可能需要多次计算，通常要花 3 倍于同等元素的时间，这也是为什么要避免使用`table`布局的原因之一**

### 4. 浏览器渲染引擎

#### (1). 主要模块

- 解析器
  - 解释 HTML 文档的解析器
  - 作用：将 HTML 文本解释成 DOM 树
- CSS 解析器

  - 它的作用是为 DOM 中的各个元素对象计算出样式信息
  - 为布局提供基础设施

- JavaScript 引擎
  - 使用 Javascript 代码可以修改网页的内容，也能修改 css 的信息
  - javascript 引擎能够解释 javascript 代码，并通过 DOM 接口和 CSS 树接口来修改网页内容和样式信息，从而改变渲染的结果
- 布局（layout）回流
  - 在 DOM 创建之后，Webkit 需要将其中的元素对象同样式信息结合起来
  - 计算他们的大小位置等布局信息
  - 形成一个能表达这所有信息的内部表示模型
- 绘图模块
  - 使用图形库将布局计算后的各个网页的节点绘制成图像结果

## 渲染引擎处理流程

[渲染树的构建、布局、及绘制中文](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction?hl=zh-cn)

![20200407223600](https://raw.githubusercontent.com/yayxs/Pics/master/img/20200407223600.png)

1.  遇见 HTML 标记，调用 HTML 解析器解析为对应的 token （一个 token 就是一个标签文本的序列化）并构建 DOM 树（就是一块内存，保存着 tokens，建立它们之间的关系）
2.  遇见 `style/link`标记 调用解析器 处理 CSS 标记并构建 CSS 样式树，即 CSSOM
3.  遇见`script`标记 调用 `javascript`解析器 处理`script`标记，绑定事件、修改 DOM 树/CSS 树 等
4.  将 `DOM `树 与 `CSS`树 合并成一个渲染树（render 树:after :before 这样的伪元素会在这个环节被构建到 DOM 树中）
5.  根据渲染树来渲染，以计算每个节点的几何信息（这一过程需要依赖图形库）
6.  **页面绘制** ： 把每一个页面图层转换为像素，对媒体文件进行解码。将各个节点绘制到屏幕上。

## 浏览器渲染阻塞

#### (1). CSS 样式渲染阻塞

link 引入的外部 css**才能够产生阻塞**

#### (2). JS 阻塞

## 重绘

由于节点的几何属性发生改变或者由于样式发生改变而不会影响布局的，称为重绘，例如`outline`, `visibility`, `color`、`background-color`等，重绘的代价是高昂的，因为浏览器必须验证 DOM 树上其他节点元素的可见性。

#### (2). 回流

回流是布局或者几何属性需要改变就称为回流。回流是影响浏览器性能的关键因素，因为其变化涉及到部分页面（或是整个页面）的布局更新。一个元素的回流可能会导致了其所有子元素以及 DOM 中紧随其后的节点、祖先节点元素的随后的回流。**有的大佬也习惯称之为重排**

##### 什么情况下浏览器会发生回流

- 添加或删除可见的 DOM 元素
- 元素的位置发生变化
- 元素的尺寸发生变化（包括外边距、内边框、边框大小、高度和宽度等）
- 内容发生变化，比如文本变化或图片被另一个不同尺寸的图片所替代。
- 页面一开始渲染的时候（这肯定避免不了）
- 浏览器的窗口尺寸变化（因为回流是根据视口的大小来计算元素的位置和大小的）

#### (3). 浏览器优化

现代浏览器大多都是通过队列机制来批量更新布局，浏览器会把修改操作放在队列中，至少一个浏览器刷新（即 16.6ms）才会清空队列，但当你**获取布局信息的时候，队列中可能有会影响这些属性或方法返回值的操作，即使没有，浏览器也会强制清空队列，触发回流与重绘来确保返回正确的值**。

主要包括以下属性或方法：

- `offsetTop`、`offsetLeft`、`offsetWidth`、`offsetHeight`
- `scrollTop`、`scrollLeft`、`scrollWidth`、`scrollHeight`
- `clientTop`、`clientLeft`、`clientWidth`、`clientHeight`
- `width`、`height`
- `getComputedStyle()`
- `getBoundingClientRect()`

#### (4). 小结

**回流必定会发生重绘，重绘不一定会引发回流** 怎么最小化重绘重排？

1. ##### CSS

   - **使用 `transform` 替代 `top`**

   - **使用 `visibility` 替换 `display: none`** ，因为前者只会引起重绘，后者会引发回流（改变了布局

   - **避免使用`table`布局**，可能很小的一个小改动会造成整个 `table` 的重新布局。

   - **尽可能在`DOM`树的最末端改变`class`**，回流是不可避免的，但可以减少其影响。尽可能在 DOM 树的最末端改变 class，可以限制了回流的范围，使其影响尽可能少的节点。

   - **避免设置多层内联样式**，CSS 选择符**从右往左**匹配查找，避免节点层级过多。

     ```
     <div>
       <a> <span></span> </a>
     </div>
     <style>
       span {
         color: red;
       }
       div > a > span {
         color: red;
       }
     </style>
     ```

     对于第一种设置样式的方式来说，浏览器只需要找到页面中所有的 `span` 标签然后设置颜色，但是对于第二种设置样式的方式来说，浏览器首先需要找到所有的 `span` 标签，然后找到 `span` 标签上的 `a` 标签，最后再去找到 `div` 标签，然后给符合这种条件的 `span` 标签设置颜色，这样的递归过程就很复杂。所以我们应该尽可能的避免写**过于具体**的 CSS 选择器，然后对于 HTML 来说也尽量少的添加无意义标签，保证**层级扁平**。

   - **将动画效果应用到`position`属性为`absolute`或`fixed`的元素上**，避免影响其他元素的布局，这样只是一个重绘，而不是回流，同时，控制动画速度可以选择 `requestAnimationFrame`，详见[探讨 requestAnimationFrame](https://github.com/LuNaHaiJiao/blog/issues/30)。

   - **避免使用`CSS`表达式**，可能会引发回流。

   - **将频繁重绘或者回流的节点设置为图层**，图层能够阻止该节点的渲染行为影响别的节点，例如`will-change`、`video`、`iframe`等标签，浏览器会自动将该节点变为图层。

   - **CSS3 硬件加速（GPU 加速）**，使用 css3 硬件加速，可以让`transform`、`opacity`、`filters`这些动画不会引起回流重绘 。但是对于动画的其它属性，比如`background-color`这些，还是会引起回流重绘的，不过它还是可以提升这些动画的性能。

2. ##### JavaScript

   - **避免频繁操作样式**，最好一次性重写`style`属性，或者将样式列表定义为`class`并一次性更改`class`属性。
   - **避免频繁操作`DOM`**，创建一个`documentFragment`，在它上面应用所有`DOM操作`，最后再把它添加到文档中。
   - **避免频繁读取会引发回流/重绘的属性**，如果确实需要多次使用，就用一个变量缓存起来。
   - **对具有复杂动画的元素使用绝对定位**，使它脱离文档流，否则会引起父元素及后续元素频繁回流。

## 事件循环 Event-loop

事件循环：Event-loop（英文名）

## 从输入`URL`到页面加载完成，发生了什么？

如下图所示，当在浏览器地址栏输入一个地址的时候，发生了什么呢

![20200407214919](https://raw.githubusercontent.com/yayxs/Pics/master/img/20200407214919.png)

### `导航`流程

- 浏览器进程收到用户输入的URL请求 
  - 浏览器进程将URL转发给网络进程
- 在网络进程中发起真正的URL请求
- 网络进程接收到了额响应头的数据 
  - 解析响应头的数据
  - 将数据转发给浏览器进程
- 浏览器接到网络进程的响应头之后 发送提交导航 消息到渲染进程
- 渲染进程节后到消息 然后准备HTML数据 接收数据的方式是直接个网络进程建立数据管道
- 渲染进程会向浏览器进程确认提交 
- 浏览器进程接收到渲染进程的消息之后 移除旧的文档 然后更新浏览器进程中的状态

### 页面的展示

1、用户的输入

- 判断地址栏是否是搜索内容 还是请求的URL
  - 搜索内容的话 就是搜索关键字
  - 如果是符合URL规则的话 合成完整的URL

2、URL的请求过程

- 首先网络进程查找本地的魂村是否缓存了该资源 
  - 缓存了 直接返回资源给浏览器进程
  - 如果在缓存中没有查找到 直接进入网络请求的流程

- 请求前的第一步就是进行DNS解析 然后获取请求域名的服务器IP地址如果是HTTPS还需要建立TLS连接

- 利用IP地址和服务器建立`TCP`连接

- 浏览器端构建请求行、请求头信息 并把和该域名相关的Cookie 等数据附加到请求头中 然后向服务器发送构建的请求信息
- 服务器接收请求信息 根据请求信息生成响应的数据（响应行、响应头、响应体）
- 网络进程进行解析响应头的内容
- 重定向
  - 接收到服务器的响应头，网络进程开始解析响应头
    - 301/302 重定向其他的URL **有一个地址 是Locatiopn字段**
- 响应数据的处理
  - `Content-Type` ：告诉浏览器服务器返回的响应体数据是什么类型

3、准备渲染进程

默认情况下 Chrome会为每个页面分配渲染进程 ，有的是多个页面运行在同一个渲染进程中。

- Chrome 默认策略是，每个标签对应一个渲染进程，从一个页面打开了另一个新的页面 新页面和当前页面属于同一站点的话 新页面会复用父页面的渲染进程
- 渲染进程策略：
  - 通常情况下 打开新的页面都会使用单独的进程
  - A和B属于同一站点的话 那么B页面复用A页面的渲染进程 如果是其他情况的话 浏览器进程则会为B创建一个新的渲染进程

4、提交文档

浏览器进程将网络进程接收到的HTML数据提交给渲染进程

5、渲染阶段

- 一旦文档被提交 渲染进程便开始解析和子资源的加载 

## 常见的浏览器内核

[参考阅读百度百科](<[https://baike.baidu.com/item/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%86%85%E6%A0%B8](https://baike.baidu.com/item/浏览器内核)>)

> 浏览器最重要或者说核心的部分是“Rendering Engine”，可大概译为“渲染引擎”，不过我们一般习惯将之称为“浏览器内核”。

**不同的浏览器内核对网页编写语法的解释也有不同，因此同一网页在不同的内核的浏览器里的渲染（显示）效果也可能不同，这也是网页编写者需要在不同内核的浏览器中测试网页显示效果的原因。**

| 内核分类           | 采用该内核的浏览器                                                                                                                                                                     |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Trident（**IE**）  | IE、傲游、世界之窗浏览器、Avant、腾讯 TT、Sleipnir、GOSURF、GreenBrowser 和 KKman 等。                                                                                                 |
| Gecko(**火狐**)    | [Mozilla Firefox](https://baike.baidu.com/item/Mozilla Firefox)、Mozilla SeaMonkey、waterfox（Firefox 的 64 位开源版）、Iceweasel、Epiphany（早期版本）、Flock（早期版本）、K-Meleon。 |
| Webkit（**谷歌**） | Google Chrome、[360 极速浏览器](https://baike.baidu.com/item/360极速浏览器)以及[搜狗高速浏览器](https://baike.baidu.com/item/搜狗高速浏览器)高速模式也使用 Webkit 作为内核             |

## 考题

### 考题一

```js
console.log(1); // 1
setTimeout(function () {
  // 异步任务一
  console.log(2); //6
});
new Promise(function (resolve) {
  console.log(3); //2
  resolve();
})
  .then(function () {
    // 异步任务
    console.log(4); //4
  })
  .then(function () {
    // 异步任务
    console.log(5); //5
  });
console.log(6); //3
```

### 考题二

```js
Promise.resolve()
  .then(function () {
    console.log("promise1");
  })
  .then(function () {
    console.log("promise2");
  });

process.nextTick(() => {
  console.log("nextTick1");
  process.nextTick(() => {
    console.log("nextTick2");
    process.nextTick(() => {
      console.log("nextTick3");
      process.nextTick(() => {
        console.log("nextTick4");
      });
    });
  });
});
```

### 考题三

```js
setTimeout(() => {
  console.log("timeout1");
}, 0);

setTimeout(() => {
  console.log("timeout2");
  Promise.resolve().then(function () {
    console.log("promise1");
  });
}, 0);

setTimeout(() => {
  console.log("timeout3");
}, 0);
```

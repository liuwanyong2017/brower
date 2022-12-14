<!--
 * @Author: liuwanyong liuwanyong2018@gmail.com
 * @Date: 2022-10-15 16:16:27
 * @LastEditors: liuwanyong liuwanyong2018@gmail.com
 * @LastEditTime: 2023-01-07 11:06:00
 * @FilePath: /brower/README.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE##
-->

# brower

浏览器概念知识构建

## 历史

浏览器目前是多进程架构的技术选型，由单进程的技术历史发展过来的。
以前的硬件不行，CPU，GPU，内存，闪存，带宽啊等等都不行，以前的网络服务的数据实体也简单，图文而已。
某个技术的发展都是跟整个生态技术的发展息息相关的。

## 线程与进程

### 线程

线程应该是跟硬件层面的 cpu 线程有关联的，但是不讨论低层的执行到底有木有关系
。我们的范畴只存在于软件技术领域。这个线程可以理解为一段可以独立运行的代码逻辑
的实际运行。
单单只看它本身木得意义，有意义的是，在时间尺度和内存尺度看管理它的进程，以及
同时并行运行的其他线程，就有意义了。
还有一个概念就是并行运行。
其实可想一个对比就是单线程运行几个独立不干扰的逻辑，跟多个线程运行它们，为了效率。
线程之间共享同一进程的内存，数据。

### 进程

进程，在我看来就是一个独立的程序从开始运行到结束
整个过程中，它用到的内存，线程活动管理，代码，数据操作等等所有的
总和。官方叫程序的运行环境。

#### 特色特性

-  多线程并行运行，某个线程出问题，整个进程出问题，会崩掉了。
-  进程之间互相独立，互不干扰。
-  进程终结，它的内存也被回收了。
-  线程之间共享进程的数据。

#### 单进程技术浏览器

运行的任务：

-  多个页面的网络发收，数据解析，渲染，交互等。
-  插件的运行。
-  对内存的读写等。
   出现任何一个问题，都要堵塞或者崩溃进程，所有页面都卡顿或者崩溃。
   放开读写给插件和进程页面，很危险。
   页面关闭，内存回收不干净，内存泄漏，越来越卡顿。

#### 多进程技术架构

主进程，
GUI 进程，网络进程，插件进程，渲染进程。这五种进程做分工协作
。

##### 主进程

人机交互，其他进程的管理，读写内存，界面显示等。它才有读写权限。安全。

##### 渲染进程

一般情况下，，，一个页面一个渲染进程，互相不干扰，关了就回收内存，很干净。包括 HTML
，css,js 转换为页面的所有逻辑都是它处理。排版引擎
blink,js 引擎 v8。
对读写
权限很严，沙箱模式下运行的。

##### GPU 进程

为了 Css 的
3D 效果，出现的，后来网页和默认的 UI 界面都是它绘制了。

##### 网络进程

网络资源的加载。

##### 插件进程

隔离插件和页面的运行影响，插件容易崩溃。

##### 评价

体验很好，流畅，安全，互相不影响。
但是，很多进程都是公共资源的副本，比如渲染进程，浪费内存。
而且各个模块之间的耦合度很高，扩展性差，关系复杂。升级困难。

## 从浏览器输入 url 开始到页面展示

##输入，浏览器主线程

浏览器主线程管交互，同时管理浏览器的页面之外的区域的
交互功能。
输入的内容，是关键词的话，就会自动搜索相关的 URL 结果
。

### 建立 TCP

输入的是 url,或者 host（自动补全成 url），就会 开始找缓存，查询是否有记录。
有记录有数据，不过期，就直接返回这个数据了。
没有的话，开始 DNS 查询。
有木有缓存记录，有就用，没有就找根 DNS 查顶级域名服务器，顶级再找下一级的域名服务器，域名都是分级组成的。就好比图书馆的检索系统，大类有小类，小类里面还有小类。
找到 IP 地址。
然后
开始下一步就是，建立 TCP 链接。假如是 HTTPS 协议，这里还要先进行 TLS 连接。

### 返回的数据情况

TCP3 次握手之后，浏览器创建请求行，请求头，把 cookie 这类的数据也会带上，然后就发送出去了。
响应，可能是 301,302。这意味着，重定向，响应头里的 location 带有新 URL，然后开始从头开始发请求。
还有其他的响应，不成功的就说明终止了。
200 的成功的，还要看响应头的 content-type，假如不是 xml 这种 html 数据，说明也结束了。
是 html，这时候，浏览器主进程会开启渲染进程，可能不会开启，
判断依据是 a 页面打开的 b 页面，ab 是同一域名的，同一协议的，这就是同一个站点了。

### 导航流程终结

开启渲染进程之后，主进程通知 渲染进程开始提交文档。渲染进程开始跟网络进场建立连接，然后
数据传输，完成后，渲染进程返回确认提交消息给主进程。
主进程更新 history，前进后退,地址栏显示这些。导航流程完成。

### 渲染阶段

接下来就是渲染阶段了。一旦渲染进程解析好了 HTML css js，资源也加载好了，渲染进程回通知主进程，主进程会停止页面 tab 上的 loading 状态。

## 渲染过程

-  html 解析，生成 DOM 树
-  CSS 文件的获取和解析，生成 CSSSheet，然后跟 DOM 树结合作用生成渲染树。
-  JS 执行，对应的 DOM 操作也加入到渲染树上。

### 生成 DOM 树

对 HTML 进行词法分析语法分析，同时遇到外部资源的 link 链接也会请求。
生成 DOM 树，就是父子兄弟关系的 树形结构。

### 样式计算

源于 HTML 上的 link 链接的 Css 文件，还有 style 标签的内容，还有标签的内联样式的解析。
然后就把这些内容解析成浏览器可以 理解的数据结构。
可以在浏览器里面输入 document.styleSheets。

-  解析 css 文件的代码，生成 cssSheet，这个跟 css 树有概念区别，这个更标准。
-  转换 css 属性里面的属性值，标准化，变量，关键词等化为标准的数据。
-  计算出每个 DOM 节点的样式
   这里需要遵循 CSS 的层叠规则和继承规则了。优先级，覆盖结果，继承属性等等场景。

### 布局

#### 创建布局树

有很多的不可见的元素，比如 header 标签，比如说 display 为 none 的元素。
要创建一个只有 可见的节点的布局树。
遍历 DOM 树，不可见的节点都忽略，生成了布局树。

#### 分层，图层树

有些 DOM 节点有层叠上下文的属性，它们被提升为单独的一层。分层的意思就是，三维的尺度来构建渲染逻辑。
需要裁切的地方也会被创建为图层。
一般情况下子节点在父节点的图层里。

图层的意义就是在于超出父容器隐藏，还有层叠上下文的用法，就是立体空间的设计价值了。

#### 图层绘制

每个图层，拆解成很简单的绘制指令，组合起来完成每个图层的绘制任务。
绘制一个元素，往往需要一系列的绘制指令，生成了绘制指令的列表，记录绘制顺序，绘制指令。
到生成绘制指令的列表这里都是渲染进程的主线程做的。

#### 交由 GPU 进程参与的合成线程进行栅格化

主线程把绘制指令交给合成线程，合成线程要做的是栅格化。
栅格化是为了渲染压力，性能调优。为了追求只渲染视口以及周边的位置的内容。
合成线程会把图层划分为图块，并优先把视口周围的图块生成位图。
图块大小通常为 256*256 ，512*512。
栅格化的任务就是把图块生成位图，最小单位就是图块。
渲染进程维护了一个栅格化的线程池，所有的图块栅格化都是在线程池内进行的。
线程池里有栅格化线程专门处理栅格化任务。

通常，栅格化过程会需要 GPU 加速生成，需要 GPU 进程参与其中。是渲染进程与 GPU 进程的合作，这叫快速栅格 化。
GPU 进程会处理栅格化线程池。
GPU 进程会生成位图，然后结果保存在 GPU 内存里。

#### 合成、显示

一旦所有的图块都栅格化了。合成线程会生成一个绘制图块的命令，交给浏览器进程。
浏览器进程里有个 viz 组件，接收这个命令，将页面内容绘制到内存中，最后再显示到屏幕上。

#### 流程总结

DOM ， CSSSheet ，cssSheet 数据标准化， DOM 节点的样式计算，层叠，继承等规则，
生成 布局树， 生成图层树， 生成绘制图层的绘制指令列表，
切割成图块，生成位图，
给浏览器主线程发送绘制指令，
绘制页面内容，存在内存里，然后显示在显示器。

### 重排的影响

重排影响了 dom 树开始的所有
流程。

### 重绘的影响

跳过了布局图和图层树的布局阶段。

### 不重排不重绘的属性

直接跳过布局和绘制阶段，比如用
transform 动画。
避开了重排和重绘阶段，跳开布局和绘制阶段。
触发了后续的合成线程里的划分图块，
栅格化等过程。这样不会麻烦主线程的性能和资源。

#### 减少重排重绘的消耗

-  集中操作 DOM 的场景，借鉴虚拟 DOM 的逻辑，还有就是将 DOM 离线，display 为 none，然后大量的样式操作，然后再显示，产生了两次重排重绘。
-  脱离文档流，减少重排的开销，这里单独的优化计算，这个区域的节点可能会重绘，但不需要重排。
-  不要一边读 DOM，一边 重设 DOM 的样式，一次性读取所有的样式，然后一次性设置所有的样式，浏览器的渲染优化会每隔一段时间批量处理渲染队列的 DOM 属性操作。读取 DOM 元素的属性会触发重排。
-  动画效果触发了 GPU 的图形操作，它的性能会很快，它很专业。用 transform 3D。但是最好动画
   要快，减少重排的频率，CPU 的压力会小。

#### JS 解析时，有样式操作会阻塞 DOM 解析

如题，需哟等待这个样式被下载，才能继续往下执行，Css 阻塞范畴。

## JS 编译

一段 JS 代码，经过编译，然后执行。这个阶段会有什么规则？

### 变量提升

这个是编译的第一个规则，为了拿到变量和函数体，放到后续可执行代码的执行的上下文中。
编译之后，是一段可执行的代码的字节码。

执行上下文中有个 变量环境对象，对象中存了变量提升的内容。
只会存变量，值是 undefined 初始化。函数变量，值是函数体的堆指针。

### 执行阶段

按照顺序执行代码，遇到变量就去变量环境对象中找，找到就用，找不到就报错。

#### 编译阶段，有相同的变量怎么办

假如变量都是函数，是覆盖，假如是函数和非函数变量，不论顺序，保函数。

## 函数调用

JS 引擎在执行一段代码时，先是编译阶段，然后是执行阶段。
编译阶段是为了生成可执行代码的执行上下文，执行上下文里有执行代码的变量环境和词法环境。
而执行代码的时候，假如遇到执行函数了，也就是调用函数了，这时候，就会从执行上下文里找到函数变量，
然后拿到函数体的堆指针，拿到函数体代码。
然后就开始编译函数体代码，需要形成新的可执行上下文和可执行代码。这里需要一种数据结构的支持，叫栈，这里是调用栈的概念。

### 栈

一种数据结构，后进先出，先进后出。它是多个层级的执行上下文的容器。

### JS 的调用栈

JS 通过栈结构来管理多个可执行上下文，每生成一个可执行上下文，就会把它压入栈里，通常叫调用栈。

一般创建全局上下文，压入栈底。
然后执行代码，遇到函数调用，生成新的执行上下文，再压入栈里。
然后执行代码，一旦遇到函数调用了，就会继续生成新的执行上下文，再压入栈里。
而一旦可执行上下文相关的可执行代码执行完毕了，这个可执行上下文就会销毁了，跳到下层的
执行上下文里的可执行代码那里，继续执行，完毕，然后销毁，直到全局执行上下文为止。

#### 利用调用栈

浏览器调试代码，可以打断点，查看调用栈信息。

#### 堆栈溢出

调用栈是有大小的，一旦压入的执行上下文太多超出调用栈的容量了，JS 引擎会报错堆栈溢出了。
常见的是过多的递归逻辑代码。` 常见的优化就是把递归变成循环。`
`
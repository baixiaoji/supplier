## 4-解析-HTML 解析器

因为 HTML 语言在语法层面并有那么严格的语法规则，导致常规的解析器并不能解析HTML文档，对应的解决方案让浏览器厂商自定义 HTML 解析器。那么，让我们一起梳理一下 HTML 解析器到底是什么吧~

### 输入（语法）

因为 HTML 语法是由 W3C 组织创建的规范中进行定义的，而且语法格式是由 DTD （Document Type Definition）定义的，该格式中定义了语言中允许的元素、属性和层次结构，适用一切的 SGML （Standard Gerneralized Markup Languge）族的语言。为了在发展的进程中向后兼容老版本的内容，DTD 存在两种模式，严格模式完全遵守 HTML 规范，其他模式支持老的浏览器使用的编辑。

### 解析算法

因为 HTML 文档语法特性（包容性），以及在解析过程中存在脚本会改变 HTML 文档（如： document.write），导致无法使用自上而下或是自下而上的解析器进行解析。

![](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image017.png)

解析过程前半段是词法分析，也就是标记化（tokenization），整体算法的核心就是状态机的改变（就是解析过程中有一个标识当前状态应是解析到哪一个阶段了）。

![](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image019.png)

同时构建 DOM 树，也就是树构建（tree construction）过程，该过程就是我们在「解析-理论剖析」讲述的一样，将对应的标记去击中语法，然后添加到 DOM 树上。此过程中也有一个状态机去维护对应的阶段。

![](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image022.gif)

最后 DOM 树是 HTML 文档的映射关系和存留 HTML 元素对外的接口（如： 对JS），每一个节点是由 DOM 元素和节点属性组成。看一个例子：

```HTML
<html>
  <body>
    <p>
      Hello World
    </p>
    <div> <img src="example.png"/></div>
  </body>
</html>
```

![](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/image015.png)

解析完，进入交互阶段开始解析处于 'deferred mode' （that should be executed after the document is parsed）的脚本，执行完这些脚本后，文档状态为 complete，触发 load 事件。

因为解析器是浏览器厂商的自定义，而 HTML语法特性比较特殊，所以解析器要有相关的容错机制，而这机制并不是 HTML 规范中强制规定，而是浏览器发展过程中的产品（友商之间互抄好的地方呗），但是后期的 HTML 5 规范中有部分容错机制的要求（webkit 的 HTML 解析器就有这样的注释）。

上述只是描述到 HTML 文档的解析，那脚本和样式的解析顺序呢？

因为 web 的模型式同步的原因，如果遇见内部 `<script>` 标签，就会中断 HTML 解析，开始执行脚本，直到脚本执行完毕，而遇到外部的脚本，解析同样中断直到请求脚本回来。这些解析模式在 HTML 4 5规范中有所描述。

毕竟突然中断 HTML 解析还是会影响页面展示的时间，那样我们需要规避不必要的因脚本而中断解析，那就是给 `<script>`添加 defer 属性，这是 HTML 5中给脚本标记为异步的标识，这样脚本通过不同的线程解析和执行。

当有脚本在执行的过程中，会触发「预解析」，此时是其他线程继续解析文档，找到需要请求加载的资源，加载这部分资源（并行加载，提高整体速度）。预解析只解析外部文件（外部脚本、样式或图片），此过程不修改 DOM 树。

样式方面，因为解析样式并不会影响 DOM 树，所以不需要中断文档解析。但同时存在一个问题，当脚本获取样式信息时，但此时样式并没有加载就会报错。对应的解决手段就是阻塞脚本，但不用浏览器阻塞的阶段不同，firefox是当样式加载或解析的时候，会阻塞所有的脚本；而 webkit 是当脚本去访问那些确定会被为加载样式影响到的属性时，阻塞脚本。

以上是刚刚完成了 DOM 树的构建，那我们马上进入后续阶段咯~


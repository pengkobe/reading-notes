## 关于标准
> 看标准是一件很头疼的事情，不过看完之后你会有顿然开朗的感觉，Preload 的前世今生都在那里，
这里，分享自己对标准做的一个"简短"的解读，不足之处，还望指正。

## Why Preload 
在开发中，很多人碰到过这样的情况，有些资源虽然不会立即执行，但是希望需要尽早获取，要是等到需要用时再去加载，需要晚点加载的原因是如依赖管理、条件加载、加载顺序控制等等。在之前，你要做到这些，是需要有额外的性能花费的。
首先，看看大家采用的方案:
* 动态插入元素(e.g. img, script, link)
  + 缺陷:script 并不能延迟执行
* 使用 XMLHttpRequest 进行异步加载
  + 缺陷1: 浏览器无法根据资源类型进行预判并优化加载，造成性能问题
  + 缺陷2: 大量的加载脚本会阻塞页面的加载，产生严重的性能问题.
  
Preload 则很好的解决了这些问题:
* 不阻塞式加载，与页面加载逻辑分离
* 资源提前预判，加载迅速
* 优先级高，不像 Prefetch/Subresource 那般模棱两可

#### 使用示例
```html
<!-- 通过标签加载 -->
<link rel="preload" href="/styles/other.css" as="style">
<!-- 通过 JS 动态插入 -->
<script>
var res = document.createElement("link");
res.rel = "preload";
res.as = "style";
res.href = "styles/other.css";
document.head.appendChild(res);
</script>
```

## 加载资源的时机
### 开发者
1. 首先，浏览器得支持 Preload
2. Preload link 标签已插入至 document
3. Preload link 标签在 document 中存在
4. href 属性改变
5. 设置/修改/移除 crossorigin 属性时
6. as 指定的资源不再是之前的资源时
7. as 指定的资源之前没有加载，当前发生改变
8. type 指定的资源之前没有加载(MIME不支持等)，当前发生改变
9. media 指定的资源支持没有加载(资源无效)，当前发生改变

### 浏览器
如果 href 属性被重置,浏览器当立即停止当前加载，浏览需要做的一些动作可以分为以下几个步骤：
1. 如果 href 属性被重置为空, 浏览器立即停止加载.
2. 解析 URL
3. 如果前两步失败，立即停止加载
4. 解析 as 属性，如果没有填写则置为空字符串，如果 as 指定资源类型无效，则抛出错误，你可以用 onerror 属性捕获到
5. 解析 type 属性，如果没有填写则置为空字符串，并开始下一步骤; 如果填写了但是不是有效的 MIME 类型，那么立即停止加载
6. 解析 media 属性，如果没有填写则置为空字符串，并开始下一步骤; 如果填写了但是不是有效的媒体类型类型，那么立即停止加载
7. 以跨域方式访问 URL 指定的资源。 请求头中的 origin 属性指定为当前页，原来默认的 origin 置为 taint

#### 一些要求  
* 浏览器加载 Preload link 元素时不能影响当前页 dom 元素的加载
* 获取资源时默认按照同源策略加载，资源信息由 as 指定，需要as 中的内容会写到请求头中，如果不指定 as，那么跟 XMLHttpRequest 请求方式并无二样。

#### 资源加载完成
浏览器需要有以下几个动作
* 成功加载时触发 load 事件，失败加载时出发 error 事件
* 将资源置入缓存
  + 浏览器会有自己的一套缓存逻辑，这里不做讨论
  + 理论上必须放入 http 资源缓存与内存( 便于当前页使用 )中。
* 确保不在当前上下文执行加载下来的资源，此外，必须确保资源不会被重复加载。

## Link 元素接口拓展
这小结反复强调 as 的重要性，不指定 as 就用不上 Preload
```javascript
partial interface HTMLLinkElement {
  [CEReactions] attribute DOMString as;
};
```
as 必须是准确无误，否则 preload 无法正确执行相关逻辑，具体可以参考上篇中所述的*特性*
as 资源类型可以有下面这些:
* image
* script 
* media
* worker
* embed
* object
* font 
* style
* document( 指iframe/frame )

## Server Push( HTTP2 )
上次探讨我们说到， Preload 和 HTTP/2 Server Push 其实是相辅相成的，事实上，
从浏览器角度来说，你用 Preload 与 Server Push 获取资源其实并无两样。
Preload 提供了一个 nopush 属性，我们可以和服务器约定是否进行 Server Push，而开发者只需要简单的加一个属性 nopush 进行标记就好了。
如果你不加，浏览器就会把 Preload 指定的资源作为 Server Push 的候选项了。
``` html
<link rel="preload" href="/app/style.css" as="style" nopush>
<link rel="preload" href="/app/script.js" as="script" nopush>
```

## 非规范内容
（未完待续）

## 下次探讨内容预告
* 直接上实例 demo 啦~


## 参考
* [Preload,W3C Editor's Draft 01 March 2017](https://w3c.github.io/preload/)
* [how-exactly-does-link-rel-preload-work](http://stackoverflow.com/questions/36641137/how-exactly-does-link-rel-preload-work)

# 性能优化之Preload
Preload其实一项新的 web 标准，使得 web 开发者可以对加载细节做进一步控制,可以自定义加载逻辑。
> Preload is destined for current navigation


## 规范
地址: https://w3c.github.io/preload/
注意:
  + preload还在修订阶段，文档不断变化中。
  + 目前只有 Chrome/Safari 高版本支持。



## 相关概念
1. prefetch: 获取可能会用到的资源
2. subresource: 针对当前页面, 但是不是很成功，简单来说，就是想得美好，但是...嘿嘿嘿



## 特性
* as
  + 浏览器能够正确设置资源优先级
  + 能够确保发出的请求符合内容安全协议,不发无用的请求。
  + 浏览器能够根据资源类型发送合适的接收请求头(Accept headers)
  + 浏览器能通过资源类型来推断是否可以重用
* onload
  + 不会阻断窗口的 onload 事件( 除非请求的资源里有进行阻断 ).



## 使用场景

### 加载非标签资源
主要指藏在 Css 或 Js 中的资源,as 则可以指明资源类型，因为浏览器不知道的类型往往加载优先级较低。
```html
<!--
  参见:https://fetch.spec.whatwg.org/#concept-request-destination
-->
<link rel="preload" href="late_discovered_thing.js" as="script">
```

### 加载字体
加载字体规则异常复杂,有些重要的字体等到真正加载的时候已经晚了。
PS: 即使符合同源策略，也需要加上 crossorigin

```html
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
```

### 只加载不执行
一般用于加载 js 文件
```js
var preload = document.createElement("link");
link.href = "myscript.js";
link.rel = "preload";
link.as = "script";
document.head.appendChild(link);
```

### 基于标签的异步加载
```html
<link rel="preload" as="style" href="async_style.css" onload="this.rel='stylesheet'">
```

### 响应式加载
```html
<link rel="preload" as="image" href="map.png" media="(max-width: 600px)">
<link rel="preload" as="script" href="map.js" media="(min-width: 601px)">
```



### 注意事项
* 如果浏览器不支持，你得有 backup 策略
* 与 http/2 相辅相成，并不是使用 http2 push 就不用使用 preload 了。


## 参考
这里其实就是对其进行简单的总结:
* [preload-what-is-it-good-for](https://www.smashingmagazine.com/2016/02/preload-what-is-it-good-for/]

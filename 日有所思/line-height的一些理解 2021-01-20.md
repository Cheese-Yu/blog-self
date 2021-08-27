# line-height的一些理解
最近同事问了一个有趣的问题，给 `<span>` 设置 `line-height` 不生效该怎么办？我的第一反应，给 `<span>` 加 `display: inline-block;` 属性，`line-height` 生效了。但我不知道为什么，于是有了这篇文章。

### 何为 line-boxes

一行文字称为一个 line-boxes，其又包含多个 inline-boxes。line-boxes 的高度由该行最高的 inline-box 决定。  
下面这个例子，设置 `<span>` 的 `line-height: 30px;`， `<div>` 的 `line-height: 24px;`，结果 div 的 height 为 30px; `<span>` 的 height 却为 24px。

```javascript
<div>这是一行<span>代码</span></div>

// css
div {
  font-size: 24px;
  line-height: 1;
}

span {
  line-height: 30px;
}
```

### span 高度为何没有被 line-height 撑起来

> 在 MDN 上，解释如下：line-height CSS 属性用于设置多行元素的空间量，如多行文本的间距。对于块级元素，它指定元素行盒（line boxes）的最小高度。对于非替代的 inline 元素，它用于计算行盒（line box）的高度。

要说上面的例子 `line-height` 并不是没有生效，只是没有撑起 `<span>` 的高度而已。  
对于 `<span>` 这种 inline 元素的 `line-height` 用于计算 line-boxes 的高度，并不是自身的高度。当给 `<span>` 设置了 `display: inline-block;` 属性时，`<span>` 就变成了块级元素。  
但 MDN 定义中的 `元素行盒(line boxes)` 和 `行盒(line box)` 的区别和关系，我还没有搞懂。待日后清楚后会再来补充本篇文章。
  
<br />
> 语雀地址 https://www.yuque.com/cheeseyu/fullstack/ai9f9b
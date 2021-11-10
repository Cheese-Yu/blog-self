# Web Components
[MDN 文档地址](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)  
[Custom elements](https://html.spec.whatwg.org/multipage/custom-elements.html#custom-elements)

### customElements

自定义 html 标签，为防止冲突，只能使用带短横线的名称。

```javascript
class UserCard extends HTMLElement {
  constructor() {
    super();

    const image = document.createElement("img");
    image.src = "https://semantic-ui.com/images/avatar2/large/kristy.png";
    image.classList.add("image");

    const container = document.createElement("div");
    container.classList.add("container");

    const name = document.createElement("p");
    name.classList.add("name");
    if (this.hasAttribute("name")) {
      // 如果存在 name 属性，读取 name 属性的值
      name.innerText = this.getAttribute("name");
    }

    const email = document.createElement("p");
    email.classList.add("email");
    email.innerText = "yourmail@some-email.com";

    const button = document.createElement("button");
    button.classList.add("button");
    button.innerText = "Follow";

    container.append(name, email, button);
    // this 表示 UserCard 实例
    this.append(image, container);
  }
}
// 声明customElements
customElements.define("user-card", UserCard);

// 使用
<user-card name="Your Name"></user-card>;
```

### Shadow Dom

创建的 Dom Tree 不能被外部修改，保持元素的私有性。出于安全考虑，一些标签是不可以挂载在 shadow root 下的，如 `<a>`

```javascript
class UserCard extends HTMLElement {
  constructor() {
    super();
    // 声明 shadow
    const shadow = this.attachShadow({ mode: "closed" });

    const name = document.createElement("p");
    name.classList.add("name");
    name.innerText = "User Name";

    shadow.appendChild(name);
  }
}
```

### HTML templates

直接在 html 中写 template 代码，通过 DOM API 来操作、修改

```javascript
<template id="customerButton">
  <style>
    :host {
      display: inline-block;
      padding: 5px 15px;
      box-sizing:border-box;
      border:1px solid var(--borderColor, #ccc);
      font-size: 14px;
      color: var(--fontColor,#333);
      border-radius: var(--borderRadius,.25em);
      cursor: pointer;
    }
    :host([type="primary"]) {
      background: #409EFF;
      border-color: #409EFF;
      color: #fff;
    }
    :host([type="success"]) {
      background: #67C23A;
      border-color: #67C23A;
      color: #fff;
    }
    :host([type="warning"]) {
      background: #E6A23C;
      border-color: #E6A23C;
      color: #fff;
    }
    :host([type="error"]) {
      background: #F56C6C;
      border-color: #F56C6C;
      color: #fff;
    }
    :host([disabled]) {
      opacity: .5;
      cursor: not-allowed;
    }
    :host([text]) {
      padding: 0;
      border: none;
      background: none;
    }
    :host([text][type="primary"]) {
      padding: 0;
      border: none;
      background: none;
      color: #409EFF;
    }
    :host([text][type="success"]) {
      padding: 0;
      border: none;
      background: none;
      color: #67C23A;
    }
    :host([text][type="warning"]) {
      padding: 0;
      border: none;
      background: none;
      color: #E6A23C;
    }
    :host([text][type="error"]) {
      padding: 0;
      border: none;
      background: none;
      color: #F56C6C;
    }
    .xw-button {
      background:none;
      outline:0;
      border:0;
      width:100%;
      height:100%;
      padding:0;
      user-select: none;
      cursor: unset;
    }
  </style>
  <div class="xw-button">
    <slot></slot>
  </div>
</template>

// 通过 ID 获取标签
const tplElem = document.getElementById('customerButton');
const content = tplElem.content.cloneNode(true);

```

### 总结

在无需考虑兼容性的情况下，非常适合做代码组件拆分，作为微前端的一种解决方案。

<br>
  
> 语雀地址 https://www.yuque.com/cheeseyu/fullstack/be6cue
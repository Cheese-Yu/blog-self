# export、export default和module.exports、exports
import / export / export default:  只有 es6 支持的导出引入  
require / module.exports / exports:  只有 node 支持的导出

### 1. export / export default / import

- 一个文件或模块中，export、import 可以有多个，export default 仅有一个
- 使用 export， import 需要加大括号(\* 除外), export default 则不需要

```javascript
// testEs6.js
"use strict";
//导出变量
export const a = "100";

//导出方法
export function dogSay() {
  console.log("wang wang");
}

//导出方法第二种
function catSay() {
  console.log("miao miao");
}
export { catSay };

//export default导出
export default {
  m: 100,
  install: function (Vue) {
    Vue.prototype.$api = api;
    Vue.prototype.$get = get;
    Vue.prototype.$post = post;
    Vue.prototype.$log = log;
  },
};
//export defult const m = 100;// 不能这样写,报错
```

```javascript
//index.js
"use strict";
var express = require("express");
var router = express.Router();

import { dogSay, catSay } from "./testEs6"; //引入 export 方法,要加{}
import obj from "./testEs6"; //引入 export default

/* GET home page. */
router.get("/", function (req, res, next) {
  dogSay();
  catSay();
  console.log(obj); //{m, install}
  res.send("恭喜你，成功验证");
});

module.exports = router;
```

### 2. require / module.exports / exports

- Node.js 默认先从缓存中加载模块，require 多次也只缓存一份(通过文件路径缓存)
- require 模块加载是同步的(作为公共依赖的模块，当然想一次加载出来，同步更好)
- exports 只是 module.exports 的引用，辅助后者添加内容用的

```javascript
exports = module.exports = {}; // 指向内存同一地址

// config.js
module.exports = {
  HOST: '127.0.0.1',
  mongo: {
    enable: true,
    package: 'egg-mongo-native',
  },
};

// a.js
exports.verifyPassword = function(user, password, done) { ... }


//b.js 引入
const config = require('config.js');
const verifyPassword = require('a.js').verifyPassword;
console.log(config.HOST);
```

参考:   
[exports、module.exports 和 export、export default 到底是咋回事](https://juejin.im/post/597ec55a51882556a234fcef)  
[require 时，exports 和 module.exports 的区别你真的懂吗？](https://juejin.im/post/5d5639c7e51d453b5c1218b4)

<br>
  
> 语雀地址 https://www.yuque.com/cheeseyu/fullstack/pooya7
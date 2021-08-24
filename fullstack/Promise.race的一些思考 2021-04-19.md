`Promise.race([...])`接受一个数组参数，一旦数组中的任何一个 Promise 完成(`resolve`或`reject`)，它就会`resolve`或`reject`，传入的是单个值。一旦传入空数组，race 讲永远不会有结果，所以千万不要传空数组。

**那么 Promise.race 执行决议后，其他 Promise 还会执行么？**
我们来看一段代码:

```javascript
Promise.observe = function (pr, cb) {
  pr.then(
    function fulfilled(res) {
      Promise.resolve(res).then(cb);
    },
    function rejected(err) {
      Promise.resolve(err).then(cb);
    }
  );
  return pr;
};

function timeoutPr(time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject("It's timeout.");
    }, time);
  });
}

function foo(time) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(`finished foo, time is ${time}.`);
    }, time);
  });
}

function run() {
  Promise.race([
    Promise.observe(foo(1500), function cleanup(msg) {
      // 这里一定能执行到
      console.log("[observe cb]", msg);
    }),
    foo(2500),
    foo(2000).then((resolve, reject) => {
      console.log("It's 2000");
    }), // 不要这样写，只是为了验证
    timeoutPr(1200),
  ])
    .then((res) => {
      console.log("[race then]", res);
    })
    .catch((err) => {
      console.log("[race catch]", err);
    });
}

run();

// 最后的结果为:
/**
 * [race catch] It's timeout.
 * [observe cb] finished foo, time is 1500.
 * It's 2000
 */
```

上面代码，`timeoutPr`是最先完成的，所以在`catch`中捕获到了其抛出的错误。`race`会忽略后续其他`Promise`的决议，但是其他`Promise`还是会继续执行的。
`Promise`不能被取消，具有外部不变性原则，这也可以解释`Promise.race`中的其他`Promise`还会继续执行。

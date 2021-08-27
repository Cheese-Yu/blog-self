# Promise.all并发限制
近日，对一个方法进行压力测试，看一下调用 10w 次该方法的耗时和 CPU 消耗，但却遇到了一些问题。话不多说，直接代码先撸起来。(此处的异步方法均为伪代码，测试结果为实际的异步方法的测试)

### 串行执行

```javascript
// 定义一个异步方法
const timeout = (t = 1000) =>
  new Promise((resolve) => setTimeout(() => resolve(t), t));

async function serialRun() {
  let num = 1000;
  let result = [];
  while (num) {
    num--;
    const time = parseInt(Math.random() * 300) + 200;
    const res = await timeout(time);
    result.push(res);
  }
  return result;
}
```

串行执行下来，总耗时 850 秒左右，node 进程 CPU 占比 110%~120%左右。  
**CPU 占比为什么会超过 100%呢？**  
因为我的电脑是 4 核 CPU，显示的占比是多核累加起来的。

### 并行

当我以为任务愉快地完成的时候，转念一想，真的放到接口中时，请求一定是并发的，而不是一个等一个地进行的。这难不住我，`Promise.all`搞起。1k 和 1w 都愉快地测完了，10w 次测试时意外发生了：进程被莫名其妙干掉了。再跑一遍盯一下情况：**内存疯长**。到了几 G 的时候我赶忙杀掉了进程，好吧，应该是瞬间实例化了过多的 promise 对象，导致内存积压。

### 并发限制

也是，接口服务也会有请求数量限制。先定义一个池子，同一时间只处理 10 个 promise，完成一个就往池子里面再塞入一个。

```javascript
// 定义一个异步方法
const timeout = (t = 1000) =>
  new Promise((resolve) => setTimeout(() => resolve(true), t));

class PromisePool {
  constructor(max, fn) {
    this.max = max; // 最大并发量
    this.fn = fn; // 自定义的请求函数
    this.pool = []; // 并发池
    this.remaining = 0; // 剩余的任务数量
    this.result = []; // 结果池
  }
  start(num) {
    if (!num) return Promise.resolve();
    this.remaining = num; //先循环把并发池塞满
    while (this.pool.length < this.max && this.remaining) {
      this.remaining--;
      this.createTask();
    }

    return this.enqueue().then(() => {
      console.log("PromisePool Finished!");
      return Promise.resolve(this.result);
    });
  }
  enqueue() {
    if (this.pool.length >= this.max) {
      return Promise.race(this.pool).then(() => {
        //如果有剩余任务，则继续创建
        if (this.remaining) {
          this.remaining--;
          this.createTask();
        }
        return this.enqueue();
      });
    } else {
      return Promise.all(this.pool);
    }
  }
  createTask() {
    const t = parseInt(Math.random() * 300) + 200;
    const task = this.fn(t);
    this.pool.push(task); // 将该任务放入pool并发池中

    task.then((res) => {
      // 任务结束后将该promise任务从并发池中移除
      this.pool.splice(this.pool.indexOf(task), 1);
      this.result.push(res); // 将结果放入结果池
    });
  }
}

async function run() {
  console.time("run");
  let num = 1000;
  // 实际并发还是达不到，只是减少了内存的积压
  const pool = new PromisePool(10, timeout); // 并发数为10
  const result = await pool.start(num);
  console.timeEnd("run");
  console.log(result);
}
```

哈，10w 次测试跑起来内存只占了最多 500M，耗时 450s 左右，node 进程 CPU 占比 240%~290%左右。为什么并发设置为 10，耗时却只减少了一半左右呢。我猜测是电脑最快也只能同时处理 2 条了，再多没有多余 CPU 了。`(此处存疑，留待后续细究)`

但上面的方法还存在一个问题，最后的结果是按照任务完成的先后顺序排序的，而非实际任务创建的顺序。改一下。

```javascript
// 定义一个异步方法
const timeout = (t = 1000) =>
  new Promise((resolve) => setTimeout(() => resolve(true), t));

class PromisePool {
  constructor(max, fn) {
    this.max = max; // 最大并发量
    this.fn = fn; // 自定义的请求函数
    this.pool = []; // 并发池
    this.remaining = 0; // 剩余的任务数量
    this.result = []; // 结果池
  }
  start(num) {
    if (!num) return Promise.resolve();
    this.remaining = num; //先循环把并发池塞满
    while (this.pool.length < this.max && this.remaining) {
      this.remaining--;
      this.createTask();
    }

    return this.enqueue().then(() => {
      console.log("PromisePool Finished!");
      // Change:
      return Promise.all(this.result); // Promise.all返回result池的所有promise
    });
  }
  enqueue() {
    if (this.pool.length >= this.max) {
      return Promise.race(this.pool).then(() => {
        //如果有剩余任务，则继续创建
        if (this.remaining) {
          this.remaining--;
          this.createTask();
        }
        return this.enqueue();
      });
    } else {
      return Promise.all(this.pool);
    }
  }
  createTask() {
    const t = parseInt(Math.random() * 300) + 200;
    const task = this.fn(t);
    this.pool.push(task); // 将该任务放入pool并发池中
    // Change:
    this.result.push(task); // 将同时将任务放入结果池

    task.then((res) => {
      // 任务结束后将该promise任务从并发池中移除
      this.pool.splice(this.pool.indexOf(task), 1);
      this.result.push(res); // 将结果放入结果池
    });
  }
}

async function run() {
  console.time("run");
  let num = 1000;
  // 实际并发还是达不到，只是减少了内存的积压
  const pool = new PromisePool(10, timeout); // 并发数为10
  const result = await pool.start(num);
  console.timeEnd("run");
  console.log(result);
}
```
  
  
> 语雀地址 https://www.yuque.com/cheeseyu/fullstack/ub5q05
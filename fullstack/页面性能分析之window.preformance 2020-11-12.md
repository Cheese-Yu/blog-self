![df66c22c29c64be7bc4d16e7aedffae3_tplv-k3u1fbpfcp-zoom-1.jpeg](https://cdn.nlark.com/yuque/0/2021/jpeg/394019/1625036490781-2691e827-873a-4b95-87bc-88e7d38f519e.jpeg#clientId=ueba5b7f8-443d-4&from=ui&id=u44cca507&margin=%5Bobject%20Object%5D&name=df66c22c29c64be7bc4d16e7aedffae3_tplv-k3u1fbpfcp-zoom-1.jpeg&originHeight=555&originWidth=912&originalType=binary&ratio=2&size=52010&status=done&style=none&taskId=u3607eb19-bede-4fa2-ab4a-0a39f1ca9d6)

- **navigationStart**: 表示从上一个文档卸载结束时的 unix 时间戳，如果没有上一个文档，这个值将和 fetchStart 相等。
- **unloadEventStart**: 表示前一个网页（与当前页面同域）unload 的时间戳，如果无前一个网页 unload 或者前一个网页与当前页面不同域，则值为 0。
- **unloadEventEnd**: 返回前一个页面 unload 时间绑定的回掉函数执行完毕的时间戳。
- **redirectStart**: 第一个 HTTP 重定向发生时的时间。有跳转且是同域名内的重定向才算，否则值为 0。
- **redirectEnd**: 最后一个 HTTP 重定向完成时的时间。有跳转且是同域名内部的重定向才算，否则值为 0。
- **fetchStart**: 浏览器准备好使用 HTTP 请求抓取文档的时间，这发生在检查本地缓存之前。
- **domainLookupStart/domainLookupEnd**: DNS 域名查询开始/结束的时间，如果使用了本地缓存（即无 DNS 查询）或持久连接，则与 fetchStart 值相等
- **connectStart**: HTTP（TCP）开始/重新 建立连接的时间，如果是持久连接，则与 fetchStart 值相等。
- **connectEnd**: HTTP（TCP） 完成建立连接的时间（完成握手），如果是持久连接，则与 fetchStart 值相等。
- **secureConnectionStart**: HTTPS 连接开始的时间，如果不是安全连接，则值为 0。
- **requestStart**: HTTP 请求读取真实文档开始的时间（完成建立连接），包括从本地读取缓存。
- **responseStart**: HTTP 开始接收响应的时间（获取到第一个字节），包括从本地读取缓存。
- **responseEnd**: HTTP 响应全部接收完成的时间（获取到最后一个字节），包括从本地读取缓存。
- **domLoading**: 开始解析渲染 DOM 树的时间，此时 Document.readyState 变为 loading，并将抛出 readystatechange 相关事件。
- **domInteractive**: 完成解析 DOM 树的时间，Document.readyState 变为 interactive，并将抛出 readystatechange 相关事件，注意只是 DOM 树解析完成，这时候并没有开始加载网页内的资源。
- **domContentLoadedEventStart**: DOM 解析完成后，网页内资源加载开始的时间，在 DOMContentLoaded 事件抛出前发生。
- **domContentLoadedEventEnd**: DOM 解析完成后，网页内资源加载完成的时间（如 JS 脚本加载执行完毕）。
- **domComplete**: DOM 树解析完成，且资源也准备就绪的时间，Document.readyState 变为 complete，并将抛出 readystatechange 相关事件。
- **loadEventStart**: load 事件发送给文档，也即 load 回调函数开始执行的时间。
- **loadEventEnd**: load 事件的回调函数执行完毕的时间。

通过以上数据，我们可以得到几个有用的时间  
**重定向耗时 redirect**: timing.redirectEnd - timing.redirectStart  
**DOM 渲染耗时 dom**: timing.domComplete - timing.domLoading  
**页面加载耗时 load**: timing.loadEventEnd - timing.navigationStart  
**页面卸载耗时 unload**: timing.unloadEventEnd - timing.unloadEventStart  
**请求耗时 request**: timing.responseEnd - timing.requestStart  
**获取性能信息时当前时间 time**: new Date().getTime()

链接：[https://juejin.im/post/6892003555818143752](https://juejin.im/post/6892003555818143752)

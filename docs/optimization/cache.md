---
title: 浏览器缓存
order: 4
---

## 从缓存再谈 HTTP 优化

缓存可以减少网络 IO 消耗，提高访问速度。

下面是 Chrome 官方对于缓存操作必要性的分析：

> 通过网络获取内容既速度缓慢又开销巨大。较大的响应需要在客户端与服务器之间进行多次往返通信，这会延迟浏览器获得和处理内容的时间，还会增加访问者的流量费用。因此，缓存并重复利用之前获取的资源的能力成为性能优化的一个关键方面。

可见，**浏览器缓存是一种操作简单、效果显著的前端性能优化手段**。

按照获取资源时请求的优先级，浏览器缓存又分为，

1. Memory Cache
2. Service Worker Cache
3. HTTP Cache(\*重要)
4. Push Cache

如何区分上面几种缓存？

**打开浏览器控制栏的 Network 面板，Size 一栏对应的形如 ("from memory cache", "from disk cache") 等描述对应的就是上述 4 种缓存**。

我们先从最主要的，最具代表性，可能也是大家最熟悉的 HTTP 缓存说起。

## HTTP 缓存

按照缓存命中的优先级，又分为：**强缓存，协商缓存**。优先级较高的是强缓存，在命中强缓存失败的情况下，才会走协商缓存。

### 强缓存

强缓存是利用 http 头中的 Expires 和 Cache-Control 两个字段来控制的，当请求再次发出时，浏览器会根据其中的 expires 和 cache-control 判断目标资源是否“命中”强缓存，若命中则直接从缓存中获取资源，不会再与服务端发生通信。

命中强缓存时，返回的 HTTP 状态码为 200。

接下来我们看一下如何实现强缓存：**从 expires 到 cache-control**。

**expires**

实现强缓存，过去我们一直用 `expires`。

当服务器返回响应时，在 Response Headers 中将过期时间写入 expires 字段,

```bash
expires: Wed, 11 Sep 2019 16:12:18 GMT
```

`expires` 是一个时间戳，标识资源的过期时间。它是一个绝对时间，弊端：客户端与服务端存在时差会导致其不准确。

**max-age**

既然绝对时间戳不准确，那就用相对时间，在 HTTP1.1 标准试图将缓存相关配置收敛进 Cache-Control 这样的大背景下，max-age 作为 expires 的补充和替换出现。

```bash
cache-control: max-age=31536000
```

`max-age` 是一个相对的时间长度，上述缓存设置表示，资源将在 31536000 秒后失效。max-age 会缓存至客户端，且优先级大于 expires。

**cache-control**

除了 max-age 外，cache-control 还支持下面的配置，

- s-maxage，优先级高于 max-age，用于代理服务器，它表示 cache 服务器上(比如 cache CDN)缓存的有效时间。
- no-cache, 资源设置了 no-cache 后，每一次发起请求都不会再去询问浏览器的缓存情况，而是直接向服务端去确认该资源是否过期，也就是直接走协商缓存。
- no-store，不使用任何缓存策略。

其中，针对资源是否能够被代理服务缓存，强缓存又分为 public 缓存和 private 缓存(默认)。

### 协商缓存

命中强缓存失败的情况下，浏览器会尝试走协商缓存。协商顾名思义，就是去问一下服务器，进而判断是重新发起请求、下载完整的响应，还是从本地获取缓存的资源。

如果服务器提示资源未改动(Not Modified)，也就是 304 状态码，那么资源就会被重定向到浏览器缓存。

接下来我们来看协商缓存的实现：**从 Last-Modified 到 Etag **。

**Last-Modified**

最初的方案为 Last-Modified，巧了，它也是一个时间戳，表示最后修改时间。如果我们启用了协商缓存，它会在首次请求时随着 Response Headers 返回：

```bash
Last-Modified: Fri, 27 Oct 2017 06:35:57 GMT
```

随后我们每次请求时，都会带上一个叫 `If-Modified-Since` 的时间戳字段，它的值就是上一次返回的 `Last-Modified`。

服务器接收到这个时间戳后，会比对该时间戳和资源在服务器上的最后修改时间是否一致，从而判断资源是否发生了变化。如果发生了变化，就会返回一个完整的响应内容，并在 Response Headers 中添加新的 Last-Modified 值；否则，返回如上图的 304 响应，Response Headers 不会再添加 Last-Modified 字段。

`Last-Modified` 存在一些弊端，比如：

- 编辑了文件，但实质内容没有修改，服务器无法感知
- 修改文件过快，比如 100ms 完成了改动，不会触发重新请求，因为 If-Modified-Since 只能检查到以秒为最小计量单位的时间差

**Etag**

虽然上述的两种情况不是很常见，但它们确实是一些 bug，这个 bug 就是服务器无法正确感知文件变化，于是作为补充 Etag 出现。

**Etag 是由服务器为每个资源生成的唯一的标识字符串**，它的通信原理和 Last-Modified 类似，

当首次请求时，我们会在响应头里获取到一个最初的标识符字符串,

```bash
ETag: W/"2a3b-1602480f459"
```

然后下一次请求时，请求头里就会带上一个值相同的、名为 `If-None-Match` 的字符串供服务端比对，

```bash
If-None-Match: W/"2a3b-1602480f459"
```

Etag 也有弊端，就是生成过程需要服务器额外付出开销，会影响服务端的性能。

## Memory Cache

MemoryCache，是指存在内存中的缓存。从优先级上来说，它是浏览器最先尝试去命中的一种缓存。从效率上来说，它是响应速度最快的一种缓存。

内存缓存是快的，也是“短命”的。它和渲染进程“生死相依”，当进程结束后，也就是 tab 关闭以后，内存里的数据也将不复存在。

哪些内容会被放入 Memory Cache ？

没有定论，不过一般来说，资源存不存内存，浏览器会秉承 “节约原则”，比如，我们发现 Base64 格式的图片几乎永远都会被放进去，一些体积不大的 JS、CSS 文件，也有较大几率地被写入内存。相比之下，较大的 JS、CSS 文件就没有这个待遇了，内存资源是有限的，它们往往被直接甩进磁盘

## Service Worker Cache

Service Worker 是一种独立于主线程之外的 Javascript 线程。它脱离于浏览器窗体，因此无法直接访问 DOM，但这样的特性也使得它可以帮助我们可以帮我们实现离线缓存、消息推送和网络代理等功能。

借助 Service worker 线程实现的离线缓存就称为 Service Worker Cache。

Service Worker 的生命周期包括 install、active、working 三个阶段。一旦 Service Worker 被 install，它将始终存在，只会在 active 与 working 之间切换，除非我们主动终止它。

下面是一个例子。

首先在入口文件中插入这样一段 JS 代码，用以判断和引入 Service Worker

```js
window.navigator.serviceWorker
  .register('/test.js')
  .then(function() {
    console.log('注册成功');
  })
  .catch(err => {
    console.error('注册失败');
  });
```

在 `test.js` 中，我们进行缓存的处理。假设我们需要缓存的文件分别是 `test.html,test.css` 和 `test.js`：

```js
// Service Worker会监听 install事件，我们在其对应的回调里可以实现初始化的逻辑
self.addEventListener('install', event => {
  event.waitUntil(
    // 考虑到缓存也需要更新，open内传入的参数为缓存的版本号
    caches.open('test-v1').then(cache => {
      return cache.addAll([
        // 此处传入指定的需缓存的文件名
        '/test.html',
        '/test.css',
        '/test.js',
      ]);
    }),
  );
});

// Service Worker会监听所有的网络请求，网络请求的产生触发的是fetch事件，我们可以在其对应的监听函数中实现对请求的拦截，进而判断是否有对应到该请求的缓存，实现从Service Worker中取到缓存的目的
self.addEventListener('fetch', event => {
  event.respondWith(
    // 尝试匹配该请求对应的缓存值
    caches.match(event.request).then(res => {
      // 如果匹配到了，调用Server Worker缓存
      if (res) {
        return res;
      }
      // 如果没匹配到，向服务端发起这个资源请求
      return fetch(event.request).then(response => {
        if (!response || response.status !== 200) {
          return response;
        }
        // 请求成功的话，将请求缓存起来。
        caches.open('test-v1').then(function(cache) {
          cache.put(event.request, response);
        });
        return response.clone();
      });
    }),
  );
});
```

## Push Cache

Push Cache 是指 HTTP2 在 server push 阶段存在的缓存。这块的知识比较新，简单了解几个特性，

- Push Cache 是缓存的最后一道防线。浏览器只有在 Memory Cache、HTTP Cache 和 Service Worker Cache 均未命中的情况下才会去询问 Push Cache。
- Push Cache 是一种存在于会话阶段的缓存，当 session 终止时，缓存也随之释放。
- 不同的页面只要共享了同一个 HTTP2 连接，那么它们就可以共享同一个 Push Cache


## Session

由于 Cookie 可以被用户篡改，不是一个安全的选择，所以利用 Session 来解决防止用户篡改的需求。

Session 是属于服务器的一个哈希表。

在使用时，先定义一个`sessions`的哈希表，并生成一个随机数当做当前用户的`sessionId`，使用随机数是为了安全，这样用户就无法通过篡改得到其他用户正确的`sessionId`。接着将本应该放到 Cookie 里的信息放到`sessions[sessionId]`里面，存放到服务器上，并将`sessionId`通过`Cookie`传给浏览器。

代码实现如下：

```JavaScript
  let sessionId = Math.random() * 100000;
  sessions[sessionId] = { sign_in_email: email };
  response.setHeader('Set-Cookie',`sessionId=${sessionId}`);
```

通过这样的操作，浏览器并不直接得到用户信息，而是给一个`sessionId`，服务器通过`sessions`匹配浏览器附带的`sessionId`，得到用户信息。避免用户通过篡改 Cookie 获取其他用户的信息，而且`sessionId`是随机生成的，完全没有规律可循，也就无法伪造其他用户的`sessionId`。

```JavaScript
  //hash 就是将 Cookie 里的信息对象化得到的
  let mySession = sessions[hash.sessionId];
  let email;
  if (mySession) {
    email = mySession.sign_in_email;
  }
```

这样也可以得到当前用户的信息，而且用户还无法伪造成其他用户。

### 与 Cookie 的联系

- Session 是利用 `sessionId` 基于 Cookie 实现的
- Session 存储在服务器中，其 sessionId 通过 Cookie 发送给浏览器
- sessionId 随 Cookie 在每次发生请求时发送给服务器

### 小结

- `sessionId` 通过 `Cookie` 发送给浏览器
- 浏览器在访问服务器时，带上这个 `sessionId`
- 服务器有一块内存存放了 `sessionId`-用户信息 的键值对哈希表（sessions）
- 这样通过 `sessionId` 就可以得到对应用户的信息

Session 太占内存了，每一个用户都要存储 （`sessionId` - 用户信息），而且不能回收，当用户量很大时，对硬件性能要求特别高。

##  LocalStorage

LocalStorage 是存放在浏览器上的一个哈希表，且只能存放字符串（对于非字符串自动转换成字符串，`.toString()`）。

浏览器有一个`window.localstorage`的 api，如下所示：

![](https://i.loli.net/2018/07/30/5b5f2c0761e0e.png)

因为 LocalStorage 的值不随页面的关闭、刷新等发生变化，所以可以用来存储某些不需要随页面刷新而发生变化的值，持久化存储。

常用场景为：记录有没有提示过用户某信息（不能用来记录密码）

### 特点

- LocalStorage 与 HTTP 无关
- 发生 HTTP 请求时不会带上 LocalStorage 的值
- 只有相同域名的页面才能互相读取 LocalStorage（没有同源政策那么严格）
- 每个域名的 LocalStorage 最大的存储量为 5Mb 左右（每个浏览器各不一样）
- LocalStorage 永久有效，除非用户手动清除

### .setItem()

这是用来存储 LocalStorage 数据的，用法如下

```JavaScript
  //语法
  localStorage.setItem('key','value')

  //实例
  localStorage.setItem('name','yyzcl')
```

![](https://i.loli.net/2018/07/30/5b5f2dfc8edd8.png)

非字符串的 `key` 和 `value` 会被转换成字符串的形式，可以存 JSON。

```JavaScript
  localStorage.setItem('yyzcl',{age: '18'})

  localStorage.setItem('yyzcl2',JSON.stringify({age: '18'}))
```

![](https://i.loli.net/2018/07/30/5b5f2f0a9a9fa.png)

### getItem()

用来读取 LocalStorage 里面的数据，用法如下：

```JavaScript
  //语法
  localStorage.getItem('key')

  //实例
  localStorage.getItem('yyzcl2')
```

![](https://i.loli.net/2018/07/30/5b5f2ff99683e.png)

### .removeItem

用来删除 LocalStorage 已经存储的值，用法如下：

```JavaScript
  //语法
  localStorage.removeItem('key')

  //实例
  localStorage.removeItem('yyzcl')
```

![](https://i.loli.net/2018/07/31/5b5f36fcabf82.png)

### .clear()

清空 LocalStorage 的值。

### 与 Cookie 的关系

- Cookie 会随 HTTP 请求发送给服务器，而 LocalStorage 不会
- Cookie 的大小一般为 4kb，而 LocalStorage 大小一般为 5Mb（各浏览器不一样）
- Cookie 有有效期（可自行设置），LocalStorage 永久有效，除非用户手动清除

## SessionStorage

SessionStorage 与 LocalStorage 类似，也有 LocalStorage 那些 api等等特征。但是，SessionStorage 在用户关闭页面（结束会话）时就会失效

## HTTP 缓存

当我们刷新页面时，如果要重新请求一遍相同的资源，那就太消耗网络和时间了。那么有没有办法让 HTTP 在请求相同资源的时候，不经过网络连接服务器，直接从内存或硬盘中读取之前下载的资源呢？有这么几种方法可以实现。

### Cache-Control

Cache-Control 通用消息头被用于在 HTTP 请求和响应中通过指定指令来实现缓存机制。用法如下：

```JavaScript
  //语法
  Cache-Control: max-age=<seconds>

  //实例
  response.setHeader('Cache-Control', 'max-age=30');
```

如果我们给一个页面的 JS 文件响应设置了 Cache-Control 后，那么在30秒内重复请求这个 JS 文件的话，浏览器会阻止这个请求，转而从内存或硬盘中读取这个 JS 文件。

这是首次加载的情况。

![](https://i.loli.net/2018/07/31/5b5ff1c46569c.png)

并且在响应体中会有一个 `Cache-Control: max-age=30` 的信息，如下图所示。

![](https://i.loli.net/2018/07/31/5b5ff41894078.png)

当浏览器在30秒之内再次请求这个 JavaScript 文件时，会是如下情形。

![](https://i.loli.net/2018/07/31/5b5ff2625ca01.png)

从图中可以看到，当设置了 Cache-Control ，并且在有效时间之内重复请求同一个文件，处理时间为 0ms，也就是说浏览器根本没有向服务器发出请求，而是从内存中读取请求的文件。

JS、CSS 文件的缓存时间一般设置成一个很大的值，比如一年。

那么期间要更新 JS、CSS 文件怎么办？

这里只需要更改下 JS、CSS 的请求 URL 即可，比如加一个查询参数。效果如下：

![](https://i.loli.net/2018/07/31/5b5ff6bfbf2b7.png)

### Expires

Expires 也是用来控制缓存，与 Cache-Control 不一样的是，Expires 响应头包含日期/时间， 即在此时候之后，响应过期。无效的日期，比如 0, 代表着过去的日期，即该资源已经过期。用法如下：

```JavaScript
  //语法
  Expires: <http-date>

  //实例
  response.setHeader('Expires', 'Wen, 31 Jul 2018 15:55:10 GMT');
```

在有效期之内，重复的请求也会被浏览器阻止，转而从内存中读取文件。

同样的更改请求的 URL 即可在有效期内更新文件。

需要注意的是，Expires 的时间是以本地时间为基准的，如果本地时间出错，请求或许会出现 bug。比如将本地时间调整至有效期之后，那么每次请求都发送给服务器，不会被浏览器阻止。

并且当 Expires 与 Cache-Control 同时设置时，Expires 会被忽略，只有 Cache-Control 起作用。

### ETag

ETag HTTP 响应头是资源的特定版本的标识符。这可以让缓存更高效，并节省带宽，因为如果内容没有改变，Web 服务器不需要发送完整的响应。而如果内容发生了变化，使用 ETag 有助于防止资源的同时更新相互覆盖（“空中碰撞”）。

上面是 MDN 对 ETag 的介绍，通俗的来讲，就是 ETag 记录了请求的文件的版本号，浏览器在每次请求这个文件的时候，会带上这个版本号(作为 If-None-Match 字段的值），服务器如果发现文件版本号没有发生变化，也就是文件没有发生改变，那就不需要重新将文件发送给浏览器，让浏览器在缓存中读取之前下载的文件即可。

可以利用 [MD5](https://zh.wikipedia.org/wiki/MD5) 值作为文件的版本号，因为即使一个文件只发送了细微的变化，它的 MD5 值也会发生很大的变化。

ETag 的用法如下：

```JavaScript
  //语法
  ETag: "<etag_value>"

  //实例
  response.setHeader('ETag', fileMd5)
```

结果如下图所示：

![](https://i.loli.net/2018/07/31/5b5ffe835bb40.png)

可以看到，虽然是利用了缓存，但与前面的两种方法相比，处理时间并非为 0ms。这是因为它确实向服务器发出了请求，只是服务器在比较 ETag 值之后，没有将文件发送给浏览器而已。

### Last-Modified

Last-Modified 是一个响应首部，其中包含源头服务器认定的资源做出修改的日期及时间。

与 ETag 类似，也是设定一个特征值，服务器通过比较特征值来判断是否让浏览器运用缓存，只不过 ETag 用的是文件的版本号，Last-Modified 用的是文件最后的修改时间。用法如下：

```JavaScript
  //语法
  Last-Modified: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT

  //实例
  let lastModified = fs.statSync('./css/style.css', 'utf8').mtime.toGMTString();
  response.setHeader('Last-Modified', lastModified);
```

只要文件没有改变，也就是文件修改时间没有发生变化，那么服务器便不会向浏览器发送这个文件，而是让浏览器去读取缓存。

与 ETag 类似的是，它也会发出请求，只是服务器不返回文件而已。

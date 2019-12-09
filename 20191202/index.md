## 304&200(from cache）总结

### http头信息

- #### 通用字段
Cache-Control: 控制缓存具体行为
Pragma: HTTP1.1时遗留的字段 值为no-cache时每次请求页面不在读缓存
Date: 愿服务器发送该资源响应报文的时间（GMT格式）

- #### request字段
if-Match: 条件请求 携带上一次请求中资源的ETag 服务器根据这个字段判断文件是否有新的修改
if-None-Match: 和if-Match相反服务器根据这个字段是否有新的修改
if-Modified-Since: 比较资源前后两次访问最后的修改时间是否一致
if-Unmodified-Since: 比较资源前后两次访问最后的修改时间是否一致

- #### response字段
ETag: 服务器生成资源的唯一标识
Vary: 代理服务器的缓存的管理信息
Age: 表示这个响应已经存活了多久（HTTP1.0的响应不带Age）

- #### 实体字段
Expires: 返回的时间是服务端的时间 这样就存在一个问题 如果客户端的时间与服务器的时间相差很大（比如时钟不同步 或者跨时区）那么误差就很大 所以在HTTP1.1版开始 使用Cache-Control:max-age=秒替代
Last-Modified: 资源最后一次修改时间

##### 注：Cache Control优先级高于Pragma、Expires(HTTP/1.1) Cache-Control > Expires > Pragma
| 请求Cache Control      |    释意
| -------- | --------- |
| max-age | 可以接收缓存最长时间 |
| max-stale | 可以接收过期的资源 但是过期时间必须小于max-stale值 |
| min-fresh | 可以接收一个更新过的资源 fresh生命期大于当前Age跟min-fresh值之和 |
| no-cache | 在源服务器返回成功的验证之前不能使用缓存响应 |
| no-store | 直接禁止浏览器和所有中继缓存存储返回的任何版本的响应 |
| no-transform | 获取没有被转换过（比如压缩）的资源 |
| only-if-cached | 希望获取缓存内容而不发起请求 |

| 响应Cache Control      |    释意 |
| -------- | --------- |
| no-cache | 缓存必须重新校验 |
| no-store | 不存储缓存 |
| no-transform | 缓存内容时不能对改变任何数据 |
| public | 响应可被任何缓存区缓存 |
| private | 只在本地缓存，不允许任何中继缓存对其进行缓存（例如，浏览器可以缓存，但是CDN不能缓存）|
| mush-revalidate | 如果缓存的内容失效，请求必须发送到服务器以进行重新验证请求失败返回504，而非中CDN |
| max-age | 只接受 Age 值小于 max-age 值，并且没有过期的资源 |
| proxy-revalidate | 与must-revalidate类似，仅能用于共享缓存（如：CDN）|
| s-maxage | 仅能用于共享缓存，一般用在cache服务器上(如：CDN) |

### 缓存对比

- #### 协商缓存（304）
if-Modified-Since/Last-Modified
浏览器在发送请求的时候服务器会检查请求头request header里面的if-Modified-Since 如果最后修改时间相同则返回304 否则给返回头（response header）添加last-Modified并返回数据（response body）
if-None-Match/ETag

浏览器在发送请求的时候服务器会检查请求头request header里面的if-None-Match的值与当前文件的内容通过hash算法（例如 nodejs: cryto.createHash('sha1')）生成的内容摘要字符对比 相同则直接返回304 否则给返回头（reponse header）添加ETag属性为当前的内容摘要字符 并且返回内容

请求头last-modified的日期与响应头的last-modified一致 请求头if-none-match的hash与响应头的etag一致 所用会返回Status Code: 304

- #### 强缓存（200from cache）
如果设置了Expires(XX时间过期)或者Cache-Control（http1.0不支持）(经历XX时间后过期)且没有过期 命中cache的情况下 from cache不去发出请求 如果强刷（如ctrl+r）会发起请求 但是如果没有修改会返回304内容未修改 如果已经改变则返回新内容

expires/cache-control虽然是强缓存 但用户主动触发的刷新行为 还是会采用缓存协商的策略 主动触发的刷新行为包括点击刷新按钮、右键刷新、f5刷新、ctrl+f5刷新等

当然如果在控制台里面选中了disable cahce则无论如何都会请求最新内容(304协商缓存、强缓存都无效)，因为1.不会检查本地是否有缓存 2.请求头信息(request header)既没有If-Modified-Since也没有If-None-Match来让服务端判断 地址栏输入的地址按下回车键，该地址页面请求（仅仅是该url）的request header都会带上cache-contro:max-age=0，所以不会命中强缓存 但是通过链接点击的地址会命中缓存

### 区别
- #### 触发200 from cache
1、直接点击链接访问
2、输入网址按回车访问
3、二维码扫描

- #### 触发304
1、刷新页面时触发
2、设置了长缓存、但Entity Tags没有移除时触发

### 流程图
![流程图]('../img/liuchengtu.png')

### Meta标签
meta是用来在HTML文档中模拟HTTP协议的响应头报文。在HTML页面加上meta标签来给请求报头加上请求字段
``` html
<meta http-equiv="Expires" content="0">
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Cache-Control" content="no-cache">
```

### 缓存方案
静态资源CDN部署 按时按需更新缓存

|  缓存对象 ｜  Cache Control配置  |
|  ------   ｜  --------  |
|  /page HTML |  no-cache  |
|  /dist/css xx.css |  max-age=31536000  |
|  /dist/js xx.js |  private,max-age=31536000  |
|  /dist/img xx.png |  max-age=86400  |

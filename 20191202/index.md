## 304&200(from cache）总结
***

### http头信息

- #### 通用字段
Cache-Control: 控制缓存具体行为 优先级：Cache-Control > Expires > Pragma
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
WeChat29292f7c9493087fb65fba0dbb00ad27.png

### Meta标签
meta是用来在HTML文档中模拟HTTP协议的响应头报文。在HTML页面加上meta标签来给请求报头加上请求字段
``` html
<meta http-equiv="Expires" content="0">
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Cache-Control" content="no-cache">
```
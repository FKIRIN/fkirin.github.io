<!--
 * @ [moduleName] - [description]
 * @Author: fengqilin <qilin.feng@hand-china.com>
 * @Date: 2019-10-14 14:20:58
 * @LastEditTime: 2019-10-14 16:49:47
 * @Copyright: Copyright (c) 2018, Hand
 -->
---
title: 不了解的http基础
date: 2019-10-12 09:48:37
tags: http
---

## http

缓存能够极大的改善网页性能，提高用户体验。

### 浏览器缓存

第一次获取资源后，根据返回的信息来如何缓存资源，可能采用的是强缓存，也可能告诉客户浏览器是协商缓存，都是根据响应的header内容来决定的。
![图一](images/2019-10/14-1.png)
浏览器在获取到资源后续请求
![图二](images/2019-10/14-2.png)

浏览器缓存包括两种类型，强缓存和协商缓存，浏览器在第一次请求之后，再次请求资源时：

- 浏览器请求某一资源时，首先获取该资源缓存的header信息，判断否属于强缓存（cache-control和expires信息），若命中则追截从缓存中获取资源信息，包括缓存header信息；本次请求不会与服务器进行通信。
- 如果没有命中强缓存，浏览器会发送请求到服务器，请求会携带第一次请求返回的有关缓存的header信息（Last-Modified/If-Modified-Since和Etag/If-None-Match），由服务器根据请求中的相关header信息来比对结果是否协商缓存命中；若命中，则服务器返回新的响应header信息更新缓存中的对应header信息，但是并不返回资源内容，告知浏览器可以直接从缓存获取；否则返回最新的资源内容。
区别: ![如果](images/2019-10/14-3.htm)

#### 强缓存相关header字段

- expires 这是http1.0时的规范；它的值为一个绝对时间的GMT格式的时间字符串，如Mon，10 Jun2015 21:31:12 GMT，如果发送请求的时间在expires之前，那么本地缓存始终有效，否则就会发送求到服务器来获取资源。

- cache-control: max-age=number，这是http1.1时出现的header信息，主要利用该字段的max-age值来进行判断，它是一个相对值；资源第一次的请求时间和Cache-Control设定的有效期，计算出一个资源过期时间，再拿这个过期时间跟当前的请求时间比较，如果请求时间再过期时间之前，就能命中缓存，否则不行。除了该字段外，还有其它几个常用的设置值：

1. no-cache： 不使用本地缓存，需要使用缓存协商，先与服务器确认返回的响应是否被更改，如果之前的响应中存在ETag，那么请求的时候会与服务器端验证，如果资源未被更改，则可以避免重新下载。
2. no-store： 直接禁止浏览器缓存数据，每次用户请求该资源，都会向服务器发送一个请求，每次都会下载完整的资源。
3. public: 可以被所有的用户缓存，包括终端用户和CDN等中间代理服务器。
4. private: 只能被终端用户的浏览器缓存，不允许CDN等中间服务器对其缓存
> 注意： 如果cache-control与expires同时存在的话，cache-control的优先级高于expires

#### 协商缓存相关的header字段

协商缓存都是由服务器来确定缓存资源是否可用的，所以客户端与服务器要通过某种标识来进行通信，从而让服务器判断请求资源是否可以缓存访问，主要涉及到下面两组header字段，这两组都是成对出现的，即第一次请求的响应头带上某个字段(Last-Modified或者Etag)，则后续请求会带上对应的请求字段(If-Modified-Since或者If-None-Match),若响应头没有Last-Modified或者Etag字段，则请求头也不会有对应的字段。

1. Last-Modified/If-Modified-Since：
   - 浏览器第一次跟服务器请求一个资源，服务器在返回这个资源的同时，在response的header上加上Last-Modified的header，这个header表示这个资源在服务器上的最后修改时间。
   - 浏览器再次跟服务器请求这个资源，在request的header上加上If-Modified-Since的header，这个header的值就是上一次请求时返回的Last-Modified的值
   - 服务器再次收到资源请求时，根据浏览器传过来If-Modified-Since和资源在服务器上的最后修改时间判断资源是否有变化，如果没有变化则返回304 Not Modified，但是不会返回资源内容，如果有变化，就正常返回资源内容。当服务器返回304Not Modified的响应时，response header中不会再添加Last-Modified的header，因为既然资源没有变化，那么Last-Modified也就不会改变。这是服务器返回304的response header
   - 浏览器返回304后，会从缓存中加载资源
   - 如果协商缓存没有命中，浏览器直接从服务器加载资源，Last-Modified的header在重新加载的时候会被更新，下次请求时，If-Modified-Since会启用上次返回的Last-Modified值
2. Etag/If-None-Match这两个值时由服务器生成的每个资源的唯一标示字符串，只要有资源变化有这个值就会改变；其判断过程与Last-Modified/If-Modified-Since类似，与Last-Modified不一样的是，当服务器返回304 Not Modified的响应时，由于Etag重新生成过，response header还会把这个Etag返回，即使这个Etag跟之前的没有花。

#### Last-Modified已经让浏览器知道缓存副本是否刷新，Http1.1中Etag解决来Last-Modified比较难解决的问题：

- 一些文件做周期性的更改，但是他的内容并不改变，这个时候我们并不希望客户端认为这个被修改了，而重新get；
- 文件改动频繁，秒一下的时间内进行修改，（1s内修改n次），if-Modified-Since能检查到的粒度是S级的，这种修改无法做判断。
- 某些服务器不能精确的到文件的最后修改时间。
这时，利用Etag能够更加准确的控制缓存，因为Etag是服务器自动生成或者由开发者生成的对应资源在服务器端的唯一标示符。
> Last-Modified与Etag是可以一起使用的，服务器会优先验证Etag，一致的情况下，才会继续比对Last-Modified，最后才决定是否返回304.

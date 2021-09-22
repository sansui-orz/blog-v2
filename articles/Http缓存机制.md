# Http缓存机制

[tag]:http|cache
[create]:2019-08-27

原文链接: https://www.cnblogs.com/ranyonsue/p/8918908.html

---

## 缓存分类:

**强缓存:**
在浏览器发送请求前，会判断该请求是否命中强缓存，如果命中，则直接加载缓存内容，不对服务器发起请求。
强缓存由expires和cache-control控制。
两者可以共存，也可以只存在一个，其中cache-control的优先级更高。

***expires:***
缓存过期时间，在该时间前可以直接从浏览器缓存中读取，无需再次请求。
该值是一个绝对时间，如果修改了客户端时间，则会导致缓存混乱。

***cache-control:***
一个相对时间，指在浏览器缓存中存活时间。
在存活时间内，则可以直接读取缓存。
该字段取值一般取：
max-age(缓存存活时间),

***no-cache(不走强缓存，直接走协商缓存)***,
no-store(禁止缓存，每次请求都从服务器获取新的)，
public(允许浏览器以及代理服务器缓存该资源),
private（仅允许浏览器缓存）
其他不常用取值为：s-maxage, public, private, must-revalidate。具体看原文

**协商缓存:**
如果没有命中强缓存，则浏览器会发送请求到服务器。
服务器根据http头信息中的Last-Modify/If-Modify-Since或者Etag/If-None-Match来判断是否命中协商缓存.
如果命中则返回状态码为304，浏览器则从缓存中加载资源。

***Last-modify:***
最后一次请求时，服务器中该资源的最后修改时间

***If-Modify-Since:***
为服务器中保存的该资源的最后修改时间。
这个时间如果与Last-modify相同，则表示该资源并没有更改，可以直接使用。
这个方法还是有点不太严谨。所以有了Etag/If-None-Macth

***Etag/If-None-Match:***
这个优先级比Last-modify高，其判读原理类似，但是对比的是一个根据每个文件生成的唯一的校验值。
即每一次修改文件，该值一定会改变。具体生成规则看原文。

**缓存是否生效也和用户操作有关:**
1. 使用F5刷新页面时，强缓存无效
2. 使用ctrl+f5刷新页面时，强缓存与协商缓存都无效

**HTTP缓存流程图:**

![图片404](https://files-1255982271.cos.ap-guangzhou.myqcloud.com/940884-20180423141951735-912699213.png!trans_webp)

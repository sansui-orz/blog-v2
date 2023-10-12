# 浅谈web安全

[tag]:web|XSS|CSRF
[create]:2021-09-23

常见web安全问题

1. 跨站脚本攻击（XSS）
2. CSRF攻击（跨站请求伪造）
3. 点击劫持
4. CDN劫持

## 跨站脚本攻击（XSS）

1. 反射型XSS攻击，携带在URL上，网站直接取出URL上的内容并展示在页面上
2. 储存型XSS攻击，用户输入的信息存储在数据库中，每次访问都将数据渲染到HTML，导致恶意脚本执行

可以看到这两种XSS的攻击方式，都是由于将不受信任的外部输入信息直接渲染到HTML中，导致恶意脚本被执行。

### 预防措施

1. 尽量不要将外部输入的信息直接渲染到页面，避免使用诸如`.innerHTML`, `.outHTML`,`docutemt.write()`等方法，尽量使用`.textContent`,`setAttribute()`等方法。使用`Vue`,`React`等框架时，也精良避免使用`v-html`/`dangerouslySetInnerHTML`功能。
2. 类似于`setTimeout` `setInterval` `eval`等可以直接运行字符串的方法，也尽量不要运行外部代码。

# CSRF攻击（跨站请求伪造）

### CSRF是怎么攻击的？

1. 受害者登录A网站，并且保留了登录凭证（Cookie）
2. 攻击者引诱受害者访问B网站
3. B网站向A网站发送了一个请求，浏览器请求头中会默认携带A网站的Cookie
4. A网站服务器收到请求后，经过验证发现用户是登录的，所以会处理请求

### 常见的CSRF攻击类型

1. GET类型的CSRF，直接使用图像ping的方式，在页面中加入一张指向攻击网站的URL地址
2. POST类型的CSRF，在发起攻击的页面中写好POST请求的Form表单，并写入脚本自动点击submit
3. 链接型的CSRF，即将A标签的链接换成将要攻击的URL，用户点击时便会发送GET请求

### 防范措施

1. 同源检测
2. 设置需要校验特殊字段，由于CSRF都是浏览器自行发起的请求，所以对于特殊字段（如TOKEN）是不会携带的
3. 给Cookie设置合适的SameSite

## 点击劫持

攻击者将目标网站通过iframe嵌入到自己的网页中，通过opacity等手段设置透明不可见，诱导用户点击iframe。

### 防范措施

1. 在HTTP头中加入`X-FRAME-OPTIONS` 属性，此属性控制页面是否可被嵌入iframe中
    1. DENY：不能被所有网站嵌套或加载
    2. SAMEORIGIN：只能被同域网站嵌套或加载
    3. ALLOW-FROM URL：可以被指定网站嵌套或加载
2. 判断当前页面是否被iframe嵌套

## HTTP严格传输安全(HSTS)

> HTTP严格传输安全（HSTS）是一种安全功能，web服务器通过它来告诉浏览器仅用HTTPS来与之通讯，而不是使用HTTP。

HSTS 会强制让浏览器使用安全链接（HTTPS）。需要注意的是，**如果之前没有使用HTTPS协议访问过该站点，那么HSTS是不会奏效的**。所以我们可以辅以重定向来辅助实现强制HTTPS。

其在响应头中的属性为**Strict-Transport-Security**。事实上，添加HSTS标头有性能优势。如果有人试图通过HTTP访问您的站点，而不是发出HTTP请求，它只是重定向到HTTPS版本。

## CDN劫持

## CDN劫持防范措施

1. 使用SRI（子资源完整性）来解决CDN劫持。是指浏览器通过验证资源的完整性（通常从CDN获取）来判断是否被篡改的安全特性。
    
    通过给Link标签或者script标签增加`integrity` 属性即可开启SRI功能。
    
    如:
    
    ```html
    <script type="text/javascript" src="//s.url.cn/xxxx/aaa.js"  integrity="sha256-xxx sha384-yyy"
        crossorigin="anonymous"></script>
    ```
    
    integrity的值分为两个部分，第一部分指定哈希值的生成算法（sha256\sha384\sha512), 第二部分是经过base64编码的实际哈希值。其值可以包含多个空格分隔的hash值，只要文件匹配其中任意一个哈希值，就可以通过校验并加载该资源。
    
2. 浏览器在解析script或者link标签中遇到integrity属性之后，会在执行脚本或者应用样式表之前对比所加载文件的哈希值和期望的哈希值。
    
    当哈希值不一致时，浏览器必须拒绝执行，并返回一个网络错误说明获取脚本或者样式表失败。
    

## 内容安全策略（CSP）

### CSP的意义

CSP的实质就是白名单机制。开发者明确告诉客户端，哪些外部可以加载和执行。

### 使用

1. 通过HTTP头设置`Content-Security-Policy` ,以下配置说明该页面只允许当前源和https://apis.google.com这两个源的脚本加载和执行。
    
    ```
    Content-Security-Policy: script-src 'self' https://apis.google.com
    ```
    
2. 通过页面`<meta>` 标签配置
    
    ```html
    <meta http-equiv="Content-Security-Policy" content="script-src 'self' https://apis.google.com">
    ```
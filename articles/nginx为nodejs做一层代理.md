# nginx为nodejs做一层代理

[tag]:nginx|server|node
[create]:2020-07-03

Nginx是一个HTTP服务器，可以将服务器上的静态文件（如HTML、图片）通过HTTP协议展现给客户端。

通常用来做以下工作：

- 静态HTTP服务器

- 反向代理服务器

- 负载均衡

- 虚拟主机

注意负载均衡与虚拟主机都是基于反向代理运行的。

## 项目中的实践

之前遇到一个问题，就是nginx与nodejs的结合使用。

比如nodejs服务开在8888端口，但是nginx服务开在7777端口。怎么把8888/test请求给7777代理呢？

就是nodejs的服务里面并没有处理/test这个路径的请求的，需要将它代理到www.example.com/test

然后在ssr的项目中是直接在当前域名下发起的请求，即localhost:8888/test的请求地址，最终要把它代理到www.example.com/test。

一开始傻傻的考虑8888端口已经被nodejs占用了，nginx怎么代理到8888端口的请求呢？很是苦恼，最后查了一下百度。恍然大悟。

不需要执着于ssr的请求要访问自己的端口，可以换个思路。将所有端口都用nginx做一遍过滤，最终就可以达到将自己想要代理的接口都代理出去的效果啦。

## 参考资料

[nginx有哪些作用？](https://zhuanlan.zhihu.com/p/54793789)

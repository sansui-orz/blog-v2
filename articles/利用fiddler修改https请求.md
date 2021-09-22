# 利用fiddler修改https请求

[tag]:css|loading|html
[create]:2019-12-13

## 在项目中遇到一个引用场景。
开发webview页面时，但是需要在特定域名下客户端才会注入一些参数。所以在开发与联调的时候修改url就很重要了。

然后我主要在使用fiddler进行url的重定向

比如有个APP叫A，然后里面嵌套了一个webview的页面。
而只有在这个webview里面才能拿到初始化参数，并且它底层有提供一些特殊的js接口。
这时候就要将打开的页面代理到本机开启的http服务进行开发了。
比如这个webview打开的url为:
https://example.com/1

你需要代理到本地的:
http://127.0.0.1:8080/1

这个情况就需要将https修改为http, 这个时候需要修改https链接的返回值，但是不能改变https链接本身。因为在这个webview中，会根据url的不同而决定是否注入初始化参数。所以为了得到初始化参数，https链接本身不能变。
这时候最好的解决方案就是使用fiddler的AutoResponder, 将https链接的请求返回修改为本地的服务。
例子:
```
rule Editor
example.com/1
http://127.0.0.1:8080/1
```

## 第二个场景

当做测试的时候，由于各种问题，不能直接在https的正式环境上测试，只有一个http://example.com/1 的测试环境。这时候如果要获得正式环境一样的测试环境与参数，就需要我们手动将http修改成https。然后在环境上就和正式环境一致了。
这个修改我们需要借助fiddler的FiddlerScript来进行。
例子:
在FiddlerScript里的OnBeforeRequest去修改url链接:
```javascript
if (oSession.HostnameIs('example.com/1') && oSession.isHTTPS) {
    oSession.fullUrl = 'https://' + oSession.hostname + oSession.PathAndQuery;
}
```
如上，在测试环境中的url就会被修改成https的链接了。
当然，如果你需要直接代理到本地服务也是可以的，只需要加上前面的AutoResponder就可以轻松做到。

但是需要注意的是，当使用react或者vue等框架进行开发时，有可能页面引用的js包和css包会出现404的问题。
这是由于其引用一般都是相对路径的引用。就像:
```
<script src="/public/js/a.js"></script>
```
如果出现了这个问题，只需要将其引用路径改为绝对路径就可以了
```
<script src="http://127.0.0.1:8080/public/js/a.js"></script>
```
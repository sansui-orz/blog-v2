# chrome中跨域限制表格

[tag]:chrome|cors
[create]:2020-07-03

| 原地址 | 目标地址 | 说明 | 是否允许请求 |
| --- | --- | --- | --------- |
| <http://a.example.com/> | <<http://a.example.com/a.txt> | 同域下 | 允许 |
| <http://a.example.com/> | <http://a.example.com/b/a.txt> | 同域下不同目录 | 允许 |
| <http://a.example.com/> | <http://a.example.com:8080/a.txt> | 同域下不同端口 | 不允许 |
| <http://a.example.com/> | <https://a.example.com/a.txt> | 同域下不同协议 | 不允许 |
| <http://a.example.com/> | <http://b.example.com/a.txt> | 不同域下 | 不允许 |
| <http://a.example.com/> | <http://a.foo.com/a.txt> | 不同域下 | 不允许 |

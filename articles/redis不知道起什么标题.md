# redis不知道起什么名字

[tag]:redis|node|js
[create]:2020-09-23

redis产生的原因在于优化mysql对于日益增长的数据处理需求的高重复I/O操作的损耗。比如说一个双十一爆款产品，在没有redis之前，每个用户请求该产品的信息都会重新去查询一次数据库，但是这其实很没有必要，因为短时间内该数据并不会发生改变。于是给redis应运而生。

redis是一个内存中键值数据库，因此它可以非常快速的检索数据。

![redius](https://lms-flies.oss-cn-guangzhou.aliyuncs.com/blog/imgs/redis.drawio.png!trans_webp)

redis的作用如上图，它主要应用场景就在于重复的数据查询操作时，提供数据缓存，用以减少响应时间以及优化程序性能。

当然，这其中还有很多问题需要我们一一去解决：

- 缓存过期 && 缓存淘汰

- 缓存过滤 && 布隆过滤器

- 缓存击穿 && 缓存雪崩

## 参考链接

[还不懂Redis？看完这个故事就明白了！](https://juejin.im/post/6872518761593700359)

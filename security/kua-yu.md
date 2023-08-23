# 跨域

### 同源策略与跨域

同源策略是浏览器的一种保护措施，目的是为了保证页面的数据都是来自同一数据源，不会被JS执行代码获取回来的数据污染。

同源指：协议://域名:端口 三者都

浏览器从一个域名的⽹页去请求另一个域名的资源时，域名、端⼝、协议任一不同，都是跨域。正常情况下，不同源的请求，返回的数据，浏览器是不会进行展示的。

举例：在一个tab页http://a.com，发起了一个请求http://api.a.com到对应的服务器，服务器收到请求后会校验http://a.com是否在数据允许展示的白名单中，如果是,服务器则在响应头中进行以下配置

```
//表示接受任意域名的请求,也可以指定域名
response.setHeader("Access-Control-Allow-Origin",
request.getHeader("origin"));
//该字段可选，是个布尔值，表示是否可以携带cookie
response.setHeader("Access-Control-Allow-Credentials",
"true");
response.setHeader("Access-Control-Allow-Methods", "GET,
HEAD, POST, PUT, PATCH, DELETE, OPTIONS");
response.setHeader("Access-Control-Allow-Headers", "*");
```

浏览器获取到响应的数据后，查看响应头，发现当前数据是允许在Origin域名中进行展示的。

![](https://cdn.jsdelivr.net/gh/yunCrush/yc-image/image/http-%E5%90%8C%E6%BA%90%E4%B8%8E%E8%B7%A8%E5%9F%9F.png)

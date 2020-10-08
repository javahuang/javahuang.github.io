# 每日随笔

## HTTP GET 支持 request body 吗

GET 请求我想定义了一个复杂的请求参数，然后给放到 request body 里面，但是我使用 fetch 放到 param 里面，貌似这个不生效， get 请求只支持 Query String Parameter？

```ts
type Query = {
  id: string;
  name: string;
  filter: {
    [key: string]: string;
  };
};
```

不同语言的实现的 http libraries 对于 GET 请求的 request body 处理可能不太一样，有些支持，有些不支持。[RFC](https://tools.ietf.org/html/rfc7231#page-24) 规范也没明确说 GET 请求是否支持 request body。

参考 [HTTP GET with request body](https://stackoverflow.com/questions/978061/http-get-with-request-body)

[elastic - A GET Request with a Body?](https://www.elastic.co/guide/en/elasticsearch/guide/current/_empty_search.html)

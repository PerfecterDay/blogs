# Okhttp简介

Okhttp是一个处理网络请求的开源项目,是安卓端最火热的轻量级框架,由移动支付Square公司贡献(该公司还贡献了Picasso)用于替代 HttpUrlConnection 和 Apache HttpClient (android API23 6.0里已移除HttpClient,现在已经打不出来)

在现代应用程序网络处理中，HTTP是我们交换数据和媒体的方式。高效地执行HTTP可以使您的数据加载更快并节省带宽。

在默认情况下，OKHTTP是高效的HTTP客户端：
1. HTTP/2支持允许对同一主机的所有请求共享一个套接字。
2. 连接池减少了请求延迟（如果HTTP/2不可用）。
3. 透明gzip缩小下载大小。
4. 响应缓存完全避免了重复请求的网络。

当网络出现问题时，OKHTTP会坚持下去：它会自动从常见的连接问题中恢复。如果您的服务有多个IP地址，在第一次连接失败时，OKHTTP将尝试备用地址。这对于IPv4+IPv6和托管在冗余数据中心中的服务是必需的。OKHTTP支持现代TLS功能（TLS 1.3、ALPN、证书固定）。它可以配置为回调以实现广泛的连接。

使用OKHTTP很容易。它的request/response API设计为具有流畅的构建者模式和不变性。它支持同步阻塞调用和带回调的异步调用。

OKHTTP支持Android 5 +（API级别21 +）和Java 8 +。


### Get 请求
发起一个 Get 请求时简单的：
```
OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {
  Request request = new Request.Builder()
      .url(url)
      .build();

  try (Response response = client.newCall(request).execute()) {
    return response.body().string();
  }
}
```

### Post 请求
```
public static final MediaType JSON
    = MediaType.get("application/json; charset=utf-8");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {
  RequestBody body = RequestBody.create(json, JSON);
  Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();
  try (Response response = client.newCall(request).execute()) {
    return response.body().string();
  }
}
```
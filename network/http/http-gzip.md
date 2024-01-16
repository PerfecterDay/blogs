## Http gzip 压缩
{docsify-updated}

`curl -vv https://www.baidu.com/ -H "Accept-Encoding: gzip,deflate"` : gzip 格式的响应
<center><img src="pics/http-gzip1.jpg" width="70%"></center>


`curl https://www.baidu.com/ -H "Accept-Encoding: gzip,deflate" | gunzip` 或者 `curl https://www.baidu.com/ -H "Accept-Encoding: gzip,deflate" --compressed`
<center><img src="pics/http-gzip2.jpg" width="70%"></center>



### 自己写的 Proxy 服务
```
 @PostMapping("/**")
public ResponseEntity<String> proxy(@RequestBody String body, HttpServletRequest servletRequest, HttpServletResponse response) {
    try {
//            HttpMethod method = HttpMethod.resolve(servletRequest.getMethod());
        HttpMethod method = HttpMethod.GET;
        String uriStr = servletRequest.getRequestURI().replace("/app/user", "");
        URI uri = new URI("https://www.baidu.com");

        HttpHeaders headers = new HttpHeaders();
        Enumeration<String> headerNames = servletRequest.getHeaderNames();

        while (headerNames.hasMoreElements()) {
            String headerName = headerNames.nextElement();
            headers.set(headerName, servletRequest.getHeader(headerName));
        }

        HttpEntity<String> httpEntity = new HttpEntity<>(body, headers);
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> serverResponse = restTemplate.exchange(uri, method, httpEntity, String.class);
        return serverResponse;

    } catch (URISyntaxException e) {
        throw new RuntimeException(e);
    }
}
```

```
curl -vv --location 'http://localhost:8050/user/exclusiveGift/aa' \
--header 'Content-Type: application/json' \
--data '{
    "pageNum": 1,
    "pageSize": 10
}'
```
<center><img src="pics/http-gzip5.jpg" width="70%"></center>



```
curl -vv --location 'http://localhost:8050/user/exclusiveGift/aa' \
--header 'Content-Type: application/json' \
--header 'Accept-Encoding: gzip,deflate' \
--data '{
    "pageNum": 1,
    "pageSize": 10
}'
```
<center><img src="pics/http-gzip3.jpg" width="70%"></center>


```
curl -vv --location 'http://localhost:8050/user/exclusiveGift/aa' \
--header 'Content-Type: application/json' \
--header 'Accept-Encoding: gzip,deflate' \
--data '{
    "pageNum": 1,
    "pageSize": 10
}' --compressed
```
<center><img src="pics/http-gzip4.jpg" width="70%"></center>

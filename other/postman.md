# postman
{docsify-updated}

1. 断言响应状态码
```
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});
```

2. 校验返回 JSON 字段
```
const jsonData = pm.response.json();

pm.test("Code is 0", function () {
  pm.expect(jsonData.code).to.eql(0);
});

pm.test("Message is success", function () {
  pm.expect(jsonData.message).to.eql("success");
});
```

3. 提取响应值存为环境变量
```
const jsonData = pm.response.json();
pm.environment.set("newId", jsonData.data.id);
```

4. 根据响应动态判断
```
const jsonData = pm.response.json();

if (jsonData.code !== 0) {
  console.warn("接口报错:", jsonData.message);
} else {
  console.log("成功创建:", jsonData.data.id);
}
```
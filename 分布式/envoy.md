# Envoy

```
sudo docker run -idt --name envoy --restart=always -e TZ="Asia/Shanghai" -e loglevel=debug -v /etc/localtime:/etc/localtime -v "/root/dev/envoy/logs:/tmp" -v "/root/dev/envoy/test.cer:/etc/envoy/test.cer" -v "/root/dev/envoy/testserver.key:/etc/envoy/test.key" -v "/root/dev/envoy/envoy.yaml:/etc/envoy/envoy.yaml" --net=host  4b976b6b0e19
```

`-e loglevel=debug`: 设置日志级别

logs 文件夹报 permission denied 错误 -> `chmod 777 logs`
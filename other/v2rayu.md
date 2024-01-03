
V2RayU 无法打开问题

证书过期问题导致的：

1. 执行
```sudo codesign --force --deep --sign - /Applications/V2rayU.app```
2. 在应用程序中找到V2rayU，右键，显示简介，勾选覆盖恶意软件保护(我的没有此选项)
3. 打开软件
4. 执行

```
sudo codesign --force --deep --sign - ~/.V2rayU/V2rayUTool
sudo codesign --force --deep --sign - ~/.V2rayU/v2ray-core/v2ray
```
然后就能正常运行了。
来自：https://discussionschinese.apple.com/thread/254968153

v2ray 无法使用？
删除 `~/.V2rayU` 下的所有文件，重启app
#  SSH 简介
{docsify-updated}

- [SSH 简介](#ssh-简介)
	- [SSH免密登录](#ssh免密登录)


移步至阮一峰老师的[博客](https://wangdoc.com/ssh/client.html)

### SSH免密登录
1. `ssh-keygen` ： 生成公私钥对
2. `ssh-copy-id 10.4.153.130` : 会在目标登录机器上的相应用户的 .ssh 文件夹下的authorized_keys文件中写入相应的公钥信息

```
[root@LXUATGMAA2 ~]# cat ~/.ssh/authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC7f4fSmAsItzNsYlVbuyqkKOV2K2Qx/v0gZEmlirmuR0umUi24Y2bpXpeKODQTvP8uM+WUmgeJUEPH1XFV26wcwXAhDBcCmxBL/aSBAIDcHccSM26qPlqltIf8N2gPAAx/7uhrEziOxmJB2TElpnKzo3GMLPt/v8gBBiAARO8LVGPo8KODoAWv9z/nXYlS1wk+9UnlomGsseEciQUOro/SgmMRynFvfA0MdfV3CCfA+/ziQHp0HQvbeeFl6MY260dOJ5D/cwmXxlYWpab+3lMwk/D18GJ0IzvpRyhLEUzUoxbtjkFr7HAkCXWdJmLkv3IRWYSB5E0EbPy4JhhRyEUCpHH2zYm4kdJ2RPFDtUjpB0KF0jQq+j4bvwBLypQg6rGmD14jzLjuVm4T8uSs0JKYO3Dx2Xc9AZMo2/M+Nam9Tk5q5PBzlsiMOKRs2LbIOlLimZ7gxYjxbmkCWHgnuN/EDBf8G+SQiMkrP242NR9qeswe2zRxdU6ZCXmuxqgYgS0= root@LXUATGMAA3.GTJA.COM.UAT
```
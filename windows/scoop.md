# Scoop 
{docsify-updated}


### Scoop 安装
powershell中运行以下命令：
1. 安装到默认目录：
    ```
    Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
    # or shorter
    iwr -useb get.scoop.sh | iex
    ```
    默认安装目录是 C:\Users\<user>\scoop. 全局安装的软件 (使用 --global 安装时) 安装到： C:\ProgramData\scoop
2. 安装到自定义目录：
   ```
    $env:SCOOP='D:\applications\scoop'
    $env:SCOOP_GLOBAL='D:\applications\globalScoopApps'
    [Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
    [Environment]::SetEnvironmentVariable('SCOOP_GLOBAL', $env:SCOOP_GLOBAL, 'Machine')
    # run the installer
   ```

### 安装配置 aria2
```
scoop install aria2

scoop config aria2-enabled false
scoop config aria2-warning-enabled false
```
PowerShell requires an execution policy in [Unrestricted, RemoteSigned, ByPass] to run Scoop. For example, to set the execution policy to 'RemoteSigned' please run :  
`Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`.

### 更换国内源
1. 更换scoop自身源：`scoop config SCOOP_REPO http://mirror.repo`
2. 针对不同的bucket，进入到相应的 bucket 目录（比如$SCOOP/buckets/main），执行：`git remote set-url origin http://mirror.repo`

推荐源：https://codechina.csdn.net/mirrors 或者  https://hub.fastgit.org/。  
示例： https://github.com/Ash258/Scoop-Sysinternals.git/ ->  https://hub.fastgit.org/Ash258/Scoop-Sysinternals.git

### 常用命令
1. `scoop search program-name` : 查找软件
2. `scoop install program-name` : 安装软件
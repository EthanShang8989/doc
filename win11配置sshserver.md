# win11配置ssh server 并且内网穿透

大部分教程来说都是下面的流程

https://www.cnblogs.com/monarch-yan/p/17941313
我是参考这个文章的

要在 Windows 11 上启动 SSH 并通过内网穿透进行访问，你可以按照以下步骤操作：

### 1. 确认 SSH 服务已安装并启用

- 按 `Win + X`，选择 **Windows 终端 (管理员)**。

- 在终端中输入以下命令以检查 SSH 服务是否已经安装：

  ```
  Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
  ```

  如果结果显示 **OpenSSH.Server** 和 **OpenSSH.Client** 已安装，说明你的系统已经具备 SSH 服务。

- 如果 SSH 服务未安装，可以使用以下命令进行安装：

  ```
  Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
  ```

- 安装完成后，启动 SSH 服务：

  ```
  Start-Service sshd
  Set-Service -Name sshd -StartupType 'Automatic'
  ```



但是我powershell和cmd配置了代理都不行。所以使用

https://github.com/PowerShell/Win32-OpenSSH/releases
这个地方直接安装。



win+x

输入services.msc
找到OpenSSH SSH Server。看到是开启就行了。

可以使用

`ssh localhost`

来检测登入了。这里输入的密码是微软账号的密码不是pin的密码。



# 设置公私钥登入

## 修改配置文件

C:\ProgramData\ssh\sshd_config

然后找到并且修改

```
PasswordAuthentication no
PubkeyAuthentication yes
```

然后重启server

`Restart-Service sshd`

## 生成公私钥对

命令行输入

`ssh-keygen -t ed25519`

然后就会生成对应的公私钥对文件。

把生成的文件id_ed25519.pub复制到C:\ProgramData\ssh下面并且重新命名成administrators_authorized_keys就可以。

### 配置本地ssh客户端

打开你的用户目录下面的~/.ssh/config

添加对应的内容。

```
 Host home 

  HostName 127.0.0.1

  Port 22

  User <your_user_name>

  IdentityFile ~/.ssh/id_ed25519

```

然后

就可以命令行登入

`ssh home`

# sakura配置ssh内网穿透




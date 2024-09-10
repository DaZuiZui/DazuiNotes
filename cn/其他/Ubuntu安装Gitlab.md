### 步骤 1：更新Ubuntu系统
首先，更新你的系统软件包，以确保你有最新的安全补丁和更新：

```bash
sudo apt update
sudo apt upgrade -y
```

### 步骤 2：安装依赖项
在安装GitLab之前，你需要确保安装了以下必要的依赖项：

```bash
sudo apt install -y curl openssh-server ca-certificates tzdata perl
```

如果你计划使用邮件通知功能，还可以安装Postfix来管理邮件传递：

```bash
sudo apt install -y postfix
```

在安装Postfix时，它会提示你配置“邮件配置类型”。选择 `Internet Site`，然后设置系统邮件名称为你的域名或默认配置即可。

### 步骤 3：安装GitLab的官方包
1. 下载GitLab的安装脚本：

```bash
https://packages.gitlab.com/gitlab/gitlab-ce
```

2. 安装GitLab：

根据你的域名或IP地址安装GitLab，将 `http://your_domain_or_IP` 替换为你的实际域名或服务器IP：

```bash
sudo EXTERNAL_URL="http://your_domain_or_IP" apt install -y gitlab-ee
```

### 步骤 4：配置并启动GitLab
安装完成后，GitLab会自动启动并运行。在浏览器中访问 `http://your_domain_or_IP`，可以看到GitLab的界面。

### 步骤 5：配置防火墙（可选）
如果你使用了防火墙，需要允许HTTP和HTTPS流量。可以运行以下命令：

```bash
sudo ufw allow http
sudo ufw allow https
sudo ufw allow OpenSSH
```

### 步骤 6：初次登录
1. 打开浏览器，访问你设置的域名或IP地址。
2. GitLab会要求你设置管理员密码。输入新的密码后，你可以使用默认的用户名 `root` 和你刚设置的密码登录GitLab。

这样，你就成功在Ubuntu上安装了GitLab。
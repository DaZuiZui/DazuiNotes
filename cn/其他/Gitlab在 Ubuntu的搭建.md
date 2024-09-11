# Ubuntu install Gitlab

## 步骤1

首先，更新你的系统软件包，以确保你有最新的安全补丁和更新：

```bash
sudo apt update
sudo apt upgrade -y
```

## 步骤 2：安装依赖项
在安装GitLab之前，你需要确保安装了以下必要的依赖项：

```bash
sudo apt install -y curl openssh-server ca-certificates tzdata perl
```

如果你计划使用邮件通知功能，还可以安装Postfix来管理邮件传递：

```bash
sudo apt install -y postfix
```

在安装Postfix时，它会提示你配置“邮件配置类型”。选择 `Internet Site`，然后设置系统邮件名称为你的域名或默认配置即可。

## 步骤 3：安装GitLab的官方包
1. 下载GitLab的安装脚本：

```bash
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```

或者去https://packages.gitlab.com/gitlab/gitlab-ce

~~~java
sudo apt --fix-broken install
sudo apt-get update
sudo apt-get upgrade
sudo dpkg -i gitlab-ce_16.2.1-ce.0_amd64.deb
~~~



2. 安装GitLab：

根据你的域名或IP地址安装GitLab，将 `http://your_domain_or_IP` 替换为你的实际域名或服务器IP：

```bash
sudo EXTERNAL_URL="http://your_domain_or_IP" apt install -y gitlab-ee
```

or 修改指定ip端口

~~~JAVA
sudo vim /etc/gitlab/gitlab.rb
~~~

## 步骤 4：配置防火墙（可选）
如果你使用了防火墙，需要允许HTTP和HTTPS流量。可以运行以下命令：

```bash
sudo ufw allow http
sudo ufw allow https
sudo ufw allow OpenSSH
```

## 步骤 5：依次输入下面的操作命令启动配置

~~~jav
# 停止gitlab服务 
sudo gitlab-ctl stop ​

# 启动gitlab服务 
sudo gitlab-ctl reconfigure ​

# 重启所有gitlab组件 
sudo gitlab-ctl restart ​

# 启动所有gitlab组件 
sudo gitlab-ctl start

# 启用开机自启动
sudo systemctl enable gitlab-runsvdir.service

~~~

## 步骤 6：初始化账号密码

~~~java
#切换路径
cd /opt/gitlab/bin/ ​

#打开控制台
sudo gitlab-rails console -e production ​
 
#查询用户账号信息并赋值给u
u=User.where(id:1).first​

#设置root的密码
u.password='root123456'

#确认密码
u.password_confirmation='root123456'

#保存信息(注意有个感叹号）
u.save!

#退出
exit
~~~

or 获取临时密码

~~~java
cat /etc/gitlab/initial_root_password
~~~

## 如果发现阻止安装

~~~java
ruby_block[wait for redis service socket] action run is blocking installation gitlab-ee (11.9.6-ee.0)
~~~

输入：

~~~java
/opt/gitlab/embedded/bin/runsvdir-start &
~~~

参考文档：https://gitlab.com/gitlab-org/omnibus-gitlab/-/issues/4257
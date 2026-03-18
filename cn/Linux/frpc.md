
# 安装frpc

## 1. 下载并解压

~~~cmd
sudo mkdir -p /opt/frp
cd /tmp

FRP_VERSION="0.67.0"
ARCH=$(uname -m)
case "$ARCH" in
  x86_64|amd64) FRP_ARCH="linux_amd64" ;;
  aarch64|arm64) FRP_ARCH="linux_arm64" ;;
  *) echo "unsupported arch: $ARCH"; exit 1 ;;
esac

curl -fL -o frp.tar.gz "https://github.com/fatedier/frp/releases/download/v${FRP_VERSION}/frp_${FRP_VERSION}_${FRP_ARCH}.tar.gz"
tar -xzf frp.tar.gz
sudo mv frp_${FRP_VERSION}_${FRP_ARCH} /opt/frp/
~~~

## 2. 安装 frpc 到 PATH

~~~cmmd
sudo install -m 0755 /opt/frp/frp_${FRP_VERSION}_${FRP_ARCH}/frpc /usr/local/bin/frpc
~~~

## 3. 验证

~~~
which frpc
frpc -v
~~~

## 1. 用户端终端（Terminal A）

1. 写 frpc 配置（只要一次）：

~~~java
cat > /tmp/frp-test/frpc.toml <<'EOF'
serverAddr = "119.28.102.42"
serverPort = 54300
auth.method = "token"
auth.token = "REPLACE_WITH_TOKEN"

[[proxies]]
name = "ssh-test"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 54301
EOF

~~~

启动frpc

~~~java
cd /tmp/frp-test/frp_0.65.0_darwin_arm64
./frpc -c /tmp/frp-test/frpc.toml
~~~

## SSH方式连接

### 工程师

在工程师机终端执行：

```
ssh-keygen -t ed25519 -C "engineer@lovbrowser" 
```

一路回车即可（不设密码可完全免提示）。

然后查看公钥：

```
cat ~/.ssh/id_ed25519.pub 
```

**把输出的整行公钥复制下来。**

## 用户电脑

在用户机终端执行（把公钥粘贴进去）：

```
mkdir -p ~/.ssh
echo "<粘贴工程师公钥> lovbrowser-support-key:RS-DEMO01:1" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## 工程师

~~~jav
ssh -p 54301 <A电脑用户名>@119.28.102.42
~~~


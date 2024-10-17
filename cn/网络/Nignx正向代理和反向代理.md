# Nignx正向代理和反向代理

## 正向代理（forward proxy）

正向代理是可以访问外部资源，目标服务器不会看到客户端的真实ip，而是只能看到代理服务器的地址。比如我翻墙软件，缓存就是正向代理。

~~~java
http {
    server {
        listen 8888;  # 正向代理监听的端口

        location / {
            proxy_pass http://$http_host$request_uri;  # 转发客户端请求到目标服务器
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;  # 转发客户端的真实 IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}

~~~

## 反向代理

反向代理是一种客户端和后端服务器之间的代理服务器，客户端的请求先发给反向代理服务器，然后在转发给后端服务器。客户端不知道后端服务器的真实地址，只与反向代理服务器交换。

使用场景负载均衡	

~~~java
客户端 ---> 反向代理服务器 ---> 后端服务器群
~~~

~~~java
http {
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;  # 转发请求给 upstream 后端服务器群
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}

~~~


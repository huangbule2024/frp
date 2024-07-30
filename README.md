[官网文档](https://gofrp.org/zh-cn/docs/overview/ "官网文档")

[frp下载地址](https://github.com/fatedier/frp/releases "下载地址")
```shell
mac 下载 darwin_amd64.tar.gz 后缀的，linux amd 64 位的下载 linux_amd64.tar.gz 后缀的
```

### 服务器运行脚本
```shell
./frps -c ./frps.ini 

```
### 配置文件
```shell
[common]
bindPort = 7000 #客户端链接服务器的端口
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = 123456
vhost_http_port = 1212；#服务器对外的http端口，比如80 443

[web]
listen_port=7777  #支持直接端口访问http
```

客户端运行脚本
```shell
./frpc -c ./frpc.toml
```

客户端配置文件
```shell
serverAddr = "101.37.xx.xx" #服务器ip
serverPort = 7000

[[proxies]]
name = "test-tcp"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6000

[[proxies]]
name = "web"
type = "http"
localIP = "127.0.0.1"
customDomains = ["服务器域名"]
localPort = 1212  #本地代理端口

[[proxies]]
name = "gz_web"
type = "tcp"
localIP="127.0.0.1"
localPort = 80
remotePort=7777 #直接端口访问http

```

### 服务器nginx配置
```shell
server {
    listen 80;
    listen [::]:80;
    server_name xxx.xxxsyscloud.com;
	location /test_yl { #微信公众号配置5个域名不够用了，测试环境加个后缀充当下
        proxy_pass http://xxx.xxxsyscloud.com:1212;
    }
	
}

```

### 客户端nginx配置
```shell
server {
    listen 1212;
    listen [::]:1212;
    server_name cloud.example.com;
    root /var/www//project/public;
    index index.html index.php;
    charset utf-8;
    location /test_yl {
        alias /var/www/project/public;
        index  index.html index.htm index.php;

        try_files $uri $uri/ /index.php?$query_string;

        if (!-e $request_filename){
            rewrite  ^/test_yl/(.*)$  /test_yl/index.php?s=$1  last;
        }
       location ~ \.php(.*)$ {
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            include fastcgi_params;
        }

    }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    location ~ \.php$ {
       root /var/www//WITMED_1_2_x/public;
       fastcgi_pass  unix:/var/run/php/php7.4-fpm.sock;
       fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
       include fastcgi_params;
    }
}

```


### supervisor 配置
```shell
[program:frp]
command = /home/itc/frp/frps -c /home/itc/frp/frps.toml
autostart=true
```
```shell
[program:frp]
command = /home/itc/frp/frpc -c /home/itc/frp/frpc.toml
autostart=true
```

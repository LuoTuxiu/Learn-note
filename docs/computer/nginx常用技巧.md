# nginx常用技巧

## nginx的用途

一款轻量级的服务器，于前端而言，可以做静态文件服务器，很方便。于后端而言，可以做各种接口路由转发，负载均衡等设置。

## mac如何部署nginx

### 1. 通过homebrew安装

homebrew是mac端经常用的包管理软件，使用之前经常需要更新

```
biew update
```

注意，这里经常应该国内网络原因导致一直卡住，可以切换到vpn环境再执行

接着执行
```
brew install nginx
```

### 查看nginx 版本

nginx -v

如果安装了则会打印：

```
nginx version: nginx/1.12.2
```

### mac端nginx的路径

| 内容  | 路径 | 
| --- | --- | 
| 配置路径 |  /usr/local/nginx/conf/nginx.conf| 

### mac端重启nginx

```
sudo nginx -s reload
```

-s的含义是： -s signal     : send signal to a master process: stop, quit, reopen, reload

### 配置示例解读：

#### 配置静态前端页面服务器

```
server {
        listen       80;
        server_name  www.luotuxiu.cn; // 每个server要起唯一的名字


        #access_log  logs/host.access.log  main; // 你可以指定你的log日志存放地址、级别
				root         /home/luo/dist;
				gzip				on;
}
```

#### http重定向跳转去https的配置

```
proxy_redirect http:// https://;
```

#### nginx做转发

```
location ^~ /api/ {
		proxy_pass http://127.0.0.1:3001;
}
```

### 访问日志及错误日志

[Nginx官方文档地址](https://docs.nginx.com/nginx/admin-guide/monitoring/logging/)

在linux中，日志地址可以在配置文件中配置，格式如下：

```
access_log log_file log_format;
```

可以去默认的nginx配置中查看，可以看到：

```
#access_log  logs/access.log  main;
```

所以如果打开此项配置，则日志在在linux中位置为：/usr/local/nginx/logs/access.log


![20201204084047](http://qiniu.luotuxiu.cn/img/20201204084047.png)


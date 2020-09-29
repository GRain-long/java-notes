# Docker安装开发环境



## Docker安装Nginx

### 搜索并拉取nginx镜像

```shell
# 搜索nginx镜像
docker search nginx

# 拉取nginx镜像
docker pull nginx

# 查看所有镜像
docker images
```



### 挂载nginx目录与静态文件

**说明**：因为docker容器是互相隔离的，在Linux上启动容器后，如果要修改配置文件啥的，每次都要进入到容器内部修改会很麻烦。因此，docker有个数据卷挂载技术，将容器内部挂载到Linux上的目录，那么该挂载目录就相当于容器内部了，修改或增删文件都能直接在该目录下进行，而不用进入到容器内部了。（不懂挂载可先略过此处，跳到后面直接运行）



- 创建挂载目录

```shell
mkdir -p /usr/local/docker-nginx/{conf,conf.d,html,logs}
```



### 配置nginx

（1）可以直接从别处copy来一个配置文件

（2）可以直接启动一个临时容器，将镜像里的配置文件copy出来

```shell
# 先启动一个nginx临时容器
docker run -d --name nginx-tmp -P nginx

# 拷贝配置文件
docker cp nginx-tmp:/etc/nginx/nginx.conf /usr/local/docker-nginx/nginx.conf

# 删除临时容器
docker rm -f nginx-tmp
```

**说明：**利用docker cp 命令将容器中的conf配置文件拷贝到本机自己设置的目录下。

**另**：nginx配置文件在 /etc/nginx 目录下，页面在 /usr/share/nginx/html 目录下，日志在 /var/log/nginx 目录下。



#### nginx.conf

```shell
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

access_log  /var/log/nginx/access.log  main;

sendfile        on;
#tcp_nopush     on;

keepalive_timeout  65;

#gzip  on;

include /etc/nginx/conf.d/*.conf;

}
```



#### index.html

```shell
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



### 启动镜像

- 分别挂载：

  1.配置文件

  2.日志目录

  3.静态资源文件

```shell
# 运行容器并挂载目录和配置文件
docker run -d --name nginx \
-v /usr/local/docker-nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /usr/local/docker-nginx/logs:/var/log/nginx \
-v /usr/local/docker-nginx/html:/user/share/nginx/html \
-p 80:80 \
nginx
```

**说明**： -d 后台运行该容器

​			--name 该容器的名称（类似于别名）

​			-v 目录挂载，将容器目录（文件）挂载到本机，中间用：隔开 

​			本机目录:容器目录 本机文件:容器文件s

​			-p 端口映射     	本机端口：容器端口

​			最后是镜像名称

- 最后访问虚拟机id地址即可



**另附**：

```shell
# docker nginx镜像官网
https://hub.docker.com/_/nginx

# 关于宿主机修改文件，而容器内文件没修改原因
https://my.oschina.net/u/4437985/blog/4642766
```



## Docker安装tomcat

### 搜索并拉取tomcat镜像

```shell
# 搜索tomcat镜像
docker search tomcat

# 拉取tomcat镜像
docker pull tomcat

# 查看镜像
docker images
```





### 创建挂载目录

- tomcat 主要挂载logs、webapps目录

```shell
# 创建相应挂载目录
mkdir -p docker-tomcat/{logs,webapps,conf}

# 查看当前目录
pwd
```

  ![image-20200929163917224](images/docker安装环境/image-20200929163917224.png)

**说明**：tree命令需要格外安装： 

```shell
yum -y install tree
```



- 拷贝tomcat的相关文件

```shell
# 先启动一个临时的tomcat
docker run -d --name tomcat-tmp -P tomcat

#拷贝相关配置文件
docker cp tomcat-tmp:/usr/local/tomcat/conf/. /usr/local/docker-tomcat/conf/

#拷贝页面 
docker cp tomcat-tmp:/usr/local/tomcat/webapps.dist/. /usr/local/docker-tomcat/webapps/

# 最后删除该临时容器
docker rm -f tomcat-tmp
```

**说明**：不知为啥，dockerhub上拉取下来的tomcat，webapps文件夹是空的，所有文件都在webapps.dist里面，所以如果一开始直接访问是会报404的。因此拷贝也要从webapps.dist中进行拷贝。



### 启动镜像

```shell
# 启动镜像
docker run -d --name tomcat \
-p 8080:8080 \
-v /usr/local/docker-tomcat/conf:/usr/local/tomcat/conf \
-v /usr/local/docker-tomcat/logs:/usr/local/tomcat/logs \
-v /usr/local/docker-tomcat/webapps:/usr/local/tomcat/webapps \
--restart=always \
tomcat
```



访问 IP地址:8080 即可看到如下页面：![image-20200929171107140](images/docker安装环境/image-20200929171107140.png)

后面如果需要运行自己的war包，可以删除webapps目录下的所有文件，直接把war丢入该目录中即可，tomcat会自动解压该war包。
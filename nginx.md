# 一，基本概念

## 1.1 nginx 是什么

1. 高性能的http和反向代理服务器。占用内存少，并发性能强
2. 支持热部署

## 1.2 反向代理

1. **正向代理**：如果把局域网外的internet想象成一个巨大的资源库，局域网中的客户端要访问internet，则需要通过代理服务器来访问，这种代理就是正向代理

   一个例子：大陆无法访问google，需要借助代理。依赖代理服务器进行网络访问，这个过程叫正向代理

2. **反向代理**：客户端对代理是无感知的，客户端不需要配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，再返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器的地址，隐藏了真实服务器的ip地址

   例如：把nginx和tomcat看成一个服务器，不去请求tomcat，而是去请求nginx的地址，这样就把tomcat的真实地址隐藏了

## 1.3 负载均衡

当单个服务器无法解决时，增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的负载均衡

## 1.4 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力

session共享问题解决方案：

1. 存储到客户端cookie
2. session复制
3. nosql数据库

用nosql解决io压力

1. 做缓存数据库，减少io操作，提高访问速度

# 二，nginx安装，命令，配置文件

## 2.1 在Linux安装nginx

**一. gcc 安装**
安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：

```sql
yum install gcc-c++
```

**二. PCRE pcre-devel 安装**
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：

```cmake
yum install -y pcre pcre-devel
```

**三. zlib 安装**
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

```cmake
yum install -y zlib zlib-devel
```

**四. OpenSSL 安装**
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

```cmake
yum install -y openssl openssl-devel
```

**五. 解压**

依然是直接命令：

```css
tar -zxvf nginx-1.12.0.tar.gz
cd nginx-1.12.0
```

**六. 配置**

其实在 nginx-1.12.0 版本中你就不需要去配置相关东西，默认就可以了。当然，如果你要自己配置目录也是可以的。
1.使用默认配置

```bash
./configure
```

**七. 编译安装**

```bash
make
make install
```

**查找安装路径**：

```undefined
whereis nginx
```

**启动，停止**

```bash
cd /usr/local/nginx/sbin/
./nginx 
./nginx -s stop
./nginx -s quit
./nginx -s reload
```
**可能有防火墙**

```bash
systemctl stop firewalld
systemctl disable firewalld
```



**启动时报80端口被占用:**
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)

` 解决办法：1、安装net-tool 包：`yum install net-tools 

> `./nginx -s quit`:此方式停止步骤是待nginx进程处理任务完毕进行停止。
> `./nginx -s stop`:此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。

查询nginx进程：

```perl
ps aux|grep nginx
```

重启 nginx

1.先停止再启动（推荐）：
对 nginx 进行重启相当于先停止再启动，即先执行停止命令再执行启动命令。如下：

```bash
./nginx -s quit
./nginx
```

2.重新加载配置文件：
当 ngin x的配置文件 nginx.conf 修改后，要想让配置生效需要重启 nginx，使用`-s reload`不用先停止 ngin x再启动 nginx 即可将配置信息在 nginx 中生效，如下：
./nginx -s reload

启动成功后，在浏览器可以看到这样的页面：



## 2.2 常用命令

要使用命令，首先要进入 `/usr/local/nginx/sbin`

|       命令        |                          作用                           |
| :---------------: | :-----------------------------------------------------: |
|    ./nginx -v     |                        查看版本                         |
|  ./nginx -s quit  |    此方式停止步骤是待nginx进程处理任务完毕进行停止。    |
|  ./nginx -s stop  | 此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程 |
| ./nginx -s reload |                    配置文件重新加载                     |



## 2.3配置文件

1. nginx配置文件的位置是：/usr/local/nginx/conf/nginx.conf
2. 配置文件由三部分组成：全局块，events块，http块

```bash
/*自己写的笔记的内容用这个引用起来，剩下的都是原来的配置文件*/
#user  nobody;
worker_processes  1;

/*worker_processes的值越大，支持的并发越多*/

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

/*这个以上就是全局块*/


events {
    worker_connections  1024;
}


/*这是events块，主要影响Nginx与用户的网络连接，上述意思是worker—processes最大连接为1024*/


/*下面是http块，是nginx中配置最频繁的部分，包含http全局块，和server块*/
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
/*http全局块的指令包含文件引入，MIME-TYPE定义，日志自定义，连接超时时间，单链接请求数上限*/

    server {
    /*server中有全局server和location组成，每个http块可以包含多个server块，每个server块相当于一个虚拟主机*/
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
		/*全局server最常见的配置是监听配置和主机名称或ip配置等*/
		
        location / {
            root   html;
            index  index.html index.htm;
        }
        /*一个server可以有多个locaiton块，主要作用是基于Nginx服务器接收到的请求字符串(server_name/uri-string)，对虚拟机名称（也可以是ip别名）之外的字符串（例如前面的uri-string）进行匹配，对特定的请求进行处理。地址定向，数据缓存和应答控制等功能。还有许多第三方模块的配置也在这里进行*/

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

# 三，反向代理

## 3.1 第一个例子

1. 实现效果：打开浏览器，输入www.123.com，（不添加host的话直接访问ip）跳转到linux系统tomcat主页
2. 准备工作：
   1. 在linux安装tomcat，使用默认端口8080，启动tomcat
3. 具体操作,配置server中的location，添加proxy_pass，**记得最后加分号**！

```bash
 server {
        listen       80;
        server_name  144.24.72.146;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            proxy_pass   http://144.24.72.146:8080;
            index  index.html index.htm;
        }

```

## 3.2 第二个例子

1. 实现效果，根据不同的路径跳转到不同的端口服务中：

   访问 ip:9001/edu 跳转到 ip:8080

   访问 ip:9001/vod 跳转到 ip:8081

2. 准备工作

   1. 准备两个tomcat服务器，一个8080，一个8081
   2. 准备测试页面

3. nginx语法

   = 严格匹配。如果这个查询匹配，那么将停止搜索并立即处理此请求。

   ~ 为区分大小写匹配(可用正则表达式)

   !~为区分大小写不匹配

   ~* 为不区分大小写匹配(可用正则表达式)

   !~*为不区分大小写不匹配

   ^~ 如果把这个前缀用于一个常规字符串,那么告诉nginx 如果路径匹配那么不测试正则表达式。

4. ```bash
       server {
           listen       80;
           server_name  144.24.72.146;
   
           #charset koi8-r;
   
           #access_log  logs/host.access.log  main;
   
           location /edu/ {
               root   html;
               proxy_pass   http://144.24.72.146:8080;
               index  index.html index.htm;
           }
   
           location /vod/ {
               root   html;
               proxy_pass   http://144.24.72.146:8081;
               index  index.html index.htm;
           }
   	}
   	
   此时访问ip/edu 和 ip/vod 会跳转到两个不同的tomcat
   ```

​	

# 四，负载均衡

1. 实现效果：访问IP:edu 负载均衡效果，平均8080和8081端口中
2. 准备工作：两个tomcat服务器：8080/edu/index.html 和 8081/edu/index.html 创建index.html页面测试
3. 负载均衡配置

```bash
http{
.....
upstream myserver{
        server 144.24.72.146:8081;
        server 144.24.72.146:8080;
    }
	server{
        listen       80;
        server_name  144.24.72.146;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        location / {
           proxy_pass http://myserver/edu/;
        }

        location /edu/ {
            root   html;
            proxy_pass   http://144.24.72.146:8080;
            index  index.html index.htm;
        }

        location /vod/ {
            root   html;
            proxy_pass   http://144.24.72.146:8081;
            index  index.html index.htm;
        }
	}
}
```

**策略：**

1. 轮询（默认）

   每个请求按照时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除

2. weight

   weight代表权重，默认为1，权重越高被分配的客户端越多

   ```bash
   server 144.24.72.146:8081 weight=5;
   server 144.24.72.146:8080 weight=10;
   ```

3. ip_hash

   每个请求按照访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决**session问题**

   ```bash
   upstream server_poll{
   	ip_hash;
   	server 144.24.72.146:8081;
   	server 144.24.72.146:8080;
   }
   ```

4. fair(第三方)

   按后端服务器的响应时间来分配，响应时间短优先分配

   ```bash
   upstream server_poll{
   	server 144.24.72.146:8081;
   	server 144.24.72.146:8080;
   	fair;
   }
   ```

   

# 五，动静分离

动静分离从目前实现的角度来讲大致分为两种：

1. 纯粹把静态文件独立成单独的域名，放在独立的服务器，也是目前主流推崇的方案
2. 动态和静态文件混合在一起发布，通过nginx分开

```bash
在服务器存入文件/data/img/01.img
location /img/ {
	root /data/
	autoindex on  #配置这个可以列出文件夹下的文件
}

此时浏览器访问：ip/data/01.img 即可访问静态资源
```

# 六，nginx高可用集群

1. 什么是高可用：如果只有一台nginx，加入nginx宕机，则无法访问服务器。高可用即使用两台nginx服务器，通过keepalived进行绑定，客户端使用虚拟ip访问这两台nginx服务器。一台主服务器，一台从服务器，当主服务器发生问题时，自动将虚拟ip与从服务器进行绑定。

2. 准备工作：
   1. 两台服务器都装nginx和keepalived
   2. 安装之后，在etc里面生成目录keepalived，有文件keepalived.cof
# Nginx高级应用
**[View in Github czHappy 2019.5.2](https://github.com/czHappy/WEB-Server-Development/blob/nginx/nginx%E9%AB%98%E7%BA%A7%E5%BA%94%E7%94%A8/Nginx%E9%AB%98%E7%BA%A7%E5%BA%94%E7%94%A8.md)**
***

## 一、配置和实验反向代理
- 配置反向代理服务器的nginx.conf
```bash
#声明两个upstream 反向代理后端服务器
    upstream nginx01{
        server 118.89.236.4:8081;
    }
    upstream nginx02{
        server 118.89.236.4:8082;
    }
  
  # 代理服务器80端口设定监听和反向代理
    server {
        listen       80;
        server_name  www.cznginx01.com;
        location / {
            proxy_pass              http://nginx01;
        }
    }
    server{
        listen 80;
        server_name www.cznginx02.com;
        location / {
            proxy_pass              http://nginx02;
        }
    }
 ```
 
 - 配置后端服务器的nginx.conf
 
 ```bash
        server{
                listen 8081;
                server_name nginx8081.com;
                root html/nginx01;
                index index.html index.htm; 
        }
        server{
                listen 8082;
                server_name nginx8082.com;
                root html/nginx01;
                index index.html index.htm;
        }
```
![](./image/80port.PNG)
- 在本机（Windows）配置hosts

```
#服务器域名解析
118.89.236.4 www.cznginx01.com
118.89.236.4 www.cznginx02.com
118.89.236.4 www.cznginx.com
```

- 实验结果  
  - 报错,502

    ![](./image/502.PNG)
  - 访问8082端口失败，查看error.log日志  
  ![](./image/error_log.PNG)
  - 排除upstream的语法错误，开启防火墙8082端口的权限  
  ![](./image/防火墙开启8082端口.PNG)
  - 成功结果 


    ![nginx01](./image/nginx01.PNG)

    ![nginx02](./image/nginx02.PNG)

***

## 二、配置和实验rewrite 以及 redirect
- rewrite和redirect的区别
  - 关于重定向
    - 通过重定向，浏览器知道页面位置发生变化，从而改变地址栏显示的地址。
    - 通过重定向，搜索引擎意识到页面被移动了，从而更新搜索引擎索引，将原来失效的链接从搜索结果中移除
    - 临时重定向(R=302)和永久重定向(R=301)都是亲搜索引擎的，是SEO的重要技术。

    - Redirect是浏览器和服务器发生两次请求，也就是服务器命令客户端“去访问某个页面”；

    - redirect的URL需要传送到客户端。

    - redirect是从一个地址跳转到另一个地址。

  - 关于重写

    - rewrite的URL只是在服务器端

    - Rewrite则是服务器内部的一个接管，在服务器内部告诉“某个页面请帮我处理这个用户的请求”，浏览器和服务器只发生一次交互，浏览器不知道是该页面做的响应，浏览器只是向服务器发出一个请求。

    - URL重写用于将页面映射到本站另一页面，**若重写到另一网络主机（域名），则按重定向处理**。

    - rewrite是把一个地址重写成另一个地址。地址栏不跳转。相当于给另一个地址加了一个别名一样。

- 上述的例子就像用户去买手机，缺货时的两种处理：让用户自己去其他地方买（Redirect）；公司从其他的地方调货（Rewrite）。

- redirect和rewrite操作

```bash
 server{
        listen 8081;
        server_name 118.89.236.4;
        root html/nginx01;

#  重定向日志以notice级别输出到error.log
    rewrite_log on;
    error_log /usr/local/nginx/logs/error.log notice;

#       地址重写
        location / {
                if (!-f $request_filename){

                        rewrite ^/test1/(.*)$ http://www.baidu.com;
                }
                rewrite ^/test2/(.*)$  /test/index.html;
        }
#  302重定向 临时重定向
        location = /redirect {
                    return 302 http://www.baidu.com;
        }
#  301重定向 永久重定向
        location = /redir {
                return 301  http://118.89.236.4/test/index.html;
        }
#  alias别名
        location /newweb {
                alias /usr/local/nginx/html/nginx01/test/;
        }

            #防盗链
        location ~* \.(gif|jpg|png|swf|flv)$ {
                valid_referers none blocked 118.89.236.4;
                if ($invalid_referer) {
                        return 403;
                }
        }

    }

```
- 对nginx01进行地址重写rewrite操作 如果文件不存在，且第一个参数匹配test2，则跳到test/index.html，如果文件存在，跳到该文件。

    ![](./image/文件存在.PNG)
    - 重定向日志

    ![](./image/rewrite_log.PNG)

- 对nginx01进行地址重写redirect操作

  - 临时重定向

    ![](./image/redirect.PNG)

  - 永久重定向

    ![](./image/redirect301.PNG)

- 配置别名alias

    ![](./image/alias.PNG)

- 配置防盗链
  - 设置页面下图片

    ![](./image/访问图片.PNG)

  - 配置防盗链  
    - 非法  
    ![](./image/非法盗链.PNG)

    - 合法  
    ![](./image/合法http_refer.PNG)

*** 

## 三、配置负载均衡

```bash
//把两个upstream合而为一，设置权重
    upstream nginxproxy{
        server 118.89.236.4:8081 weight=1 ;
        server 118.89.236.4:8082 weight=2 ;
    }
    server{
        listen 80;
        server_name www.cznginx.com;
        location / {
                proxy_pass http://nginxproxy;
        }
    }
```
![](./image/负载均衡.PNG)
- 发现
>配置后每刷新三次，有两次访问到nginx02，一次nginx01，说明该权重设置并不是按概率随机的，而是通过计数器计数的方式决定下一次访问哪一个后端主机的

- 结合负载均衡算法
- 首先测定ip_hash,本机访问服务器时，被锁定在了nginx04

    ```bash
    upstream nginxproxy{
       #least_conn;
        ip_hash;
        server 118.89.236.4:8081 weight=1 ;
        server 118.89.236.4:8082 weight=20 ;
        server 118.89.236.4:8083 weight=3;
        server 118.89.236.4:8084 weight=44;
        # max_fails=3 fail_timeout=10s;
    }
    ```
- ip_hash和least_conn同用，冲突
- least_conn与weight配合使用,发现并没有什么大用，还是根据权重来，应该是客户端太少了(2台)，而tcp又是长连接，所以相当于每个虚拟主机都连了2个客户端，因此真正起作用的就是weight了。

    ```bash
    upstream nginxproxy{
        least_conn;
        server 118.89.236.4:8081 weight=1 ;
        server 118.89.236.4:8082 weight=2 ;
        server 118.89.236.4:8083 weight=3;
        server 118.89.236.4:8084 weight=4 max_fails=3 fail_timeout=10s;
    }
    ```
***
## 四、配置nginx实现动静分离
- 基本思路
  - 静态文件如jpg png html 等代理到提供静态资源的后端服务器
  - 动态文件如jsp php 等代理到提供动态资源的后端服务器
- 配置文件
```bash
    server{
        listen 80;
        server_name www.cznginx.com;
        location / {
                proxy_pass http://nginxproxy;
        }
        location ~ .*.(gif|jpg|png|bmp|swf|css|js|html|htm) {
                proxy_pass http://nginx01;
        }
        location ~ .*.(php|asp|jsp|cgi|perl) {
                proxy_pass http://nginx02;
        }
    }

```
- 安装php-fpm运行php动态文件
    - 安装 `yum install php php-mysql php-fpm`
    - 启动 `systemctl start php-fpm`
    - 在动态服务器server模块添加

    ```bash
     location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    ```
- 访问动态服务器失败，报错502Bad gateway
  - 首先排除防火墙问题，关闭防火墙仍然无效
  - 查看错误日志error.log

    ![](./image/php日志.PNG)
  - 找到错误，php-fpm.sock路径不正确，找到本机该文件路径/var/run/php-fpm/php-fpm.sock;修改重启服务即可

- 结果
  - 访问静态资源

    ![](./image/static.PNG)
  - 访问动态资源

    ![](./image/dynamic.PNG)

  - 注：这两种资源不在同一个后端服务器上，但是通过正则表达式识别url参数可以代理到相应的后端服务器进行请求。


***
## 配置keepalived，实验和观察节点掉线和上线情况。
- keepalived安装 
  - `yum -y install keepalived`

- 实验架构图  

    ![](./image/架构.PNG)

- 在两台结点都修改keepalived的配置文件，下面是备机的keepalived.conf,主机不用改

```bash
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_02 # server id 要改 唯一性
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state BACKUP # 角色要改
    interface enp0s8 #根据自己本机的网络接口，有的是eth0
    virtual_router_id 51
    priority 80 #优先级小
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1112
    }
    virtual_ipaddress {
        192.168.56.118/24 #自己设定VIP
    }
}

```

- 实验结果
  - 第一次实验，失败，主备机都存在虚拟IP，怀疑二者不能互通，应该是防火墙的问题，开启vrrp协议

    ```bash
    firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0  --protocol vrrp -j ACCEP

    firewall-cmd --reload
    ```
  - 第二次实验成功，结点1，2都开启keepalived，使用ip a查看网卡enp0s8时只能在主机MASTER看到虚拟IP 192.68.56.118,关闭主机，在备机上ip a,立刻能查看到虚拟IP 192.68.56.118 
  - 主备正常情况

    ![](./image/准备.PNG)

  - 主机MASTER宕机,备机接管

    ![](./image/MB.PNG)

***

## 五、NGINX配置调优

- 全局配置优化
  - 系统信息查看  
    ![](./image/cpu_info.PNG)


  - 最大打开文件数查看  
    ![](./image/打开文件数.PNG)
    ```bash
    worker_processes  1;#工作进程数 和主机核心数一致
    worker_rlimit_nofile 100000;#系统最大打开文件数
    ```

- event模块优化
    ```bash
    events {
    use epoll;#开启epoll网络模型高效
    worker_connections  100000;#增大每个进程最大连接数
    multi_accept on;#打开multi_accept
    }
    ```
- http模块优化
  - 安全性   
  `  server_tokens off; #在http 模块当中配置隐藏nginx版本号
`
  - 日志记录优化(减少磁盘I/O)

    ```bash
    #access_log  logs/access.log  main buffer=2k;#带缓冲的日志读写 防止写日志大量占用IO
    access_log off;#关闭访问日志
    error_log logs/error.log crit;#只记录严重错误
    ```

  - 连接模块优化

    ```bash
    #connect module
    sendfile  on; #sendfile()是立即将数据从磁盘读到OS缓存。因为这种拷贝是在内核完成的，sendfile()要比组合read()和write()以及打开关闭丢弃缓冲更加有效
    tcp_nopush  on;#tcp_nopush 告诉nginx在一个数据包里发送所有头文件，而不一个接一个的发送
    tcp_nodelay on;#告诉nginx不要缓存数据，而是一段一段的发送–当需要及时发送数据时，就应该给应用设置这个属性，这样发送一小块数据信息时就不能立即得到返回值。
    keepalive_timeout  60;#超时自动断开连接
    client_header_timeout 30;#设置请求头和请求体(各自)的超时时间。
    client_body_timeout 30;
    reset_timedout_connection on;#告诉nginx关闭不响应的客户端连接。这将会释放那个客户端所占有的内存空间
    send_timeout 10;#指定客户端的响应超时时间

    ```
  - 限制模块优化
    ```bash
    #limit module
    limit_conn_zone $binary_remote_addr zone=addr:5m; #limit_conn_zone 设置用于保存各种key（比如当前连接数）的共享内存的参数
    limit_conn addr 100;#limit_conn 为给定的key设置最大连接数。这里key是addr允许每一个IP地址最多同时打开有100个连接
    ```

  - Gzip模块优化
    ```bash
    #######
    #GZIP module
    gzip  on; #打开
    gzip_min_length 1k;#最小长度
    gzip_buffers 4 4k;#缓存
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_vary on;
    gzip_proxied expired no-cache no-store private auth;
    gzip_disable "MSIE [1-6]\.";
    ```
  - 缓存模块优化

    ```bash
    #cache module
    open_file_cache max=100000 inactive=20s; #打开缓存的同时也指定了缓存最大数目，以及缓存的时间
    open_file_cache_valid 30s; #指定检测正确信息的间隔时间。
    open_file_cache_min_uses 2; #定义了open_file_cache中指令参数不活动时间期间里最小的文件数。
    open_file_cache_errors on;#指定了当搜索一个文件时是否缓存错误信息
    ```

- 优化前后对比
  - 压测工具Apache24 ab工具
  - 优化前单用户1000次访问

    ![](./image/优化之前1000.PNG)

  - 优化后单用户1000次访问

    ![](./image/优化后.PNG)

  - 优化前并发用户数100，单个访问次数1000

    ![](./image/优化之前带并发用户数.PNG)

  - 优化后并发用户数100，单个访问次数1000

    ![](./image/优化之后带并发用户数.PNG)

- 优化前后数据分析
  - 单用户共1000次访问，优化后
    - nginx版本号隐藏，避免access.log文件过于庞大占用硬盘存储空间
    - 连接失败率20%不变
    - 每秒处理请求(吞吐率)从11.58提高到15.22
    - 速度传输率从9.79KB/S提高到12.77KB/S

  - 100并发度共1000次访问，优化后
    - nginx版本号隐藏，避免access.log文件过于庞大占用硬盘存储空间
    - 连接失败率从79.8%降低至20.7%
    - 每秒处理请求(吞吐率)从17.12提高到23.05
    - 速度传输率从14.48KB/S提高到19.34KB/S 

***

## 实验参考
- [安装nginx php](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-centos-7)
- [centos7防火墙操作](https://www.tecmint.com/install-configure-firewalld-in-centos-ubuntu/)
- [keepalived 操作](https://klionsec.github.io/2017/12/23/keepalived-nginx/)
- [rewrite和redirect](https://weblogs.asp.net/owscott/rewrite-vs-redirect-what-s-the-difference)
- [yum 使用](https://www.tecmint.com/20-linux-yum-yellowdog-updater-modified-commands-for-package-mangement/)
- [centos7配置网卡](https://www.serverlab.ca/tutorials/linux/administration-linux/how-to-configure-centos-7-network-settings/)
- [ab 命令](https://www.cnblogs.com/yueminghai/p/6412254.html)
- [nginx 优化配置](https://www.linuxprobe.com/nginx-optimization.html)
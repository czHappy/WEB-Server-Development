# Nginx 入门实验
- [***Read it on Github***](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx.md)
## 安装Nginx依赖库
- 依赖库
    - Nginx中gzip模块需要zip库 rewrite模块需要pcre库 ssI功能需要openssl库
- 安装PCRE库
```bash
$ cd /usr/local/ #切换到当前目录下

 #wget方式下载Nginx安装包
$ wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.36.tar.gz  

#解压命令
$ tar -zxvf pcre-8.36.tar.gz  

$ cd pcre-8.36
#./configure是源代码安装的第一步，主要的作用是对即将安装的软件进行配置，检查当前的环境是否满足要安装软件的依赖关系
$ ./configure

#make是用来编译的，它从Makefile中读取指令，然后编译
$ make 

#make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置
$ make install
```

- 安装zlib库
```bash
$ cd /usr/local/ 
$ wget http://zlib.net/zlib-1.2.8.tar.gz 
$ tar -zxvf zlib-1.2.8.tar.gz 
$ cd zlib-1.2.8 
$ ./configure
$ make 
$ make install
```

- 安装ssl(过程一样，不再赘述)

## 安装Nginx
- 安装Nginx
```bash
$ cd /usr/local/
$ wget http://nginx.org/download/nginx-1.8.0.tar.gz 
$ tar -zxvf nginx-1.8.0.tar.gz 
$ cd nginx-1.8.0 
#prefix指定Nginx资源资源文件的路径(安装路径)
#如果不配置 --prefix 选项，安装后：可执行文件默认放在/usr /local/bin，库文件默认放在/usr/local/lib，配置文件默认放在/usr/local/etc，其它的资源文件放在/usr /local/share，会很分散凌乱
$ ./configure --prefix=/usr/local/nginx  
--with-pcre=/usr/local/pcre-8.36 
--with-zlib=/usr/local/zlib-1.2.8 指的是zlib-1.2.8 的源码路径。
$ make 
$ make install
```
- 启动Nginx
```bash
$ /usr/local/nginx/sbin/nginx
```
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/查看Nginx进程.PNG?raw=true)

- 新增Nginx环境变量
```bash
#为了方便对Nginx命令的调用
vim /etc/profile
export PATH="$PATH:/usr/local/nginx/sbin
```
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/添加环境变量.PNG?raw=true)

-  观察nginx进程状态 top命令
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/top.PNG?raw=true)

- 查看公网IP地址 http://118.89.236.4，浏览器输入，得到以下画面，启动成功
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/网页.PNG?raw=true)



## Nginx管理服务
- 编写控制Nginx自启的service
```bash
# 在/usr/lib/systemd/system路径下编写nginx.ervice
[unit]
Description=nginx
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx reload
ExecStop=/usr/local/nginx/sbin/nginx quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
- nginx.service生效
```bash

#生成符号链接 实现开机自启动
systemctl enable nginx.service

#启动nginx.service服务
systemctl start nginx.service

#nginx.service使得原生的nginx启动 重启 关闭操作转换成了由systemd控制的常规服务操作 更加方便
```
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/nginx_service.PNG?raw=true)

## nginx配置文件
- 内容解读
```bash
#user  nobody;
#工作核数
worker_processes  1;
#错误日志
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#进程pid
#pid        logs/nginx.pid;
#最大连接数
events {
    worker_connections  1024;
}

#全局块 配置影响nginx全局的指令
http {
    include       mime.types;
    default_type  application/octet-stream;
    #日志格式说明
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    #keepalive_timeout  0;
    keepalive_timeout  65;
    #gzip  on;
    #server块  配置影响nginx服务器或与用户的网络连接
    server {
        listen       80;#监听端口 默认80
        server_name  localhost;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        
        #根目录路径 index识别文件是 index.html index.htm
        location / {
            root   html;
            index  index.html index.htm;
        }
        #错误页面
        #error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    #其他PHP 自定义虚拟主机等注释不再赘述 与上面格式相同
}
```
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/配置文件.PNG?raw=true)
- 重要字段
    - 网页根目录路径
    ```bash
    #通过配置index.html，增添内容或者重命名可以自定义主页
     location / {
                root   html;
                index  index.html index.htm;
            }
    ```
    ![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/修改配置主页内容.PNG?raw=true)
    - 监听端口 80
    - 错误日志记录在 error.log
    


## nginx日志管理
- access.log
``` bash
#日志格式
log_format access '$remote_addr - $remote_user [$time_local] "$request" "$request_time" $status $body_bytes_sent "$http_referer" "$http_user_agent" $http_x_forwarded_for';

#查询到本机IP地址60.247.41.106 与本机访问公网主页的access日志相呼应
$remote_addr：与 $http_x_forwarded_for 用以记录客户端的ip地址；
$remote_user：用来记录客户端用户名称；
$time_local：用来记录访问时间与时区；
$request：用来记录请求的http的方式与url；
$request_time：用来记录请求时间；
$status：用来记录请求状态；成功是200，
$body_bytes_sent：记录发送给客户端文件主体内容大小；
$http_referer：用来记录从那个页面链接访问过来的；
$http_user_agent：记录客户端浏览器的相关信息。
```
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/本机IP地址.PNG?raw=true)
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/access_log.PNG?raw=true)


- error.log

![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/error_log.PNG?raw=true)

- 日志统计命令
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/日志统计操作1.PNG?raw=true)
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/日志统计操作2.PNG?raw=true)

- 日志按天切割与压缩
    - 新建脚本daily_nginx_log.sh 存放在/opt/目录下
```bash
#!/bin/bash

#自动分割nginx的日志，包括access.log和error.log
#每天00:00执行此脚本 将前一天的access.log重命名为access-xxxx-xx-xx.log格式，并重新打开日志文件
#Nginx日志文件所在目录 这里后面一定要跟一个斜线 否则导致拼接错误
LOG_PATH=/usr/local/nginx/logs/
#获取昨天的日期
YESTERDAY=$(date -d "yesterday" +%Y-%m-%d)
#获取pid文件路径
PID=${LOG_PATH}nginx.pid
#分割日志
mv ${LOG_PATH}access.log ${LOG_PATH}access-${YESTERDAY}.log
mv ${LOG_PATH}error.log ${LOG_PATH}error-${YESTERDAY}.log
#压缩命令
gzip ${LOG_PATH}access-${YESTERDAY}.log
gzip ${LOG_PATH}error-${YESTERDAY}.log

#向Nginx主进程发送USR1信号，重新打开日志文件
kill -USR1 `cat ${PID}`
```

* 定时任务设置 crontab
```bash
#编辑定时任务
crontab -e

#写入定时任务 格式 minute hour * * * bash_command shell_script
00 00 * * * /bin/bash /opt/daily_nginx_log.sh
```
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/crontab.PNG?raw=true)
- 运行结果（为了方便改一下定时时间）
```bash
#原理
#每次到指定时间后 daily_nginx_log.sh被启动，将上一天的日志mv到新建日志文件，该日志文件自动重命名为kind-yyyy-mm-dd.log
#mv操作事实上是一个剪切操作，这样的话能够保持原生的access.log或error.log被删去，而系统又会自动创建这两个文件，
#所以执行完脚本后这两个文件的内容是空的
#上一天的日志内容被转存到以天分割的压缩日志文件中
```
![](https://github.com/czHappy/WEB-Server-Development/blob/nginx/exp%20nginx/nginx/新增压缩功能.PNG?raw=true)

## 基于awk的脚本对日志文件进行分析统计
- 编写脚本 logStatsProg.sh 实现对web日志web_log.tsv 进行分析统计
```bash
#!/bin/bash

function usage
{
echo "Useage: logStatsProg.sh [-tsh <num>] [-tsi <num>] [-turl <num>] [-sc] [-t4xx <num>] [-urls <url> <n>]"
echo "command	args		description"
echo "-tsh	<number>	统计来源主机top x 和分别对应的总次数"
echo "-tsi	<number>	统计来源主机top x IP 和分别对应的总次数"
echo "-turl	<number>	统计最频繁被访问的URL top x"
echo "-sc			统计不同相应状态码的出现次数和对应百分比"
echo "-t4xx	<number>	统计不同4XX状态码对应的top x URL和对应出现总次数"
echo "-urls	<url><n>	给定URL输出top x访问来源主机"
echo ""

}
#查询top x 主机
function top_source_host
{
#echo "number = $1"
#sort是默认从小到大的所以需要逆序
(sed -e '1d' web_log.tsv | awk -F '\t' '{a[$1]+=1} END {for(i in a) {print a[i],i}}' | sort -n -r | head -n $1)
}
#查询top x IP
function top_source_ip
{
 (sed -e '1d' web_log.tsv|awk -F '\t' '{if($1~/^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$/) print $1}'|awk '{a[$1]++} END {for(i in a){print i,a[i]}}'|sort -nr -k2|head -n $1)
}
#查询top x URL
function top_url
{
awk -F '\t' '{a[$5]+=1} END {for(i in a) {print i}}' web_log.tsv | sort -n -r | head -n $1
}
#查询状态码
function status_code
{
awk -F '\t'  'NR>1{a[$6]++} END {for(i in a) {ratio=100*a[i]/(NR-1); printf("%s, %.8f%, %s\n", i, ratio, a[i])}}' web_log.tsv 
}
#查询4xx状态码
function t4xx_sc
{
    #先把4xx状态码全部求出来
    a=$(sed -e '1d' web_log.tsv |awk -F '\t' '{if($6~/^4+/) a[$6]++} END {for(i in a) print i}')
    #对于每一个4xx，求其top x
    for i in $a
    do
        (sed -e '1d' web_log.tsv |awk -F '\t' '{if($6~/^'$i'/) a[$6][$5]++} END {for(i in a){for(j in a[i]){print i,j,a[i][j]}}}'|sort -nr -k3|head -n $1)
    done
}

function url_top_host
{
#awk -F '\t' '{if($5~/'$1'/) a[$5]+=1} END {for(i in a) {print i}}' web_log.tsv | sort -n -r | head -n $2
awk -F '\t' '{if($5=="'$1'") a[$1]++} END {for(i in a){print i,a[i]}}' web_log.tsv |sort -nr -k2|head -n  $2

}


if [ $# == 0 ];then
	usage
#	echo "无参数，跳出"
	exit
fi
#echo "here"
while [ "$1" != "" ]; do
 #   echo "incycle"
    case $1 in
	-tsh )	iftsh=1
		shift
		if [[ $1 != -* && $1 ]];then
			tshn=$1
		else
			echo "There need a parameter after -tsh"
			exit
		fi
		;;

	-tsi ) iftsi=1
		shift
		if [[ $1 != -* && $1 ]];then
			tsin=$1
		else
			echo "There need a parameter after -tsi"
			exit
		fi
		;;

	-turl ) ifturl=1
		shift
		if [[ $1 != -* && $1 ]];then
			turln=$1
		else
			echo "There need a parameter after -turl"
			exit
		fi
		;;

	-sc ) ifsc=1
		;;

	-t4xx ) ift4xx=1
		shift
		if [[ $1 != -* && $1 ]];then
			t4xxn=$1
		else
			echo "There need a parameter after -t4xx"
			exit
		fi
		;;

	-urls ) ifurls=1
		shift
		if [[ $1 != -* && $1 ]];then
			url=$1
		else
			echo "There need two parameter after -t4xx"
			exit
		fi
		shift
		if [[ $1 != -* && $1 ]];then
			urln=$1
		else
			echo "There need two parameter after -t4xx"
			exit
		fi

		;;

	-h ) usage
		exit
		;;
	* ) usage
		
		exit 
		;;
     esac
     shift
done

if [[ $iftsh && $tshn ]]; then
	top_source_host $tshn
fi
if [[ $iftsi && $tsin ]]; then
	top_source_ip $tsin
fi
if [[ $ifturl && $turln ]]; then
	top_url $turln
fi
if [[ $ifsc ]] ;then
	status_code 
fi
if [[ $ift4xx && $t4xxn ]]; then
	t4xx_sc $t4xxn
fi
if [[ $ifurls && $url && $urln ]] ;then
	url_top_host $url $urln
fi	

```

- 统计结果
	- 统计最频繁访问的top 100 主机
    
    ![](https://github.com/CUCCS/linux-2019-czHappy/blob/exp04/exp04/results/top100_host.PNG?raw=true)
	
	- 统计4xx状态码信息
    
    ![](https://github.com/CUCCS/linux-2019-czHappy/blob/exp04/exp04/results/4xx.PNG?raw=true)

	- 统计最最频繁访问的top100 IP
    
    ![](https://github.com/CUCCS/linux-2019-czHappy/blob/exp04/exp04/results/top100_ip.PNG?raw=true)


## 实验心得
- 通过学习nginx基本命令了解其功能，阅读相关配置文件了解其基本流程和原理
- 自己编写service实现对nginx的自动控制
- 自己编写脚本实现对日志文件的分析统计
- 除了收获了课堂知识之外，查阅了大量课外资料实现进阶的操作

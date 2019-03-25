# 实验二 用TELNET观察HTTP协议

## 安装telnet
- **安装telnet工具**
``` bash
yum list |grep telnet   //列出当前可用的rpm包
yum install telnet-server　　//安装telnet-server 服务端
yum install telnet　　　　//安装telnet 客户端
#安装xinetd
yum install -y xinetd systemctl enable xinetd.service  //设置xinetd开机自启动
```
![](https://github.com/czHappy/WEB-Server-Development/blob/http/exp%20telnet/image/%E5%AE%89%E8%A3%85telnet.PNG?raw=true)
- **开启xinetd服务**
``` bash
systemctl enable xinetd.service 
systemctl status xinetd.service 
systemctl start xinetd.service
```

- **测试telnet是否可用**
```bash
 telnet localhost
 #发现错误
#Trying 127.0.0.1...
#telnet: connect to address 127.0.0.1: Connection refused
#需要开启23端口
netstat -tunlp #查看当前网络状态下的可用连接
firewall-cmd --query-port=23/tcp#查看端口是否开启
firewall-cmd --zone=public --add-port=23/tcp --permanent#开启23端口
```
![](https://github.com/czHappy/WEB-Server-Development/blob/http/exp%20telnet/image/%E6%9F%A5%E7%9C%8B23%E7%AB%AF%E5%8F%A3%E6%98%AF%E5%90%A6%E5%BC%80%E5%90%AF.PNG?raw=true)

- **修改配置文件 /etc/xinetd.d/telnet**
```bush
#default:yes  
# description: The telnet server servestelnet sessions; it uses \  
# unencrypted username/password pairs for authentication.  
service telnet
{        
    flags = REUSE            
    socket_type  = stream           
    wait = no            
    user = root                       
    server =/usr/sbin/in.telnetd                       
    log_on_failure  += USERID                
    disable = no   
}
``` 
![图片](https://github.com/czHappy/WEB-Server-Development/blob/http/exp%20telnet/image/%E4%BF%AE%E6%94%B9xinetd%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%BD%BFTelnet%E5%8A%9F%E8%83%BD%E7%94%9F%E6%95%88.PNG?raw=true)

- **查看xinetd启动成功**
```bash
ps -ef | grep xinetd 
```
![](https://github.com/czHappy/WEB-Server-Development/blob/http/exp%20telnet/image/%E7%94%A8fire-cmd%E5%BC%80%E5%90%AF23%E5%8F%B7%E7%AB%AF%E5%8F%A3.PNG?raw=true)

---


## 使用telnet向服务器发出请求

- **请求百度页面**
```bash
telnet www.baidu.com 80#一定要跟80 虽然是默认但一定要加 否则没有结果
```
![](https://github.com/czHappy/WEB-Server-Development/blob/http/exp%20telnet/image/%E8%AF%B7%E6%B1%82%E7%99%BE%E5%BA%A6%E9%A1%B5%E9%9D%A2.PNG?raw=true)



---

## 解析HTTP字段作用

>- Accept-Ranges
>告诉WEB服务器自己接受什么介质类型，*/* 表示任何类型，type/* 表示该类型下的所有子类型，type/sub-type。
>
>- Accept-Charset： 浏览器申明自己接收的字符集
>- Cache-Control：请求：no-cache（不要缓存的实体，要求现在从WEB服务器去取）
>- Connection: 针对该连接所预期的选项Connection: close
>- Content-Length：    WEB 服务器告诉浏览器自己响应的对象的长度。
>- Content-Type：      WEB 服务器告诉浏览器自己响应的对象的类型。
>
>- Date : 此条消息被发送时的日期和时间(以RFC 7231中定义的"HTTP日期"格式来表示)
>
>- ETag:  对于某个资源的某个特定版本的一个标识符，通常是一个消息散列
>- Accept-Encoding： 浏览器申明自己接收的编码方法，通常指定压缩方法，是否支持压缩，支持什么压缩方法 （gzip，deflate）
>
>- Last-Modified所请求的对象的最后修改日期(按照 RFC 7231 中定义的“超文本传输协议日期”格式来表示)
>
>- P3P :  P3P策略相关设置
>- Pragma与具体的实现相关，这些响应头可能在请求/回应链中的不同时候产生不同的效果
>- Server服务器的名称  
>- Set-Cookie： 设置HTTP cookie, 包括公司名，时间，有效时长，域名等等
>- Vary： 告知下游的代理服务器，应当如何对以后的请求协议头进行匹配，以决定是否可使用已缓存的响应内容而不是重新从原服务器请求新的内容。
>-X-Ua-Compatible: 强制浏览器的渲染方式，默认使用chrome来渲染，然后再按照IE该浏览器的最新版本来渲染


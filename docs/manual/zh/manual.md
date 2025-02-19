## 首次使用必看！
  
### 1. 环境

该手册教程，默认系统是linux，程序是proxy；所有操作需要root权限；  

如果你的是windows，请使用windows版本的proxy.exe即可。  
  
### 2. 使用配置文件

接下来的教程都是通过命令行参数介绍使用方法，也可以通过读取配置文件获取参数。

具体格式是通过@符号指定配置文件，例如:./proxy @configfile.txt  

configfile.txt里面的格式是，第一行是子命令名称，第二行开始一行一个参数，  

格式:`参数 参数值`，没有参数值的直接写参数，比如:--nolog  

比如configfile.txt内容如下:

```shell
http
-t tcp
-p :33080
--forever
```

### 3. 调试输出

默认情况下，日志输出的信息不包含文件行数，某些情况下为了排除程序问题，快速定位问题，  

可以使用--debug参数，输出代码行数和毫秒时间。  

### 4. 使用日志文件

默认情况下，日志是直接在控制台显示出来的，如果要保存到文件，可以使用--log参数，  

比如: --log proxy.log，日志就会输出到proxy.log方便排除问题。  


### 5. 生成加密通讯需要的证书文件 

http，tcp，udp代理过程会和上级通讯，为了安全我们采用加密通讯，当然可以选择不加密通信通讯，本教程所有和上级通讯都采用加密，需要证书文件。
  
***所有端必须使用相同的proxy.crt和proxy.key***

1.通过下面的命令生成自签名的证书和key文件。  
`./proxy keygen -C proxy`  
会在当前程序目录下面生成证书文件proxy.crt和key文件proxy.key。   

2.通过下面的命令生，使用自签名证书proxy.crt和key文件proxy.key签发新证书:goproxy.crt和goproxy.key。  
`./proxy keygen -s -C proxy -c goproxy`  
会在当前程序目录下面生成证书文件goproxy.crt和key文件goproxy.key。   

3.默认情况下证书的里面的域名是随机的，可以使用`-n test.com`参数指定。  

4.更多用法:`proxy keygen --help`。    

### 6. 后台运行

默认执行proxy之后，如果要保持proxy运行，不能关闭命令行。  

如果想在后台运行proxy，命令行可以关闭，只需要在命令最后加上--daemon参数即可。  

比如:  

`./proxy http -t tcp -p "0.0.0.0:38080" --daemon`   

### 7. 守护运行
守护运行参数--forever，比如: `proxy http --forever` ， 

proxy会fork子进程，然后监控子进程，如果子进程异常退出，5秒后重启子进程。  

该参数配合后台运行参数--daemon和日志参数--log，可以保障proxy一直在后台执行不会因为意外退出，  

而且可以通过日志文件看到proxy的输出日志内容。  

比如: `proxy http -p ":9090" --forever --log proxy.log --daemon`  

### 8. 安全建议

当VPS在nat设备后面，vps上网卡IP都是内网IP，这个时候可以通过-g参数添加vps的外网ip防止死循环。  

假设你的vps外网ip是23.23.23.23，下面命令通过-g参数设置23.23.23.23  

`./proxy http -g "23.23.23.23"`  

### 9. 负载均衡和高可用

HTTP(S)\SOCKS5\SPS代理支持上级负载均衡和高可用，多个上级重复-P参数即可。  

负载均衡策略支持5种，可以通过`--lb-method`参数指定:

roundrobin 轮流使用

leastconn  使用最小连接数的

leasttime  使用连接时间最小的

hash     使用根据客户端地址计算出一个固定上级

weight    根据每个上级的权重和连接数情况，选择出一个上级

提示:

负载均衡检查时间间隔可以通过`--lb-retrytime`设置，单位毫秒

负载均衡连接超时时间可以通过`--lb-timeout`设置，单位毫秒

如果负载均衡策略是权重(weight)，-P格式为:2.2.2.2:3880?w=1，1就是权重，大于0的整数。  

如果负载均衡策略是hash，默认是根据客户端地址选择上级，可以通过开关`--lb-hashtarget`使用访问的目标地址选择上级。  


### 10. 代理跳板跳转

http(s)代理，SPS代理，内网穿透，tcp代理都支持通过中间第三方代理连接上级，  

参数是:--jumper，所有格式如下:  

```text
 http://username:password@host:port  
 http://host:port  
 https://username:password@host:port  
 https://host:port  
 socks5://username:password@host:port  
 socks5://host:port  
 socks5s://username:password@host:port  
 socks5s://host:port  
 ss://method:password@host:port
```

http，socks5代表的是普通的http和socks5代理。 

https，socks5s代表的是通过tls保护的http和socks5代理，  

也就是http代理 over TLS ， socks over TLS。  

### 11. 域名黑白名单

socks/http(s)/sps代理都支持域名黑白名单。 

用--stop参数指定一个域名黑名单列表文件，那么当用户连接文件里面这些域名的时候连接就会被断开。    

用--only参数指定一个域名白名单列表文件，那么当用户连接文件里面这些域名之外的域名的时候连接就会被断开。 

如果同时设置了--stop和--only，那么只有--only会起作用。  

黑白域名名单文件内容格式如下:

```text
**.baidu.com
*.taobao.com
a.com
192.168.1.1
192.168.*.*
?.qq.com
```

说明:  

1.一行一个域名，域名写法支持通配符`*`和`?`，`*`代表任意个字符，`?`代表一个任意字符，

2.`**.baidu.com` 匹配无论是多少级所有后缀是`.baidu.com`的域名。  

3.`*.taobao.com` 匹配后缀是`.taobao.com`的三级域名。  

4.还可以直接是IP地址。  

5.`#`开头的为注释。  

## 1.HTTP代理

### 1.1.普通一级HTTP代理

![1.1](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/http-1.png)  

`./proxy http -t tcp -p "0.0.0.0:38080"`  
  
### 1.2.普通二级HTTP代理

![1.2](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/http-2.png) 

使用本地端口8090，假设上级HTTP代理是`22.22.22.22:8080`  

`./proxy http -t tcp -p "0.0.0.0:8090" -T tcp -P "22.22.22.22:8080" `  

我们还可以指定网站域名的黑白名单文件，一行一个域名，匹配规则是最右匹配，比如:baidu.com，匹配的是*.*.baidu.com，黑名单的域名直接走上级代理，白名单的域名不走上级代理。  

`./proxy http -p "0.0.0.0:8090" -T tcp -P "22.22.22.22:8080"  -b blocked.txt -d direct.txt`  
  
### 1.3.HTTP二级代理(加密)

> 注意: 后面二级代理使用的`proxy.crt`和`proxy.key`应与一级代理一致  

![1.3](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/http-tls-2.png)  
一级HTTP代理(VPS，IP:22.22.22.22)  
`./proxy http -t tls -p ":38080" -C proxy.crt -K proxy.key`  
  
二级HTTP代理(本地Linux)  
`./proxy http -t tcp -p ":8080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  
那么访问本地的8080端口就是访问VPS上面的代理端口38080。  
  
二级HTTP代理(本地windows)  
`./proxy.exe http -t tcp -p ":8080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  
然后设置你的windos系统中，需要通过代理上网的程序的代理为http模式，地址为：127.0.0.1，端口为：8080，程序即可通过加密通道通过vps上网。  
  
### 1.4.HTTP三级代理(加密)
![1.3](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/http-tls-3.png)  
一级HTTP代理VPS_01，IP:22.22.22.22  
`./proxy http -t tls -p ":38080" -C proxy.crt -K proxy.key`  
二级HTTP代理VPS_02，IP:33.33.33.33  
`./proxy http -t tls -p ":28080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  
三级HTTP代理(本地)  
`./proxy http -t tcp -p ":8080" -T tls -P "33.33.33.33:28080" -C proxy.crt -K proxy.key`  
那么访问本地的8080端口就是访问一级HTTP代理上面的代理端口38080。  
  
### 1.5.Basic认证
对于代理HTTP协议我们可以basic进行Basic认证，认证的用户名和密码可以在命令行指定  
`./proxy http -t tcp -p ":33080" -a "user1:pass1" -a "user2:pass2"`  
多个用户，重复-a参数即可。  
也可以放在文件中，格式是一行一个"用户名:密码"，然后用-F指定。  
`./proxy http -t tcp -p ":33080" -F auth-file.txt`   
  
另外，http(s)代理还集成了外部HTTP API认证，我们可以通过--auth-url参数指定一个http url接口地址，  
然后有用户连接的时候，proxy会GET方式请求这url，带上下面四个参数，如果返回HTTP状态码204，代表认证成功  
其它情况认为认证失败。  
比如:  
`./proxy http -t tcp -p ":33080" --auth-url "http://test.com/auth.php"`  
用户连接的时候，proxy会GET方式请求这url("http://test.com/auth.php")，  
带上user，pass，ip，local_ip，target五个参数:  
http://test.com/auth.php?user={USER}&pass={PASS}&ip={IP}&local_ip={LOCAL_IP}&target={TARGET}  
user:用户名  
pass:密码  
ip:用户的IP，比如:192.168.1.200  
local_ip:用户访问的服务器的IP，比如:3.3.3.3  
target:用户访问的URL，比如:http://demo.com:80/1.html或https://www.baidu.com:80  

如果没有-a或-F或--auth-url参数，就是关闭Basic认证。  

### 1.6.HTTP代理流量强制走上级HTTP代理
默认情况下，proxy会智能判断一个网站域名是否无法访问，如果无法访问才走上级HTTP代理.通过--always可以使全部HTTP代理流量强制走上级HTTP代理。  
`./proxy http --always -t tls -p ":28080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  

### 1.7.HTTP(S)通过SSH中转
![1.7](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/http-ssh-1.png)  
说明:ssh中转的原理是利用了ssh的转发功能，就是你连接上ssh之后，可以通过ssh代理访问目标地址。  
假设有:vps  
- IP是2.2.2.2， ssh端口是22， ssh用户名是:user， ssh用户密码是:demo  
- 用户user的ssh私钥名称是user.key    

#### *1.7.1 ssh用户名和密码的方式* 
本地HTTP(S)代理28080端口，执行:  
`./proxy http -T ssh -P "2.2.2.2:22" -u user -D demo -t tcp -p ":28080"`  
#### *1.7.2 ssh用户名和密钥的方式* 
本地HTTP(S)代理28080端口，执行:  
`./proxy http -T ssh -P "2.2.2.2:22" -u user -S user.key -t tcp -p ":28080"`  

### 1.8.KCP协议传输
![1.8](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/http-kcp.png)  
KCP协议需要--kcp-key参数设置一个密码用于加密解密数据  

一级HTTP代理(VPS，IP:22.22.22.22)  
`./proxy http -t kcp -p ":38080" --kcp-key mypassword`  
  
二级HTTP代理(本地Linux)  
`./proxy http -t tcp -p ":8080" -T kcp -P "22.22.22.22:38080" --kcp-key mypassword`  
那么访问本地的8080端口就是访问VPS上面的代理端口38080，数据通过kcp协议传输，注意kcp走的是udp协议协议，所以防火墙需放开38080的udp协议。  

### 1.9 HTTP(S)反向代理 
![1.9](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/fxdl.png)  
proxy不仅支持在其他软件里面通过设置代理的方式，为其他软件提供代理服务，而且支持直接把请求的网站域名解析到proxy监听的ip上，然后proxy监听80和443端口，那么proxy就会自动为你代理访问需要访问的HTTP(S)网站。  

使用方式:  
在"最后一级proxy代理"的机器上，因为proxy要伪装成所有网站，网站默认的端口HTTP是80，HTTPS是443，让proxy监听80和443端口即可.参数-p多个地址用逗号分割。  
`./proxy http -t tcp -p :80，:443`    

这个命令就在机器上启动了一个proxy代理，同时监听80和443端口，既可以当作普通的代理使用，也可以直接把需要代理的域名解析到这个机器的IP上。  

如果有上级代理那么参照上面教程设置上级即可，使用方式完全一样。  
`./proxy http -t tcp -p :80，:443 -T tls -P "2.2.2.2:33080" -C proxy.crt -K proxy.key`   

注意:  
proxy所在的服务器的DNS解析结果不能受到自定义的解析影响，不然就死循环了，proxy代理最好指定`--dns 8.8.8.8`参数。  
  
### 1.10 HTTP(S)透明代理
该模式需要具有一定的网络基础，相关概念不懂的请自行搜索解决。  
假设proxy现在在路由器上运行，启动命令如下:  
`./proxy http -t tcp -p :33080 -T tls -P "2.2.2.2:33090" -C proxy.crt -K proxy.key`   

然后添加iptables规则，下面是参考规则:  
```shell
#上级proxy服务端服务器IP地址:
proxy_server_ip=2.2.2.2

#路由器运行proxy监听的端口:
proxy_local_port=33080

#下面的就不用修改了
#create a new chain named PROXY
iptables -t nat -N PROXY

# Ignore your PROXY server's addresses
# It's very IMPORTANT， just be careful。  

iptables -t nat -A PROXY -d $proxy_server_ip -j RETURN

# Ignore LANs IP address
iptables -t nat -A PROXY -d 0.0.0.0/8 -j RETURN
iptables -t nat -A PROXY -d 10.0.0.0/8 -j RETURN
iptables -t nat -A PROXY -d 127.0.0.0/8 -j RETURN
iptables -t nat -A PROXY -d 169.254.0.0/16 -j RETURN
iptables -t nat -A PROXY -d 172.16.0.0/12 -j RETURN
iptables -t nat -A PROXY -d 192.168.0.0/16 -j RETURN
iptables -t nat -A PROXY -d 224.0.0.0/4 -j RETURN
iptables -t nat -A PROXY -d 240.0.0.0/4 -j RETURN

# Anything to port 80 443 should be redirected to PROXY's local port
iptables -t nat -A PROXY -p tcp --dport 80 -j REDIRECT --to-ports $proxy_local_port
iptables -t nat -A PROXY -p tcp --dport 443 -j REDIRECT --to-ports $proxy_local_port

# Apply the rules to nat client
iptables -t nat -A PREROUTING -p tcp -j PROXY
# Apply the rules to localhost
iptables -t nat -A OUTPUT -p tcp -j PROXY
```
- 清空整个链 iptables -F 链名比如iptables -t nat -F PROXY
- 删除指定的用户自定义链 iptables -X 链名 比如 iptables -t nat -X PROXY
- 从所选链中删除规则 iptables -D 链名 规则详情 比如 iptables -t nat -D PROXY -d 223.223.192.0/255.255.240.0 -j RETURN

### 1.11 自定义DNS
--dns-address和--dns-ttl参数，用于自己指定proxy访问域名的时候使用的dns（--dns-address）  
以及解析结果缓存时间（--dns-ttl）秒数，避免系统dns对proxy的干扰，另外缓存功能还能减少dns解析时间提高访问速度。  
比如：  
`./proxy http -p ":33080" --dns-address "8.8.8.8:53" --dns-ttl 300`  

### 1.12 自定义加密
proxy的http(s)代理在tcp之上可以通过tls标准加密以及kcp协议加密tcp数据，除此之外还支持在tls和kcp之后进行自定义  
加密，也就是说自定义加密和tls|kcp是可以联合使用的，内部采用AES256加密，使用的时候只需要自己定义一个密码即可，  
加密分为两个部分，一部分是本地(-z)是否加密解密，一部分是与上级(-Z)传输是否加密解密。  
自定义加密要求两端都是proxy才可以，下面分别用二级，三级为例:  

二级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy http -t tcp -z demo_password -p :7777`  
本地二级执行:  
`proxy http -T tcp -P 2.2.2.2:777 -Z demo_password -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级加密传输访问目标网站。  


三级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy http -t tcp -z demo_password -p :7777`  
二级vps(ip:3.3.3.3)上执行:  
`proxy http -T tcp -P 2.2.2.2:7777 -Z demo_password -t tcp -z other_password -p :8888` 
本地三级执行:  
`proxy http -T tcp -P 3.3.3.3:8888 -Z other_password -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级加密传输访问目标网站。  

### 1.13 压缩传输
proxy的http(s)代理在tcp之上可以通过tls标准加密以及kcp协议加密tcp数据，在自定义加密之前还可以对数据进行压缩，  
也就是说压缩功能和自定义加密和tls|kcp是可以联合使用的，压缩分为两个部分，一部分是本地(-m)是否压缩传输，  
一部分是与上级(-M)传输是否压缩。  
压缩要求两端都是proxy才可以，压缩也在一定程度上保护了(加密)数据，下面分别用二级，三级为例:  

二级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy http -t tcp -m -p :7777`  
本地二级执行:  
`proxy http -T tcp -P 2.2.2.2:777 -M -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级压缩传输访问目标网站。  


三级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy http -t tcp -m -p :7777`  
二级vps(ip:3.3.3.3)上执行:  
`proxy http -T tcp -P 2.2.2.2:7777 -M -t tcp -m -p :8888` 
本地三级执行:  
`proxy http -T tcp -P 3.3.3.3:8888 -M -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级压缩传输访问目标网站。  

### 1.14 负载均衡

HTTP(S)代理支持上级负载均衡，多个上级重复-P参数即可。  

`proxy http --lb-method=hash -T tcp -P 1.1.1.1:33080 -P 2.1.1.1:33080 -P 3.1.1.1:33080`

### 1.14.1 设置重试间隔和超时时间

`proxy http --lb-method=leastconn --lb-retrytime 300 --lb-timeout 300 -T tcp -P 1.1.1.1:33080 -P 2.1.1.1:33080 -P 3.1.1.1:33080 -t tcp -p :33080`

### 1.14.2 设置权重

`proxy http --lb-method=weight -T tcp -P 1.1.1.1:33080?w=1 -P 2.1.1.1:33080?w=2 -P 3.1.1.1:33080?w=1 -t tcp -p :33080`

### 1.14.3 使用目标地址选择上级

`proxy http --lb-hashtarget --lb-method=hash -T tcp -P 1.1.1.1:33080 -P 2.1.1.1:33080 -P 3.1.1.1:33080 -t tcp -p :33080`

### 1.15 限速

限速100K，通过`-l`参数即可指定，比如:100K 2000K 1M . 0意味着无限制。  

`proxy http -t tcp -p 2.2.2.2:33080 -l 100K`

### 1.16 指定出口IP

`--bind-listen`参数，就可以开启客户端用入口IP连接过来的，就用入口IP作为出口IP访问目标网站的功能。如果绑定了不正确的IP会导致代理不能工作，此时代理会尝试不绑定IP去访问目标，同时日志会提示。

`proxy http -t tcp -p 2.2.2.2:33080 --bind-listen`

### 1.17 证书参数使用base64数据

默认情况下-C，-K参数是crt证书和key文件的路径，

如果是base64://开头，那么就认为后面的数据是base64编码的，会解码后使用。  

### 1.18 智能模式
智能模式设置，可以是intelligent|direct|parent三者之一。  
默认是:intelligent。  
每个值的含义如下:  
`--intelligent=direct`，不在blocked里面的目标都直连。  
`--intelligent=parent`，不在direct里面的目标都走上级。  
`--intelligent=intelligent`，blocked和direct里面都没有的目标，智能判断是否使用上级访问目标。  

### 1.19 查看帮助
`./proxy help http`  
  
## 2.TCP代理
  
### 2.1.普通一级TCP代理
![2.1](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/tcp-1.png)  
本地执行:  
`./proxy tcp -p ":33080" -T tcp -P "192.168.22.33:22"`  
那么访问本地33080端口就是访问192.168.22.33的22端口。  
  
### 2.2.普通二级TCP代理
![2.2](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/tcp-2.png)  
VPS(IP:22.22.22.33)执行:  
`./proxy tcp -p ":33080" -T tcp -P "127.0.0.1:8080"`  
本地执行:  
`./proxy tcp -p ":23080" -T tcp -P "22.22.22.33:33080"`  
那么访问本地23080端口就是访问22.22.22.33的8080端口。  
  
### 2.3.普通三级TCP代理
![2.3](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/tcp-3.png)  
一级TCP代理VPS_01，IP:22.22.22.22  
`./proxy tcp -p ":38080" -T tcp -P "66.66.66.66:8080"`  
二级TCP代理VPS_02，IP:33.33.33.33  
`./proxy tcp -p ":28080" -T tcp -P "22.22.22.22:38080"`  
三级TCP代理(本地)  
`./proxy tcp -p ":8080" -T tcp -P "33.33.33.33:28080"`  
那么访问本地8080端口就是通过加密TCP隧道访问66.66.66.66的8080端口。  
  
### 2.4.加密二级TCP代理
![2.4](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/tcp-tls-2.png)  
VPS(IP:22.22.22.33)执行:  
`./proxy tcp -t tls -p ":33080" -T tcp -P "127.0.0.1:8080" -C proxy.crt -K proxy.key`  
本地执行:  
`./proxy tcp -p ":23080" -T tls -P "22.22.22.33:33080" -C proxy.crt -K proxy.key`  
那么访问本地23080端口就是通过加密TCP隧道访问22.22.22.33的8080端口。  
  
### 2.5.加密三级TCP代理
![2.5](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/tcp-tls-3.png)  
一级TCP代理VPS_01，IP:22.22.22.22  
`./proxy tcp -t tls -p ":38080" -T tcp -P "66.66.66.66:8080" -C proxy.crt -K proxy.key`  
二级TCP代理VPS_02，IP:33.33.33.33  
`./proxy tcp -t tls -p ":28080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  
三级TCP代理(本地)  
`./proxy tcp -p ":8080" -T tls -P "33.33.33.33:28080" -C proxy.crt -K proxy.key`  
那么访问本地8080端口就是通过加密TCP隧道访问66.66.66.66的8080端口。  
  
### 2.6.通过代理连接上级
有时候proxy所在的网络不能直接访问外网，需要通过一个https或者socks5代理才能上网，那么这个时候  
-J参数就可以帮助你让proxy的tcp端口映射的时候通过https或者socks5代理去连接上级-P，将外部端口映射到本地。  
-J参数格式如下:  

https代理写法:  
代理需要认证，用户名:username 密码:password  
https://username:password@host:port  
代理不需要认证  
https://host:port  

socks5代理写法:
代理需要认证，用户名:username 密码:password
socks5://username:password@host:port
代理不需要认证
socks5://host:port

host:代理的IP或者域名
port:代理的端口

### 2.7.查看帮助
`./proxy help tcp`  
  
## 3.UDP代理
  
### 3.1.普通一级UDP代理
![3.1](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/udp-1.png)  
本地执行:  
`./proxy udp -p ":5353" -T udp -P "8.8.8.8:53"`  
那么访问本地UDP:5353端口就是访问8.8.8.8的UDP:53端口。  
  
### 3.2.普通二级UDP代理
![3.2](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/udp-2.png)  
VPS(IP:22.22.22.33)执行:  
`./proxy tcp -p ":33080" -T udp -P "8.8.8.8:53"`  
本地执行:  
`./proxy udp -p ":5353" -T tcp -P "22.22.22.33:33080"`  
那么访问本地UDP:5353端口就是通过TCP隧道，通过VPS访问8.8.8.8的UDP:53端口。  
  
### 3.3.普通三级UDP代理
![3.3](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/udp-3.png)  
一级TCP代理VPS_01，IP:22.22.22.22  
`./proxy tcp -p ":38080" -T udp -P "8.8.8.8:53"`  
二级TCP代理VPS_02，IP:33.33.33.33  
`./proxy tcp -p ":28080" -T tcp -P "22.22.22.22:38080"`  
三级TCP代理(本地)  
`./proxy udp -p ":5353" -T tcp -P "33.33.33.33:28080"`  
那么访问本地5353端口就是通过TCP隧道，通过VPS访问8.8.8.8的53端口。  
  
### 3.4.加密二级UDP代理
![3.4](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/udp-tls-2.png)  
VPS(IP:22.22.22.33)执行:  
`./proxy tcp -t tls -p ":33080" -T udp -P "8.8.8.8:53" -C proxy.crt -K proxy.key`  
本地执行:  
`./proxy udp -p ":5353" -T tls -P "22.22.22.33:33080" -C proxy.crt -K proxy.key`  
那么访问本地UDP:5353端口就是通过加密TCP隧道，通过VPS访问8.8.8.8的UDP:53端口。  
  
### 3.5.加密三级UDP代理
![3.5](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/udp-tls-3.png)  
一级TCP代理VPS_01，IP:22.22.22.22  
`./proxy tcp -t tls -p ":38080" -T udp -P "8.8.8.8:53" -C proxy.crt -K proxy.key`  
二级TCP代理VPS_02，IP:33.33.33.33  
`./proxy tcp -t tls -p ":28080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  
三级TCP代理(本地)  
`./proxy udp -p ":5353" -T tls -P "33.33.33.33:28080" -C proxy.crt -K proxy.key`  
那么访问本地5353端口就是通过加密TCP隧道，通过VPS_01访问8.8.8.8的53端口。  
  
### 3.6.查看帮助
`./proxy help udp`  
  
## 4.内网穿透
### 4.1、原理说明
内网穿透，分为两个版本，“多链接版本”和“多路复用版本”，一般像web服务这种不是长时间连接的服务建议用“多链接版本”，如果是要保持长时间连接建议使用“多路复用版本”。
1. 多链接版本，对应的子命令是tserver，tclient，tbridge。  
1. 多路复用版本，对应的子命令是server，client，bridge。  
1. 多链接版本和多路复用版本的参数和使用方式完全一样。  
1. 多路复用版本的server，client可以开启压缩传输，参数是--c。 
1. server，client要么都开启压缩，要么都不开启，不能只开一个。  

下面的教程以“多路复用版本”为例子，说明使用方法。    
内网穿透由三部分组成:client端，server端，bridge端；client和server主动连接bridge端进行桥接。  
  
### 4.2、TCP普通用法
背景:  
- 公司机器A提供了web服务80端口  
- 有VPS一个，公网IP:22.22.22.22  

需求:  
在家里能够通过访问VPS的28080端口访问到公司机器A的80端口  
  
步骤:  
1. 在vps上执行  
    `./proxy bridge -p ":33080" -C proxy.crt -K proxy.key`  
    `./proxy server -r ":28080@:80" -P "127.0.0.1:33080" -C proxy.crt -K proxy.key`  
  
1. 在公司机器A上面执行  
    `./proxy client -P "22.22.22.22:33080" -C proxy.crt -K proxy.key`  

1. 完成  
  
### 4.3、微信接口本地开发
背景:  
- 自己的笔记本提供了nginx服务80端口  
- 有VPS一个，公网IP:22.22.22.22  

需求:  
在微信的开发帐号的网页回调接口配置里面填写地址:http://22.22.22.22/calback.php  
然后就可以访问到笔记本的80端口下面的calback.php，如果需要绑定域名，可以用自己的域名  
比如:wx-dev.xxx.com解析到22.22.22.22，然后在自己笔记本的nginx里  
配置域名wx-dev.xxx.com到具体的目录即可。  

  
步骤:  
1. 在vps上执行，确保vps的80端口没被其它程序占用。  
    `./proxy bridge -p ":33080" -C proxy.crt -K proxy.key`  
    `./proxy server -r ":80@:80" -P "22.22.22.22:33080" -C proxy.crt -K proxy.key`  

1. 在自己笔记本上面执行  
    `./proxy client -P "22.22.22.22:33080" -C proxy.crt -K proxy.key`  

1. 完成  
  
### 4.4、UDP普通用法
背景:  
- 公司机器A提供了DNS解析服务，UDP:53端口  
- 有VPS一个，公网IP:22.22.22.22  
  
需求:  
在家里能够通过设置本地dns为22.22.22.22，使用公司机器A进行域名解析服务。  
  
步骤:  
1. 在vps上执行  
    `./proxy bridge -p ":33080" -C proxy.crt -K proxy.key`  
    `./proxy server --udp -r ":53@:53" -P "127.0.0.1:33080" -C proxy.crt -K proxy.key`  

1. 在公司机器A上面执行  
    `./proxy client -P "22.22.22.22:33080" -C proxy.crt -K proxy.key`  

1. 完成  
  
### 4.5、高级用法一
背景:  
- 公司机器A提供了web服务80端口  
- 有VPS一个，公网IP:22.22.22.22  
  
需求:  
为了安全，不想在VPS上能够访问到公司机器A，在家里能够通过访问本机的28080端口，  
通过加密隧道访问到公司机器A的80端口。  
  
步骤:  
1. 在vps上执行  
    `./proxy bridge -p ":33080" -C proxy.crt -K proxy.key`  
  
1. 在公司机器A上面执行  
    `./proxy client -P "22.22.22.22:33080" -C proxy.crt -K proxy.key`  
  
1. 在家里电脑上执行  
    `./proxy server -r ":28080@:80" -P "22.22.22.22:33080" -C proxy.crt -K proxy.key`  
  
1. 完成  
  
### 4.6、高级用法二
提示:  
如果同时有多个client连接到同一个bridge，需要指定不同的key，可以通过--k参数设定，--k可以是任意唯一字符串，  
只要在同一个bridge上唯一即可。  
server连接到bridge的时候，如果同时有多个client连接到同一个bridge，需要使用--k参数选择client。  
暴露多个端口重复-r参数即可.-r格式是:"本地IP:本地端口@clientHOST:client端口"。  
  
背景:  
- 公司机器A提供了web服务80端口，ftp服务21端口  
- 有VPS一个，公网IP:22.22.22.22  
  
需求:  
在家里能够通过访问VPS的28080端口访问到公司机器A的80端口  
在家里能够通过访问VPS的29090端口访问到公司机器A的21端口  
  
步骤:  
1. 在vps上执行  
    `./proxy bridge -p ":33080" -C proxy.crt -K proxy.key`  
    `./proxy server -r ":28080@:80" -r ":29090@:21" --k test -P "127.0.0.1:33080" -C proxy.crt -K proxy.key`  

1. 在公司机器A上面执行  
    `./proxy client --k test -P "22.22.22.22:33080" -C proxy.crt -K proxy.key` 

1. 完成  
  
### 4.7.server的-r参数
  -r完整格式是:`PROTOCOL://LOCAL_IP:LOCAL_PORT@[CLIENT_KEY]CLIENT_LOCAL_HOST:CLIENT_LOCAL_PORT`  
  
  4.7.1.协议PROTOCOL:tcp、udp、ptcp、pudp。  
  比如: `-r "udp://:10053@:53" -r "tcp://:10800@:1080" -r ":8080@:80"`  
  如果指定了--udp参数，PROTOCOL默认为udp，那么:`-r ":8080@:80"`默认为udp;  
  如果没有指定--udp参数，PROTOCOL默认为tcp，那么:`-r ":8080@:80"`默认为tcp;  
  
  4.7.2.CLIENT_KEY:默认是default。  
  比如: -r "udp://:10053@[test1]:53" -r "tcp://:10800@[test2]:1080" -r ":8080@:80"  
  如果指定了--k参数，比如--k test，那么:`-r ":8080@:80"`CLIENT_KEY默认为test;  
  如果没有指定--k参数，那么:`-r ":8080@:80"`CLIENT_KEY默认为default;  
  
  4.7.3.LOCAL_IP为空默认是:`0.0.0.0`，CLIENT_LOCAL_HOST为空默认是:`127.0.0.1`; 

### 4.8.server和client通过代理连接bridge
有时候server或者client所在的网络不能直接访问外网，需要通过一个https或者socks5代理才能上网，那么这个时候  
-J参数就可以帮助你让server或者client通过https或者socks5代理去连接bridge。  
-J参数格式如下:  

https代理写法:  
代理需要认证，用户名:username 密码:password  
https://username:password@host:port  
代理不需要认证  
https://host:port  

socks5代理写法:
代理需要认证，用户名:username 密码:password
socks5://username:password@host:port
代理不需要认证
socks5://host:port

host:代理的IP或者域名
port:代理的端口

### 4.9.内网穿透HTTP服务

通常HTTP请求客户端会使用server的ip和端口去设置HOST字段，但是与期望的后端实际HOST不一样，这样就造成了tcp是通的，  
但后端依赖HOST字段定位虚拟主机就不能工作.现在用--http-host参数强制设置http头部的HOST字段值为后端实际的  
域名和端口即可轻松解决。  

`server`的--http-host参数格式如下:

`--http-host www.test.com:80@2200`，如果server监听多个端口，只需要重复`--http-host`参数设置每个端口的HOST即可。  

实例:

比如client本地nginx，127.0.0.1:80提供了web服务，其中绑定了一个域名`local.com`。  

那么server的启动参数可以如下:

`proxy server -P :30000 -r :2500@127.0.0.1:80 --http-host local.com@2500`  

解释:

`-r :2500@127.0.0.1:80` 和 `--http-host local.com:80@2500` 里面的2500端口是server本地监听的端口

当使用http协议请求server的ip:2500端口的时候，http的头部HOST字段就会被设置为`local.com`。  

### 4.10 关于流量统计
如果单独启动一个server对接上级是proxy-admin控制面板，需要在上级控制面板里面新建一个映射，获得个映射规则的ID，

然后启动server的时候加上参数 --server-id=映射规则的ID 才能统计到流量。

### 4.11 关于p2p
内网穿透支持在server和client网络情况满足的情况下，server和client之间通过p2p直接连接，开启方法是：

在启动bridge，server，client到时候都加上`--p2p`参数即可。server的-r参数可以针对端口是否启用p2p（ptcp和pudp）。

如果server和client之间p2p打洞失败，那么会自动切换使用bridge中转传输数据。

### 4.12 客户端key白名单
内网穿透bridge可以设置客户端key白名单，参数是--client-keys，格式可以是：

a.文件名,文件内容一行一个客户端key只能包含数字字母下划线，也就是客户端启动参数--k的值，只有客户端key在此白名单的客户端才能连接。# 开头的行，为注释。

b.“base64://”开头的base64编码的上面a说明的文件内容，比如：base64://ajfpoajsdfa=

c.”str://“开头的英文逗号分割的多个key，比如：str://default,company,school

默认是空，允许所有key。

### 4.13 网络NAT类型判断

nat类型判断,方便查看网络是否支持p2p，可以执行：`proxy tools -a nattype`

### 4.14 查看帮助
`./proxy help bridge`  
`./proxy help server`  
`./proxy help client`  
  
## 5.SOCKS5代理
提示:

SOCKS5代理，支持CONNECT，UDP协议，不支持BIND，支持用户名密码认证。  

***如果你的VPS是阿里云，腾讯云这种VPS，就是ifconfig看不见你的公网IP，只能看见内网IP，***

***那么需要加上`-g VPS公网IP`参数，SOCKS5代理的UDP功能才能正常工作。***

### 5.1.普通SOCKS5代理
`./proxy socks -t tcp -p "0.0.0.0:38080"`  
  
### 5.2.普通二级SOCKS5代理
![5.2](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/socks-2.png)  
使用本地端口8090，假设上级SOCKS5代理是`22.22.22.22:8080`  
`./proxy socks -t tcp -p "0.0.0.0:8090" -T tcp -P "22.22.22.22:8080" `  
我们还可以指定网站域名的黑白名单文件，一行一个域名，匹配规则是最右匹配，比如:baidu.com，匹配的是*.*.baidu.com，黑名单的域名域名直接走上级代理，白名单的域名不走上级代理;如果域名即在黑名单又在白名单中，那么黑名单起作用。  
`./proxy socks -p "0.0.0.0:8090" -T tcp -P "22.22.22.22:8080"  -b blocked.txt -d direct.txt`  
  
### 5.3.SOCKS二级代理(加密)
![5.3](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/socks-tls-2.png)  
一级SOCKS代理(VPS，IP:22.22.22.22)  
`./proxy socks -t tls -p ":38080" -C proxy.crt -K proxy.key`  
  
二级SOCKS代理(本地Linux)  
`./proxy socks -t tcp -p ":8080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  
那么访问本地的8080端口就是访问VPS上面的代理端口38080。  
  
二级SOCKS代理(本地windows)  
`./proxy.exe socks -t tcp -p ":8080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  
然后设置你的windos系统中，需要通过代理上网的程序的代理为socks5模式，地址为：127.0.0.1，端口为：8080，程序即可通过加密通道通过vps上网。  
  
### 5.4.SOCKS三级代理(加密)
![5.4](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/socks-tls-3.png)  
一级SOCKS代理VPS_01，IP:22.22.22.22  
`./proxy socks -t tls -p ":38080" -C proxy.crt -K proxy.key`  
二级SOCKS代理VPS_02，IP:33.33.33.33  
`./proxy socks -t tls -p ":28080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  
三级SOCKS代理(本地)  
`./proxy socks -t tcp -p ":8080" -T tls -P "33.33.33.33:28080" -C proxy.crt -K proxy.key`  
那么访问本地的8080端口就是访问一级SOCKS代理上面的代理端口38080。  
  
### 5.5.SOCKS代理流量强制走上级SOCKS代理
默认情况下，proxy会智能判断一个网站域名是否无法访问，如果无法访问才走上级SOCKS代理.通过--always可以使全部SOCKS代理流量强制走上级SOCKS代理。  
`./proxy socks --always -t tls -p ":28080" -T tls -P "22.22.22.22:38080" -C proxy.crt -K proxy.key`  
  
### 5.6.SOCKS通过SSH中转
![5.6](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/socks-ssh.png)  
说明:ssh中转的原理是利用了ssh的转发功能，就是你连接上ssh之后，可以通过ssh代理访问目标地址。  
假设有:vps  
- IP是2.2.2.2， ssh端口是22， ssh用户名是:user， ssh用户密码是:demo
- 用户user的ssh私钥名称是user.key   

#### *5.6.1 ssh用户名和密码的方式*
本地SOCKS5代理28080端口，执行:  
`./proxy socks -T ssh -P "2.2.2.2:22" -u user -D demo -t tcp -p ":28080"`  
#### *5.6.2 ssh用户名和密钥的方式*
本地SOCKS5代理28080端口，执行:  
`./proxy socks -T ssh -P "2.2.2.2:22" -u user -S user.key -t tcp -p ":28080"`  

那么访问本地的28080端口就是通过VPS访问目标地址。  

### 5.7.认证
对于socks5代理协议我们可以进行用户名密码认证，认证的用户名和密码可以在命令行指定  
`./proxy socks -t tcp -p ":33080" -a "user1:pass1" -a "user2:pass2"`  
多个用户，重复-a参数即可。  
也可以放在文件中，格式是一行一个"用户名:密码"，然后用-F指定。  
`./proxy socks -t tcp -p ":33080" -F auth-file.txt`  

另外，socks5代理还集成了外部HTTP API认证，我们可以通过--auth-url参数指定一个http url接口地址，  
然后有用户连接的时候，proxy会GET方式请求这url，带上下面三个参数，如果返回HTTP状态码204，代表认证成功  
其它情况认为认证失败。  
比如:  
`./proxy socks -t tcp -p ":33080" --auth-url "http://test.com/auth.php"`  
用户连接的时候，proxy会GET方式请求这url("http://test.com/auth.php")，  
带上user，pass，ip，local_ip四个参数:  
http://test.com/auth.php?user={USER}&pass={PASS}&ip={IP}&local_ip={LOCAL_IP}  
user:用户名  
pass:密码  
ip:用户的IP，比如:192.168.1.200  
local_ip:用户访问的服务器的IP，比如:3.3.3.3   

如果没有-a或-F或--auth-url参数，就是关闭认证。  

### 5.8.KCP协议传输
KCP协议需要--kcp-key参数设置一个密码用于加密解密数据  

一级HTTP代理(VPS，IP:22.22.22.22)  
`./proxy socks -t kcp -p ":38080" --kcp-key mypassword`  
  
二级HTTP代理(本地Linux)  
`./proxy socks -t tcp -p ":8080" -T kcp -P "22.22.22.22:38080" --kcp-key mypassword`  
那么访问本地的8080端口就是访问VPS上面的代理端口38080，数据通过kcp协议传输。  

### 5.9.自定义DNS
--dns-address和--dns-ttl参数，用于自己指定proxy访问域名的时候使用的dns（--dns-address）  
以及解析结果缓存时间（--dns-ttl）秒数，避免系统dns对proxy的干扰，另外缓存功能还能减少dns解析时间提高访问速度。  
比如：  
`./proxy socks -p ":33080" --dns-address "8.8.8.8:53" --dns-ttl 300`  

### 5.10 自定义加密
proxy的socks代理在tcp之上可以通过tls标准加密以及kcp协议加密tcp数据，除此之外还支持在tls和kcp之后进行自定义加密，也就是说自定义加密和tls|kcp是可以联合使用的，内部采用AES256加密，使用的时候只需要自己定义一个密码即可，  
加密分为两个部分，一部分是本地(-z)是否加密解密，一部分是与上级(-Z)传输是否加密解密。  

自定义加密要求两端都是proxy才可以。  

下面分别用二级，三级为例:  

二级实例
一级vps(ip:2.2.2.2)上执行:  
`proxy socks -t tcp -z demo_password -p :7777`  
本地二级执行:  
`proxy socks -T tcp -P 2.2.2.2:777 -Z demo_password -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级加密传输访问目标网站。  


三级实例
一级vps(ip:2.2.2.2)上执行:  
`proxy socks -t tcp -z demo_password -p :7777`  
二级vps(ip:3.3.3.3)上执行:  
`proxy socks -T tcp -P 2.2.2.2:7777 -Z demo_password -t tcp -z other_password -p :8888` 
本地三级执行:  
`proxy socks -T tcp -P 3.3.3.3:8888 -Z other_password -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级加密传输访问目标网站。  

### 5.11 压缩传输
proxy的socks代理在tcp之上可以通过自定义加密和tls标准加密以及kcp协议加密tcp数据，在自定义加密之前还可以  
对数据进行压缩，也就是说压缩功能和自定义加密和tls|kcp是可以联合使用的，压缩分为两个部分，   
一部分是本地(-m)是否压缩传输，一部分是与上级(-M)传输是否压缩。  

压缩要求两端都是proxy才可以，压缩也在一定程度上保护了(加密)数据。  

下面分别用二级，三级为例:  

二级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy socks -t tcp -m -p :7777`  
本地二级执行:  
`proxy socks -T tcp -P 2.2.2.2:777 -M -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级压缩传输访问目标网站。  


三级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy socks -t tcp -m -p :7777`  
二级vps(ip:3.3.3.3)上执行:  
`proxy socks -T tcp -P 2.2.2.2:7777 -M -t tcp -m -p :8888` 
本地三级执行:  
`proxy socks -T tcp -P 3.3.3.3:8888 -M -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级压缩传输访问目标网站。  


### 5.12 负载均衡

SOCKS代理支持上级负载均衡，多个上级重复-P参数即可。  

`proxy socks --lb-method=hash -T tcp -P 1.1.1.1:33080 -P 2.1.1.1:33080 -P 3.1.1.1:33080  -p :33080 -t tcp`

### 5.12.1 设置重试间隔和超时时间

`proxy socks --lb-method=leastconn --lb-retrytime 300 --lb-timeout 300 -T tcp -P 1.1.1.1:33080 -P 2.1.1.1:33080 -P 3.1.1.1:33080 -p :33080 -t tcp`

### 5.12.2 设置权重

`proxy socks --lb-method=weight -T tcp -P 1.1.1.1:33080?w=1 -P 2.1.1.1:33080?w=2 -P 3.1.1.1:33080?w=1 -p :33080 -t tcp`

### 5.12.3 使用目标地址选择上级

`proxy socks --lb-hashtarget --lb-method=hash -T tcp -P 1.1.1.1:33080 -P 2.1.1.1:33080 -P 3.1.1.1:33080 -p :33080 -t tcp`

### 5.13 限速

限速100K，通过`-l`参数即可指定，比如:100K 2000K 1M . 0意味着无限制。  

`proxy socks -t tcp -p 2.2.2.2:33080 -l 100K`

### 5.14 指定出口IP

`--bind-listen`参数，就可以开启客户端用入口IP连接过来的，就用入口IP作为出口IP访问目标网站的功能。如果入口IP是内网IP，出口IP不会使用入口IP。

`proxy socks -t tcp -p 2.2.2.2:33080 --bind-listen`

### 5.15 级联认证

SOCKS5支持级联认证，-A可以设置上级认证信息。  

上级:

`proxy socks -t tcp -p 2.2.2.2:33080 -a user:pass`

本地:

`proxy socks -T tcp -P 2.2.2.2:33080 -A user:pass -t tcp -p :33080`

### 5.16 证书参数使用base64数据

默认情况下-C，-K参数是crt证书和key文件的路径，

如果是base64://开头，那么就认为后面的数据是base64编码的，会解码后使用。  


### 5.17 智能模式
智能模式设置，可以是intelligent|direct|parent三者之一。  
默认是:intelligent。  
每个值的含义如下:  
`--intelligent=direct`，不在blocked里面的目标都直连。  
`--intelligent=parent`，不在direct里面的目标都走上级。  
`--intelligent=intelligent`，blocked和direct里面都没有的目标，智能判断是否使用上级访问目标。  

### 5.18.查看帮助
`./proxy help socks`  

## 6.SPS协议转换

### 6.1 功能介绍
代理协议转换使用的是sps子命令，sps本身不提供代理功能，只是接受代理请求"转换并转发"给已经存在的http(s)代理或者socks5代理或者ss代理；sps可以把已经存在的http(s)代理或者socks5代理或ss代理转换为一个端口同时支持http(s)和socks5和ss的代理，而且http(s)代理支持正向代理和反向代理(SNI)，转换后的SOCKS5代理，当上级是SOCKS5或者SS时仍然支持UDP功能；另外对于已经存在的http(s)代理或者socks5代理，支持tls、tcp、kcp三种模式，支持链式连接，也就是可以多个sps结点层级连接构建加密通道。

`ss`功能支持的加密方法为:aes-128-cfb ， aes-128-ctr ， aes-128-gcm ， aes-192-cfb ， aes-192-ctr ， aes-192-gcm ， aes-256-cfb ， aes-256-ctr ， aes-256-gcm ， bf-cfb ， cast5-cfb ， chacha20 ， chacha20-ietf ， chacha20-ietf-poly1305 ， des-cfb ， rc4-md5 ， rc4-md5-6 ， salsa20 ， xchacha20

### 6.2 HTTP(S)转HTTP(S)+SOCKS5+SS
假设已经存在一个普通的http(s)代理：127.0.0.1:8080，现在我们把它转为同时支持http(s)和socks5和ss的普通代理，转换后的本地端口为18080，ss加密方式:aes-192-cfb，ss密码:pass。  
命令如下：  
`./proxy sps -S http -T tcp -P 127.0.0.1:8080 -t tcp -p :18080 -h aes-192-cfb -j pass`

假设已经存在一个tls的http(s)代理：127.0.0.1:8080，现在我们把它转为同时支持http(s)和socks5和ss的普通代理，转换后的本地端口为18080，tls需要证书文件，ss加密方式:aes-192-cfb，ss密码:pass。  
命令如下：  
`./proxy sps -S http -T tls -P 127.0.0.1:8080 -t tcp -p :18080 -C proxy.crt -K proxy.key -h aes-192-cfb -j pass`   

假设已经存在一个kcp的http(s)代理（密码是：demo123）：127.0.0.1:8080，现在我们把它转为同时支持http(s)和socks5和ss的普通代理，转换后的本地端口为18080，ss加密方式:aes-192-cfb，ss密码:pass。  
命令如下：  
`./proxy sps -S http -T kcp -P 127.0.0.1:8080 -t tcp -p :18080 --kcp-key demo123 -h aes-192-cfb -j pass`  

### 6.3 SOCKS5转HTTP(S)+SOCKS5+SS
假设已经存在一个普通的socks5代理：127.0.0.1:8080，现在我们把它转为同时支持http(s)和socks5和ss的普通代理，转换后的本地端口为18080，ss加密方式:aes-192-cfb，ss密码:pass。    
命令如下：  
`./proxy sps -S socks -T tcp -P 127.0.0.1:8080 -t tcp -p :18080 -h aes-192-cfb -j pass`

假设已经存在一个tls的socks5代理：127.0.0.1:8080，现在我们把它转为同时支持http(s)和socks5和ss的普通代理，转换后的本地端口为18080，tls需要证书文件，ss加密方式:aes-192-cfb，ss密码:pass。   
命令如下：  
`./proxy sps -S socks -T tls -P 127.0.0.1:8080 -t tcp -p :18080 -C proxy.crt -K proxy.key -h aes-192-cfb -j pass`   

假设已经存在一个kcp的socks5代理（密码是：demo123）：127.0.0.1:8080，现在我们把它转为同时支持http(s)和socks5和ss的普通代理，转换后的本地端口为18080，ss加密方式:aes-192-cfb，ss密码:pass。  
命令如下：  
`./proxy sps -S socks -T kcp -P 127.0.0.1:8080 -t tcp -p :18080 --kcp-key demo123 -h aes-192-cfb -j pass`  

### 6.4 SS转HTTP(S)+SOCKS5+SS
SPS上级和本地支持ss协议，上级可以是SPS或者标准的ss服务。  
SPS本地默认提供HTTP(S)\SOCKS5\SPS三种代理，当上级是SOCKS5时转换后的SOCKS5和SS支持UDP功能。  
假设已经存在一个普通的SS或者SPS代理(开启了ss，加密方式:aes-256-cfb，密码:demo)：127.0.0.1:8080，现在我们把它转为同时支持http(s)和socks5和ss的普通代理，转换后的本地端口为18080，转换后的ss加密方式:aes-192-cfb，ss密码:pass。  
命令如下：  
`./proxy sps -S ss -H aes-256-cfb -J pass -T tcp -P 127.0.0.1:8080 -t tcp -p :18080 -h aes-192-cfb -j pass`。  

### 6.5 链式连接 
![6.4](https://raw.githubusercontent.com/snail007/goproxy/master/doc/images/sps-tls.png)  
上面提过多个sps结点可以层级连接构建加密通道，假设有如下vps和家里的pc电脑。  
vps01：2.2.2.2  
vps02：3.3.3.3  
现在我们想利用pc和vps01和vps02构建一个加密通道，本例子用tls加密也可以用kcp，在pc上访问本地18080端口就是访问vps01的本地8080端口。  
首先在vps01(2.2.2.2)上我们运行一个只有本地可以访问的http(s)代理，执行：  
`./proxy http -t tcp -p 127.0.0.1:8080`  

然后在vps01(2.2.2.2)上运行一个sps结点，执行：  
`./proxy sps -S http -T tcp -P 127.0.0.1:8080 -t tls -p :8081 -C proxy.crt -K proxy.key`  

然后在vps02(3.3.3.3)上运行一个sps结点，执行：  
`./proxy sps -S http -T tls -P 2.2.2.2:8081 -t tls -p :8082 -C proxy.crt -K proxy.key`  

然后在pc上运行一个sps结点，执行：  
`./proxy sps -S http -T tls -P 3.3.3.3:8082 -t tcp -p :18080 -C proxy.crt -K proxy.key`  

完成。  

### 6.6 监听多个端口 
一般情况下监听一个端口就可以，不过如果作为反向代理需要同时监听80和443两个端口，那么-p参数是支持的，  
格式是：`-p 0.0.0.0:80，0.0.0.0:443`，多个绑定用逗号分隔即可。  

### 6.7 认证功能 
sps支持http(s)\socks5代理认证，可以级联认证，有四个重要的信息:  
1:用户发送认证信息`user-auth`。   
2:设置的本地认证信息`local-auth`。  
3:设置的连接上级使用的认证信息`parent-auth`。  
4:最终发送给上级的认证信息`auth-info-to-parent`。  
他们的情况关系如下:  

| user-auth | local-auth | parent-auth | auth-info-to-paren 
| ------ | ------ | ------ | ------  
| 有/没有  | 有     |     有   |   来自parent-auth  
| 有/没有  | 没有    |    有    |   来自parent-auth  
| 有/没有  | 有     |     没有  |   无  
| 没有   | 没有    |   没有    |   无  
| 有    | 没有    |   没有    |   来自user-auth  

对于sps代理我们可以进行用户名密码认证，认证的用户名和密码可以在命令行指定    
`./proxy sps -S http -T tcp -P 127.0.0.1:8080 -t tcp -p ":33080" -a "user1:pass1" -a "user2:pass2"`  
多个用户，重复-a参数即可。  
也可以放在文件中，格式是一行一个"用户名:密码"，然后用-F指定。  
`./proxy sps -S http -T tcp -P 127.0.0.1:8080 -t tcp -p ":33080" -F auth-file.txt`  

如果上级有认证，下级可以通过-A参数设置认证信息，比如:  
上级:`./proxy sps -S http -T tcp -P 127.0.0.1:8080 -t tcp -p ":33080" -a "user1:pass1" -a "user2:pass2"`  
下级:`./proxy sps -S http -T tcp -P 127.0.0.1:8080 -A "user1:pass1" -t tcp -p ":33080" `  

另外，sps代理，本地认证集成了外部HTTP API认证，我们可以通过--auth-url参数指定一个http url接口地址，    
然后有用户连接的时候，proxy会GET方式请求这url，带上下面四个参数，如果返回HTTP状态码204，代表认证成功  
其它情况认为认证失败。  
比如:  
`./proxy sps -S http -T tcp -P 127.0.0.1:8080 -t tcp -p ":33080" --auth-url "http://test.com/auth.php"`  
用户连接的时候，proxy会GET方式请求这url("http://test.com/auth.php")，  
带上user，pass，ip，target四个参数:  
http://test.com/auth.php?user={USER}&pass={PASS}&ip={IP}&target={TARGET}  
user:用户名   
pass:密码   
ip:用户的IP，比如:192.168.1.200   
target:如果客户端是http(s)代理请求，这里代表的是请求的完整url，其它情况为空。  

如果没有-a或-F或--auth-url参数，就是关闭本地认证。  
如果没有-A参数，连接上级不使用认证。  

**设置单独认证信息**

如果存在多个不同上级，而且他们的密码有的一样有的不一样，那么可以针对每个上级设置认证信息，  
同时还可以用-A参数设置一个全局认证信息，如果某个上级没有单独设置认证信息就使用全局设置的认证信息。  
认证信息和上级写在一起。  
格式是: a:b@2.2.2.2:33080#1  
说明:  
    用户是a密码是b，如果用户名和密码保护特殊符号可以使用urlencode进行编码  
    如果是ss，那么a就是加密方法，b是密码，比如:aes-192-cfb:your_pass  
`@`是间隔符号，如果有认证信息，必须有@，没有认证信息可以省略@  
2.2.2.2:33080 是上级地址  
`#`1 是设置权重，没有可以省略，详细说明可以参考手册***权重部分*

**设置单独认证信息的协议类型**

如果存在多个不同上级，而且他们的密码和传输协议有的一样有的不一样，那么可以针对每个上级设置认证信息的协议类型，  
同时还可以用-S和-T参数设置一个全局认证信息协议类型，如果某个上级没有单独设置认证信息协议类型就使用全局设置的认证信息协议类型。  
认证信息协议类型和上级写在一起。  
格式是: `http://a:b@2.2.2.2:33080#1`  
说明: 
`http://` 是协议类型，可能的类型以及含有如下:

```text
http  等价于 -S http -T tcp  
https  等价于 -S http -T tls --parent-tls-single ， 也就是http(s)代理 over TLS  
https2 等价于 -S http -T tls  
socks5  等价于 -S socks -T tcp  
socks5s 等价于 -S socks -T tls --parent-tls-single ， 也就是socks over TLS  
socks5s2 等价于 -S socks -T tls  
ss     等价于 -S ss -T tcp  
httpws    等价于  -S http -T ws
httpwss   等价于  -S http -T wss
socks5ws  等价于  -S socks -T ws
socks5wss 等价于  -S socks -T wss
```

### 6.8 自定义加密
proxy的sps代理在tcp之上可以通过tls标准加密以及kcp协议加密tcp数据，除此之外还支持在tls和kcp之后进行  
自定义加密，也就是说自定义加密和tls|kcp是可以联合使用的，内部采用AES256加密，使用的时候只需要自己定义  
一个密码即可，加密分为两个部分，一部分是本地(-z)是否加密解密，一部分是与上级(-Z)传输是否加密解密。  

自定义加密要求两端都是proxy才可以。  

下面分别用二级，三级为例:  

假设已经存在一个http(s)代理:`6.6.6.6:6666`  

二级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy sps -S http -T tcp -P 6.6.6.6:6666 -t tcp -z demo_password -p :7777`  
本地二级执行:  
`proxy sps -T tcp -P 2.2.2.2:777 -Z demo_password -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级加密传输访问目标网站。  


三级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy sps -S http -T tcp -P 6.6.6.6:6666 -t tcp -z demo_password -p :7777`  
二级vps(ip:3.3.3.3)上执行:  
`proxy sps -T tcp -P 2.2.2.2:7777 -Z demo_password -t tcp -z other_password -p :8888` 
本地三级执行:  
`proxy sps -T tcp -P 3.3.3.3:8888 -Z other_password -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级加密传输访问目标网站。  

### 6.9 压缩传输
proxy的sps代理在tcp之上可以通过自定义加密和tls标准加密以及kcp协议加密tcp数据，在自定义加密之前还可以  
对数据进行压缩，也就是说压缩功能和自定义加密和tls|kcp是可以联合使用的，压缩分为两个部分，   
一部分是本地(-m)是否压缩传输，一部分是与上级(-M)传输是否压缩。  

压缩要求两端都是proxy才可以，压缩也在一定程度上保护了(加密)数据。  

下面分别用二级，三级为例:  

二级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy sps -t tcp -m -p :7777`  
本地二级执行:  
`proxy sps -T tcp -P 2.2.2.2:777 -M -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级压缩传输访问目标网站。  


三级实例

一级vps(ip:2.2.2.2)上执行:  
`proxy sps -t tcp -m -p :7777`  
二级vps(ip:3.3.3.3)上执行:  
`proxy sps -T tcp -P 2.2.2.2:7777 -M -t tcp -m -p :8888` 
本地三级执行:  
`proxy sps -T tcp -P 3.3.3.3:8888 -M -t tcp -p :8080`  
这样通过本地代理8080访问网站的时候就是通过与上级压缩传输访问目标网站。  

### 6.10 禁用协议
SPS默认情况下一个端口支持http(s)和socks5两种代理协议，我们可以通过参数禁用某个协议  
比如:  
1.禁用HTTP(S)代理功能只保留SOCKS5代理功能，参数:`--disable-http`。  
`proxy sps -T tcp -P 3.3.3.3:8888 -M -t tcp -p :8080 --disable-http`   

1.禁用SOCKS5代理功能只保留HTTP(S)代理功能，参数:`--disable-socks`。  
`proxy sps -T tcp -P 3.3.3.3:8888 -M -t tcp -p :8080 --disable-http`   

### 6.11 限速

假设存在SOCKS5上级:

`proxy socks -p 2.2.2.2:33080 -z password -t tcp`

sps下级，限速100K

`proxy sps -S socks -P 2.2.2.2:33080 -T tcp -Z password -l 100K -t tcp -p :33080`

通过`-l`参数即可指定，比如:100K 2000K 1M . 0意味着无限制。  

### 6.12 指定出口IP

`--bind-listen`参数，就可以开启客户端用入口IP连接过来的，就用入口IP作为出口IP访问目标网站的功能。如果入口IP是内网IP，出口IP不会使用入口IP。

`proxy sps -S socks -P 2.2.2.2:33080 -T tcp -Z password -l 100K -t tcp --bind-listen -p :33080`

### 6.13 证书参数使用base64数据

默认情况下-C，-K参数是crt证书和key文件的路径，

如果是base64://开头，那么就认为后面的数据是base64编码的，会解码后使用。  

### 6.14 独立服务
sps功能不强制指定一个上级，当上级为空，sps本身即可完成完整的代理功能.如果指定了上级那么就和之前一样使用上级连接目标。  
下面这个命令，就是一键开启http(s)\ss\socks服务。  
`./proxy sps -p :33080`  

### 6.15 目标重定向
sps功能提供的http(s)\socks5\ss代理功能，客户端通过sps代理去连接指定的“目标”，这个“目标”一般是网站也可能是任意的tcp地址，
网站”目标“一般是foo.com:80,foo.com:443，sps支持使用--rewrite参数指定一个“目标”重定向规则文件，对目标进行重定向，客户端是无感知的，
比如你对“目标”：demo.com:80重定向到192.168.0.12:80，那么客户端访问网站demo.com，其实访问的是192.168.0.12提供的网站服务。
“目标”重定向规则文件示例：

```text
# example
www.a.com:80     10.0.0.2:8080
**.b.com:80      10.0.0.2:80
192.168.0.11:80  10.0.0.2:8080
```

### 6.16 查看帮助

`./proxy help sps`  

## 7.KCP配置 

### 7.1 配置介绍 
proxy的很多功能都支持kcp协议，凡是使用了kcp协议的功能都支持这里介绍的配置参数。  
所以这里统一对KCP配置参数进行介绍。  

### 7.2 详细配置 
所有的KCP配置参数共有17个，你可以都不用设置，他们都有默认值，如果为了或者最好的效果，  
就需要自己根据自己根据网络情况对参数进行配置。由于kcp配置很复杂需要一定的网络基础知识，  
如果想获得kcp参数更详细的配置和解说，请自行搜索。每个参数的命令行名称以及默认值和简单的功能说明如下：  
```
--kcp-key="secrect"        pre-shared secret between client and server
--kcp-method="aes"         encrypt/decrypt method， can be: aes， aes-128， aes-192， salsa20， blowfish， 
                           twofish， cast5， 3des， tea， xtea， xor， sm4， none
--kcp-mode="fast"       profiles: fast3， fast2， fast， normal， manual
--kcp-mtu=1350             set maximum transmission unit for UDP packets
--kcp-sndwnd=1024          set send window size(num of packets)
--kcp-rcvwnd=1024          set receive window size(num of packets)
--kcp-ds=10                set reed-solomon erasure coding - datashard
--kcp-ps=3                 set reed-solomon erasure coding - parityshard
--kcp-dscp=0               set DSCP(6bit)
--kcp-nocomp               disable compression
--kcp-acknodelay           be carefull! flush ack immediately when a packet is received
--kcp-nodelay=0            be carefull!
--kcp-interval=50          be carefull!
--kcp-resend=0             be carefull!
--kcp-nc=0                 be carefull! no congestion
--kcp-sockbuf=4194304      be carefull!
--kcp-keepalive=10         be carefull!
```
提示：  
参数：--kcp-mode中的四种fast3， fast2， fast， normal模式，  
相当于设置了下面四个参数：  
normal：`--nodelay=0 --interval=40 --resend=2 --nc=1`  
fast ：`--nodelay=0 --interval=30 --resend=2 --nc=1`  
fast2：`--nodelay=1 --interval=20 --resend=2 --nc=1`  
fast3：`--nodelay=1 --interval=10 --resend=2 --nc=1`  

## 8.安全DNS 

### 8.1 介绍
众所周知DNS是UDP端口53提供的服务，但是随着网络的发展一些知名DNS服务器也支持TCP方式dns查询，比如谷歌的8.8.8.8，proxy的DNS防污染服务器原理就是在本地启动一个proxy的DNS代理服务器，它用TCP的方式通过上级代理进行dns查询。如果它和上级代理通讯采用加密的方式，那么就可以进行安全无污染的DNS解析，还支持独立服务，并发解析，支持增强的hosts文件功能支持灵活的并发解析转发。 

dns解析顺序：  
1.使用参数--hosts解析。   
2.要解析的域名在1中没有找到，就使用参数--forward规则解析。  
3.要解析的域名在1和2中都没有找到，就使用默认--default解析，--default默认行为参数值有三种：proxy，direct，system。  
    三种参数值解释如下：  
    proxy：通过上级去连接-q参数指定的dns服务器去解析域名。  
    direct：通过本地网络去连接-q参数指定的dns服务器去解析域名。  
    system：通过系统dns去解析域名。  

提示：  
--hosts    参数指定的host文件格式和系统hosts文件一致，而且域名支持通配符，可以参考hosts文件。  
--forward  参数指定的解析转发规则文件，格式可以参考resolve.rules文件，域名支持通配符，支持为每个域名指定多个dns服务器并发解析，谁最快解析成功就用谁的解析结果。    
-q         参数可以指定多个远程dns服务器，执行并发解析谁最快解析成功就用谁的解析结果，默认是：1.1.1.1，8.8.8.8，9.9.9.9，多个用逗号分割，  
           比如还可以带端口：1.1.1.1，8.8.8.8#53，9.9.9.9  

如果独立服务，不需要上级：
可以执行：  
`proxy dns --default system -p :5353`  
or  
`proxy dns --default direct -p :5353`  

### 8.2 使用示例

#### 8.2.1 普通HTTP(S)上级代理

假设有一个上级代理：2.2.2.2:33080  
本地执行：  
`proxy dns -S http -T tcp -P 2.2.2.2:33080 -p :53`  
那么本地的UDP端口53就提供了DNS解析功能。  

#### 8.2.2 普通SOCKS5上级代理

假设有一个上级代理：2.2.2.2:33080  
本地执行：  
`proxy dns -S socks -T tcp -P 2.2.2.2:33080 -p :53`  
那么本地的UDP端口53就提供了DNS解析功能。 

#### 8.2.3 TLS加密的HTTP(S)上级代理

假设有一个上级代理：2.2.2.2:33080  
上级代理执行的命令是：
`proxy http -t tls -C proxy.crt -K proxy.key -p :33080`
本地执行：  
`proxy dns -S http -T tls -P 2.2.2.2:33080  -C proxy.crt -K proxy.key -p :53`  
那么本地的UDP端口53就提供了安全防污染DNS解析功能。 

#### 8.2.4 TLS加密的SOCKS5上级代理

假设有一个上级代理：2.2.2.2:33080  
上级代理执行的命令是：
`proxy socks -t tls -C proxy.crt -K proxy.key -p :33080`
本地执行：  
`proxy dns -S socks -T tls -P 2.2.2.2:33080  -C proxy.crt -K proxy.key -p :53`  
那么本地的UDP端口53就提供了安全防污染DNS解析功能。  

#### 8.2.5 KCP加密的HTTP(S)上级代理

假设有一个上级代理：2.2.2.2:33080  
上级代理执行的命令是：
`proxy http -t kcp -p :33080`
本地执行：  
`proxy dns -S http -T kcp -P 2.2.2.2:33080 -p :53`  
那么本地的UDP端口53就提供了安全防污染DNS解析功能。 

#### 8.2.6 KCP加密的SOCKS5上级代理

假设有一个上级代理：2.2.2.2:33080  
上级代理执行的命令是：
`proxy socks -t kcp -p :33080`
本地执行：  
`proxy dns -S socks -T kcp -P 2.2.2.2:33080 -p :53`  
那么本地的UDP端口53就提供了安全防污染DNS解析功能。 

#### 8.2.7 自定义加密的HTTP(S)上级代理

假设有一个上级代理：2.2.2.2:33080  
上级代理执行的命令是：
`proxy http -t tcp -p :33080 -z password`
本地执行：  
`proxy dns -S http -T tcp -Z password -P 2.2.2.2:33080 -p :53`  
那么本地的UDP端口53就提供了安全防污染DNS解析功能。 

#### 8.2.8 自定义加密的SOCKS5上级代理
 
假设有一个上级代理：2.2.2.2:33080  
上级代理执行的命令是：
`proxy socks -t kcp -p :33080 -z password`
本地执行：  
`proxy dns -S socks -T tcp -Z password -P 2.2.2.2:33080 -p :53`  
那么本地的UDP端口53就提供了安全防污染DNS解析功能。

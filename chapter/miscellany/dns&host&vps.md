# DNS、Host、VPN和科学上网

## DNS

域名系统，用来把机器名字转换成IP地址。举个栗子：DNS就是把`www.google.com`这个域名解析成对应的IP地址`8.8.8.8`

## Host

在互联网的早期，网络只有几台电脑。人们用hosts文件记录机器名字到IP的映射,随着网络的扩大Host映射已经不能满足需求，所以发明了DNS域名系统。但是hosts文件仍然保留在操作系统中，hosts文件的优先级高于DNS查询。操作系统首先会在hosts文件中找域名对应的IP地址，没有找到它才会去问DNS服务器。

## VPN

VPN （Virtual Private Network），虚拟专用网。举个栗子：如果一名公司员工在外出差住宾馆，如果需要到公司内网查资料这个时候使用的是宾馆的公共网络，数据传输途中要经过酒店的路由，中途任何有技术的人都可以看到你跟内部网络之间的明文通信。但是现在需要访问公司内部网络，为了保障公司内部数据不外泄，需要建立一条加密的专用信道。发送数据的时候加密，接收数据的时候解密，加解密的方式事先设定好。这样第三方看到加密过后的数据也无法理解其中的含义。

## 科学上网

我们的ZF发明了伟大的GFW，于此对应我们伟大的程序员发明了用VPN科学上谷歌的方法。如果你明文请求Google主机，GFW会直接重置连接。但是你在国外有个VPN代理服务器，代理服务器帮你请求Google；再把Google的响应用加密的方式转发给你。因为你跟VPN代理之间是加密传输，GFW不知道你访问的是Google，它不可能把所有发到国外的请求都重置掉，所以成功突破封锁。

### SS搭建

1. `wget –no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh`

2. `chmod +x shadowsocks.sh`

3. `./shadowsocks.sh 2>&1 | tee shadowsocks.log`

> 稍等片刻之后，按照提示依次输入密码-确认密码-输入端口号-确认端口号，选择加密方式-回车确认。加密方式我们一定要选择【7】，如果你的苹果设备想实现科学上网的话。


用cat查看下配置文件即可看见密码了

`cat /etc/shadowsocks.json`

### ss常用命令

启动：service shadowsocks start

停止：service shadowsocks stop

重启：service shadowsocks restart

状态：service shadowsocks status

### SS客户端下载

[shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows)
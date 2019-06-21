# rsync+ssh 同步数据

背景：为了方便本地编译的静态资源同步到测试环境的服务器上，简化手动同步操作流程，使用了 rsync 这个 linux 系统下的数据同步工具。

## rsync

### 常用参数

常用参数组合`-avz | -rltpD`

特殊参数 `--chmod=a+rwx` 或者 `--chmod=777` 解决向服务器推送文件时，服务器的文件权限问题。
举个栗子：`rsync -p --chmod=777`

| 选项                 | 说明                                                     |
| -------------------- | -------------------------------------------------------- |
| -r, ––recursive      | 对子目录以递归模式处理                                   |
| -l, ––links          | 保持软链接                                               |
| -p, ––perms          | 保持文件权限                                             |
| -t, ––times          | 保持文件时间信息                                         |
| -g, ––group          | 保持文件属组信息                                         |
| -o, ––owner          | 保持文件属主信息 (super-user only)                       |
| -D                   | 保持设备文件和特殊文件 (super-user only)                 |
| -P                   | 显示同步过程及进度等信息等同于 -partial -progress        |
| –a                   | 归档相当于 -rlptgoDP                                     |
| -e                   | 使用的信道协议，如 -e 'ssh -p 22'                        |
| -z, ––compress       | 在传输文件时进行压缩处理                                 |
| -v, ––verbose        | 详细输出模式                                             |
| -q, ––quiet          | 精简输出模式                                             |
| ––exclude=PATTERN    | 指定排除一个不需要传输的文件匹配模式                     |
| ––exclude-from=FILE  | 从 FILE 中读取排除规则                                   |
| ––delete             | 删除那些接收端还有而发送端已经不存在的文件               |
| ––delete-before      | 接收者在传输之前进行删除操作 (默认)                      |
| ––password-file=FILE | --daemon 模式 从 FILE 中读取口令，以避免在终端上输入口令 |
| ––partial            | 保留那些因故没有完全传输的文件，以是加快随后的再次传输   |
| ––progress           | 在传输时显示传输过程                                     |
| -H, --hard-links     | 保持硬链接文件                                           |

## rsync 传输模式主要有三种：

### 本地传输：也就是说本地的一个文件目录拷贝到另一个文件夹下类似于 cp

```js
rsync [OPTION...] SRC... [DEST]
// 举个例子 意思是把当前文件夹下的dist文件夹下的内容同步到backup/test文件下下面
rsync -r ./dist/ ./backup/test
```

### 通过远程 Shell 使用（借助 rcp, ssh 等通道来传输数据）

```js
拉: rsync [OPTION]... [USER@]HOST:SRC [DEST]
推: rsync [OPTION]... SRC [USER@]HOST:DEST

//举个栗子：拉取远端服务器上的backup目录到本地dist目录下
//rsync -r username@host:/backup/ ./dist/

//同理推送就是本地dist目录同步到远端/backup目录下
//rsync -r ./dist/ username@host:/backup/
```

### 守护进程方式(socket)，需要配置服务端与客户端

#### 服务端

rsync 安装

```js
# rpm -qa|grep rsync  #检查是否安装过rsync
# yum install rsync   #如果未安装，使用yum安装rsync
```

配置文件：`/etc/rsyncd.conf`

```bash
uid = rsync                        #用户id
gid = rsync
use chroot = no                    #安全性，内网一般不考虑，设为no
max connections = 200              #最多有多少个客户端连接我
timeout = 300                      #超时时间，秒
pid file = /var/run/rsyncd.pid     #pid文件
lock file = /var/run/rsync.lock    #传输时会给文件加锁
log file = /var/log/rsyncd.log     #日志文件
[test]                             #模块
path = /test/                      #客户端来同步，就是同步该目录
ignore errors                      #传输过程中遇到错误，自动忽略
read only = false                  #可读可写
list = false                       #不允许列表
hosts allow = 10.0.0.0/24          #允许的IP段
hosts deny = 0.0.0.0/32            #拒绝
auth users = rsync_backup          #这是个虚拟用户
secrets file = /etc/rsync.password #虚拟用户对应的密码文件
```

创建系统用户 rsync 用来启动服务

```bash
useradd rsync -s /sbin/nologin
授权
chown -R rsync.rsync /test
```

在/etc/rsync.password 中添加虚拟用户

`echo "rsync_backup:123456" > /etc/rsync.password`

设置权限密码文件权限

`chmod 600 /etc/rsync.password`

启动服务端, 以守护进程方式

`rsync --daemon` 端口： tcp 873

添加开机启动

`echo "/usr/bin/rsync --daemon" >> /etc/rc.local`

#### 客户端

客户端只需要密码文件,文件里只存放密码：

`echo "123456" > /etc/rsync.password`

设置文件权限：

`chmod 600 /etc/rsync.password`

往服务端推送文件：

`rsync -avz /data/ rsync_backup@10.0.0.1::/test --password-file=/etc/rsync.password`

从服务端拉取文件：

`rsync -avz rsync_backup@10.0.0.7::/test /data --password-file=/etc/rsync.password`

排除指定的文件和目录

`rsync -avz /data/ rsync_backup@10.0.0.1::/test --password-file=/etc/rsync.password --exclude=".DS_Store"`

## 基于 ssh 的 rsync 的实现

### 配置 ssh

- 生成 sshKey
  - windows 平台`C:\Users\用户名\.ssh`执行`ssh-keygen`
  - Mac 平台 `cd ~/.ssh`执行`ssh-keygen`

### 配置 config

由于本地可能需要配置多个 ssh key，使得不同的 host 能使用不同的 ssh key。

举个栗子：

```bash
# github
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_rsa_github
    PreferredAuthentications publickey
    User malcn
# gitlab
Host gitlab.com
    HostName gitlab.com
    IdentityFile ~/.ssh/id_rsa_gitlab
    PreferredAuthentications publickey

```

### 配置服务器

刚才我们生成了一个 ssh-key,我们需要把对应生成的公钥拷贝到服务器

- 拷贝公钥信息`cat ~/.ssh/id_rsa_rsync.pub`

- 进入服务器`.ssh`目录，新建 authorized_keys 文件(`vi authorized_keys`)，把对应的公钥信息添加到文件里，如果已存在该文件就追加信息进去。

### 大功告成，现在 ssh 通道已经搭建完毕

现在可以执行命令推送本地文件到服务器啦

`rsync -rltpD --chmod=a+rwx ./data/ host:/backup/`

### 不得不说的一点是 windows 系统并没有 rsync 我们需要借助 cwRsync 进 rsync 操作

- 下载 cwRsync 并安装

- 配置环境变量

- 命令行模式下执行`rsync` 查看是否配置成功

## 思考：

### ssh 工作原理是什么，它为什么能做到不需要用户输入密码就能与服务器端建立安全的会话通道

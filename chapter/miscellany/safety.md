# 安全

## 什么是XSS？

XSS（Cross Site Scripting），跨域脚本攻击。

### XSS攻击

XSS攻击就是攻击者通过各种办法，在用户访问的网页中插入自己的脚本，让其在用户访问网页时在其浏览器中进行执行。攻击者通过插入的脚本的执行，来获得用户的信息，比如cookie，发送到攻击者自己的网站(跨站了)。所以称为跨站脚本攻击。攻击类型主要有两种：反射型和存储型

### 反射型XSS

特征：

* 恶意代码并没有保存在目标网站，恶意脚本附加到 url 中。常见于诱导用户点击此链接才会引起攻击

危害：

* 窃取cookies，读取目标网站的cookie发送到黑客的服务器上

### 存储型XSS

* 恶意代码被非法存储到目标网站的数据库

* 常见于评论系统，留言

### 如何防范

* 转义
* HttpOnly
* 开启CSP网页安全政策防止XSS攻击 

#### 什么是CSP

CSP是网页安全政策(Content Security Policy)的缩写。是一种由开发者定义的安全性政策申明，通过CSP所约束的责任指定可信的内容来源，（内容可以是指脚本、图片、style 等远程资源）。通过CSP协定，可以防止XSS攻击，让web处一个安全运行的环境中。

CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。它的实现和执行全部由浏览器完成，开发者只需提供配置。CSP 大大增强了网页的安全性。攻击者即使发现了漏洞，也没法注入脚本，除非还控制了一台列入了白名单的可信主机。

开启方式:

1. 通过 HTTP 头信息的Content-Security-Policy的字段。

2. 在网页中设置`<meta>`标签，如：

    ``` bash
    <meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none'; style-src cdn.example.org third-party.org; child-src https:">
    ```

## CSRF攻击

CSRF（Cross Site Request Forgery），跨站点请求伪造。CSRF攻击者在用户已经登录目标网站之后，诱使用户访问一个攻击页面，利用目标网站对用户的信任，以用户身份在攻击页面对目标网站发起伪造用户操作的请求，达到攻击目的。

举个栗子：

如果微博有一个加关注的接口是用get方式后面接？加参数的形式实现加关注的话，我只需要在我的微博下发一篇博文，内容里面写一个img标签：
`<img style="width:0;" src="www.weibo.com?followUserGuid=1"   />`

那么只要有人打开我这篇博文，那就会自动关注我。

### 如何防御

* 尽量使用POST，限制GET，当然POST并不是万无一失，可以构造一个form表单提交，form可以跨域post数据

* 加验证码

* Referer Check

    > Referer Check在Web最常见的应用就是“防止图片盗链”。同理，Referer Check也可以被用于检查请求是否来自合法的“源”（Referer值是否是指定页面，或者网站的域），如果都不是，那么就极可能是CSRF攻击。但是因为服务器并不是什么时候都能取到Referer，所以也无法作为CSRF防御的主要手段。但是用Referer Check来监控CSRF攻击的发生，倒是一种可行的方法。

* Token校验
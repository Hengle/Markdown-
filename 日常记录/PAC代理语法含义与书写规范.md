
# PAC代理语法含义与书写规范
[TOC]

一直以来使用ShadowSocksFQ，基本上默认的PAC代理模式己能满足所需，实在个别pac不方便的就转成用全局代理模式也能愉快FQ。 

_只是最近学习前端的知识，需要FQ访问   
[MDN web docs](https://developer.mozilla.org/)这个网站来查看资料，深深感觉来回切换不方便，于是还是得研究一下怎么来个自定义的规则方便一些。_

## ShadowSocks PAC规则

ShadowSocks默认使用GFWList规则和使用adblock plus的引擎。要想自己添加自定义的用户规则，最好熟悉一下其规则：

中文版：[Adblock Plus过滤规则](https://adblockplus.org/zh_CN/filters)

自定义代理规则的设置语法与GFWlist相同，语法规则如下：

```hljs
1.  通配符支持。比如 `*.example.com/*` 实际书写时可省略 `*` ， 如`.example.com/` ， 和 `*.example.com/*` 效果一样
2.  正则表达式支持。以 `\` 开始和结束， 如 `\[\w]+:\/\/example.com\`
3.  例外规则 `@@` ，如 `@@*.example.com/*` 满足 `@@` 后规则的地址不使用代理
4.  匹配地址开始和结尾 `|` ，如 `|http://example.com` 、 `example.com|` 分别表示以 `http://example.com` 开始和以 `example.com` 结束的地址
5.  `||` 标记，如 `||example.com` 则 `http://example.com` 、`https://example.com` 、 `ftp://example.com` 等地址均满足条件
6.  注释 `!` 。 如 `!我是注释`
7.  分隔符^，表示除了字母、数字或者 _ - . % 之外的任何字符。如 `http://example.com^` ，`http://example.com/` 和 `http://example.com:8000/` 均满足条件，而 `http://example.com.ar/` 不满足条件
```

## 什么是PAC

维基百科摘录的关于PAC的解释：

代理自动配置（英语：Proxy auto-config，简称PAC）是一种网页浏览器技术，用于定义浏览器该如何自动选择适当的代理服务器来访问一个网址。

一个PAC文件包含一个JavaScript形式的函数“FindProxyForURL(url, host)”。这个函数返回一个包含一个或多个访问规则的字符串。用户代理根据这些规则适用一个特定的代理其或者直接访问。当一个代理服务器无法响应的时候，多个访问规则提供了其他的后备访问方法。浏览器在访问其他页面以前，首先访问这个PAC文件。PAC文件中的URL可能是手工配置的，也可能是是通过网页的网络代理自发现协议（Web Proxy Autodiscovery Protocol）自动配置的。

源自网络的图解：   
![](https://www.tielemao.com/wp-content/uploads/2018/05/PAC.png)

简单说来，PAC就是一种配置规则，它能让你的浏览器智能判断哪些网站走代理，哪些不需要走代理。shadowsocks 目录下有一个 pac.txt 文件，而pac.txt这个文件是可以使用在线PAC或通过本地的GWFlist去更新的。所以并不建议用户自定义的规则直接加在pac.txt上面。

![](https://www.tielemao.com/wp-content/uploads/2018/05/ShadowSocks_PAC.jpg)

打开 pac.txt 文件，可以看到头部是如下内容：

![](https://www.tielemao.com/wp-content/uploads/2018/05/ShadowSocks_PAC01.jpg)

可以看出pac配置文件使用的是JavaScript语法，里面定义了一个变量rules，是一个JSon数组格式的数据类型，数组里面存放的是各种URL的通配符，那么在pac模式下，如果当访问符合这个数组里面任意一个URL通配符的网址时，系统会走代理，反之直连。比如访问谷歌搜索首页时，会走代理，而访问百度、新浪等国内网站则会选择直连方式。

PAC，一个自动代理配置脚本，包含了很多使用 JavaScript 编写的规则，它能够决定网络流量走默认通道还是代理服务器通道，控制的流量类型包括：HTTP、HTTPS 和 FTP。

一段 JavaScript 脚本：

```language-js
function FindProxyForURL(url, host) {
  return "DIRECT";
}
```

上面就是一个最简洁的 PAC 文件，意思是所有流量都直接进入互联网，不走代理。

## PAC的优势

PAC自动代理属于智能判断模式，相比全局代理，它的优点有：

1.  不影响国内网站的访问速度，防止无意义的绕路；
2.  节省Shadowsocks服务的流量，节省服务器资源；
3.  控制方便。
4.  自动容灾。

## PAC 语法和函数

PAC不但使用在ShadowSocks上很方便，实际上它也可以配合其它代理服务端（如Squid），运用在浏览器上，不过需要你去弄懂它的语法和函数。

url 字段表示浏览器地址栏输入的待访问地址，host 为该地址对应的 hostname，return 语句有三种指令：

* DIRECT，表示无代理直接连接
* PROXY host:port，表示走host:port 的 proxy 服务
* SOCKS host:port，表示走host:port 的 socks 服务

而返回的接口可以是多个代理串联：

```language-js
return "PROXY 222.20.74.89:8800; SOCKS 222.20.74.89:8899; DIRECT";
```

上面代理的意思是，默认走222.20.74.89:8800 的 proxy 服务；如果代理挂了或者超时，则走 222.20.74.89:8899的 socks 代理；如果 socks 也挂了，则无代理直接连接。从这里可以看出 PAC 的一大优势：自动容灾。

PAC 提供了几个内置的函数，下面一一介绍下：

**dnsDomainIs**

类似于 ==，但是对大小写不敏感，

```language-text
if (dnsDomainIs(host, "google.com") ||
    dnsDomainIs(host, "www.google.com")) {
  return "DIRECT";
}
```

**shExpMatch**

```hljs
Shell 正则匹配，* 匹配用的比较多，可以是 *.http://example.com，也可以是下面这样：
```

* 1

```language-js
if (shExpMatch(host, "vpn.domain.com") ||
    shExpMatch(url, "http://abcdomain.com/folder/*")) {
  return "DIRECT";
}
```

**isInNet**

判断是否在网段内容，比如 10.1.0.0 这个网段，10.1.1.0 就在网段中，

```language-js
if (isInNet(dnsResolve(host), "172.16.0.0", "255.240.0.0")) {
  return "DIRECT";
}
```

**myIpAddress**

返回主机的 IP，

```language-text
if (isInNet(myIpAddress(), "10.10.1.0", "255.255.255.0")) {
  return "PROXY 10.10.5.1:8080";
}
```

**dnsResolve**

通过 DNS 查询主机 ip，

```language-js
if (isInNet(dnsResolve(host), "10.0.0.0", "255.0.0.0") ||
    isInNet(dnsResolve(host), "172.16.0.0",  "255.240.0.0") ||
    isInNet(dnsResolve(host), "192.168.0.0", "255.255.0.0") ||
    isInNet(dnsResolve(host), "127.0.0.0", "255.255.255.0")) {
  return "DIRECT";
}
```

**isPlainHostName**

判断是否为诸如barret/，server-name/ 这样的主机名。

```language-js
if (isPlainHostName(host)) {
  return "DIRECT";
}
```

**isResolvable**

判断主机是否可访问。

```language-js
if (isResolvable(host)) {
  return "PROXY proxy1.example.com:8080";
}
```

**dnsDomainLevels**

返回是几级域名，比如 dnsDomainLevels 返回的结果就是 1。

```language-js
if (dnsDomainLevels(host) > 0) {
  return "PROXY proxy1.example.com:8080";
} else {
  return "DIRECT";
}
```

**weekdayRange**

周一到周五

```language-js
if (weekdayRange("MON", "FRI")) {
  return "PROXY proxy1.example.com:8080";
} else {
  return "DIRECT";
}
```

**dateRange**

一月到五月

```language-js
if (dateRange("JAN", "MAR"))  {
  return "PROXY proxy1.example.com:8080";  
} else {
  return "DIRECT";
}
```

**timeRange**

八点到十八点，

```language-js
if (timeRange(8, 18)) {
  return "PROXY proxy1.example.com:8080";
} else {
  return "DIRECT";  
}
```

**alert**

这个弹窗警报信息函数可以用来配合浏览器的控制台进行调试

```language-js
resolved_host = dnsResolve(host);
alert(resolved_host);
```

## PAC 文件的安装和注意事项

在 Windows 系统中，通过「Internet选项 -> 连接 -> 局域网设置 -> 使用自动配置脚本」可以找到配置处，下方的地址栏填写 PAC 文件的 URI，这个 URI 可以是本地资源路径(file:///)，也可以是网络资源路径。

Chrome 中可以在「chrome://settings/ -> 显示高级设置 -> 更改代理服务器设置」中找到 PAC 填写地址。

**需要注意的几点：**

* PAC 文件被访问时，返回的文件类型（Content-Type）应该为：application/x-ns-proxy-autoconfig，当然，如果你不写，一般浏览器也能够自动辨别。
* FindProxyByUrl(url, host) 中的 host 在上述函数对比时无需转换成小写，对大小写不敏感。
* 没必要对 dnsResolve(host) 的结果做缓存，DNS 在解析的时候会将结果缓存到系统中。

## PAC文件及user-rule文件的语法规则

当一个网站被墙，如何添加到PAC里面让其能够正常访问呢？以MDN web doc这个网站为例，在Shadowsocks里面，可以有如下两个方式：

### 1. 添加到 `pac.txt` 文件中

编辑 `pac.txt` 文件，模仿里面的一些URL通配符，在rules（中括号）列表中再添加一个，例如`"||developer.mozilla.org^",` ，注意不要忘记了 `,` 半角逗号，当然如果你是在最后添加的就可以不加半角逗号。这样配置下来就是所有 `developer.mozilla.org`域名下的网址都将走Shadowsocks代理。   
![](https://www.tielemao.com/wp-content/uploads/2018/05/ShadowSocks_PAC02.jpg)

### 2. 添加到 `user-rule.txt` 文件中（推荐）

编辑 `user-rule.txt` 文件，这里和 `pac.txt` 文件语法不完全相同，user-rule文件中，每一行表示一个URL通配符，但是通配符语法类似。例如添加一行`||developer.mozilla.org^` ，然后记得右键shadowsocks小飞机图标-PAC-从GFWList更新本地PAC。

推荐使用添加到user-rule.txt的这种自定义用户规则的方法，因为pac文件里的规则是有可能被在线更新掉的。
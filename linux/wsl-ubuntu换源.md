# 说明
被wsl下Ubuntu的apt-get update折腾惨了

## 原因
最开始说是dns的问题，就加了谷歌的dns
```
sudo vi /etc/resolv.conf
# 在文件里增加如下dns
nameserver 8.8.8.8
```

但是这样的增加在关闭windows terminal后又失效了。搜索许久发现有的要改“/etc/systemd/resolved.conf”，有的要改“/etc/NetworkManager/NetworkManager.conf”

中间弄的有点懵逼

参考这篇文章：[Ubuntu下修改DNS重启也能用的方法](https://www.yubosun.com/article/k82r066e.html)，参考的是第二条方法。但是关掉客户端后，必须调用下“sudo resolvconf -u”命令。
```
sudo vi /etc/resolvconf/resolv.conf.d/base
# 这是一个新文件，直接加如下内容
nameserver 8.8.8.8
nameserver 114.114.114.114
# 保存后，刷新
sudo resolvconf -u
# 再打开后就能看到新加的
sudo vi /etc/resolv.conf
```

## 换源
```
sudo vi /etc/apt/sources.list
# 内容

deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

不知道为啥，这个开始能用。现在也不行了。。。名称也需要对应上。focal

```
sudo apt upgrade
# 这个不行
sudo apt-get update
```

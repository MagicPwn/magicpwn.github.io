---
layout: post
title: Ubuntu 安装metasploit
description: metasploit 是常用的渗透测试利器，那么如何在Ubuntu上安装呢？
category: blog
---
# 安装msf

## 打开终端，进入安装目录
```
cd /opt

curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall

chmod 755 msfinstall

./msfinstall
```
等它自动安装完毕，然后先不要启动，目前最新版的这个msf会问你要不要用自带到database，你想用自带到数据库就回车，想用自己到数据库就输入no回车

# 安装postgresql
 ```
apt-get install postgresql  

su - postgres                      

psql                                   

\password 123456
```
# 可以开始使用啦
```
~#msfconsole
msf>
```

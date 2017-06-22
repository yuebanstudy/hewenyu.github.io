---
title: 基于itchat实现微信发送信息
date: 2017-06-21 20:54:16
tags: 
- python
categories: 
- python
comments: true
---

## 前景提要
* 官方介绍 ：http://itchat.readthedocs.io/zh/latest/
* python 环境支持 2.7 , 3.5

本人环境使用的是python 3.5 版本的，其他版本的请适当修改代码

* 创建一个登录脚本，维持稳定在线

<!--more-->

![](http://ww1.sinaimg.cn/large/87028554gy1fgt5p82hd1j20fb046dft.jpg)



```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
import itchat
itchat.auto_login(hotReload=True, enableCmdQR=True)
itchat.run()
```



```shell
# chmod +x login.py
# ./login.py
```

![](http://ww1.sinaimg.cn/large/87028554gy1fgt5p87dzpj20jd07m3yl.jpg)



* 通过手机扫描二维码即可登录

![](http://ww1.sinaimg.cn/large/87028554gy1fgt5p80y2dj20gj06jmx4.jpg)


*  退出程序ctrl +c


```shell
# nohup python3.5 login.py &
```

* 将程序挂起，就不需要继续扫码了，然后我们可以在这里找到一个itchat.pkl的文件

![](http://ww1.sinaimg.cn/large/87028554gy1fgt5p7xh9yj20ig015dfp.jpg)


* 这个就是我们需要调用的东西了，靠他可以发送微信消息了


```shell
# vi send_msg.py
```


```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
import itchat
msg = 'hello'
# 此处为itchat.pkl文件绝对路径
path_pkl = r'/usr/lib/zabbix/alertscripts/itchat.pkl'
itchat.auto_login(hotReload=True, statusStorageDir=path_pkl)
# 我是一个新创建的微信号，只拉了一个讨论组，所以就默认发送第一个讨论组
group = itchat.get_chatrooms(update=True)
itchat.send(msg=msg, toUserName=group[0]["UserName"])
```


```shell
# chmod +x send_msg.py
# ./send_msg.py
```
* 然后就可以在微信讨论组里看到了通过脚本控制发送的消息

![](http://ww1.sinaimg.cn/large/87028554gy1fgt5p82nhoj20c504na9w.jpg)


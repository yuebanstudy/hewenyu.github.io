---
title: Install Kibana with Debian Package
date: 2017-06-22 20:00:16
tags: 
- ELK
categories: 
- ELK
comments: true
---

## Kibana install


## 步骤
* 下载并安装公共签名密钥

```shell
# wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```


<!--more-->
### deb-repo

* 你可能需要在Debian安装apt-transport-https:

```shell
# apt-get install apt-transport-https
```

* 将 repository 保存到 /etc/apt/sources.list.d/elastic-5.x.list

```shell
# echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" |  tee -a /etc/apt/sources.list.d/elastic-5.x.list
```

* 你可以安装 Debian package Logstash,命令如下:

```shell
# apt-get update && apt-get install kibana
```


### deb-package


* 64
```shell
# wget https://artifacts.elastic.co/downloads/kibana/kibana-5.4.2-amd64.deb
# sha1sum kibana-5.4.2-amd64.deb
# dpkg -i kibana-5.4.2-amd64.deb
```

* 32
```shell
# wget https://artifacts.elastic.co/downloads/kibana/kibana-5.4.2-i386.deb
# sha1sum kibana-5.4.2-i386.deb
# dpkg -i kibana-5.4.2-i386.deb
```


* 通过输入命令来查询logstash安装在哪里了

```shell
# whereis kibana

```


* 获得输出结果
```shell
kibana: /etc/kibana /usr/share/kibana
```



* 配置kibana随系统启动，运行下面的命令:

```
# /bin/systemctl daemon-reload
# /bin/systemctl enable kibana.service
```

* kibanah可以按照如下方式启动和停止:

```
# systemctl start kibana.service
# systemctl stop kibana.service
```


* localhost:5601 or http://YOURDOMAIN.com:5601

![](http://ww1.sinaimg.cn/large/87028554gy1fgv1zp003oj21g30ongnm.jpg)
![](http://ww1.sinaimg.cn/large/87028554gy1fgv223s07gj21gf0pjwgv.jpg)


## 参考资料
* [kibana](https://www.elastic.co/guide/en/kibana/current/deb.html#deb)
* [connect-to-elasticsearch](https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html)
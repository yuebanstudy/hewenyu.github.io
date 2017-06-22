---
title: Logstash install
date: 2017-06-20 20:55:16
tags: 
- ELK
categories: 
- ELK
comments: true
---

## Logstash install

* Logstash 需要java环境  java 9 不支持，请自行安装

* 输入命令可以查询java版本

```shell
# java -version
```

* 可以获得如下信息

```shell
# java version "1.8.0_131"
# Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
# Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
```

<!--more-->

## 步骤
* 下载并安装公共签名密钥
 
```shell
# wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
 
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
# apt-get update && apt-get install logstash
```

* 通过输入命令来查询logstash安装在哪里了
```shell
# whereis logstash
```

* 获得输出结果
```shell
logstash: /etc/logstash /usr/share/logstash
```

* 测试logstash
```shell 
 # cd /usr/share/logstash
 # bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

* 具体运行结果如下
![](http://ww1.sinaimg.cn/large/87028554gy1fgtriugdntj211b05s0ta.jpg)
 
* 出现警告信息：是没有找到配置路径，把命令改一下

```shell 
./bin/logstash --path.settings /etc/logstash   -e 'input { stdin { } } output { stdout {} }'
```

* 具体运行结果如下

![](http://ww1.sinaimg.cn/large/87028554gy1fgtsl5y5rhj20t202ft8o.jpg)

## 参考资料
* 官网文档地址 ：[logstash](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html#package-repositories)



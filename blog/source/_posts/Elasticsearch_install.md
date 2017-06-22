---
title: Install Elasticsearch with Debian Package
date: 2017-06-21 20:55:16
tags: 
- ELK
categories: 
- ELK
comments: true
---

## Install Elasticsearch with Debian Package
* for Debian, Ubuntu, and other Debian-based systems
* 需要java环境,请自行准备



* 官方参考资料：https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html#deb-key

* 其他版本系统参考资料：https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html

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
# echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
```

* 你可以安装 Debian package Elasticsearch,命令如下:

```shell
# apt-get update && sudo apt-get install elasticsearch
```

* Elasticsearch v5.4.2 Debian 软件包可以从网站上下载并安装:

```shell
# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.4.2.deb
# sha1sum elasticsearch-5.4.2.deb
# sudo dpkg -i elasticsearch-5.4.2.deb
```

* 配置Elasticsearch随系统启动，运行下面的命令:
```shell
# /bin/systemctl daemon-reload
# /bin/systemctl enable elasticsearch.service
```

* Elasticsearch可以按照如下方式启动和停止:

```shell
# systemctl start elasticsearch.service
# systemctl stop elasticsearch.service
```


### 当系统启用了日志记录，日志记录的信息都可以用journalctl命令:

* To tail the journal:

```shell
# journalctl -f
```

* To list journal entries for the elasticsearch service:

```shell
# journalctl --unit elasticsearch
```

* To list journal entries for the elasticsearch service starting from a given time:

```shell
# journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"
```



* 我的机器是阿里云1c 1G的,启动的时候就报出如下错误

```shell
# Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x0000000085330000, 2060255232, 0) failed; error='Cannot alloc
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 2060255232 bytes for committing reserved memory.
```

* 根据机器配置不同可以修改相对应的参数

```shell
# vi /etc/elasticsearch/jvm.options
```

* 将 -Xms 和  -Xmx 参数修改一下就可以正常启动了
```shell
-Xms256m
-Xmx256m
```

* 默认参数是运行在9200端口下的,可以通过如下命令确认:
```shell
# curl -XGET 'localhost:9200/?pretty'
```

* 返回结果如下

```shell
{
  "name" : "0NcYzNP",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "-QmXkqg6T0uVrUAbNZhggA",
  "version" : {
    "number" : "5.4.2",
    "build_hash" : "929b078",
    "build_date" : "2017-06-15T02:29:28.122Z",
    "build_snapshot" : false,
    "lucene_version" : "6.5.1"
  },
  "tagline" : "You Know, for Search"
}
```


## 配置Elasticsearch

* Elasticsearch 的默认加载配置路径： /etc/elasticsearch/elasticsearch.yml . 

* 配置的详细解释可以参考官方文档：[elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)
 

* Debian package 也有一个配置文件 (/etc/default/elasticsearch), 允许你配置如下参数:

| 参数 | 注释 |
| --- | --- |
|ES_USER|The user to run as, defaults to elasticsearch|
|ES_GROUP|The group to run as, defaults to elasticsearch|
|JAVA_HOME|Set a custom Java path to be used|
|MAX_OPEN_FILES|Maximum number of open files, defaults to 65536|
|MAX_LOCKED_MEMORY|Maximum locked memory size. Set to unlimited if you use the bootstrap.memory_lock option in elasticsearch.yml|
|MAX_MAP_COUNT|Maximum number of memory map areas a process may have. If you use mmapfs as index store type, make sure this is set to a high value. For more information, check the linux kernel documentation about max_map_count. This is set via sysctl before starting elasticsearch. Defaults to 262144|
|LOG_DIR|Log directory, defaults to /var/log/elasticsearch|
|DATA_DIR|Data directory, defaults to /var/lib/elasticsearch|
|CONF_DIR|Configuration file directory (which needs to include elasticsearch.yml and log4j2.properties files), defaults to /etc/elasticsearch|
|ES_JAVA_OPTS|Any additional JVM system properties you may want to apply|
|RESTART_ON_UPGRADE|Configure restart on package upgrade, defaults to false. This means you will have to restart your elasticsearch instance after installing a package manually. The reason for this is to ensure, that upgrades in a cluster do not result in a continuous shard reallocation resulting in high network traffic and reducing the response times of your cluster|


* The Debian package places config files, logs, and the data directory in the appropriate locations for a Debian-based system 
https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html#deb-layout

| Type | Description | Default Location | Setting |
| --- | --- | --- | --- |
|home|Elasticsearch home directory or $ES_HOME|/usr/share/elasticsearch||
|bin|Binary scripts including elasticsearch to start a node and elasticsearch-plugin to install plugins|/usr/share/elasticsearch/bin||
|conf|Configuration files including elasticsearch.yml|/etc/elasticsearch|path.conf|
|conf|Environment variables including heap size, file descriptors|/etc/default/elasticsearch||
|data|The location of the data files of each index / shard allocated on the node. Can hold multiple locations.|/var/lib/elasticsearch|path.data|
|logs|Log files location.|/var/log/elasticsearch|path.logs|
|plugins|Plugin files location. Each plugin will be contained in a subdirectory.|/usr/share/elasticsearch/plugins||
|repo|Shared file system repository locations. Can hold multiple locations. A file system repository can be placed in to any subdirectory of any directory specified here.|Not configured|path.repo|
|script|Location of script files.|/etc/elasticsearch/scripts|path.scripts|

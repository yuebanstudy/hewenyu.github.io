---
title: Parsing Logs with Logstash
date: 2017-06-24 21:55:16
tags: 
- ELK
categories: 
- ELK
comments: true
---

## 步骤

* deb:
```shell
# curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.4.2-amd64.deb
# sudo dpkg -i filebeat-5.4.2-amd64.deb
```

* 安装后可以看到如下

    ![](http://ww1.sinaimg.cn/large/87028554gy1fguazvz6hdj20el01ut8o.jpg)


<!--more-->
##   Parsing Logs with Logstash

### filebeat config

* 在安装完成之后你需要配置filebeat，通过修改filebeat.yml，以nginx的日志为例


```shell
# cd /etc/filebeat
# vi filebeat.yml
```

* filebeat.yml

```shell
filebeat.prospectors:

# Each - is a prospector. Most options can be set at the prospector level, so
# you can use different prospectors for various configurations.
# Below are the prospector specific configurations.

- input_type: log

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
     - /var/log/nginx/*.log

#----------------------------- Logstash output --------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]
```



* 在数据源的机器上执行如下命令:
```shell
# cd /usr/share/filebeat/bin/
# ./filebeat -e -c /etc/filebeat/filebeat.yml -d "publish"
```

### logstash config

* 创建一个logstash的配置文件

```shell
# cd /etc/logstash/conf.d
# vi first-pipeline.conf
```

* first-pipeline.conf
```shell
input {
    beats {
        port => "5044"
    }
}
# The filter part of this file is commented out to indicate that it is
# optional.
# filter {
#
# }
output {
    stdout { codec => rubydebug }
}
```

* 确认logstash配置文件是否写错

```shell
# cd /usr/share/logstash/
# bin/logstash  --path.settings /etc/logstash  -f /etc/logstash/conf.d/first-pipeline.conf --config.test_and_exit
```

* 获取ok ( The --config.test_and_exit option parses your configuration file and reports any errors.)

```
Sending Logstash's logs to /var/log/logstash which is now configured via log4j2.properties
Configuration OK
```

* 运行logstash

```shell
# bin/logstash  --path.settings /etc/logstash  -f /etc/logstash/conf.d/first-pipeline.conf --config.reload.automatic
```

* 在filebeat开启的时候可以获得类似于如下的信息：
```
{
    "@timestamp" => 2017-06-22T14:58:32.169Z,
        "offset" => 13300,
      "@version" => "1",
    "input_type" => "log",
          "beat" => {
        "hostname" => "iZm5e7jlki70utmw22zj76Z",
            "name" => "iZm5e7jlki70utmw22zj76Z",
         "version" => "5.4.2"
    },
          "host" => "iZm5e7jlki70utmw22zj76Z",
        "source" => "/var/log/nginx/access.log",
       "message" => "115.61.84.162 - - [22/Jun/2017:22:06:06 +0800] \"GET http://open.163.com/ HTTP/1.1\" 200 612 \"http://open.163.com/\" \"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)\"",
          "type" => "log",
          "tags" => [
        [0] "beats_input_codec_plain_applied"
    ]
}
```

![](http://ww1.sinaimg.cn/large/87028554gy1fgudmcufgrj21700ldtaj.jpg)



* 修改first-pipeline.conf，使用grok

```
input {
    beats {
        port => "5044"
    }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}
output {
    stdout { codec => rubydebug }
}
```

* 删除日志记录的节点，我们可以重头读取日志

```
# rm /usr/share/filebeat/bin/data/registry
# bin/logstash  --path.settings /etc/logstash  -f /etc/logstash/conf.d/first-pipeline.conf --config.reload.automatic
```

* 可以看到我们获取到了的日志改变了，获取了更加详细的日志

![](http://ww1.sinaimg.cn/large/87028554gy1fguel0mr7jj218o0le0up.jpg)


* 启用goip

```
input {
    beats {
        port => "5044"
    }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
    geoip {
        source => "clientip"
    }
}
output {
    stdout { codec => rubydebug }
}
```

* 同样删除registry，重启程序，可以看到信息又更新了

![](http://ww1.sinaimg.cn/large/87028554gy1fgueuz7s9aj21820jmq4f.jpg)

### indexing data into elasticsearch

* 将first-pipeline.conf 再度修改，将output指向elasticsearch
```
input {
    beats {
        port => "5044"
    }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
    geoip {
        source => "clientip"
    }
}
#output {
#    stdout { codec => rubydebug }
#}
output {
    elasticsearch {
        hosts => [ "localhost:9200" ]
    }
}
```

* 同样删除registry，重启程序，然后再启动elasticsearch


```
# systemctl start elasticsearch.service
# curl -XGET 'http://localhost:9200/logstash-2017.06.23/_search?pretty&q=response=200'
# curl -XGET 'http://localhost:9200/logstash-2017.06.23/_search?pretty&q=geoip.city_name=Zhengzhou'
```


* 命令中logstash-DATE，替换 DATE 变成正确的时间, 格式如下 YYYY.MM.DD，通过请求，我们可以获得类似如下的数据

![](http://ww1.sinaimg.cn/large/87028554gy1fgv11tdt1tj20u70gojsa.jpg)

* 完整版

```
{
  "took" : 11,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 2.1282318,
    "hits" : [
      {
        "_index" : "logstash-2017.06.23",
        "_type" : "log",
        "_id" : "AVzTKaFXQUT1Lre_3sa9",
        "_score" : 2.1282318,
        "_source" : {
          "request" : "http://www.dajie.com/",
          "agent" : "\"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)\"",
          "geoip" : {
            "city_name" : "Zhengzhou",
            "timezone" : "Asia/Shanghai",
            "ip" : "115.61.84.162",
            "latitude" : 34.6836,
            "country_name" : "China",
            "country_code2" : "CN",
            "continent_code" : "AS",
            "country_code3" : "CN",
            "region_name" : "Henan",
            "location" : {
              "lon" : 113.5325,
              "lat" : 34.6836
            },
            "region_code" : "41",
            "longitude" : 113.5325
          },
          "offset" : 182,
          "auth" : "-",
          "ident" : "-",
          "input_type" : "log",
          "verb" : "GET",
          "source" : "/var/log/nginx/access.log",
          "message" : "115.61.84.162 - - [23/Jun/2017:06:50:10 +0800] \"GET http://www.dajie.com/ HTTP/1.1\" 200 612 \"http://www.dajie.com/\" \"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)\"",
          "type" : "log",
          "tags" : [
            "beats_input_codec_plain_applied"
          ],
          "referrer" : "\"http://www.dajie.com/\"",
          "@timestamp" : "2017-06-23T04:15:32.219Z",
          "response" : "200",
          "bytes" : "612",
          "clientip" : "115.61.84.162",
          "@version" : "1",
          "beat" : {
            "hostname" : "iZm5e7jlki70utmw22zj76Z",
            "name" : "iZm5e7jlki70utmw22zj76Z",
            "version" : "5.4.2"
          },
          "host" : "iZm5e7jlki70utmw22zj76Z",
          "httpversion" : "1.1",
          "timestamp" : "23/Jun/2017:06:50:10 +0800"
        }
      },
      {
        "_index" : "logstash-2017.06.23",
        "_type" : "log",
        "_id" : "AVzTOcnEQUT1Lre_3sbu",
        "_score" : 2.1282318,
        "_source" : {
          "request" : "http://www.dajie.com/",
          "agent" : "\"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)\"",
          "geoip" : {
            "city_name" : "Zhengzhou",
            "timezone" : "Asia/Shanghai",
            "ip" : "115.61.84.162",
            "latitude" : 34.6836,
            "country_name" : "China",
            "country_code2" : "CN",
            "continent_code" : "AS",
            "country_code3" : "CN",
            "region_name" : "Henan",
            "location" : {
              "lon" : 113.5325,
              "lat" : 34.6836
            },
            "region_code" : "41",
            "longitude" : 113.5325
          },
          "offset" : 182,
          "auth" : "-",
          "ident" : "-",
          "input_type" : "log",
          "verb" : "GET",
          "source" : "/var/log/nginx/access.log",
          "message" : "115.61.84.162 - - [23/Jun/2017:06:50:10 +0800] \"GET http://www.dajie.com/ HTTP/1.1\" 200 612 \"http://www.dajie.com/\" \"Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)\"",
          "type" : "log",
          "tags" : [
            "beats_input_codec_plain_applied"
          ],
          "referrer" : "\"http://www.dajie.com/\"",
          "@timestamp" : "2017-06-23T04:33:13.588Z",
          "response" : "200",
          "bytes" : "612",
          "clientip" : "115.61.84.162",
          "@version" : "1",
          "beat" : {
            "hostname" : "iZm5e7jlki70utmw22zj76Z",
            "name" : "iZm5e7jlki70utmw22zj76Z",
            "version" : "5.4.2"
          },
          "host" : "iZm5e7jlki70utmw22zj76Z",
          "httpversion" : "1.1",
          "timestamp" : "23/Jun/2017:06:50:10 +0800"
        }
      }
    ]
  }
}
```

* 去web上查看结果

![](http://ww1.sinaimg.cn/large/87028554gy1fgv26ygstoj21gu0pbgr5.jpg)

## 参考资料：

* 官方文档： [filebeat](https://www.elastic.co/guide/en/beats/filebeat/5.4/filebeat-getting-started.html)
* 官方文档： [logstash](https://www.elastic.co/guide/en/logstash/current/advanced-pipeline.html#testing-initial-pipeline)
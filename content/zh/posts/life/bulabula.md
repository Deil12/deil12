+++
title = "bulabula"
description = ""
date = "2023-11-14T17:08:56+08:00"
lastmod = "2023-11-14T17:08:56+08:00"
tags = ["日常"]
dropCap = false
displayCopyright = false
gitinfo = false
toc = false
+++

## 准备

- filebeat ~~+ logstash~~ + elasticsearch + kibana。(7.X开始就自带JDK)

------

- 下载

  elasticsearch:https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.6.0-linux-x86_64.tar.gz
  kibana:https://artifacts.elastic.co/downloads/kibana/kibana-8.6.0-linux-x86_64.tar.gz
  logstash:https://artifacts.elastic.co/downloads/logstash/logstash-8.6.0-linux-x86_64.tar.gz

  filebeat:https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.6.0-linux-x86_64.tar.gz

  ```sh
  mkdir /opt/elk && cd /opt/elk
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.6.0-linux-x86_64.tar.gz
  wget https://artifacts.elastic.co/downloads/kibana/kibana-8.6.0-linux-x86_64.tar.gz
  wget https://artifacts.elastic.co/downloads/logstash/logstash-8.6.0-linux-x86_64.tar.gz
  wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.6.0-linux-x86_64.tar.gz
  tar -zxvf elasticsearch-8.6.0-linux-x86_64.tar.gz
  tar -zxvf kibana-8.6.0-linux-x86_64.tar.gz
  tar -zxvf filebeat-8.6.0-linux-x86_64.tar.gz
  
  chown -R elk.elk /opt/elk
  ```

- 环境

```sh
vi /etc/profile
# 在文件末尾添加
export ES_HOME=/opt/elk/elasticsearch-8.6.0
export KIBANA_HOME=/opt/elk/kibana-8.6.0
export LOGSTASH_HOME=/opt/elk/logstash-8.6.0
export BEAT_HOME=/opt/elk/filebeat-8.6.0
export PATH=$PATH:$ES_HOME/bin:$KIBANA_HOME/bin:$LOGSTASH_HOME/bin:$BEAT_HOME/bin

# 执行变更
source /etc/profile
```



- 用户

```sh
useradd -d /home/elk -m elk
echo 'abc@123'|passwd elk --stdin
```

或

```sh
adduser elk && passwd elk
```



------

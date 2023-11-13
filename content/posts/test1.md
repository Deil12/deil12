# ELK8部署、配置xpark认证

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

## 部署elasticsearch

1. 安装es

```sh
vim config/elasticsearch.yml
# 修改配置
node.name: elk-1
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["elk-1"]
```

###### 报错处理

```properties
bound or publishing to a non-loopback address, enforcing bootstrap checks
ERROR: [4] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2]: max number of threads [3795] for user [elastic] is too low, increase to at least [4096]
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[4]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
ERROR: Elasticsearch did not exit normally - check the logs at /home/elastic/elasticsearch-7.10.2/logs/elasticsearch.log
```

```sh
su root && vim /etc/security/limits.conf
# 修改最大进程数和最大线程数
# 在文件末尾添加
elk hard nofile 65536
elk soft nofile 65536
elk hard nproc 4096
elk soft nproc 4096
```

```sh
su root && vi /etc/sysctl.conf
# 在文件末尾添加
vm.max_map_count = 262144
echo "vm.max_map_count = 262144" >> /etc/sysctl.conf
# 执行变更
sysctl -p

# 控制台启动
./bin/elasticsearch
```



```properties
# 首次控制台启动后输出
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  kYF5WDRKJWA9H6cRdSG7

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  0955e99c2f6cb6a98329d4edf2681e604297a74910066f81698be23c904abd69

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjYuMCIsImFkciI6WyIxMC43OS4xMC4yOTo5MjAwIl0sImZnciI6IjA5NTVlOTljMmY2Y2I2YTk4MzI5ZDRlZGYyNjgxZTYwNDI5N2E3NDkxMDA2NmY4MTY5OGJlMjNjOTA0YWJkNjkiLCJrZXkiOiJPb3pnbzRzQmlZaTlwZFlObnl2Qjp3TWRQMjZFSlE2dXZKbWw0LXVBSFlnIn0=

ℹ️ Configure other nodes to join this cluster:
• Copy the following enrollment token and start new Elasticsearch nodes with `bin/elasticsearch --enrollment-token <token>` (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjYuMCIsImFkciI6WyIxMC43OS4xMC4yOTo5MjAwIl0sImZnciI6IjA5NTVlOTljMmY2Y2I2YTk4MzI5ZDRlZGYyNjgxZTYwNDI5N2E3NDkxMDA2NmY4MTY5OGJlMjNjOTA0YWJkNjkiLCJrZXkiOiJPSXpnbzRzQmlZaTlwZFlObnl1LTptUWYzNGhDLVM3Q1lsSG5NblpmMkFBIn0=

  If you're running in Docker, copy the enrollment token and run:
  `docker run -e "ENROLLMENT_TOKEN=<token>" docker.elastic.co/elasticsearch/elasticsearch:8.6.0`
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# 自动生成配置
xpack.security.enabled: true

xpack.security.enrollment.enabled: true

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12

# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12

# 且config目录下增加一个certs目录
# --> certs
#   --> http_ca.crt
#   --> http.p12
#   --> transport.p12
```



```sh
# 关闭控制台服务，后台启动；
ps -ef|grep elasticsearch|grep -v grep|awk '{print $2}'|xargs kill

./elasticsearch -d 

#自定义内置用户密码（方便对接）
./elasticsearch-reset-password -u elastic -i
./elasticsearch-reset-password -u kibana -i
```



------

## 部署kibana

```sh
vim config/kibana.yml
# 修改配置
server.port: 5601
server.host: x.x.x.x
elasticsearch.hosts: ["https://x.x.x.x:9200"]
elasticsearch.username: "kibana"
elasticsearch.password: "abc@123"
elasticsearch.ssl.certificateAuthorities: [ "/opt/elk/elasticsearch-8.6.0/config/certs/http_ca.crt" ]
i18n.locale: "zh-CN"
```

或

```sh
server.port: 5601
server.host: x.x.x.x
elasticsearch.hosts: ['https://x.x.x.x:9200']
elasticsearch.ssl.certificateAuthorities: [ "/opt/elk/elasticsearch-8.6.0/config/certs/http_ca.crt" ]
i18n.locale: "zh-CN"
elasticsearch.serviceAccountToken: AAEAAWVsYXN0aWMva2liYW5hL2Vucm9sbC1wcm9jZXNzLXRva2VuLTE2OTkzMTg5MDYwNzI6eEEtWHBCUjJSemF2eFNLTDRSTWYyQQ
xpack.fleet.outputs: [{id: fleet-default-output, name: default, is_default: true, is_default_monitoring: true, type: elasticsearch, hosts: ['https://x.x.x.x:9200'], ca_trusted_fingerprint: 0955e99c2f6cb6a98329d4edf2681e604297a74910066f81698be23c904abd69}]
```

```sh
# 控制台启动
./bin/kibana
```



###### 报错处理

（账密配置改为token配置）

```properties
[ERROR][elasticsearch-service] Unable to retrieve version information from Elasticsearch nodes. 
```

```sh
bin/kibana-setup --enrollment-token eyJ2ZXIiOiI4LjYuMCIsImFkciI6WyIxMC43OS4xMC4yOTo5MjAwIl0sImZnciI6IjA5NTVlOTljMmY2Y2I2YTk4MzI5ZDRlZGYyNjgxZTYwNDI5N2E3NDkxMDA2NmY4MTY5OGJlMjNjOTA0YWJkNjkiLCJrZXkiOiJPb3pnbzRzQmlZaTlwZFlObnl2Qjp3TWRQMjZFSlE2dXZKbWw0LXVBSFlnIn0=
```



```sh
# 关闭控制台服务，后台启动
netstat -tunlp | grep 5601
kill xxx

nohup ./bin/kibana > /dev/null &
exit
```



```properties
# 用开发工具卡的控制台定义pipeline
PUT _ingest/pipeline/xxx-log
{
  "description": "xxx-log",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{TIMESTAMP_ISO8601:log_time} [%{NOTSPACE:thread}] %{NOTSPACE:log_level} %{NOTSPACE:java_class} - %{GREEDYDATA:content}"]
      }
    },
    {
        "date": {
            "field": "log_time",
            "formats": ["yyyy-MM-dd HH:mm:ss.SSS"],
            "timezone": "Asia/Shanghai",
            "target_field": "@timestamp"
        }
    }
  ]
}
```



------

## ~~安装logstash~~

```sh
vim config/logstash.yml
# 修改配置
api.http.host: 0.0.0.0
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.username: elastic
xpack.monitoring.elasticsearch.password: abc@123
xpack.monitoring.elasticsearch.hosts: ["https://127.0.0.1:9200"]
xpack.monitoring.elasticsearch.ssl.certificate_authority: "/opt/elk/elasticsearch-8.6.0/config/certs/http_ca.crt"
xpack.monitoring.elasticsearch.ssl.ca_trusted_fingerprint: 0955e99c2f6cb6a98329d4edf2681e604297a74910066f81698be23c904abd69

cp config/logstash-sample.conf config/logstash-custom.conf && vim logstash-custom.conf
# 修改logstash-custom.conf配置
input {
  tcp {
    mode => "server"
    host => "10.79.10.29"
    port => 5040
    codec => json_lines
  }
  beats {
	port => "5044"
  }
}

output {
  elasticsearch {
    hosts => ["https://127.0.0.1:9200"]
    index => "tang-bg-%{[service_name]}-%{+YYYY.MM.dd}"
    ssl => true
    ssl_certificate_verification => false
    cacert => "/opt/elk/elasticsearch-8.6.0/config/certs/http_ca.crt"
    ca_trusted_fingerprint => "0955e99c2f6cb6a98329d4edf2681e604297a74910066f81698be23c904abd69"
    user => "elastic"
    password => "abc@123"
  }
  stdout {
    codec => rubydebug
  }
}
```

```sh
# 控制台启动
./bin/logstash -f ./config/logstash-custom.conf

# 后台启动
nohup ./bin/logstash -f ./config/logstash-custom.conf > /dev/null &
exit
```



------

## 安装filebeat

```sh
cp filebeat.yml filebeat.yml.bak && vim filebeat.yml

# 修改配置
filebeat.inputs:
#备注：日志有Error,warm,info,debug四种，所以每个服务要配置4个
- type: log
  enable: true
  paths:
    #监控的文件路径
    #- /opt/tools/elk/filebeat-8.6.0-linux-x86_64/data/registry/filebeat/*
    - /opt/xxxService-1.0.0/bin/logs/**/*.log
  #将自定义字段放在顶层发送
  fields_under_root: true
  #自定义字段
  fields:
    #本服务的ip
    runip: x.x.x.x
    #本服务的端口
    runPort: xxxx
    service_name: xxxService
  #日志的编码格式
  encoding: utf-8
  #排除redis信息行
  # exclude_lines: ['CSRedis','call method']
  # 多行日志合并
  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
  multiline.negate: true
  multiline.match: after

  
setup.ilm.enabled: false
setup.template.enabled: false
output.elasticsearch:
  hosts: ["https://x.x.x.x:9200"]
  username: "elastic"
  password: "abc@123"
  index: "tang-bg-%{[service_name]}-%{+yyyy.MM.dd}"
  ssl.verification_mode: certificate
  ssl.certificate_authorities: ["/opt/elk/elasticsearch-8.6.0/config/certs/http_ca.crt"]
  pipline: "xxx-log"
    
processors:
  - script:
      lang: javascript
      id: timestamp_filter
      tag: enable
      source: >
        function process(event) {
            var str= event.Get("message");
            var time =str.substring(0, 23).replace(",",".");
            event.Put("start_time",time);
        }
  - timestamp:
      field: start_time
      timezone: Asia/Shanghai
      layouts:
        - '2006-01-02 15:04:05'
        - '2006-01-02 15:04:05.999'
      test:
        - '2019-06-22 16:33:51'
```

```sh
# 控制台启动
./filebeat -e -c filebeat.yml

# 后台启动
nohup ./filebeat -e -c filebeat.yml > /dev/null &
exit
```

或

```sh
# 修改配置
vim filebeat.yml
filebeat.inputs:
 - type: log
 id: 1
 enable: true
 paths: 
   - /data/app/ap/logs/*.log  # 要采集的日志文件或路径
# output.elasticsearch:  # 由于本文架构是filebeat的output是到logstash，故关闭默认output.elasticsearch；
output.logstash:
    hosts: ["172.16.0.1:5041"]  # 这里的端口要与logstash-sample.conf配置里的一致；

- 配置完成，临时启动filebeat；
nohup ./filebeat -e -c filebeat.yml > /dev/null 2>&1

- 由于通过nohub方式启动filebeat，运行一段时间后filebeat自动退出；原因是filebeat默认会定期检测文件是否有新的内容，如果超过一定时间检测的文件没有新数据写入，那么filebeat会自动退出，解决办法就是将filebeat通过系统后台的方式长期运行；
    - 添加systemctl服务启动配置
    vim  /etc/systemd/system/filebeat.service
    
    [Unit]
    Description=Filebeat is a lightweight shipper for metrics.
    Documentation=https://www.elastic.co/products/beats/filebeat
    Wants=network-online.target
    After=network-online.target
    
    [Service]
    Environment="LOG_OPTS=-e"
    Environment="CONFIG_OPTS=-c /data/filebeat/filebeat-8.8.1-linux-x86_64/filebeat.yml"
    Environment="PATH_OPTS=-path.home /data/filebeat/filebeat-8.8.1-linux-x86_64/filebeat -path.config /data/filebeat/fileb
    eat-8.8.1-linux-x86_64 -path.data /data/filebeat/filebeat-8.8.1-linux-x86_64/data -path.logs /data/filebeat/filebeat-8.
    8.1-linux-x86_64/logs"
    ExecStart=/data/filebeat/filebeat-8.8.1-linux-x86_64/filebeat $LOG_OPTS $CONFIG_OPTS $PATH_OPTS
    Restart=always
    
    [Install]
    WantedBy=multi-user.target
    
    - 授予可执行权限
    chmod +x /etc/systemd/system/filebeat.service
    
    - 配置开机启动等
    systemctl daemon-reload
    systemctl enable filebeat
    systemctl start filebeat
```


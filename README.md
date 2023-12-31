# 11-03_elk
HW_11-3_ELK

# Домашнее задание к занятию 3 «ELK»

### Задание 1. Elasticsearch
Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный.

Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на
сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name.

Установка **Elasticsearch**:

```
apt update && apt install gnupg apt-transport-https
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add
echo "deb [trusted=yes] https://mirror.yandex.ru/mirrors/elastic/7/ stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
apt update && apt install elasticsearch
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```
Проверка состояния работы сервиса:
```
systemctl status elasticsearch.service
```
<kbd>![](img/systemctl_status_elasticsearch.png)</kbd>

Изменим **cluster_name** в **/etc/elasticsearch/elasticsearch.yml** на **mityaevgv**:

<kbd>![](img/cluster_name_elasticsearch_yml.png)</kbd>

Перезапустим сервис **elasticsearch.service**:
```
systemctl restart elasticsearch
```
Проверка параметров сервера:
```
curl -X GET 'localhost:9200/_cluster/health?pretty
```
<kbd>![](img/curl_elasticsearch.png)</kbd>
---
### Задание 2. Kibana

Установите и запустите Kibana.

Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, 
где будет выполнен запрос GET /_cluster/health?pretty.

Установим **Kibana**:
```
apt install kibana
systemctl daemon-reload
systemctl enable kibana.service
systemctl start kibana.service
```
Проверка состояния работы сервиса:
```
systemctl status kibana.service
```
<kbd>![](img/systemctl_status_kibana.png)</kbd>

Внесем изменения в конфигурационный файл **/etc/kibana/kibana.yml**:
```
server.port: 5601
server.host: "0.0.0.0"
```
<kbd>![](img/kibana_yml_amendments.png)</kbd>

Перезапустим сервис **kibana.service**:
```
systemctl restart kibana.service
```
Зайдем в веб-интерфейс **Elasticsearch** по адресу **http://192.168.1.95:5601/app/dev_tools#/console** и
сделаем запрос **GET /_cluster/health?pretty**:

<kbd>![](img/get_cluster_health_command.png)</kbd>

### Задание 3. Logstash

Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог
Nginx в Elasticsearch.

Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.

Установим **Logstash**:
```
apt install logstash
systemctl daemon-reload
systemctl enable logstash.service
systemctl start logstash.service
```
Проверка состояния работы сервиса:
```
systemctl status logstash.service
```
<kbd>![](img/systemctl_status_logstash.png)</kbd>

Создадим простой конфиг-файл **/etc/logstash/conf.d/nginx_logstash.conf**, который будет
передавать логи из **Nginx** в **Elasticsearch**:
```
input {
  file {
    path => "/var/log/nginx/access.log"
    type => "nginx"
    start_position => "beginning"
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    data_stream => "true"
  }
}
```
---
### Задание 4. Filebeat.

Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat.

Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.

Установим **Logstash**:
```
apt install filebeat
systemctl daemon-reload
systemctl enable filebeat.service
systemctl start filebeat.service
```
Проверка состояния работы сервиса:
```
systemctl status filebeat.service
```
<kbd>![](img/systemctl_status_filebeat.png)</kbd>

Внесем изменения в конфиг-файл **/etc/logstash/conf.d/nginx_logstash.conf** в **Logstash**:
```
input {
  beats {
    port => 5044
  }
}
```
Внесем изменения в конфиг-файл **/etc/filebeat/filebeat.yml**:
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
processors:
  - drop_fields:
      fields: ["beat", "input_type", "prospector", "input", "host", "agent", "ecs"]
output.logstash:
  hosts: ["localhost:5044"]
```
<kbd>![](img/kibana_filebeat_nginx.png)</kbd>

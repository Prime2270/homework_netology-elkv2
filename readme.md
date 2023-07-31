# Домашнее задание к занятию "`elk`" - `Яковлев Константин`

### Задание 1

Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный.

Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name.

![job1](https://github.com/Prime2270/homework_netology-elkv2/blob/main/screenshots/job1.png)

![job1elasicsearch]()

```
установка elasticsearch:

apt update && apt install gnupg apt-transport-https -y
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add
echo "deb [trusted=yes] https://mirror.yandex.ru/mirrors/elastic/7/ stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
apt update && apt install elasticsearch -y



systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
systemctl status elasticsearch.service


поменяем cluster_name в nano /etc/elasticsearch/elasticsearch.yml на yakovlevks:
редактируем поля: (cluster.name:) (network.host:) (http.port:)
добавим поля cluster.initial_master_nodes: node-1

cluster.name: yakovlevks
node.name: node-1
cluster.initial_master_nodes: node-1
network.host: "0.0.0.0"
http.port: 9200

перезапустим сервис

systemctl restart elasticsearch

сделаем запрос в консоли

curl -X GET 'localhost:9200/_cluster/health?pretty'
```
### Задание 2

Установите и запустите Kibana.

Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, где будет выполнен запрос GET /_cluster/health?pretty.

![job2](https://github.com/Prime2270/homework_netology-elkv2/blob/main/screenshots/job2.png)

![job2kibana]()

```
установим kibana:

apt install kibana -y
systemctl daemon-reload
systemctl enable kibana.service
systemctl start kibana.service
systemctl status kibana.service

Внесем изменения в конфигурационный файл nano /etc/kibana/kibana.yml:

server.port: 5601
server.host: "0.0.0.0"

перезапустим сервис

systemctl restart kibana.service

проверим elasticsearch по адресу http://<your ip>:5601/app/dev_tools#/console 
и сделаем запрос GET /_cluster/health?pretty:
```

### В процессе поднятия нового стека т.к. старая виртуалка перестала запускаться

### Задание 3

Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог Nginx в Elasticsearch.

Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.

![job3.1]()

![job3.2]()

![job3logstash]()

```
установка nginx

apt install nginx -y

изменим парва на логи
chmod -R o+r /var/log/nginx

Установим Logstash:

apt install logstash -y
systemctl daemon-reload
systemctl enable logstash.service
systemctl start logstash.service
systemctl status logstash.service



cоздадим конфиг-файл nano /etc/logstash/conf.d/nginx_to_logstash.conf, 
который будет передавать логи из Nginx в Elasticsearch:

input {
  file {
    path => "/var/log/nginx/access.log"
    type => "nginx"
    start_position => "beginning"
  }
}

filter {
        grok {
        match => { "message" => "%{IPORHOST:remote_ip} - %{DATA:user_name}
        \[%{HTTPDATE:access_time}\] \"%{WORD:http_method} %{DATA:url}
        HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code} %{NUMBER:body_sent_bytes}
        \"%{DATA:referrer}\" \"%{DATA:agent}\"" }
        }
        mutate {
        remove_field =>  "host"
        }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
	user => "root"
    data_stream => "true"
  }
}


проверка логов самого logstash

nano /var/log/logstash/logstash-plain.log
```

### Задание 4

Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat.

Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.

![job4]()

![job4filebeat]()

```
установка filebeat:

apt install filebeat -y
systemctl daemon-reload
systemctl enable filebeat.service
systemctl start filebeat.service
systemctl status filebeat.service

изменения в конфиг-файл nano /etc/logstash/conf.d/nginx_to_logstash.conf в Logstash:

input {
  beats {
    port => 5044
  }
}

systemctl restart logstash.service

изменения в конфиг-файл nano /etc/filebeat/filebeat.yml:

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

systemctl restart filebeat.service
```

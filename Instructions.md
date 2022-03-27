Пример реализации SIEM-системы с использованием Elasticsearch, Kibana и Logstash
=====================
Для выполнения понадобится:
* подключение к сети Интернет
* VM Ubuntu
* VM CentOS Stream в минимальной конфигурации.
Минимальные требования:
* 2 ядра процессора
* ОЗУ 6 Гб
* Объем диска 40 Гб.
Рекомендуемые требования:
* 4 ядра процессора
* ОЗУ 8 Гб
* Объем диска 60 Гб.
Установка компонентов на CentOS
-----------------------------------
### Elasticsearch (v.7.10.2) ссылка на ресурс: <https://www.elastic.co/guide/en/elasticsearch/reference/7.10/rpm.html>
```wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-x86_64.rpm
sudo rpm --install elasticsearch-7.10.2-x86_64.rpm```
Добавление сервиса в автозапуск и запуск сервиса
````sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service````
### Kibana (v.7.10.2) ссылка на ресурс: <https://www.elastic.co/guide/en/kibana/7.10/rpm.html>
````wget https://artifacts.elastic.co/downloads/kibana/kibana-7.10.2-x86_64.rpm
shasum -a 512 kibana-7.10.2-x86_64.rpm 
sudo rpm --install kibana-7.10.2-x86_64.rpm````
Добавление сервиса в автозапуск и запуск сервиса
````sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service````
### Logstash (v.7.10.2) ссылка на ресурс: <https://www.elastic.co/guide/en/logstash/7.10/installing-logstash.html>
Так как у нас уже имеется файл .rpm, загружаем файл в директорию и в этой директории пишем команды:
````wget logstash-7.10.2-x86_64.rpm
shasum -a 512 kibana-7.10.2-x86_64.rpm 
sudo rpm --install logstash-7.10.2-x86_64.rpm````
Добавление сервиса в автозапуск и запуск сервиса
````sudo systemctl daemon-reload
sudo systemctl enable logstash.service
sudo systemctl start logstash.service````
Установка компонентов на хостовой машине Windows
-----------------------------------
### Winlogbeat (v.7.10.2) ссылка на ресурс: <https://www.elastic.co/guide/en/beats/winlogbeat/7.10/winlogbeat-installation-configuration.html>
* Скачать .zip файл с сайта.
* Извлечь содержимое .zip в директорию C:\Program Files
* Переименовать папку в Winlogbeat
* открыть PowerShell (PS) в качестве администратора
* Написать следующие команды:
````cd 'C:\Program Files\Winlogbeat'
.\install-service-winlogbeat.ps1````
Конфигурация сервисов
-----------------------------------
### Elasticsearch
Конфигурационный файл расположен в /etc/elasticsearch/elasticsearch.yml.
Необходимо добавить в elasticsearch.yml в раздел Network следующие строки:
`network.bind_host: 0.0.0.0
transport.host: localhost
xpack.security.enabled: false`
_Строка_ `network.bind_host: 0.0.0.0` _включит_ _удаленный_ _доступ_ _к_ _серверу_ _Elasticsearch_.
_Значение_ _ip_ _0.0.0.0_ _противоречит_ _правилам_ _безопасности_, _в_ _данном_ _случае_ _используется_ _только_ _для_ _примера._
Перезапустить службу
````sudo systemctl restart elasticsearch````
### Kibana
Конфигурационный файл расположен в /etc/kibana/kibana.yml.
Необходимо изменить файл до следующего вида:
![Рисунок2](https://user-images.githubusercontent.com/77727504/160297208-a05da0ba-a381-476d-8441-26535b967285.png)

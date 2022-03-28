Пример реализации SIEM-системы с использованием Elasticsearch, Kibana и Logstash
=====================
Для выполнения понадобится:
* подключение к сети Интернет
* VM [Ubuntu 18.04](https://releases.ubuntu.com/18.04/)
* VM [CentOS 8 Stream](https://trendoceans.com/how-to-install-centos-stream-8-x86_64/) в минимальной конфигурации

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
### Elasticsearch (v.7.10.2)]
ссылка на ресурс: <https://www.elastic.co/guide/en/elasticsearch/reference/7.10/rpm.html>
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-x86_64.rpm
sudo rpm --install elasticsearch-7.10.2-x86_64.rpm
```
Добавление сервиса в автозапуск и запуск сервиса
````
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
````
### Kibana (v.7.10.2)
ссылка на ресурс: <https://www.elastic.co/guide/en/kibana/7.10/rpm.html>
````
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.10.2-x86_64.rpm
shasum -a 512 kibana-7.10.2-x86_64.rpm 
sudo rpm --install kibana-7.10.2-x86_64.rpm
````
Добавление сервиса в автозапуск и запуск сервиса
````
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
````
### Logstash (v.7.10.2)
ссылка на ресурс: <https://www.elastic.co/guide/en/logstash/7.10/installing-logstash.html>

Так как у нас уже имеется файл .rpm, загружаем файл в директорию и в этой директории пишем команды:
````
wget logstash-7.10.2-x86_64.rpm
shasum -a 512 kibana-7.10.2-x86_64.rpm 
sudo rpm --install logstash-7.10.2-x86_64.rpm
````
Добавление сервиса в автозапуск и запуск сервиса
````
sudo systemctl daemon-reload
sudo systemctl enable logstash.service
sudo systemctl start logstash.service
````
Установка компонентов на хостовой машине Windows
-----------------------------------
### Winlogbeat (v.7.10.2)
ссылка на ресурс: <https://www.elastic.co/guide/en/beats/winlogbeat/7.10/winlogbeat-installation-configuration.html>
* Скачать .zip файл с сайта.
* Извлечь содержимое .zip в директорию C:\Program Files
* Переименовать папку в Winlogbeat
* открыть PowerShell (PS) в качестве администратора
* Написать следующие команды:
````
cd 'C:\Program Files\Winlogbeat'
.\install-service-winlogbeat.ps1
````
Конфигурация сервисов
-----------------------------------
### Elasticsearch
Конфигурационный файл расположен в /etc/elasticsearch/elasticsearch.yml.
Необходимо добавить в elasticsearch.yml в раздел Network следующие строки:
```
network.bind_host: 0.0.0.0
transport.host: localhost
xpack.security.enabled: false
```
Строка `network.bind_host: 0.0.0.0` включит удаленный доступ к серверу Elasticsearch.
**Значение ip 0.0.0.0 противоречит правилам безопасности, в данном случае используется только для примера.
Перезапустить службу**
````sudo systemctl restart elasticsearch````
### Kibana
Конфигурационный файл расположен в /etc/kibana/kibana.yml.
Необходимо изменить файл до следующего вида:

![Рисунок2](https://user-images.githubusercontent.com/77727504/160297208-a05da0ba-a381-476d-8441-26535b967285.png)

Перезапустить Kibana
```
sudo systemctl restart kibana
```
Далее следует проверить настройки firewall. В данном случае его лучше выключить.
````
sudo systemctl disable firewalld.service
sudo systemctl start elasticsearch.service
````
Теперь есть возможность получить доступ к Kibana из браузера.
Зайти можно по ссылке `http://your_VM_IP:5601`
И выбрать "Explore my own"

![image](https://user-images.githubusercontent.com/77727504/160297847-79ffc0f8-aa6b-44c4-8beb-1e1d67b31179.png)
### Logstash
Конфигурационный файл Logstash содержится в /etc/logstash/conf.d/.
Скопируем пример конфигурационного файла в данную директорию, назовем logstash.conf и перезагрузим сервис:
```
cp /etc/logstash/logstash-sample.conf /etc/logstash/conf.d/logstash.conf
sudo systemctl restart logstash
```
Настройка объекта мониторинга (Winlogbeat)
-----------------------------------
Конфигурационный файл winlogbeat.yml имеет разрешение на изменение только для администраторов. Необходимо предварительно выдать права для обычных пользователей.
Для настройки отправки событий с Winlogbeat необходимо внести изменения в winlogbeat.yml:
```
output.elasticsearch:
  hosts: ["ip_SIEM:9200"]
```
и 
```
logging.to_files: true
logging.files:
  path: C:\ProgramData\winlogbeat\Logs
logging.level: info
```
Сохранить изменения в файле и проверить работу winlogbeat следующей командой:
```
PS C:\Program Files\Winlogbeat> .\winlogbeat.exe test config -c .\winlogbeat.yml -e
```
Настройка ресурсов winlogbeat командой:
```
PS > .\winlogbeat.exe setup -e
```
Команды запуска и остановки через PS:
```
PS C:\Program Files\Winlogbeat> Start-Service winlogbeat
PS C:\Program Files\Winlogbeat> Stop-Service winlogbeat
```
Посмотреть работу сервиса можно командой:
```
PS C:\Program Files\Winlogbeat> services.msc
```
Запись событий с прямым доступом до хранилища с помощью winlogbeat
-----------------------------------
Далее необходимо обратиться к Kibana в браузере и настроить ее на получение сообщений от Winlogbeat.
Открываем меню слева, заходим по пути "Stack Management"(в самом низу панели) > "Index Patterns" и создаем новый.
В поле "Index pattern name" пишем регулярное выражение "winlogbeat-*"

![image](https://user-images.githubusercontent.com/77727504/160298603-0e56da65-fe2b-473e-950b-8333d363dab5.png)

Далее в поле "Time field" указываем "event.created".
![image](https://user-images.githubusercontent.com/77727504/160298733-2a288956-4871-4bea-9f9a-093d27ecbb72.png)

Возвращаемся в левую панель меню и выбираем "Discover". Высветятся все логи от winlogbeat.
Есть возможность настроить фильтр по логам:

![image](https://user-images.githubusercontent.com/77727504/160298789-1f0c690d-552a-4d19-9d24-1128a8735c2f.png)

Запись событий с доступом через сервис Logstash
-----------------------------------
Сейчас используем второй вариант записи событий с помощью сервиса Logstash.
Для этого необходимо изменить winlogbeat.yml:
Закомментировать строки:
```
output.elasticsearch:
 #hosts: ["ip-адрес SIEM:9200"]
```
Найти строки и указать значения:
```
output.logstash:
  hosts: ["ip-адрес SIEM:5044"]
```
Теперь есть возможность настраивать логи с помощью сервиса logstash. Конфигурационный файл logstash.conf имеет функцию filter используемую для фильтрации и конфигурации логов.

ссылка на ресурс: <https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html>

Установка сервиса OpenDistro
-----------------------------------
На данный момент ELK настроены таким образом, что данные между сервисами передаются в открытом виде, что не соответствует нормам безопасности. Для обеспечения безопасного соединения используется OpenDistro. 
### OpenDistro Kibana
Установка производится из директории /usr/share/kibana.
```
sudo bin/kibana-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/kibana-plugins/opendistro-security/opendistroSecurityKibana-1.13.0.1.zip --allow-root
```
В конфигурационный файл /etc/kibana/kibana.yml необходимо внести следующие изменения:
![image](https://user-images.githubusercontent.com/77727504/160299959-10994ffc-ae98-48e1-baf8-9051b6609b8b.png)

### OpenDistro Elasticsearch
Установка производится из директории /usr/share/elasticsearch.
```
sudo bin/elasticsearch-plugin install https://d3g5vo6xdbdb9a.cloudfront.net/downloads/elasticsearch-plugins/opendistro-security/opendistro-security-1.13.1.0.zip
```
Необходимо утвердительно ответить на все вопросы при установке.
Далее происходит выпуск сертификатов для защищенного соединения. В нашем случае можно использовать только самоподписанные сертификаты.
```
cd /usr/share/elasticsearch/plugins/opendistro_security/tools/
chmod +x install_demo_configuration.sh
./install_demo_configuration.sh
```
При установке отвечаем на все вопросы "y".
### Настройка Logstash на безопасное соединение
Необходимо внести изменения в конфигурационный файл /etc/logstash/conf.d/logstash.conf

![image](https://user-images.githubusercontent.com/77727504/160300057-c84a5711-8d1b-465c-a93d-ae99efd30f57.png)

После всех внесенных изменений сервисы должны быть перезапущены.
```
systemctl restart elasticsearch
systemctl restart logstash
systemctl restart kibana
```
### Проверка работы настроенного защищенного соединения в браузере
При переходе на сайт kibana будет предоставлено окно для аутентификации (admin:admin).

![image](https://user-images.githubusercontent.com/77727504/160300143-716b74c7-a398-4bc6-9ef0-bfa978b45a9c.png)

Также теперь в сервисе Kibana появилась новая возможность создавать ролевую модель пользователя. Необходимо в меню выбрать "Security" и можно посмотреть всех внутренних пользователей.

![image](https://user-images.githubusercontent.com/77727504/160300224-7adc6108-b873-4abe-a43a-7e1eb38c2d6e.png)

Например, есть возможность создать пользователя с единственным правом чтения.

![image](https://user-images.githubusercontent.com/77727504/160300278-7631ca98-35f3-4c09-b057-1c622a5874ce.png)

Установка централизованного сбора логов с помощью rsyslog
-----------------------------------
ссылка на ресурс: <https://www.elastic.co/blog/how-to-centralize-logs-with-rsyslog-logstash-and-elasticsearch-on-ubuntu-14-04>

Было бы очень удобно, если бы логи со всех серверов сети собирались на одной машине. Здесь были бы все важные сообщения об ошибках и неполадках. Их можно было бы очень быстро проанализировать. Отправление логов на удаленный сервер (logstash) будет производиться с помощью сервиса rsyslog на VM Ubuntu.
### VM Ubuntu (rsyslog-сервер)
Проверим наличие и статус работы сервиса rsyslog
 ```
 systemctl status rsyslog
 ```
 В файле /etc/rsyslog.conf необходимо расскомментировать строки для отправки логов по UDP-протоколу:
 ![image](https://user-images.githubusercontent.com/77727504/160300947-e87bcebb-3662-4b89-834c-276aa4f6f231.png)
 
 Далее необходимо добавить в конец файла /etc/rsyslog.d/50-default.conf строку:
 ```
 *.*                                      @ip_Ubuntu:514
 ```
 ![image](https://user-images.githubusercontent.com/77727504/160301106-4131c427-202e-4a23-8692-d760aad0f00b.png)
 
 Далее необходимо перезагрузить rsyslog-сервис
 ```
 systemctl restart rsyslog
 ```
 Необходимо проверить работу ufw и отключить сервис (в лучшем случае необходимо было проверить открыт ли порт 514/udp).
 Elasticsearch требует, чтобы все документы, которые он получает, были в формате JSON, и rsyslog предоставляет способ сделать это с помощью шаблона. Создайте новый файл конфигурации для форматирования сообщений в формате JSON перед отправкой в Logstash:
 ```
 sudo nano /etc/rsyslog.d/01-json-template.conf
 ```
Содержимое файла:
```
template(name="json-template"
  type="list") {
    constant(value="{")
      constant(value="\"@timestamp\":\"")     property(name="timereported" dateFormat="rfc3339")
      constant(value="\",\"@version\":\"1")
      constant(value="\",\"message\":\"")     property(name="msg" format="json")
      constant(value="\",\"sysloghost\":\"")  property(name="hostname")
      constant(value="\",\"severity\":\"")    property(name="syslogseverity-text")
      constant(value="\",\"facility\":\"")    property(name="syslogfacility-text")
      constant(value="\",\"programname\":\"") property(name="programname")
      constant(value="\",\"procid\":\"")      property(name="procid")
    constant(value="\"}\n")
}
```
Далее создадим файл для настройки отправки rsyslog-сервера:
```
sudo nano /etc/rsyslog.d/60-output.conf
```
Содержимое:
```
# This line sends all lines to defined IP address at port 10514,
# using the "json-template" format template
*.*                         @private_ip_logstash:10514;json-template
```
\*.\* в начале означает обработку оставшейся части строки для всех сообщений журнала. Символы @означают использование UDP (@@ для TCP). IP-адрес или имя хоста после @, куда пересылать сообщения.
### CentOS (logstash)
Настроим logstash на получение сообщений от rsyslog.
Изменим файл /etc/logstash/conf.d/logstash.conf:
```
# This input block will listen on port 10514 for logs to come in.
# host should be an IP on the Logstash server.
# codec => "json" indicates that we expect the lines we're receiving to be in JSON format
# type => "rsyslog" is an optional identifier to help identify messaging streams in the pipeline.
input {
  udp {
    host => "ip_SIEM"
    port => 10514
    codec => "json"
    type => "rsyslog"
  }
}
# This is an empty filter block.  You can later add other filters here to further process
# your log lines
filter { }
# This output block will send all events of type "rsyslog" to Elasticsearch at the configured
# host and port into daily indices of the pattern, "rsyslog-YYYY.MM.DD"
output {
  if [type] == "rsyslog" {
    elasticsearch {
      hosts => [ "https://localhost:9200" ]
      index => "logstash-rsyslog"
      ssl_certificate_verification => "false"
      user => "admin"
      password => "admin"
    }
  }
}
```
Перезапустим службы:
Ubuntu
```
sudo systemctl restart rsyslog 
```
CentOS
```
sudo systemctl restart logstash
```
Запустим сайт Kibana в браузере. Проверим, прослушивается ли порт 10514
CentOS (logstash)
```
netstat -na | grep 10514
```
Чтобы установить ошибки работы Logstash, можно воспользоваться командой:
```
cd /usr/share/logstash/bin
./logstash -f /etc/logstash/conf.d/logstash.conf --verbose
```
На сайте Kibana создаем новый index pattern для работы с логами от rsyslog:
![image](https://user-images.githubusercontent.com/77727504/160301864-c30deb65-75b1-4b37-b837-f718b42c0ae9.png)

Полученные сообщения в заданном формате:

![image](https://user-images.githubusercontent.com/77727504/160301896-474c3ef9-ecdb-4748-bd74-16a1f8afc145.png)




 










 

# Домашнее задание к занятию "`Кластеризация и балансировка нагрузки`" - `Тен Денис`

### Задание 1
- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по [ссылке](2/)
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

### Решение Задание 1

Устанавливаем HAProxy
```
yum install haproxy && systemctl enable --now haproxy && systemctl status haproxy
```
![image](https://github.com/killakazzak/10-02-slb-cluster-hw/assets/32342205/b52a7428-84cd-4229-840b-fa90347eac9b)

Запускаем http python сервер для порта 8888

```
vim http_index_8888.py
```

```python
import http.server
import socketserver

PORT = 8888
SERVER_NAME = "Сервер 8888"

Handler = http.server.SimpleHTTPRequestHandler
Handler.extensions_map['.html'] = 'text/html'

class MyHandler(Handler):
    def log_message(self, format, *args):
        print(f"[{SERVER_NAME}] {format % args}")

    def do_GET(self):
        if self.path == '/':
            self.path = '/index.html'
        return Handler.do_GET(self)

with socketserver.TCPServer(("", PORT), MyHandler) as httpd:
    print(f"{SERVER_NAME} запущен на порту {PORT}")
    httpd.serve_forever()
```
```
python3 http_index_8888.py &
```

Запускаем http python сервер для порта 9999  
```
vim http_index_9999.py
```
```python
import http.server
import socketserver

PORT = 9999
SERVER_NAME = "Сервер 9999"

Handler = http.server.SimpleHTTPRequestHandler
Handler.extensions_map['.html'] = 'text/html'

class MyHandler(Handler):
    def log_message(self, format, *args):
        print(f"[{SERVER_NAME}] {format % args}")

    def do_GET(self):
        if self.path == '/':
            self.path = '/index1.html'
        return Handler.do_GET(self)

with socketserver.TCPServer(("", PORT), MyHandler) as httpd:
    print(f"{SERVER_NAME} запущен на порту {PORT}")
    httpd.serve_forever()
```
```
python3 http_index_9999.py &
```
Проверка
```
netstat -tulpn | grep -E '8888|9999'
```
![image](https://github.com/killakazzak/10-02-slb-cluster-hw/assets/32342205/bfe7774b-b58e-49aa-ad57-9d781216e327)

Настраиваем HAProxy
```
vim /etc/haproxy/haproxy.cfg
```
```
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

    # utilize system-wide crypto-policies
    ssl-default-bind-ciphers PROFILE=SYSTEM
    ssl-default-server-ciphers PROFILE=SYSTEM

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

listen stats # веб-страница со статистикой
bind :888
mode http
stats enable
stats uri /stats
stats refresh 5s
stats realm Haproxy\ Statistics

frontend frontend # секция фронтенд
mode http
bind :8088
default_backend backend

backend backend # секция бэкенд
mode http
balance roundrobin
server s1 127.0.0.1:8888 check
server s2 127.0.0.1:9999 check
```


Проверка
![image](https://github.com/killakazzak/10-02-slb-cluster-hw/assets/32342205/06d734b2-09fd-483f-a469-a085daf71416)

![image](https://github.com/killakazzak/10-02-slb-cluster-hw/assets/32342205/bb854518-6bc4-4f4e-b377-acc1b2c01dce)




---
### Задание 2
- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

### Решение Задание 2

---

## Задания со звёздочкой*
Эти задания дополнительные. Их можно не выполнять. На зачёт это не повлияет. Вы можете их выполнить, если хотите глубже разобраться в материале.

---

### Задание 3*
- Настройте связку HAProxy + Nginx как было показано на лекции.
- Настройте Nginx так, чтобы файлы .jpg выдавались самим Nginx (предварительно разместите несколько тестовых картинок в директории /var/www/), а остальные запросы переадресовывались на HAProxy, который в свою очередь переадресовывал их на два Simple Python server.
- На проверку направьте конфигурационные файлы nginx, HAProxy, скриншоты с запросами jpg картинок и других файлов на Simple Python Server, демонстрирующие корректную настройку.

### Решение Задание 3*

---

### Задание 4*
- Запустите 4 simple python сервера на разных портах.
- Первые два сервера будут выдавать страницу index.html вашего сайта example1.local (в файле index.html напишите example1.local)
- Вторые два сервера будут выдавать страницу index.html вашего сайта example2.local (в файле index.html напишите example2.local)
- Настройте два бэкенда HAProxy
- Настройте фронтенд HAProxy так, чтобы в зависимости от запрашиваемого сайта example1.local или example2.local запросы перенаправлялись на разные бэкенды HAProxy
- На проверку направьте конфигурационный файл HAProxy, скриншоты, демонстрирующие запросы к разным фронтендам и ответам от разных бэкендов.

### Решение Задание 4*


------

